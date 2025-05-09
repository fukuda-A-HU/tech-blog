---
title: "Flutter から Rust プラグインを読み出す方法"
emoji: "🦀"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Flutter", "Rust", "プラグイン", "FFI", "クロスプラットフォーム"]
published: false
---

## ゴール(はじめに)：FlutterアプリからRustの機能を利用する

Flutterでアプリを開発する際、高速な処理や既存のネイティブライブラリを活用したいケースがあります。Rustは高いパフォーマンスとメモリ安全性を両立したプログラミング言語であり、Flutterとの組み合わせは非常に魅力的です。

この記事では、FlutterアプリからRustで書かれたコードを呼び出す基本的な方法を解説します。シンプルな計算関数をRustで実装し、Flutterから使う例を通して、以下のことを学びます：

1. Flutter用のRustプラグインのプロジェクト構成
2. RustとDartの連携方法
3. FFI（Foreign Function Interface）を使ったクロス言語呼び出し
4. クロスプラットフォームでのビルド方法

## やってみた結果

この記事の手順を実行することで、次のことが実現できます：

- Rustで実装した関数をFlutterアプリから呼び出せる
- メモリ安全性を保ちながら高速な処理をRust側で行える
- iOS/Android両方のプラットフォームでRustのコードを共有できる
- Flutterの開発に慣れながら、RustとFFIの基本的な仕組みを理解できる

## 開発環境

- Flutter 3.10.0以上
- Android Studio または Visual Studio Code
- Rust（rustup経由でインストール）
- cargo-ndk（Android向けRustのビルド用）
- Android SDK（Android向け）
- Xcode 14以上（iOS向け）

フォルダ構造：
```
flutter_rust_plugin/
├── android/               # Android固有の設定
├── ios/                  # iOS固有の設定
├── lib/                  # Dartコード
│   └── flutter_rust_plugin.dart  # プラグインのDart API
├── native/               # Rustコード
│   ├── Cargo.toml        # Rust依存関係設定
│   └── src/              # Rustソースコード
└── example/              # サンプルFlutterアプリ
```

## 事前準備

### 1. Flutter SDKのインストール

