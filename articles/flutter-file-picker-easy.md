---
title: "Flutterでfile_pickerを使って簡単にファイルを選択する方法"
emoji: "📂"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Flutter", "FilePicker", "モバイルアプリ", "ファイル操作", "初心者"]
published: false
---

## はじめに：Flutterアプリでファイル選択が必要な理由

モバイルアプリやWebアプリを開発していると、ユーザーに端末内のファイルを選択してもらいたい場合があります。例えば：

- プロフィール画像のアップロード
- ドキュメントのインポート
- 画像や動画の選択
- バックアップファイルの読み込み

このような機能を実装するには、**file_picker**パッケージが非常に便利です。プラットフォームに関係なく、シンプルな方法でファイル選択UIを表示し、選択されたファイルへのパスやデータを取得できます。

この記事では、Flutterアプリで`file_picker`パッケージを使ってファイルを選択する方法を、初心者にもわかりやすく解説します。

## file_pickerとは？

`file_picker`は、複数のプラットフォーム（Android、iOS、Web、デスクトップ）でファイル選択機能を提供するFlutterプラグインです。プラットフォームごとにネイティブのファイル選択ダイアログを表示し、選択結果を統一されたAPIで取得できます。

主な特徴：
- 単一または複数ファイルの選択をサポート
- ファイル拡張子による選択フィルター
- 特定のメディアタイプ（画像、ビデオ、音声）の選択
- ファイルパスまたはバイトデータの取得
- カメラからの直接撮影（iOS/Android）
- クロスプラットフォームでの一貫した動作

## 今回作るもの

シンプルなファイル選択機能を備えたFlutterアプリを作成します。具体的には、次の機能を実装します：

- 画像ファイルの選択と表示
- 複数のファイル選択
- ファイルタイプによるフィルタリング
- 選択したファイルの情報表示

## 必要な環境

- Flutter開発環境（Flutter SDKがインストール済み）
- お好みのコードエディタ（VS Codeなど）
- 基本的なFlutterの知識

## 必要なパッケージのインストール

まずはFlutterプロジェクトを作成し、file_pickerを使うためのパッケージをインストールします。

```bash
# Flutterプロジェクトの作成
flutter create file_picker_app
cd file_picker_app

# file_pickerパッケージのインストール
flutter pub add file_picker
```

モバイルデバイスで画像を表示するために、`path_provider`パッケージも追加しておくと便利です：

```bash
flutter pub add path_provider
```

ここでインストールするパッケージ：
- `file_picker`: ファイル選択ダイアログを表示し、選択結果を取得するパッケージ
- `path_provider`: ファイルパスの操作を簡単にするユーティリティパッケージ

## iOS設定（Info.plist）

iOSでファイルピッカーを使用するには、Info.plistに権限を追加する必要があります。`ios/Runner/Info.plist`ファイルを開き、`<dict>`タグの中に以下を追加します：

```xml
<key>NSPhotoLibraryUsageDescription</key>
<string>写真を選択するためにフォトライブラリへのアクセスが必要です</string>
<key>NSCameraUsageDescription</key>
<string>写真を撮影するためにカメラへのアクセスが必要です</string>
<key>NSMicrophoneUsageDescription</key>
<string>ビデオを録画するためにマイクへのアクセスが必要です</string>
<key>NSDocumentsFolderUsageDescription</key>
<string>ドキュメントを選択するためにファイルへのアクセスが必要です</string>
<key>UIBackgroundModes</key>
<array>
  <string>fetch</string>
  <string>remote-notification</string>
</array>
```

## Android設定（build.gradle）

Androidでは、`android/app/build.gradle`ファイルの`android`セクションで以下の設定を確認してください：

```gradle
android {
    // ...
    defaultConfig {
        // ...
        minSdkVersion 19 // 最低でも19以上が必要
        // ...
    }
    // ...
}
```

## シンプルなファイル選択機能の実装

まずは基本的なファイル選択機能を実装します。`lib/main.dart`の内容を以下のように書き換えてください：

