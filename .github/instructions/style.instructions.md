---
description: 'Tailwind CSS v4 のスタイリングパターンとダークテーマのガイドライン'
applyTo: '**/*.{astro,css}'
---

# Tailwind CSS の指示

## Tailwind CSS v4 の設定

このプロジェクトは `@tailwindcss/vite` プラグインを介して Tailwind CSS v4.1.14 を使用します。

### グローバル CSS のセットアップ

- `global.css` で Tailwind をインポートします: `@import "tailwindcss";`
- 別個の `tailwind.config.js` ファイルは使用しません
- 設定は Vite プラグインを通じて処理されます

## ダークテーマのスタイリング

すべての UI コンポーネントはダークテーマの色を使用しなければなりません:

### カラーパレット

- 背景色: `bg-slate-800`、`bg-slate-900`、`bg-slate-950`
- テキスト色: `text-slate-100`、`text-slate-200`、`text-slate-300`
- ボーダー色: `border-slate-700`、`border-slate-600`
- hover/focus 状態のためのアクセントカラー

### 共通パターン

- カードとコンテナ: `bg-slate-800 rounded-xl p-6 shadow-lg`
- hover エフェクト: `hover:bg-slate-700 transition-colors duration-200`
- ボーダー: `border border-slate-700`
- 視覚的な変化のためのグラデーション: `bg-gradient-to-br from-slate-800 to-slate-900`
- 背景エフェクト: `backdrop-blur-sm bg-slate-900/50`

### レスポンシブデザイン

- レスポンシブプレフィックスを使用してください: `sm:`、`md:`、`lg:`、`xl:`
- モバイルファーストのアプローチ
- すべての画面サイズで読みやすさを確保してください

## ユーティリティクラス

- 可能な限りカスタム CSS よりユーティリティクラスを優先してください
- 意味的なグループ化を使用してください: レイアウト、スペーシング、色、タイポグラフィ
- ユーティリティの組み合わせを読みやすく保守しやすい状態に保ってください

## モダンな UI パターン

- 角丸: `rounded-lg`、`rounded-xl`、`rounded-2xl`
- スムーズなトランジション: `transition-all duration-200 ease-in-out`
- 奥行きのためのシャドウ: `shadow-md`、`shadow-lg`、`shadow-xl`
- アクセシビリティのための focus 状態: `focus:ring-2 focus:ring-blue-500`