まだFlutter SDKをインストールしていない場合は、[Flutter公式サイト](https://flutter.dev/docs/get-started/install)の手順に従ってインストールしてください。

### 2. Rustのインストール

Rustがインストールされていない場合は、[Rust公式サイト](https://www.rust-lang.org/tools/install)から rustup をインストールし、Rustの開発環境をセットアップします。

```bash
# Rustのインストール
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
# パスを適用
source $HOME/.cargo/env
# インストール確認
rustc --version
cargo --version
```

### 3. cargo-ndkのインストール

Android向けのRustコードをビルドするために、cargo-ndkをインストールします。

```bash
cargo install cargo-ndk
```

### 4. Flutterプロジェクトの作成

新しいFlutterプラグインプロジェクトを作成します。

```bash
flutter create --template=plugin flutter_rust_plugin
cd flutter_rust_plugin
```

## やったこと

### 1. Rustのプロジェクト作成

まず、Flutter プロジェクト内に Rust のコードを配置するディレクトリを作成し、新しい Rust プロジェクトを初期化します。

```bash
mkdir -p native/src
cd native
cargo init --lib
```

### 2. Rustで計算関数を実装

`native/src/lib.rs` ファイルを編集して、簡単な数学関数を実装します：

```rust
use std::panic;

#[no_mangle]
pub extern "C" fn add(a: i32, b: i32) -> i32 {
    a + b
}

#[no_mangle]
pub extern "C" fn subtract(a: i32, b: i32) -> i32 {
    a - b
}

#[no_mangle]
pub extern "C" fn multiply(a: i32, b: i32) -> i32 {
    a * b
}

#[no_mangle]
pub extern "C" fn divide(a: i32, b: i32) -> f64 {
    if b == 0 {
        return 0.0; // エラーケースは単純化のため0を返す
    }
    a as f64 / b as f64
}

// Rustパニックハンドラを設定
#[no_mangle]
pub extern "C" fn init_rust_runtime() {
    // パニック時のハンドラをセットアップ
    panic::set_hook(Box::new(|panic_info| {
        eprintln!("Rust panic occurred: {:?}", panic_info);
    }));
}
```

### 3. Cargo.toml の設定

Rustプロジェクトの設定ファイルである`native/Cargo.toml`を編集して、必要な設定を行います：

```toml
[package]
name = "flutter_rust_plugin"
version = "0.1.0"
edition = "2021"

[lib]
name = "flutter_rust_plugin"
# cdylib: C互換の動的ライブラリを生成
crate-type = ["cdylib", "staticlib"]

[dependencies]
# 必要に応じて依存関係を追加
```

### 4. Android向けビルドスクリプトの作成

Android向けにRustコードをビルドするためのスクリプトを作成します。`native/build-android.sh`というファイルを作成します：

```bash
#!/bin/bash
set -e  # エラー発生時に停止

# Android ABI のターゲットリスト
ANDROID_ABIS=("armeabi-v7a" "arm64-v8a" "x86" "x86_64")
# Rust ターゲットプラットフォームとの対応
RUST_TARGETS=("armv7-linux-androideabi" "aarch64-linux-android" "i686-linux-android" "x86_64-linux-android")

# Android NDKのパスを環境変数から取得またはデフォルト値を使用
ANDROID_NDK=${ANDROID_NDK_HOME:-"$HOME/Android/Sdk/ndk"}
echo "Using Android NDK at: $ANDROID_NDK"

if [ ! -d "$ANDROID_NDK" ]; then
    echo "Error: Android NDK not found at $ANDROID_NDK"
    echo "Please set ANDROID_NDK_HOME environment variable to your NDK path"
    exit 1
fi

mkdir -p ../android/src/main/jniLibs

for i in ${!ANDROID_ABIS[@]}; do
    ABI=${ANDROID_ABIS[$i]}
    TARGET=${RUST_TARGETS[$i]}
    
    echo "Building for $ABI ($TARGET)..."
    
    # Rustターゲットのインストール（もし存在しなければ）
    rustup target add $TARGET
    
    # cargo-ndkを使用してビルド
    cargo ndk --target $TARGET --platform 21 -- build --release
    
    # ビルドした.soファイルをAndroidプロジェクトにコピー
    mkdir -p ../android/src/main/jniLibs/$ABI
    cp ../target/$TARGET/release/libflutter_rust_plugin.so ../android/src/main/jniLibs/$ABI/
done

echo "Android build completed!"
```

実行権限を付与します：

```bash
chmod +x build-android.sh
```

### 5. iOS向けビルドスクリプトの作成

iOS向けにRustコードをビルドするためのスクリプトを作成します。`native/build-ios.sh`というファイルを作成します：

```bash
#!/bin/bash
set -e  # エラー発生時に停止

# iOS向けRustターゲットのリスト
IOS_TARGETS=("aarch64-apple-ios" "x86_64-apple-ios")

# 各ターゲット向けにビルド
for TARGET in ${IOS_TARGETS[@]}; do
    echo "Building for $TARGET..."
    rustup target add $TARGET
    cargo build --release --target=$TARGET
done

# ユニバーサルライブラリの作成
echo "Creating universal library..."
mkdir -p ../ios/Libs

# lipo コマンドを使用して、異なるアーキテクチャのライブラリを一つに統合
lipo -create \
    ../target/aarch64-apple-ios/release/libflutter_rust_plugin.a \
    ../target/x86_64-apple-ios/release/libflutter_rust_plugin.a \
    -output ../ios/Libs/libflutter_rust_plugin.a

echo "iOS build completed!"
```

実行権限を付与します：

```bash
chmod +x build-ios.sh
```

### 6. Dart側のFFIインターフェース

Flutterアプリから Rust の関数を呼び出すために、FFI インターフェースを実装します。`lib/flutter_rust_plugin.dart` ファイルを編集します：

```dart
import 'dart:async';
import 'dart:ffi';
import 'dart:io';

import 'package:flutter/foundation.dart';
import 'package:ffi/ffi.dart';

// Rustの関数を表すDart型定義
typedef AddFunc = Int32 Function(Int32 a, Int32 b);
typedef SubtractFunc = Int32 Function(Int32 a, Int32 b);
typedef MultiplyFunc = Int32 Function(Int32 a, Int32 b);
typedef DivideFunc = Double Function(Int32 a, Int32 b);
typedef InitRustRuntimeFunc = Void Function();

// Dartから呼び出すための型定義
typedef AddFuncDart = int Function(int a, int b);
typedef SubtractFuncDart = int Function(int a, int b);
typedef MultiplyFuncDart = int Function(int a, int b);
typedef DivideFuncDart = double Function(int a, int b);
typedef InitRustRuntimeFuncDart = void Function();

class FlutterRustPlugin {
  // ライブラリの読み込み
  static final DynamicLibrary _lib = _loadLibrary();
  
  // 各関数へのポインタを取得
  static final AddFuncDart _add = _lib
      .lookup<NativeFunction<AddFunc>>('add')
      .asFunction<AddFuncDart>();
  
  static final SubtractFuncDart _subtract = _lib
      .lookup<NativeFunction<SubtractFunc>>('subtract')
      .asFunction<SubtractFuncDart>();
  
  static final MultiplyFuncDart _multiply = _lib
      .lookup<NativeFunction<MultiplyFunc>>('multiply')
      .asFunction<MultiplyFuncDart>();
  
  static final DivideFuncDart _divide = _lib
      .lookup<NativeFunction<DivideFunc>>('divide')
      .asFunction<DivideFuncDart>();
      
  static final InitRustRuntimeFuncDart _initRustRuntime = _lib
      .lookup<NativeFunction<InitRustRuntimeFunc>>('init_rust_runtime')
      .asFunction<InitRustRuntimeFuncDart>();
  
  // プラグインの初期化
  static void initialize() {
    _initRustRuntime();
  }
  
  // Rustの関数を呼び出すラッパーメソッド
  static int add(int a, int b) {
    return _add(a, b);
  }
  
  static int subtract(int a, int b) {
    return _subtract(a, b);
  }
  
  static int multiply(int a, int b) {
    return _multiply(a, b);
  }
  
  static double divide(int a, int b) {
    return _divide(a, b);
  }
  
  // プラットフォームごとに適切なライブラリを読み込む
  static DynamicLibrary _loadLibrary() {
    if (Platform.isAndroid) {
      return DynamicLibrary.open("libflutter_rust_plugin.so");
    } else if (Platform.isIOS) {
      return DynamicLibrary.process();
    } else {
      throw UnsupportedError('Unsupported platform: ${Platform.operatingSystem}');
    }
  }
}
```

### 7. Android側の設定

Android側での連携を設定します。`android/build.gradle` ファイルを編集して、JNIライブラリをビルドプロセスに含めます：

```gradle
// ...既存のコード...

android {
    // ...既存の設定...

    // 以下を追加
    sourceSets {
        main {
            jniLibs.srcDirs = ['src/main/jniLibs']
        }
    }
}
```

### 8. iOS側の設定

iOS向けにビルドしたライブラリを含めるために、`ios/flutter_rust_plugin.podspec`ファイルを編集します：

```ruby
# ...既存のコード...

Pod::Spec.new do |s|
  s.name             = 'flutter_rust_plugin'
  s.version          = '0.0.1'
  s.summary          = 'A Flutter plugin with Rust implementation.'
  # ...既存の設定...
  
  # 以下を追加
  s.vendored_libraries = 'Libs/libflutter_rust_plugin.a'
  s.static_framework = true
  s.library = 'flutter_rust_plugin'
end
```

### 9. サンプルアプリでの使用

サンプルアプリを実装して、Rustプラグインの機能を試します。`example/lib/main.dart`ファイルを編集します：

```dart
import 'package:flutter/material.dart';
import 'package:flutter_rust_plugin/flutter_rust_plugin.dart';

void main() {
  // Rustランタイムの初期化
  FlutterRustPlugin.initialize();
  runApp(const MyApp());
}

class MyApp extends StatefulWidget {
  const MyApp({Key? key}) : super(key: key);

  @override
  State<MyApp> createState() => _MyAppState();
}

class _MyAppState extends State<MyApp> {
  final TextEditingController _aController = TextEditingController(text: "10");
  final TextEditingController _bController = TextEditingController(text: "5");
  String _result = "計算結果がここに表示されます";

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        appBar: AppBar(
          title: const Text('Rustプラグインの例'),
        ),
        body: Padding(
          padding: const EdgeInsets.all(16.0),
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.stretch,
            children: [
              TextField(
                controller: _aController,
                keyboardType: TextInputType.number,
                decoration: const InputDecoration(labelText: '値 A'),
              ),
              const SizedBox(height: 8),
              TextField(
                controller: _bController,
                keyboardType: TextInputType.number,
                decoration: const InputDecoration(labelText: '値 B'),
              ),
              const SizedBox(height: 16),
              Row(
                mainAxisAlignment: MainAxisAlignment.spaceEvenly,
                children: [
                  ElevatedButton(
                    onPressed: _add,
                    child: const Text('足し算'),
                  ),
                  ElevatedButton(
                    onPressed: _subtract,
                    child: const Text('引き算'),
                  ),
                  ElevatedButton(
                    onPressed: _multiply,
                    child: const Text('掛け算'),
                  ),
                  ElevatedButton(
                    onPressed: _divide,
                    child: const Text('割り算'),
                  ),
                ],
              ),
              const SizedBox(height: 24),
              Container(
                padding: const EdgeInsets.all(16),
                color: Colors.grey[200],
                child: Text(
                  _result,
                  style: const TextStyle(fontSize: 18),
                  textAlign: TextAlign.center,
                ),
              ),
            ],
          ),
        ),
      ),
    );
  }

  // 入力値を取得
  (int, int) _getInputValues() {
    int a = int.tryParse(_aController.text) ?? 0;
    int b = int.tryParse(_bController.text) ?? 0;
    return (a, b);
  }

  // 足し算の実行
  void _add() {
    try {
      final (a, b) = _getInputValues();
      final result = FlutterRustPlugin.add(a, b);
      setState(() {
        _result = "$a + $b = $result";
      });
    } catch (e) {
      setState(() {
        _result = "エラーが発生しました: $e";
      });
    }
  }

  // 引き算の実行
  void _subtract() {
    try {
      final (a, b) = _getInputValues();
      final result = FlutterRustPlugin.subtract(a, b);
      setState(() {
        _result = "$a - $b = $result";
      });
    } catch (e) {
      setState(() {
        _result = "エラーが発生しました: $e";
      });
    }
  }

  // 掛け算の実行
  void _multiply() {
    try {
      final (a, b) = _getInputValues();
      final result = FlutterRustPlugin.multiply(a, b);
      setState(() {
        _result = "$a × $b = $result";
      });
    } catch (e) {
      setState(() {
        _result = "エラーが発生しました: $e";
      });
    }
  }

  // 割り算の実行
  void _divide() {
    try {
      final (a, b) = _getInputValues();
      final result = FlutterRustPlugin.divide(a, b);
      setState(() {
        _result = "$a ÷ $b = $result";
      });
    } catch (e) {
      setState(() {
        _result = "エラーが発生しました: $e";
      });
    }
  }
}
```

### 10. ビルドと実行

1. まずRustコードをビルドします：

```bash
cd native
# Androidビルド
./build-android.sh
# iOSビルド（macOSの場合）
./build-ios.sh
cd ..
```

2. サンプルアプリを実行します：

```bash
cd example
flutter run
```

## 実装の動作確認

アプリが起動したら、2つの数値を入力し、各演算ボタンをタップすると、Rustの関数が呼び出され結果が表示されます。

## 結論に対しての補足

### RustとC++の比較

Rustは以下の点でC++と比較して優位性があります：

1. **メモリ安全性**: Rustはコンパイル時にメモリエラーを検出します
2. **並行処理の安全性**: データ競合の問題を型システムで防ぎます
3. **モダンなパッケージ管理**: Cargoによる依存関係の管理が簡単です

ただし、学習曲線がやや急で、既存のC++ライブラリとの連携には追加の作業が必要な場合があります。

### クロスプラットフォームビルドの注意点

1. **Android NDKバージョン**: 使用するNDKのバージョンによって設定が異なる場合があります
2. **iOS向けビットコード**: App Storeの要件に応じてbitcodeを有効にする場合があります
3. **デバッグ方法**: ネイティブコードのデバッグには専用のツールが必要です

### FFIの代替手段

今回はDart FFIを使いましたが、他にも以下の方法があります：

1. **flutter_rust_bridge**: Rust-Dart間の型安全な橋渡しを自動生成するツール
2. **メソッドチャネル**: 従来のFlutterプラグインの仕組み
3. **uniffi-rs**: 複数言語向けFFIラッパーを生成するRustツール

プロジェクトの規模や要件に応じて、最適な方法を選ぶと良いでしょう。

## おわりに

この記事では、FlutterアプリからRustの関数を呼び出す基本的な方法を解説しました。単純な数値計算の例でしたが、この仕組みを応用すれば、画像処理や機械学習、暗号化処理などの高速な処理をRustで実装し、美しいFlutterのUIと組み合わせることができます。

Rustの安全性と高速性、Flutterのクロスプラットフォーム性という両方の利点を活かすことで、高品質なモバイルアプリケーションの開発が可能になります。初期設定は少し複雑ですが、一度テンプレートを作成しておけば、今後のプロジェクトでも再利用可能です。

Rust × Flutterの組み合わせは、特にパフォーマンスクリティカルなアプリケーションや、セキュリティが重要なアプリケーションにおいて、大きな価値を発揮するでしょう。ぜひこの方法を試して、モダンなクロス言語プログラミングを体験してみてください。
