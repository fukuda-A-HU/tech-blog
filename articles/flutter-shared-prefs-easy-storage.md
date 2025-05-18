---
title: "Flutterでshared_preferencesを使って簡単にデータを保存する方法"
emoji: "🔖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Flutter", "SharedPreferences", "モバイルアプリ", "データ保存", "初心者"]
published: false
---

## はじめに：FlutterアプリでShared Preferencesを使う理由

モバイルアプリを開発していると、設定やユーザーの状態など、簡単なデータを保存したいときがあります。例えば：

- ユーザーの設定（ダークモード/ライトモードなど）
- ログイン状態の保存
- アプリの使用状況（初回起動かどうかなど）
- 単純な値のキャッシュ

こういった小さなデータを保存するには、**Shared Preferences**が最適です。SQLiteよりも簡単に使えて、小さなデータの保存に向いています。

この記事では、Flutterアプリで`shared_preferences`パッケージを使ってデータを保存する方法を、初心者にもわかりやすく解説します。

## Shared Preferencesとは？

Shared Preferencesは、キーと値のペアでデータを保存する軽量なストレージシステムです。基本的な種類の値（文字列、数値、真偽値など）を簡単に保存できます。

主な特徴：
- 設定項目など小さなデータの保存に最適
- キー・バリュー形式でシンプル
- 非同期APIでアクセスが簡単
- プラットフォームごとに最適な実装（iOSではNSUserDefaults、AndroidではSharedPreferences）

## 今回作るもの

シンプルな設定画面とカウンター機能を例に、以下の操作を実装します：

- 設定値の保存（文字列、数値、真偽値）
- 設定値の読み込み
- カウンターの値の保存と取得

## 必要な環境

- Flutter開発環境（Flutter SDKがインストール済み）
- お好みのコードエディタ（VS Codeなど）
- 基本的なFlutterの知識

## 必要なパッケージのインストール

まずはFlutterプロジェクトを作成し、Shared Preferencesを使うためのパッケージをインストールします。

```bash
# Flutterプロジェクトの作成
flutter create preferences_app
cd preferences_app

# Shared Preferencesパッケージのインストール
flutter pub add shared_preferences
```

ここでインストールするパッケージ：
- `shared_preferences`: Flutter公式のシンプルな永続ストレージソリューション

## 設定マネージャークラスの作成

Shared Preferencesの操作をまとめた設定マネージャークラスを作成します。`lib/settings_manager.dart`というファイルを作成しましょう：

```dart
import 'package:shared_preferences/shared_preferences.dart';

class SettingsManager {
  // キー定数
  static const String usernameKey = 'username';
  static const String isDarkModeKey = 'isDarkMode';
  static const String counterKey = 'counter';
  static const String lastUpdatedKey = 'lastUpdated';

  // ユーザー名の保存
  static Future<bool> setUsername(String username) async {
    final SharedPreferences prefs = await SharedPreferences.getInstance();
    return prefs.setString(usernameKey, username);
  }

  // ユーザー名の取得
  static Future<String> getUsername() async {
    final SharedPreferences prefs = await SharedPreferences.getInstance();
    return prefs.getString(usernameKey) ?? '名無しさん'; // デフォルト値
  }

  // ダークモード設定の保存
  static Future<bool> setDarkMode(bool isDarkMode) async {
    final SharedPreferences prefs = await SharedPreferences.getInstance();
    return prefs.setBool(isDarkModeKey, isDarkMode);
  }

  // ダークモード設定の取得
  static Future<bool> getDarkMode() async {
    final SharedPreferences prefs = await SharedPreferences.getInstance();
    return prefs.getBool(isDarkModeKey) ?? false; // デフォルト値はfalse
  }

  // カウンター値の保存
  static Future<bool> setCounter(int value) async {
    final SharedPreferences prefs = await SharedPreferences.getInstance();
    return prefs.setInt(counterKey, value);
  }

  // カウンター値の取得
  static Future<int> getCounter() async {
    final SharedPreferences prefs = await SharedPreferences.getInstance();
    return prefs.getInt(counterKey) ?? 0; // デフォルト値は0
  }

  // カウンター値の増加
  static Future<int> incrementCounter() async {
    final SharedPreferences prefs = await SharedPreferences.getInstance();
    final int currentValue = prefs.getInt(counterKey) ?? 0;
    final int newValue = currentValue + 1;
    await prefs.setInt(counterKey, newValue);
    await prefs.setString(lastUpdatedKey, DateTime.now().toString());
    return newValue;
  }

  // 最終更新日時の取得
  static Future<String> getLastUpdated() async {
    final SharedPreferences prefs = await SharedPreferences.getInstance();
    return prefs.getString(lastUpdatedKey) ?? '未更新';
  }

  // すべての設定を消去
  static Future<bool> clearAll() async {
    final SharedPreferences prefs = await SharedPreferences.getInstance();
    return await prefs.clear();
  }
}
```

このクラスで実装していること：

