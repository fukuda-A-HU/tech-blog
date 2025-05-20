---
title: "Flutterを用いた、Githubでデータ連携を実現するタスク管理アプリの作り方"
emoji: "📱"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["flutter", "github", "api", "タスク管理", "モバイルアプリ"]
published: false
---

## ゴール(はじめに)：Flutter × GitHub APIでタスク管理アプリを作る

この記事では、Flutterを使ってGitHub APIと連携するタスク管理アプリを作成します。GitHubのIssuesをタスクとして扱い、アプリ内でタスクの作成・編集・削除を行えるようにします。この記事を通じて、Flutterの基本的なUI構築方法からGitHub APIを用いた外部サービス連携まで、実践的なアプリ開発の流れを学ぶことができます。

## やってみた結果

このチュートリアルを完了すると、以下のことが実現できます：

- Flutterを用いたモバイルアプリの基本構造の理解
- GitHub APIとの連携方法の習得
- タスク管理の基本機能（作成・表示・編集・削除）の実装
- GitHub Issuesを活用したデータ同期の実現

この記事を読んでいただくことで、他の外部APIとの連携方法にも応用可能なスキルを身につけることができます。

## 開発環境
- OS: Windows/macOS/Linux (いずれでも可)
- Flutter SDK: 3.10.0以上
- Dart: 3.0.0以上
- IDE: Visual Studio Code または Android Studio
- GitHubアカウント

## 事前準備
1. Flutter SDKをインストールしていること
2. GitHubアカウントを持っていること
3. 開発用のリポジトリをGitHubに作成しておくこと

## やったこと

ここからは実際に開発を進めていきます。段階的に説明していきますので、初心者の方も安心してついてきてください。

## Flutterプロジェクトの作成

まず、新しいFlutterプロジェクトを作成します：

```bash
flutter create github_task_app
cd github_task_app
```

このコマンドで「github_task_app」という名前のFlutterプロジェクトが作成されます。

## 必要なパッケージのインストール

GitHub APIとの連携に必要なパッケージをインストールします。`pubspec.yaml`ファイルに以下の依存関係を追加します：

```yaml
dependencies:
  flutter:
    sdk: flutter
  http: ^1.1.0  # HTTP通信用
  shared_preferences: ^2.2.0  # ローカルストレージ用
  flutter_dotenv: ^5.1.0  # 環境変数管理用
  provider: ^6.0.5  # 状態管理用
```

依存関係を追加したら、以下のコマンドで依存関係をインストールします：

```bash
flutter pub get
```

## 環境変数の設定

GitHubのAPIトークンを安全に管理するために、.envファイルを作成します。プロジェクトのルートディレクトリに`.env`ファイルを作成し、以下の内容を追加します：

```
GITHUB_TOKEN=your_personal_access_token
GITHUB_USERNAME=your_github_username
GITHUB_REPO=your_repository_name
```

注意：`.env`ファイルはバージョン管理から除外するために、`.gitignore`に追加してください。

## GitHub Personal Access Tokenの取得

GitHubのAPIを使用するには、Personal Access Tokenが必要です。以下の手順で取得しましょう：

1. GitHubにログインし、Settings > Developer settings > Personal access tokensに移動
2. 「Generate new token」をクリック
3. トークンの説明を入力し、「repo」のスコープを選択
4. トークンを生成し、コピーして`.env`ファイルに貼り付け

## モデルの作成

まず、タスクを表すモデルクラスを作成します。`lib/models`ディレクトリを作成し、そこに`task.dart`ファイルを作成します：

```dart
class Task {
  final int id;
  final String title;
  final String description;
  final String state;
  final DateTime createdAt;
  final DateTime updatedAt;

  Task({
    required this.id,
    required this.title,
    required this.description,
    required this.state,
    required this.createdAt,
    required this.updatedAt,
  });

  factory Task.fromJson(Map<String, dynamic> json) {
    return Task(
      id: json['number'] as int,
      title: json['title'] as String,
      description: json['body'] as String? ?? '',
      state: json['state'] as String,
      createdAt: DateTime.parse(json['created_at'] as String),
      updatedAt: DateTime.parse(json['updated_at'] as String),
    );
  }

  Map<String, dynamic> toJson() {
    return {
      'title': title,
      'body': description,
      'state': state,
    };
  }
}
```

