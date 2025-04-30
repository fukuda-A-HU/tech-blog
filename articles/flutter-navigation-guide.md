---
title: "Flutter初心者のための画面遷移ガイド"
emoji: "🚀"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Flutter", "Dart", "モバイル開発", "ナビゲーション", "初心者向け"]
published: false
---

# Flutter初心者のための画面遷移ガイド

こんにちは！この記事では、Flutter初心者向けに、アプリ内での画面遷移（ナビゲーション）の基本的な実装方法を解説します。複数の画面を持つアプリを簡単に作れるようになりましょう！

## はじめに

モバイルアプリは通常、複数の画面で構成されています。ユーザーはホーム画面から詳細画面へ、あるいは設定画面へと移動します。こうした画面間の移動を管理する仕組みを「ナビゲーション」と呼びます。

Flutterでは、このナビゲーションを簡単かつ柔軟に実装できる機能が提供されています。この記事では、以下の内容を学びます：

- 基本的な画面遷移の実装方法
- 引数を渡しながらの画面遷移
- 名前付きルートを使った画面遷移
- Navigatorを活用した応用テクニック

## 基本的な画面遷移

### プロジェクトの準備

まずは新しいFlutterプロジェクトを作成します（既存のプロジェクトがある場合はスキップしてください）：

```bash
flutter create navigation_demo
cd navigation_demo
```

### シンプルな画面遷移の実装

最もシンプルな画面遷移は、`Navigator.push`メソッドを使って行います。以下の例では、ホーム画面から詳細画面へ移動する実装を示します：

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
      title: '画面遷移デモ',
      theme: ThemeData(
        primarySwatch: Colors.blue,
        useMaterial3: true,
      ),
      home: const HomeScreen(),
    );
  }
}

// ホーム画面
class HomeScreen extends StatelessWidget {
  const HomeScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('ホーム画面'),
      ),
      body: Center(
        child: ElevatedButton(
          onPressed: () {
            // 詳細画面へ遷移
            Navigator.push(
              context,
              MaterialPageRoute(builder: (context) => const DetailScreen()),
            );
          },
          child: const Text('詳細画面へ'),
        ),
      ),
    );
  }
}

// 詳細画面
class DetailScreen extends StatelessWidget {
  const DetailScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('詳細画面'),
      ),
      body: Center(
        child: ElevatedButton(
          onPressed: () {
            // 前の画面に戻る
            Navigator.pop(context);
          },
          child: const Text('戻る'),
        ),
      ),
    );
  }
}
```

このコードでは、ホーム画面の「詳細画面へ」ボタンをタップすると詳細画面に遷移し、詳細画面の「戻る」ボタンをタップするとホーム画面に戻ります。

`Navigator.push`は新しい画面をスタックの上に積み重ねるイメージで、`Navigator.pop`は一番上の画面をスタックから取り除くイメージです。

## 引数を渡しながらの画面遷移

実際のアプリでは、画面遷移時にデータも一緒に渡したいケースが多いです。例えば、商品一覧から商品詳細へ遷移する際に、選択した商品の情報を渡す必要があります。

以下の例では、引数を渡しながら画面遷移する方法を示します：

```dart
// 商品一覧画面
class ProductListScreen extends StatelessWidget {
  final List<Product> products = [
    Product(id: 1, name: 'スマートフォン', price: 50000),
    Product(id: 2, name: 'ノートパソコン', price: 120000),
    Product(id: 3, name: 'ワイヤレスイヤホン', price: 15000),
  ];

  ProductListScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('商品一覧'),
      ),
      body: ListView.builder(
        itemCount: products.length,
        itemBuilder: (context, index) {
          return ListTile(
            title: Text(products[index].name),
            subtitle: Text('${products[index].price}円'),
            onTap: () {
              // 選択した商品の情報を詳細画面に渡す
              Navigator.push(
                context,
                MaterialPageRoute(
                  builder: (context) => ProductDetailScreen(product: products[index]),
                ),
              );
            },
          );
        },
      ),
    );
  }
}

