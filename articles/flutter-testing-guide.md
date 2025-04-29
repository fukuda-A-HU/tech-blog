---
title: "Flutter初心者向け: ユニットテストとウィジェットテストの基本"
emoji: "🧪"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Flutter", "Dart", "Test", "UnitTest", "WidgetTest"]
published: false
---

# Flutter初心者向け: ユニットテストとウィジェットテストの基本

## はじめに

Flutterアプリケーションの品質を確保するためには、適切なテストが欠かせません。テストを書くことで、バグの早期発見や機能の安定性を保証することができます。

この記事では、Flutter初心者の方向けに、基本的なユニットテストとウィジェットテストの書き方を解説します。難しい概念はなるべく避け、実践的なコード例を中心に説明していきます。

## Flutterのテスト概要

Flutterでは主に以下の3種類のテストがサポートされています：

1. **ユニットテスト**: 関数やクラスなど、単一の機能をテストします
2. **ウィジェットテスト**: Flutterのウィジェットの振る舞いをテストします
3. **インテグレーションテスト**: アプリ全体の動作をテストします

今回は最初の2つ、ユニットテストとウィジェットテストの基本を解説します。

## プロジェクトの準備

Flutterプロジェクトを作成すると、自動的に`test`ディレクトリが作られ、基本的なテスト環境が整います。テストに必要なパッケージは`pubspec.yaml`に次のように記述されています：

```yaml
dev_dependencies:
  flutter_test:
    sdk: flutter
```

追加のテストツールが必要な場合は、以下のパッケージも便利です：

```yaml
dev_dependencies:
  # 既存のものに加えて
  mockito: ^5.4.2
  test: ^1.24.3
```

## ユニットテストの書き方

まず簡単な計算を行うクラスを例に、ユニットテストの書き方を見ていきましょう。

### テスト対象のクラス

`lib/calculator.dart`ファイルを作成し、以下のような単純な計算機クラスを定義します：

```dart
class Calculator {
  int add(int a, int b) {
    return a + b;
  }
  
  int subtract(int a, int b) {
    return a - b;
  }
  
  int multiply(int a, int b) {
    return a * b;
  }
  
  double divide(int a, int b) {
    if (b == 0) {
      throw ArgumentError('除数は0以外の値を指定してください');
    }
    return a / b;
  }
}
```

### テストコードの作成

`test/calculator_test.dart`ファイルを作成し、以下のようにテストを記述します：

```dart
import 'package:flutter_test/flutter_test.dart';
import 'package:your_app_name/calculator.dart'; // あなたのアプリ名に変更してください

void main() {
  group('Calculator', () {
    late Calculator calculator;
    
    setUp(() {
      // 各テストの前に実行される
      calculator = Calculator();
    });
    
    test('足し算が正しく計算されること', () {
      // 実行
      final result = calculator.add(2, 3);
      
      // 検証
      expect(result, 5);
    });
    
    test('引き算が正しく計算されること', () {
      final result = calculator.subtract(5, 2);
      expect(result, 3);
    });
    
    test('掛け算が正しく計算されること', () {
      final result = calculator.multiply(4, 3);
      expect(result, 12);
    });
    
    test('割り算が正しく計算されること', () {
      final result = calculator.divide(10, 2);
      expect(result, 5.0);
    });
    
    test('0で割ると例外がスローされること', () {
      // 例外をテストする場合
      expect(() => calculator.divide(10, 0), throwsArgumentError);
    });
  });
}
```

### ユニットテストの基本要素

- **test**: 個々のテストケースを定義します
- **expect**: 期待する結果と実際の結果を比較します
- **group**: 関連するテストをグループ化します
- **setUp**: 各テストの前に実行される初期化処理を定義します
- **tearDown**: 各テストの後に実行されるクリーンアップ処理を定義します（上記例では使用していません）

## ウィジェットテストの書き方

次に、簡単なカウンターウィジェットを例にウィジェットテストの書き方を見ていきましょう。

### テスト対象のウィジェット

`lib/counter_widget.dart`ファイルを作成し、以下のようなカウンターウィジェットを定義します：

```dart
import 'package:flutter/material.dart';

class CounterWidget extends StatefulWidget {
  const CounterWidget({Key? key}) : super(key: key);

  @override
  _CounterWidgetState createState() => _CounterWidgetState();
}

class _CounterWidgetState extends State<CounterWidget> {
  int _counter = 0;

  void _incrementCounter() {
    setState(() {
      _counter++;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Column(
      mainAxisAlignment: MainAxisAlignment.center,
      children: <Widget>[
        Text(
          'カウント:',
        ),
        Text(
          '$_counter',
          style: Theme.of(context).textTheme.headlineMedium,
        ),
        ElevatedButton(
          onPressed: _incrementCounter,
          child: const Text('増加'),
        ),
      ],
    );
  }
}
```

### ウィジェットテストの作成

`test/counter_widget_test.dart`ファイルを作成し、以下のようにテストを記述します：

```dart
import 'package:flutter/material.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:your_app_name/counter_widget.dart'; // あなたのアプリ名に変更してください

void main() {
  group('CounterWidget', () {
    testWidgets('初期状態でカウンターが0であること', (WidgetTester tester) async {
      // ウィジェットをビルド
      await tester.pumpWidget(const MaterialApp(
        home: Scaffold(
          body: CounterWidget(),
        ),
      ));
      
      // 検証
      expect(find.text('0'), findsOneWidget);
      expect(find.text('1'), findsNothing);
    });
    
    testWidgets('ボタンを押すとカウンターが増加すること', (WidgetTester tester) async {
      // ウィジェットをビルド
      await tester.pumpWidget(const MaterialApp(
        home: Scaffold(
          body: CounterWidget(),
        ),
      ));
      
      // 初期状態の検証
      expect(find.text('0'), findsOneWidget);
      
      // ボタンを押す
      await tester.tap(find.byType(ElevatedButton));
      // アニメーションなどを含む更新を完了させる
      await tester.pump();
      
      // 更新後の状態を検証
      expect(find.text('1'), findsOneWidget);
      expect(find.text('0'), findsNothing);
    });
  });
}
```

