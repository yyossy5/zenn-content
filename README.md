# zenn-content

- [アカウントにGitHubリポジトリを連携してZennのコンテンツを管理する - Zenn](https://zenn.dev/zenn/articles/connect-to-github)
- [Zenn CLIをインストールする - Zenn](https://zenn.dev/zenn/articles/install-zenn-cli)
- [ZennのMarkdown記法一覧 - Zenn](https://zenn.dev/zenn/articles/markdown-guide)

## Zenn CLI

### 新しい記事を作成

```bash
npx zenn new:article
```

### 記事のプレビュー

```bash
npx zenn preview
```

を実行後、ブラウザで https://localhost:8000 にアクセス

### 記事の公開

markdown上で`published: true`にしておく(`false`にしておくと下書き状態）。
その後以下を実行

```bash
git add .
git commit
git push origin main
```