## GitHub APIサービスの作成

次に、GitHub APIと通信するためのサービスクラスを作成します。`lib/services`ディレクトリを作成し、`github_service.dart`ファイルを作成します：

```dart
import 'dart:convert';
import 'package:flutter_dotenv/flutter_dotenv.dart';
import 'package:http/http.dart' as http;
import '../models/task.dart';

class GitHubService {
  final String _baseUrl = 'https://api.github.com';
  late final String _token;
  late final String _username;
  late final String _repo;

  GitHubService() {
    _token = dotenv.env['GITHUB_TOKEN'] ?? '';
    _username = dotenv.env['GITHUB_USERNAME'] ?? '';
    _repo = dotenv.env['GITHUB_REPO'] ?? '';
  }

  Map<String, String> get _headers => {
        'Authorization': 'token $_token',
        'Content-Type': 'application/json',
        'Accept': 'application/vnd.github.v3+json',
      };

  // すべてのタスク(Issues)を取得
  Future<List<Task>> getTasks() async {
    final response = await http.get(
      Uri.parse('$_baseUrl/repos/$_username/$_repo/issues?state=all'),
      headers: _headers,
    );

    if (response.statusCode == 200) {
      final List<dynamic> issuesJson = jsonDecode(response.body);
      return issuesJson.map((json) => Task.fromJson(json)).toList();
    } else {
      throw Exception('Failed to load tasks: ${response.statusCode}');
    }
  }

  // 新しいタスク(Issue)を作成
  Future<Task> createTask(String title, String description) async {
    final response = await http.post(
      Uri.parse('$_baseUrl/repos/$_username/$_repo/issues'),
      headers: _headers,
      body: jsonEncode({
        'title': title,
        'body': description,
      }),
    );

    if (response.statusCode == 201) {
      return Task.fromJson(jsonDecode(response.body));
    } else {
      throw Exception('Failed to create task: ${response.statusCode}');
    }
  }

  // タスク(Issue)を更新
  Future<Task> updateTask(int issueNumber, String title, String description, String state) async {
    final response = await http.patch(
      Uri.parse('$_baseUrl/repos/$_username/$_repo/issues/$issueNumber'),
      headers: _headers,
      body: jsonEncode({
        'title': title,
        'body': description,
        'state': state,
      }),
    );

    if (response.statusCode == 200) {
      return Task.fromJson(jsonDecode(response.body));
    } else {
      throw Exception('Failed to update task: ${response.statusCode}');
    }
  }
  
  // タスク(Issue)を閉じる（完了とする）
  Future<Task> closeTask(int issueNumber) async {
    return updateTask(issueNumber, '', '', 'closed');
  }
}
```

## 状態管理クラスの作成

アプリの状態を管理するためのProviderクラスを作成します。`lib/providers`ディレクトリを作成し、`task_provider.dart`ファイルを作成します：

