---
title: "Flutter初心者向け：簡単なWebアプリ開発入門"
emoji: "🌐"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Flutter", "Web", "Dart", "フロントエンド", "クロスプラットフォーム"]
published: false
---

# Flutter初心者向け：簡単なWebアプリ開発入門

こんにちは！この記事では、Flutter初心者の方向けに、Flutterを使って簡単なWebアプリを作る方法を解説します。Flutterは元々モバイルアプリ開発のためのフレームワークとして登場しましたが、現在では Web、デスクトップアプリの開発にも対応しています。

この記事を読むことで、Flutterの基本的な概念を理解し、シンプルな天気予報アプリをWeb向けに開発する方法が分かるようになります。

## 目次

1. Flutterとは？
2. 開発環境のセットアップ
3. Flutterプロジェクトの作成
4. Web対応の設定
5. シンプルな天気予報アプリの作成
6. Webアプリのビルドとデプロイ
7. まとめと次のステップ

## 1. Flutterとは？

Flutterは、Googleが開発したオープンソースのUIフレームワークです。1つのコードベースから、iOS、Android、Web、デスクトップアプリを開発できるのが大きな特徴です。

Flutterの主な利点：

- **クロスプラットフォーム開発**: 一度コードを書けば、複数のプラットフォームで動作します
- **ホットリロード**: コードの変更をリアルタイムで確認できます
- **カスタマイズ可能なウィジェット**: 美しいUIを柔軟に構築できます
- **高パフォーマンス**: ネイティブに近いパフォーマンスを発揮します

## 2. 開発環境のセットアップ

まずは、Flutterの開発環境をセットアップしましょう。

### Flutter SDKのインストール

1. [Flutter公式サイト](https://flutter.dev/docs/get-started/install)から、お使いのOSに合わせてFlutter SDKをダウンロードします。

2. ダウンロードしたZIPファイルを解凍し、任意の場所に配置します（例: `C:\flutter` や `~/flutter`）。

3. 環境変数のPATHに、Flutter SDKの `bin` ディレクトリを追加します。

### 開発ツールのインストール

Flutter開発には、Visual Studio Code（VS Code）や Android Studio などのIDEを使用できます。ここでは、VS Codeを例に説明します。

1. [VS Code](https://code.visualstudio.com/)をダウンロードしてインストールします。

2. VS Codeを起動し、拡張機能タブ（Ctrl+Shift+X）で「Flutter」を検索し、インストールします。Dart拡張機能も自動的にインストールされます。

### インストールの確認

ターミナル（コマンドプロンプト）で以下のコマンドを実行し、Flutterが正しくインストールされているか確認します：

```bash
flutter doctor
```

このコマンドは、Flutter開発環境に問題がないかチェックし、見つかった問題の解決方法を提案してくれます。

## 3. Flutterプロジェクトの作成

開発環境が準備できたら、新しいFlutterプロジェクトを作成しましょう。

```bash
flutter create weather_app
cd weather_app
```

このコマンドは `weather_app` という名前のプロジェクトを作成します。プロジェクト名はスネークケース（小文字とアンダースコア）を使用する必要があります。

## 4. Web対応の設定

Flutter 2.0以降では、Webサポートはデフォルトで有効になっています。念のため、Webサポートが有効になっているか確認しましょう：

```bash
flutter devices
```

Webデバイス（Chrome、Edge など）が表示されない場合は、以下のコマンドでWebサポートを有効にします：

```bash
flutter config --enable-web
```

次に、アプリをWebモードで実行するためのコマンドを実行します：

```bash
flutter run -d chrome
```

これにより、サンプルのカウンターアプリがChromeで起動します。

## 5. シンプルな天気予報アプリの作成

次に、プロジェクトの内容を編集して、シンプルな天気予報アプリを作成しましょう。今回は外部APIとの連携は省略し、モックデータを使用してUIを構築します。

### 1. `lib/main.dart`ファイルを編集

まず、`lib/main.dart`ファイルを開き、内容を以下のコードに置き換えます：

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const WeatherApp());
}

class WeatherApp extends StatelessWidget {
  const WeatherApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter Weather App',
      theme: ThemeData(
        primarySwatch: Colors.blue,
        visualDensity: VisualDensity.adaptivePlatformDensity,
      ),
      home: const WeatherHomePage(title: '天気予報アプリ'),
    );
  }
}

class WeatherHomePage extends StatefulWidget {
  const WeatherHomePage({super.key, required this.title});

  final String title;

  @override
  State<WeatherHomePage> createState() => _WeatherHomePageState();
}

