---
title: "CLI から .gitignore ファイルを簡単に生成する方法"
emoji: "🧹"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["git", "gitignore", "cli", "開発ツール"]
published: false
---

## ゴール(はじめに)：CLI から .gitignore ファイルを効率よく生成する

この記事では、コマンドラインから簡単に .gitignore ファイルを生成する方法を紹介します。プロジェクト開始時に毎回 .gitignore の内容を考えるのは手間がかかりますが、gitignore.io という便利なサービスを CLI から活用することで、この作業を効率化できます。

## やってみた結果

- コマンドライン一発で、プロジェクトに最適な .gitignore ファイルを生成できる
- 複数の開発環境や言語を組み合わせた .gitignore も簡単に作成できる
- エイリアスを設定することで、さらに作業効率が向上する
- 既存の .gitignore ファイルへの追加も可能

## 開発環境
- macOS / Linux / Windows (WSL)
- bash / zsh などのシェル環境
- curl（ほとんどのシステムに標準でインストール済み）

## 事前準備

特別なインストールは不要です。ほとんどのシステムには標準で `curl` コマンドがインストールされていますが、念のため確認しましょう。

```bash
curl --version
```

バージョン情報が表示されれば OK です。表示されない場合は、OSに合わせて curl をインストールしてください。

## やったこと

### 基本的な使い方

gitignore.io は Web API として提供されているので、curl コマンドで簡単にアクセスできます。基本的な使い方は次のとおりです：

```bash
curl -sL https://gitignore.io/api/プラットフォーム名,言語名
```

例えば、Node.js プロジェクトの .gitignore を生成するには：

```bash
curl -sL https://gitignore.io/api/node > .gitignore
```

これだけで、Node.js プロジェクトに最適な .gitignore ファイルが生成されます。

### 複数の環境を組み合わせる

複数の開発環境や言語を組み合わせることもできます。カンマ区切りでプラットフォームや言語を指定します：

```bash
curl -sL https://gitignore.io/api/node,visualstudiocode,macos > .gitignore
```

これで Node.js、VS Code、macOS に関する .gitignore 設定がまとめて生成されます。

### 利用可能なテンプレート一覧を取得する

どのようなプラットフォームや言語に対応しているか確認したい場合は：

```bash
curl -sL https://gitignore.io/api/list
```

このコマンドを実行すると、利用可能なすべてのテンプレート名が表示されます。

### シェルエイリアスの設定

毎回長いコマンドを入力するのは面倒なので、シェルのエイリアスを設定すると便利です。

#### Bash/Zsh の場合

`~/.bashrc` または `~/.zshrc` に以下の行を追加します：

```bash
# .gitignore を生成するエイリアス
gitignore() {
  curl -sL "https://gitignore.io/api/$*" > .gitignore
}
```

ファイルを保存したら、次のコマンドでシェル設定を再読み込みします：

```bash
# Bash の場合
source ~/.bashrc

# Zsh の場合
source ~/.zshrc
```

これで次のように簡単に使えるようになります：

```bash
gitignore node,visualstudiocode,macos
```

#### Windows PowerShell の場合

PowerShell プロファイルに関数を追加します：

```powershell
function New-GitIgnore {
  param(
    [Parameter(Mandatory=$true)]
    [string]$Template
  )
  $response = Invoke-WebRequest -Uri "https://gitignore.io/api/$Template" -UseBasicParsing
  $response.Content | Out-File -FilePath .gitignore -Encoding utf8
}

Set-Alias -Name gitignore -Value New-GitIgnore
```

### 既存の .gitignore に追加する

既に .gitignore ファイルがある場合に、新しい設定を追加するには：

```bash
curl -sL https://gitignore.io/api/python >> .gitignore
```

`>` の代わりに `>>` を使うことで、既存のファイルに追記できます。

### よく使われる組み合わせの例

#### Web フロントエンド開発

```bash
curl -sL https://gitignore.io/api/node,visualstudiocode,macos,windows > .gitignore
```

#### Python 開発

```bash
curl -sL https://gitignore.io/api/python,visualstudiocode,pycharm,macos,linux > .gitignore
```

#### Java 開発

```bash
curl -sL https://gitignore.io/api/java,maven,gradle,eclipse,intellij > .gitignore
```

## 結論に対しての補足
- [gitignore.io の Web サイト](https://gitignore.io/) - GUI から .gitignore を生成したい場合
- [gitignore.io の GitHub リポジトリ](https://github.com/toptal/gitignore.io) - プロジェクトのソースコード
- [GitHub の gitignore テンプレート集](https://github.com/github/gitignore) - 別の gitignore リソース

## おわりにorまとめ

gitignore.io の CLI 活用方法を紹介しました。コマンド一つで最適な .gitignore ファイルを生成できるため、新しいプロジェクトの立ち上げ時間を短縮できます。何度も使うことになるコマンドなので、エイリアスを設定してさらに効率的に作業することをおすすめします。

開発環境ごとに適切な .gitignore を用意することで、不要なファイルのコミットを防ぎ、リポジトリをクリーンに保つことができます。ぜひ日常の開発ワークフローに取り入れてみてください！
