---
title: "Git フックの基本的な使い方"
emoji: "🪝"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Git", "GitHooks", "開発ツール", "自動化", "CI"]
published: false
---

## ゴール(はじめに)：Git フックで開発プロセスを自動化する

Gitを使っていると「コミット前に特定のチェックを実行したい」「プッシュ時に自動テストを走らせたい」など、特定のGit操作の前後で何らかの処理を自動実行したいケースがあります。そんな時に役立つのが「Git フック」です。

この記事では、Git フックの基本から実践的な例まで、初心者にもわかりやすく解説します。これを読めば、開発フローを自動化し、ミスを未然に防ぐ術を身につけることができます。

## やってみた結果

この記事を読むことで、次のことが実現できるようになります：

- Git フックの仕組みと基本的な使い方を理解できる
- 一般的なフックを利用して開発プロセスを自動化できる
- コミット前のコード品質チェックやテスト実行が可能になる
- プッシュ前の自動化されたチェックで、不具合を含むコードのリポジトリへの混入を防げる

## 開発環境

- Git（バージョン2.20以上推奨）
- テキストエディタ（VS Code、Vim、任意のエディタ）
- ターミナル（Bash、PowerShellなど）

## 事前準備

本記事は以下のことを前提としています：

1. Gitがインストール済み
2. 基本的なGitコマンド（init、add、commit、push）の知識がある

