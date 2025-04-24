---
title: "Zenn CLIで作る空のFlutterプロジェクトをAndroidシミュレーターで起動してみる"
emoji: "📱"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Flutter", "Android", "Dart", "モバイル開発", "Zenn"]
published: false
---

## ゴール：FlutterプロジェクトをAndroidシミュレーターで動かす

この記事では、Flutterの開発環境を整え、空のFlutterプロジェクトを作成し、Androidシミュレーターで実行するまでの手順を説明します。モバイルアプリ開発を始めたい方や、Flutterの環境構築で躓いている方に向けて、できるだけ分かりやすく解説していきます。

## やってみた結果

この記事の手順に従うことで、以下のことができるようになります：
- Flutter SDKをセットアップする
- 新しいFlutterプロジェクトを作成する
- Androidシミュレーターを設定・起動する
- 作成したFlutterアプリをシミュレーター上で実行する

## 開発環境

- OS: macOS / Windows 10/11
- Flutter: 3.16.0
- Android Studio: 2023.1.1
- Zenn CLI: 0.1.150

## 事前準備

Flutterアプリの開発とAndroidシミュレーターでの実行には、いくつかのツールをインストールする必要があります。以下の手順で準備していきましょう。

### 1. Flutter SDKのインストール

#### macOS環境の場合

macOSにはHomebrewを使用してFlutterをインストールする方法が便利です。

```bash
# Homebrewがインストールされていない場合はインストール
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Homebrewを使ってFlutterをインストール
brew install --cask flutter

# もしくは手動でダウンロードしてインストールする場合
cd ~/development
curl -O https://storage.googleapis.com/flutter_infra_release/releases/stable/macos/flutter_macos_3.16.0-stable.zip
unzip flutter_macos_3.16.0-stable.zip

# PATHを通す（.zshrcや.bashrcなどに追加）
export PATH="$PATH:~/development/flutter/bin"
```

#### Windows環境の場合

