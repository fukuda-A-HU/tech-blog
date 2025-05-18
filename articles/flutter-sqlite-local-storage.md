---
title: "Flutterでsqliteを使って端末にデータを保存する方法"
emoji: "💾"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Flutter", "SQLite", "モバイルアプリ", "データベース", "初心者"]
published: false
---

## はじめに：FlutterアプリでSQLiteを使う理由

モバイルアプリを開発していると、ユーザーのデータを保存する必要が出てきます。例えば：

- ユーザーの設定情報
- アプリで作成したメモやタスク
- オフラインでも使えるようにするためのデータのキャッシュ

こういったデータを保存するために、Flutterでは**SQLite**というデータベースを使うことができます。

この記事では、FlutterアプリでSQLiteを使って端末にデータを保存する方法を、初心者でもわかりやすく解説します。

## SQLiteとは？

SQLiteは軽量なデータベースで、大きなサーバーは必要ありません。アプリ内に組み込んで使うことができるため、モバイルアプリに最適です。

主な特徴：
- サーバー不要（アプリ内に組み込み）
- 容量が小さい
- 高速
- 信頼性が高い
- SQLという標準的な言語で操作できる

## 今回作るもの

簡単なメモアプリを例に、以下の操作を実装します：

- メモの作成（Create）
- メモの読み取り（Read）
- メモの更新（Update）
- メモの削除（Delete）

これらの操作をまとめてCRUD操作と呼びます。

## 必要な環境

- Flutter開発環境（Flutter SDKがインストール済み）
- お好みのコードエディタ（VS Codeなど）
- 基本的なFlutterの知識

## 必要なパッケージのインストール

まずはFlutterプロジェクトを作成し、SQLiteを使うためのパッケージをインストールします。

```bash
# Flutterプロジェクトの作成
flutter create memo_app
cd memo_app

# SQLiteパッケージのインストール
flutter pub add sqflite path
```

ここでインストールする2つのパッケージ：
- `sqflite`: FlutterでSQLiteを使うためのパッケージ
- `path`: ファイルパスを扱うためのパッケージ（データベースファイルの場所を指定するのに使います）

## データベースヘルパークラスの作成

SQLiteの操作をまとめたヘルパークラスを作成します。`lib/database_helper.dart`というファイルを作成しましょう：

```dart
import 'package:sqflite/sqflite.dart';
import 'package:path/path.dart';

class DatabaseHelper {
  // シングルトンパターン
  static final DatabaseHelper _instance = DatabaseHelper._internal();
  factory DatabaseHelper() => _instance;
  DatabaseHelper._internal();

  // データベース本体
  static Database? _database;

  // データベースの取得（なければ作成）
  Future<Database> get database async {
    if (_database != null) return _database!;
    _database = await _initDatabase();
    return _database!;
  }

  // データベースの初期化
  Future<Database> _initDatabase() async {
    // データベースファイルのパスを取得
    String path = join(await getDatabasesPath(), 'memo_database.db');
    
    // データベースを開く（なければ作成）
    return await openDatabase(
      path,
      version: 1,
      onCreate: _onCreate,
    );
  }

  // テーブル作成
  Future<void> _onCreate(Database db, int version) async {
    await db.execute('''
      CREATE TABLE memos(
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        title TEXT NOT NULL,
        content TEXT NOT NULL,
        createdAt TEXT NOT NULL
      )
    ''');
  }

  // メモを追加
  Future<int> insertMemo(Map<String, dynamic> memo) async {
    Database db = await database;
    return await db.insert('memos', memo);
  }

  // 全てのメモを取得
  Future<List<Map<String, dynamic>>> getMemos() async {
    Database db = await database;
    return await db.query('memos', orderBy: 'createdAt DESC');
  }

  // メモを更新
  Future<int> updateMemo(Map<String, dynamic> memo) async {
    Database db = await database;
    return await db.update(
      'memos',
      memo,
      where: 'id = ?',
      whereArgs: [memo['id']],
    );
  }

  // メモを削除
  Future<int> deleteMemo(int id) async {
    Database db = await database;
    return await db.delete(
      'memos',
      where: 'id = ?',
      whereArgs: [id],
    );
  }
}
```

