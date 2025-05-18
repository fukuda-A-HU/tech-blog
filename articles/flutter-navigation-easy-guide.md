---
title: "Flutter初心者向け: 簡単にマスターする画面遷移の方法"
emoji: "🚀"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Flutter", "Dart", "モバイル開発", "画面遷移", "初心者向け"]
published: false
---

# Flutter初心者向け: 簡単にマスターする画面遷移の方法

こんにちは！この記事では、Flutter初心者の方向けに、アプリ内で画面を切り替える「画面遷移」の実装方法を解説します。複数の方法を紹介するので、あなたのプロジェクトに合った方法を選べるようになりましょう！

## はじめに

アプリ開発において、画面遷移は基本中の基本です。ログイン画面からホーム画面へ、商品一覧から詳細画面へ...など、ユーザーは様々な画面間を行き来します。Flutterでは、この画面遷移を実装するための方法がいくつか用意されています。

この記事では、次の内容を学びます：

- 基本的な画面遷移（Navigator 1.0）
- 宣言的な画面遷移（Navigator 2.0 / Router）
- 人気のルーティングライブラリ
- 画面遷移時のアニメーション

## 1. 基本的な画面遷移（Navigator 1.0）

Flutterで最も基本的な画面遷移は、`Navigator`クラスを使用する方法です。これは命令型のAPIで、「次の画面に進む」「前の画面に戻る」といった操作を直感的に記述できます。

### 新しい画面への遷移

```dart
// 基本的な画面遷移（プッシュ）
Navigator.push(
  context,
  MaterialPageRoute(builder: (context) => SecondScreen()),
);

// 名前付きルートを使用した遷移
Navigator.pushNamed(context, '/second');
```

### 前の画面に戻る

```dart
// 前の画面に戻る（ポップ）
Navigator.pop(context);

// データを返しながら前の画面に戻る
Navigator.pop(context, '選択結果');
```

### 実践例: 基本的な画面遷移

以下は、ボタンをタップして次の画面に遷移し、また前の画面に戻る簡単な例です。

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
      home: const FirstScreen(),
    );
  }
}

// 最初の画面
class FirstScreen extends StatelessWidget {
  const FirstScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('最初の画面'),
      ),
      body: Center(
        child: ElevatedButton(
          onPressed: () {
            // 次の画面に遷移
            Navigator.push(
              context,
              MaterialPageRoute(builder: (context) => const SecondScreen()),
            );
          },
          child: const Text('次の画面へ'),
        ),
      ),
    );
  }
}

// 2つ目の画面
class SecondScreen extends StatelessWidget {
  const SecondScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('2つ目の画面'),
      ),
      body: Center(
        child: ElevatedButton(
          onPressed: () {
            // 前の画面に戻る
            Navigator.pop(context);
          },
          child: const Text('前の画面に戻る'),
        ),
      ),
    );
  }
}
```

### 名前付きルートの設定

複数の画面を扱う場合は、名前付きルート（Named Routes）を使うと便利です。`MaterialApp`の`routes`プロパティで定義します。

```dart
MaterialApp(
  title: '画面遷移デモ',
  initialRoute: '/',
  routes: {
    '/': (context) => const FirstScreen(),
    '/second': (context) => const SecondScreen(),
    '/third': (context) => const ThirdScreen(),
  },
);

// 使い方
Navigator.pushNamed(context, '/second');
```

### 画面遷移時のデータの受け渡し

画面間でデータを渡したい場合は、コンストラクタやルート引数を使用します。

```dart
// コンストラクタでデータを渡す場合
Navigator.push(
  context,
  MaterialPageRoute(
    builder: (context) => DetailScreen(item: selectedItem),
  ),
);

// 名前付きルートでデータを渡す場合
Navigator.pushNamed(
  context,
  '/detail',
  arguments: {'id': 123, 'title': '商品タイトル'},
);