```dart
import 'package:flutter/foundation.dart';
import '../models/task.dart';
import '../services/github_service.dart';

class TaskProvider with ChangeNotifier {
  final GitHubService _gitHubService = GitHubService();
  List<Task> _tasks = [];
  bool _isLoading = false;
  String _error = '';

  List<Task> get tasks => _tasks;
  bool get isLoading => _isLoading;
  String get error => _error;

  // タスク一覧を読み込む
  Future<void> loadTasks() async {
    _isLoading = true;
    _error = '';
    notifyListeners();

    try {
      _tasks = await _gitHubService.getTasks();
    } catch (e) {
      _error = e.toString();
    } finally {
      _isLoading = false;
      notifyListeners();
    }
  }

  // 新しいタスクを作成する
  Future<void> addTask(String title, String description) async {
    _isLoading = true;
    _error = '';
    notifyListeners();

    try {
      final newTask = await _gitHubService.createTask(title, description);
      _tasks.add(newTask);
    } catch (e) {
      _error = e.toString();
    } finally {
      _isLoading = false;
      notifyListeners();
    }
  }

  // タスクを更新する
  Future<void> updateTask(int taskId, String title, String description, String state) async {
    _isLoading = true;
    _error = '';
    notifyListeners();

    try {
      final updatedTask = await _gitHubService.updateTask(taskId, title, description, state);
      final index = _tasks.indexWhere((task) => task.id == taskId);
      if (index != -1) {
        _tasks[index] = updatedTask;
      }
    } catch (e) {
      _error = e.toString();
    } finally {
      _isLoading = false;
      notifyListeners();
    }
  }

  // タスクを完了にする
  Future<void> completeTask(int taskId) async {
    _isLoading = true;
    _error = '';
    notifyListeners();

    try {
      final updatedTask = await _gitHubService.closeTask(taskId);
      final index = _tasks.indexWhere((task) => task.id == taskId);
      if (index != -1) {
        _tasks[index] = updatedTask;
      }
    } catch (e) {
      _error = e.toString();
    } finally {
      _isLoading = false;
      notifyListeners();
    }
  }
}
```

## アプリのメイン画面の作成

アプリのメイン画面を作成します。`lib/main.dart`ファイルを以下のように編集します：

```dart
import 'package:flutter/material.dart';
import 'package:flutter_dotenv/flutter_dotenv.dart';
import 'package:provider/provider.dart';
import 'providers/task_provider.dart';
import 'screens/home_screen.dart';

void main() async {
  // .envファイルを読み込み
  await dotenv.load();
  
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return ChangeNotifierProvider(
      create: (ctx) => TaskProvider(),
      child: MaterialApp(
        title: 'GitHub Task App',
        theme: ThemeData(
          primarySwatch: Colors.blue,
          useMaterial3: true,
        ),
        home: const HomeScreen(),
      ),
    );
  }
}
```

## ホーム画面の作成

タスク一覧を表示するホーム画面を作成します。`lib/screens`ディレクトリを作成し、`home_screen.dart`ファイルを作成します：

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';
import '../providers/task_provider.dart';
import '../models/task.dart';
import 'task_detail_screen.dart';
import 'task_form_screen.dart';

class HomeScreen extends StatefulWidget {
  const HomeScreen({super.key});

  @override
  State<HomeScreen> createState() => _HomeScreenState();
}

class _HomeScreenState extends State<HomeScreen> {
  @override
  void initState() {
    super.initState();
    // 画面表示時にタスク一覧を読み込む
    Future.microtask(() => 
      Provider.of<TaskProvider>(context, listen: false).loadTasks()
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('GitHub Task Manager'),
        actions: [
          IconButton(
            icon: const Icon(Icons.refresh),
            onPressed: () {
              Provider.of<TaskProvider>(context, listen: false).loadTasks();
            },
          ),
        ],
      ),
      body: Consumer<TaskProvider>(
        builder: (ctx, taskProvider, child) {
          if (taskProvider.isLoading) {
            return const Center(child: CircularProgressIndicator());
          }
          
          if (taskProvider.error.isNotEmpty) {
            return Center(
              child: Column(
                mainAxisAlignment: MainAxisAlignment.center,
                children: [
                  Text('エラーが発生しました: ${taskProvider.error}'),
                  const SizedBox(height: 16),
                  ElevatedButton(
                    onPressed: () {
                      taskProvider.loadTasks();
                    },
                    child: const Text('再読み込み'),
                  ),
                ],
              ),
            );
          }

          final tasks = taskProvider.tasks;
          
          if (tasks.isEmpty) {
            return const Center(child: Text('タスクはありません。新しいタスクを追加してください。'));
          }

          return ListView.builder(
            itemCount: tasks.length,
            itemBuilder: (ctx, index) {
              final task = tasks[index];
              return TaskListItem(task: task);
            },
          );
        },
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () {
          Navigator.of(context).push(
            MaterialPageRoute(builder: (_) => const TaskFormScreen()),
          );
        },
        child: const Icon(Icons.add),
      ),
    );
  }
}

