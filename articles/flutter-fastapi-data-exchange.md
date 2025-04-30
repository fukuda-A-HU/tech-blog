---
title: "FlutterとFastAPIで文字列・画像・音声をやり取りする方法"
emoji: "🔄"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Flutter", "FastAPI", "Python", "API", "モバイル開発"]
published: false
---

# FlutterとFastAPIで文字列・画像・音声をやり取りする方法

こんにちは！この記事では、FlutterアプリからFastAPIサーバーに対して文字列、画像、音声データをやり取りする方法について解説します。モバイルアプリとバックエンドサーバー間でのデータ通信は、多くのアプリケーションで必要とされる基本的な機能です。

## はじめに

FlutterはGoogleが開発したクロスプラットフォームのUIフレームワークで、一つのコードベースからiOS、Android、Web、デスクトップアプリを開発できます。一方、FastAPIはPythonの最新機能を活用した高速なWebフレームワークで、APIの開発に特に適しています。

この記事では、これらの2つの技術を組み合わせて、以下の内容を実装する方法を紹介します：

1. 文字列データの送受信
2. 画像ファイルのアップロード/ダウンロード
3. 音声データの送受信

## 開発環境の準備

### FastAPI側の準備

まず、必要なパッケージをインストールします：

```bash
pip install fastapi uvicorn python-multipart pillow
```

- fastapi: FastAPIフレームワーク本体
- uvicorn: ASGIサーバー
- python-multipart: フォームデータ処理用
- pillow: 画像処理用

### Flutter側の準備

プロジェクトのpubspec.yamlに必要なパッケージを追加します：

```yaml
dependencies:
  flutter:
    sdk: flutter
  http: ^1.1.0
  image_picker: ^1.0.4
  file_picker: ^6.0.0
  audioplayers: ^5.2.0
  record: ^4.4.4
  path_provider: ^2.1.1
```

- http: HTTP通信用
- image_picker: カメラやギャラリーから画像を選択
- file_picker: ファイル選択
- audioplayers: 音声再生
- record: 音声録音
- path_provider: ファイルパスの取得

## 1. 文字列データの送受信

### FastAPI側の実装

まず、シンプルなFastAPIサーバーを作成します。以下のコードを`main.py`として保存します：

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class TextMessage(BaseModel):
    message: str

@app.post("/echo")
async def echo(text_message: TextMessage):
    return {"message": f"受信したメッセージ: {text_message.message}"}

@app.get("/hello")
async def hello():
    return {"message": "Hello from FastAPI!"}

if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

サーバーを起動するには：

```bash
python main.py
```

または：

```bash
uvicorn main:app --reload --host 0.0.0.0 --port 8000
```

### Flutter側の実装

Flutterアプリからテキストメッセージを送信し、レスポンスを受信するコードを作成します：

```dart
import 'package:flutter/material.dart';
import 'package:http/http.dart' as http;
import 'dart:convert';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter FastAPI Demo',
      theme: ThemeData(
        primarySwatch: Colors.blue,
        useMaterial3: true,
      ),
      home: const HomePage(),
    );
  }
}

class HomePage extends StatefulWidget {
  const HomePage({super.key});

  @override
  State<HomePage> createState() => _HomePageState();
}

class _HomePageState extends State<HomePage> {
  final TextEditingController _controller = TextEditingController();
  String _response = '';
  final String baseUrl = 'http://10.0.2.2:8000'; // AndroidエミュレータからローカルホストにアクセスするためのIP

  Future<void> _sendMessage() async {
    try {
      final response = await http.post(
        Uri.parse('$baseUrl/echo'),
        headers: {'Content-Type': 'application/json'},
        body: jsonEncode({'message': _controller.text}),
      );

      if (response.statusCode == 200) {
        final data = jsonDecode(response.body);
        setState(() {
          _response = data['message'];
        });
      } else {
        setState(() {
          _response = 'エラー: ${response.statusCode}';
        });
      }
    } catch (e) {
      setState(() {
        _response = 'エラー: $e';
      });
    }
  }

  Future<void> _getMessage() async {
    try {
      final response = await http.get(Uri.parse('$baseUrl/hello'));

      if (response.statusCode == 200) {
        final data = jsonDecode(response.body);
        setState(() {
          _response = data['message'];
        });
      } else {
        setState(() {
          _response = 'エラー: ${response.statusCode}';
        });
      }
    } catch (e) {
      setState(() {
        _response = 'エラー: $e';
      });
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('文字列通信デモ'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            TextField(
              controller: _controller,
              decoration: const InputDecoration(
                labelText: 'メッセージ',
                border: OutlineInputBorder(),
              ),
            ),
            const SizedBox(height: 16),
            Row(
              mainAxisAlignment: MainAxisAlignment.spaceEvenly,
              children: [
                ElevatedButton(
                  onPressed: _sendMessage,
                  child: const Text('送信'),
                ),
                ElevatedButton(
                  onPressed: _getMessage,
                  child: const Text('受信'),
                ),
              ],
            ),
            const SizedBox(height: 16),
            Text('レスポンス: $_response'),
          ],
        ),
      ),
    );
  }
}
```

