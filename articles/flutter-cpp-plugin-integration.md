---
title: "Flutter から C++ プラグインを読み出す方法"
emoji: "🔌"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Flutter", "C++", "プラグイン", "FFI", "クロスプラットフォーム"]
published: false
---

## ゴール(はじめに)：FlutterアプリからC++の機能を利用する

FlutterはDartで書かれたクロスプラットフォームのUIフレームワークですが、時には高速な処理や既存のネイティブライブラリを利用したいケースがあります。特に計算処理が多い場合や、C/C++で書かれた既存ライブラリを再利用したい場合には、C++とFlutterを連携させる方法が役立ちます。

この記事では、FlutterアプリからC++のコードを呼び出す基本的な方法を解説します。シンプルな数値計算を行うC++の関数をFlutterから使う例を通して、以下のことを学びます：

1. Flutter用のC++プラグインのプロジェクト構成
2. C++とDartの橋渡しとなるコードの書き方
3. メソッドチャネルを使ったプラットフォーム間通信
4. Dartから簡単にC++の関数を呼び出す方法

## やってみた結果

この記事の手順を実行することで、以下のことが実現できます：

- C++で実装した単純な計算関数をFlutterアプリから呼び出せる
- 複雑な計算処理をC++側で行いパフォーマンスを向上させられる
- iOS/Android両方のプラットフォームでC++のコードを共有できる
- Flutterの開発に慣れながら、C++プラグインの基本的な仕組みを理解できる

Flutterの魅力はクロスプラットフォーム開発ですが、C++との連携でその可能性がさらに広がります。

## 開発環境

- Flutter 3.10.0以上
- Android Studio または Visual Studio Code
- Android SDK（Android向け）
- Xcode 14以上（iOS向け）
- CMake 3.10以上（C++のビルド用）
- 必要に応じてNDK（Android向けネイティブ開発キット）

フォルダ構造：
```
math_plugin/
├── android/                # Android固有のネイティブコード
│   ├── src/main/cpp/      # C++コード（Android向け）
│   └── CMakeLists.txt     # Android用CMake設定
├── ios/                   # iOS固有のネイティブコード
│   └── Classes/           # C++コード（iOS向け）
├── lib/                   # Dartコード
│   └── math_plugin.dart   # プラグインのDart API
└── example/               # サンプルFlutterアプリ
```

## 事前準備

### 1. Flutter SDKのインストール