// 受け取り側
final args = ModalRoute.of(context)!.settings.arguments as Map<String, dynamic>;
final id = args['id'];
final title = args['title'];
```

## 2. 宣言的な画面遷移（Navigator 2.0 / Router）

Flutter 1.22以降では、より宣言的なアプローチである「Navigator 2.0」が導入されました。これは特にWebアプリケーションやディープリンク対応、複雑なナビゲーション管理に適しています。

Navigator 2.0は、画面を「どのように」遷移させるかではなく、「どの画面が表示されるべきか」を宣言的に記述します。

### Router の基本的な実装

```dart
MaterialApp.router(
  routerDelegate: MyRouterDelegate(),
  routeInformationParser: MyRouteInformationParser(),
);
```

Navigator 2.0は強力ですが、実装が複雑になりがちです。以下は簡略化した例です：

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MyApp());
}

// アプリのページ状態を表すクラス
class MyAppState {
  final String currentPage;
  final Object? arguments;

  MyAppState(this.currentPage, {this.arguments});
}

// ルーターデリゲート
class MyRouterDelegate extends RouterDelegate<MyAppState>
    with ChangeNotifier, PopNavigatorRouterDelegateMixin<MyAppState> {
  @override
  final GlobalKey<NavigatorState> navigatorKey;

  MyRouterDelegate() : navigatorKey = GlobalKey<NavigatorState>();

  String _currentPage = '/';
  Object? _arguments;

  @override
  MyAppState get currentConfiguration {
    return MyAppState(_currentPage, arguments: _arguments);
  }

  @override
  Widget build(BuildContext context) {
    return Navigator(
      key: navigatorKey,
      pages: [
        MaterialPage(
          key: const ValueKey('home'),
          child: const FirstScreen(),
        ),
        if (_currentPage == '/second')
          MaterialPage(
            key: const ValueKey('second'),
            child: const SecondScreen(),
          ),
      ],
      onPopPage: (route, result) {
        if (!route.didPop(result)) {
          return false;
        }

        // ポップ時に状態を更新
        if (_currentPage != '/') {
          _currentPage = '/';
          notifyListeners();
        }
        return true;
      },
    );
  }

  // 画面遷移のメソッド
  void toSecondPage() {
    _currentPage = '/second';
    notifyListeners();
  }

  void toHomePage() {
    _currentPage = '/';
    notifyListeners();
  }

  @override
  Future<void> setNewRoutePath(MyAppState configuration) async {
    _currentPage = configuration.currentPage;
    _arguments = configuration.arguments;
  }
}

// ルート情報パーサー
class MyRouteInformationParser extends RouteInformationParser<MyAppState> {
  @override
  Future<MyAppState> parseRouteInformation(RouteInformation routeInformation) async {
    final uri = Uri.parse(routeInformation.uri.toString());
    final pathSegments = uri.pathSegments;

    if (pathSegments.isEmpty) {
      return MyAppState('/');
    }

    if (pathSegments.first == 'second') {
      return MyAppState('/second');
    }

    return MyAppState('/');
  }

  @override
  RouteInformation restoreRouteInformation(MyAppState configuration) {
    return RouteInformation(uri: Uri.parse(configuration.currentPage));
  }
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp.router(
      title: 'Navigator 2.0 デモ',
      routerDelegate: MyRouterDelegate(),
      routeInformationParser: MyRouteInformationParser(),
    );
  }
}

// 以下は画面のウィジェット（FirstScreen, SecondScreen）
// 省略（上記の例と同様）
class FirstScreen extends StatelessWidget {
  const FirstScreen({super.key});

  @override
  Widget build(BuildContext context) {
    final RouterDelegate delegate = Router.of(context).routerDelegate as MyRouterDelegate;
    
    return Scaffold(
      appBar: AppBar(
        title: const Text('最初の画面'),
      ),
      body: Center(
        child: ElevatedButton(
          onPressed: () {
            // Navigator 2.0での画面遷移
            (delegate as MyRouterDelegate).toSecondPage();
          },
          child: const Text('次の画面へ'),
        ),
      ),
    );
  }
}

class SecondScreen extends StatelessWidget {
  const SecondScreen({super.key});

  @override
  Widget build(BuildContext context) {
    final RouterDelegate delegate = Router.of(context).routerDelegate as MyRouterDelegate;
    
    return Scaffold(
      appBar: AppBar(
        title: const Text('2つ目の画面'),
      ),
      body: Center(
        child: ElevatedButton(
          onPressed: () {
            // Navigator 2.0での画面遷移
            (delegate as MyRouterDelegate).toHomePage();
          },
          child: const Text('前の画面に戻る'),
        ),
      ),
    );
  }
}
```

