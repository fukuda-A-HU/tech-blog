---
title: "JavaScriptのGSAPで簡単アニメーション作成入門"
emoji: "✨"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["JavaScript", "GSAP", "アニメーション", "Web開発", "フロントエンド"]
published: false
---

# JavaScriptのGSAPでかんたんアニメーション

こんにちは！この記事では、GSAP (GreenSock Animation Platform) の基本的な使い方を紹介します。GSAPは、Webサイトやアプリケーションでプロフェッショナルなアニメーションを簡単に作成できるJavaScriptライブラリです。

## 目次

1. [GSAPとは](#gsapとは)
2. [導入方法](#導入方法)
3. [基本的なアニメーション](#基本的なアニメーション)
4. [イージング（動きの緩急）](#イージング動きの緩急)
5. [タイムライン](#タイムライン)
6. [スクロールトリガー](#スクロールトリガー)
7. [まとめ](#まとめ)

## GSAPとは

GSAP（GreenSock Animation Platform）は、高性能なJavaScriptアニメーションライブラリです。以下のような特徴があります：

- ブラウザの互換性が高い
- 優れたパフォーマンスで滑らかなアニメーションを実現
- 複雑なアニメーションも簡潔なコードで表現できる
- SVG、Canvas、CSSプロパティなど、ほぼすべてのものをアニメーション可能

## 導入方法

GSAPを使うには、いくつかの方法があります：

### CDNを使う方法

```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.12.2/gsap.min.js"></script>
```

### npmを使う方法

```bash
npm install gsap
```

インストール後、プロジェクトにインポートします：

```javascript
import { gsap } from "gsap";
```

## 基本的なアニメーション

GSAPの基本的な使い方は非常にシンプルです。`gsap.to()`、`gsap.from()`、`gsap.fromTo()`などのメソッドを使います。

### gsap.to() - 終了状態を指定

```javascript
// ボックスを右に300px、1秒かけて移動
gsap.to(".box", {
  x: 300,
  duration: 1
});
```

### gsap.from() - 開始状態を指定

```javascript
// ボックスを透明な状態から表示
gsap.from(".box", {
  opacity: 0,
  duration: 1
});
```

### gsap.fromTo() - 開始と終了の両方を指定

```javascript
// 左から右へスライドイン
gsap.fromTo(".box", 
  { x: -100, opacity: 0 },
  { x: 0, opacity: 1, duration: 1 }
);
```

## イージング（動きの緩急）

イージングを使うと、アニメーションの動きに緩急をつけることができます。

```javascript
gsap.to(".box", {
  x: 300,
  duration: 2,
  ease: "power2.inOut" // イージング関数の指定
});
```

GSAPには多くのイージング関数が用意されています：

- `power1`, `power2`, `power3`, `power4` (強さのレベル)
- `.in`, `.out`, `.inOut` (イージングの適用タイミング)
- `elastic`, `bounce`, `rough`, `slow`, `steps`など

## タイムライン

複数のアニメーションを連続または並行して実行するには、タイムラインを使います。

```javascript
// タイムラインの作成
const tl = gsap.timeline();

// 連続したアニメーション
tl.to(".box1", { x: 100, duration: 1 })
  .to(".box2", { y: 50, duration: 0.5 })
  .to(".box3", { rotation: 360, duration: 1.5 });
```

タイムラインに位置パラメータを追加することもできます：

```javascript
tl.to(".box1", { x: 100, duration: 1 })
  .to(".box2", { y: 50, duration: 1 }, "<") // 前のアニメーションと同時に開始
  .to(".box3", { rotation: 360, duration: 1 }, "+=0.5"); // 0.5秒後に開始
```

## スクロールトリガー

ScrollTriggerプラグイン（別途インポートが必要）を使うと、スクロールに連動したアニメーションを作成できます。

```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.12.2/gsap.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.12.2/ScrollTrigger.min.js"></script>
```

```javascript
// スクロールトリガーの登録
gsap.registerPlugin(ScrollTrigger);

// スクロールに連動したアニメーション
gsap.to(".box", {
  x: 500,
  duration: 2,
  scrollTrigger: {
    trigger: ".box",
    start: "top center", // 要素の上端が画面中央に来たとき開始
    end: "bottom center", // 要素の下端が画面中央に来たとき終了
    scrub: true, // スクロールに合わせてアニメーションを進行
    markers: true // 開発用のマーカー表示（デバッグ用）
  }
});
```

## 実用的な例

以下は、要素が画面に入ったときにフェードインするシンプルな実装例です：

```html
<div class="container">
  <div class="fade-in-element">Hello GSAP!</div>
  <div class="fade-in-element">アニメーションの世界へようこそ</div>
  <div class="fade-in-element">簡単にプロ級のエフェクトが作れます</div>
</div>

<style>
  .container {
    margin-top: 500px; /* スクロール用に余白を作成 */
  }
  .fade-in-element {
    padding: 20px;
    margin: 20px;
    background-color: #f0f0f0;
    font-size: 24px;
    opacity: 0; /* 初期状態は非表示 */
  }
</style>

<script>
  gsap.registerPlugin(ScrollTrigger);
  
  // すべてのfade-in-elementに対してアニメーションを適用
  gsap.utils.toArray('.fade-in-element').forEach(element => {
    gsap.from(element, {
      opacity: 0,
      y: 50,
      duration: 1,
      scrollTrigger: {
        trigger: element,
        start: "top bottom-=100", // 要素が画面下から100px入ったとき
        toggleActions: "play none none none"
      }
    });
  });
</script>
```

## まとめ

GSAPは非常に強力で使いやすいアニメーションライブラリです。この記事で紹介した基本的な機能だけでも、多くの場面で活用できるでしょう。

- `gsap.to()`, `gsap.from()`, `gsap.fromTo()`で基本的なアニメーション
- イージング関数で自然な動き
- タイムラインで複雑なアニメーションの制御
- ScrollTriggerでスクロール連動アニメーション

より詳しい情報やチュートリアルは、[GSAP公式サイト](https://greensock.com/gsap/)で確認することができます。

実際に手を動かしてアニメーションを作成してみると、GSAPの可能性を実感できると思います。ぜひ試してみてください！

## 参考リンク

- [GSAP公式ドキュメント](https://greensock.com/docs/)
- [GreenSock チートシート](https://greensock.com/cheatsheet/)
- [GSAPフォーラム](https://greensock.com/forums/)
- [GreenSock - CodePen](https://codepen.io/GreenSock)