1. **キー定数**：保存するデータのキーを一元管理
2. **データの種類別メソッド**：文字列、真偽値、数値などの保存と読み込み
3. **デフォルト値**：データがない場合のデフォルト値の設定
4. **便利なメソッド**：カウンターの増加など、実用的な機能

## メイン画面の実装

設定画面とカウンターを統合した画面を作成します。`lib/main.dart`を以下のように実装しましょう：

```dart
import 'package:flutter/material.dart';
import 'settings_manager.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatefulWidget {
  const MyApp({Key? key}) : super(key: key);

  @override
  _MyAppState createState() => _MyAppState();
}

class _MyAppState extends State<MyApp> {
  bool _isDarkMode = false;
  
  @override
  void initState() {
    super.initState();
    _loadTheme();
  }
  
  // テーマ設定を読み込む
  Future<void> _loadTheme() async {
    final isDarkMode = await SettingsManager.getDarkMode();
    setState(() {
      _isDarkMode = isDarkMode;
    });
  }

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Shared Preferences デモ',
      theme: _isDarkMode ? ThemeData.dark() : ThemeData.light(),
      home: const HomePage(),
    );
  }
}

class HomePage extends StatefulWidget {
  const HomePage({Key? key}) : super(key: key);

  @override
  _HomePageState createState() => _HomePageState();
}

class _HomePageState extends State<HomePage> {
  String _username = '名無しさん';
  bool _isDarkMode = false;
  int _counter = 0;
  String _lastUpdated = '未更新';
  final TextEditingController _usernameController = TextEditingController();

  @override
  void initState() {
    super.initState();
    _loadAllSettings();
  }

  // 全ての設定を読み込む
  Future<void> _loadAllSettings() async {
    final username = await SettingsManager.getUsername();
    final isDarkMode = await SettingsManager.getDarkMode();
    final counter = await SettingsManager.getCounter();
    final lastUpdated = await SettingsManager.getLastUpdated();
    
    setState(() {
      _username = username;
      _isDarkMode = isDarkMode;
      _counter = counter;
      _lastUpdated = lastUpdated;
      _usernameController.text = username;
    });
  }

  // ユーザー名を保存
  Future<void> _saveUsername() async {
    final newUsername = _usernameController.text.trim();
    if (newUsername.isNotEmpty) {
      await SettingsManager.setUsername(newUsername);
      setState(() {
        _username = newUsername;
      });
      ScaffoldMessenger.of(context).showSnackBar(
        const SnackBar(content: Text('ユーザー名を保存しました')),
      );
    }
  }

  // テーマ設定を変更
  Future<void> _toggleTheme(bool value) async {
    await SettingsManager.setDarkMode(value);
    setState(() {
      _isDarkMode = value;
    });
    
    // アプリ全体のテーマを変更
    final _MyAppState? appState = context.findAncestorStateOfType<_MyAppState>();
    if (appState != null) {
      appState.setState(() {
        appState._isDarkMode = value;
      });
    }
  }

  // カウンターを増加
  Future<void> _incrementCounter() async {
    final newValue = await SettingsManager.incrementCounter();
    final lastUpdated = await SettingsManager.getLastUpdated();
    setState(() {
      _counter = newValue;
      _lastUpdated = lastUpdated;
    });
  }

  // すべての設定をリセット
  Future<void> _resetAllSettings() async {
    await SettingsManager.clearAll();
    await _loadAllSettings();
    ScaffoldMessenger.of(context).showSnackBar(
      const SnackBar(content: Text('すべての設定をリセットしました')),
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Shared Preferences デモ'),
        actions: [
          IconButton(
            icon: const Icon(Icons.refresh),
            onPressed: _resetAllSettings,
            tooltip: 'すべてリセット',
          ),
        ],
      ),
      body: Padding(
        padding: const EdgeInsets.all(16.0),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            // ユーザー設定セクション
            Card(
              child: Padding(
                padding: const EdgeInsets.all(16.0),
                child: Column(
                  crossAxisAlignment: CrossAxisAlignment.start,
                  children: [
                    const Text(
                      '設定',
                      style: TextStyle(fontSize: 20, fontWeight: FontWeight.bold),
                    ),
                    const SizedBox(height: 16),
                    // ユーザー名設定
                    Row(
                      children: [
                        Expanded(
                          child: TextField(
                            controller: _usernameController,
                            decoration: const InputDecoration(
                              labelText: 'ユーザー名',
                              border: OutlineInputBorder(),
                            ),
                          ),
                        ),
                        const SizedBox(width: 8),
                        ElevatedButton(
                          onPressed: _saveUsername,
                          child: const Text('保存'),
                        ),
                      ],
                    ),
                    const SizedBox(height: 16),
                    // テーマ設定
                    Row(
                      children: [
                        const Text('ダークモード：'),
                        Switch(
                          value: _isDarkMode,
                          onChanged: _toggleTheme,
                        ),
                      ],
                    ),
                  ],
                ),
              ),
            ),
            const SizedBox(height: 24),
            // カウンターセクション
            Card(
              child: Padding(
                padding: const EdgeInsets.all(16.0),
                child: Column(
                  crossAxisAlignment: CrossAxisAlignment.start,
                  children: [
                    const Text(
                      'カウンター',
                      style: TextStyle(fontSize: 20, fontWeight: FontWeight.bold),
                    ),
                    const SizedBox(height: 16),
                    Center(
                      child: Text(
                        '$_counter',
                        style: const TextStyle(fontSize: 48, fontWeight: FontWeight.bold),
                      ),
                    ),
                    const SizedBox(height: 8),
                    Center(
                      child: Text(
                        '最終更新: $_lastUpdated',
                        style: const TextStyle(fontSize: 12, color: Colors.grey),
                      ),
                    ),
                    const SizedBox(height: 16),
                    Center(
                      child: ElevatedButton(
                        onPressed: _incrementCounter,
                        child: const Text('カウントアップ'),
                      ),
                    ),
                  ],
                ),
              ),
            ),
            const SizedBox(height: 24),
            // 現在の状態表示
            Card(
              child: Padding(
                padding: const EdgeInsets.all(16.0),
                child: Column(
                  crossAxisAlignment: CrossAxisAlignment.start,
                  children: [
                    const Text(
                      '現在の状態',
                      style: TextStyle(fontSize: 20, fontWeight: FontWeight.bold),
                    ),
                    const SizedBox(height: 8),
                    Text('ユーザー名: $_username'),
                    Text('ダークモード: ${_isDarkMode ? 'オン' : 'オフ'}'),
                    Text('カウンター値: $_counter'),
                  ],
                ),
              ),
            ),
          ],
        ),
      ),
    );
  }

  @override
  void dispose() {
    _usernameController.dispose();
    super.dispose();
  }
}
```

