---
title: "Flutter初心者のためのUnity連携ガイド - flutter-unity-widgetの使い方"
emoji: "🎮"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["flutter", "unity", "mobile", "gamedev", "crossplatform"]
published: false
---

# Flutter初心者のためのUnity連携ガイド - flutter-unity-widgetの使い方

こんにちは！この記事では、Flutter初心者の方向けに、Unityで作成したコンテンツをFlutterアプリに埋め込む方法を解説します。「flutter-unity-widget」というプラグインを使って、UnityのリッチなゲームやインタラクティブコンテンツをFlutterアプリに統合する手順を、ステップバイステップで紹介します。

## この記事で学べること

- flutter-unity-widgetの基本的な使い方
- Unity側の準備とエクスポート方法
- Flutter側での統合手順
- 実際に動作する簡単なサンプルプロジェクトの作成

Unity開発の経験がなくても大丈夫です！この記事を通じて、Flutterアプリの中でUnityコンテンツを動かす基本的な方法を理解できるようになります。

## 前提条件

この記事は以下のツールがインストールされていることを前提としています。まだインストールしていない場合は、各ツールの公式サイトからダウンロードしてください。

- Flutter SDK（最新版）
- Android Studio または Visual Studio Code
- Unity Hub と Unity 2020.3 LTS 以降
- iOS開発の場合は Xcode（macOS環境）

## flutter-unity-widgetとは？