// 商品詳細画面
class ProductDetailScreen extends StatelessWidget {
  final Product product;

  const ProductDetailScreen({super.key, required this.product});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('商品詳細'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16.0),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Text('商品名: ${product.name}', style: const TextStyle(fontSize: 20)),
            const SizedBox(height: 8),
            Text('価格: ${product.price}円', style: const TextStyle(fontSize: 18)),
            const SizedBox(height: 16),
            ElevatedButton(
              onPressed: () {
                Navigator.pop(context);
              },
              child: const Text('戻る'),
            ),
          ],
        ),
      ),
    );
  }
}

// 商品モデル
class Product {
  final int id;
  final String name;
  final int price;

  Product({required this.id, required this.name, required this.price});
}
```

この例では、商品一覧画面から商品をタップすると、その商品の情報を`ProductDetailScreen`のコンストラクタに渡して詳細画面を表示しています。

## 名前付きルートを使った画面遷移

小規模なアプリでは前述の方法で問題ないですが、アプリが大きくなると画面遷移の管理が複雑になります。そんなときは「名前付きルート」を使うと便利です。

名前付きルートを使うと、画面に名前を付けて管理できるため、コードがシンプルになり、全体の画面構成も把握しやすくなります。

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
      title: '名前付きルートデモ',
      initialRoute: '/', // 初期画面
      routes: {
        '/': (context) => const HomeScreen(),
        '/settings': (context) => const SettingsScreen(),
        '/profile': (context) => const ProfileScreen(),
      },
    );
  }
}

class HomeScreen extends StatelessWidget {
  const HomeScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('ホーム'),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            ElevatedButton(
              onPressed: () {
                Navigator.pushNamed(context, '/settings');
              },
              child: const Text('設定画面へ'),
            ),
            const SizedBox(height: 16),
            ElevatedButton(
              onPressed: () {
                Navigator.pushNamed(context, '/profile');
              },
              child: const Text('プロフィール画面へ'),
            ),
          ],
        ),
      ),
    );
  }
}

class SettingsScreen extends StatelessWidget {
  const SettingsScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('設定'),
      ),
      body: Center(
        child: ElevatedButton(
          onPressed: () {
            Navigator.pop(context);
          },
          child: const Text('戻る'),
        ),
      ),
    );
  }
}

class ProfileScreen extends StatelessWidget {
  const ProfileScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('プロフィール'),
      ),
      body: Center(
        child: ElevatedButton(
          onPressed: () {
            Navigator.pop(context);
          },
          child: const Text('戻る'),
        ),
      ),
    );
  }
}
```

この例では、`MaterialApp`の`routes`プロパティで画面と名前の対応を定義し、`Navigator.pushNamed`で名前を指定して画面遷移しています。

### 引数付きの名前付きルート

名前付きルートでも、引数を渡す必要がある場合があります。次の例では、引数を渡す方法を示します：

```dart
// MaterialAppの設定
MaterialApp(
  title: '名前付きルートデモ',
  initialRoute: '/',
  routes: {
    '/': (context) => const HomeScreen(),
    '/product': (context) => const ProductDetailScreen(),
  },
  onGenerateRoute: (settings) {
    // 商品詳細画面の場合、引数を渡す
    if (settings.name == '/product') {
      final args = settings.arguments as Product;
      return MaterialPageRoute(
        builder: (context) {
          return ProductDetailScreen(product: args);
        },
      );
    }
    return null;
  },
);

// ホーム画面から商品詳細へ遷移
Navigator.pushNamed(
  context,
  '/product',
  arguments: Product(id: 1, name: 'スマートフォン', price: 50000),
);
```

この例では、`onGenerateRoute`を使って、名前付きルートに引数を渡しています。

