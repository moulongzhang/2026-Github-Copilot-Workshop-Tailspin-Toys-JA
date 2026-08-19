---
description: 'Astro + Drizzle/Node SQLite データレイヤーのための Vitest ユニットテストガイドライン'
applyTo: '**/*.test.ts'
---

# ユニットテストのガイドライン（Vitest + Drizzle/Node SQLite）

ユニットテストは **Vitest**（`npm run test:unit`）で実行されます。これらは最も価値の高い、フレームワークに依存しない 2 つのレイヤーをカバーします:

1. **純粋な transforms**（`db/transforms.ts`）— CSV パース、説明文の構築、重複排除、決定的なレーティング。
2. **データアクセスヘルパー**（`src/lib/games.ts`）— 順序付け、検索 — 実際のインメモリ **Node SQLite** データベースに対して実行されます。

> [!IMPORTANT]
> テストは Astro ランタイムから独立させてください。ヘルパーは**注入可能な `db`** 引数を受け取ります。テストはインメモリデータベースを渡し、ページは実際のクライアントを渡します。データロジックをユニットテストするために Astro サーバーを起動しないでください。

## ファイル構造

- テストはコードの隣に配置してください: `transforms.test.ts` を `transforms.ts` の隣に、`games.test.ts` を `games.ts` の隣に。
- 命名パターン: `<module>.test.ts`。
- `describe('<module / function>')` ブロックと `it('does X when Y')` のケースを使用してください。
- ヘルパーとフィクスチャに型注釈を追加してください — このコードベースは明示的な型を必要とします。

## 純粋な transforms のテスト

- データベースは不要です — 関数をインポートして、その出力をアサートしてください。
- カバーする内容: ハッピーパス、空/空白の入力、オプションフィールドが欠けている行、重複排除、そして**決定性**（例: `ratingFromTitle` は同じ title に対して同じ値を返し、3.0〜5.0 の範囲内に収まる）。
- 入力/出力のマトリックスには `it.each` を使ったテーブル駆動のケースを優先してください。

```ts
import { describe, it, expect } from 'vitest';
import { ratingFromTitle } from './transforms';

describe('ratingFromTitle', () => {
  it('is deterministic and within range', () => {
    const a = ratingFromTitle('Code Quest');
    const b = ratingFromTitle('Code Quest');
    expect(a).toBe(b);
    expect(a).toBeGreaterThanOrEqual(3.0);
    expect(a).toBeLessThanOrEqual(5.0);
  });
});
```

## データアクセスヘルパーのテスト

- 共有ヘルパー `createTestDatabase()`（`db/test-helpers.ts`）を使って、テストごとに新しいインメモリデータベースを構築してください。これは `:memory:` の Node SQLite クライアントに対してマイグレーションを実行します。
- テストが必要とするフィクスチャのみをシードし、その `db` を使ってヘルパーを呼び出してください。
- 常に安価なもの（カウント、合計、順序）を先にアサートしてから、深いオブジェクトの形をアサートしてください。

```ts
import { describe, it, expect, beforeEach } from 'vitest';
import { createTestDatabase } from '../../db/test-helpers';
import { getAllGames, getGameById } from './games';

describe('getAllGames', () => {
  let db: Awaited<ReturnType<typeof createTestDatabase>>;

  beforeEach(async () => {
    db = await createTestDatabase();
    // …publishers、categories、games をシードする…
  });

  it('returns games ordered by title with their relations', async () => {
    const games = await getAllGames(db);
    const titles = games.map((g) => g.title);
    expect(titles).toEqual([...titles].sort());
    expect(games[0].category).not.toBeNull();
  });
});
```

## 必須のカバレッジ

- 有効なデータでの成功ケース
- 見つからないケース（存在しない id に対する `getGameById` は `null` を返す）
- 空のデータベース/コレクションのシナリオ
- 順序の保証（title のアルファベット順）— 静的ビルドはこれが決定的であることに依存します
- シード由来の値の決定性

## ベストプラクティス

- Arrange-Act-Assert に従ってください。
- `it` ごとに 1 つの振る舞い。1 つのケースで無関係なことをアサートしないでください。
- データベースをモックしないでください — インメモリの Node SQLite インスタンスは高速で、実際の SQL/join を実行します。
- フィクスチャは最小限にしつつ、リレーション（game → publisher、game → category）を代表するものにしてください。
- スキーマの変更でテストが壊れた場合は、`npm run db:generate` でマイグレーションを再生成し、フィクスチャを更新してください。