このクラスで実装していること：

1. **シングルトンパターン**：データベース接続を一つだけ維持するための設計
2. **データベースの初期化**：アプリ起動時にデータベースを準備
3. **テーブル作成**：メモを保存するためのテーブル構造を定義
4. **CRUD操作**：メモの作成・読み取り・更新・削除の機能

## メモのモデルクラスの作成

メモを表すモデルクラスを作成します。`lib/memo.dart`というファイルを作成しましょう：

```dart
class Memo {
  final int? id;
  final String title;
  final String content;
  final String createdAt;

  Memo({
    this.id,
    required this.title,
    required this.content,
    required this.createdAt,
  });

  // MapからMemoオブジェクトを作成（データベースから読み込み時に使用）
  factory Memo.fromMap(Map<String, dynamic> map) {
    return Memo(
      id: map['id'],
      title: map['title'],
      content: map['content'],
      createdAt: map['createdAt'],
    );
  }

  // MemoオブジェクトをMapに変換（データベースに保存時に使用）
  Map<String, dynamic> toMap() {
    return {
      'id': id,
      'title': title,
      'content': content,
      'createdAt': createdAt,
    };
  }
}
```

## メイン画面の実装

メモの一覧を表示し、新規作成・編集・削除ができる画面を作成します。`lib/main.dart`を以下のように実装しましょう：

```dart
import 'package:flutter/material.dart';
import 'memo.dart';
import 'database_helper.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'メモアプリ',
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      home: const MemoListScreen(),
    );
  }
}

class MemoListScreen extends StatefulWidget {
  const MemoListScreen({Key? key}) : super(key: key);

  @override
  _MemoListScreenState createState() => _MemoListScreenState();
}

class _MemoListScreenState extends State<MemoListScreen> {
  final DatabaseHelper _databaseHelper = DatabaseHelper();
  List<Memo> _memos = [];

  @override
  void initState() {
    super.initState();
    _loadMemos();
  }

  // メモの読み込み
  Future<void> _loadMemos() async {
    final memoMaps = await _databaseHelper.getMemos();
    setState(() {
      _memos = memoMaps.map((memoMap) => Memo.fromMap(memoMap)).toList();
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('メモ一覧'),
      ),
      body: _memos.isEmpty
          ? const Center(child: Text('メモがありません。右下の+ボタンから追加してください。'))
          : ListView.builder(
              itemCount: _memos.length,
              itemBuilder: (context, index) {
                final memo = _memos[index];
                return ListTile(
                  title: Text(memo.title),
                  subtitle: Text(
                    memo.content,
                    maxLines: 1,
                    overflow: TextOverflow.ellipsis,
                  ),
                  trailing: Text(memo.createdAt.split(' ')[0]), // 日付のみ表示
                  onTap: () => _showMemoDialog(memo), // タップで編集
                  onLongPress: () => _deleteMemo(memo), // 長押しで削除確認
                );
              },
            ),
      floatingActionButton: FloatingActionButton(
        onPressed: () => _showMemoDialog(null), // nullを渡して新規作成
        child: const Icon(Icons.add),
      ),
    );
  }

  // メモの追加・編集ダイアログ
  Future<void> _showMemoDialog(Memo? memo) async {
    final isEditing = memo != null;
    final titleController = TextEditingController(text: isEditing ? memo.title : '');
    final contentController = TextEditingController(text: isEditing ? memo.content : '');

    await showDialog(
      context: context,
      builder: (context) {
        return AlertDialog(
          title: Text(isEditing ? 'メモを編集' : '新しいメモ'),
          content: Column(
            mainAxisSize: MainAxisSize.min,
            children: [
              TextField(
                controller: titleController,
                decoration: const InputDecoration(labelText: 'タイトル'),
              ),
              TextField(
                controller: contentController,
                decoration: const InputDecoration(labelText: '内容'),
                maxLines: 3,
              ),
            ],
          ),
          actions: [
            TextButton(
              onPressed: () => Navigator.pop(context),
              child: const Text('キャンセル'),
            ),
            TextButton(
              onPressed: () async {
                // 作成日時
                final now = DateTime.now().toString();
                
                if (isEditing) {
                  // 既存メモの更新
                  final updatedMemo = Memo(
                    id: memo.id,
                    title: titleController.text,
                    content: contentController.text,
                    createdAt: memo.createdAt,
                  );
                  await _databaseHelper.updateMemo(updatedMemo.toMap());
                } else {
                  // 新規メモの作成
                  final newMemo = Memo(
                    title: titleController.text,
                    content: contentController.text,
                    createdAt: now,
                  );
                  await _databaseHelper.insertMemo(newMemo.toMap());
                }
                
                Navigator.pop(context);
                _loadMemos(); // メモリストを更新
              },
              child: Text(isEditing ? '更新' : '追加'),
            ),
          ],
        );
      },
    );
  }

  // メモ削除の確認ダイアログ
  Future<void> _deleteMemo(Memo memo) async {
    final confirmed = await showDialog<bool>(
      context: context,
      builder: (context) {
        return AlertDialog(
          title: const Text('確認'),
          content: Text('「${memo.title}」を削除しますか？'),
          actions: [
            TextButton(
              onPressed: () => Navigator.pop(context, false),
              child: const Text('キャンセル'),
            ),
            TextButton(
              onPressed: () => Navigator.pop(context, true),
              child: const Text('削除'),
            ),
          ],
        );
      },
    );

    if (confirmed == true && memo.id != null) {
      await _databaseHelper.deleteMemo(memo.id!);
      _loadMemos(); // メモリストを更新
    }
  }
}
```

