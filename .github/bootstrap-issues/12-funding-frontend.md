# [Front End] 資金調達の進捗をプログレスバーとして一覧と詳細に表示する

親 issue #9 の資金調達の進捗表示のうち、**フロントエンド（UI）層**を担当する子 issue です。この pull request は Stacked Pull Requests の一番上の層であり、**#11（Back End）の完了に依存します**。#11 で型・データアクセスヘルパー・`src/lib/funding.ts` の進捗計算関数が用意されていることを前提に、その上に積み上げてください。

ここでは、資金調達の進捗を表示するプログレスバーコンポーネントを追加し、ゲーム一覧ページとゲーム詳細ページで表示し、Playwright の e2e テストでカバーします。

## 対象ファイル

- `src/components/FundingProgress.astro` — プログレスバーコンポーネント（新規）
- `src/components/GameCard.astro` — 一覧カードへの組み込み
- `src/pages/game/[id].astro` — 詳細ページへの組み込み
- `e2e-tests/funding.spec.ts` — Playwright の e2e テスト（新規）

## 受け入れ条件

- [ ] `src/components/FundingProgress.astro` を新規作成し、`StarRating.astro` と同じパターンで `src/lib/funding.ts` の `fundingPercent()` などのヘルパーを使ってプログレスバーを描画する。調達率（%）・調達済み金額・目標金額・支援者数（`backerCount`）が読み取れるようにする
- [ ] `src/components/GameCard.astro`（一覧カード）と `src/pages/game/[id].astro`（詳細ページ）に `FundingProgress` を組み込む
- [ ] プログレスバーに `role="progressbar"` と `aria-valuenow`・`aria-valuemin`・`aria-valuemax` を付与し、プロジェクトのアクセシビリティガイドライン（スクリーンリーダー対応、色以外での情報伝達）に従う
- [ ] コンポーネントとその主要要素に `data-testid` 属性（例: `funding-progress`・`funding-percent`・`backer-count`）を付与する
- [ ] `e2e-tests/funding.spec.ts` を新規作成し、一覧ページと詳細ページで進捗バーおよび支援者数が表示されることを Playwright の e2e テストで検証する
- [ ] スタイリングは Tailwind CSS を用い、既存コンポーネントのデザインと調和させる
- [ ] `npm run test:e2e` が成功する

## 補足

このコンポーネントはプレゼンテーションに徹し、進捗率や達成判定のロジックは #11 で追加した `src/lib/funding.ts` の純粋関数に委ねてください。UI 側でパーセント計算を重複実装しないようにします。
