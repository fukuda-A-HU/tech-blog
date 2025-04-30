---
title: "Flutter初心者向け: 基本的なウィジェットとその配置方法"
emoji: "🧩"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Flutter", "Dart", "モバイル開発", "UI", "初心者向け"]
published: false
---

# Flutter初心者向け: 基本的なウィジェットとその配置方法

こんにちは！この記事では、Flutter初心者の方向けに、基本的なウィジェットの種類とそれらを画面上に配置する方法を解説します。Flutterを始めたばかりの方でも、様々なウィジェットを使って魅力的なUIを作れるようになりましょう！

## はじめに

Flutterでは、画面上のすべての要素が「ウィジェット」と呼ばれるパーツで構成されています。テキスト、ボタン、画像、コンテナなど、すべてがウィジェットです。そして、これらのウィジェットを組み合わせることで、複雑なユーザーインターフェースを構築していきます。

この記事では、次の内容を学びます：

- 基本的なウィジェットの種類
- レイアウトウィジェットの使い方
- ウィジェットをカスタマイズする方法
- 実践的なウィジェット配置の例

## 基本的なウィジェットの種類

Flutterには数多くのウィジェットが用意されていますが、まずは基本的なものから見ていきましょう。

### 1. 表示系ウィジェット

これらは画面に情報を表示するための基本的なウィジェットです。

#### Text

テキストを表示するための最も基本的なウィジェットです。

```dart
Text(
  'こんにちは、Flutter！',
  style: TextStyle(
    fontSize: 24,
    fontWeight: FontWeight.bold,
    color: Colors.blue,
  ),
)
```

#### Image

画像を表示するためのウィジェットです。ネットワーク画像、アセット画像、ファイル画像などをサポートしています。

```dart
// アセット画像の表示
Image.asset('assets/images/flutter_logo.png')

// ネットワーク画像の表示
Image.network('https://example.com/image.jpg')
```

#### Icon

マテリアルデザインのアイコンやカスタムアイコンを表示します。

```dart
Icon(
  Icons.favorite,
  color: Colors.red,
  size: 30,
)
```

### 2. 入力系ウィジェット

ユーザーからの入力を受け付けるためのウィジェットです。

#### Button

タップ操作を受け付けるボタンウィジェットです。

```dart
ElevatedButton(
  onPressed: () {
    // ボタンがタップされたときの処理
    print('ボタンがタップされました');
  },
  child: Text('タップしてください'),
)
```

その他にも、`TextButton`（フラットなボタン）や`OutlinedButton`（輪郭のあるボタン）などがあります。

#### TextField

テキスト入力を受け付けるウィジェットです。

```dart
TextField(
  decoration: InputDecoration(
    labelText: 'メールアドレス',
    hintText: 'example@email.com',
    border: OutlineInputBorder(),
  ),
  onChanged: (text) {
    // テキストが変更されたときの処理
    print('入力値: $text');
  },
)
```

### 3. コンテナ系ウィジェット

他のウィジェットを格納し、装飾するためのウィジェットです。

#### Container

最も基本的なコンテナウィジェットで、サイズ、パディング、マージン、装飾などを設定できます。

```dart
Container(
  width: 200,
  height: 100,
  padding: EdgeInsets.all(16),
  margin: EdgeInsets.symmetric(vertical: 10),
  decoration: BoxDecoration(
    color: Colors.amber,
    borderRadius: BorderRadius.circular(10),
    boxShadow: [
      BoxShadow(
        color: Colors.grey.withOpacity(0.5),
        spreadRadius: 2,
        blurRadius: 5,
        offset: Offset(0, 3),
      ),
    ],
  ),
  child: Text('こんにちは'),
)
```

#### Card

マテリアルデザインのカードを表現するウィジェットです。

```dart
Card(
  elevation: 4, // 影の強さ
  child: Padding(
    padding: EdgeInsets.all(16),
    child: Text('カード内のテキスト'),
  ),
)
```

## レイアウトウィジェット

複数のウィジェットを配置するためのレイアウトウィジェットは、UIデザインの骨組みとなる重要な要素です。

### 1. Row と Column

`Row`は子ウィジェットを水平方向に、`Column`は垂直方向に配置します。

```dart
// 水平配置（Row）
Row(
  mainAxisAlignment: MainAxisAlignment.spaceEvenly, // 主軸方向の配置
  crossAxisAlignment: CrossAxisAlignment.center,   // 交差軸方向の配置
  children: [
    Icon(Icons.star, size: 50),
    Icon(Icons.star, size: 50),
    Icon(Icons.star, size: 50),
  ],
)

// 垂直配置（Column）
Column(
  mainAxisAlignment: MainAxisAlignment.center,
  crossAxisAlignment: CrossAxisAlignment.start,
  children: [
    Text('1行目'),
    Text('2行目'),
    Text('3行目'),
  ],
)
```

### 2. Stack

子ウィジェットを重ねて表示します。

```dart
Stack(
  children: [
    Image.asset('assets/background.jpg'),
    Positioned(
      bottom: 10,
      right: 10,
      child: Text('キャプション', style: TextStyle(color: Colors.white)),
    ),
  ],
)
```

### 3. Expanded と Flexible

`Row`や`Column`の中で、使用可能なスペースを調整するためのウィジェットです。

```dart
Row(
  children: [
    // 利用可能なスペースの1/3を使用
    Expanded(
      flex: 1,
      child: Container(color: Colors.red, height: 100),
    ),
    // 利用可能なスペースの2/3を使用
    Expanded(
      flex: 2,
      child: Container(color: Colors.blue, height: 100),
    ),
  ],
)
```

## ウィジェットのカスタマイズ

