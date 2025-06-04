---
title: "ghqコマンドでGitリポジトリを効率的に管理する方法"
emoji: "📂"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["git", "ghq", "go", "github", "terminal"]
published: false
---

## ゴール(はじめに)：Gitリポジトリの管理を効率化する

この記事では、`ghq`コマンドを使ってGitリポジトリを効率的に管理する方法を解説します。複数のGitリポジトリを扱う開発者にとって、リポジトリの管理は時間がかかる作業になり得ます。`ghq`を使えば、リポジトリのクローン、検索、アクセスが簡単になり、開発ワークフローを大幅に改善できます。

## やってみた結果
- すべてのGitリポジトリが一貫した場所に整理された
- コマンド一つでリポジトリの検索とアクセスが可能になった
- リポジトリのクローン作業が標準化され簡略化された
- 複数のGitホスティングサービス（GitHub、GitLab、Bitbucketなど）のリポジトリを統一的に管理できるようになった
- `fzf`や`peco`と組み合わせることで、インタラクティブなリポジトリ選択が可能になった

この記事を読めば、`ghq`の基本的な使い方からより高度な活用法まで理解でき、あなたのGitリポジトリ管理が劇的に改善するでしょう。

## 開発環境
- macOS/Linux/Windows（WSL）
- Git（インストール済み）
- Go言語（ghqをインストールするため）
- ターミナル（bash/zsh/fish など）

## 事前準備
- Gitがインストール済みであること
- Go言語がインストール済みであること（または他のインストール方法を選択）

## やったこと

### 1. ghqとは何か

`ghq`は、Go言語で書かれたGitリポジトリ管理ツールで、以下のような特徴があります：

- リポジトリをホスト名とパスに基づいて標準化された場所にクローンする
- リモートリポジトリの一覧表示と検索機能
- `ghq get`コマンドで簡単にリポジトリをクローンできる
- `ghq list`でクローン済みのリポジトリを一覧表示できる
- `fzf`や`peco`などのフィルタリングツールと連携可能

