---
name: quality-checks
description: このプロジェクトのすべてのテスト、lint、品質チェックの実行を担当します — Vitest のユニットテスト、Playwright の E2E テスト、ESLint の実行、失敗のデバッグ、コード変更の検証、commit・push・merge 前の準備状態の確認を行います。test、lint、検証コマンド（`npm run test:unit`、`npm run test:e2e`、`npm run lint` など）を直接実行する代わりに、このスキルを使用してください。
allowed-tools:
  - shell
---

# 品質チェック（quality-checks）

これは単一の Astro アプリケーション（Astro 7 + Drizzle ORM / Node SQLite）です。すべてのコマンドは npm スクリプト経由でリポジトリのルートから実行します。

## クイックリファレンス

| テストスイート | コマンド | 使用するタイミング |
|------------|----------------------------|-------------|
| ユニットテスト (Vitest) | `npm run test:unit` | データ層 / 変換 / ヘルパーを変更した後 |
| フロントエンド E2E テスト (Playwright) | `npm run test:e2e` | UI / ページ / コンポーネントを変更した後 |
| Lint (ESLint) | `npm run lint` | TypeScript または Astro を変更した後 |
| 型チェック (tsgo + astro check) | `npm run typecheck:all` | TypeScript または Astro を変更した後 |

すべてのコマンドは、依存関係がインストール済み（`npm ci`）であること、および E2E については Playwright の Chromium ブラウザが利用可能（`npx playwright install chromium`）であることを前提としています。

---

## 検証スイートの実行

### ユニットテスト

```bash
npm run test:unit
```

- `db/**/*.test.ts` と `src/**/*.test.ts` に対して Vitest（`vitest run`）を実行します。
- 純粋な seed / transform 関数と Drizzle のデータアクセスヘルパーを、インメモリの Node SQLite データベースに対してテストします。

### フロントエンド E2E テスト

```bash
npm run test:e2e
```

- Playwright の `webServer` は、まず静的サイトを**ビルド**し（`prebuild` スクリプトが `db:migrate` + `db:seed` を実行）、`astro preview` でポート 4321 上に配信します。
- ビルドされた `dist/` の出力に対して、`e2e-tests/` 内のすべての Playwright スペックを実行します（ホームページ、ゲーム一覧 / 詳細ページ、アクセシビリティ、404）。

### Lint

```bash
npm run lint
```

- プロジェクト内のすべての TypeScript および Astro ファイルに対して ESLint を実行します。
- commit する前にエラーがゼロで通過する必要があります。

### 型チェック

```bash
npm run typecheck:all
```

- `npm run typecheck` は、ネイティブの **TypeScript 7** コンパイラ（`@typescript/native-preview` の `tsgo`）を、純粋な TypeScript（`db/`、`src/lib/`、`src/types/`、各種設定、テスト）に対して `tsconfig.tsgo.json`（`--noEmit`）経由で実行します。
- `npm run typecheck:astro` は `astro sync` を実行した後、`.astro` ファイルに対して `astro check` を実行します（従来の `typescript` パッケージを使用）。
- 型チェックは lint とは独立しています — `tsgo` は ESLint に影響せず、ESLint は引き続き従来の `typescript` パッケージを使用します。commit する前に、両方ともエラーがゼロで通過する必要があります。

---

## デバッグ & トラブルシューティング

### 環境 / セットアップの失敗

**症状**: `command not found`、モジュールの欠落、または `Cannot find package`。

```bash
npm ci
npx playwright install --with-deps chromium   # E2E の場合のみ必要
```

- Node 22.13 以降が利用可能であることを確認してください: `node --version`。
- 生成された Astro の型が欠落しているというエディタ / 型エラーが出る場合は `npx astro sync` を実行してください。

---

### データベース / ビルド時のデータ

**症状**: 空のページ、`no such table`、またはゲームページが生成されないビルド。

SQLite データベースは `astro build` の**前に** migrate と seed を実行する必要があります。`prebuild` / `predev` スクリプトが自動的に実行しますが、手動で実行することもできます:

```bash
npm run db:setup     # db:migrate + db:seed
```

- データベースは `tailspin.db`（gitignore 対象）にあり、`db/games.csv` から再生成されます。
- クリーンな再ビルドを強制するには: `rm -f tailspin.db && rm -rf dist && npm run build`。

---

### ポートの競合

**症状**: ポート 4321 で `Address already in use`。

```bash
lsof -ti :4321 | xargs kill
```

その後、失敗したコマンドを再実行してください。別のチェックアウトから残った古い `astro dev` / `astro preview` サーバーに注意してください — Playwright はローカルで 4321 上の既存サーバーを再利用します。