class TaskListItem extends StatelessWidget {
  final Task task;

  const TaskListItem({super.key, required this.task});

  @override
  Widget build(BuildContext context) {
    final isCompleted = task.state == 'closed';
    
    return Card(
      margin: const EdgeInsets.symmetric(horizontal: 16, vertical: 8),
      child: ListTile(
        title: Text(
          task.title,
          style: TextStyle(
            decoration: isCompleted ? TextDecoration.lineThrough : null,
            fontWeight: FontWeight.bold,
          ),
        ),
        subtitle: Text(
          task.description.length > 50 
            ? '${task.description.substring(0, 50)}...' 
            : task.description,
        ),
        leading: CircleAvatar(
          backgroundColor: isCompleted ? Colors.green : Colors.blue,
          child: Icon(
            isCompleted ? Icons.check : Icons.pending_actions,
            color: Colors.white,
          ),
        ),
        trailing: IconButton(
          icon: const Icon(Icons.more_vert),
          onPressed: () {
            _showTaskOptions(context, task);
          },
        ),
        onTap: () {
          Navigator.of(context).push(
            MaterialPageRoute(
              builder: (_) => TaskDetailScreen(taskId: task.id),
            ),
          );
        },
      ),
    );
  }

  void _showTaskOptions(BuildContext context, Task task) {
    showModalBottomSheet(
      context: context,
      builder: (ctx) {
        return SafeArea(
          child: Column(
            mainAxisSize: MainAxisSize.min,
            children: [
              ListTile(
                leading: const Icon(Icons.visibility),
                title: const Text('詳細を表示'),
                onTap: () {
                  Navigator.of(ctx).pop();
                  Navigator.of(context).push(
                    MaterialPageRoute(
                      builder: (_) => TaskDetailScreen(taskId: task.id),
                    ),
                  );
                },
              ),
              if (task.state == 'open')
                ListTile(
                  leading: const Icon(Icons.check),
                  title: const Text('完了にする'),
                  onTap: () {
                    Navigator.of(ctx).pop();
                    Provider.of<TaskProvider>(context, listen: false)
                        .completeTask(task.id);
                  },
                ),
            ],
          ),
        );
      },
    );
  }
}
```

## タスク詳細画面の作成

タスクの詳細を表示する画面を作成します。`lib/screens/task_detail_screen.dart`ファイルを作成します：

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';
import '../providers/task_provider.dart';
import '../models/task.dart';
import 'task_form_screen.dart';

class TaskDetailScreen extends StatelessWidget {
  final int taskId;

  const TaskDetailScreen({super.key, required this.taskId});

  @override
  Widget build(BuildContext context) {
    final taskProvider = Provider.of<TaskProvider>(context);
    final task = taskProvider.tasks.firstWhere((task) => task.id == taskId);

    return Scaffold(
      appBar: AppBar(
        title: const Text('タスクの詳細'),
        actions: [
          if (task.state == 'open')
            IconButton(
              icon: const Icon(Icons.edit),
              onPressed: () {
                Navigator.of(context).push(
                  MaterialPageRoute(
                    builder: (_) => TaskFormScreen(taskToEdit: task),
                  ),
                );
              },
            ),
        ],
      ),
      body: SingleChildScrollView(
        padding: const EdgeInsets.all(16.0),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Card(
              child: Padding(
                padding: const EdgeInsets.all(16.0),
                child: Column(
                  crossAxisAlignment: CrossAxisAlignment.start,
                  children: [
                    Row(
                      mainAxisAlignment: MainAxisAlignment.spaceBetween,
                      children: [
                        Expanded(
                          child: Text(
                            task.title,
                            style: Theme.of(context).textTheme.headlineSmall,
                          ),
                        ),
                        Chip(
                          label: Text(
                            task.state == 'open' ? '進行中' : '完了',
                          ),
                          backgroundColor: task.state == 'open'
                              ? Colors.blue
                              : Colors.green,
                          labelStyle: const TextStyle(color: Colors.white),
                        ),
                      ],
                    ),
                    const SizedBox(height: 16),
                    const Text(
                      '説明:',
                      style: TextStyle(
                        fontWeight: FontWeight.bold,
                        fontSize: 16,
                      ),
                    ),
                    const SizedBox(height: 8),
                    Text(task.description.isEmpty
                        ? '説明はありません'
                        : task.description),
                    const SizedBox(height: 24),
                    _buildInfoRow('作成日', _formatDate(task.createdAt)),
                    const SizedBox(height: 8),
                    _buildInfoRow('更新日', _formatDate(task.updatedAt)),
                  ],
                ),
              ),
            ),
            const SizedBox(height: 24),
            if (task.state == 'open')
              SizedBox(
                width: double.infinity,
                child: ElevatedButton.icon(
                  icon: const Icon(Icons.check),
                  label: const Text('タスクを完了にする'),
                  style: ElevatedButton.styleFrom(
                    backgroundColor: Colors.green,
                    foregroundColor: Colors.white,
                    padding: const EdgeInsets.symmetric(vertical: 12),
                  ),
                  onPressed: () {
                    _completeTask(context, task);
                  },
                ),
              ),
          ],
        ),
      ),
    );
  }

  Widget _buildInfoRow(String label, String value) {
    return Row(
      children: [
        Text(
          '$label: ',
          style: const TextStyle(fontWeight: FontWeight.bold),
        ),
        Text(value),
      ],
    );
  }

  String _formatDate(DateTime date) {
    return '${date.year}/${date.month}/${date.day} ${date.hour}:${date.minute.toString().padLeft(2, '0')}';
  }

  void _completeTask(BuildContext context, Task task) {
    showDialog(
      context: context,
      builder: (ctx) => AlertDialog(
        title: const Text('タスクを完了にしますか？'),
        content: const Text('このタスクを完了としてマークしますか？この操作は元に戻せません。'),
        actions: [
          TextButton(
            child: const Text('キャンセル'),
            onPressed: () => Navigator.of(ctx).pop(),
          ),
          TextButton(
            child: const Text('完了にする'),
            onPressed: () {
              Navigator.of(ctx).pop();
              Provider.of<TaskProvider>(context, listen: false)
                  .completeTask(task.id);
              Navigator.of(context).pop(); // 詳細画面を閉じる
            },
          ),
        ],
      ),
    );
  }
}
```