まだFlutter SDKをインストールしていない場合は、[Flutter公式サイト](https://flutter.dev/docs/get-started/install)の手順に従ってインストールしてください。

### 2. 必要なツールのインストール

Androidの場合はAndroid Studio、iOSの場合はXcodeが必要です。また、C++のビルドにはCMakeが必要です。

```bash
# macOSの場合
brew install cmake

# Ubuntuの場合
sudo apt-get install cmake
```

### 3. Flutterプロジェクトの作成

新しいFlutterプロジェクトを作成します。今回はプラグインプロジェクトとして作成します。

```bash
flutter create --template=plugin math_plugin
cd math_plugin
```

## やったこと

### 1. C++コードの作成（Android用）

まず、Android向けのC++コードを実装します。ここでは簡単な数学関数を実装します。

`android/src/main/cpp/math_functions.h` ファイルを作成します：

```cpp
#ifndef MATH_FUNCTIONS_H
#define MATH_FUNCTIONS_H

#ifdef __cplusplus
extern "C" {
#endif

// 2つの整数の和を計算する
int add(int a, int b);

// 2つの整数の差を計算する
int subtract(int a, int b);

// 2つの整数の積を計算する
int multiply(int a, int b);

// 2つの整数の商を計算する（ゼロ除算チェック付き）
double divide(int a, int b);

#ifdef __cplusplus
}
#endif

#endif // MATH_FUNCTIONS_H
```

続いて、`android/src/main/cpp/math_functions.cpp` ファイルを作成し、関数を実装します：

```cpp
#include "math_functions.h"
#include <stdexcept>

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
        return 0.0; // エラー時は0を返す（実際のアプリではエラー処理が必要）
    }
    return static_cast<double>(a) / b;
}
```

次に、JNI（Java Native Interface）を使ってJavaとC++を連携させるブリッジコードを作成します。
`android/src/main/cpp/math_plugin.cpp` ファイルを作成します：

```cpp
#include <jni.h>
#include "math_functions.h"

// JNIを通してJavaからC++の関数を呼び出すためのブリッジメソッド
extern "C" JNIEXPORT jint JNICALL
Java_com_example_math_1plugin_MathPlugin_nativeAdd(
        JNIEnv* env, jobject /* this */, jint a, jint b) {
    return add(a, b);
}

extern "C" JNIEXPORT jint JNICALL
Java_com_example_math_1plugin_MathPlugin_nativeSubtract(
        JNIEnv* env, jobject /* this */, jint a, jint b) {
    return subtract(a, b);
}

extern "C" JNIEXPORT jint JNICALL
Java_com_example_math_1plugin_MathPlugin_nativeMultiply(
        JNIEnv* env, jobject /* this */, jint a, jint b) {
    return multiply(a, b);
}

extern "C" JNIEXPORT jdouble JNICALL
Java_com_example_math_1plugin_MathPlugin_nativeDivide(
        JNIEnv* env, jobject /* this */, jint a, jint b) {
    return divide(a, b);
}
```

### 2. CMakeファイルの設定（Android用）

C++コードをビルドするために、`android/CMakeLists.txt` ファイルを作成または編集します：

```cmake
cmake_minimum_required(VERSION 3.10)

# プロジェクト名を設定
project(math_plugin)

# C++の標準を設定（C++11を使用）
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# ソースファイルを追加
add_library(math_plugin SHARED
    src/main/cpp/math_functions.cpp
    src/main/cpp/math_plugin.cpp
)

# インクルードディレクトリを設定
target_include_directories(math_plugin PRIVATE
    src/main/cpp
)

# Android Log ライブラリをリンク
find_library(log-lib log)
target_link_libraries(math_plugin ${log-lib})
```

### 3. Android側のプラグインコード

`android/src/main/kotlin/com/example/math_plugin/MathPlugin.kt` ファイルを編集して、JNIでC++の関数を呼び出すインターフェースを作成します：

```kotlin
package com.example.math_plugin

import androidx.annotation.NonNull
import io.flutter.embedding.engine.plugins.FlutterPlugin
import io.flutter.plugin.common.MethodCall
import io.flutter.plugin.common.MethodChannel
import io.flutter.plugin.common.MethodChannel.MethodCallHandler
import io.flutter.plugin.common.MethodChannel.Result

class MathPlugin: FlutterPlugin, MethodCallHandler {
  private lateinit var channel : MethodChannel

  override fun onAttachedToEngine(@NonNull flutterPluginBinding: FlutterPlugin.FlutterPluginBinding) {
    channel = MethodChannel(flutterPluginBinding.binaryMessenger, "math_plugin")
    channel.setMethodCallHandler(this)
  }

  override fun onMethodCall(@NonNull call: MethodCall, @NonNull result: Result) {
    when (call.method) {
      "add" -> {
        val a = call.argument<Int>("a") ?: 0
        val b = call.argument<Int>("b") ?: 0
        result.success(nativeAdd(a, b))
      }
      "subtract" -> {
        val a = call.argument<Int>("a") ?: 0
        val b = call.argument<Int>("b") ?: 0
        result.success(nativeSubtract(a, b))
      }
      "multiply" -> {
        val a = call.argument<Int>("a") ?: 0
        val b = call.argument<Int>("b") ?: 0
        result.success(nativeMultiply(a, b))
      }
      "divide" -> {
        val a = call.argument<Int>("a") ?: 0
        val b = call.argument<Int>("b") ?: 1
        result.success(nativeDivide(a, b))
      }
      else -> {
        result.notImplemented()
      }
    }
  }

  override fun onDetachedFromEngine(@NonNull binding: FlutterPlugin.FlutterPluginBinding) {
    channel.setMethodCallHandler(null)
  }

  // C++の関数を呼び出すネイティブメソッド
  private external fun nativeAdd(a: Int, b: Int): Int
  private external fun nativeSubtract(a: Int, b: Int): Int
  private external fun nativeMultiply(a: Int, b: Int): Int
  private external fun nativeDivide(a: Int, b: Int): Double

  companion object {
    // JNIライブラリのロード
    init {
      System.loadLibrary("math_plugin")
    }
  }
}
```

### 4. Android側のbuild.gradleの設定

`android/build.gradle` ファイルを編集して、CMakeのビルド設定を追加します：

```gradle
android {
    // 既存の設定...

    // 以下を追加
    externalNativeBuild {
        cmake {
            path "CMakeLists.txt"
        }
    }
}
```

### 5. iOS側のC++コードの設定

iOS側でも同じC++コードを使いますが、Objective-Cとの連携が必要です。
`ios/Classes/MathFunctions.h` と `ios/Classes/MathFunctions.mm` を作成して同じ関数を実装します。

`ios/Classes/MathFunctions.h` には先ほどと同じ内容を記述します。
`ios/Classes/MathFunctions.mm` には以下のように実装します：

```objc
#import "MathFunctions.h"

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
        return 0.0;
    }
    return (double)a / b;
}
```

### 6. iOS側のプラグインコード

`ios/Classes/MathPlugin.m` を編集してObjective-CからC++の関数を呼び出すインターフェースを作成します：

```objc
#import "MathPlugin.h"
#import "MathFunctions.h"

@implementation MathPlugin
+ (void)registerWithRegistrar:(NSObject<FlutterPluginRegistrar>*)registrar {
  FlutterMethodChannel* channel = [FlutterMethodChannel
      methodChannelWithName:@"math_plugin"
            binaryMessenger:[registrar messenger]];
  MathPlugin* instance = [[MathPlugin alloc] init];
  [registrar addMethodCallDelegate:instance channel:channel];
}

- (void)handleMethodCall:(FlutterMethodCall*)call result:(FlutterResult)result {
  if ([@"add" isEqualToString:call.method]) {
    NSNumber *a = call.arguments[@"a"];
    NSNumber *b = call.arguments[@"b"];
    result(@(add([a intValue], [b intValue])));
  } else if ([@"subtract" isEqualToString:call.method]) {
    NSNumber *a = call.arguments[@"a"];
    NSNumber *b = call.arguments[@"b"];
    result(@(subtract([a intValue], [b intValue])));
  } else if ([@"multiply" isEqualToString:call.method]) {
    NSNumber *a = call.arguments[@"a"];
    NSNumber *b = call.arguments[@"b"];
    result(@(multiply([a intValue], [b intValue])));
  } else if ([@"divide" isEqualToString:call.method]) {
    NSNumber *a = call.arguments[@"a"];
    NSNumber *b = call.arguments[@"b"];
    result(@(divide([a intValue], [b intValue])));
  } else {
    result(FlutterMethodNotImplemented);
  }
}

@end
```

### 7. Dart側のインターフェース

最後に、Flutterアプリから使用するためのDartインターフェースを実装します。
`lib/math_plugin.dart` ファイルを編集します：

```dart
import 'dart:async';

import 'package:flutter/services.dart';

class MathPlugin {
  static const MethodChannel _channel = MethodChannel('math_plugin');

  /// C++で実装された加算関数を呼び出します
  static Future<int> add(int a, int b) async {
    final int result = await _channel.invokeMethod('add', {'a': a, 'b': b});
    return result;
  }

  /// C++で実装された減算関数を呼び出します
  static Future<int> subtract(int a, int b) async {
    final int result = await _channel.invokeMethod('subtract', {'a': a, 'b': b});
    return result;
  }

  /// C++で実装された乗算関数を呼び出します
  static Future<int> multiply(int a, int b) async {
    final int result = await _channel.invokeMethod('multiply', {'a': a, 'b': b});
    return result;
  }

  /// C++で実装された除算関数を呼び出します
  static Future<double> divide(int a, int b) async {
    final double result = await _channel.invokeMethod('divide', {'a': a, 'b': b});
    return result;
  }
}
```

### 8. サンプルアプリでの使用

`example/lib/main.dart` を編集して、作成したプラグインを使用するサンプルアプリを作成します：

```dart
import 'package:flutter/material.dart';
import 'package:math_plugin/math_plugin.dart';

void main() {
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
          title: const Text('C++プラグインの例'),
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
  Future<void> _add() async {
    try {
      final (a, b) = _getInputValues();
      final int result = await MathPlugin.add(a, b);
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
  Future<void> _subtract() async {
    try {
      final (a, b) = _getInputValues();
      final int result = await MathPlugin.subtract(a, b);
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
  Future<void> _multiply() async {
    try {
      final (a, b) = _getInputValues();
      final int result = await MathPlugin.multiply(a, b);
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
  Future<void> _divide() async {
    try {
      final (a, b) = _getInputValues();
      final double result = await MathPlugin.divide(a, b);
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

## 実装の動作確認

以下のコマンドで、サンプルアプリを実行します。
```bash
cd example
flutter run
```

アプリが起動したら、2つの数値を入力し、各演算ボタンをタップすると、C++の関数が呼び出され結果が表示されます。

## 結論に対しての補足

### パフォーマンス比較

C++の数値計算処理はDartよりも高速なケースが多いですが、単純な四則演算のような軽い処理では体感できる差はないでしょう。しかし、以下のようなケースではC++の優位性が発揮されます：

- 行列演算や画像処理などの計算量の多い処理
- 既存のC/C++ライブラリを再利用したい場合
- メモリ使用量を厳密に制御したい場合

### プラットフォーム間の違い

今回の実装では、AndroidとiOSで共通のC++コードを使用していますが、JNIやObjective-Cのブリッジコードはプラットフォーム固有です。実際のアプリ開発では、このプラットフォーム固有の部分をできるだけ小さくして、大部分のロジックを共通のC++コードにまとめるアプローチが効果的です。

### Flutterにおける他のネイティブコード連携方法

今回はメソッドチャネルを使ってC++とDartを連携させましたが、他にも以下の方法があります：

1. **dart:ffi** - Dart Foreign Function Interfaceを使ったより直接的な連携方法
2. **PlatformChannel** - 複雑なデータ型の受け渡しが可能なより柔軟な通信方法
3. **Pigeon** - コード生成ツールを使ってプラットフォーム間の通信部分を自動生成する方法

プロジェクトの規模や要件に応じて、最適な方法を選ぶと良いでしょう。

## おわりに

この記事では、FlutterアプリからC++の関数を呼び出す基本的な方法を解説しました。単純な数値計算の例でしたが、この仕組みを応用すれば、複雑な計算処理や既存のC++ライブラリをFlutterアプリに統合することができます。

Flutterの強力なUI機能とC++の高速処理能力を組み合わせることで、美しいユーザーインターフェースと高性能な処理を両立したアプリケーションが開発できます。特に、ゲーム開発や科学計算、画像処理などの分野では、この連携が大きな効果を発揮するでしょう。

最初は設定や連携コードの作成が煩雑に感じるかもしれませんが、テンプレートを作成しておけば、次回からの実装はずっと簡単になります。ぜひ自分のプロジェクトに取り入れて、Flutterの可能性を広げてみてください。
