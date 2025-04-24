---
title: "Flutter初心者のためのUnity AR連携ガイド - ARFoundationをFlutterアプリに埋め込む方法"
emoji: "🔮"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["flutter", "unity", "ar", "arfoundation", "mobile"]
published: false
---

# Flutter初心者のためのUnity AR連携ガイド - ARFoundationをFlutterアプリに埋め込む方法

こんにちは！この記事では、Flutter初心者向けに、Flutter-Unity-Widgetを使ってUnityで作成したARFoundationのプロジェクトをFlutterアプリに統合する方法を詳しく解説します。Unityの強力なAR機能とFlutterの使いやすいUI構築を組み合わせて、魅力的なARアプリを作成しましょう！

## この記事で学べること

- Flutter-Unity-Widgetの基本的な使い方
- UnityのARFoundationプロジェクトの準備方法
- FlutterとUnityの連携における重要な注意点
- AR機能をFlutterアプリに統合するための手順
- トラブルシューティングとよくある問題の解決方法

## 前提条件

この記事は以下のツールを既にインストールしていることを前提としています：

- Flutter SDK (最新版)
- Android Studio または Visual Studio Code
- Unity Hub と Unity 2020.3 LTS 以降
- iOS開発の場合は Xcode（macOS環境）

まだインストールしていない場合は、各ツールの公式サイトからダウンロードしてインストールしてください。

## 1. Flutter-Unity-Widgetについて

