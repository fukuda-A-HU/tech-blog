---
title: "npmとCDNによるプラグイン追加の方法と違い"
emoji: "📦"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["JavaScript", "npm", "CDN", "フロントエンド", "初心者"]
published: false
---

## ゴール(はじめに)：JavaScriptプロジェクトへのプラグイン追加方法を理解する

Webフロントエンド開発では、さまざまなJavaScriptライブラリやプラグインを利用することがよくあります。それらを追加する方法として主に「npm」と「CDN」の2つの方法がありますが、どちらを選べばよいか迷うことも多いでしょう。

この記事では、npmとCDNによるプラグイン追加の方法の違いや、それぞれのメリット・デメリットについて解説します。プロジェクトの状況に応じて適切な方法を選べるようになることを目指します。

## やってみた結果

この記事を読むことで、次のことがわかるようになります：

- npmとCDNの基本的な違い
- それぞれの方法でプラグインを追加する手順
- プロジェクトの状況に応じた適切な選択方法
- 実際のコード例で見る両者の違い

これにより、あなたのプロジェクトに最適なプラグイン導入方法を選べるようになります。

## 開発環境

- ブラウザ（Chrome、Firefox、Safari など）
- テキストエディタ（Visual Studio Code など）
- Node.js（npmを使用する場合）

フォルダ構造の例：
```
my-project/
├── node_modules/      # npmでインストールしたパッケージ（npm使用時のみ）
├── package.json       # プロジェクト設定ファイル（npm使用時のみ）
├── package-lock.json  # 依存関係ロックファイル（npm使用時のみ）
├── index.html         # HTMLファイル
└── js/
    └── main.js        # JavaScriptファイル
```

## 事前準備

npmを使う場合は、Node.jsがインストールされている必要があります。CDNを使う場合は特別な準備は必要ありません。

Node.jsがインストールされているか確認するには、ターミナルで次のコマンドを実行します：

```bash
node -v
npm -v
```

バージョン番号が表示されれば、インストール済みです。

## やったこと

ここでは、人気のあるJavaScriptライブラリ「Lodash」を例にして、npmとCDNの両方の方法でプロジェクトに追加する方法を説明します。

## npmによるプラグイン追加

npmは「Node Package Manager」の略で、JavaScriptのパッケージ管理ツールです。プロジェクトに必要なライブラリを管理するためのツールとして広く使われています。

### 1. プロジェクトの初期化

まず、プロジェクトディレクトリでnpmを初期化します：

```bash
mkdir my-project
cd my-project
npm init -y
```

これにより、`package.json`ファイルが作成されます。

### 2. プラグインのインストール

次に、必要なライブラリ（ここではLodash）をインストールします：

```bash
npm install lodash
```

このコマンドにより：
- `node_modules`フォルダにLodashがインストールされます
- `package.json`ファイルの`dependencies`にLodashが追加されます
- `package-lock.json`ファイルが生成または更新されます

### 3. JavaScriptからの利用方法

インストールしたライブラリは、次のように利用できます：

```javascript
// ESモジュールの場合
import _ from 'lodash';

// CommonJSの場合
const _ = require('lodash');

// 使用例
const result = _.chunk(['a', 'b', 'c', 'd'], 2);
console.log(result); // [['a', 'b'], ['c', 'd']]
```

## CDNによるプラグイン追加

CDNは「Content Delivery Network」の略で、ライブラリをホスティングするサーバーからファイルを直接取得する方法です。

### 1. HTMLファイルにスクリプトタグを追加

CDNを使用する場合は、HTMLファイルに直接スクリプトタグを追加します：

```html
<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <title>CDN例</title>
</head>
<body>
  <!-- コンテンツ -->
  
  <!-- CDNからLodashを読み込み -->
  <script src="https://cdn.jsdelivr.net/npm/lodash@4.17.21/lodash.min.js"></script>
  
  <!-- 自分のJavaScriptファイル -->
  <script src="js/main.js"></script>
</body>
</html>
```

### 2. JavaScriptからの利用方法

CDNで読み込んだライブラリは、グローバル変数として利用できます：

```javascript
// main.js
// Lodashはグローバルな _ 変数として利用可能
const result = _.chunk(['a', 'b', 'c', 'd'], 2);
console.log(result); // [['a', 'b'], ['c', 'd']]
```

## npmとCDNの違い

### npmのメリット

1. **バージョン管理が簡単**：`package.json`でバージョンを一元管理できる
2. **依存関係の自動解決**：必要なパッケージが自動的にインストールされる
3. **ビルドツールとの統合**：Webpack、Rollupなどのビルドツールと連携しやすい
4. **オフライン開発可能**：一度インストールすれば、ネット接続なしで開発可能
5. **Tree Shaking対応**：必要な機能だけを取り込むことができる

### npmのデメリット

1. **セットアップが必要**：Node.jsのインストールとプロジェクトの初期化が必要
2. **ディスク容量を消費**：`node_modules`が大きくなりがち
3. **複雑さが増す**：小規模プロジェクトでは過剰な場合も

### CDNのメリット

1. **セットアップが簡単**：HTMLに1行追加するだけ
2. **すぐに使える**：ビルドステップが不要
3. **キャッシュの利用**：同じCDNを使用している他のサイトからキャッシュを共有可能
4. **ディスク容量を節約**：ローカルにファイルを保存しない

### CDNのデメリット

1. **インターネット接続が必要**：オフライン開発ができない
2. **バージョン管理が煩雑**：HTMLを手動で更新する必要がある
3. **外部サービスへの依存**：CDNサービスが停止するとサイトに影響が出る
4. **Tree Shakingができない**：ライブラリ全体を読み込む必要がある

## 結論に対しての補足

### どちらを選ぶべきか

- **npmを選ぶべき場合**：
  - 中~大規模のプロジェクト
  - モジュールバンドラー（WebpackなどCDN）を使用する場合
  - 複数のライブラリを利用する場合
  - 本番環境向けの開発

- **CDNを選ぶべき場合**：
  - 小規模プロジェクトやデモ
  - 簡単な設定で素早く始めたい場合
  - ビルドプロセスを避けたい場合
  - プロトタイプ作成時

### 両方を組み合わせる方法

開発時はnpmを使用し、本番環境ではCDNを使うハイブリッドな方法も可能です：

```json
// package.json
{
  "dependencies": {
    "lodash": "^4.17.21"
  },
  "scripts": {
    "build": "webpack --mode production"
  }
}
```

```javascript
// webpack.config.js (一部)
module.exports = {
  // ...
  externals: {
    lodash: '_'
  }
};
```

### 参考リンク

- [npmの公式サイト](https://www.npmjs.com/)
- [CDN比較：jsDelivr](https://www.jsdelivr.com/)
- [CDN比較：unpkg](https://unpkg.com/)
- [CDN比較：cdnjs](https://cdnjs.com/)

## おわりに

npmとCDNはどちらも優れたライブラリ導入方法ですが、プロジェクトの規模や要件によって適切な選択は変わります。

小規模なプロジェクトや素早く始めたい場合はCDN、本格的な開発や大規模プロジェクトではnpmが適しているでしょう。また、開発の段階によって使い分けることも有効な戦略です。

どちらを選ぶにしても、この記事で紹介した両方のアプローチの長所と短所を理解しておくことで、より効率的なWeb開発が可能になります。