Navigator 2.0は強力ですが、複雑なため、多くの開発者はサードパーティのルーティングライブラリを使用しています。

## 3. 人気のルーティングライブラリ

より簡単に高度なルーティングを実現するため、いくつかの人気ライブラリがあります。

### 3.1 GoRouter

GoRouterは、Navigator 2.0の機能を簡単に使えるようにしたライブラリです。特にディープリンク対応やWebアプリケーションに適しています。

まず、依存関係を追加します：

```yaml
dependencies:
  go_router: ^6.0.0  # バージョンは適宜更新してください
```

GoRouterの基本的な使い方：

```dart
import 'package:flutter/material.dart';
import 'package:go_router/go_router.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp.router(
      routerConfig: _router,
      title: 'GoRouter デモ',
    );
  }
}

// ルーター設定
final _router = GoRouter(
  routes: [
    GoRoute(
      path: '/',
      builder: (context, state) => const HomeScreen(),
      routes: [
        GoRoute(
          path: 'details/:id',
          builder: (context, state) {
            // URLパラメータを取得
            final id = state.pathParameters['id']!;
            return DetailsScreen(id: id);
          },
        ),
      ],
    ),
  ],
);

// ホーム画面
class HomeScreen extends StatelessWidget {
  const HomeScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('ホーム')),
      body: Center(
        child: ElevatedButton(
          onPressed: () {
            // GoRouterを使った画面遷移
            context.go('/details/123');
          },
          child: const Text('詳細画面へ'),
        ),
      ),
    );
  }
}

// 詳細画面
class DetailsScreen extends StatelessWidget {
  final String id;
  
  const DetailsScreen({super.key, required this.id});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('詳細')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Text('ID: $id'),
            ElevatedButton(
              onPressed: () {
                // 前の画面に戻る
                context.pop();
              },
              child: const Text('戻る'),
            ),
          ],
        ),
      ),
    );
  }
}
```

### 3.2 auto_route

auto_routeは、コード生成を活用して型安全なルーティングを提供するライブラリです。

依存関係の追加：

```yaml
dependencies:
  auto_route: ^5.0.0

dev_dependencies:
  auto_route_generator: ^5.0.0
  build_runner: ^2.3.0
```

routes.dart ファイルを作成：

```dart
import 'package:auto_route/auto_route.dart';
import 'package:flutter/material.dart';

part 'routes.gr.dart'; // コード生成されるファイル

@AutoRouterConfig()
class AppRouter extends _$AppRouter {
  @override
  List<AutoRoute> get routes => [
        AutoRoute(page: HomeRoute.page, initial: true),
        AutoRoute(page: DetailsRoute.page),
      ];
}

@RoutePage()
class HomeScreen extends StatelessWidget {
  const HomeScreen({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('ホーム')),
      body: Center(
        child: ElevatedButton(
          onPressed: () {
            // auto_routeを使った画面遷移
            context.router.push(DetailsRoute(id: '123'));
          },
          child: const Text('詳細画面へ'),
        ),
      ),
    );
  }
}

@RoutePage()
class DetailsScreen extends StatelessWidget {
  final String id;

  const DetailsScreen({Key? key, @PathParam('id') required this.id})
      : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('詳細')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Text('ID: $id'),
            ElevatedButton(
              onPressed: () {
                // 前の画面に戻る
                context.router.pop();
              },
              child: const Text('戻る'),
            ),
          ],
        ),
      ),
    );
  }
}

// コード生成の実行: flutter packages pub run build_runner build
```

これを使用するには、以下のコマンドでコード生成を行います：

```bash
flutter packages pub run build_runner build
```

### 3.3 GetX

GetXは、状態管理、依存性注入、ルーティングなど多機能なライブラリですが、ここではルーティング機能のみ紹介します。

依存関係の追加：