各ウィジェットは様々なプロパティを持っており、これらを調整することでカスタマイズできます。

### スタイルの適用

多くのウィジェットは`style`プロパティを持っており、これを使って見た目をカスタマイズできます。

```dart
Text(
  'スタイル付きテキスト',
  style: TextStyle(
    fontSize: 20,
    fontWeight: FontWeight.bold,
    color: Colors.deepPurple,
    letterSpacing: 1.2,
    fontFamily: 'Roboto',
  ),
)
```

### パディングとマージンの適用

`Padding`ウィジェットやコンテナのプロパティを使って、余白を設定できます。

```dart
Padding(
  padding: EdgeInsets.all(16), // 全方向に16の余白
  child: Text('パディング付きテキスト'),
)

Container(
  margin: EdgeInsets.only(top: 10, left: 20), // 上と左に余白
  child: Text('マージン付きテキスト'),
)
```

## 実践的なウィジェット配置の例

以下は、これまで学んだウィジェットを組み合わせて、シンプルなプロフィールカードを作成する例です。

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'ウィジェットデモ',
      theme: ThemeData(
        primarySwatch: Colors.blue,
        useMaterial3: true,
      ),
      home: const HomePage(),
    );
  }
}

class HomePage extends StatelessWidget {
  const HomePage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('プロフィール'),
      ),
      body: Center(
        child: Card(
          margin: const EdgeInsets.all(16),
          elevation: 4,
          child: Padding(
            padding: const EdgeInsets.all(16),
            child: Column(
              mainAxisSize: MainAxisSize.min,
              children: [
                // プロフィール画像
                const CircleAvatar(
                  radius: 50,
                  backgroundImage: NetworkImage(
                    'https://via.placeholder.com/150',
                  ),
                ),
                const SizedBox(height: 16),
                // 名前
                const Text(
                  '山田 太郎',
                  style: TextStyle(
                    fontSize: 24,
                    fontWeight: FontWeight.bold,
                  ),
                ),
                const SizedBox(height: 8),
                // 肩書き
                const Text(
                  'フラッターエンジニア',
                  style: TextStyle(
                    fontSize: 16,
                    color: Colors.grey,
                  ),
                ),
                const SizedBox(height: 16),
                // 自己紹介
                const Text(
                  'Flutterを使ったアプリ開発が大好きです。UI/UXに関心があり、使いやすいアプリを作るのが得意です。',
                  textAlign: TextAlign.center,
                ),
                const SizedBox(height: 16),
                // コンタクトボタン
                Row(
                  mainAxisAlignment: MainAxisAlignment.spaceEvenly,
                  children: [
                    ElevatedButton.icon(
                      onPressed: () {},
                      icon: const Icon(Icons.email),
                      label: const Text('メール'),
                    ),
                    OutlinedButton.icon(
                      onPressed: () {},
                      icon: const Icon(Icons.phone),
                      label: const Text('電話'),
                    ),
                  ],
                ),
              ],
            ),
          ),
        ),
      ),
    );
  }
}
```

このコードは、以下の要素を含むプロフィールカードを作成します：
- プロフィール画像（CircleAvatar）
- 名前と肩書きのテキスト
- 自己紹介文
- 連絡先ボタン（メールと電話）

## よく使われるウィジェットの組み合わせパターン

実際のアプリ開発では、特定のパターンでウィジェットを組み合わせることが多いです。

### リスト表示

`ListView`を使ったリスト表示は、多くのアプリで見られる一般的なパターンです。

```dart
ListView.builder(
  itemCount: 10,
  itemBuilder: (context, index) {
    return ListTile(
      leading: CircleAvatar(child: Text('${index + 1}')),
      title: Text('アイテム ${index + 1}'),
      subtitle: Text('アイテムの説明文'),
      trailing: const Icon(Icons.arrow_forward_ios),
      onTap: () {
        // タップ時の処理
      },
    );
  },
)
```

### グリッド表示

`GridView`を使って、アイテムをグリッド状に表示するパターンです。

```dart
GridView.builder(
  gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
    crossAxisCount: 2, // 横に並べる数
    childAspectRatio: 1.0, // 子要素の縦横比
    crossAxisSpacing: 10, // 横方向のスペース
    mainAxisSpacing: 10, // 縦方向のスペース
  ),
  itemCount: 10,
  itemBuilder: (context, index) {
    return Card(
      child: Center(
        child: Text('アイテム ${index + 1}'),
      ),
    );
  },
)
```

## まとめ

この記事では、Flutterの基本的なウィジェットとそれらを画面上に配置する方法について解説しました。以下が主なポイントです：

- Flutterではすべての画面要素がウィジェットです
- 表示系、入力系、コンテナ系など様々な種類のウィジェットがあります
- レイアウトウィジェット（Row、Column、Stack等）を使ってUIを構成します
- ウィジェットの各種プロパティを使って、見た目をカスタマイズできます
- 実際のアプリでは、様々なウィジェットを組み合わせて使います

Flutterの強みは、これらのウィジェットを組み合わせることで、複雑なUIも比較的簡単に実装できる点にあります。この記事を参考に、様々なウィジェットを試してみて、Flutterのウィジェットシステムの理解を深めていただければ幸いです。

## 参考リンク

- [Flutter公式ウィジェットカタログ](https://flutter.dev/docs/development/ui/widgets)
- [Flutter基本ウィジェット](https://flutter.dev/docs/development/ui/widgets/basics)
- [レイアウトウィジェット](https://flutter.dev/docs/development/ui/widgets/layout)
- [マテリアルデザインウィジェット](https://flutter.dev/docs/development/ui/widgets/material)