[Flutter-Unity-Widget](https://github.com/juicycleff/flutter-unity-view-widget)は、FlutterアプリにUnityのコンテンツをシームレスに統合するためのプラグインです。このプラグインを使えば、UnityのリッチなゲームやARなどのインタラクティブコンテンツをFlutterアプリ内に埋め込むことができます。

### 主な特徴

- Unityビューとのコミュニケーション
- Unityコンテンツのフルスクリーン表示とサイズ調整
- AR/VRコンテンツのサポート
- Android、iOS両プラットフォームへの対応

## 2. プロジェクトの準備

### 2.1 FlutterプロジェクトとUnityプロジェクトの作成

まず、Flutterプロジェクトと、ARFoundationを使用したUnityプロジェクトをそれぞれ作成する必要があります。

#### Flutterプロジェクトの作成

```bash
flutter create flutter_unity_ar_app
cd flutter_unity_ar_app
```

#### Unityプロジェクトの作成

1. Unity Hubを開き、「新規プロジェクトの作成」をクリックします
2. 3Dテンプレートを選択します
3. プロジェクト名を「UnityARProject」とし、場所を選択して「作成」をクリックします

### 2.2 Unityプロジェクトの設定

ARFoundationを追加するには、Unity Package Managerを使用します：

1. Unity Editorの「Window」>「Package Manager」を開きます
2. 「+」ボタンをクリックし、「Add package from git URL...」を選択します
3. 以下のパッケージを順番に追加します：
   - `com.unity.xr.arfoundation`
   - `com.unity.xr.arcore` (Android用)
   - `com.unity.xr.arkit` (iOS用)

これらのパッケージがインストールされたら、ARFoundationのセットアップを行います：

1. 「Edit」>「Project Settings」>「XR Plug-in Management」を開きます
2. 「Install XR Plugin Management」をクリックします
3. Androidタブを選択し、「ARCore」にチェックを入れます
4. iOSタブを選択し、「ARKit」にチェックを入れます

### 2.3 シンプルなARサンプルシーンの作成

ARFoundationを使った簡単な平面検出サンプルを作成しましょう：

1. 新しいシーンを作成し、「ARScene」という名前で保存します
2. Hierarchyウィンドウを右クリックし、「XR」>「AR Session Origin」を選択します
3. 同様に「XR」>「AR Session」も追加します
4. AR Session Originを選択し、Inspectorで「AR Session Origin」コンポーネントを確認します
5. AR Session Originの下にある「AR Camera」がメインカメラになります
6. AR Session Originを選択し、「Component」>「XR」>「AR Plane Manager」を追加します
7. 「Component」>「XR」>「AR Raycast Manager」も追加します

平面を視覚化するために：

1. Projectウィンドウで「Create」>「Material」を選択し、「PlaneMaterial」という名前で作成します
2. このマテリアルの色を青系に設定し、Alphaを0.5程度に設定して半透明にします
3. AR Plane ManagerのInspectorで、「Plane Prefab」に新しく平面用のPrefabを作成してアサインします

## 3. Unity側の準備

### 3.1 Unity向けのFlutter連携スクリプト

Unity側でFlutterとの通信を行うためのスクリプトを作成しましょう。まず、新しいC#スクリプトを作成し、「FlutterUnityInterface.cs」という名前を付けます：

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.Events;
using System.Runtime.InteropServices;
using UnityEngine.XR.ARFoundation;
using UnityEngine.XR.ARSubsystems;

public class FlutterUnityInterface : MonoBehaviour
{
    // ARセッション
    [SerializeField] ARSession session;
    // 平面検出マネージャー
    [SerializeField] ARPlaneManager planeManager;
    // レイキャストマネージャー
    [SerializeField] ARRaycastManager raycastManager;
    
    // オブジェクト配置用のプレハブ
    [SerializeField] GameObject objectPrefab;
    
    // 検出された平面の情報
    [HideInInspector]
    public string detectedPlanes = "0";
    
    // 配置されたオブジェクトのリスト
    private List<GameObject> placedObjects = new List<GameObject>();
    
    void Start()
    {
        if (planeManager != null)
        {
            planeManager.planesChanged += OnPlanesChanged;
        }
    }
    
    // 平面検出時のイベント
    private void OnPlanesChanged(ARPlanesChangedEventArgs args)
    {
        detectedPlanes = planeManager.trackables.count.ToString();
        // Flutterに検出した平面の数を通知
        SendMessageToFlutter("OnPlanesChanged", detectedPlanes);
    }
    
    // FlutterからのメッセージをUnityで処理するメソッド
    public void HandleMessageFromFlutter(string message)
    {
        if (message == "RESET_AR_SESSION")
        {
            StartCoroutine(ResetARSession());
        }
        else if (message.StartsWith("PLACE_OBJECT_AT:"))
        {
            // 座標情報を解析してオブジェクトを配置
            string[] coords = message.Replace("PLACE_OBJECT_AT:", "").Split(',');
            if (coords.Length >= 2)
            {
                float x = float.Parse(coords[0]);
                float y = float.Parse(coords[1]);
                PlaceObjectAtPosition(new Vector2(x, y));
            }
        }
    }
    
    // ARセッションをリセットする
    private IEnumerator ResetARSession()
    {
        if (session != null)
        {
            session.Reset();
        }
        
        // 配置済みのオブジェクトをクリア
        foreach (var obj in placedObjects)
        {
            Destroy(obj);
        }
        placedObjects.Clear();
        
        yield return null;
    }
    
    // 指定した画面位置にオブジェクトを配置
    private void PlaceObjectAtPosition(Vector2 screenPosition)
    {
        List<ARRaycastHit> hits = new List<ARRaycastHit>();
        if (raycastManager.Raycast(screenPosition, hits, TrackableType.PlaneWithinPolygon))
        {
            Pose hitPose = hits[0].pose;
            GameObject newObject = Instantiate(objectPrefab, hitPose.position, hitPose.rotation);
            placedObjects.Add(newObject);
            
            // Flutterにオブジェクト配置成功を通知
            SendMessageToFlutter("OnObjectPlaced", 
                                hitPose.position.x + "," + 
                                hitPose.position.y + "," + 
                                hitPose.position.z);
        }
    }
    
    // UnityからFlutterへメッセージを送信する
    private void SendMessageToFlutter(string methodName, string message)
    {
        #if UNITY_ANDROID && !UNITY_EDITOR
            using (AndroidJavaClass unityPlayer = new AndroidJavaClass("com.unity3d.player.UnityPlayer"))
            {
                AndroidJavaObject activity = unityPlayer.GetStatic<AndroidJavaObject>("currentActivity");
                activity.Call("SendMessageToFlutter", methodName, message);
            }
        #elif UNITY_IOS && !UNITY_EDITOR
            // iOSの場合はObjective-Cブリッジを通じて呼び出す（必要に応じて実装）
            NativeAPI.SendMessageToFlutter(methodName, message);
        #endif
        
        // エディタでのデバッグ用
        Debug.Log($"SendMessageToFlutter: {methodName} - {message}");
    }
}

#if UNITY_IOS && !UNITY_EDITOR
// iOSネイティブブリッジ
public class NativeAPI
{
    [DllImport("__Internal")]
    public static extern void SendMessageToFlutter(string methodName, string message);
}
#endif
```

### 3.2 ARシーンの構成

1. 作成したFlutterUnityInterfaceスクリプトを空のゲームオブジェクトにアタッチします
2. Inspectorで以下のように参照を設定します：
   - Session: シーンのARSessionをドラッグ＆ドロップ
   - Plane Manager: ARPlaneManagerをドラッグ＆ドロップ
   - Raycast Manager: ARRaycastManagerをドラッグ＆ドロップ
   - Object Prefab: 配置したいオブジェクトのPrefab（例えば単純な立方体）をドラッグ＆ドロップ

### 3.3 Unityプロジェクトのエクスポート準備

Flutterと連携するためには、Unityプロジェクトを特定の方法でビルドする必要があります：

1. 「File」>「Build Settings」を開きます
2. プラットフォームをAndroidまたはiOSに切り替えます（まだの場合）
3. 「Player Settings」をクリックし、以下の設定を行います：

#### Android設定：
- 「Other Settings」>「Rendering」>「Auto Graphics API」のチェックを外し、OpenGLESを優先順位の一番上に設定します
- 「Other Settings」>「Identification」>「Minimum API Level」を24以上に設定します
- 「Other Settings」>「Configuration」>「Scripting Backend」をIL2CPPに設定します
- 「Publishing Settings」>「Build」で「Custom Base Gradle Template」にチェックを入れます

#### iOS設定：
- 「Other Settings」>「Configuration」>「Scripting Backend」をIL2CPPに設定します
- 「Other Settings」>「Identification」>「Target minimum iOS Version」を11.0以上に設定します

### 3.4 Unityプロジェクトのエクスポート

Android向けの設定：

1. 「File」>「Build Settings」でAndroidプラットフォームを選択します
2. 「Switch Platform」をクリックします（まだの場合）
3. 「Export Project」にチェックを入れます
4. 「Build」ボタンをクリックし、エクスポート先として「unity_ar_export」フォルダを選択します

iOS向けの設定：

1. 「File」>「Build Settings」でiOSプラットフォームを選択します
2. 「Switch Platform」をクリックします（まだの場合）
3. 「Export Project」にチェックを入れます
4. 「Build」ボタンをクリックし、エクスポート先として「unity_ar_export_ios」フォルダを選択します

## 4. Flutterプロジェクトの設定

### 4.1 依存関係の追加

プロジェクトのpubspec.yamlファイルに、flutter_unity_widgetプラグインを追加します：

```yaml
dependencies:
  flutter:
    sdk: flutter
  cupertino_icons: ^1.0.2
  
  # Unity連携用プラグイン
  flutter_unity_widget: ^2022.2.0  # 常に最新バージョンを確認してください
```

そして、依存関係を更新します：

```bash
flutter pub get
```

### 4.2 UnityエクスポートプロジェクトのFlutterプロジェクトへのリンク

#### Androidの場合：

1. 「android/settings.gradle」ファイルを開きます
2. 以下のような記述を追加します：

```gradle
include ':unityExport'
project(':unityExport').projectDir = file('../unity_ar_export/unity_ar_export')
```

3. 「android/app/build.gradle」ファイルを開き、dependenciesセクションに以下を追加します：

```gradle
dependencies {
    implementation project(':unityExport')
    // 他の依存関係...
}
```

#### iOSの場合：

1. Xcodeでプロジェクトを開きます（Flutter側で `open ios/Runner.xcworkspace` を実行）
2. Unity iOSプロジェクトとFlutterプロジェクトを手動で統合する必要があります
   （詳細な手順はflutter-unity-widgetのドキュメントを参照してください）

### 4.3 Flutter側でUnityウィジェットを使用する

以下のように、Flutterアプリ内でUnityウィジェットを使用したメインアプリを実装します：

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
      title: 'Flutter Unity AR Demo',
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      home: const UnityARView(title: 'Flutter Unity AR 連携'),
    );
  }
}

class UnityARView extends StatefulWidget {
  const UnityARView({Key? key, required this.title}) : super(key: key);
  final String title;

  @override
  _UnityARViewState createState() => _UnityARViewState();
}

class _UnityARViewState extends State<UnityARView> {
  // UnityWidgetのコントローラー
  UnityWidgetController? _unityWidgetController;
  
  // 検出された平面の数
  int _detectedPlanes = 0;
  
  // メッセージログ
  List<String> _messages = [];
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text(widget.title),
      ),
      body: Column(
        children: [
          // Unity View（アプリの多くの部分を占める）
          Expanded(
            flex: 3,
            child: Container(
              color: Colors.black,
              child: UnityWidget(
                onUnityCreated: _onUnityCreated,
                onUnityMessage: _onUnityMessage,
                onUnitySceneLoaded: _onUnitySceneLoaded,
                fullscreen: false,
              ),
            ),
          ),
          
          // UIコントロール部分
          Expanded(
            flex: 1,
            child: Container(
              padding: const EdgeInsets.all(8.0),
              color: Colors.grey[200],
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.stretch,
                children: [
                  // ステータス情報
                  Text(
                    '検出された平面: $_detectedPlanes',
                    style: TextStyle(fontWeight: FontWeight.bold),
                  ),
                  const SizedBox(height: 8),
                  
                  // 操作ボタン
                  Row(
                    mainAxisAlignment: MainAxisAlignment.spaceEvenly,
                    children: [
                      ElevatedButton(
                        onPressed: _resetARSession,
                        child: const Text('ARセッションリセット'),
                      ),
                      ElevatedButton(
                        onPressed: _placeObject,
                        child: const Text('オブジェクトを配置'),
                      ),
                    ],
                  ),
                  
                  // メッセージログ（最新のものだけ表示）
                  Expanded(
                    child: Container(
                      margin: const EdgeInsets.only(top: 8),
                      padding: const EdgeInsets.all(4),
                      color: Colors.black12,
                      child: _messages.isNotEmpty
                          ? Text(_messages.last)
                          : const Text('ログメッセージがここに表示されます'),
                    ),
                  ),
                ],
              ),
            ),
          ),
        ],
      ),
    );
  }

  // Unityビューが作成されたときの処理
  void _onUnityCreated(UnityWidgetController controller) {
    _unityWidgetController = controller;
    _addMessage('Unity View が作成されました');
  }

  // Unityからメッセージを受け取ったときの処理
  void _onUnityMessage(dynamic message) {
    print('Unity message: $message');
    
    // メッセージの処理
    if (message.toString().startsWith('OnPlanesChanged:')) {
      // 平面検出情報の更新
      final planes = message.toString().split(':')[1];
      setState(() {
        _detectedPlanes = int.tryParse(planes) ?? 0;
        _addMessage('平面が検出されました: $_detectedPlanes');
      });
    } else if (message.toString().startsWith('OnObjectPlaced:')) {
      // オブジェクト配置情報
      final position = message.toString().split(':')[1];
      _addMessage('オブジェクトを配置しました: $position');
    } else {
      _addMessage('メッセージ: $message');
    }
  }

  // Unityシーンがロードされたときの処理
  void _onUnitySceneLoaded(SceneLoaded scene) {
    print('Unity scene loaded: ${scene.name}');
    _addMessage('シーンがロードされました: ${scene.name}');
  }

  // ARセッションをリセット
  void _resetARSession() {
    _unityWidgetController?.postMessage(
      'FlutterUnityInterface',
      'HandleMessageFromFlutter',
      'RESET_AR_SESSION',
    );
    _addMessage('ARセッションのリセットをリクエストしました');
  }

  // オブジェクトを配置
  void _placeObject() {
    // 画面中央に配置する場合
    final screenWidth = MediaQuery.of(context).size.width;
    final screenHeight = MediaQuery.of(context).size.height * 0.75; // Unity Viewの大体の高さ
    
    _unityWidgetController?.postMessage(
      'FlutterUnityInterface',
      'HandleMessageFromFlutter',
      'PLACE_OBJECT_AT:${screenWidth/2},${screenHeight/2}',
    );
    _addMessage('オブジェクト配置をリクエストしました');
  }
  
  // メッセージログに追加
  void _addMessage(String message) {
    setState(() {
      _messages.add('[${DateTime.now().toIso8601String().substring(11, 19)}] $message');
      // 最大10件までに制限
      if (_messages.length > 10) {
        _messages.removeAt(0);
      }
    });
  }
}
```

## 5. 注意点と落とし穴

Flutter-Unity-WidgetでARプロジェクトを統合する際には、いくつかの注意点があります：

### 5.1 パフォーマンスの問題

- UnityのARシーンはかなりのリソースを消費するため、アプリ全体のパフォーマンスに影響を与える可能性があります
- バックグラウンドでUnityを実行し続けると、バッテリーの消費が速くなります
- 必要ないときはUnityViewを破棄することを検討してください

### 5.2 ビルドサイズの増加

- UnityをFlutterアプリに組み込むと、アプリのサイズが大幅に増加します（通常50MB〜100MB以上）
- Google PlayやApp Storeでアプリをリリースする際は、App Bundleを使用してサイズを最適化することをお勧めします

### 5.3 プラットフォーム固有の問題

#### Android：

- 一部のAndroidデバイスではOpenGL ESの問題が発生することがあります
- ARCoreは全てのAndroidデバイスでサポートされているわけではないため、対応デバイスの確認が必要です

#### iOS：

- iOSでの連携は少し複雑で、手動設定が必要な箇所があります
- ARKitはiOS 11以降のデバイスでのみ動作します
- ビットコードを有効にする場合、ビルド時間が大幅に増加します

### 5.4 デバッグの難しさ

- UnityとFlutterの間でのデバッグは複雑になります
- 問題が発生した場合、Unityの問題なのかFlutterの問題なのかを切り分けることが難しい場合があります
- ログを適切に設定し、Unityとの通信を詳細に記録することをお勧めします

## 6. トラブルシューティング

### 6.1 ビルドエラー

**問題**: Androidビルド時に「Manifest merger failed」エラーが発生する

**解決策**: android/app/src/main/AndroidManifest.xmlファイルで、tools:replaceやtools:removeを使って競合を解決します：

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    package="com.example.flutter_unity_ar_app">
    
    <application
        android:label="flutter_unity_ar_app"
        android:icon="@mipmap/ic_launcher"
        tools:replace="android:label">
        <!-- 他の設定 -->
    </application>
</manifest>
```

**問題**: iOSのXcodeでコンパイルエラーが発生する

**解決策**: Xcodeプロジェクトの設定から「Enable Bitcode」をNoに設定します。また、Podsのスキームで「Build Configuration」をReleaseからDebugに変更してみてください。

### 6.2 実行時エラー

**問題**: UnityのARシーンがロードされない、または黒画面になる

**解決策**:
1. AndroidデバイスがARCoreをサポートしているか確認します
2. アプリにカメラ権限が付与されているか確認します
3. Unityのビルド設定が正しいか確認します（特にGraphics APIの設定）

**問題**: ARセッションが開始されない

**解決策**:
1. デバイスが十分な光環境にあるか確認します
2. ARKit/ARCoreのバージョンの互換性を確認します
3. デバイスのARコア/ARKitアプリを最新バージョンに更新します

## 7. まとめと発展

この記事では、Flutter-Unity-Widgetを使用して、Unityで作成したARFoundationプロジェクトをFlutterアプリに統合する方法を解説しました。これにより、Flutterの豊富なUIコンポーネントと、UnityのパワフルなAR機能を組み合わせたアプリケーションを開発することができます。

発展として、以下のような機能を追加してみるとよいでしょう：

1. 複数のARオブジェクトを配置し、Flutterから個別に操作する機能
2. AR体験の写真や動画を撮影する機能
3. オブジェクト認識を追加し、特定のオブジェクトを認識したときにFlutterUI側に通知する機能
4. クラウドアンカーを活用した複数ユーザーでの共有AR体験

Flutter初心者でも、この記事のステップを丁寧に追いながら、魅力的なARアプリケーションを作成できるはずです。ぜひ挑戦してみてください！

## 参考リンク

- [Flutter-Unity-Widget GitHub](https://github.com/juicycleff/flutter-unity-view-widget)
- [Unity ARFoundation ドキュメント](https://docs.unity3d.com/Packages/com.unity.xr.arfoundation@4.2/manual/index.html)
- [Flutter 公式サイト](https://flutter.dev/)
- [ARCore サポートデバイス](https://developers.google.com/ar/devices)
- [ARKit サポートデバイス](https://www.apple.com/jp/augmented-reality/)