---

### Playwright / E2E テストの失敗

**症状**: テストのタイムアウト、要素が見つからない、または誤った HTTP ステータス。

1. **ブラウザがインストールされていない**: `npx playwright install --with-deps chromium`。
2. **古いサーバーの再利用**: 4321 上に残った dev / preview サーバーが古い HTML を配信することがあります。これを停止し（ポートの競合を参照）、`webServer` が再ビルドするよう再実行してください。
3. **Locator の変更**: `data-testid` がリネームまたは削除された場合は、スペックを一致するよう更新してください。
4. **404 の期待値**: 存在しないゲーム ID（例: `/game/99999`）は、静的出力では**本物の 404** です — ページ内のエラーメッセージではなく、`not-found` の testid に対してアサートしてください。
5. **不安定なテスト**: ハードコードされた待機を、自動リトライされる web-first アサーションに置き換えてください（[playwright.instructions.md](../../instructions/playwright.instructions.md) を参照）。**`waitForTimeout` は絶対に使用しないでください。**

より速く反復するために、単一のスペックを実行します:

```bash
npx playwright test e2e-tests/games.spec.ts
```

---

### ユニットテストの失敗

**症状**: `npm run test:unit` でのアサーション失敗。

1. **失敗したアサーションを読む** — Vitest は期待値と受信値をインラインで表示します。
2. **インメモリデータベース**: ヘルパーのテストは、テストごとに新しい `:memory:` の Node SQLite データベースを構築し、マイグレーションを実行し、フィクスチャを seed します。スキーマ変更が反映されない場合は、`npm run db:generate` でマイグレーションを再生成してください。
3. **決定性**: 星評価はタイトルの安定したハッシュ（`ratingFromTitle`）から導出され、`Math.random` は決して使用しません。評価アサーションが不安定な場合、通常は非決定的なデータが紛れ込んだことを意味します。

単一のファイルを実行します:

```bash
npx vitest run src/lib/games.test.ts
```

---

### Lint の失敗

**症状**: `npm run lint` からの ESLint エラー。

1. **安全な問題を自動修正する**: `npm run lint -- --fix`。
2. **未使用の変数**: 意図的に未使用の識別子には `_` を接頭辞として付けてください。
3. **TypeScript の型エラー**: 欠落している型注釈を追加するか、誤った型を修正してください。
4. **`--fix` 後に残るエラー**: 手動で解決してください — 正当な理由なく `eslint-disable` で抑制しないでください。

---

### ローカルと CI の差異

**症状**: ローカルではテストが通るが CI では失敗する（またはその逆）。

- **Node バージョンの不一致**: CI は現在の Node LTS リリースを使用します。
- **データベースの状態**: CI は常にクリーンな seed からビルドします。ローカルでは、古いデータが疑われる場合は `tailspin.db` を削除して再ビルドしてください。
- **ビルド版と dev**: CI は `astro preview` を介してビルドされた `dist/` をテストします。ローカルでは `astro dev` に対してではなく、（先にビルドを行う）`npm run test:e2e` で再現してください。

---

## 検証ポリシー

### commit / merge の前にテストが通過する必要があります

- 変更を commit する前に、既存のすべてのテストが通過する必要があります
- 明示的な正当化なしにテストをスキップまたは無効化しないでください
- 壊れたテストは merge をブロックします — 無視せず修正してください
- 変更したコードのテストだけでなく、テストスイート全体を実行してください
- 新しい機能には適切なテストカバレッジを付けて提供する必要があります

> [!NOTE]
> このスキルはテストの**実行、検証、デバッグ**を扱います。テストコードの**書き方** — 構造、フィクスチャ、命名、locator、品質基準 — については、唯一の信頼できる情報源である instructions ファイルに従ってください:
> - ユニットテスト（`**/*.test.ts`）: [unit-tests.instructions.md](../../instructions/unit-tests.instructions.md)
> - フロントエンド E2E（`e2e-tests/*.spec.ts`）: [playwright.instructions.md](../../instructions/playwright.instructions.md)

---

## commit 前チェックリスト

1. lint を実行する（フロントエンドのファイルを変更した場合）: `npm run lint`
2. 型チェックを実行する（TypeScript / Astro ファイルを変更した場合）: `npm run typecheck:all`
3. ユニットテストを実行する（データ層 / ヘルパーを変更した場合）: `npm run test:unit`
4. E2E テストを実行する（UI を変更した場合）: `npm run test:e2e`
5. 新しい機能に適切なテストカバレッジがあることを確認する
6. テストが壊れたり、スキップされたり、無効化されたりしていないことを確認する