```yaml
dependencies:
  get: ^4.6.5
```

GetXを使ったルーティングの例：

```dart
import 'package:flutter/material.dart';
import 'package:get/get.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return GetMaterialApp(
      title: 'GetX ルーティングデモ',
      initialRoute: '/',
      getPages: [
        GetPage(name: '/', page: () => const HomeScreen()),
        GetPage(
          name: '/details/:id',
          page: () => const DetailsScreen(),
        ),
      ],
    );
  }
}

class HomeScreen extends StatelessWidget {
  const HomeScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('ホーム')),
      body: Center(
        child: ElevatedButton(
          onPressed: () {
            // GetXを使った画面遷移
            Get.toNamed('/details/123');
          },
          child: const Text('詳細画面へ'),
        ),
      ),
    );
  }
}

class DetailsScreen extends StatelessWidget {
  const DetailsScreen({super.key});

  @override
  Widget build(BuildContext context) {
    // パラメータの取得
    final id = Get.parameters['id'];
    
    return Scaffold(
      appBar: AppBar(title: const Text('詳細')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Text('ID: $id'),
            ElevatedButton(
              onPressed: () {
                // 前の画面に戻る
                Get.back();
              },
              child: const Text('戻る'),
            ),
          ],
        ),
      ),
    );
  }
}
```

## 4. 画面遷移時のアニメーション

Flutterでは、画面遷移時のアニメーションをカスタマイズすることもできます。

### 基本的なアニメーションのカスタマイズ

```dart
Navigator.push(
  context,
  PageRouteBuilder(
    pageBuilder: (context, animation, secondaryAnimation) => const SecondScreen(),
    transitionsBuilder: (context, animation, secondaryAnimation, child) {
      // フェードイン効果
      return FadeTransition(opacity: animation, child: child);
    },
    transitionDuration: const Duration(milliseconds: 500),
  ),
);
```

### よく使われる遷移アニメーションの例

```dart
// スライド遷移
SlideTransition(
  position: Tween<Offset>(
    begin: const Offset(1, 0), // 右からスライドイン
    end: Offset.zero,
  ).animate(animation),
  child: child,
)

// スケール遷移
ScaleTransition(
  scale: animation,
  child: child,
)

// 回転遷移
RotationTransition(
  turns: animation,
  child: child,
)
```

### Hero アニメーション

2つの画面間で要素を共有するようなアニメーション効果も実装できます。

```dart
// 1つ目の画面
Hero(
  tag: 'imageHero',
  child: Image.network('https://example.com/image.jpg'),
)

// 2つ目の画面（遷移先）
Hero(
  tag: 'imageHero', // 同じタグを使う
  child: Image.network('https://example.com/image.jpg'),
)
```

## まとめ

この記事では、Flutterでの画面遷移の実装方法を複数紹介しました：

1. **基本的な画面遷移（Navigator 1.0）**
   - 直感的で簡単な命令型API
   - 小〜中規模のアプリに適している

2. **宣言的な画面遷移（Navigator 2.0 / Router）**
   - 複雑なルーティングに対応
   - Webアプリやディープリンクに最適
   - ただし実装が複雑

3. **人気のルーティングライブラリ**
   - GoRouter: Navigator 2.0の簡易版
   - auto_route: 型安全なルーティング
   - GetX: 多機能フレームワークの一部

4. **画面遷移アニメーション**
   - カスタムアニメーションの実装
   - Heroアニメーション

初心者の方は、まず基本的なNavigator 1.0から始めて、徐々に必要に応じて高度な方法を試してみることをおすすめします。プロジェクトの規模や要件に合わせて、最適な画面遷移の方法を選択してください。

## 参考リンク

- [Flutter公式ドキュメント - Navigation](https://docs.flutter.dev/development/ui/navigation)
- [Flutter Navigator 2.0 ドキュメント](https://docs.flutter.dev/development/ui/navigation/navigator-2)
- [GoRouter パッケージ](https://pub.dev/packages/go_router)
- [auto_route パッケージ](https://pub.dev/packages/auto_route)
- [GetX パッケージ](https://pub.dev/packages/get)