```dart
import 'dart:io';
import 'package:flutter/material.dart';
import 'package:file_picker/file_picker.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'File Picker デモ',
      theme: ThemeData(
        primarySwatch: Colors.blue,
        visualDensity: VisualDensity.adaptivePlatformDensity,
      ),
      home: const FilePickerDemo(),
    );
  }
}

class FilePickerDemo extends StatefulWidget {
  const FilePickerDemo({Key? key}) : super(key: key);

  @override
  _FilePickerDemoState createState() => _FilePickerDemoState();
}

class _FilePickerDemoState extends State<FilePickerDemo> {
  // 選択したファイルのパス
  String? _filePath;
  // 選択したファイル名
  String? _fileName;
  // 選択したファイルのサイズ
  String? _fileSize;
  // 選択したファイルの拡張子
  String? _fileExt;

  // 単一のファイルを選択するメソッド
  Future<void> _pickSingleFile() async {
    try {
      // ファイル選択ダイアログを表示（全てのファイルタイプが選択可能）
      FilePickerResult? result = await FilePicker.platform.pickFiles();

      // ユーザーがファイルを選択した場合
      if (result != null) {
        // 選択したファイル情報を取得
        PlatformFile file = result.files.first;
        
        setState(() {
          _filePath = file.path; // ファイルパス（WebではNull）
          _fileName = file.name; // ファイル名
          _fileSize = _formatFileSize(file.size); // ファイルサイズ
          _fileExt = file.extension; // ファイル拡張子
        });
      } else {
        // ユーザーがダイアログをキャンセルした場合
        print("ファイル選択がキャンセルされました");
      }
    } catch (e) {
      print("エラーが発生しました: $e");
    }
  }

  // ファイルサイズを読みやすい形式に変換
  String _formatFileSize(int size) {
    if (size < 1024) {
      return '$size B';
    } else if (size < 1024 * 1024) {
      return '${(size / 1024).toStringAsFixed(2)} KB';
    } else if (size < 1024 * 1024 * 1024) {
      return '${(size / (1024 * 1024)).toStringAsFixed(2)} MB';
    } else {
      return '${(size / (1024 * 1024 * 1024)).toStringAsFixed(2)} GB';
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('File Picker デモ'),
      ),
      body: Center(
        child: Padding(
          padding: const EdgeInsets.all(20.0),
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: <Widget>[
              // ファイルが選択されている場合、情報を表示
              if (_filePath != null) ...[
                const Text(
                  '選択したファイル情報:',
                  style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold),
                ),
                const SizedBox(height: 10),
                // ファイルが画像の場合は表示
                if (_fileExt != null &&
                    ['jpg', 'jpeg', 'png', 'gif', 'webp'].contains(_fileExt!.toLowerCase()))
                  ClipRRect(
                    borderRadius: BorderRadius.circular(8),
                    child: Image.file(
                      File(_filePath!),
                      width: 200,
                      height: 200,
                      fit: BoxFit.cover,
                    ),
                  ),
                const SizedBox(height: 20),
                // ファイル情報をリスト表示
                _buildInfoRow('ファイル名', _fileName ?? '不明'),
                _buildInfoRow('ファイルサイズ', _fileSize ?? '不明'),
                _buildInfoRow('ファイル拡張子', _fileExt ?? '不明'),
                _buildInfoRow('ファイルパス', _filePath ?? '不明'),
                const SizedBox(height: 20),
              ],
              // ファイル選択ボタン
              ElevatedButton.icon(
                onPressed: _pickSingleFile,
                icon: const Icon(Icons.file_open),
                label: const Text('ファイルを選択'),
                style: ElevatedButton.styleFrom(
                  padding: const EdgeInsets.symmetric(
                    horizontal: 20, 
                    vertical: 12,
                  ),
                ),
              ),
            ],
          ),
        ),
      ),
    );
  }

  // 情報行を表示するウィジェット
  Widget _buildInfoRow(String label, String value) {
    return Padding(
      padding: const EdgeInsets.only(bottom: 8.0),
      child: Row(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          SizedBox(
            width: 120,
            child: Text(
              '$label:',
              style: const TextStyle(fontWeight: FontWeight.bold),
            ),
          ),
          Expanded(
            child: Text(value),
          ),
        ],
      ),
    );
  }
}
```

## より高度なファイル選択機能の追加

ファイルタイプのフィルタリングや複数ファイルの選択など、より高度な機能を追加してみましょう。`lib/main.dart`に以下のメソッドを追加します。

