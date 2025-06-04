---
title: "ghコマンドでGitHubリポジトリを作成し、コミットをPushする方法"
emoji: "🚀"
type: "tech"
topics: ["gh", "GitHub", "git", "CLI", "初心者"]
published: false
---

## ゴール(はじめに)：ghコマンドでGitHubリポジトリ作成＆Push

GitHub公式CLI「gh」を使って、
- 空のリモートリポジトリを作成
- 今いるブランチのコミットをGitHubにPush
する手順を、初心者向けにやさしく解説します。

## やってみた結果
- コマンドだけでGitHubリポジトリを新規作成できるようになった
- 手元のコミットをすぐGitHubにアップできるようになった
- Web画面を開かずにリポジトリ管理ができて便利！

この記事を読めば、誰でも簡単に「gh」コマンドでリモートリポジトリ作成＆Pushができるようになります。

## 開発環境
- OS: Linux (Ubuntuなど)
- gh CLI（GitHub CLI）: インストール済み
- git: インストール済み
- GitHubアカウント

## 事前準備
- ghコマンドのインストール＆ログイン（`gh auth login`）
- gitの初期化＆コミット済みのローカルリポジトリ

## やったこと

### 1. 空のリモートリポジトリを作成

まず、ghコマンドでGitHub上に新しいリポジトリを作成します。

```bash
gh repo create <GitHubユーザー名>/<リポジトリ名> --public --confirm
```

- `<GitHubユーザー名>`：自分のGitHubアカウント名
- `<リポジトリ名>`：作りたいリポジトリ名
- `--public`：公開リポジトリ（非公開にしたい場合は`--private`）
- `--confirm`：対話なしで作成

例：
```bash
gh repo create myname/sample-repo --public --confirm
```

### 2. リモートリポジトリをgit remoteに追加

作成したリポジトリをリモート（origin）として追加します。

```bash
git remote add origin https://github.com/<GitHubユーザー名>/<リポジトリ名>.git
```

例：
```bash
git remote add origin https://github.com/myname/sample-repo.git
```

### 3. 今いるブランチのコミットをPush

現在のブランチ（例: main/master）をリモートにPushします。

```bash
git push -u origin $(git branch --show-current)
```

- `-u`は今後のpush/pullを省略できる設定です

## ghコマンドのインストール
（インストール済みの場合はスキップ可）

```bash
# Ubuntuの場合
sudo apt install gh
```

## ghコマンドの設定
GitHubアカウントと連携するには、

```bash
gh auth login
```

で認証を済ませておきましょう。

## 補足・参考
- [GitHub CLI公式ドキュメント](https://cli.github.com/manual/)
- [gh repo createの詳細](https://cli.github.com/manual/gh_repo_create)

## おわりに

ghコマンドを使えば、Web画面を開かずにリポジトリ作成やPushができてとても便利です。初心者の方もぜひ活用してみてください！
