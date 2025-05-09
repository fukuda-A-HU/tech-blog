---
title: "Flutter で環境変数を扱う方法：flutter_dotenv 入門"
emoji: "🔐"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Flutter", "Dart", "環境変数", "dotenv", "モバイル開発"]
published: false
---

## ゴール(はじめに)：Flutterアプリで安全に環境変数を管理する

アプリ開発において、APIキーや接続情報などの機密情報をソースコードに直接書くことは避けるべき悪い習慣です。誤ってGitHubなどに公開してしまうと、悪用されるリスクがあります。

この記事では、Flutter開発において環境変数を安全に管理するための `flutter_dotenv` パッケージの使い方を解説します。このパッケージを使うことで、以下の目標を達成できます：

1. APIキーなどの機密情報をコードから分離できる
2. 開発環境と本番環境で異なる設定を切り替えられる
3. GitHubなどでソースを公開する際も安全に開発ができる

## やってみた結果

この記事の手順を実行すると、以下のことが実現できます：

- `.env` ファイルを使った環境変数の管理方法がわかる
- 機密情報をソースコードから分離できる
- 開発・ステージング・本番環境で設定を簡単に切り替えられる
- アプリ内で環境変数を安全に利用できる

flutter_dotenvを活用すれば、チーム開発においても安全かつ効率的に環境設定を管理できるようになります。

## 開発環境

- Flutter 3.0.0以上
- Dart 2.17.0以上
- 任意のエディタ（VS Code推奨）

フォルダ構造：
```
my_flutter_app/
├── .env                  # 環境変数ファイル（Gitで管理しない）
├── .env.example          # 環境変数の例（Gitで管理する）
├── .gitignore            # Gitの除外設定
├── pubspec.yaml          # Flutterプロジェクト設定
├── lib/
│   ├── main.dart         # メインコード
│   └── config.dart       # 環境変数アクセス用のクラス
└── assets/               # アセットディレクトリ
    └── .env.development  # 開発環境用環境変数（例）
```

## 事前準備

Flutterプロジェクトが既に作成されていることを前提とします。まだプロジェクトを作っていない場合は、以下のコマンドでプロジェクトを作成してください。

```bash
flutter create my_flutter_app
cd my_flutter_app
```

## やったこと

### 1. flutter_dotenvパッケージの導入

まず、flutter_dotenvパッケージを導入します。`pubspec.yaml`ファイルを開き、dependenciesセクションに以下を追加します：

```yaml
dependencies:
  flutter:
    sdk: flutter
  # 以下を追加
  flutter_dotenv: ^5.0.2  # 最新バージョンを確認してください
```

パッケージをインストールします：

```bash
flutter pub get
```

### 2. 環境変数ファイルの作成

プロジェクトのルートディレクトリに `.env` ファイルを作成します。このファイルには実際の環境変数が記述され、Gitには含めません。

```
# .env ファイル
API_URL=https://api.example.com
API_KEY=your_secret_api_key
DEBUG_MODE=true
```

また、チームメンバーや将来の自分のために、どのような環境変数が必要かわかるよう、`.env.example` ファイルも作成します。

```
# .env.example ファイル
API_URL=https://api.example.com
API_KEY=your_api_key_here
DEBUG_MODE=true
```

### 3. .gitignoreの設定

環境変数ファイルが誤ってGitにコミットされないよう、`.gitignore` ファイルに以下を追加します：

```
# 環境変数ファイル
.env
.env.local
.env.production
```

一方、例示用ファイルはGitで管理するため、除外しません：

```
# .gitignoreには以下を含めない
# .env.example
```

### 4. アセットの設定

環境変数ファイルをアプリに含めるには、`pubspec.yaml` ファイルの `flutter` セクションに以下を追加します：

```yaml
flutter:
  assets:
    - .env
    - assets/.env.development  # 複数の環境設定を使う場合
    - assets/.env.production   # 複数の環境設定を使う場合
```

### 5. 環境変数の読み込み

アプリ起動時に環境変数を読み込むために、`main.dart` ファイルを以下のように編集します：

```dart
import 'package:flutter/material.dart';
import 'package:flutter_dotenv/flutter_dotenv.dart';

Future<void> main() async {
  // 環境変数ファイルの読み込み
  await dotenv.load(fileName: ".env");
  
  // アプリケーションの起動
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter Dotenv Demo',
      theme: ThemeData(primarySwatch: Colors.blue),
      home: const HomePage(),
    );
  }
}

class HomePage extends StatelessWidget {
  const HomePage({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    // 環境変数を取得
    final apiUrl = dotenv.env['API_URL'] ?? 'API URLが設定されていません';
    
    return Scaffold(
      appBar: AppBar(title: const Text('Flutter Dotenv デモ')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            const Text('環境変数の値:', style: TextStyle(fontSize: 18)),
            const SizedBox(height: 8),
            Text(apiUrl, style: TextStyle(fontSize: 16, color: Colors.blue)),
            const SizedBox(height: 20),
            // デバッグモードかどうかを表示
            Text(
              '現在のモード: ${dotenv.env['DEBUG_MODE'] == 'true' ? 'デバッグ' : '本番'}',
              style: TextStyle(fontSize: 16),
            ),
          ],
        ),
      ),
    );
  }
}
```

