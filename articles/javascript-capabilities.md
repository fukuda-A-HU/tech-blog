---
title: "JavaScriptでできること総まとめ - 初心者からプロまで"
emoji: "🚀"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["JavaScript", "フロントエンド", "Web開発", "プログラミング"]
published: false
---

# JavaScriptでできること総まとめ

こんにちは！この記事では、JavaScriptを使って実現できることを幅広く紹介します。初心者の方からプロの開発者まで、JavaScriptの可能性を再確認できる内容になっています。

## 目次

1. [基本的なDOM操作](#基本的なdom操作)
2. [イベント処理](#イベント処理)
3. [データ操作と計算](#データ操作と計算)
4. [非同期処理](#非同期処理)
5. [Web API の活用](#web-api-の活用)
6. [グラフィックスとアニメーション](#グラフィックスとアニメーション)
7. [サーバーサイドJavaScript (Node.js)](#サーバーサイドjavascript-nodejs)
8. [モバイル・デスクトップアプリ開発](#モバイルデスクトップアプリ開発)
9. [最新のJavaScript機能](#最新のjavascript機能)
10. [まとめ](#まとめ)

## 基本的なDOM操作

JavaScriptの最も基本的な用途は、HTML要素を操作することです。

```javascript
// 要素の取得
const element = document.getElementById('myElement');
const elements = document.querySelectorAll('.myClass');

// コンテンツの変更
element.innerHTML = '<strong>新しいコンテンツ</strong>';
element.textContent = 'プレーンテキスト';

// スタイルの変更
element.style.color = 'blue';
element.style.fontSize = '20px';

// クラスの追加・削除
element.classList.add('highlight');
element.classList.remove('hidden');
element.classList.toggle('active');

// 要素の作成と追加
const newElement = document.createElement('div');
newElement.textContent = '新しい要素';
document.body.appendChild(newElement);
```

## イベント処理

ユーザーの操作に反応するために、イベントリスナーを使用します。

```javascript
// クリックイベント
document.getElementById('myButton').addEventListener('click', function(event) {
  console.log('ボタンがクリックされました');
  event.preventDefault(); // デフォルトの動作を防止
});

// フォーム送信
document.getElementById('myForm').addEventListener('submit', function(event) {
  event.preventDefault();
  const formData = new FormData(this);
  console.log(formData.get('username'));
});

// キーボードイベント
document.addEventListener('keydown', function(event) {
  console.log(`キーが押されました: ${event.key}`);
});

// マウスイベント
element.addEventListener('mouseover', function() {
  this.style.backgroundColor = 'yellow';
});

element.addEventListener('mouseout', function() {
  this.style.backgroundColor = '';
});
```

## データ操作と計算

JavaScriptはデータ操作にも優れています。

```javascript
// 配列操作
const fruits = ['リンゴ', 'バナナ', 'オレンジ'];
fruits.push('イチゴ');
fruits.map(fruit => fruit.length);
fruits.filter(fruit => fruit.length > 3);
fruits.reduce((acc, fruit) => acc + fruit.length, 0);

// オブジェクト操作
const person = {
  name: '山田太郎',
  age: 30,
  hobbies: ['読書', '旅行']
};

const { name, age } = person; // 分割代入
const newPerson = { ...person, job: 'エンジニア' }; // スプレッド構文

// 日付操作
const now = new Date();
const year = now.getFullYear();
const formattedDate = now.toLocaleDateString('ja-JP');

// 数値操作・計算
const roundedNumber = Math.round(3.7);
const randomNumber = Math.random() * 100;
```

## 非同期処理

JavaScriptは非同期処理を簡単に扱えます。

```javascript
// Promise
function fetchData() {
  return new Promise((resolve, reject) => {
    setTimeout(() => resolve('データが取得できました'), 1000);
  });
}

fetchData()
  .then(data => console.log(data))
  .catch(error => console.error(error));

// async/await
async function getData() {
  try {
    const result = await fetchData();
    console.log(result);
  } catch (error) {
    console.error(error);
  }
}

// Fetch API
fetch('https://api.example.com/data')
  .then(response => response.json())
  .then(data => console.log(data))
  .catch(error => console.error('エラーが発生しました:', error));
```

## Web API の活用

ブラウザが提供する様々なWeb APIを利用できます。

```javascript
// ローカルストレージ
localStorage.setItem('username', '山田太郎');
const username = localStorage.getItem('username');

// Geolocation API
navigator.geolocation.getCurrentPosition(
  position => console.log(`緯度: ${position.coords.latitude}, 経度: ${position.coords.longitude}`),
  error => console.error('位置情報を取得できませんでした')
);

// Web Notifications
if ('Notification' in window) {
  Notification.requestPermission().then(permission => {
    if (permission === 'granted') {
      new Notification('新着メッセージ', {
        body: '新しいメッセージが届きました',
        icon: 'message-icon.png'
      });
    }
  });
}

// IndexedDB（クライアントサイドデータベース）
const request = indexedDB.open('myDatabase', 1);
request.onupgradeneeded = event => {
  const db = event.target.result;
  db.createObjectStore('customers', { keyPath: 'id' });
};
```

## グラフィックスとアニメーション

視覚的な表現やアニメーションも作成できます。

```javascript
// Canvas API
const canvas = document.getElementById('myCanvas');
const ctx = canvas.getContext('2d');
ctx.fillStyle = 'red';
ctx.fillRect(10, 10, 100, 100);

// SVG操作
const svgElement = document.createElementNS('http://www.w3.org/2000/svg', 'circle');
svgElement.setAttribute('cx', '50');
svgElement.setAttribute('cy', '50');
svgElement.setAttribute('r', '40');
svgElement.setAttribute('fill', 'blue');
document.querySelector('svg').appendChild(svgElement);

// CSS アニメーション制御
document.getElementById('box').addEventListener('click', function() {
  this.style.transition = 'transform 0.5s ease-in-out';
  this.style.transform = 'rotate(45deg) scale(1.2)';
});

// requestAnimationFrame
function animate(time) {
  element.style.left = (Math.sin(time / 1000) * 50 + 50) + 'px';
  requestAnimationFrame(animate);
}
requestAnimationFrame(animate);
```

## サーバーサイドJavaScript (Node.js)

Node.jsを使えば、JavaScriptはサーバーサイドでも動作します。

```javascript
// HTTPサーバー
const http = require('http');
const server = http.createServer((req, res) => {
  res.statusCode = 200;
  res.setHeader('Content-Type', 'text/plain');
  res.end('Hello World');
});
server.listen(3000, '127.0.0.1', () => {
  console.log('サーバーが起動しました');
});

// ファイル操作
const fs = require('fs');
fs.readFile('input.txt', 'utf8', (err, data) => {
  if (err) throw err;
  console.log(data);
});

// Express（Webフレームワーク）
const express = require('express');
const app = express();
app.get('/', (req, res) => {
  res.send('Hello World!');
});
app.listen(3000);
```

## モバイル・デスクトップアプリ開発

JavaScriptはWebだけでなく、様々なプラットフォームで利用できます。

```javascript
// React Native（モバイルアプリ）
import React from 'react';
import { Text, View } from 'react-native';

export default function App() {
  return (
    <View style={{ flex: 1, justifyContent: 'center', alignItems: 'center' }}>
      <Text>こんにちは、React Native!</Text>
    </View>
  );
}

// Electron（デスクトップアプリ）
const { app, BrowserWindow } = require('electron');

function createWindow() {
  const win = new BrowserWindow({
    width: 800,
    height: 600,
    webPreferences: {
      nodeIntegration: true
    }
  });
  win.loadFile('index.html');
}

app.whenReady().then(createWindow);
```

## 最新のJavaScript機能

JavaScriptは常に進化しています。ECMAScript（JavaScript標準）の最新機能も活用できます。

```javascript
// オプショナルチェイニング
const user = { profile: { name: '田中' } };
const userName = user?.profile?.name; // エラーにならない

// Nullish coalescing（null結合演算子）
const count = someValue ?? 0; // someValueがnullまたはundefinedの場合に0を代入

// BigInt（大きな整数）
const bigNumber = 1234567890123456789012345678901234567890n;

// Dynamic import（動的インポート）
const loadModule = async () => {
  const module = await import('./myModule.js');
  module.default();
};

// TopLevelAwait（トップレベルでのawait）
const data = await fetch('https://api.example.com/data').then(r => r.json());
console.log(data);
```

## まとめ

この記事ではJavaScriptでできることを幅広く紹介しました。JavaScriptは：

- Webページの動的な操作（DOM操作）
- ユーザーインタラクションの処理（イベント処理）
- データの操作と計算
- 非同期処理によるAPIとの連携
- ブラウザのWeb APIの活用
- グラフィックスとアニメーション
- サーバーサイド開発（Node.js）
- クロスプラットフォームアプリ開発

など、多岐にわたる用途に活用できます。

モダンなJavaScriptは非常に強力で、シンプルなスクリプトから複雑なアプリケーションまで、様々なプログラムを作成できます。この記事が、あなたのJavaScript活用の幅を広げるきっかけになれば幸いです。

## 参考リンク

- [MDN Web Docs - JavaScript](https://developer.mozilla.org/ja/docs/Web/JavaScript)
- [JavaScript.info](https://ja.javascript.info/)
- [Node.js公式サイト](https://nodejs.org/ja/)
- [ECMAScript最新情報](https://tc39.es/)