```dart
// ファイル選択クラスに追加するメソッド

  // 画像ファイルのみを選択するメソッド
  Future<void> _pickImageFile() async {
    FilePickerResult? result = await FilePicker.platform.pickFiles(
      type: FileType.image, // 画像ファイルのみをフィルター
    );

    if (result != null) {
      PlatformFile file = result.files.first;
      setState(() {
        _filePath = file.path;
        _fileName = file.name;
        _fileSize = _formatFileSize(file.size);
        _fileExt = file.extension;
      });
    }
  }

  // カスタム拡張子でファイルを選択するメソッド
  Future<void> _pickCustomFile() async {
    FilePickerResult? result = await FilePicker.platform.pickFiles(
      type: FileType.custom,
      allowedExtensions: ['pdf', 'doc', 'docx', 'txt'], // 選択可能な拡張子を指定
    );

    if (result != null) {
      PlatformFile file = result.files.first;
      setState(() {
        _filePath = file.path;
        _fileName = file.name;
        _fileSize = _formatFileSize(file.size);
        _fileExt = file.extension;
      });
    }
  }

  // 複数ファイルを選択するメソッド
  List<PlatformFile> _selectedFiles = [];

  Future<void> _pickMultipleFiles() async {
    FilePickerResult? result = await FilePicker.platform.pickFiles(
      allowMultiple: true, // 複数選択を許可
      type: FileType.any, // すべてのタイプのファイルを選択可能
    );

    if (result != null) {
      setState(() {
        _selectedFiles = result.files;
        // 最初のファイルを表示用に設定（オプション）
        if (_selectedFiles.isNotEmpty) {
          final firstFile = _selectedFiles.first;
          _filePath = firstFile.path;
          _fileName = firstFile.name;
          _fileSize = _formatFileSize(firstFile.size);
          _fileExt = firstFile.extension;
        }
      });
    }
  }
```

また、上記のメソッドを利用するために、`build`メソッド内のウィジェットも追加します。`ElevatedButton.icon`のあとに以下のウィジェットを追加しましょう：

```dart
// 先ほどのElevatedButtonの下に追加
const SizedBox(height: 10),
// 画像選択ボタン
ElevatedButton.icon(
  onPressed: _pickImageFile,
  icon: const Icon(Icons.image),
  label: const Text('画像を選択'),
  style: ElevatedButton.styleFrom(
    padding: const EdgeInsets.symmetric(horizontal: 20, vertical: 12),
  ),
),
const SizedBox(height: 10),
// ドキュメント選択ボタン
ElevatedButton.icon(
  onPressed: _pickCustomFile,
  icon: const Icon(Icons.description),
  label: const Text('ドキュメントを選択'),
  style: ElevatedButton.styleFrom(
    padding: const EdgeInsets.symmetric(horizontal: 20, vertical: 12),
  ),
),
const SizedBox(height: 10),
// 複数ファイル選択ボタン
ElevatedButton.icon(
  onPressed: _pickMultipleFiles,
  icon: const Icon(Icons.file_copy),
  label: const Text('複数ファイルを選択'),
  style: ElevatedButton.styleFrom(
    padding: const EdgeInsets.symmetric(horizontal: 20, vertical: 12),
  ),
),

// 複数ファイルが選択されている場合、その数を表示
if (_selectedFiles.isNotEmpty) ...[
  const SizedBox(height: 20),
  Text('${_selectedFiles.length}個のファイルが選択されています', 
    style: const TextStyle(fontSize: 16, fontWeight: FontWeight.bold)),
  const SizedBox(height: 10),
  // 選択したファイル名のリスト（最初の5つのみ）
  ...List.generate(
    _selectedFiles.length > 5 ? 5 : _selectedFiles.length,
    (index) => Padding(
      padding: const EdgeInsets.only(bottom: 5),
      child: Text(
        '${index + 1}. ${_selectedFiles[index].name}',
        style: const TextStyle(fontSize: 14),
      ),
    ),
  ),
  if (_selectedFiles.length > 5) 
    Text('...他${_selectedFiles.length - 5}個のファイル'),
],
```

## 選択したファイルの活用方法

選択したファイルは、様々な方法で利用できます。以下のコードサンプルを参考にしてください：

### ファイルの読み込み

```dart
// 選択したファイルを読み込むメソッド
Future<void> _readFileContent() async {
  if (_filePath != null && _fileExt != null) {
    if (['txt', 'json', 'md', 'html', 'css', 'js'].contains(_fileExt!.toLowerCase())) {
      try {
        final file = File(_filePath!);
        final contents = await file.readAsString();
        print('ファイルの内容: $contents');
        // ここでファイルの内容を表示したり、処理したりすることができます
      } catch (e) {
        print('ファイルの読み込みエラー: $e');
      }
    }
  }
}
```

### ファイルのコピー

```dart
import 'dart:io';
import 'package:path_provider/path_provider.dart';
import 'package:path/path.dart' as path;

// ファイルをアプリのディレクトリにコピーするメソッド
Future<String?> _copyFileToAppDir() async {
  if (_filePath != null && _fileName != null) {
    try {
      // アプリのドキュメントディレクトリを取得
      final appDir = await getApplicationDocumentsDirectory();
      final newPath = path.join(appDir.path, _fileName!);
      
      // ファイルをコピー
      await File(_filePath!).copy(newPath);
      
      print('ファイルをコピーしました: $newPath');
      return newPath;
    } catch (e) {
      print('ファイルコピーエラー: $e');
      return null;
    }
  }
  return null;
}
```

