---
title: "Flutter初心者のためのRiverpod入門ガイド"
emoji: "🦅"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["flutter", "riverpod", "dart", "状態管理", "モバイル開発"]
published: false
---

# Flutter初心者のためのRiverpod入門ガイド

こんにちは！この記事では、Flutter初心者の方向けに、状態管理ライブラリ「Riverpod」の基本的な使い方を解説します。シンプルなコード例を通して、Riverpodを使った効率的な状態管理の方法を学んでいきましょう。

## Riverpodとは？

Riverpodは、Flutter開発における状態管理のための強力なライブラリです。「Provider」の作者によって開発された次世代の状態管理ソリューションで、以下のような特徴があります：

- 型安全性が高い
- コンパイル時にエラーを検出できる
- 依存関係の管理が簡単
- テストがしやすい
- コード分割がシンプル

Providerパッケージを使ったことがある方は、Riverpodはその進化版だと考えてください。より使いやすく、よりパワフルになっています。

## インストール方法

まずは、プロジェクトにRiverpodを追加しましょう。`pubspec.yaml`ファイルに以下の依存関係を追加します：

```yaml
dependencies:
  flutter:
    sdk: flutter
  flutter_riverpod: ^2.4.0  # 最新バージョンを確認してください
```

次に、ターミナルで以下のコマンドを実行して、パッケージをインストールします：

```bash
flutter pub get
```

## Riverpodの基本的な使い方

### 1. アプリケーションのセットアップ

まず、アプリケーション全体をProviderScopeでラップします。これにより、アプリケーション内でRiverpodを使用できるようになります：

```dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';

void main() {
  runApp(
    // ProviderScopeでアプリケーション全体をラップする
    ProviderScope(
      child: MyApp(),
    ),
  );
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Riverpod Demo',
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      home: HomePage(),
    );
  }
}
```

### 2. シンプルな状態管理：カウンターアプリ

Riverpodを使った最も基本的な例として、カウンターアプリを実装してみましょう。

#### プロバイダーの定義

まず、状態を保持するプロバイダーを定義します：

```dart
// 単純な値を提供するProvider
final counterProvider = StateProvider<int>((ref) => 0);
```

`StateProvider`は、シンプルな状態を管理するために使用します。ここでは、初期値が0のint型の状態を提供しています。

#### 状態の使用と更新

次に、このプロバイダーを使用して、UIを構築します：

```dart
class HomePage extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // プロバイダーから値を取得
    final count = ref.watch(counterProvider);
    
    return Scaffold(
      appBar: AppBar(
        title: Text('Riverpod カウンター'),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Text(
              'ボタンを押した回数:',
              style: TextStyle(fontSize: 18),
            ),
            Text(
              '$count',
              style: TextStyle(fontSize: 40, fontWeight: FontWeight.bold),
            ),
          ],
        ),
      ),
      floatingActionButton: FloatingActionButton(
        // 状態を更新
        onPressed: () => ref.read(counterProvider.notifier).state++,
        tooltip: 'インクリメント',
        child: Icon(Icons.add),
      ),
    );
  }
}
```

ここでの重要なポイント：

- `ConsumerWidget`を使うことで、widgetのビルドメソッドに`WidgetRef`が提供されます
- `ref.watch(counterProvider)`で状態の値を取得しています
- 状態が変更されると、このウィジェットは自動的に再ビルドされます
- `ref.read(counterProvider.notifier).state++`で状態を更新しています

### 3. 複雑な状態管理：TodoListアプリ

より実践的な例として、簡単なTodoリストアプリを実装してみましょう。

#### データモデルとプロバイダー

```dart
// Todoアイテムのモデル
class Todo {
  final String id;
  final String title;
  final bool completed;

  Todo({
    required this.id,
    required this.title,
    this.completed = false,
  });

  // 新しい状態のTodoを作成するメソッド
  Todo copyWith({String? title, bool? completed}) {
    return Todo(
      id: this.id,
      title: title ?? this.title,
      completed: completed ?? this.completed,
    );
  }
}

// Todoリストを管理するNotifierProvider
class TodoListNotifier extends StateNotifier<List<Todo>> {
  TodoListNotifier() : super([]);

  // 新しいTodoを追加
  void addTodo(String title) {
    state = [
      ...state,
      Todo(
        id: DateTime.now().toString(),
        title: title,
      ),
    ];
  }

  // Todoの完了状態を切り替え
  void toggleTodo(String id) {
    state = [
      for (final todo in state)
        if (todo.id == id)
          todo.copyWith(completed: !todo.completed)
        else
          todo,
    ];
  }

  // Todoを削除
  void removeTodo(String id) {
    state = state.where((todo) => todo.id != id).toList();
  }
}

// Todoリストのプロバイダー
final todoListProvider = StateNotifierProvider<TodoListNotifier, List<Todo>>((ref) {
  return TodoListNotifier();
});
```

