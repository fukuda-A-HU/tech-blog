---
title: "Flutterで外部リンクを開く：url_launcherの使い方"
emoji: "🔗"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Flutter", "Dart", "url_launcher", "モバイルアプリ", "初心者向け"]
published: false
---

## ゴール(はじめに)：アプリから外部リンクやアプリを起動する

Flutterアプリ開発において、ウェブサイトを開いたり、電話をかけたり、メールを送信したりするなど、外部のアプリやサービスと連携する機能は非常に重要です。この記事では、Flutterの`url_launcher`パッケージを使って、アプリから様々な外部リンクやアプリを起動する方法を解説します。

## やってみた結果

この記事を読むことで、以下のことが簡単にできるようになります：

- Flutterアプリからウェブサイトを開く
- 電話アプリを起動して特定の番号に発信する
- メールアプリを起動して新規メールを作成する
- SMSアプリを起動してメッセージを作成する
- マップアプリを起動して特定の場所を表示する

また、リンクが開けるかどうかの事前確認方法や、外部リンク起動時の例外処理についても学べます。

## 開発環境

- Flutter 3.10.0以上
- Dart 3.0.0以上
- Android Studio または VS Code
- Android/iOSデバイスまたはエミュレーター

## 事前準備

まず、`url_launcher`パッケージをプロジェクトに追加する必要があります。`pubspec.yaml`ファイルに以下の依存関係を追加します：

```yaml
dependencies:
  flutter:
    sdk: flutter
  url_launcher: ^6.1.12  # 最新バージョンを使用してください
```

その後、依存関係を取得します：

```bash
flutter pub get
```

### プラットフォーム固有の設定

#### Android

Androidでは、外部リンクを開くための特別な設定は基本的に必要ありません。ただし、Android 11 (API レベル 30) 以上をターゲットにする場合、`AndroidManifest.xml`ファイルに以下の設定を追加することをおすすめします：

```xml
<queries>
  <intent>
    <action android:name="android.intent.action.VIEW" />
    <data android:scheme="https" />
  </intent>
  <intent>
    <action android:name="android.intent.action.DIAL" />
    <data android:scheme="tel" />
  </intent>
  <intent>
    <action android:name="android.intent.action.SEND" />
    <data android:mimeType="*/*" />
  </intent>
</queries>
```

これにより、アプリがウェブサイト、電話、共有機能などにアクセスする権限が明示的に宣言されます。

#### iOS

iOSでは、Info.plistファイルに使用するURLスキームを追加する必要があります。`ios/Runner/Info.plist`ファイルを開き、`<dict>`タグ内に以下を追加します：

```xml
<key>LSApplicationQueriesSchemes</key>
<array>
  <string>https</string>
  <string>http</string>
  <string>tel</string>
  <string>mailto</string>
  <string>sms</string>
  <string>maps</string>
</array>
```

## やったこと

それでは、`url_launcher`を使って様々なタイプのリンクを開く方法を見ていきましょう。

## url_launcherの基本的な使い方

まず、パッケージをインポートします：

```dart
import 'package:url_launcher/url_launcher.dart';
```

### ウェブサイトを開く

ウェブサイトを開くための基本的なコードは次のようになります：

```dart
Future<void> _launchURL() async {
  final Uri url = Uri.parse('https://zenn.dev');
  if (!await launchUrl(url)) {
    throw Exception('Could not launch $url');
  }
}
```

これを呼び出すボタンを作成してみましょう：

```dart
ElevatedButton(
  onPressed: _launchURL,
  child: Text('Zennを開く'),
)
```

### 外部ブラウザで開くか内部ウェブビューで開くかを指定する

デフォルトでは、ウェブURLは外部ブラウザで開きます。アプリ内のウェブビューで開きたい場合は、`launchUrl`の第二引数に`LaunchMode.inAppWebView`を指定します：

```dart
Future<void> _launchInAppWebView() async {
  final Uri url = Uri.parse('https://zenn.dev');
  if (!await launchUrl(
    url,
    mode: LaunchMode.inAppWebView,
  )) {
    throw Exception('Could not launch $url');
  }
}
```