class _WeatherHomePageState extends State<WeatherHomePage> {
  // 都市リスト
  final List<Map<String, dynamic>> cities = [
    {'name': '東京', 'weather': '晴れ', 'temperature': 28, 'icon': Icons.wb_sunny},
    {'name': '大阪', 'weather': '曇り', 'temperature': 26, 'icon': Icons.cloud},
    {'name': '札幌', 'weather': '雨', 'temperature': 18, 'icon': Icons.beach_access},
    {'name': '福岡', 'weather': '晴れ', 'temperature': 30, 'icon': Icons.wb_sunny},
    {'name': '沖縄', 'weather': '曇り時々晴れ', 'temperature': 32, 'icon': Icons.wb_cloudy},
  ];

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text(widget.title),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            const Text(
              '今日の天気予報',
              style: TextStyle(fontSize: 24, fontWeight: FontWeight.bold),
            ),
            const SizedBox(height: 20),
            Expanded(
              child: ListView.builder(
                itemCount: cities.length,
                itemBuilder: (context, index) {
                  final city = cities[index];
                  return Card(
                    margin: const EdgeInsets.symmetric(horizontal: 16, vertical: 8),
                    elevation: 4,
                    child: ListTile(
                      leading: Icon(
                        city['icon'] as IconData,
                        color: _getWeatherColor(city['weather'] as String),
                        size: 40,
                      ),
                      title: Text(
                        city['name'] as String,
                        style: const TextStyle(fontSize: 18, fontWeight: FontWeight.bold),
                      ),
                      subtitle: Text(city['weather'] as String),
                      trailing: Text(
                        '${city['temperature']}°C',
                        style: const TextStyle(fontSize: 20),
                      ),
                    ),
                  );
                },
              ),
            ),
          ],
        ),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () {
          // 天気データの更新処理（実際にはAPIリクエストなどが入る）
          setState(() {
            // データを更新する代わりに、温度を少し変更してみる
            for (var city in cities) {
              city['temperature'] = (city['temperature'] as int) + ([-1, 0, 1]..shuffle()).first;
            }
          });
        },
        tooltip: '天気を更新',
        child: const Icon(Icons.refresh),
      ),
    );
  }

  // 天気に応じた色を返す
  Color _getWeatherColor(String weather) {
    switch (weather) {
      case '晴れ':
        return Colors.orange;
      case '曇り':
      case '曇り時々晴れ':
        return Colors.grey;
      case '雨':
        return Colors.blue;
      default:
        return Colors.black;
    }
  }
}
```

このコードでは、都市のリストとそれぞれの天気情報を表示するシンプルな天気予報アプリを作成しています。各都市はカード形式で表示され、右下のボタンをクリックすると温度がランダムに更新される仕組みです。

## 6. Webアプリのビルドとデプロイ

アプリが完成したら、Webアプリとしてビルドしましょう。

### Webアプリのビルド

以下のコマンドを実行して、Webアプリをビルドします：

```bash
flutter build web
```

ビルドが完了すると、`build/web` ディレクトリに静的ファイルが生成されます。

### デプロイオプション

ビルドしたWebアプリは、以下のようなさまざまな方法でデプロイできます：

1. **GitHub Pages**
   
   GitHubリポジトリにコードをプッシュし、GitHub Pagesで公開できます。

2. **Firebase Hosting**
   
   Firebaseのホスティングサービスを使用すると、数分でデプロイできます：

   ```bash
   # Firebaseツールのインストール
   npm install -g firebase-tools
   
   # Firebaseにログイン
   firebase login
   
   # プロジェクトの初期化
   firebase init
   
   # デプロイ
   firebase deploy
   ```

3. **Netlify、Vercel、GitHub Pagesなど**
   
   他にもNetlifyやVercelなどの静的ホスティングサービスを使用できます。ビルドした`build/web`ディレクトリの内容をアップロードするだけです。

## 7. まとめと次のステップ

この記事では、Flutter初心者向けに、簡単な天気予報Webアプリの作成方法を解説しました。ここで学んだことを基に、さらに機能を追加してみましょう。

### 次のステップとして考えられること：

1. **実際の天気APIと連携する**
   - OpenWeatherMap APIなどを使用して、実際の天気データを取得する
   - `http` パッケージを使用してAPIリクエストを実装する

2. **状態管理を導入する**
   - Provider、Riverpod、BLoCなどの状態管理ライブラリを使用する
   - アプリの状態をより効率的に管理する

3. **UIをカスタマイズする**
   - アニメーションを追加する
   - ダークモードを実装する
   - レスポンシブデザインを強化する

4. **テスト実装**
   - ユニットテスト、ウィジェットテストを実装して品質を高める

### 参考資料：

- [Flutter公式ドキュメント](https://flutter.dev/docs)
- [Flutter Webサポート](https://flutter.dev/web)
- [dartpad.dev](https://dartpad.dev/) - ブラウザでDartとFlutterを試せるサイト
- [pub.dev](https://pub.dev/) - Flutterパッケージのリポジトリ

Flutter Web開発は日々進化しています。この記事が、あなたのFlutter Web開発の第一歩になれば幸いです。楽しいコーディングを！
