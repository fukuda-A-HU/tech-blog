---
title: "JavaScriptのDraggableライブラリで簡単にドラッグ&ドロップ機能を実装する"
emoji: "🔄"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["javascript", "draggable", "frontend", "web", "ui"]
published: false
---

## ゴール(はじめに)：Shopify Draggableライブラリの基本的な使い方を学ぶ

モダンなWebアプリケーションでは、ドラッグ&ドロップによるユーザーインタラクションが重要な要素となっています。カンバンボードやソート可能なリスト、ドラッグ可能な要素など、多くのUIコンポーネントがドラッグ&ドロップ機能を必要としています。

この記事では、Shopifyが開発した「Draggable」というJavaScriptライブラリを使って、簡単にドラッグ&ドロップ機能を実装する方法を紹介します。

## やってみた結果

Draggableライブラリを使うことで、以下のようなことが簡単に実装できるようになります：
- 要素をドラッグ可能にする
- ドラッグ中の視覚的なフィードバック
- ソート可能なリストの作成
- ドラッグ&ドロップのイベント処理

この記事を読むことで、JavaScriptの知識があれば、誰でも簡単にドラッグ&ドロップ機能を実装できるようになるでしょう。

## 開発環境
- 任意のブラウザ（Chrome, Firefox, Safariなど）
- テキストエディタ（VS Code, Atom, Sublimeなど）
- Node.js（npmでパッケージをインストールする場合）

## 事前準備

基本的なHTMLとCSSの知識、そして初歩的なJavaScriptの知識があれば十分です。

## やったこと

Shopify Draggableライブラリを使って、ドラッグ&ドロップ機能を実装するまでの手順を順を追って説明します。

## Draggableのインストール

Draggableライブラリをプロジェクトに追加するには、いくつかの方法があります。

### 1. npmを使う場合

```bash
npm install @shopify/draggable --save
```

### 2. CDNを使う場合

HTMLファイルに以下のスクリプトタグを追加するだけです。

```html
<script src="https://cdn.jsdelivr.net/npm/@shopify/draggable@1.0.0-beta.12/lib/draggable.bundle.js"></script>
```

## 基本的な使い方

まず、簡単なドラッグ可能な要素を作成してみましょう。

### HTMLの準備

```html
<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Draggable サンプル</title>
  <style>
    .container {
      display: flex;
      flex-wrap: wrap;
      gap: 10px;
      padding: 20px;
    }
    
    .draggable-item {
      width: 100px;
      height: 100px;
      background-color: #3498db;
      color: white;
      display: flex;
      justify-content: center;
      align-items: center;
      cursor: move;
      border-radius: 5px;
      user-select: none;
    }
    
    .draggable-item.draggable-source--is-dragging {
      opacity: 0.5;
    }
  </style>
</head>
<body>
  <h1>Draggable サンプル</h1>
  
  <div class="container">
    <div class="draggable-item">Item 1</div>
    <div class="draggable-item">Item 2</div>
    <div class="draggable-item">Item 3</div>
    <div class="draggable-item">Item 4</div>
  </div>

  <!-- Draggableライブラリの読み込み -->
  <script src="https://cdn.jsdelivr.net/npm/@shopify/draggable@1.0.0-beta.12/lib/draggable.bundle.js"></script>
  
  <script>
    // JavaScriptコードはここに記述
  </script>
</body>
</html>
```

### 基本的なドラッグ機能の実装

上記のHTMLファイルの `<script>` タグ内に以下のコードを記述します。

```javascript
// Draggableインスタンスを作成
const draggable = new Draggable.Draggable(document.querySelectorAll('.container'), {
  draggable: '.draggable-item'
});

// ドラッグ開始時のイベント
draggable.on('drag:start', () => {
  console.log('ドラッグ開始');
});

// ドラッグ中のイベント
draggable.on('drag:move', () => {
  console.log('ドラッグ中');
});

// ドラッグ終了時のイベント
draggable.on('drag:stop', () => {
  console.log('ドラッグ終了');
});
```

