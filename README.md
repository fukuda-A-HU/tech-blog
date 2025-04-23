## TechBlogだよ

最初にnpm installしておくこと

```bash
npm install
```

記事や本を変更してPushしたら、GithubActionsが自動でQiitaやZennにPushしてくれる。

## Qiita のCLIを用いた記事作成フロー

記事の作成
```bash
npx qiita new 記事ファイルのベース名
```

記事のプレビュー
```bash
npx qiita preview
```

## Zenn のCLIを用いた記事作成フロー

記事の場合
```bash
npx zenn new:article
npx zenn preview
npx zenn preview --open
```

本の場合
```bash
npx zenn new:book
npx zenn preview
npx zenn preview --open
```

### Qiita CLI の使い方





### Qiita CLI の使い方
https://github.com/increments/qiita-cli?tab=readme-ov-file

### Qiita CLI をGithubActionsで使う
https://qiita.com/Qiita/items/32c79014509987541130

### Zenn CLI の使い方
https://zenn.dev/zenn/articles/zenn-cli-guide

### Zenn CLI をGithubActionsで使う
https://zenn.dev/zenn/articles/zenn-cli-github-actions
```