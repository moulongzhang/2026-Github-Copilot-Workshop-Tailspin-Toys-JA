# [Back End] 資金調達フィールドをデータアクセス層に通し、進捗計算ヘルパーを追加する

親 issue #9 の資金調達の進捗表示のうち、**バックエンド（データアクセス）層**を担当する子 issue です。この pull request は Stacked Pull Requests の中間の層であり、**#10（Data Base）の完了に依存します**。#10 でスキーマとシードデータが用意されていることを前提に、その上に積み上げてください。後続の #12（Front End）はこの pull request の完了に依存します。

ここでは、#10 で追加した資金調達フィールドを型定義とデータアクセスヘルパーに通し、資金調達の進捗を計算する純粋関数を新設してユニットテストでカバーします。

## 対象ファイル

- `src/types/game.ts` — `Game` インターフェースへのフィールド追加
- `src/lib/games.ts` — `gameSelection`・`GameSelectionRow`・`mapGame()` の更新
- `src/lib/funding.ts` — 資金調達の進捗を計算する純粋関数（新規）
- `src/lib/funding.test.ts` — 進捗計算ヘルパーの Vitest ユニットテスト（新規）

## 受け入れ条件

- [ ] `src/types/game.ts` の `Game` インターフェースに `fundingGoal`・`amountRaised`・`backerCount`（いずれも `number`）を追加する
- [ ] `src/lib/games.ts` の `gameSelection`・`GameSelectionRow`・`mapGame()` を更新し、`getAllGames()` と `getGameById()` が新しいフィールドを返すようにする
- [ ] `src/lib/funding.ts` を新規作成し、`ratings.ts` と同様にフレームワーク非依存・副作用なしの純粋関数として、調達率を 0〜100 にクランプして返す `fundingPercent(amountRaised, fundingGoal)` と、目標達成済みかを判定する `isFunded(amountRaised, fundingGoal)` を実装する（`fundingGoal` が 0 の場合の 0 除算を安全に扱う）
- [ ] `src/lib/funding.test.ts` に、序盤・達成間近・達成済み・目標 0 などの境界を含む Vitest のユニットテストを追加する
- [ ] `npm run test:unit` と `npm run typecheck` が成功する

## 補足

`funding.ts` は Astro ランタイムに依存させず、`ratings.ts` と同じく純粋関数として実装してください。金額の表示整形（円のフォーマットなど）が必要であれば、この層の純粋関数として追加してもかまいません。
