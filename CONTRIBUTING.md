# Tailspin Toys へのコントリビュート

[fork]: https://github.com/github-samples/tailspin-toys/fork
[pr]: https://github.com/github-samples/tailspin-toys/compare
[code-of-conduct]: CODE_OF_CONDUCT.md

Tailspin Toys への貢献に関心をお寄せいただきありがとうございます！このクラウドファンディングプラットフォームを、ゲームクリエイターとバッカーの双方にとって最高のものにするために、皆さんの協力が欠かせません。

このプロジェクトへの貢献は、[プロジェクトのオープンソースライセンス](LICENSE)の下で公開に[リリース](https://help.github.com/articles/github-terms-of-service/#6-contributions-under-repository-license)されます。

このプロジェクトは [Contributor Code of Conduct][code-of-conduct] とともにリリースされている点にご注意ください。このプロジェクトに参加することで、その条項に従うことに同意したものとみなされます。

## はじめに

### 前提条件

アプリケーションをローカルで実行・テストする前に、以下をインストールする必要があります:

- **Node.js 22.13+** - [Download](https://nodejs.org/) | [Homebrew](https://formulae.brew.sh/formula/node)
- **Git** - [Download](https://git-scm.com/downloads) | [Homebrew](https://formulae.brew.sh/formula/git)

### 開発環境のセットアップ

1. リポジトリをフォークしてクローンします:
   ```bash
   git clone https://github.com/YOUR-USERNAME/tailspin-toys.git
   cd tailspin-toys
   ```

2. 依存関係をインストールします:
   ```bash
   npm ci
   npx playwright install chromium   # E2E テストにのみ必要
   ```

3. 開発サーバーを起動します:
   ```bash
   npm run dev
   ```

4. ブラウザーで [http://localhost:4321](http://localhost:4321) を開きます

## プロジェクト構成

- `db/` - Drizzle スキーマ、マイグレーション、変換処理、シード、および `games.csv`
- `src/lib/` - データベースクライアントとデータアクセスヘルパー
- `src/` - Astro のページ、レイアウト、コンポーネント、スタイル、型
- `e2e-tests/` - Playwright E2E テスト

## 変更を加える

### データレイヤー（Drizzle + Node SQLite）

- `db/schema.ts` でテーブルを定義します。スキーマ変更後は `npm run db:generate` でマイグレーションを生成します
- すべての関数のパラメーターと戻り値に型ヒントを使用します
- データアクセスヘルパーは注入可能な `db` 引数とともに `src/lib/` に置きます
- データレイヤーの変更には Vitest テストを追加または更新します
- 提出前にテストを実行します: `npm run test:unit`
   - すべてのテストがパスする必要があります

### フロントエンド（Astro）

- UI は `.astro` ページとコンポーネントとして構築し、フロントマターでデータをクエリします（静的出力）
- Tailwind CSS のユーティリティクラスを使ってダークテーマに従います
- テストのために、インタラクティブな要素へ `data-testid` 属性を追加します
- 提出前に E2E テストを実行します: `npm run test:e2e`
   - すべてのテストがパスする必要があります

## Pull Request の提出

### Issue

すべての変更リクエストは Issue から始める必要があります。Issue を PR と併せて作成しても構いませんが、Issue は必ず作成しなければなりません。

### ワークフロー

1. 変更用に `main` から新しいブランチを作成します:
   ```bash
   git checkout -b feature/your-feature-name
   ```

2. ドキュメント化されたコーディング標準に従って変更を加えます。

3. 何も壊れていないことを確認するためにテストスイートを実行します:
   ```bash
   npm run lint
   npm run test:unit
   npm run test:e2e
   ```

4. 明確で分かりやすいメッセージとともに変更をコミットします:
   ```bash
   git commit -m "Add feature: brief description of changes"
   ```

5. 自分のフォークにプッシュし、[pull request を提出][pr]します。

6. pull request がレビューされてマージされるのを待ちます。

### Pull Request のガイドライン

- 適切な pull request テンプレートを使用し、すべてのセクションを記入してください。
- 変更は焦点を絞ってください。関連のない複数の変更がある場合は、別々の pull request として提出してください。
- *何を* および *なぜ* を説明する明確なコミットメッセージを書いてください。
- 変更がアプリケーションの動作に影響する場合は、ドキュメントを更新してください。
- レビューを依頼する前に、すべてのテストがパスすることを確認してください。
- フィードバックに迅速に対応し、調整を行う準備をしておいてください。

## Issue の報告

バグを見つけた、または機能リクエストがありますか？次の内容とともに [Issue を開いて](https://github.com/github-samples/tailspin-toys/issues/new)ください:

- 明確で分かりやすいタイトル
- 再現手順（バグの場合）
- 期待される動作と実際の動作
- 該当する場合はスクリーンショット
- 環境の詳細（OS、ブラウザー、Node のバージョン）

## 参考資料

- [How to Contribute to Open Source](https://opensource.guide/how-to-contribute/)
- [Using Pull Requests](https://help.github.com/articles/about-pull-requests/)
- [Writing Good Commit Messages](http://tbaggery.com/2008/04/19/a-note-about-git-commit-messages.html)
- [GitHub Help](https://help.github.com)