### ウィジェットテストの基本要素

- **testWidgets**: ウィジェットテストのケースを定義します
- **tester.pumpWidget**: テスト対象のウィジェットをビルドします
- **tester.pump**: ウィジェットの状態更新後に再描画を促します
- **find**: ウィジェットを検索するためのメソッド群を提供します
- **findsOneWidget/findsNothing**: 検索結果の検証に使います

### よく使うFinderメソッド

```dart
// テキストで検索
find.text('テキスト');

// ウィジェットの型で検索
find.byType(ElevatedButton);

// キーで検索
find.byKey(const Key('my-key'));

// ウィジェットのインスタンスで検索
find.byWidget(myWidget);

// セマンティクスラベルで検索
find.bySemanticsLabel('ラベル');
```

## モックを使ったテスト

実際のアプリ開発では、APIやデータベースなどの外部依存があります。これらをテストする際には「モック」と呼ばれるテスト用の代替実装を使います。

簡単な例として、HTTPリクエストを行うサービスをモック化してテストする方法を見てみましょう。

### テスト対象のサービス

`lib/user_service.dart` を作成します：

```dart
import 'dart:convert';
import 'package:http/http.dart' as http;

class User {
  final int id;
  final String name;

  User({required this.id, required this.name});

  factory User.fromJson(Map<String, dynamic> json) {
    return User(
      id: json['id'],
      name: json['name'],
    );
  }
}

class UserService {
  final http.Client client;
  
  UserService({required this.client});
  
  Future<User> fetchUser(int id) async {
    final response = await client.get(
      Uri.parse('https://api.example.com/users/$id'),
    );
    
    if (response.statusCode == 200) {
      return User.fromJson(json.decode(response.body));
    } else {
      throw Exception('ユーザー情報の取得に失敗しました');
    }
  }
}
```

### モックを使ったテスト

`test/user_service_test.dart` を作成します：

```dart
import 'package:flutter_test/flutter_test.dart';
import 'package:http/http.dart' as http;
import 'package:mockito/annotations.dart';
import 'package:mockito/mockito.dart';
import 'package:your_app_name/user_service.dart'; // あなたのアプリ名に変更してください

// モッククラスを生成するアノテーション
@GenerateMocks([http.Client])
import 'user_service_test.mocks.dart'; // mockitoがビルド時に生成するファイル

void main() {
  group('UserService', () {
    test('正常なレスポンスからユーザーを取得できること', () async {
      // モックの作成
      final client = MockClient();
      
      // モックの振る舞いを定義
      when(client.get(Uri.parse('https://api.example.com/users/1')))
          .thenAnswer((_) async => http.Response(
                '{"id": 1, "name": "テストユーザー"}', 
                200,
              ));
      
      // テスト対象のサービスを作成
      final userService = UserService(client: client);
      
      // 実行
      final user = await userService.fetchUser(1);
      
      // 検証
      expect(user.id, 1);
      expect(user.name, 'テストユーザー');
    });
    
    test('エラーレスポンスの場合は例外をスローすること', () async {
      final client = MockClient();
      
      when(client.get(Uri.parse('https://api.example.com/users/1')))
          .thenAnswer((_) async => http.Response(
                'Not Found', 
                404,
              ));
      
      final userService = UserService(client: client);
      
      // 例外がスローされることを期待
      expect(userService.fetchUser(1), throwsException);
    });
  });
}
```

> 注意: mockitoを使う場合は、`build_runner`パッケージも必要です：
> ```
> flutter pub add dev:build_runner
> ```
> 
> モッククラスを生成するために以下のコマンドを実行します：
> ```
> flutter pub run build_runner build
> ```

## テストの実行方法

Flutterのテストは、コマンドラインやIDEから簡単に実行できます。

### コマンドラインからの実行

```bash
# 全てのテストを実行
flutter test

# 特定のテストファイルを実行
flutter test test/calculator_test.dart

# 特定のディレクトリのテストを実行
flutter test test/units/
```

### Visual Studio Codeでの実行

Visual Studio Codeを使っている場合は、テストファイルを開いた状態で:

1. テストファイル上部の「Run Tests」をクリック
2. または、各テストケースの横にある実行ボタンをクリック

### IntelliJ IDEAやAndroid Studioでの実行

IntelliJ IDEAやAndroid Studioを使っている場合は:

1. テストファイルを右クリック → Run
2. または、ガターの実行ボタンをクリック

## まとめ

この記事では、Flutter初心者の方向けに、基本的なユニットテストとウィジェットテストの書き方を解説しました。

テストを書くことで以下のメリットがあります：

1. バグの早期発見
2. リファクタリングの安全性向上
3. コードの品質向上
4. 仕様の明確化と文書化

最初はテストを書くのに時間がかかるかもしれませんが、プロジェクトが大きくなるにつれて、その価値は大きくなっていきます。少しずつテストを書く習慣をつけていくことをおすすめします。

## 参考リンク

- [Flutter公式ドキュメント - テスト](https://docs.flutter.dev/testing)
- [Flutter公式サンプル - テスト](https://github.com/flutter/samples/tree/master/testing_app)
- [Mockito パッケージ](https://pub.dev/packages/mockito)
- [flutter_test パッケージ](https://api.flutter.dev/flutter/flutter_test/flutter_test-library.html)
