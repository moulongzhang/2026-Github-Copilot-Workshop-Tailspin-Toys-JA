# [Data Base] games テーブルに資金調達フィールドを追加し、マイグレーションとシードデータを更新する

親 issue #9 の資金調達の進捗表示のうち、**データベース層**を担当する子 issue です。この pull request は Stacked Pull Requests の一番下の層であり、**依存する子 issue はありません**。最初に着手してください。後続の #11（Back End）はこの pull request の完了に依存します。

ここでは、`games` テーブルに資金調達に関するフィールドを追加し、Drizzle のマイグレーションを生成し、`db/games.csv` のシードデータと変換ロジックを更新します。

## 対象ファイル

- `db/schema.ts` — `games` テーブルへのカラム追加
- `db/migrations/` — `npm run db:generate` で生成される新しいマイグレーション
- `db/games.csv` — シードデータへの列追加
- `db/transforms.ts` — `GameCsvRow` 型と `parseGamesCsv()` の更新
- `db/seed.ts` — `games` への `insert` に新しいフィールドを追加
- `db/transforms.test.ts` — 変換ロジックのユニットテスト

## 受け入れ条件

- [ ] `db/schema.ts` の `games` テーブルに、目標金額 `fundingGoal`（`integer('funding_goal').notNull()`）、調達済み金額 `amountRaised`（`integer('amount_raised').notNull().default(0)`）、支援者数 `backerCount`（`integer('backer_count').notNull().default(0)`）を追加する
- [ ] `npm run db:generate` を実行して `db/migrations/` に新しいマイグレーションファイルを生成し、コミットに含める（既存のマイグレーションは編集しない）
- [ ] `db/games.csv` に `FundingGoal`・`AmountRaised`・`BackerCount` の列を追加し、既存のすべてのゲーム行に妥当な値（円単位の整数、`AmountRaised` は `FundingGoal` 以下）を設定する
- [ ] `db/transforms.ts` の `GameCsvRow` インターフェースに `fundingGoal`・`amountRaised`・`backerCount`（`number`）を追加し、`parseGamesCsv()` が新しい列を数値としてパースする
- [ ] `db/seed.ts` の `games` への `insert` に新しいフィールドを渡す
- [ ] `db/transforms.test.ts` に、新しい列がパースされることを検証する Vitest のユニットテストを追加する
- [ ] `npm run db:setup`（`db:migrate` と `db:seed`）がエラーなく完了し、`npm run test:unit` が成功する

## 補足

`AmountRaised` は `FundingGoal` を超えないようにし、達成間近・達成済み・序盤など進捗のばらつきがシード内に含まれるようにすると、後続の UI（#12）でプログレスバーの見え方を確認しやすくなります。
