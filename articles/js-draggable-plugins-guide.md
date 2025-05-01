---
title: "Shopify Draggableのプラグインを使ってドラッグ&ドロップ機能を拡張する"
emoji: "🔌"
type: "tech"
topics: ["javascript", "draggable", "frontend", "web", "ui"]
published: false
---

## ゴール(はじめに)：Shopify Draggableのプラグインを活用してドラッグ&ドロップ機能を強化する

Shopify Draggableライブラリのプラグインシステムを使用することで、ドラッグ&ドロップのユーザー体験を大幅に改善できます。この記事では、よく使用される主要なプラグインの実装方法と、カスタムプラグインの作成方法について解説します。

## やってみた結果

以下のような機能を簡単に実装できるようになります：
- スムーズなアニメーション効果
- 自動スクロール
- 要素のスナップ機能
- ドラッグ中の制約設定
- カスタムアニメーション

この記事を読むことで、より洗練されたドラッグ&ドロップインターフェースを実装できるようになります。

## 開発環境
- 任意のモダンブラウザ（Chrome, Firefox, Safari等）
- テキストエディタ（VS Code推奨）
- Node.js（npmでパッケージをインストールする場合）

フォルダ構造：
```
draggable-plugins-demo/
├── index.html
├── styles.css
├── script.js
└── package.json
```

## 事前準備

基本的なDraggableの使用方法を理解していることが望ましいですが、初めての方でも理解できるように説明します。

## やったこと

### 1. 基本的なプラグインの使い方

まず、Draggableのプラグインシステムの基本的な使い方を説明します。以下のHTMLから始めましょう：

```html
<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Draggable Plugins デモ</title>
  <style>
    .container {
      padding: 20px;
      display: grid;
      grid-template-columns: repeat(3, 1fr);
      gap: 10px;
    }
    
    .draggable-item {
      background: #3498db;
      color: white;
      padding: 20px;
      text-align: center;
      cursor: move;
      user-select: none;
      border-radius: 4px;
    }
    
    .draggable-item.draggable--over {
      background: #2980b9;
    }
    
    .draggable-item.draggable--animation {
      transition: transform 0.2s ease;
    }
  </style>
</head>
<body>
  <div class="container">
    <div class="draggable-item">Item 1</div>
    <div class="draggable-item">Item 2</div>
    <div class="draggable-item">Item 3</div>
    <div class="draggable-item">Item 4</div>
    <div class="draggable-item">Item 5</div>
    <div class="draggable-item">Item 6</div>
  </div>

  <script src="https://cdn.jsdelivr.net/npm/@shopify/draggable@1.0.0-beta.12/lib/draggable.bundle.js"></script>
  <script src="script.js"></script>
</body>
</html>
```

### 2. アニメーションプラグインの実装

script.jsに以下のコードを追加して、アニメーションプラグインを実装します：

```javascript
// アニメーションプラグインの定義
class AnimatePlugin extends Draggable.Plugins.AbstractPlugin {
  static get pluginName() {
    return 'animate';
  }

  getDefaultOptions() {
    return {
      duration: 200,
      easingFunction: 'ease-in-out'
    };
  }

  attach() {
    this.draggable.on('drag:start', this.onDragStart.bind(this));
    this.draggable.on('drag:stop', this.onDragStop.bind(this));
  }

  onDragStart(event) {
    event.source.classList.add('draggable--animation');
  }

  onDragStop(event) {
    setTimeout(() => {
      event.source.classList.remove('draggable--animation');
    }, this.options.duration);
  }
}

// Draggableインスタンスの作成とプラグインの適用
const draggable = new Draggable.Sortable(document.querySelectorAll('.container'), {
  draggable: '.draggable-item',
  plugins: [AnimatePlugin]
});
```

### 3. スナッププラグインの追加

グリッドへのスナップ機能を実装するプラグインを追加します：

```javascript
class SnapPlugin extends Draggable.Plugins.AbstractPlugin {
  static get pluginName() {
    return 'snap';
  }

  getDefaultOptions() {
    return {
      gridSize: 20
    };
  }

  attach() {
    this.draggable.on('drag:move', this.onDragMove.bind(this));
  }

  onDragMove(event) {
    const { gridSize } = this.options;
    const { source } = event;
    
    // 現在の位置を取得
    const rect = source.getBoundingClientRect();
    const x = rect.left;
    const y = rect.top;
    
    // グリッドにスナップ
    const snappedX = Math.round(x / gridSize) * gridSize;
    const snappedY = Math.round(y / gridSize) * gridSize;
    
    // 位置を更新
    source.style.transform = `translate3d(${snappedX}px, ${snappedY}px, 0)`;
  }
}

// プラグインを追加
draggable.addPlugin(SnapPlugin);
```

### 4. 自動スクロールプラグインの実装

ドラッグ中に画面端に近づくと自動スクロールする機能を追加します：

```javascript
class AutoScrollPlugin extends Draggable.Plugins.AbstractPlugin {
  static get pluginName() {
    return 'autoScroll';
  }

  getDefaultOptions() {
    return {
      margin: 50,
      speed: 10
    };
  }

  attach() {
    this.scrollInterval = null;
    this.draggable.on('drag:move', this.onDragMove.bind(this));
    this.draggable.on('drag:stop', this.onDragStop.bind(this));
  }

  onDragMove(event) {
    const { clientY } = event.sensorEvent;
    const { innerHeight } = window;
    const { margin, speed } = this.options;

    if (clientY < margin) {
      // 上にスクロール
      this.startScrolling(-speed);
    } else if (clientY > innerHeight - margin) {
      // 下にスクロール
      this.startScrolling(speed);
    } else {
      this.stopScrolling();
    }
  }

  onDragStop() {
    this.stopScrolling();
  }

  startScrolling(speed) {
    if (this.scrollInterval) return;
    
    this.scrollInterval = setInterval(() => {
      window.scrollBy(0, speed);
    }, 16);
  }

  stopScrolling() {
    if (this.scrollInterval) {
      clearInterval(this.scrollInterval);
      this.scrollInterval = null;
    }
  }
}

// プラグインを追加
draggable.addPlugin(AutoScrollPlugin);
```

## 結論に対しての補足

### プラグイン開発のベストプラクティス

1. **パフォーマンスの考慮**
   - 重い処理は`requestAnimationFrame`を使用
   - イベントリスナーは適切にクリーンアップ

2. **エラーハンドリング**
   - プラグインのメソッド内でtry-catchを使用
   - エラー状態をユーザーに通知

3. **オプションの柔軟性**
   - デフォルト値を提供
   - オプションのバリデーション実装

### 関連リンク
- [Shopify Draggable GitHub](https://github.com/Shopify/draggable)
- [Draggable プラグインAPI](https://github.com/Shopify/draggable/tree/master/src/Plugins)
- [AbstractPlugin クラス](https://github.com/Shopify/draggable/blob/master/src/Plugins/AbstractPlugin/AbstractPlugin.js)

## おわりに

Shopify Draggableのプラグインシステムを活用することで、ドラッグ&ドロップのユーザー体験を大きく向上させることができます。基本的なプラグインから始めて、徐々に複雑な機能を追加していくことで、より洗練されたインターフェースを構築できます。

プラグインの開発には、ユーザビリティとパフォーマンスのバランスを考慮することが重要です。また、他の開発者が使用することを想定して、適切なドキュメンテーションとエラーハンドリングを実装することをお勧めします。

今回紹介したプラグインは基本的な実装例ですが、これらをベースに、プロジェクトの要件に合わせてカスタマイズしていくことで、より高度な機能を実現できます。