## タスク作成・編集フォームの作成

新しいタスクを作成したり、既存のタスクを編集するためのフォーム画面を作成します。`lib/screens/task_form_screen.dart`ファイルを作成します：

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';
import '../models/task.dart';
import '../providers/task_provider.dart';

class TaskFormScreen extends StatefulWidget {
  final Task? taskToEdit;

  const TaskFormScreen({super.key, this.taskToEdit});

  @override
  State<TaskFormScreen> createState() => _TaskFormScreenState();
}

class _TaskFormScreenState extends State<TaskFormScreen> {
  final _formKey = GlobalKey<FormState>();
  final _titleController = TextEditingController();
  final _descriptionController = TextEditingController();
  var _isLoading = false;

  @override
  void initState() {
    super.initState();
    // 編集モードの場合、既存のデータをフォームに設定
    if (widget.taskToEdit != null) {
      _titleController.text = widget.taskToEdit!.title;
      _descriptionController.text = widget.taskToEdit!.description;
    }
  }

  @override
  void dispose() {
    _titleController.dispose();
    _descriptionController.dispose();
    super.dispose();
  }

  Future<void> _saveForm() async {
    if (!_formKey.currentState!.validate()) {
      return;
    }

    setState(() {
      _isLoading = true;
    });

    try {
      final taskProvider = Provider.of<TaskProvider>(context, listen: false);
      
      if (widget.taskToEdit == null) {
        // 新規タスクの作成
        await taskProvider.addTask(
          _titleController.text,
          _descriptionController.text,
        );
      } else {
        // 既存タスクの更新
        await taskProvider.updateTask(
          widget.taskToEdit!.id,
          _titleController.text,
          _descriptionController.text,
          widget.taskToEdit!.state,
        );
      }
      
      if (mounted) {
        Navigator.of(context).pop();
      }
    } catch (error) {
      await showDialog(
        context: context,
        builder: (ctx) => AlertDialog(
          title: const Text('エラーが発生しました'),
          content: Text(error.toString()),
          actions: [
            TextButton(
              child: const Text('OK'),
              onPressed: () => Navigator.of(ctx).pop(),
            ),
          ],
        ),
      );
    } finally {
      if (mounted) {
        setState(() {
          _isLoading = false;
        });
      }
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text(widget.taskToEdit == null ? '新しいタスク' : 'タスクを編集'),
        actions: [
          IconButton(
            icon: const Icon(Icons.save),
            onPressed: _saveForm,
          ),
        ],
      ),
      body: _isLoading
          ? const Center(child: CircularProgressIndicator())
          : Padding(
              padding: const EdgeInsets.all(16.0),
              child: Form(
                key: _formKey,
                child: Column(
                  children: [
                    TextFormField(
                      controller: _titleController,
                      decoration: const InputDecoration(
                        labelText: 'タイトル',
                        border: OutlineInputBorder(),
                      ),
                      validator: (value) {
                        if (value == null || value.isEmpty) {
                          return 'タイトルを入力してください';
                        }
                        return null;
                      },
                      textInputAction: TextInputAction.next,
                    ),
                    const SizedBox(height: 16),
                    TextFormField(
                      controller: _descriptionController,
                      decoration: const InputDecoration(
                        labelText: '説明',
                        border: OutlineInputBorder(),
                        alignLabelWithHint: true,
                      ),
                      maxLines: 10,
                      keyboardType: TextInputType.multiline,
                    ),
                    const SizedBox(height: 24),
                    SizedBox(
                      width: double.infinity,
                      height: 50,
                      child: ElevatedButton(
                        onPressed: _saveForm,
                        style: ElevatedButton.styleFrom(
                          backgroundColor: Theme.of(context).primaryColor,
                          foregroundColor: Colors.white,
                        ),
                        child: Text(
                          widget.taskToEdit == null ? 'タスクを作成' : 'タスクを更新',
                          style: const TextStyle(fontSize: 16),
                        ),
                      ),
                    ),
                  ],
                ),
              ),
            ),
    );
  }
}
```

## アプリの実行

ここまでのコードを実装したら、以下のコマンドでアプリを実行できます：

```bash
flutter run
```

アプリが起動したら、GitHubに存在するIssuesがタスクとして表示され、新しいタスクを作成したり、既存のタスクを編集したり、完了としてマークしたりすることができます。

## 結論に対しての補足

- このアプリは、GitHub APIの仕様変更によって動作しなくなる可能性があります。GitHub APIの公式ドキュメントを参照して、最新の仕様に合わせて適宜コードを変更してください。
- 認証エラーが発生する場合は、Personal Access Tokenの権限設定を確認してください。
- より高度な機能を追加したい場合は、GitHub APIの他のエンドポイントも活用できます（例：ラベル機能、アサイン機能など）。

## 参考リンク
- [Flutter公式ドキュメント](https://flutter.dev/docs)
- [GitHub REST API v3](https://docs.github.com/en/rest)
- [http パッケージ](https://pub.dev/packages/http)
- [provider パッケージ](https://pub.dev/packages/provider)
- [flutter_dotenv パッケージ](https://pub.dev/packages/flutter_dotenv)

## おわりに

この記事では、FlutterとGitHub APIを連携させたタスク管理アプリの作成方法を解説しました。このプロジェクトを通じて、Flutterの基本的なUI構築からRESTful APIとの連携、状態管理の実装まで幅広いスキルを習得できたと思います。

実際のプロジェクトでは、より複雑な機能や美しいUIが必要になるかもしれませんが、この記事で学んだ基本的な知識があれば、それらの実装も決して難しくありません。ぜひ、この記事を基に自分だけのオリジナルアプリを開発してみてください。

GitHubのAPIを使った連携は、タスク管理だけでなく様々なアプリケーションに応用できます。この知識を活かして、あなたのアイデアを形にしていってください！