> **注意**: 実際のIPアドレスやポート番号は、ご自身の環境に合わせて変更してください。
> - Android エミュレータ: 10.0.2.2
> - iOS シミュレータ: localhost
> - 実機: サーバーの実際のIPアドレス

## 2. 画像ファイルのアップロード/ダウンロード

### FastAPI側の実装

画像のアップロードとダウンロードを処理するエンドポイントを追加します：

```python
import os
from fastapi import FastAPI, File, UploadFile, Form
from fastapi.responses import FileResponse
from pydantic import BaseModel
import shutil

app = FastAPI()

# アップロードしたファイルを保存するディレクトリ
UPLOAD_DIR = "uploads"
os.makedirs(UPLOAD_DIR, exist_ok=True)

# 文字列通信（前述のコード）
# ...

@app.post("/upload-image")
async def upload_image(file: UploadFile = File(...), description: str = Form(None)):
    # ファイルの保存
    file_path = os.path.join(UPLOAD_DIR, file.filename)
    with open(file_path, "wb") as buffer:
        shutil.copyfileobj(file.file, buffer)
    
    return {
        "filename": file.filename,
        "description": description,
        "message": "画像がアップロードされました"
    }

@app.get("/images/{filename}")
async def get_image(filename: str):
    file_path = os.path.join(UPLOAD_DIR, filename)
    if os.path.exists(file_path):
        return FileResponse(file_path)
    return {"error": "ファイルが見つかりません"}

# 以下、起動コード
# ...
```

### Flutter側の実装

画像のアップロードとダウンロードを行うFlutterコードを追加します：

```dart
import 'package:flutter/material.dart';
import 'package:http/http.dart' as http;
import 'dart:convert';
import 'package:image_picker/image_picker.dart';
import 'dart:io';
import 'package:path/path.dart' as path;

class ImageUploadPage extends StatefulWidget {
  const ImageUploadPage({super.key});

  @override
  State<ImageUploadPage> createState() => _ImageUploadPageState();
}

class _ImageUploadPageState extends State<ImageUploadPage> {
  File? _selectedImage;
  final ImagePicker _picker = ImagePicker();
  String _response = '';
  String? _downloadedImageUrl;
  final String baseUrl = 'http://10.0.2.2:8000';

  Future<void> _pickImage() async {
    final XFile? pickedFile = await _picker.pickImage(source: ImageSource.gallery);
    
    if (pickedFile != null) {
      setState(() {
        _selectedImage = File(pickedFile.path);
        _downloadedImageUrl = null;
      });
    }
  }

  Future<void> _uploadImage() async {
    if (_selectedImage == null) {
      setState(() {
        _response = '画像が選択されていません';
      });
      return;
    }

    try {
      final request = http.MultipartRequest('POST', Uri.parse('$baseUrl/upload-image'));
      
      // ファイルの追加
      request.files.add(
        await http.MultipartFile.fromPath(
          'file',
          _selectedImage!.path,
          filename: path.basename(_selectedImage!.path),
        ),
      );
      
      // メタデータの追加
      request.fields['description'] = '画像の説明';
      
      // リクエスト送信
      final streamedResponse = await request.send();
      final response = await http.Response.fromStream(streamedResponse);
      
      if (response.statusCode == 200) {
        final data = jsonDecode(response.body);
        setState(() {
          _response = data['message'];
          _downloadedImageUrl = '$baseUrl/images/${data['filename']}';
        });
      } else {
        setState(() {
          _response = 'エラー: ${response.statusCode}';
        });
      }
    } catch (e) {
      setState(() {
        _response = 'エラー: $e';
      });
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('画像アップロードデモ'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            // 選択した画像を表示
            if (_selectedImage != null)
              Container(
                width: 200,
                height: 200,
                decoration: BoxDecoration(
                  border: Border.all(color: Colors.grey),
                ),
                child: Image.file(_selectedImage!),
              ),
              
            // ダウンロードした画像を表示
            if (_downloadedImageUrl != null)
              Container(
                width: 200,
                height: 200,
                decoration: BoxDecoration(
                  border: Border.all(color: Colors.grey),
                ),
                child: Image.network(_downloadedImageUrl!),
              ),
              
            const SizedBox(height: 16),
            
            Row(
              mainAxisAlignment: MainAxisAlignment.spaceEvenly,
              children: [
                ElevatedButton(
                  onPressed: _pickImage,
                  child: const Text('画像を選択'),
                ),
                ElevatedButton(
                  onPressed: _uploadImage,
                  child: const Text('アップロード'),
                ),
              ],
            ),
            
            const SizedBox(height: 16),
            Text('レスポンス: $_response'),
          ],
        ),
      ),
    );
  }
}
```