### 電話をかける

電話アプリを起動して特定の番号に発信するには、`tel:`スキームを使用します：

```dart
Future<void> _makePhoneCall() async {
  final Uri phoneUri = Uri(
    scheme: 'tel',
    path: '0312345678',
  );
  if (!await launchUrl(phoneUri)) {
    throw Exception('電話アプリを起動できませんでした');
  }
}
```

電話をかけるボタンの例：

```dart
ElevatedButton(
  onPressed: _makePhoneCall,
  child: Text('電話をかける'),
)
```

### メールを送信する

メールアプリを起動して新規メールを作成するには、`mailto:`スキームを使用します：

```dart
Future<void> _sendEmail() async {
  final Uri emailUri = Uri(
    scheme: 'mailto',
    path: 'example@example.com',
    queryParameters: {
      'subject': 'Flutter url_launcher の件',
      'body': 'こんにちは、お問い合わせがあります。'
    },
  );
  if (!await launchUrl(emailUri)) {
    throw Exception('メールアプリを起動できませんでした');
  }
}
```

### SMSを送信する

SMSアプリを起動して特定の番号にメッセージを送信するには、`sms:`スキームを使用します：

```dart
Future<void> _sendSMS() async {
  final Uri smsUri = Uri(
    scheme: 'sms',
    path: '09012345678',
    queryParameters: {'body': 'こんにちは、テストメッセージです。'},
  );
  if (!await launchUrl(smsUri)) {
    throw Exception('SMSアプリを起動できませんでした');
  }
}
```

### 地図アプリを開く

地図アプリを起動して特定の場所を表示するには、`https://maps.google.com`のURLを使用します：

```dart
Future<void> _openMap() async {
  // 東京駅の座標
  final Uri mapUri = Uri.parse('https://maps.google.com/?q=35.681236,139.767125');
  if (!await launchUrl(mapUri, mode: LaunchMode.externalApplication)) {
    throw Exception('地図アプリを起動できませんでした');
  }
}
```

### URLが開けるか事前に確認する

URLを開く前に、そのURLが開けるかどうかを事前に確認することができます：

```dart
Future<void> _checkAndLaunchURL() async {
  final Uri url = Uri.parse('https://zenn.dev');
  
  if (await canLaunchUrl(url)) {
    await launchUrl(url);
  } else {
    // URLを開けない場合の処理
    print('このURLは開けません: $url');
  }
}
```

## より実践的なサンプルコード

以下は、様々なタイプのリンクを起動するボタンを持つシンプルなFlutterアプリの例です：