[flutter-unity-widget](https://github.com/juicycleff/flutter-unity-view-widget)は、FlutterアプリにUnityのコンテンツをシームレスに統合するためのプラグインです。

このプラグインを使うことで、以下のようなことが可能になります：

- Unityで作成したゲームやインタラクティブコンテンツをFlutterアプリに埋め込む
- Unityコンテンツとのデータのやり取り（Unityからの通知やFlutterからUnityへのコマンド送信など）
- Unityビューのサイズ調整やフルスクリーン表示
- AR/VRコンテンツの統合

Unityはリッチなグラフィックスやゲーム要素、ARなどを比較的簡単に実装できるため、これらの機能をFlutterアプリに追加したい場合に大変便利です。

## 実装手順

Unityコンテンツを含むFlutterアプリを作成する手順は大きく分けて以下の5つのステップになります：

1. Flutterプロジェクトの準備
2. Unityプロジェクトの作成
3. Unityプロジェクトのエクスポート
4. Flutterプロジェクトへの統合
5. Flutter-Unityの通信処理の実装

それでは順番に見ていきましょう！

## Step 1: Flutterプロジェクトの準備

まず、新しいFlutterプロジェクトを作成します。

```bash
flutter create flutter_unity_sample
cd flutter_unity_sample
```

次に、pubspec.yamlファイルを開き、flutter-unity-widgetパッケージを依存関係に追加します。

```yaml
dependencies:
  flutter:
    sdk: flutter
  cupertino_icons: ^1.0.2
  flutter_unity_widget: ^2022.2.0 # 最新バージョンを確認してください
```

追加したら、依存関係を更新します。

```bash
flutter pub get
```

## Step 2: Unityプロジェクトの作成

次に、統合するUnityプロジェクトを作成します。今回は簡単な回転する立方体を表示するプロジェクトを作成します。

### Unityプロジェクトの新規作成

1. Unity Hubを開きます
2. 「新規プロジェクト」ボタンをクリックします
3. テンプレートとして「3D」を選択します
4. プロジェクト名を「UnityProject」にして、場所はFlutterプロジェクトと同じ階層か覚えやすい場所を指定します
5. 「作成」ボタンをクリックします

### シーンの作成

Unityが起動したら、以下の手順で簡単なシーンを作成します：

1. 新しいシーンを作成し、「MainScene」という名前で保存します
2. シーンに立方体を追加します：「GameObject」>「3D Object」>「Cube」
3. 立方体を回転させるスクリプトを作成します：
   - 「Project」ウィンドウで右クリックし、「Create」>「C# Script」を選択
   - 「RotateCube」という名前を付けます
   - スクリプトをダブルクリックして編集します

RotateCube.csスクリプトの内容を以下のように編集します：

```csharp
using UnityEngine;

public class RotateCube : MonoBehaviour
{
    public float rotationSpeed = 30.0f;
    
    void Update()
    {
        // 毎フレーム、Y軸を中心に回転
        transform.Rotate(0, rotationSpeed * Time.deltaTime, 0);
    }
    
    // Flutterからのメッセージを受け取るメソッド
    public void ChangeColor(string colorHex)
    {
        Color newColor;
        if (ColorUtility.TryParseHtmlString("#" + colorHex, out newColor))
        {
            GetComponent<Renderer>().material.color = newColor;
        }
    }
}
```

4. このスクリプトを立方体にアタッチします：
   - Hierarchyウィンドウで立方体を選択
   - Inspector上部のAddComponentボタンをクリック
   - 「Scripts」>「RotateCube」を選択

5. Flutter-Unity間の通信を行うためのスクリプトを作成します：
   - 「FlutterUnityInterface」という名前のC#スクリプトを作成
   - 以下のコードを入力します

```csharp
using UnityEngine;
using UnityEngine.UI;
using System.Runtime.InteropServices;

public class FlutterUnityInterface : MonoBehaviour
{
    // Unityからのメッセージを送るためのメソッド
    public void SendMessageToFlutter(string message)
    {
        #if UNITY_ANDROID && !UNITY_EDITOR
            using (AndroidJavaClass unityPlayer = new AndroidJavaClass("com.unity3d.player.UnityPlayer"))
            {
                AndroidJavaObject activity = unityPlayer.GetStatic<AndroidJavaObject>("currentActivity");
                activity.Call("SendMessageToFlutter", message);
            }
        #elif UNITY_IOS && !UNITY_EDITOR
            // iOSの場合はObjective-Cブリッジを通じて呼び出す
            // 実際に実装する場合は追加の対応が必要
            NativeAPI.SendMessageToFlutter(message);
        #endif
        
        // エディタでのデバッグ用
        Debug.Log($"SendMessageToFlutter: {message}");
    }
    
    // Flutterからのメッセージを受け取るメソッド
    public void HandleMessage(string message)
    {
        Debug.Log($"Received message from Flutter: {message}");
        
        // メッセージに応じて処理を分ける
        if (message.StartsWith("COLOR:"))
        {
            string colorCode = message.Replace("COLOR:", "");
            GameObject cube = GameObject.Find("Cube");
            if (cube != null)
            {
                cube.GetComponent<RotateCube>().ChangeColor(colorCode);
            }
        }
        else if (message == "PING")
        {
            // Flutterにレスポンスを返す
            SendMessageToFlutter("PONG");
        }
    }
}
```

6. 空のゲームオブジェクトを作成し、このスクリプトをアタッチします：
   - Hierarchyウィンドウで右クリックし、「Create Empty」を選択
   - 名前を「FlutterUnityInterface」に変更
   - FlutterUnityInterfaceスクリプトをアタッチします

## Step 3: Unityプロジェクトのエクスポート

Unityプロジェクトをエクスポートする前に、以下の設定を行います：

### ビルド設定の構成（Android向け）

1. 「File」>「Build Settings」を開きます
2. 「Android」を選択し、「Switch Platform」をクリックします（必要な場合）
3. 「Player Settings」をクリックし、以下の設定を行います：
   - 「Other Settings」>「Rendering」>「Auto Graphics API」のチェックを外し、OpenGLESを上位に設定
   - 「Other Settings」>「Identification」>「Minimum API Level」を24以上に設定
   - 「Other Settings」>「Configuration」>「Scripting Backend」をIL2CPPに設定
   - 「Publishing Settings」>「Build」で「Custom Base Gradle Template」にチェックを入れる

### Unityプロジェクトのエクスポート

1. 「File」>「Build Settings」を開く
2. 「Export Project」にチェックを入れる
3. 「Export」ボタンをクリックし、エクスポート先として「unity_export」フォルダを指定する（Flutterプロジェクトの隣に作成するとよい）

これで、Unityプロジェクトがエクスポートされます。

## Step 4: Flutterプロジェクトへの統合

### Androidの設定（Android向け）

Flutterプロジェクトの「android」フォルダにある設定ファイルを編集します：

1. `android/settings.gradle`ファイルを開き、以下の内容を追加します：

```gradle
include ':unityExport'
project(':unityExport').projectDir = file('../../unity_export/unityExport')
```

2. `android/app/build.gradle`を開き、dependenciesセクションに以下を追加します：

```gradle
dependencies {
    implementation project(':unityExport')
    // 他の依存関係...
}
```

3. `android/app/src/main/AndroidManifest.xml`にカメラ権限（ARを使用する場合）を追加します：

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    package="com.example.flutter_unity_sample">
    
    <uses-permission android:name="android.permission.CAMERA" />
    
    <application
       ...
    </application>
</manifest>
```

### iOSの設定（iOS向け：macOSのみ）

iOSでビルドする場合は、XcodeでUnityプロジェクトとFlutterプロジェクトを統合する必要があります。詳細なiOS用の設定はやや複雑なため、公式ドキュメントを参照してください。

## Step 5: Flutterアプリの実装

最後に、FlutterアプリからUnityコンテンツを表示し、通信する部分を実装します。

`lib/main.dart`を以下のように編集します：

```dart
import 'package:flutter/material.dart';
import 'package:flutter_unity_widget/flutter_unity_widget.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter Unity Sample',
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      home: const UnityViewExample(),
    );
  }
}