1. [Flutter公式サイト](https://flutter.dev/docs/get-started/install/windows)からFlutter SDKをダウンロードします
2. ダウンロードしたzipファイルを解凍し、任意のフォルダ（例：`C:\flutter`）に配置します
3. 環境変数のPathにFlutterのbinディレクトリを追加します
   - スタートメニューで「環境変数」と検索
   - 「システム環境変数の編集」を選択
   - 「環境変数」ボタンをクリック
   - 「PATH」変数を選択して「編集」をクリック
   - 「新規」をクリックし、`C:\flutter\bin`を追加

コマンドプロンプトかPowerShellから以下のコマンドを実行して確認します：

```powershell
flutter --version
```

以下のような出力が表示されれば成功です。

```
Flutter 3.16.0 • channel stable • https://github.com/flutter/flutter.git
Framework • revision db7ef5bf9f (5 days ago) • 2023-11-15 11:25:44 -0800
Engine • revision 74d16627b9
Tools • Dart 3.2.0 • DevTools 2.28.0
```

### 2. Android Studioのインストール

#### macOS環境の場合

```bash
# Homebrewを使う場合
brew install --cask android-studio

# もしくは公式サイトからダウンロードしてインストール
# https://developer.android.com/studio
```

#### Windows環境の場合

1. [Android Studio公式サイト](https://developer.android.com/studio)からインストーラーをダウンロード
2. インストーラーを実行し、画面の指示に従ってインストール
3. インストール完了後、Android Studioを起動

### 3. Android SDKの設定

どのOS環境でも、Android Studioの設定画面から、「SDK Manager」を開き、以下のコンポーネントがインストールされていることを確認します：

- Android SDK Platform（最新版）
- Android SDK Command-line Tools
- Android Emulator
- Android SDK Build-Tools

#### Windows特有の設定

Windowsの場合、Android SDKのパスをシステム環境変数に追加することをお勧めします：
1. システム環境変数に `ANDROID_HOME` を追加し、値を Android SDK の場所に設定します（通常は `C:\Users\[ユーザー名]\AppData\Local\Android\Sdk`）
2. `PATH` 環境変数に以下を追加：
   - `%ANDROID_HOME%\tools`
   - `%ANDROID_HOME%\platform-tools`

### 4. Flutterの設定とライセンスの同意

どのOS環境でも以下のコマンドを実行して、必要なライセンスに同意し、Flutterの診断を実行します。

```bash
flutter doctor --android-licenses
flutter doctor
```

#### macOS特有の設定

macOSでは、iOSシミュレーターを使用する場合は追加の設定が必要です：

```bash
# Xcodeのインストール（App Storeから）
# コマンドラインツールのインストール
sudo xcode-select --install

# Xcodeのライセンス同意
sudo xcodebuild -license accept
```

#### Windows特有の設定

Windowsでは、開発環境が正しく設定されているか確認するために、管理者権限でコマンドプロンプトを開いて `flutter doctor` を実行することをお勧めします。

## Flutterプロジェクトの作成

環境の準備ができたら、新しいFlutterプロジェクトを作成します。このステップはどのOS環境でも同じです。

```bash
# 新しいFlutterプロジェクトを作成
flutter create my_flutter_app

# プロジェクトディレクトリに移動
cd my_flutter_app
```

プロジェクトが作成されると、以下のようなディレクトリ構造が生成されます：

```
my_flutter_app/
  ├── android/       # Androidプラットフォーム固有のコード
  ├── ios/           # iOSプラットフォーム固有のコード
  ├── lib/           # Dartコード（主な開発領域）
  ├── test/          # テストコード
  ├── pubspec.yaml   # プロジェクトの依存関係を定義
  └── ...
```

### プロジェクトの中身を確認

作成したプロジェクトのソースコードを確認してみましょう。主要なファイルは`lib/main.dart`で、アプリのエントリーポイントとなります。

```dart
// lib/main.dart の内容
import 'package:flutter/material.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter Demo',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.deepPurple),
        useMaterial3: true,
      ),
      home: const MyHomePage(title: 'Flutter Demo Home Page'),
    );
  }
}

class MyHomePage extends StatefulWidget {
  const MyHomePage({super.key, required this.title});
  final String title;

  @override
  State<MyHomePage> createState() => _MyHomePageState();
}

// 以下省略...
```

これはFlutterのデフォルトのサンプルアプリで、カウンターアプリが実装されています。

## Androidシミュレーターの設定と起動

Flutterプロジェクトを実行するために、Androidシミュレーターを設定します。

### 1. Android Virtual Device (AVD) の作成

#### 全OS共通の手順

Android Studioを起動し、「AVD Manager」を開きます。これは、「Tools > AVD Manager」または右上のツールバーのAVDアイコンからアクセスできます。

1. 「Create Virtual Device」をクリックします
2. デバイス定義を選択します（例：「Pixel 4」）
3. システムイメージを選択します（例：「API 30」）
4. 「Next」をクリックし、必要に応じてAVDの設定をカスタマイズします
5. 「Finish」をクリックしてAVDを作成します

### 2. シミュレーターの起動

#### macOS環境の場合

コマンドラインから起動する場合：

```bash
# 利用可能なデバイスを一覧表示
flutter emulators

# 特定のエミュレータを起動（名前はflutter emulatorsの出力から）
flutter emulators --launch <emulator_id>
```

#### Windows環境の場合

コマンドプロンプトやPowerShellから起動する場合：

```powershell
# 利用可能なデバイスを一覧表示
flutter emulators

# 特定のエミュレータを起動
flutter emulators --launch <emulator_id>
```

また、Windows環境ではパフォーマンスを向上させるために、BIOSでHardware Virtualization Technology（Intel VT-x/AMD-V）を有効にしていることを確認すると良いでしょう。

#### macOS特有の注意点

macOSユーザーは、Android以外にもiOS用の開発を行う場合は、iOSシミュレーターを起動することもできます：

```bash
# iOSシミュレーターを起動
open -a Simulator
```

## Flutterアプリの実行

エミュレーターが起動したら、Flutterアプリを実行できます。このステップはどのOS環境でも同じです。

```bash
# プロジェクトディレクトリ内で実行
flutter run
```

### OS別のトラブルシューティング

#### macOS特有の問題

- **XcodeとCocoaPodsのバージョン互換性**: iOSビルドでエラーが発生する場合は、CocoaPodsを最新バージョンに更新してみてください。
  ```bash
  sudo gem install cocoapods
  ```

#### Windows特有の問題

- **パス長の制限**: Windowsにはファイルパスの長さに制限があります。ネストされた深いディレクトリでプロジェクトを作成すると問題が発生する場合があります。
- **アンチウイルスソフトウェア**: ビルド中にファイルアクセスが遅くなる場合は、アンチウイルスソフトウェアの除外リストにプロジェクトフォルダを追加してみてください。

## アプリの基本的な変更

`lib/main.dart`を開き、以下のように色やテキストを変更してみます：

```dart
// ColorSchemeの色を変更
theme: ThemeData(
  colorScheme: ColorScheme.fromSeed(seedColor: Colors.green),
  useMaterial3: true,
),

// ホームページのタイトルを変更
home: const MyHomePage(title: 'My First Flutter App'),
```

変更を保存すると、実行中のアプリに変更がホットリロードされ、即座に反映されます。ターミナルで「r」を押すことで手動でホットリロードを実行することもできます。

## プラットフォーム別の特徴と注意点

### macOSでの開発

macOSでは、iOSとAndroid両方の開発環境を構築できるため、両プラットフォーム向けのテストが容易です。

- **iOSシミュレーター**: Android以外にiOSシミュレーターも使用可能
- **パフォーマンス**: 一般的に仮想化のパフォーマンスが良好
- **M1/M2チップ対応**: Rosetta 2を使って、Intel向けの開発ツールも利用可能

### Windowsでの開発

- **ハードウェア加速**: Android Emulatorのパフォーマンスを向上させるために、HAXM（Intel Hardware Accelerated Execution Manager）のインストールを推奨
- **WSL対応**: Windows Subsystem for Linux (WSL) を使用することで、Linux環境での開発も可能
- **iOS開発の制限**: Windowsでは直接iOSアプリのビルドはできないため、iOS向けの開発にはmacOSが必要

## おわりに

この記事では、Flutter開発環境のセットアップから、新しいFlutterプロジェクトの作成、Androidシミュレーターでのアプリ実行までの手順を、macOSとWindowsの各環境に対応して説明しました。

それぞれのOS環境には特有の設定やトラブルシューティングのポイントがありますが、Flutterの基本的な開発フローは共通しています。環境構築さえできれば、あとは直感的なUI設計と高速な開発サイクルを活かして、クロスプラットフォームのアプリ開発を進めることができます。

次のステップとしては、Flutterの基本的なウィジェットやレイアウト、状態管理などについて学ぶことをお勧めします。公式ドキュメントやチュートリアルが充実しているので、それらを参考にして学習を進めてみてください。

## 参考リンク

- [Flutter公式ドキュメント](https://flutter.dev/docs)
- [Flutterインストールガイド（macOS）](https://flutter.dev/docs/get-started/install/macos)
- [Flutterインストールガイド（Windows）](https://flutter.dev/docs/get-started/install/windows)
- [Dart言語ツアー](https://dart.dev/guides/language/language-tour)
- [Flutter Widget カタログ](https://flutter.dev/docs/development/ui/widgets)