## 動作の流れ

この実装で実現できること：

1. **メモの一覧表示**：
   - アプリ起動時にSQLiteからメモを読み込み、リスト表示します

2. **新規メモの作成**：
   - 右下の+ボタンをタップするとダイアログが表示される
   - 入力完了後、SQLiteにメモを保存

3. **メモの編集**：
   - リストのメモをタップするとダイアログが表示される
   - 編集完了後、SQLiteのメモが更新される

4. **メモの削除**：
   - リストのメモを長押しすると削除確認ダイアログが表示される
   - 確認すると、SQLiteからメモが削除される

## SQLiteの主なメリット

1. **オフライン対応**：
   - インターネット接続がなくても、データの保存・読み込みができます

2. **データの永続化**：
   - アプリを閉じても、次回起動時にデータが残っています

3. **高速な処理**：
   - 軽量なデータベースなので、操作が高速です

4. **標準的なSQL構文**：
   - SQLの知識があれば、複雑なクエリも実行できます

## 注意点とベストプラクティス

1. **大量データの処理**：
   - 数万件以上のデータを扱う場合は、一度に全てを読み込まず、ページネーションを使いましょう

2. **トランザクション**：
   - 複数の操作をまとめて行う場合は、トランザクションを使いましょう
   ```dart
   await database.transaction((txn) async {
     await txn.insert('memos', memo1);
     await txn.insert('memos', memo2);
   });
   ```

3. **バックアップ**：
   - 重要なデータはクラウドなどにバックアップする仕組みを考えましょう

4. **バージョン管理**：
   - データベースのスキーマを変更する場合は、バージョン番号を上げて適切にマイグレーション処理を書きましょう

## まとめ

今回はFlutterでSQLiteを使って簡単なメモアプリを作りました。SQLiteを使うことで、以下のことが実現できました：

- 端末にデータを永続的に保存できる
- オフラインでもアプリが使える
- データの追加・読み取り・更新・削除（CRUD操作）が簡単にできる

実際のアプリ開発では、より複雑なデータ構造やリレーションシップを扱うこともありますが、基本的な流れは今回紹介した通りです。状態管理とSQLiteを組み合わせることで、より使いやすいアプリを作ることができます。

ぜひこの記事を参考に、自分のアプリにSQLiteを導入してみてください！
