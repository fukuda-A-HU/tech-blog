---
title: "A-Frameで簡単3Dウェブサイトを作る"
emoji: "🌐"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["aframe", "vr", "webvr", "3d", "webdevelopment"]
published: false
---

# A-Frameで簡単3Dウェブサイトを構築しよう

こんにちは！この記事では、A-Frameを使って簡単な3Dウェブサイトを作成する方法をご紹介します。WebVRフレームワークであるA-Frameを使うと、HTMLだけで3D空間を作れるようになります。

## A-Frameとは？

[A-Frame](https://aframe.io)は、WebVRコンテンツを簡単に作成するためのウェブフレームワークです。HTMLタグを使って3Dオブジェクトを配置できるため、WebGLやThree.jsの知識がなくても3D/VRコンテンツを作ることができます。

## 開発環境の準備

このチュートリアルでは、以下のファイル構成でプロジェクトを作成します：

```
aframe-simple-site/
├── index.html      # メインのHTMLファイル
├── styles.css      # CSSスタイルシート
└── script.js       # JavaScriptファイル
```

では、各ファイルを作成していきましょう。

## 1. HTMLファイルの作成

まずは`index.html`ファイルを作成します。これが私たちの3Dサイトのメインファイルとなります。

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <title>A-Frame 簡単3Dサイト</title>
    <meta name="description" content="A-Frameで作る簡単な3Dサイト">
    <!-- A-Frameライブラリの読み込み -->
    <script src="https://aframe.io/releases/1.3.0/aframe.min.js"></script>
    <link rel="stylesheet" type="text/css" href="styles.css">
    <script src="script.js" defer></script>
  </head>
  <body>
    <a-scene>
      <!-- 環境設定 -->
      <a-sky color="#87CEEB"></a-sky>
      
      <!-- 床 -->
      <a-plane position="0 0 0" 
               rotation="-90 0 0" 
               width="20" 
               height="20" 
               color="#7BC8A4"
               shadow></a-plane>
      
      <!-- オブジェクト -->
      <a-box position="-1 0.5 -3" 
             rotation="0 45 0" 
             color="#4CC3D9" 
             shadow></a-box>
             
      <a-sphere position="0 1.25 -5" 
                radius="1.25" 
                color="#EF2D5E" 
                shadow></a-sphere>
                
      <a-cylinder position="1 0.75 -3" 
                  radius="0.5" 
                  height="1.5" 
                  color="#FFC65D" 
                  shadow></a-cylinder>
                  
      <a-torus position="0 3 -5"
               color="#7BC8A4"
               radius="1"
               radius-tubular="0.1"></a-torus>
      
      <!-- テキスト -->
      <a-text value="A-Frameでの3D世界へようこそ！" 
              position="0 2 -3.5" 
              color="black" 
              width="5" 
              align="center"></a-text>
      
      <!-- カメラ設定 -->
      <a-camera>
        <a-cursor color="#FAFAFA"></a-cursor>
      </a-camera>
    </a-scene>
  </body>
</html>
```

### HTMLファイルの解説

- `<a-scene>`: A-Frameのシーンを定義します。これがVR空間の入れ物となります。
- `<a-sky>`: 背景となる空を設定します。今回は水色にしています。
- `<a-plane>`: 床となる平面オブジェクトです。`rotation="-90 0 0"`で水平な床になります。
- プリミティブオブジェクト：
  - `<a-box>`: 箱型オブジェクト。`position`で位置を、`rotation`で回転を、`color`で色を指定しています。
  - `<a-sphere>`: 球体オブジェクト。`radius`で半径を指定します。
  - `<a-cylinder>`: 円柱オブジェクト。`height`で高さを指定します。
  - `<a-torus>`: ドーナツ型のオブジェクト。
- `<a-text>`: 3D空間内にテキストを表示します。
- `<a-camera>`: ユーザー視点のカメラを設定します。
  - `<a-cursor>`: 画面中央のカーソルです。これでオブジェクトを選択できます。

## 2. CSSファイルの作成

次に`styles.css`ファイルを作成します。A-Frameでは3Dオブジェクトのスタイリングは主にHTMLの属性で行いますが、パフォーマンスチューニングやUI要素のためにCSSを使います。

```css
body {
  margin: 0;
  padding: 0;
  overflow: hidden;
  font-family: 'Arial', sans-serif;
}

/* ページ読み込み中に表示するローダー */
.a-loader-title {
  color: #4CC3D9;
  font-size: 1.5em;
}

/* VR以外の情報表示用のオーバーレイ */
.info-overlay {
  position: fixed;
  top: 20px;
  left: 20px;
  background: rgba(0, 0, 0, 0.5);
  color: white;
  padding: 10px;
  border-radius: 5px;
  z-index: 999;
  pointer-events: none;
}
```

### CSSファイルの解説

- `body`のオーバーフローを隠すことで、スクロールバーが表示されなくなります。
- `.a-loader-title`はA-Frameが読み込み中に表示するタイトルのスタイルです。
- `.info-overlay`は3D空間外に情報を表示するためのスタイルです（必要に応じて使用できます）。

## 3. JavaScriptファイルの作成

最後に`script.js`ファイルを作成して、インタラクティブな要素を追加します。

```javascript
// DOMが読み込まれたら実行
document.addEventListener('DOMContentLoaded', function() {
  // すべての箱型オブジェクトを取得
  const boxes = document.querySelectorAll('a-box');
  
  // 箱をクリックすると回転するイベントを追加
  boxes.forEach(box => {
    box.addEventListener('click', function() {
      // クリックしたら回転アニメーションを開始
      this.setAttribute('animation', {
        property: 'rotation',
        to: '0 360 0',
        dur: 2000,
        easing: 'easeInOutQuad'
      });
    });
    
    // マウスオーバーで色が変わるようにする
    box.addEventListener('mouseenter', function() {
      this.setAttribute('color', '#FF9500');
    });
    
    // マウスが離れたら元の色に戻す
    box.addEventListener('mouseleave', function() {
      this.setAttribute('color', '#4CC3D9');
    });
  });
  
  // 球体をクリックするとサイズが変わるイベントを追加
  const spheres = document.querySelectorAll('a-sphere');
  spheres.forEach(sphere => {
    sphere.addEventListener('click', function() {
      // クリックしたらサイズ変更アニメーションを開始
      this.setAttribute('animation', {
        property: 'radius',
        to: Math.random() * 1.5 + 0.5, // 0.5から2.0の間でランダムなサイズに
        dur: 1000,
        easing: 'easeOutElastic'
      });
    });
  });
});

// 環境情報を表示する関数（必要に応じて使用）
function showEnvironmentInfo() {
  const infoOverlay = document.createElement('div');
  infoOverlay.className = 'info-overlay';
  infoOverlay.innerHTML = `
    <p>A-Frame Version: ${AFRAME.version}</p>
    <p>WebVR対応: ${AFRAME.utils.device.checkHeadsetConnected() ? 'あり' : 'なし'}</p>
  `;
  document.body.appendChild(infoOverlay);
}
```

### JavaScriptファイルの解説

- `DOMContentLoaded`イベントリスナーで、ページ読み込み後にスクリプトを実行します。
- 箱型オブジェクト（`a-box`）に複数のイベントリスナーを追加しています：
  - クリックすると回転アニメーションが実行されます。
  - マウスオーバーで色が変わります。
  - マウスが離れると元の色に戻ります。
- 球体オブジェクト（`a-sphere`）にもイベントリスナーを追加：
  - クリックするとランダムなサイズに変化します。
- 環境情報表示関数は、A-Frameのバージョンやデバイス情報を表示するオプション機能です。

## 動作確認方法

1. 上記の3つのファイルを作成したら、ローカルサーバーを起動してアクセスします。
2. シンプルなローカルサーバーを起動するには、プロジェクトフォルダで以下のコマンドを実行：
   - Python 3の場合: `python -m http.server`
   - NPMがインストール済みの場合: `npx http-server`
3. ブラウザで `http://localhost:8000` (または指定されたポート) にアクセスすると、作成した3Dサイトが表示されます。

## 発展させるには

このシンプルなサイトを発展させる方法はたくさんあります：

- A-Frameのコンポーネントシステムを使って、より複雑な動きを実装する
- 3Dモデル（glTFやOBJ形式）を読み込んで表示する
- 物理エンジンを追加して、オブジェクト間の衝突を実装する
- 音声や動画をテクスチャとして使用する
- VRコントローラーとの連携を実装する

## まとめ

A-Frameを使うと、HTMLタグだけで簡単に3D/VRウェブサイトを作ることができます。今回作成したシンプルなサイトを基盤に、さらに機能を追加して独自の3D空間を作ってみてください。

WebVRはブラウザで動作するため、ユーザーはインストール不要で3D体験ができる点が大きなメリットです。クリエイティブな3Dウェブサイト作りを楽しんでください！
