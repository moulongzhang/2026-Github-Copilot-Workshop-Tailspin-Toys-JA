---
description: 'Playwright テスト生成の指示'
applyTo: '**/*.spec.ts'
---

# テスト作成のガイドライン

## コード品質の基準

- **ロケーター**: 堅牢性とアクセシビリティのために、ユーザー向けのロールベースのロケーター（`getByRole`、`getByLabel`、`getByText` など）を優先してください。`test.step()` を使って操作をグループ化し、テストの可読性とレポートを向上させてください。
- **タイムアウト**: Playwright 組み込みの自動待機メカニズムのみに頼ってください。`waitForTimeout`、デフォルトタイムアウトの延長、`waitForLoadState` などのハードコードされた待機は決して使わないでください。
- **アサーション**: 自動リトライされる web-first のアサーションを使用してください。これらのアサーションは `await` キーワードで始まります（例: `await expect(locator).toHaveText()`）。実際にコンテンツや構造を気にする場合は、単なる `toBeVisible()` よりも、意味のある状態を検証するアサーション — `toHaveText`、`toContainText`、`toHaveCount`、`toMatchAriaSnapshot`、`toHaveURL` — を優先してください。`toBeVisible()` は有効な自動リトライアサーションであり、本当の存在/可視性のチェックには適切です。ただ、より具体的なアサーションのほうが意図をよく表す場合には使わないでください。
- **明確さ**: 意図を明確に示す説明的なテストとステップのタイトルを使用してください。コメントは複雑なロジックや自明でない操作を説明する場合にのみ追加してください。

## テストの構造

- **インポート**: `import { test, expect } from '@playwright/test';` で始めてください。
- **構成**: ある機能に関連するテストは `test.describe()` ブロックの下にグループ化してください。
- **フック**: `describe` ブロック内のすべてのテストに共通するセットアップ操作（例: ページへの遷移）には `beforeEach` を使用してください。
- **タイトル**: `Feature - Specific action or scenario` のような明確な命名規則に従ってください。


## ファイルの構成

- **場所**: すべてのテストファイルを `e2e-tests/` ディレクトリに保存してください。
- **命名**: `<feature-or-page>.spec.ts` の規約を使用してください（例: `login.spec.ts`、`search.spec.ts`）。
- **スコープ**: 主要なアプリケーション機能またはページごとに 1 つのテストファイルを目指してください。

## アサーションのベストプラクティス

- **UI 構造**: コンポーネントのアクセシビリティツリー構造を検証するには `toMatchAriaSnapshot` を使用してください。これは包括的でアクセシブルなスナップショットを提供します。
- **要素数**: ロケーターで見つかった要素の数をアサートするには `toHaveCount` を使用してください。
- **テキストコンテンツ**: 完全一致には `toHaveText` を、部分一致には `toContainText` を使用してください。
- **ナビゲーション**: 操作後のページ URL を検証するには `toHaveURL` を使用してください。


## テスト構造の例

```typescript
import { test, expect } from '@playwright/test';

test.describe('Movie Search Feature', () => {
  test.beforeEach(async ({ page }) => {
    // 各テストの前にアプリケーションへ遷移する
    await page.goto('https://debs-obrien.github.io/playwright-movies-app');
  });

  test('Search for a movie by title', async ({ page }) => {
    await test.step('Activate and perform search', async () => {
      await page.getByRole('search').click();
      const searchInput = page.getByRole('textbox', { name: 'Search Input' });
      await searchInput.fill('Garfield');
      await searchInput.press('Enter');
    });

    await test.step('Verify search results', async () => {
      // 検索結果のアクセシビリティツリーを検証する
      await expect(page.getByRole('main')).toMatchAriaSnapshot(`
        - main:
          - heading "Garfield" [level=1]
          - heading "search results" [level=2]
          - list "movies":
            - listitem "movie":
              - link "poster of The Garfield Movie The Garfield Movie rating":
                - /url: /playwright-movies-app/movie?id=tt5779228&page=1
                - img "poster of The Garfield Movie"
                - heading "The Garfield Movie" [level=2]
      `);
    });
  });
});
```

## 作成とイテレーションの戦略

> [!NOTE]
> このファイルはスペックの書き方をカバーしています。E2E スイートを*実行する*には、`quality-checks` skill を使用し、`npx playwright test` を直接呼び出さないでください。

1. **実行**: `quality-checks` skill を通じてスイートを実行します。
2. **失敗のデバッグ**: テストの失敗を分析し、根本原因を特定します。
3. **イテレーション**: 必要に応じてロケーター、アサーション、テストロジックを改善し、skill を通じて再実行します。
4. **検証**: テストが一貫して合格し、意図した機能をカバーしていることを確認します。
5. **報告**: テスト結果と発見された問題についてフィードバックを提供します。

## 品質チェックリスト

テストを確定する前に、以下を確認してください:
- [ ] すべてのロケーターがアクセシブルで具体的であり、strict モード違反を使用していない
- [ ] テストが論理的にグループ化され、明確な構造に従っている
- [ ] アサーションが意味のあるもので、ユーザーの期待を反映している
- [ ] テストが一貫した命名規則に従っている
- [ ] コードが適切にフォーマットされ、コメントが付けられている
