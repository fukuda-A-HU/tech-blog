---
title: "JavaScriptのDraggableライブラリでスワップアニメーションを実装する方法"
emoji: "🔄"
type: "tech"
topics: ["javascript", "draggable", "frontend", "web", "ui"]
published: false
---

## ゴール(はじめに)：Shopify DraggableのSwapAnimationプラグインでリッチなUIを実現する

この記事では、Shopify Draggableライブラリの「SwapAnimation」プラグインを使って、要素がドラッグ＆ドロップされる際に美しいスワップアニメーションを実装する方法を紹介します。このプラグインを使うことで、ユーザー体験を大幅に向上させることができます。

## やってみた結果

このプラグインを導入することで、以下のような効果が得られます：
- ドラッグ中の要素が他の要素と入れ替わる際に、スムーズなアニメーションが発生する
- 要素の動きが視覚的に分かりやすくなり、ユーザー体験が向上する
- 最小限のコードで専門的なUI効果を実現できる

この記事を読むことで、数分でプロフェッショナルなドラッグ＆ドロップのアニメーション効果を実装できるようになります。

## 開発環境
- 任意のモダンブラウザ（Chrome、Firefox、Safari等）
- テキストエディタ（VS Codeなど）
- Node.js（npmでパッケージをインストールする場合）

フォルダ構造：
```
draggable-swapanimation-demo/
├── index.html
├── styles.css
└── script.js
```

## 事前準備

基本的なHTML、CSS、JavaScriptの知識があれば十分です。Shopify Draggableの基礎知識があるとより理解しやすいですが、初めての方でも理解できるように説明します。

## やったこと

### 1. 基本的なHTML構造の作成

まず、ドラッグ＆ドロップ可能な要素を含むHTML構造を作成します。index.htmlファイルに以下のコードを記述します：

```html
<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Draggable SwapAnimation デモ</title>
  <link rel="stylesheet" href="styles.css">
</head>
<body>
  <h1>Draggable SwapAnimation デモ</h1>

  <div class="container">
    <div class="item" data-id="1">アイテム 1</div>
    <div class="item" data-id="2">アイテム 2</div>
    <div class="item" data-id="3">アイテム 3</div>
    <div class="item" data-id="4">アイテム 4</div>
    <div class="item" data-id="5">アイテム 5</div>
    <div class="item" data-id="6">アイテム 6</div>
  </div>

  <!-- CDNからShopify Draggableを読み込み -->
  <script src="https://cdn.jsdelivr.net/npm/@shopify/draggable@1.0.0-beta.12/lib/draggable.bundle.js"></script>
  <script src="script.js"></script>
</body>
</html>
```

### 2. CSSスタイルの作成

次に、要素のスタイルを定義します。styles.cssファイルに以下のコードを記述します：

```css
body {
  font-family: 'Helvetica Neue', Arial, sans-serif;
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
}

h1 {
  text-align: center;
  margin-bottom: 30px;
}

.container {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 15px;
}

.item {
  background-color: #3498db;
  color: white;
  padding: 20px;
  border-radius: 5px;
  text-align: center;
  cursor: grab;
  user-select: none;
  transition: transform 0.15s ease;
}

.item:hover {
  background-color: #2980b9;
}

/* ドラッグ中のスタイル */
.item.draggable-source--is-dragging {
  opacity: 0.5;
}

/* スワップ中のアニメーションスタイル */
.item.draggable-swappable--is-swapping {
  transition: transform 300ms cubic-bezier(0.18, 0.67, 0.6, 1.22);
}

/* ドラッグ可能要素がドロップされる場所のスタイル */
.item.draggable--over {
  background-color: #27ae60;
}
```

### 3. JavaScriptでDraggableとSwapAnimationを実装

最後に、Draggableを初期化し、SwapAnimationプラグインを設定します。script.jsファイルに以下のコードを記述します：