class UnityViewExample extends StatefulWidget {
  const UnityViewExample({Key? key}) : super(key: key);

  @override
  _UnityViewExampleState createState() => _UnityViewExampleState();
}

class _UnityViewExampleState extends State<UnityViewExample> {
  // UnityWidgetのコントローラー
  UnityWidgetController? _unityWidgetController;
  
  // Unityからのメッセージを保存する変数
  String _messageFromUnity = 'Unityからのメッセージはここに表示されます';
  
  // 色の選択肢
  final List<Color> _colors = [
    Colors.red,
    Colors.green,
    Colors.blue,
    Colors.yellow,
    Colors.purple,
  ];

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Flutter Unity 連携サンプル'),
      ),
      body: Column(
        children: [
          // Unityビューを表示（画面の上部2/3程度）
          Expanded(
            flex: 3,
            child: UnityWidget(
              onUnityCreated: _onUnityCreated,
              onUnityMessage: _onUnityMessage,
              onUnitySceneLoaded: _onUnitySceneLoaded,
              fullscreen: false,
            ),
          ),
          
          // Flutterで実装したUI（画面の下部1/3程度）
          Expanded(
            flex: 2,
            child: Container(
              padding: const EdgeInsets.all(16.0),
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.stretch,
                children: [
                  // Unityからのメッセージを表示
                  Text(
                    _messageFromUnity,
                    style: const TextStyle(fontSize: 16),
                  ),
                  const SizedBox(height: 16),
                  
                  // 通信テスト用ボタン
                  ElevatedButton(
                    onPressed: _sendPingToUnity,
                    child: const Text('Unity通信テスト (PING送信)'),
                  ),
                  const SizedBox(height: 16),
                  
                  // 色変更のセクション
                  const Text('立方体の色を変更:'),
                  const SizedBox(height: 8),
                  
                  // 色ボタンの行
                  Row(
                    mainAxisAlignment: MainAxisAlignment.spaceEvenly,
                    children: _colors.map((color) {
                      return GestureDetector(
                        onTap: () => _changeUnityColor(color),
                        child: Container(
                          width: 40,
                          height: 40,
                          decoration: BoxDecoration(
                            color: color,
                            shape: BoxShape.circle,
                            border: Border.all(color: Colors.black, width: 2),
                          ),
                        ),
                      );
                    }).toList(),
                  ),
                ],
              ),
            ),
          ),
        ],
      ),
    );
  }

  // Unityビューが作成されたときのコールバック
  void _onUnityCreated(UnityWidgetController controller) {
    _unityWidgetController = controller;
    setState(() {
      _messageFromUnity = 'Unity初期化完了！';
    });
  }

  // Unityからメッセージを受け取るコールバック
  void _onUnityMessage(dynamic message) {
    print('Unityからのメッセージ: $message');
    setState(() {
      _messageFromUnity = 'Unityからのメッセージ: $message';
    });
  }

  // Unityのシーンがロードされたときのコールバック
  void _onUnitySceneLoaded(SceneLoaded sceneInfo) {
    print('Unityシーンがロードされました: ${sceneInfo.name}');
    setState(() {
      _messageFromUnity = 'シーンロード完了: ${sceneInfo.name}';
    });
  }

  // UnityにPINGメッセージを送信
  void _sendPingToUnity() {
    _unityWidgetController?.postMessage(
      'FlutterUnityInterface',
      'HandleMessage',
      'PING',
    );
    setState(() {
      _messageFromUnity = 'PINGをUnityに送信しました...';
    });
  }

  // 立方体の色を変更するメッセージを送信
  void _changeUnityColor(Color color) {
    // Colorを#RRGGBB形式の文字列に変換（#を除く）
    String colorHex = color.value.toRadixString(16).substring(2, 8);
    
    _unityWidgetController?.postMessage(
      'FlutterUnityInterface',
      'HandleMessage',
      'COLOR:$colorHex',
    );
    
    setState(() {
      _messageFromUnity = '色を変更: #$colorHex';
    });
  }
}
```

## アプリのビルドと実行

これで、UnityコンテンツをFlutterアプリに統合した準備が整いました。アプリをビルドして実行しましょう。

```bash
flutter run
```

Androidデバイスまたはエミュレータでアプリを実行すると、以下のような画面が表示されるはずです：

- 画面上部：Unityで実装した回転する立方体
- 画面下部：Flutterで実装したUI（メッセージ表示エリアとボタン）

UIの色ボタンをタップすると、Unityの立方体の色が変わります。また、「Unity通信テスト」ボタンをタップすると、Unityに「PING」メッセージが送信され、Unityから「PONG」レスポンスが返ってきます。

## よくあるトラブルシューティング

### ビルドエラー

**問題**: Androidビルド時に「Manifest merger failed」というエラーが出る

**解決策**: AndroidManifest.xmlで競合している設定がある場合、tools:replaceを使って解決します：

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    package="com.example.flutter_unity_sample">
    
    <application
        android:label="flutter_unity_sample"
        android:icon="@mipmap/ic_launcher"
        tools:replace="android:label">
        <!-- 他の設定 -->
    </application>
</manifest>
```