## file_pickerの主な機能と利用方法

### 1. 基本的なファイル選択オプション

```dart
// 単一ファイル選択（すべてのタイプ）
FilePickerResult? result = await FilePicker.platform.pickFiles();

// 単一ファイル選択（特定のタイプ）
FilePickerResult? result = await FilePicker.platform.pickFiles(
  type: FileType.image, // FileType.audio, FileType.video, FileType.media, FileType.any
);

// 複数ファイル選択
FilePickerResult? result = await FilePicker.platform.pickFiles(
  allowMultiple: true,
);

// カスタム拡張子によるフィルタリング
FilePickerResult? result = await FilePicker.platform.pickFiles(
  type: FileType.custom,
  allowedExtensions: ['jpg', 'pdf', 'doc'],
);
```

### 2. ダイアログのカスタマイズオプション

```dart
FilePickerResult? result = await FilePicker.platform.pickFiles(
  // ダイアログのタイトル（iOSのみ）
  dialogTitle: '画像を選択してください',
  
  // 初期ディレクトリ（デスクトップのみ）
  initialDirectory: '/Users/username/Documents',
  
  // ロックアップ種類（ディレクトリを選択可能にする、デスクトップのみ）
  lockParentWindow: true,
  
  // 「すべてのファイル」オプションを表示（Webのみ）
  allowCompression: true,
);
```

### 3. バイトデータの直接取得

Webプラットフォームなど、ファイルパスが取得できない環境では、バイトデータを直接取得できます：

```dart
FilePickerResult? result = await FilePicker.platform.pickFiles(
  type: FileType.image,
  withData: true, // ファイルのバイトデータを取得
);

if (result != null) {
  Uint8List? fileBytes = result.files.first.bytes;
  String fileName = result.files.first.name;
  
  // バイトデータを使用した処理
  // 例: APIアップロード、イメージ表示など
}
```

### 4. ファイル保存ダイアログ（デスクトップのみ）

```dart
String? outputFile = await FilePicker.platform.saveFile(
  dialogTitle: 'ファイルを保存',
  fileName: 'my_document.pdf',
);

if (outputFile != null) {
  // 選択されたパスにファイルを保存する処理
}
```

## プラットフォーム別の注意点

### モバイル（Android/iOS）

- **権限**：適切な権限が必要です（Info.plistやAndroidManifestで設定）
- **パフォーマンス**：大きなファイルの取り扱いには注意しましょう
- **パス**：パスの形式がプラットフォームごとに異なります

### Web

- **セキュリティ制限**：Webではファイルシステムへの直接アクセスはできません
- **ファイルパス**：Webではファイルパスの代わりにバイトデータを使用します
- **サイズ制限**：ブラウザによってファイルサイズ制限があります

### デスクトップ（Windows/macOS/Linux）

- **フォルダ選択**：デスクトップではフォルダ選択も可能です
- **ファイルパス**：OSによってパスの形式が異なります

## 実装のベストプラクティス

1. **エラー処理**：
   - 常に例外処理を行い、ユーザーにフィードバックを提供しましょう
   
2. **非同期処理**：
   - `async/await`を使用し、UIをブロックしないようにしましょう

3. **ユーザービリティ**：
   - 選択中と選択後のUIフィードバックを提供しましょう
   - ファイルサイズ制限を設けることを検討しましょう

4. **メモリ管理**：
   - 大きなファイルを扱う場合は、メモリ使用量に注意しましょう
   - 不要になったファイルデータは解放しましょう

## まとめ

`file_picker`パッケージは、Flutter開発において、クロスプラットフォームでのファイル選択機能を簡単に実装するための非常に便利なツールです。主なメリットは：

- 単一コードでAndroid、iOS、Web、デスクトップをカバー
- シンプルで使いやすいAPI
- 様々なファイルタイプのサポート
- カスタマイズ可能なオプション

この記事で紹介した基本的な使い方をマスターすれば、あなたのFlutterアプリにファイル選択・管理機能を簡単に追加できるようになります。

アプリの要件に応じて、この基本実装をカスタマイズし、より高度な機能を追加してみてください。例えば、選択したファイルのサムネイル生成、ファイルのアップロード機能、選択したファイルの編集機能など、様々な拡張が可能です。

ぜひこの記事を参考に、あなたのFlutterアプリに`file_picker`を導入してみてください！