まだGitをインストールしていない場合は、[Gitの公式サイト](https://git-scm.com/)からダウンロードしてインストールしてください。

## Gitフックとは

Git フックとは、特定のGit操作が実行される前後に自動的に実行されるスクリプトです。例えば：

- コミット前に構文チェックを実行
- プッシュ前に自動テストを実行
- コミットメッセージのフォーマットを強制

これらの自動化によって、チームでのコード品質の維持や、開発プロセスの効率化が可能になります。

## Gitフックの種類

Git フックは大きく分けて「クライアントサイドフック」と「サーバーサイドフック」の2種類があります。

### クライアントサイドフック

開発者のローカル環境で実行されるフックです。主なものとして：

1. **pre-commit**: コミット前に実行（コードのlintなどに使用）
2. **prepare-commit-msg**: コミットメッセージ作成前に実行
3. **commit-msg**: コミットメッセージ検証に使用
4. **post-commit**: コミット後に実行（通知などに使用）
5. **pre-push**: プッシュ前に実行（テストなどに使用）

### サーバーサイドフック

リモートリポジトリのサーバー側で実行されるフックです：

1. **pre-receive**: プッシュを受け取る前に実行
2. **update**: 各ブランチごとに実行
3. **post-receive**: プッシュ処理完了後に実行（デプロイなどに使用）

今回は主にローカル環境で使えるクライアントサイドフックに焦点を当てます。

## Gitフックの場所と基本設定

Git リポジトリ内の `.git/hooks` ディレクトリにフックのサンプルファイルがあります。

```bash
# Gitリポジトリに移動
cd your-repository

# hooksディレクトリの中身を確認
ls -la .git/hooks
```

このディレクトリには `.sample` という拡張子のついたサンプルファイルがあります。これらを有効にするには、`.sample` 拡張子を削除するだけです。

## 最初のGitフックを作成する（pre-commit）

では実際に、コミット時にファイルをチェックする簡単なpre-commitフックを作成してみましょう。

### Step 1: サンプルの拡張子を削除して有効化

```bash
# サンプルファイルをコピーして.sampleを削除
cp .git/hooks/pre-commit.sample .git/hooks/pre-commit

# 実行権限を確認（なければ付与）
chmod +x .git/hooks/pre-commit
```

### Step 2: pre-commitスクリプトを編集

エディタで `.git/hooks/pre-commit` を開き、次のような簡単なスクリプトを追加します：

```bash
#!/bin/sh

# ステージされたファイルの中にTODOコメントがないかチェック
if git diff --cached --name-only | xargs grep -l "TODO" > /dev/null; then
  echo "エラー: コミット対象のファイルにTODOコメントが含まれています"
  echo "以下のファイルを確認してください:"
  git diff --cached --name-only | xargs grep -l "TODO"
  exit 1
fi

# すべてのチェックを通過したのでコミットを許可
exit 0
```

このスクリプトは、コミットしようとしているファイルの中に「TODO」というコメントがあれば、コミットを中止します。

### Step 3: フックのテスト

テスト用のファイルを作成し、TODOコメントを含めてみましょう：

```bash
# テスト用ファイル作成
echo "console.log('Hello');" > test.js
echo "// TODO: Fix this later" >> test.js

# ファイルをステージング
git add test.js

# コミットを試みる
git commit -m "Add test.js"
```

コミットを実行すると、pre-commitフックが動作してエラーメッセージが表示され、コミットが中止されるはずです。

TODOコメントを削除してから再度コミットを試みると、今度は成功するはずです：

```bash
# TODOコメントを削除
sed -i '/TODO/d' test.js

# 再度ステージング
git add test.js

# コミットを試みる
git commit -m "Add test.js"
```

## 実用的なGitフックの例

### コードスタイルチェック（pre-commit）

JavaScriptプロジェクトでESLintを使用したコードスタイルチェック：

```bash
#!/bin/sh

# ステージされたJSファイルを検査
STAGED_FILES=$(git diff --cached --name-only --diff-filter=ACM | grep '\.js$')

if [ "$STAGED_FILES" = "" ]; then
  exit 0
fi

# ESLintでチェック
echo "ESLintでコードをチェック中..."
npx eslint $STAGED_FILES

if [ $? -ne 0 ]; then
  echo "ESLintのチェックに失敗しました。修正してから再度コミットしてください。"
  exit 1
fi

exit 0
```

### コミットメッセージのフォーマット強制（commit-msg）

コミットメッセージが特定のフォーマットに従っているか検証するフック：

```bash
#!/bin/sh

COMMIT_MSG_FILE=$1
COMMIT_MSG=$(cat $COMMIT_MSG_FILE)

# コミットメッセージのフォーマットをチェック
# 例: "feat: メッセージ" または "fix: メッセージ" など
if ! echo "$COMMIT_MSG" | grep -qE "^(feat|fix|docs|style|refactor|test|chore):.+"; then
  echo "エラー: コミットメッセージが正しいフォーマットではありません"
  echo "正しいフォーマット例: feat: 新機能の追加 または fix: バグの修正"
  exit 1
fi

exit 0
```

### プッシュ前のテスト実行（pre-push）

プッシュ前に単体テストを実行するフック：

```bash
#!/bin/sh

# テストを実行
echo "単体テストを実行中..."
npm test

if [ $? -ne 0 ]; then
  echo "テストに失敗しました。テストが成功するまでプッシュできません。"
  exit 1
fi

echo "テスト成功！プッシュを続行します。"
exit 0
```

## プロジェクト全体で共有できるフック設定

通常、`.git/hooks` ディレクトリはバージョン管理されません。チーム全体でフックを共有するには次の方法があります：

### 1. プロジェクトディレクトリにフックを格納

プロジェクトのルートに `git-hooks` などのディレクトリを作成し、そこにフックスクリプトを保存します：

```bash
mkdir -p .githooks
# フックをここに作成
```

そして、Gitの設定でこのディレクトリをフック用ディレクトリとして指定します：

```bash
git config core.hooksPath .githooks
```

これにより、チーム全員がリポジトリをクローンしたときに同じフックを利用できます。

### 2. husky（npmパッケージ）を使用する

Node.jsプロジェクトでは、[husky](https://github.com/typicode/husky)というパッケージを使うと簡単にフックを設定・共有できます：

```bash
# huskyのインストール
npm install husky --save-dev

# huskyの初期化
npx husky install
```

`package.json`に設定を追加：

```json
{
  "scripts": {
    "prepare": "husky install"
  },
  "husky": {
    "hooks": {
      "pre-commit": "npm run lint",
      "pre-push": "npm test"
    }
  }
}
```

これで、npmのインストール時に自動的にフックが設定されます。

## フックをスキップする方法

フックをスキップしたい場合は、`--no-verify` オプションを使用します：

```bash
# pre-commitフックをスキップしてコミット
git commit --no-verify -m "緊急修正"

# pre-pushフックをスキップしてプッシュ
git push --no-verify origin main
```

ただし、チームでの開発では、特別な理由がない限りフックをスキップするのは避けるべきです。

## 結論に対しての補足

### 注意点

- フックはシェルスクリプトなので、環境によって動作が異なる場合があります
- Windowsでは`.sh`ではなく`.bat`や`.ps1`を使うか、Git Bash環境を利用する必要があるかもしれません
- 重いチェックをフックに入れると、Git操作が遅くなる可能性があります

### 関連ツールやサービス

- [husky](https://github.com/typicode/husky) - Gitフックの設定を簡単にするnpmパッケージ
- [lint-staged](https://github.com/okonet/lint-staged) - ステージされたファイルに対してのみlintを実行
- [commitlint](https://github.com/conventional-changelog/commitlint) - コミットメッセージの検証ツール
- [pre-commit](https://pre-commit.com/) - 言語に依存しないGitフックフレームワーク

## おわりに

Git フックを活用することで、単純なミスや品質の低いコードがリポジトリに混入することを防げます。また、チーム全体で統一された開発プロセスを自動化できるというメリットもあります。

初めのうちは簡単なフックから試して、徐々に自分のワークフローに合わせたフックを追加していくのがおすすめです。今回ご紹介した例を参考に、ぜひ自分のプロジェクトでGitフックを活用してみてください。

効果的な自動化によって、より品質の高いコードを効率よく開発できるようになるでしょう！