## 3. 音声データの送受信

### FastAPI側の実装

音声ファイルのアップロードとダウンロードを処理するエンドポイントを追加します：

```python
@app.post("/upload-audio")
async def upload_audio(file: UploadFile = File(...)):
    # ファイルの保存
    file_path = os.path.join(UPLOAD_DIR, file.filename)
    with open(file_path, "wb") as buffer:
        shutil.copyfileobj(file.file, buffer)
    
    return {
        "filename": file.filename,
        "message": "音声がアップロードされました"
    }

@app.get("/audio/{filename}")
async def get_audio(filename: str):
    file_path = os.path.join(UPLOAD_DIR, filename)
    if os.path.exists(file_path):
        return FileResponse(file_path)
    return {"error": "ファイルが見つかりません"}
```

### Flutter側の実装

音声の録音とアップロードを行うFlutterコードを追加します：

```dart
import 'package:flutter/material.dart';
import 'package:http/http.dart' as http;
import 'dart:convert';
import 'dart:io';
import 'package:path/path.dart' as path;
import 'package:record/record.dart';
import 'package:audioplayers/audioplayers.dart';
import 'package:path_provider/path_provider.dart';

class AudioUploadPage extends StatefulWidget {
  const AudioUploadPage({super.key});

  @override
  State<AudioUploadPage> createState() => _AudioUploadPageState();
}

class _AudioUploadPageState extends State<AudioUploadPage> {
  final Record _recorder = Record();
  final AudioPlayer _audioPlayer = AudioPlayer();
  String? _recordedFilePath;
  String? _downloadedAudioUrl;
  bool _isRecording = false;
  String _response = '';
  final String baseUrl = 'http://10.0.2.2:8000';

  Future<void> _startRecording() async {
    try {
      final tempDir = await getTemporaryDirectory();
      _recordedFilePath = '${tempDir.path}/recorded_audio.m4a';
      
      // マイク権限の確認
      if (await _recorder.hasPermission()) {
        await _recorder.start(path: _recordedFilePath!);
        setState(() {
          _isRecording = true;
        });
      } else {
        setState(() {
          _response = 'マイクの権限がありません';
        });
      }
    } catch (e) {
      setState(() {
        _response = 'エラー: $e';
      });
    }
  }

  Future<void> _stopRecording() async {
    try {
      await _recorder.stop();
      setState(() {
        _isRecording = false;
      });
    } catch (e) {
      setState(() {
        _response = 'エラー: $e';
      });
    }
  }

  Future<void> _playRecordedAudio() async {
    if (_recordedFilePath != null) {
      await _audioPlayer.play(DeviceFileSource(_recordedFilePath!));
    }
  }

  Future<void> _uploadAudio() async {
    if (_recordedFilePath == null) {
      setState(() {
        _response = '音声が録音されていません';
      });
      return;
    }

    try {
      final request = http.MultipartRequest('POST', Uri.parse('$baseUrl/upload-audio'));
      
      // ファイルの追加
      request.files.add(
        await http.MultipartFile.fromPath(
          'file',
          _recordedFilePath!,
          filename: 'recorded_audio.m4a',
        ),
      );
      
      // リクエスト送信
      final streamedResponse = await request.send();
      final response = await http.Response.fromStream(streamedResponse);
      
      if (response.statusCode == 200) {
        final data = jsonDecode(response.body);
        setState(() {
          _response = data['message'];
          _downloadedAudioUrl = '$baseUrl/audio/${data['filename']}';
        });
      } else {
        setState(() {
          _response = 'エラー: ${response.statusCode}';
        });
      }
    } catch (e) {
      setState(() {
        _response = 'エラー: $e';
      });
    }
  }

  Future<void> _playDownloadedAudio() async {
    if (_downloadedAudioUrl != null) {
      await _audioPlayer.play(UrlSource(_downloadedAudioUrl!));
    }
  }

  @override
  void dispose() {
    _recorder.dispose();
    _audioPlayer.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('音声録音・アップロードデモ'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            Text(_isRecording ? '録音中...' : '録音停止中'),
            const SizedBox(height: 16),
            Row(
              mainAxisAlignment: MainAxisAlignment.spaceEvenly,
              children: [
                ElevatedButton(
                  onPressed: _isRecording ? _stopRecording : _startRecording,
                  child: Text(_isRecording ? '録音停止' : '録音開始'),
                ),
                ElevatedButton(
                  onPressed: _recordedFilePath != null ? _playRecordedAudio : null,
                  child: const Text('録音再生'),
                ),
              ],
            ),
            const SizedBox(height: 16),
            Row(
              mainAxisAlignment: MainAxisAlignment.spaceEvenly,
              children: [
                ElevatedButton(
                  onPressed: _uploadAudio,
                  child: const Text('アップロード'),
                ),
                ElevatedButton(
                  onPressed: _downloadedAudioUrl != null ? _playDownloadedAudio : null,
                  child: const Text('ダウンロード再生'),
                ),
              ],
            ),
            const SizedBox(height: 16),
            Text('レスポンス: $_response'),
          ],
        ),
      ),
    );
  }
}
```