```dart
import 'package:flutter/material.dart';
import 'package:url_launcher/url_launcher.dart';

void main() => runApp(MyApp());

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'URL Launcher デモ',
      theme: ThemeData(
        primarySwatch: Colors.blue,
        useMaterial3: true,
      ),
      home: UrlLauncherDemo(),
    );
  }
}

class UrlLauncherDemo extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('URL Launcher デモ'),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            _buildLaunchButton(
              context,
              'ウェブサイトを開く',
              Icons.web,
              _launchWebsite,
            ),
            _buildLaunchButton(
              context,
              '電話をかける',
              Icons.phone,
              _makePhoneCall,
            ),
            _buildLaunchButton(
              context,
              'メールを送信',
              Icons.email,
              _sendEmail,
            ),
            _buildLaunchButton(
              context,
              'SMSを送信',
              Icons.sms,
              _sendSMS,
            ),
            _buildLaunchButton(
              context,
              '地図を開く',
              Icons.map,
              _openMap,
            ),
          ],
        ),
      ),
    );
  }

  Widget _buildLaunchButton(
    BuildContext context,
    String text,
    IconData icon,
    VoidCallback onPressed,
  ) {
    return Padding(
      padding: const EdgeInsets.symmetric(vertical: 8.0),
      child: ElevatedButton.icon(
        onPressed: () => _handleButtonPress(context, onPressed),
        icon: Icon(icon),
        label: Text(text),
        style: ElevatedButton.styleFrom(
          padding: EdgeInsets.symmetric(horizontal: 24, vertical: 12),
          textStyle: TextStyle(fontSize: 16),
        ),
      ),
    );
  }

  void _handleButtonPress(BuildContext context, VoidCallback action) {
    try {
      action();
    } catch (e) {
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(content: Text('エラーが発生しました: $e')),
      );
    }
  }

  Future<void> _launchWebsite() async {
    final Uri url = Uri.parse('https://zenn.dev');
    if (!await launchUrl(url)) {
      throw Exception('Could not launch $url');
    }
  }

  Future<void> _makePhoneCall() async {
    final Uri phoneUri = Uri(scheme: 'tel', path: '0312345678');
    if (!await launchUrl(phoneUri)) {
      throw Exception('電話アプリを起動できませんでした');
    }
  }

  Future<void> _sendEmail() async {
    final Uri emailUri = Uri(
      scheme: 'mailto',
      path: 'example@example.com',
      queryParameters: {
        'subject': 'Flutter url_launcher の件',
        'body': 'こんにちは、お問い合わせがあります。'
      },
    );
    if (!await launchUrl(emailUri)) {
      throw Exception('メールアプリを起動できませんでした');
    }
  }

  Future<void> _sendSMS() async {
    final Uri smsUri = Uri(
      scheme: 'sms',
      path: '09012345678',
      queryParameters: {'body': 'こんにちは、テストメッセージです。'},
    );
    if (!await launchUrl(smsUri)) {
      throw Exception('SMSアプリを起動できませんでした');
    }
  }

  Future<void> _openMap() async {
    // 東京駅の座標
    final Uri mapUri = Uri.parse('https://maps.google.com/?q=35.681236,139.767125');
    if (!await launchUrl(mapUri, mode: LaunchMode.externalApplication)) {
      throw Exception('地図アプリを起動できませんでした');
    }
  }
}
```

## 結論に対しての補足

### 実際のアプリでの使用例

`url_launcher`パッケージは、以下のようなケースで活用できます：

- アプリのヘルプページや利用規約など、外部ウェブサイトへのリンク
- カスタマーサポートへの電話やメールの問い合わせ機能
- 店舗の場所を地図で表示する機能
- SNSシェア機能（TwitterやFacebookなど）

### 注意点と対応策

1. **プラットフォーム固有の動作**
   - iOSとAndroidでは、URLスキームの対応状況が異なる場合があります
   - 必ずエラーハンドリングを実装し、URLが開けない場合の代替手段を用意しましょう

2. **アプリ内ウェブビューの制限**
   - アプリ内ウェブビューでは、特定のウェブサイト機能が動作しない場合があります
   - 重要なウェブコンテンツは、外部ブラウザで開くことを検討しましょう

3. **ユーザー体験の考慮**
   - ユーザーの同意なしに電話やメールアプリを突然起動すると、ユーザー体験が損なわれる可能性があります
   - アクションの前に確認ダイアログを表示することを検討してください

## おわりに

この記事では、Flutterの`url_launcher`パッケージを使って、アプリから外部リンクやアプリを起動する方法を紹介しました。ウェブサイト、電話、メール、SMS、地図など、様々な外部サービスとの連携が簡単に実装できることがわかりました。

`url_launcher`は非常に便利なパッケージですが、適切なエラーハンドリングとユーザー体験の考慮を忘れないようにしましょう。特に、異なるデバイスやOSバージョンでの動作確認は重要です。

あなたのアプリに外部リンク機能を追加して、より便利で使いやすいアプリを作成してみてください！

### 参考リンク
- [url_launcher パッケージ公式ドキュメント](https://pub.dev/packages/url_launcher)
- [Flutter公式ドキュメント - URL起動の実装](https://flutter.dev/docs/cookbook/navigation/navigate-to-a-web-page)
- [Flutter公式サンプル - URL Launcher](https://github.com/flutter/plugins/tree/main/packages/url_launcher/url_launcher/example)
