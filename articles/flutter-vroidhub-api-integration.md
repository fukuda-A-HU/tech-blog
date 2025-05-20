---
title: "【Flutter入門】VRoidHub APIを使ったアバター表示アプリの作り方"
emoji: "👾"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["flutter", "vroidhub", "api", "dart", "初心者"]
published: false
---

## ゴール(はじめに)：FlutterでVRoidHubAPIを使う
FlutterアプリからVRoidHubAPIを利用し、VRoidHub上のアバター情報を取得・表示する方法を初心者向けに解説します。公式ドキュメント（https://developer.vroid.com/api/）を参考に、APIキーの取得からデータ表示までをシンプルなコード例で紹介します。

## やってみた結果
Flutterアプリ上でVRoidHubAPIから取得したアバター情報をリスト表示できるようになりました。この記事を読めば、APIの基本的な使い方とデータ取得・表示の流れが理解できます。

## 開発環境
- OS: Linux（macやWindowsでも可）
- Flutter 3.x
- VSCode
- Android/iOSエミュレータ
- httpパッケージ

プロジェクト構成例:
```
lib/
  main.dart
pubspec.yaml
```

## 事前準備
- VRoidHubの開発者登録とAPIキー取得（https://developer.vroid.com/）
- Flutterプロジェクトの作成済みであること
- zenn-cliインストール済み

## やったこと
1. 必要なパッケージのインストール
2. APIキーの安全な管理
3. HTTPリクエストの送信
4. レスポンスデータの解析
5. UIへのデータ表示

## 必要なパッケージのインストール
`pubspec.yaml`に`http`パッケージを追加します。

```yaml
# pubspec.yaml
...
dependencies:
  flutter:
    sdk: flutter
  http: ^1.2.0
```

ターミナルで以下を実行：
```bash
flutter pub get
```

## APIキーの設定
VRoidHubAPIを利用するには、APIキー（アクセストークン）が必要です。公式サイトでアプリ登録し、APIキーを取得してください。

APIキーはソースコードに直接書かず、`--dart-define`や`.env`ファイルなどで安全に管理しましょう。

例：`lib/main.dart`で環境変数から取得
```dart
import 'package:flutter_dotenv/flutter_dotenv.dart';

final apiKey = dotenv.env['VROIDHUB_API_KEY'];
```

`.env`ファイル例
```
VROIDHUB_API_KEY=取得したAPIキー
```

`pubspec.yaml`に`flutter_dotenv`を追加し、`main()`で`dotenv.load()`を呼び出してください。

## HTTPリクエストの送信
`http`パッケージを使ってVRoidHubAPIにGETリクエストを送信します。

例：アバター一覧取得APIを呼び出す
```dart
import 'package:http/http.dart' as http;
import 'dart:convert';

Future<void> fetchAvatars(String apiKey) async {
  final url = Uri.parse('https://api.vroid.com/me/avatars');
  final response = await http.get(
    url,
    headers: {
      'Authorization': 'Bearer $apiKey',
    },
  );
  if (response.statusCode == 200) {
    print('Success: ${response.body}');
  } else {
    print('Error: ${response.statusCode}');
  }
}
```

## レスポンスデータの解析
APIから返されるJSONレスポンスを解析し、必要なデータを抽出します。

例：アバター名リストを抽出
```dart
Future<List<String>> getAvatarNames(String apiKey) async {
  final url = Uri.parse('https://api.vroid.com/me/avatars');
  final response = await http.get(
    url,
    headers: {'Authorization': 'Bearer $apiKey'},
  );
  if (response.statusCode == 200) {
    final data = json.decode(response.body);
    final avatars = data['avatars'] as List<dynamic>;
    return avatars.map((a) => a['name'] as String).toList();
  } else {
    throw Exception('Failed to load avatars');
  }
}
```

## UIへのデータ表示
取得したアバター名リストをFlutterのウィジェットで表示します。

例：ListViewでアバター名を表示
```dart
import 'package:flutter/material.dart';

class AvatarList extends StatelessWidget {
  final List<String> avatarNames;
  const AvatarList({super.key, required this.avatarNames});

  @override
  Widget build(BuildContext context) {
    return ListView.builder(
      itemCount: avatarNames.length,
      itemBuilder: (context, index) {
        return ListTile(
          title: Text(avatarNames[index]),
        );
      },
    );
  }
}
```

`FutureBuilder`と組み合わせて非同期取得にも対応できます。

## 結論に対しての補足
- VRoidHubAPIの詳細は[公式ドキュメント](https://developer.vroid.com/api/)を参照してください。
- APIキーの管理には十分注意しましょう。
- 実際のアプリではエラーハンドリングや認証フローの実装も必要です。

## おわりにorまとめ
FlutterからVRoidHubAPIを使う基本的な流れを解説しました。APIの使い方に慣れれば、さまざまなデータ取得や連携が可能です。まずはシンプルな実装から始めてみましょう！