**問題**: 「Native libraries」関連のエラーが出る

**解決策**: flutter-unity-widgetのバージョンとUnityのバージョンの互換性を確認してください。また、Unityの設定でIL2CPPが正しく設定されているか確認します。

### 実行時の問題

**問題**: Unityのコンテンツが表示されない（黒い画面になる）

**解決策**:
1. Unityプロジェクトが正しくエクスポートされているか確認
2. ビルド設定が正しいか確認（特にGraphics APIの設定）
3. UnityWidgetが正しく初期化されているか確認

**問題**: FlutterとUnity間の通信が機能しない

**解決策**:
1. GameObject名とメソッド名が完全に一致しているか確認
2. コンソールログを確認して、メッセージが正しく送受信されているか確認

## まとめ

この記事では、Flutter初心者向けに、flutter-unity-widgetを使ってUnityコンテンツをFlutterアプリに統合する方法を解説しました。以下の内容を学びました：

- Unity側での準備とスクリプト作成
- Unityプロジェクトのエクスポート
- Flutterプロジェクトとの統合
- Flutter-Unity間の通信方法

flutter-unity-widgetを使うことで、FlutterアプリにUnity製の3Dコンテンツ、ゲーム、ARなどを埋め込むことができます。これにより、Flutter単体では実現が難しい機能も、比較的簡単に実装できるようになります。

ただし注意点として、Unityを統合するとアプリのサイズが大きくなることと、ビルド設定がやや複雑になることがあります。目的に応じて検討しましょう。

## 参考リンク

- [flutter-unity-widget GitHub](https://github.com/juicycleff/flutter-unity-view-widget)
- [Flutter公式サイト](https://flutter.dev/)
- [Unity公式サイト](https://unity.com/ja)
