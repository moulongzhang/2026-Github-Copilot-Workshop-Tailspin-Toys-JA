---
description: 'Astro アプリのための Drizzle ORM + Node SQLite データレイヤーのパターン'
applyTo: 'db/**/*.ts,src/lib/*.ts'
---

# Drizzle ORM + Node SQLite の指示

アプリのデータは、Node.js 組み込みの `node:sqlite` ドライバー上の **Drizzle ORM** を通じてアクセスされるローカルの SQLite データベースに存在します。これは Astro ページの frontmatter から**ビルド時**に消費されます — ランタイムの API サーバーはありません。スキーマの変更は **drizzle-kit** のマイグレーションで管理されます。

## レイアウト

- `db/schema.ts` — Drizzle のテーブル定義（`publishers`、`categories`、`games`）と推論された行の型。スキーマの唯一の信頼できる情報源です。
- `db/transforms.ts` — **純粋**関数（CSV パース、説明文の構築、重複排除、決定的な `ratingFromTitle`）。DB アクセスなし — ユニットテストが容易です。
- `db/seed.ts` — transforms を使った `db/games.csv` からの冪等なシード。
- `db/migrate.ts` — 生成されたマイグレーションを適用します。
- `db/migrations/` — 生成された SQL マイグレーション（手で編集しないでください）。
- `db/test-helpers.ts` — `createTestDatabase()` はテスト用にマイグレーション済みのインメモリ Node SQLite db を返します。
- `src/lib/db.ts` — `createDatabase(url)` / `getDatabase()` は `DATABASE_URL`（デフォルトはローカルの `tailspin.db` ファイル）から Drizzle クライアントを構築します。
- `src/lib/games.ts` — ページとテストで使用される、型付き・**注入可能な db** のデータアクセスヘルパー。

## スキーマの規約

- 明示的なカラム名（`text`、`integer`、`real`）で `sqliteTable` を使用してください。
- 主キー: `integer('id').primaryKey({ autoIncrement: true })`。
- 必須カラムには `.notNull()` を付け、null 許容カラム（例: `starRating`）は null 許容のままにしてください。
- 外部キーは `.references(() => other.id)` を使用してください。
- 推論された型（`typeof table.$inferSelect`）をエクスポートし、それらからアプリ向けの型を構築してください — 行の形を手で再宣言しないでください。

## マイグレーションのワークフロー

1. `schema.ts` を編集します。
2. マイグレーションを生成します: `npm run db:generate`（drizzle-kit）。
3. ローカルで適用 + シードします: `npm run db:setup`（`db:migrate` + `db:seed`）。
4. スキーマの変更**と** `db/migrations/` 内の生成されたマイグレーションの両方をコミットします。

> [!IMPORTANT]
> データベースは `astro build` の**前に**マイグレーションとシードが必要です。`prebuild`/`predev` の npm スクリプトが `db:setup` を自動的に実行し、CI はこの順序に依存しています。

## データアクセスヘルパー（注入可能な db）

ヘルパーは最初の引数として `db` インスタンスを受け取るため、実際のクライアント（ページ内）とインメモリクライアント（テスト内）の両方で動作します:

```ts
import { asc, count, eq } from 'drizzle-orm';
import type { Database } from './db';
import { games } from '../../db/schema';

export async function getAllGameIds(db: Database): Promise<number[]> {
  const rows = await db.select({ id: games.id }).from(games).orderBy(asc(games.title));
  return rows.map((r) => r.id);
}
```

- 静的ビルドが決定的になるよう、常に安定したカラム（title）で `order by` してください。
- 生の行をアプリ向けの `Game`/`Publisher`/`Category` 型に一箇所でマッピングし、Drizzle の行の形をコンポーネントに漏らさないでください。
- 順序付け/検索のロジックは、ページではなく `games.ts` に保ってください。

## 決定性

シード由来の値は、ビルド間で再現可能でなければなりません。star レーティングは title の安定したハッシュ（`ratingFromTitle`）から導出してください — **決して** `Math.random()` を使わないでください。

## テスト

transforms は直接ユニットテストし、ヘルパーは `createTestDatabase()` に対してテストしてください。[`unit-tests.instructions.md`](unit-tests.instructions.md) を参照してください。

## Node.js の要件

データレイヤーは実験的フラグなしで組み込みの `node:sqlite` モジュールを使用するため、Node.js 22.13 以降が必要です。プラットフォーム固有のバイナリを同梱するサードパーティの SQLite ドライバーを導入しないでください。

## 型チェック

データレイヤー（`db/**/*.ts`、`src/lib/*.ts`）は `npm run typecheck` によって型チェックされ、これは `tsconfig.tsgo.json` に対してネイティブの **TypeScript 7** コンパイラ（`tsgo`、`@typescript/native-preview` 由来）を実行します。`tsgo` が検証できるよう、ヘルパーは明示的なパラメーターと戻り値の型を付けてエクスポートしたままにしてください。リントは影響を受けません — ESLint + `typescript-eslint` は引き続きクラシックな `typescript` パッケージで実行されます。