`ghq`は元々[motemen](https://github.com/motemen)氏によって開発され、現在も活発にメンテナンスされています。

### 2. ghqのインストール

`ghq`をインストールするにはいくつかの方法があります。

#### Go言語を使ったインストール

```bash
go install github.com/x-motemen/ghq@latest
```

#### macOSの場合（Homebrew）

```bash
brew install ghq
```

#### Windows（Scoop）

```bash
scoop install ghq
```

#### ArchLinux

```bash
sudo pacman -S ghq
```

インストールが完了したら、バージョンを確認してインストールが成功したことを確認しましょう：

```bash
ghq --version
```

### 3. ghqの基本設定

`ghq`はデフォルトで`~/ghq`ディレクトリにリポジトリを保存します。このディレクトリを変更したい場合は、Gitの設定を編集します：

```bash
# デフォルトのルートディレクトリを設定
git config --global ghq.root "~/Projects"

# 複数のルートディレクトリを設定する場合
git config --global --add ghq.root "~/src"
git config --global --add ghq.root "~/go/src"
```

複数のルートディレクトリを設定すると、最初に設定したディレクトリがデフォルトになります。

#### GitHub Enterpriseの設定（必要な場合）

会社などでGitHub Enterpriseを使っている場合は、以下のように設定できます：

```bash
git config --global ghq.github.com.host "github.company.com"
```

### 4. リポジトリのクローン（ghq get）

`ghq get`コマンドを使ってリポジトリをクローンします：

```bash
# GitHubリポジトリをクローン
ghq get github.com/user/repo

# または短縮形
ghq get user/repo

# GitLabリポジトリをクローン
ghq get gitlab.com/user/repo

# 特定のGitリポジトリをクローン
ghq get https://example.com/user/repo.git
```

クローン時にオプションを追加することもできます：

```bash
# 既存のリポジトリを更新（pull）
ghq get -u user/repo

# 浅いクローン（履歴を制限）
ghq get --shallow user/repo

# 特定のブランチをクローン
ghq get -b develop user/repo
```

### 5. クローン済みリポジトリの管理（ghq list）

クローン済みのリポジトリを一覧表示するには：

```bash
# すべてのリポジトリを一覧表示
ghq list

# フルパスで一覧表示
ghq list --full-path

# 特定のキーワードで検索
ghq list golang

# 完全一致で検索
ghq list -e exact-name
```

### 6. リポジトリへの移動

`cd`コマンドと組み合わせて、簡単にリポジトリのディレクトリに移動できます：

```bash
cd $(ghq list --full-path | grep user/repo)
```

ただし、これは少し面倒なので、次のセクションで説明するようなインタラクティブなツールと組み合わせるとより効率的です。

### 7. fzfやpecoとの連携

`ghq`を`fzf`や`peco`などのフィルタリングツールと組み合わせることで、インタラクティブなリポジトリ選択が可能になります。

#### fzfのインストール（必要な場合）

```bash
# macOS
brew install fzf

# その他の環境
git clone --depth 1 https://github.com/junegunn/fzf.git ~/.fzf
~/.fzf/install
```

#### pecoのインストール（必要な場合）

```bash
# macOS
brew install peco

# Go
go install github.com/peco/peco/cmd/peco@latest
```

#### シェル関数の設定

bashの場合は`~/.bashrc`に、zshの場合は`~/.zshrc`に以下のような関数を追加します：

##### fzfを使う場合

```bash
# fzfを使ったリポジトリ選択と移動
function ghq-fzf() {
  local selected_dir=$(ghq list | fzf --preview "ls -la $(ghq root)/{}")
  if [ -n "$selected_dir" ]; then
    cd $(ghq root)/$selected_dir
  fi
}
# エイリアス設定
alias gr='ghq-fzf'
```

##### pecoを使う場合

```bash
# pecoを使ったリポジトリ選択と移動
function ghq-peco() {
  local selected_dir=$(ghq list | peco)
  if [ -n "$selected_dir" ]; then
    cd $(ghq root)/$selected_dir
  fi
}
# エイリアス設定
alias gr='ghq-peco'
```

設定を有効にするには、新しいシェルセッションを開くか、設定ファイルを読み込み直します：

```bash
source ~/.bashrc  # bashの場合
# または
source ~/.zshrc   # zshの場合
```

これで、`gr`コマンドを実行するだけでリポジトリを簡単に選択して移動できるようになります。

### 8. その他の便利な使い方

#### リポジトリのルートフォルダを取得

```bash
ghq root
```

#### リポジトリのURLを取得

```bash
ghq list -p user/repo
```

#### リポジトリを開く（GUIツール）

macOSのFinderでリポジトリを開く例：

```bash
open $(ghq list --full-path | grep user/repo)
```

VSCodeで開く例：

```bash
code $(ghq list --full-path | grep user/repo)
```

### 9. 高度な設定とカスタマイズ

#### プロジェクト固有のホストURL設定

```bash
git config --global "ghq.MY_HOST.root" "/path/to/repos"
```

#### デフォルトのGitHubユーザー設定

自分のリポジトリを簡単にクローンするには：

```bash
git config --global ghq.user "your-github-username"
```

これにより、`ghq get -u mine/repo`のように`mine`を使って自分のリポジトリを簡単に指定できます。

## 補足・参考

### ghqを使うメリット

1. **統一的な管理**: すべてのリポジトリが一定の構造で整理される
2. **高速アクセス**: リポジトリへの移動が簡単になる
3. **標準化**: リポジトリのクローンプロセスが標準化される
4. **キーワード検索**: リポジトリをキーワードで検索できる
5. **他ツールとの連携**: fzf、peco、その他のツールと連携して機能を拡張できる

### 関連リソース

- [ghq公式リポジトリ](https://github.com/x-motemen/ghq)
- [fzfリポジトリ](https://github.com/junegunn/fzf)
- [pecoリポジトリ](https://github.com/peco/peco)
- [Go言語インストールガイド](https://golang.org/doc/install)

### 関連ツール

- **hub / gh**: GitHub CLI（コマンドライン操作を簡略化）
- **lazygit**: ターミナルベースのGit UI
- **tig**: テキストモードのGitインターフェース

### ベストプラクティス

- **一貫したディレクトリ構造を維持する**: ghqのデフォルト構造を尊重し、手動での変更を避ける
- **定期的に更新する**: `ghq get -u`を使って、全リポジトリを一括更新する習慣を持つ
- **エイリアスを設定する**: よく使うコマンドはエイリアスを設定して効率化する
- **自動化スクリプトを作成する**: よく行う操作はスクリプト化しておく

### 全リポジトリのアップデートスクリプト例

```bash
#!/bin/bash
# すべてのリポジトリを更新するスクリプト
for repo in $(ghq list --full-path); do
  echo "Updating $repo..."
  cd $repo && git pull --ff-only
done
```

## おわりに

`ghq`を使うことで、Gitリポジトリの管理が格段に効率化されます。特に多くのプロジェクトに関わっている開発者にとって、この統一的なリポジトリ管理方法は非常に有効です。

最初は少し設定に手間がかかりますが、一度習慣化すれば、リポジトリのクローン、検索、アクセスが驚くほど簡単になります。また、`fzf`や`peco`との連携を設定しておけば、ターミナル上でのリポジトリ操作がさらに快適になります。

自分のワークフローに合わせて`ghq`の使い方をカスタマイズし、開発効率を向上させましょう。リポジトリ管理の悩みから解放され、本来の開発作業に集中できるようになります。