これだけで、要素をドラッグできるようになります。ブラウザでHTMLファイルを開き、アイテムをドラッグしてみましょう。

## ソータブル（並べ替え可能な）リストの作成

次に、ドラッグ&ドロップで順序を変更できるリストを作成します。

### HTMLの準備

```html
<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Sortable サンプル</title>
  <style>
    .container {
      width: 300px;
      margin: 0 auto;
      padding: 20px;
      background-color: #f5f5f5;
      border-radius: 5px;
    }
    
    .sortable-list {
      list-style: none;
      padding: 0;
      margin: 0;
    }
    
    .sortable-item {
      padding: 15px;
      background-color: white;
      border: 1px solid #ddd;
      margin-bottom: 10px;
      cursor: move;
      border-radius: 3px;
      user-select: none;
    }
    
    .sortable-item.draggable-mirror {
      opacity: 0.8;
      background-color: #f8f8f8;
    }
    
    .sortable-item.draggable-source--is-dragging {
      opacity: 0.4;
    }
  </style>
</head>
<body>
  <h1>Sortable サンプル</h1>
  
  <div class="container">
    <ul class="sortable-list">
      <li class="sortable-item">タスク 1</li>
      <li class="sortable-item">タスク 2</li>
      <li class="sortable-item">タスク 3</li>
      <li class="sortable-item">タスク 4</li>
      <li class="sortable-item">タスク 5</li>
    </ul>
  </div>

  <!-- Draggableライブラリの読み込み -->
  <script src="https://cdn.jsdelivr.net/npm/@shopify/draggable@1.0.0-beta.12/lib/draggable.bundle.js"></script>
  
  <script>
    // JavaScriptコードはここに記述
  </script>
</body>
</html>
```

### Sortableの実装

上記のHTMLファイルの `<script>` タグ内に以下のコードを記述します。

```javascript
// Sortableインスタンスを作成
const sortable = new Draggable.Sortable(document.querySelectorAll('.sortable-list'), {
  draggable: '.sortable-item'
});

// 並び替え開始時のイベント
sortable.on('sortable:start', (event) => {
  console.log('並び替え開始:', event.data.dragEvent.source.textContent);
});

// 並び替え完了時のイベント
sortable.on('sortable:stop', (event) => {
  console.log('並び替え完了');
});

// 並び替えによる順序変更時のイベント
sortable.on('sortable:sort', (event) => {
  console.log(`${event.data.dragEvent.source.textContent} が移動中...`);
});
```

これで、リスト内の要素をドラッグして順序を変更できるようになります。

## 複数のコンテナ間でのドラッグ&ドロップ

複数のコンテナ間で要素を移動できるようにしてみましょう。

### HTMLの準備