#### UIの実装

```dart
class TodoListPage extends ConsumerWidget {
  final TextEditingController _controller = TextEditingController();

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // Todoリストを取得
    final todos = ref.watch(todoListProvider);
    
    return Scaffold(
      appBar: AppBar(
        title: Text('Riverpod TodoList'),
      ),
      body: Column(
        children: [
          Padding(
            padding: const EdgeInsets.all(8.0),
            child: Row(
              children: [
                Expanded(
                  child: TextField(
                    controller: _controller,
                    decoration: InputDecoration(
                      hintText: '新しいタスクを入力',
                    ),
                    onSubmitted: (value) {
                      if (value.isNotEmpty) {
                        // 新しいTodoを追加
                        ref.read(todoListProvider.notifier).addTodo(value);
                        _controller.clear();
                      }
                    },
                  ),
                ),
                IconButton(
                  icon: Icon(Icons.add),
                  onPressed: () {
                    if (_controller.text.isNotEmpty) {
                      // 新しいTodoを追加
                      ref.read(todoListProvider.notifier).addTodo(_controller.text);
                      _controller.clear();
                    }
                  },
                ),
              ],
            ),
          ),
          Expanded(
            child: ListView.builder(
              itemCount: todos.length,
              itemBuilder: (context, index) {
                final todo = todos[index];
                return ListTile(
                  leading: Checkbox(
                    value: todo.completed,
                    onChanged: (_) {
                      // Todoの完了状態を切り替え
                      ref.read(todoListProvider.notifier).toggleTodo(todo.id);
                    },
                  ),
                  title: Text(
                    todo.title,
                    style: TextStyle(
                      decoration: todo.completed
                          ? TextDecoration.lineThrough
                          : TextDecoration.none,
                    ),
                  ),
                  trailing: IconButton(
                    icon: Icon(Icons.delete),
                    onPressed: () {
                      // Todoを削除
                      ref.read(todoListProvider.notifier).removeTodo(todo.id);
                    },
                  ),
                );
              },
            ),
          ),
        ],
      ),
    );
  }
}
```

### 4. 非同期データの扱い方

Riverpodは非同期データの処理も簡単です。例えば、APIからデータを取得する場合：

```dart
// APIからユーザーデータを取得する関数
Future<List<User>> fetchUsers() async {
  // APIリクエストのシミュレーション
  await Future.delayed(Duration(seconds: 2));
  return [
    User(id: 1, name: '山田太郎'),
    User(id: 2, name: '佐藤花子'),
    User(id: 3, name: '鈴木一郎'),
  ];
}

// ユーザーモデル
class User {
  final int id;
  final String name;

  User({required this.id, required this.name});
}

// 非同期データを提供するプロバイダー
final usersProvider = FutureProvider<List<User>>((ref) async {
  return fetchUsers();
});
```

UIで非同期データを使用する：

```dart
class UsersPage extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // 非同期データを監視
    final asyncUsers = ref.watch(usersProvider);
    
    return Scaffold(
      appBar: AppBar(
        title: Text('ユーザー一覧'),
      ),
      body: asyncUsers.when(
        // データ読み込み中
        loading: () => Center(child: CircularProgressIndicator()),
        // エラー発生時
        error: (err, stack) => Center(child: Text('エラーが発生しました: $err')),
        // データ取得成功時
        data: (users) {
          return ListView.builder(
            itemCount: users.length,
            itemBuilder: (context, index) {
              final user = users[index];
              return ListTile(
                title: Text(user.name),
                subtitle: Text('ID: ${user.id}'),
              );
            },
          );
        },
      ),
    );
  }
}
```

## まとめ

この記事では、Riverpodの基本的な使い方について解説しました：

1. **セットアップ**: アプリケーション全体を`ProviderScope`でラップする
2. **基本的な状態管理**: `StateProvider`を使った単純な状態管理
3. **複雑な状態管理**: `StateNotifierProvider`を使ったより複雑な状態管理
4. **非同期データの処理**: `FutureProvider`を使った非同期データの取得と表示

Riverpodは学習曲線がややあるものの、一度習得すれば非常に強力で柔軟な状態管理が可能になります。特に大規模なアプリケーションでは、コードの保守性と可読性が大幅に向上します。

ぜひこの記事を参考に、あなたのFlutterアプリケーションでRiverpodを試してみてください！

## 参考リンク

- [Riverpod公式ドキュメント](https://riverpod.dev/)
- [Riverpod GitHubリポジトリ](https://github.com/rrousselGit/riverpod)
- [Flutter公式サイト](https://flutter.dev/)