## メインアプリの統合

最後に、これらの機能を一つのアプリにまとめます。以下のコードをFlutterプロジェクトのmain.dartに追加します：

```dart
import 'package:flutter/material.dart';

// 上記で作成した各ページのimport
// import 'text_page.dart';
// import 'image_upload_page.dart';
// import 'audio_upload_page.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter FastAPI デモ',
      theme: ThemeData(
        primarySwatch: Colors.blue,
        useMaterial3: true,
      ),
      home: const MainPage(),
    );
  }
}

class MainPage extends StatelessWidget {
  const MainPage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Flutter + FastAPI デモ'),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            ElevatedButton(
              onPressed: () {
                Navigator.push(
                  context,
                  MaterialPageRoute(builder: (context) => const HomePage()),
                );
              },
              child: const Text('文字列通信デモ'),
            ),
            const SizedBox(height: 16),
            ElevatedButton(
              onPressed: () {
                Navigator.push(
                  context,
                  MaterialPageRoute(builder: (context) => const ImageUploadPage()),
                );
              },
              child: const Text('画像アップロードデモ'),
            ),
            const SizedBox(height: 16),
            ElevatedButton(
              onPressed: () {
                Navigator.push(
                  context,
                  MaterialPageRoute(builder: (context) => const AudioUploadPage()),
                );
              },
              child: const Text('音声録音・アップロードデモ'),
            ),
          ],
        ),
      ),
    );
  }
}
```

## 注意点

1. **セキュリティ**：本番環境では、適切な認証や入力検証を実装してください。
2. **ファイルサイズ**：大きなファイルの場合、チャンク分割などを検討する必要があります。
3. **クロスオリジン**：本番環境ではCORS（Cross-Origin Resource Sharing）の設定が必要になることがあります。
4. **エラーハンドリング**：実際のアプリでは、より堅牢なエラー処理を実装してください。

## まとめ

この記事では、FlutterとFastAPIを使って以下の内容を実装する方法を紹介しました：

1. 文字列データの送受信
2. 画像ファイルのアップロード/ダウンロード
3. 音声データの送受信

これらの基本的な実装を土台にして、より複雑なアプリケーションを構築することができます。たとえば、画像分析APIと連携したり、音声認識サービスを統合したりといった拡張も可能です。

最後に、本番環境に展開する際は、セキュリティ対策や最適化を忘れないようにしましょう。

## 参考リンク

- [Flutter公式ドキュメント](https://flutter.dev/docs)
- [FastAPI公式ドキュメント](https://fastapi.tiangolo.com/)
- [HTTP package for Flutter](https://pub.dev/packages/http)
- [Image Picker for Flutter](https://pub.dev/packages/image_picker)
- [Record package for Flutter](https://pub.dev/packages/record)