### 6. 環境変数アクセス用のクラスの作成

より整理された方法として、専用のクラスを作成して環境変数にアクセスすることをおすすめします。`lib/config.dart` ファイルを作成し、次のコードを追加します：

```dart
import 'package:flutter_dotenv/flutter_dotenv.dart';

class Config {
  // APIのURL
  static String get apiUrl => dotenv.env['API_URL'] ?? '';
  
  // APIキー
  static String get apiKey => dotenv.env['API_KEY'] ?? '';
  
  // デバッグモード
  static bool get isDebug => dotenv.env['DEBUG_MODE'] == 'true';
  
  // 環境変数の存在確認
  static bool hasEnv(String name) => dotenv.env.containsKey(name);
}
```

この方法なら、以下のように簡単に環境変数にアクセスできます：

```dart
import 'config.dart';

// 環境変数の使用例
void example() {
  final url = Config.apiUrl;
  final apiKey = Config.apiKey;
  
  if (Config.isDebug) {
    print('デバッグモードで動作中');
  }
}
```

### 7. 複数の環境設定ファイルの利用

開発・ステージング・本番など、複数の環境を切り替えたい場合は、各環境ごとに設定ファイルを用意できます。

1. `assets/.env.development` ファイルを作成：
```
API_URL=https://dev-api.example.com
API_KEY=dev_api_key
DEBUG_MODE=true
```

2. `assets/.env.production` ファイルを作成：
```
API_URL=https://api.example.com
API_KEY=prod_api_key
DEBUG_MODE=false
```

3. 環境に応じてファイルを選択するように `main.dart` を修正：

```dart
import 'package:flutter/material.dart';
import 'package:flutter_dotenv/flutter_dotenv.dart';

Future<void> main() async {
  // 環境変数ファイルの読み込み（環境に応じて切り替え）
  const String environment = String.fromEnvironment('ENVIRONMENT', defaultValue: 'development');
  
  await dotenv.load(fileName: environment == 'production' 
      ? "assets/.env.production" 
      : "assets/.env.development");
  
  runApp(const MyApp());
}
```

4. 実行時に環境を指定してアプリを起動：

```bash
# 開発環境として実行
flutter run --dart-define=ENVIRONMENT=development

# 本番環境として実行
flutter run --dart-define=ENVIRONMENT=production
```

## 結論に対しての補足

### 環境変数のセキュリティ

flutter_dotenvを使ってアプリにバンドルされる環境変数ファイルは、APKやIPAファイルを解析することで参照できてしまう可能性があります。そのため、以下のような対策も検討してください：

1. **極秘情報は避ける**: エンドユーザーデバイスに配布されるアプリに、非常に重要な秘密情報を含めないようにする
2. **サーバーサイド認証**: クライアントのAPI認証には、APIキーだけでなくサーバーサイドでの認証も組み合わせる
3. **暗号化**: 環境変数値を暗号化し、実行時に復号化する方法も考えられます

### その他の環境変数管理ライブラリ

flutter_dotenv以外にも、以下のようなパッケージがあります：

1. **flutter_config**: Flutterに特化した環境変数管理
2. **envied**: コードジェネレータを使った安全な環境変数管理

### 公式ドキュメント

詳しくは、以下の公式ドキュメントを参照してください：
- [flutter_dotenv パッケージ](https://pub.dev/packages/flutter_dotenv)
- [Dart 環境変数](https://dart.dev/guides/environment-declarations)

## おわりに

この記事では、flutter_dotenvを使ってFlutterアプリで環境変数を安全に管理する方法を紹介しました。環境変数を適切に管理することは、セキュリティの観点からも開発効率の観点からも重要です。

特に複数人での開発やオープンソースプロジェクトでは、APIキーなどの機密情報を適切に管理することで、セキュリティリスクを大幅に軽減できます。また、異なる環境（開発・テスト・本番）での設定切り替えも簡単になり、開発効率が向上します。

モバイルアプリでの環境変数管理は、Webアプリケーションとは異なる点があることを忘れないでください。最終的に端末にバンドルされることを考慮したセキュリティ設計が重要です。

ぜひこの記事を参考に、あなたのFlutterプロジェクトでも環境変数を安全に管理してみてください。
