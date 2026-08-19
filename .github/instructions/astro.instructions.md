---
description: 'ページ、レイアウト、コンポーネント、ルーティングのための Astro コンポーネントパターン'
applyTo: '**/*.astro'
---

# Astro コンポーネントの指示

## Astro コンポーネントのパターン

Astro は UI のすべて（ページ、レイアウト、コンポーネント、ルーティング、コンテンツ）を扱います。サイトは**完全にプリレンダリング**されます（`output: 'static'`）— クライアントサイドの UI フレームワークも別個の API サーバーもありません。ページは `src/lib/` にある Drizzle/Node SQLite のデータアクセスヘルパーを介して、ビルド時に **frontmatter で直接**データを読み込みます。

### コンポーネントの構造

```astro
---
// Frontmatter: ビルド時に実行される（静的出力）
import Layout from '../layouts/Layout.astro';
import GameCard from '../components/GameCard.astro';
import { getDatabase } from '../lib/db';
import { getAllGames } from '../lib/games';

interface Props {
  title: string;
}

const { title } = Astro.props;
const games = await getAllGames(getDatabase());
---

<Layout title={title}>
  {games.map((game) => <GameCard {game} />)}
</Layout>
```

## レイアウト

- 再利用可能なレイアウトコンポーネントを `src/layouts/` に作成してください
- コンテンツの注入には `<slot />` を使用してください
- 共通要素を含めてください: `<head>`、ナビゲーション、フッター
- レイアウトでグローバルスタイルをインポートしてください

### レイアウトの例

```astro
---
interface Props {
  title: string;
}
const { title } = Astro.props;
---

<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width" />
    <title>{title}</title>
  </head>
  <body>
    <slot />
  </body>
</html>
```

## ページ

- ページは `src/pages/` に作成してください
- ファイルベースのルーティング: `src/pages/about.astro` → `/about`
- 動的ルート: `src/pages/game/[id].astro`
- ブランド化された `src/pages/404.astro` を用意してください — 静的出力では、生成されたページのない URL はすべて実際の 404 になります。

### 動的ルート（静的出力）

`output: 'static'` では、すべての動的ルートが `getStaticPaths()` でそのページを列挙し、`prerender = true` を設定する必要があります。データアクセスヘルパーを使って frontmatter でデータをクエリしてください:

```astro
---
import type { GetStaticPaths } from 'astro';
import Layout from '../../layouts/Layout.astro';
import { getDatabase } from '../../lib/db';
import { getAllGameIds, getGameById } from '../../lib/games';

export const prerender = true;

export const getStaticPaths = (async () => {
  const ids = await getAllGameIds(getDatabase());
  return ids.map((id) => ({ params: { id: String(id) } }));
}) satisfies GetStaticPaths;

const { id } = Astro.params;
const game = await getGameById(getDatabase(), Number(id));
---

<Layout title="Game Details - Tailspin Toys">
  <!-- ゲームの詳細 -->
</Layout>
```

## データアクセス

- ビルド時のデータは、**Drizzle ORM + Node SQLite** を介してローカルの SQLite データベースから取得されます（[`drizzle.instructions.md`](drizzle.instructions.md) を参照）。
- `src/lib/db.ts` から `getDatabase()` を、`src/lib/games.ts` から型付けされたヘルパーをインポートしてください。
- データベースは `astro build` の前にマイグレーションとシードが必要です。`prebuild` の npm スクリプト（`db:setup`）がこれを処理します。

## クライアントインタラクティビティ（まれ）

Svelte/React のレイヤーはありません。ページに本当にクライアントの動作が必要な場合は、標準の DOM API を使ってスコープ付きの Astro `<script>` を追加してください。キーボードとフォーカスの動作が自動的に得られるよう、ネイティブのインタラクティブ要素（`<button>`、`<a href>`）を優先してください。

## TypeScript

- 型安全な props のために TypeScript を使用してください
- frontmatter で `Props` インターフェースを定義してください
- コンポーネントのインポートとヘルパーの戻り値に型を付けてください
- リントや型チェックの前に、`npx astro sync` を実行してルート/コンテンツの型を（再）生成してください
- `.astro` ファイルは `npm run typecheck:astro`（`astro sync` を実行してから `astro check` を実行）によって、クラシックな `typescript` パッケージで型チェックされます。`db/`、`src/lib/`、`src/types/` の純粋な TypeScript は、`npm run typecheck`（ネイティブの TS 7 コンパイラ、`tsgo`）によって別途型チェックされ、これは `.astro` ファイルを処理**しません**。

## ベストプラクティス

- データの取得は frontmatter（ビルド時）に保ち、クライアントサイドでの取得は避けてください
- クライアントサイドの JavaScript を最小限にしてください — デフォルトでは JS は一切出力されません
- レイアウトからグローバル CSS スタイルをインポートして使用してください
- インタラクティブ要素には常に `data-testid` を含めてください（`ui.instructions.md` を参照）