## Navigatorを活用した応用テクニック

### ダイアログの表示

画面遷移ではなく、ダイアログを表示する場合も`Navigator`を使います：

```dart
// 確認ダイアログを表示
showDialog(
  context: context,
  builder: (BuildContext context) {
    return AlertDialog(
      title: const Text('確認'),
      content: const Text('この操作を実行しますか？'),
      actions: [
        TextButton(
          onPressed: () {
            Navigator.pop(context, 'Cancel'); // 'Cancel'を返して閉じる
          },
          child: const Text('キャンセル'),
        ),
        TextButton(
          onPressed: () {
            Navigator.pop(context, 'OK'); // 'OK'を返して閉じる
          },
          child: const Text('OK'),
        ),
      ],
    );
  },
).then((value) {
  // ダイアログの結果を処理
  if (value == 'OK') {
    // OKが押された場合の処理
    print('OK押されました');
  }
});
```

### 画面の置き換え

新しい画面に遷移して、元の画面を履歴から削除したい場合は`pushReplacement`を使います：

```dart
// 現在の画面をログイン画面に置き換え
Navigator.pushReplacement(
  context,
  MaterialPageRoute(builder: (context) => const LoginScreen()),
);
```

これは、ログアウト時にホーム画面からログイン画面に遷移する際などに便利です。

### 複数画面の置き換え

`pushAndRemoveUntil`を使うと、特定の条件で画面スタックをクリアしながら新しい画面に遷移できます：

```dart
// すべての画面を削除してホーム画面に遷移
Navigator.pushAndRemoveUntil(
  context,
  MaterialPageRoute(builder: (context) => const HomeScreen()),
  (route) => false, // falseを返すとすべての画面を削除
);
```

これは、ログイン成功後にログイン画面を含むすべての画面履歴を削除してホーム画面に遷移する場合などに使います。

## 画面遷移のベストプラクティス

1. **コードの整理**: 画面遷移のコードが複雑になる場合は、ナビゲーション用のヘルパークラスを作成すると管理しやすくなります。

2. **引数の型チェック**: 名前付きルートで引数を渡す場合は、型を明示的にチェックして、予期しないエラーを防ぎましょう。

3. **戻る操作の考慮**: ユーザーが「戻る」操作をした場合の動作を適切に設計しましょう。特にフォームの入力中などは重要です。

4. **遷移アニメーションのカスタマイズ**: `PageRouteBuilder`を使って遷移アニメーションをカスタマイズできます。

```dart
Navigator.push(
  context,
  PageRouteBuilder(
    pageBuilder: (context, animation, secondaryAnimation) => const DetailScreen(),
    transitionsBuilder: (context, animation, secondaryAnimation, child) {
      const begin = Offset(1.0, 0.0);
      const end = Offset.zero;
      const curve = Curves.easeInOut;
      var tween = Tween(begin: begin, end: end).chain(CurveTween(curve: curve));
      var offsetAnimation = animation.drive(tween);
      return SlideTransition(position: offsetAnimation, child: child);
    },
  ),
);
```

## まとめ

Flutterでの画面遷移の基本的な実装方法について解説しました：

- `Navigator.push`と`Navigator.pop`を使った基本的な画面遷移
- 引数を渡しながらの画面遷移
- 名前付きルートを使った画面管理
- ダイアログや画面置き換えなどの応用テクニック

これらの知識を活用すれば、複数の画面を持つ洗練されたFlutterアプリケーションを開発できるようになります。是非、実際のプロジェクトに取り入れてみてください！

## 参考リンク

- [Flutter Navigation & Routing（公式ドキュメント）](https://flutter.dev/docs/development/ui/navigation)
- [Navigator API（Flutter API）](https://api.flutter.dev/flutter/widgets/Navigator-class.html)
- [MaterialPageRoute（Flutter API）](https://api.flutter.dev/flutter/material/MaterialPageRoute-class.html)