```html
<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>マルチコンテナ Sortable サンプル</title>
  <style>
    .containers {
      display: flex;
      justify-content: space-around;
      padding: 20px;
    }
    
    .container {
      width: 250px;
      min-height: 300px;
      padding: 15px;
      background-color: #f5f5f5;
      border-radius: 5px;
    }
    
    .container-title {
      text-align: center;
      margin-bottom: 15px;
      font-weight: bold;
    }
    
    .sortable-list {
      list-style: none;
      padding: 0;
      margin: 0;
      min-height: 250px;
    }
    
    .sortable-item {
      padding: 15px;
      background-color: white;
      border: 1px solid #ddd;
      margin-bottom: 10px;
      cursor: move;
      border-radius: 3px;
      user-select: none;
    }
    
    .to-do { background-color: #ffcccc; }
    .in-progress { background-color: #ffffcc; }
    .done { background-color: #ccffcc; }
    
    .sortable-item.draggable-mirror {
      opacity: 0.8;
    }
    
    .sortable-item.draggable-source--is-dragging {
      opacity: 0.4;
    }
  </style>
</head>
<body>
  <h1>かんばんボード サンプル</h1>
  
  <div class="containers">
    <div class="container to-do">
      <div class="container-title">未着手</div>
      <ul class="sortable-list">
        <li class="sortable-item">デザインを確認する</li>
        <li class="sortable-item">ドキュメントを書く</li>
        <li class="sortable-item">テストを実行する</li>
      </ul>
    </div>
    
    <div class="container in-progress">
      <div class="container-title">作業中</div>
      <ul class="sortable-list">
        <li class="sortable-item">コードを実装する</li>
        <li class="sortable-item">バグを修正する</li>
      </ul>
    </div>
    
    <div class="container done">
      <div class="container-title">完了</div>
      <ul class="sortable-list">
        <li class="sortable-item">要件を分析する</li>
        <li class="sortable-item">計画を立てる</li>
      </ul>
    </div>
  </div>

  <!-- Draggableライブラリの読み込み -->
  <script src="https://cdn.jsdelivr.net/npm/@shopify/draggable@1.0.0-beta.12/lib/draggable.bundle.js"></script>
  
  <script>
    // JavaScriptコードはここに記述
  </script>
</body>
</html>
```

### マルチコンテナSortableの実装

上記のHTMLファイルの `<script>` タグ内に以下のコードを記述します。

```javascript
// コンテナ間でのSortableインスタンスを作成
const sortable = new Draggable.Sortable(document.querySelectorAll('.sortable-list'), {
  draggable: '.sortable-item',
  containers: document.querySelectorAll('.sortable-list')
});

// 要素が別のコンテナに移動した時のイベント
sortable.on('sortable:sorted', (event) => {
  // 移動元と移動先のコンテナ情報を取得
  const sourceContainer = event.data.oldContainer.closest('.container').querySelector('.container-title').textContent;
  const destinationContainer = event.data.newContainer.closest('.container').querySelector('.container-title').textContent;
  
  console.log(`「${event.data.dragEvent.source.textContent}」を「${sourceContainer}」から「${destinationContainer}」に移動しました`);
});
```

これで、異なるコンテナ間で要素を移動できるカンバンボードが完成します。

## 結論に対しての補足

### Draggableライブラリの他の機能

Draggableライブラリは、この記事で紹介した基本的なドラッグ&ドロップ機能以外にも、以下のような機能を提供しています：

- **Droppable**: 特定の領域に要素をドロップできるようにする
- **Swappable**: 要素間の位置を交換する
- **Collidable**: 他の要素との衝突を検出する
- **Snappable**: 要素を特定の位置にスナップする
- **Plugins**: アニメーションや追加機能をカスタマイズできるプラグイン

### 参考リンク

- [Shopify Draggable GitHub](https://github.com/Shopify/draggable) - 公式リポジトリ
- [Draggable ドキュメント](https://github.com/Shopify/draggable/tree/master/src/Draggable) - 基本的なドラッグ機能のドキュメント
- [Sortable ドキュメント](https://github.com/Shopify/draggable/tree/master/src/Sortable) - ソータブル機能のドキュメント

## おわりに

この記事では、Shopify Draggableライブラリを使って、簡単にドラッグ&ドロップ機能を実装する方法を紹介しました。HTMLとCSSを準備し、少しのJavaScriptコードを書くだけで、ユーザー体験を向上させるインタラクティブなUIを実現できることがおわかりいただけたでしょう。

実際のプロジェクトでは、これらの基本的な実装をベースにして、さらに機能を追加していくことで、より高度なドラッグ&ドロップ機能を実現することができます。例えば、データの永続化、アニメーション効果の追加、モバイルデバイスでのタッチイベントへの対応などが考えられます。

ぜひDraggableライブラリを活用して、魅力的なドラッグ&ドロップ機能を備えたWebアプリケーションを作ってみてください！