```javascript
document.addEventListener('DOMContentLoaded', () => {
  // Sortableインスタンスを作成
  const sortable = new Draggable.Sortable(document.querySelector('.container'), {
    draggable: '.item',    // ドラッグ可能な要素のセレクタ
    mirror: {
      appendTo: 'body',     // ドラッグ中のミラー要素をどこに追加するか
      constrainDimensions: true  // ミラー要素のサイズを保持する
    },
    swapAnimation: {
      duration: 300,                     // アニメーション時間（ミリ秒）
      easingFunction: 'cubic-bezier(.12,.88,.40,.88)', // イージング関数
      horizontal: true                   // 水平方向のスワップを有効化
    },
    plugins: [Draggable.Plugins.SwapAnimation]  // SwapAnimationプラグインを有効化
  });

  // ドラッグ開始イベント
  sortable.on('sortable:start', (event) => {
    console.log('ドラッグ開始:', event.data.dragEvent.source);
  });

  // スワップイベント
  sortable.on('sortable:swap', (event) => {
    console.log('スワップ:', event.data.dragEvent.over);
  });

  // ドラッグ終了イベント
  sortable.on('sortable:stop', (event) => {
    console.log('ドラッグ終了');
    
    // ドラッグ後の並び順を取得
    const items = document.querySelectorAll('.item');
    const order = Array.from(items).map(item => item.getAttribute('data-id'));
    console.log('現在の並び順:', order);
  });
});
```

## SwapAnimationのカスタマイズ

SwapAnimationプラグインには、以下のようなオプションをカスタマイズできます：

```javascript
swapAnimation: {
  duration: 200,                       // アニメーション時間（ミリ秒）
  easingFunction: 'ease-in-out',       // イージング関数（CSS互換の値）
  horizontal: true,                    // 水平方向のスワップを有効化
  vertical: true                       // 垂直方向のスワップを有効化
}
```

### イージング関数の例

異なるイージング関数を試して、最適なアニメーション効果を見つけることができます：

- `ease`: デフォルトの滑らかなアニメーション
- `linear`: 一定速度のアニメーション
- `ease-in`: ゆっくり始まり、速く終わる
- `ease-out`: 速く始まり、ゆっくり終わる
- `cubic-bezier(0.68, -0.55, 0.27, 1.55)`: バウンス効果のあるアニメーション

## 結論に対しての補足

### SwapAnimationプラグインの利点

1. **視覚的フィードバック**
   - ユーザーの操作に対する明確なフィードバックを提供
   - UI要素の変化が視覚的に理解しやすい

2. **カスタマイズ性**
   - アニメーション速度、イージング関数のカスタマイズが可能
   - 方向（水平・垂直）の制御が可能

3. **パフォーマンス**
   - CSSベースのアニメーションでパフォーマンスが高い
   - モバイルデバイスでも滑らかに動作

### 注意点とヒント
- 複雑すぎるアニメーションはユーザー体験を低下させる可能性があります。シンプルで効果的なアニメーションを目指しましょう。
- アニメーション時間は適切な長さに設定しましょう。短すぎると効果が分かりづらく、長すぎるとユーザーを待たせることになります。
- `transform`プロパティを使ったアニメーションはパフォーマンスが高いため、CSSでカスタマイズする場合は`transform`を活用しましょう。

### 関連リンク
- [Shopify Draggable GitHub](https://github.com/Shopify/draggable)
- [SwapAnimation プラグインのドキュメント](https://github.com/Shopify/draggable/tree/main/src/Plugins/SwapAnimation)
- [Draggable デモページ](https://shopify.github.io/draggable/examples/)

## おわりに

Shopify DraggableのSwapAnimationプラグインを使うことで、数行のコードだけで専門的なドラッグ&ドロップアニメーションを実装できることが分かりました。このプラグインは、リスト並び替えやカード操作など、多くのWebアプリケーションで使える汎用的なツールです。

スワップアニメーションは単なる見た目の装飾ではなく、ユーザーが操作の結果を視覚的に理解するための重要な手がかりを提供します。そのため、適切に実装することでアプリケーションの使いやすさと満足度を向上させることができます。

ぜひこの記事を参考に、あなたのプロジェクトにスワップアニメーションを導入してみてください。少しの労力で大きな効果を得ることができるでしょう。
