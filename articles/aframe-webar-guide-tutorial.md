---
title: "A-Frameでシンプルなウェブ拡張現実（WebAR）サイトを構築する"
emoji: "👓"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["aframe", "ar", "webar", "webxr", "javascript"]
published: false
---

# A-Frameでシンプルなウェブ拡張現実（WebAR）サイトを構築する

こんにちは！この記事では、A-Frameとその拡張ライブラリを使って、シンプルなWebARサイトを作成する方法をご紹介します。スマートフォンやタブレットで体験できる拡張現実（AR）コンテンツをHTML、JavaScript、CSSだけで作れるようになります。

## WebARとは？

WebARは、特別なアプリをインストールせずに、ウェブブラウザだけで拡張現実（AR）体験を提供する技術です。カメラを通して現実世界にデジタルコンテンツを重ねて表示することができます。

## A-Frameとは？

[A-Frame](https://aframe.io)は、WebVR/WebXRコンテンツを簡単に作成するためのウェブフレームワークです。HTMLマークアップを使って3Dシーンを構築できるため、3Dグラフィックスの専門知識がなくても、VRやARの体験を作ることができます。

## 開発環境の準備

このチュートリアルでは、以下のファイル構成でプロジェクトを作成します：

```
aframe-webar-site/
├── index.html    # メインのHTMLファイル 
├── styles.css    # CSSスタイルシート
└── script.js     # JavaScriptファイル
```

それでは、各ファイルを作成していきましょう。

## 1. HTMLファイルの作成

まずは`index.html`ファイルを作成します。これが私たちのWebARサイトのメインファイルとなります。

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <title>A-Frame WebAR デモ</title>
    <meta name="description" content="A-Frameで作るシンプルなWebARサイト">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <!-- A-Frameライブラリの読み込み -->
    <script src="https://aframe.io/releases/1.3.0/aframe.min.js"></script>
    <!-- AR.jsライブラリの読み込み -->
    <script src="https://raw.githack.com/AR-js-org/AR.js/master/aframe/build/aframe-ar.js"></script>
    <link rel="stylesheet" type="text/css" href="styles.css">
    <script src="script.js" defer></script>
  </head>
  <body>
    <!-- arjs属性：ARシーンの設定 -->
    <a-scene embedded arjs="sourceType: webcam; debugUIEnabled: false; detectionMode: mono_and_matrix;">
      <!-- 3Dオブジェクト -->
      <!-- マーカーが認識されたときに表示される要素 -->
      <a-marker preset="hiro">
        <!-- 箱 -->
        <a-box position="0 0.5 0" 
               material="color: #4CC3D9;" 
               scale="1 1 1" 
               animation="property: rotation; to: 0 360 0; loop: true; dur: 5000; easing: linear"></a-box>
        <!-- テキスト -->
        <a-text value="Hello WebAR!" 
                position="0 1.5 0" 
                align="center" 
                color="#FFFFFF" 
                scale="1.5 1.5 1.5"
                rotation="-90 0 0"></a-text>
      </a-marker>
      
      <!-- カメラ -->
      <a-entity camera></a-entity>
    </a-scene>
    
    <!-- UIオーバーレイ -->
    <div class="ui-container">
      <div class="intro-panel">
        <h2>WebAR デモ</h2>
        <p>「HIRO」マーカーにカメラを向けてください</p>
        <button id="start-btn">開始</button>
      </div>
    </div>
  </body>
</html>
```

### HTMLファイルの解説

- `<meta name="viewport">`タグでモバイルでのレンダリングを最適化します。
- A-Frameライブラリに加えて、AR.jsライブラリも読み込んでいます。AR.jsはA-Frameの拡張で、WebARの機能を提供します。
- `<a-scene>`タグに`embedded`と`arjs`属性を追加して、AR機能を有効化しています：
  - `sourceType: webcam`: カメラ映像を使用するように設定
  - `debugUIEnabled: false`: デバッグUIを無効化
  - `detectionMode: mono_and_matrix`: マーカー検出モードを設定
- `<a-marker preset="hiro">`タグで、「HIRO」という標準マーカーを使用するように指定しています。このマーカーはAR.jsが認識できる標準マーカーです。
- マーカー内には、3Dの箱（`<a-box>`）とテキスト（`<a-text>`）を配置しています。
- 箱にはアニメーション属性を追加して、常に回転するようにしています。
- UIオーバーレイは、通常のHTMLとCSSで実装しています。

## 2. CSSファイルの作成

次に`styles.css`ファイルを作成します。A-FrameとARのシーンに加えて、UIのスタイルを設定します。

```css
body {
  margin: 0;
  padding: 0;
  overflow: hidden;
  font-family: 'Arial', sans-serif;
}

.a-enter-vr {
  display: none; /* VRモードボタンを非表示 */
}

/* UIオーバーレイのスタイル */
.ui-container {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  pointer-events: none; /* カメラの操作を妨げないように */
  z-index: 999;
}

.intro-panel {
  position: absolute;
  bottom: 20px;
  left: 50%;
  transform: translateX(-50%);
  background-color: rgba(0, 0, 0, 0.7);
  color: white;
  padding: 15px;
  border-radius: 10px;
  text-align: center;
  max-width: 80%;
  pointer-events: auto;
}

button {
  background-color: #4CC3D9;
  border: none;
  color: white;
  padding: 10px 20px;
  border-radius: 5px;
  font-size: 16px;
  cursor: pointer;
  margin-top: 10px;
}

button:hover {
  background-color: #3AA8BE;
}

.hidden {
  display: none;
}
```

### CSSファイルの解説

- `body`スタイルで、余白をなくし、オーバーフローを隠します。
- `.a-enter-vr`クラスを非表示にして、A-Frameのデフォルトのボタンを隠しています。
- `.ui-container`は画面全体を覆いますが、`pointer-events: none`で背後のAR体験の操作を妨げないようにしています。
- `.intro-panel`は画面下部に表示される説明パネルです。
- `button`のスタイルで、アプリケーションの開始ボタンのデザインを指定しています。
- `.hidden`クラスは、要素を非表示にするために使用します。

## 3. JavaScriptファイルの作成

最後に`script.js`ファイルを作成して、インタラクティブな要素とイベント処理を追加します。

```javascript
document.addEventListener('DOMContentLoaded', function() {
  // 要素の取得
  const startBtn = document.getElementById('start-btn');
  const introPanel = document.querySelector('.intro-panel');
  
  // 'HIRO'マーカーの画像URLとダウンロードリンクの表示
  introPanel.innerHTML += `
    <div class="marker-info">
      <p>下記のマーカーを印刷するか、別の画面に表示してください：</p>
      <img src="https://raw.githubusercontent.com/AR-js-org/AR.js/master/data/images/hiro.png" 
           alt="HIRO Marker" width="150">
      <p><a href="https://raw.githubusercontent.com/AR-js-org/AR.js/master/data/images/hiro.png" 
            download="hiro-marker.png" 
            style="color: #4CC3D9; text-decoration: none;">
            マーカーをダウンロード</a></p>
    </div>
  `;
  
  // マーカー検出イベントの処理
  const marker = document.querySelector('a-marker');
  let markerVisible = false;
  
  marker.addEventListener('markerFound', function() {
    markerVisible = true;
    if (introPanel.classList.contains('hidden')) {
      showMessage('マーカーが検出されました！');
    }
  });
  
  marker.addEventListener('markerLost', function() {
    markerVisible = false;
    if (introPanel.classList.contains('hidden')) {
      showMessage('マーカーが見えなくなりました。再度カメラをマーカーに向けてください。');
    }
  });
  
  // スタートボタンのイベント処理
  startBtn.addEventListener('click', function() {
    introPanel.classList.add('hidden');
    if (!markerVisible) {
      showMessage('HIROマーカーにカメラを向けてください');
    }
  });
  
  // メッセージを表示する関数
  function showMessage(text, duration = 3000) {
    // 既存のメッセージがあれば削除
    const existingMsg = document.querySelector('.message');
    if (existingMsg) {
      existingMsg.remove();
    }
    
    // 新しいメッセージ要素を作成
    const msg = document.createElement('div');
    msg.className = 'message';
    msg.textContent = text;
    msg.style.cssText = `
      position: fixed;
      top: 20px;
      left: 50%;
      transform: translateX(-50%);
      background-color: rgba(0, 0, 0, 0.7);
      color: white;
      padding: 10px 20px;
      border-radius: 20px;
      z-index: 1000;
      text-align: center;
    `;
    
    // ドキュメントに追加
    document.body.appendChild(msg);
    
    // 指定時間後にメッセージを非表示
    setTimeout(() => {
      msg.remove();
    }, duration);
  }
  
  // 動的にオブジェクトを追加する例（必要に応じて使用）
  function addDynamicObject() {
    // 例：マーカーに新しい球体オブジェクトを追加
    const sphere = document.createElement('a-sphere');
    sphere.setAttribute('position', '0 1 0');
    sphere.setAttribute('radius', '0.5');
    sphere.setAttribute('color', '#EF2D5E');
    
    marker.appendChild(sphere);
  }
  
  // デバイスの状態確認
  function checkDeviceCapabilities() {
    // WebXRがサポートされているか確認
    if ('xr' in navigator) {
      navigator.xr.isSessionSupported('immersive-ar')
        .then((supported) => {
          if (!supported) {
            showMessage('このデバイスはイマーシブARをサポートしていませんが、ブラウザベースのARは機能する可能性があります。');
          }
        });
    } else {
      showMessage('WebXRはこのブラウザではサポートされていません。Chrome/Safariの最新版をお試しください。');
    }
  }
  
  // デバイスの機能確認を実行
  checkDeviceCapabilities();
});
```

### JavaScriptファイルの解説

- `DOMContentLoaded`イベントリスナーで、ページ読み込み後に処理を実行します。
- マーカー画像の表示とダウンロードリンクをUIに追加しています。
- マーカー検出イベント（`markerFound`と`markerLost`）をリスニングして、ユーザーに適切なフィードバックを提供します。
- スタートボタンのクリックイベントを処理して、イントロパネルを非表示にします。
- `showMessage`関数で一時的なメッセージを表示します。
- `addDynamicObject`関数は、JavaScriptを使って動的に3Dオブジェクトを追加する例です。
- `checkDeviceCapabilities`関数で、デバイスのAR機能をチェックして、適切なメッセージを表示します。

## 動作確認方法

1. 上記の3つのファイルを作成したら、ローカルサーバーを起動してアクセスします。
   - Node.jsがインストールされている場合: `npx http-server`
   - Pythonがインストールされている場合: `python -m http.server`

2. スマートフォンやタブレットで、ローカルサーバーのURLにアクセスします（例：`http://192.168.1.xxx:8080`）。
   - 注意：WebARはHTTPS環境か、ローカルホストでないと動作しない場合があります。本番環境ではHTTPSの利用を推奨します。

3. カメラへのアクセス許可を求められたら「許可」をタップします。

4. 画面上に表示される「HIROマーカー」を印刷するか、別の画面に表示します。

5. スマートフォンのカメラをマーカーに向けると、3Dオブジェクトが表示されます。

## 発展させるには

このシンプルなWebARサイトを発展させる方法はさまざまあります：

- 複数のマーカーを使用して、異なるコンテンツを表示する
- マーカーレスARを実装して、平面検出を利用する
- 3Dモデル（glTFやOBJ形式）を読み込んで表示する
- インタラクティブな要素を追加して、タップやジェスチャーでオブジェクトを操作できるようにする
- 位置情報と組み合わせて、特定の場所でARコンテンツを表示する

## 注意点

- WebARは新しい技術であり、すべてのデバイスやブラウザで同じように動作するわけではありません。
- 最新のiOSではSafari、AndroidではChromeが最も互換性があります。
- カメラ許可やHTTPSの要件など、セキュリティ上の制約があります。
- 屋外での使用時は、光の条件によってマーカー認識の精度が変わることがあります。

## まとめ

A-FrameとAR.jsを組み合わせることで、簡単にWebARサイトを作成できます。今回のチュートリアルでは、HTMLマークアップとJavaScriptを使った基本的なARエクスペリエンスの作り方を学びました。この基礎をもとに、さらに複雑で魅力的なARコンテンツを開発してみてください。

拡張現実技術は、商品のバーチャル試着、教育、ゲーム、観光案内など、様々な分野で活用できます。WebARの大きな利点は、特別なアプリをインストールする必要がなく、ウェブブラウザだけで体験できることです。

ぜひ、この入門ガイドを参考に、あなただけのWebAR体験を創造してみてください！
