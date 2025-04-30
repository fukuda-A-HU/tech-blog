---
title: "開発効率アップ！GitHub CLIの基本的な使い方"
emoji: "⚡"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["GitHub", "CLI", "Git", "開発ツール"]
published: false
---

# 開発効率アップ！GitHub CLIの基本的な使い方

## はじめに

GitHub CLIとは、GitHubをコマンドラインから操作するための公式ツールです。リポジトリの作成、Issue・PRの操作など、様々なGitHub機能にターミナルからアクセスできるため、開発ワークフローを効率化できます。

この記事では、GitHub CLI（`gh`コマンド）の基本的な使い方を解説します。

## インストール方法

各OSに合わせたインストール方法は以下の通りです。

### macOS

Homebrewを使用する場合:

```bash
brew install gh
```

### Windows

scoop を使用する場合:

```bash
scoop install gh
```

chocolatey を使用する場合:

```bash
choco install gh
```

### Linux (Ubuntu/Debian)

```bash
sudo apt install gh
```

その他のOSやインストール方法は[公式ドキュメント](https://github.com/cli/cli#installation)を参照してください。

## 初期設定（認証）

GitHub CLIを使用するには、まずGitHubアカウントとの認証が必要です。

```bash
gh auth login
```

対話形式で以下の項目を選択します:

1. GitHubへのログイン方法（HTTPS/SSH）
2. 認証方法（ブラウザ/トークン）
3. 認証情報の保存先

ブラウザでの認証を選ぶと、ブラウザが開いてGitHubの認証ページに移動します。認証に成功すると、ターミナルに成功メッセージが表示されます。

## リポジトリ操作

### リポジトリの一覧表示

```bash
gh repo list
```

### リポジトリの作成

新しいリポジトリを作成します。

```bash
gh repo create my-project --public
```

`--private`オプションをつけると非公開リポジトリになります。

### リポジトリのクローン

```bash
gh repo clone owner/repo-name
```

## Issue操作

### Issueの一覧表示

```bash
gh issue list
```

### Issueの詳細表示

```bash
gh issue view 123
```

### Issueの作成

```bash
gh issue create --title "バグの修正" --body "詳細な説明をここに書きます"
```

対話形式でIssueを作成する場合:

```bash
gh issue create
```

### Issueのクローズ

```bash
gh issue close 123
```

## プルリクエスト操作

### プルリクエストの一覧表示

```bash
gh pr list
```

### プルリクエストの作成

カレントブランチからプルリクエストを作成します。

```bash
gh pr create --title "新機能の追加" --body "詳細な説明"
```

対話形式の場合:

```bash
gh pr create
```

### プルリクエストのチェックアウト

```bash
gh pr checkout 123
```

### プルリクエストのマージ

```bash
gh pr merge 123
```

マージ方法を指定する場合:

```bash
gh pr merge 123 --merge  # 通常マージ
gh pr merge 123 --squash  # スカッシュマージ
gh pr merge 123 --rebase  # リベースマージ
```

## その他の便利な機能

### リリース作成

```bash
gh release create v1.0.0 --title "バージョン1.0.0" --notes "リリースノート"
```

### Gistの操作

Gistを作成する:

```bash
gh gist create file.txt
```

### ワークフローの実行

GitHub Actionsのワークフローを手動で実行:

```bash
gh workflow run workflow-name.yml
```

### コマンドのヘルプ表示

各コマンドのヘルプを表示:

```bash
gh help
gh help issue
gh help pr
```

## エイリアスの設定

よく使うコマンドはエイリアスとして設定すると便利です。

```bash
# プルリクエスト一覧の表示を「gh prl」で実行できるようにする
gh alias set prl 'pr list'

# 自分に割り当てられたIssueを表示
gh alias set mi 'issue list --assignee @me'
```

## まとめ

GitHub CLIを使用することで、GitHubの操作をターミナルから効率的に行うことができます。特に以下のメリットがあります：

- ブラウザの切り替えが減少し、開発フローが効率化される
- スクリプトと組み合わせて自動化が可能
- 複雑な操作も簡単なコマンドで実行できる

ぜひGitHub CLIを日々の開発ワークフローに取り入れて、作業効率を向上させてみてください！

## 参考リンク

- [GitHub CLI 公式ドキュメント](https://cli.github.com/manual/)
- [GitHub CLI リポジトリ](https://github.com/cli/cli)