## 動作の流れ

この実装で実現できること：

1. **設定の保存と読み込み**：
   - ユーザー名の保存と読み込み
   - ダークモード設定の切り替えと保存
   - アプリ再起動時に設定を復元

2. **カウンターの永続化**：
   - カウンター値をインクリメントして保存
   - アプリ再起動後もカウンターの値を維持
   - 最終更新日時の記録

3. **設定のリセット**：
   - すべての設定を初期状態に戻す機能

## Shared Preferencesの主なメリット

1. **シンプルさ**：
   - SQLiteと比べてとても使いやすい
   - 複雑な設定が不要

2. **速度**：
   - 小さなデータの読み書きが高速

3. **型安全**：
   - 文字列、整数、真偽値など基本的な型のデータを保存可能

4. **プラットフォーム最適化**：
   - iOSとAndroidそれぞれの最適な実装を使用

## Shared Preferencesの主な制限

1. **小さなデータ向け**：
   - 大量のデータやリスト、複雑なオブジェクトの保存には不向き
   - 保存できる値の種類は限られている（文字列、整数、真偽値、文字列リスト、double）

2. **構造化データには不向き**：
   - SQLiteのような複雑なクエリや関連付けができない

3. **バージョン管理**：
   - データ構造の変更管理が難しい

## よくあるユースケース

Shared Preferencesは以下のような目的に最適です：

1. **ユーザー設定**：
   - テーマ設定（ダーク/ライトモード）
   - 通知設定
   - アプリの表示設定

2. **セッション情報**：
   - ユーザーID
   - ログイン状態
   - 認証トークン（セキュリティ要件が厳しくない場合）

3. **アプリ状態**：
   - 最後に使用したページ
   - 初回起動フラグ
   - ツアーガイド表示済みフラグ

## 注意点とベストプラクティス

1. **キー管理**：
   - キー名は定数として一元管理する（タイプミスを防ぐため）

2. **デフォルト値**：
   - nullチェックとデフォルト値の提供を忘れない
   - `prefs.getString(key) ?? 'デフォルト値'`のように使う

3. **機密データは保存しない**：
   - パスワードなどの機密情報は`flutter_secure_storage`などのセキュアな方法で保存する

4. **JSONデータの保存**：
   - 複雑なオブジェクトを保存したい場合は、JSONに変換してから文字列として保存する
   ```dart
   // 保存時
   final String jsonString = jsonEncode(myObject.toJson());
   await prefs.setString('myObject', jsonString);
   
   // 読み込み時
   final String? jsonString = prefs.getString('myObject');
   final myObject = jsonString != null ? MyObject.fromJson(jsonDecode(jsonString)) : null;
   ```

## まとめ

Shared Preferencesは、Flutterアプリで簡単にデータを保存するための優れた選択肢です。以下のような場合に特に有用です：

- 単純な設定値の保存
- アプリの状態維持
- 小さなデータの永続化

SQLiteと比べると機能は限られますが、その分、はるかに導入と使用が簡単です。多くの場合、小規模なアプリやシンプルな設定の保存には、Shared Preferencesで十分です。

複雑なデータ構造やリレーショナルデータが必要な場合はSQLiteを検討し、機密性の高い情報を保存する場合は`flutter_secure_storage`などのセキュアなストレージソリューションを使用しましょう。

ぜひこの記事を参考に、自分のFlutterアプリにShared Preferencesを導入してみてください！
