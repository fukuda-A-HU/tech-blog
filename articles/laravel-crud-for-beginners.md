---
title: "Laravel初心者のためのCRUD実装入門"
emoji: "🔥"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Laravel", "PHP", "Web開発", "初心者向け", "CRUD"]
published: false
---

# Laravel初心者のためのCRUD実装入門

こんにちは！この記事では、Laravel初心者の方向けに、基本的なCRUD（Create, Read, Update, Delete）機能を実装する方法を解説します。PHPやLaravelを始めたばかりの方でも、簡単にWebアプリケーションを構築できるよう丁寧に説明していきます。

## はじめに

LaravelはPHPのフレームワークの中でも特に人気が高く、エレガントな構文と充実した機能を提供しています。このフレームワークを使うと、シンプルなCRUDアプリケーションから複雑なWebサービスまで、効率良く開発することができます。

この記事では、MacOS環境でLaravelをセットアップし、簡単なタスク管理アプリケーションを作成しながらCRUD操作の基本を学んでいきましょう。

## 環境構築

まずは開発環境を整えましょう。MacOSで開発するための環境構築手順を説明します。

### 1. PHPのインストール

MacOSには標準でPHPがインストールされていますが、バージョンが古い場合があります。Homebrewを使って最新版をインストールしましょう。

```bash
# Homebrewをまだインストールしていない場合は、下記コマンドでインストール
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# PHPをインストール
brew install php

# バージョンを確認
php -v
```

PHP 8.0以上がインストールされていれば問題ありません。

### 2. Composerのインストール

ComposerはPHPのパッケージマネージャーで、Laravelのインストールに必要です。

```bash
brew install composer

# インストールを確認
composer --version
```

### 3. Laravelプロジェクトの作成

Composerを使ってLaravelプロジェクトを作成します。

```bash
composer create-project laravel/laravel task-manager

# 作成したプロジェクトディレクトリに移動
cd task-manager

# 開発サーバーを起動
php artisan serve
```

`php artisan serve`コマンドを実行すると、通常は`http://127.0.0.1:8000`でローカルサーバーが起動します。ブラウザでこのURLにアクセスすると、Laravelのウェルカムページが表示されるはずです。

## データベースの設定

CRUDアプリケーションを作るには、データベースが必要です。デフォルトではLaravelはMySQLを使用する設定になっていますが、初心者の方は設定が簡単なSQLiteを使用することをお勧めします。

### 1. SQLiteデータベースの準備

```bash
# databaseディレクトリ内にSQLiteファイルを作成
touch database/database.sqlite
```

### 2. 環境設定ファイルの編集

`.env`ファイルを開き、データベース設定を変更します：

```
DB_CONNECTION=sqlite
# 以下の3行はコメントアウトするか削除してください
# DB_HOST=127.0.0.1
# DB_PORT=3306
# DB_DATABASE=laravel
# DB_USERNAME=root
# DB_PASSWORD=
```

これで、SQLiteデータベースを使う準備ができました。

## タスク管理アプリの作成

それでは、シンプルなタスク管理アプリケーションを作成していきましょう。ユーザーができることは次の通りです：

- タスクの一覧表示（Read）
- 新しいタスクの作成（Create）
- タスクの編集（Update）
- タスクの削除（Delete）

### 1. モデルとマイグレーションファイルの作成

まず、`Task`モデルとそれに対応するデータベースマイグレーションファイルを作成します。

```bash
php artisan make:model Task -m
```

`-m`オプションを付けることで、モデルと同時にマイグレーションファイルも作成されます。

### 2. マイグレーションファイルの編集

作成されたマイグレーションファイル（`database/migrations/yyyy_mm_dd_hhmmss_create_tasks_table.php`）を開き、タスクテーブルの構造を定義します：

```php
public function up()
{
    Schema::create('tasks', function (Blueprint $table) {
        $table->id();
        $table->string('title');
        $table->text('description')->nullable();
        $table->boolean('completed')->default(false);
        $table->timestamps();
    });
}
```

これで、タスクテーブルには以下のカラムが定義されました：
- `id`: 自動増分のプライマリーキー
- `title`: タスクのタイトル
- `description`: タスクの詳細説明（任意）
- `completed`: タスクの完了状態（デフォルトは未完了）
- `created_at`と`updated_at`: 作成日時と更新日時（timestampsで自動作成）

### 3. マイグレーションの実行

定義したテーブル構造をデータベースに反映させます：

```bash
php artisan migrate
```

### 4. Taskモデルの編集

`app/Models/Task.php`を開き、マスアサインメントのための属性を設定します：

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;

class Task extends Model
{
    use HasFactory;
    
    protected $fillable = [
        'title',
        'description',
        'completed'
    ];
    
    protected $casts = [
        'completed' => 'boolean',
    ];
}
```

`$fillable`配列には、一括代入を許可するカラムを指定します。また、`$casts`配列は、データベースから取得した値のキャスト（型変換）を定義します。

### 5. コントローラーの作成

次に、タスクの操作を行うコントローラーを作成します：

```bash
php artisan make:controller TaskController --resource
```

`--resource`オプションを付けると、CRUD操作のための基本的なメソッドが含まれたコントローラーが生成されます。

### 6. ルートの定義

`routes/web.php`ファイルを開き、タスク管理のためのルートを定義します：

```php
<?php

use Illuminate\Support\Facades\Route;
use App\Http\Controllers\TaskController;

// ウェルカムページ
Route::get('/', function () {
    return view('welcome');
});

// タスク管理のルート
Route::resource('tasks', TaskController::class);
```

`Route::resource`を使うことで、RESTfulなルートが自動的に定義されます。定義されるルートを確認するには、次のコマンドを実行します：

```bash
php artisan route:list
```

### 7. コントローラーの実装

生成された`app/Http/Controllers/TaskController.php`を開き、各メソッドを実装していきます：

```php
<?php

namespace App\Http\Controllers;

use App\Models\Task;
use Illuminate\Http\Request;

class TaskController extends Controller
{
    /**
     * タスク一覧を表示
     */
    public function index()
    {
        $tasks = Task::latest()->get(); // 最新順に取得
        return view('tasks.index', compact('tasks'));
    }

    /**
     * タスク作成フォームを表示
     */
    public function create()
    {
        return view('tasks.create');
    }

    /**
     * 新しいタスクを保存
     */
    public function store(Request $request)
    {
        $request->validate([
            'title' => 'required|max:255',
            'description' => 'nullable',
        ]);

        Task::create([
            'title' => $request->title,
            'description' => $request->description,
            'completed' => false,
        ]);

        return redirect()->route('tasks.index')
            ->with('success', 'タスクが正常に作成されました！');
    }

    /**
     * 個別タスクの詳細を表示
     */
    public function show(Task $task)
    {
        return view('tasks.show', compact('task'));
    }

    /**
     * タスク編集フォームを表示
     */
    public function edit(Task $task)
    {
        return view('tasks.edit', compact('task'));
    }

    /**
     * 既存タスクを更新
     */
    public function update(Request $request, Task $task)
    {
        $request->validate([
            'title' => 'required|max:255',
            'description' => 'nullable',
        ]);

        $task->update([
            'title' => $request->title,
            'description' => $request->description,
            'completed' => $request->has('completed'),
        ]);

        return redirect()->route('tasks.index')
            ->with('success', 'タスクが正常に更新されました！');
    }

    /**
     * タスクを削除
     */
    public function destroy(Task $task)
    {
        $task->delete();

        return redirect()->route('tasks.index')
            ->with('success', 'タスクが正常に削除されました！');
    }
}
```

### 8. ビューの作成

タスク管理アプリのビューを作成します。まず、ビューを格納するディレクトリを作成します：

```bash
mkdir -p resources/views/tasks
```

#### レイアウトファイルの作成

`resources/views/layouts/app.blade.php`を作成します：

```html
<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>タスク管理アプリ</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
</head>
<body>
    <nav class="navbar navbar-expand-lg navbar-dark bg-dark">
        <div class="container">
            <a class="navbar-brand" href="{{ route('tasks.index') }}">タスク管理アプリ</a>
        </div>
    </nav>

    <div class="container mt-4">
        @if(session('success'))
            <div class="alert alert-success">
                {{ session('success') }}
            </div>
        @endif

        @yield('content')
    </div>

    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/js/bootstrap.bundle.min.js"></script>
</body>
</html>
```

#### タスク一覧ビュー

`resources/views/tasks/index.blade.php`を作成します：

```html
@extends('layouts.app')

@section('content')
    <div class="d-flex justify-content-between align-items-center mb-4">
        <h1>タスク一覧</h1>
        <a href="{{ route('tasks.create') }}" class="btn btn-primary">新規タスク作成</a>
    </div>

    @if($tasks->isEmpty())
        <div class="alert alert-info">
            タスクはまだありません。新しいタスクを作成しましょう！
        </div>
    @else
        <div class="table-responsive">
            <table class="table table-striped">
                <thead>
                    <tr>
                        <th>タイトル</th>
                        <th>状態</th>
                        <th>作成日</th>
                        <th>操作</th>
                    </tr>
                </thead>
                <tbody>
                    @foreach($tasks as $task)
                        <tr>
                            <td>{{ $task->title }}</td>
                            <td>
                                @if($task->completed)
                                    <span class="badge bg-success">完了</span>
                                @else
                                    <span class="badge bg-warning">未完了</span>
                                @endif
                            </td>
                            <td>{{ $task->created_at->format('Y/m/d H:i') }}</td>
                            <td class="d-flex">
                                <a href="{{ route('tasks.show', $task->id) }}" class="btn btn-sm btn-info me-2">詳細</a>
                                <a href="{{ route('tasks.edit', $task->id) }}" class="btn btn-sm btn-primary me-2">編集</a>
                                <form action="{{ route('tasks.destroy', $task->id) }}" method="POST" onsubmit="return confirm('本当に削除しますか？');">
                                    @csrf
                                    @method('DELETE')
                                    <button type="submit" class="btn btn-sm btn-danger">削除</button>
                                </form>
                            </td>
                        </tr>
                    @endforeach
                </tbody>
            </table>
        </div>
    @endif
@endsection
```

#### タスク作成ビュー

`resources/views/tasks/create.blade.php`を作成します：

```html
@extends('layouts.app')

@section('content')
    <div class="card">
        <div class="card-header">
            <h2>新規タスク作成</h2>
        </div>
        <div class="card-body">
            <form action="{{ route('tasks.store') }}" method="POST">
                @csrf
                <div class="mb-3">
                    <label for="title" class="form-label">タイトル</label>
                    <input type="text" class="form-control @error('title') is-invalid @enderror" id="title" name="title" value="{{ old('title') }}" required>
                    @error('title')
                        <div class="invalid-feedback">{{ $message }}</div>
                    @enderror
                </div>
                <div class="mb-3">
                    <label for="description" class="form-label">詳細</label>
                    <textarea class="form-control @error('description') is-invalid @enderror" id="description" name="description" rows="3">{{ old('description') }}</textarea>
                    @error('description')
                        <div class="invalid-feedback">{{ $message }}</div>
                    @enderror
                </div>
                <div class="d-flex">
                    <button type="submit" class="btn btn-primary">保存</button>
                    <a href="{{ route('tasks.index') }}" class="btn btn-secondary ms-2">キャンセル</a>
                </div>
            </form>
        </div>
    </div>
@endsection
```

#### タスク詳細ビュー

`resources/views/tasks/show.blade.php`を作成します：

```html
@extends('layouts.app')

@section('content')
    <div class="card">
        <div class="card-header d-flex justify-content-between align-items-center">
            <h2>タスク詳細</h2>
            <a href="{{ route('tasks.index') }}" class="btn btn-secondary">戻る</a>
        </div>
        <div class="card-body">
            <div class="mb-3">
                <h5>タイトル</h5>
                <p>{{ $task->title }}</p>
            </div>
            <div class="mb-3">
                <h5>詳細</h5>
                <p>{{ $task->description ?: '詳細はありません' }}</p>
            </div>
            <div class="mb-3">
                <h5>状態</h5>
                @if($task->completed)
                    <span class="badge bg-success">完了</span>
                @else
                    <span class="badge bg-warning">未完了</span>
                @endif
            </div>
            <div class="mb-3">
                <h5>作成日時</h5>
                <p>{{ $task->created_at }}</p>
            </div>
            <div class="mb-3">
                <h5>更新日時</h5>
                <p>{{ $task->updated_at }}</p>
            </div>
            <div class="d-flex">
                <a href="{{ route('tasks.edit', $task->id) }}" class="btn btn-primary">編集</a>
                <form action="{{ route('tasks.destroy', $task->id) }}" method="POST" class="ms-2" onsubmit="return confirm('本当に削除しますか？');">
                    @csrf
                    @method('DELETE')
                    <button type="submit" class="btn btn-danger">削除</button>
                </form>
            </div>
        </div>
    </div>
@endsection
```

#### タスク編集ビュー

`resources/views/tasks/edit.blade.php`を作成します：

```html
@extends('layouts.app')

@section('content')
    <div class="card">
        <div class="card-header">
            <h2>タスク編集</h2>
        </div>
        <div class="card-body">
            <form action="{{ route('tasks.update', $task->id) }}" method="POST">
                @csrf
                @method('PUT')
                <div class="mb-3">
                    <label for="title" class="form-label">タイトル</label>
                    <input type="text" class="form-control @error('title') is-invalid @enderror" id="title" name="title" value="{{ old('title', $task->title) }}" required>
                    @error('title')
                        <div class="invalid-feedback">{{ $message }}</div>
                    @enderror
                </div>
                <div class="mb-3">
                    <label for="description" class="form-label">詳細</label>
                    <textarea class="form-control @error('description') is-invalid @enderror" id="description" name="description" rows="3">{{ old('description', $task->description) }}</textarea>
                    @error('description')
                        <div class="invalid-feedback">{{ $message }}</div>
                    @enderror
                </div>
                <div class="mb-3 form-check">
                    <input type="checkbox" class="form-check-input" id="completed" name="completed" {{ $task->completed ? 'checked' : '' }}>
                    <label class="form-check-label" for="completed">完了</label>
                </div>
                <div class="d-flex">
                    <button type="submit" class="btn btn-primary">更新</button>
                    <a href="{{ route('tasks.index') }}" class="btn btn-secondary ms-2">キャンセル</a>
                </div>
            </form>
        </div>
    </div>
@endsection
```

## アプリケーションの実行

これで、基本的なタスク管理アプリケーションの実装は完了です。アプリケーションを実行して動作を確認しましょう：

```bash
php artisan serve
```

ブラウザで`http://127.0.0.1:8000/tasks`にアクセスすると、タスク一覧画面が表示されます。ここから新規タスクの作成、詳細表示、編集、削除といった操作が行えます。

## まとめ

この記事では、Laravel初心者向けに基本的なCRUD操作を実装したタスク管理アプリケーションを作成しました。主な学習ポイントは以下の通りです：

1. Laravelプロジェクトの作成と環境構築
2. モデルとマイグレーションの定義
3. リソースコントローラーの活用
4. ビューの作成とBladeテンプレートの基本
5. CRUDの各操作（Create, Read, Update, Delete）の実装

これらの基本を理解すれば、さらに機能を追加したり、より複雑なアプリケーションを開発したりすることができるようになります。

次のステップとしては、以下のような機能を追加してみることをお勧めします：

- ユーザー認証の導入
- カテゴリー機能の追加
- タスクの期限設定
- ページネーション
- APIの実装

Laravelを使った開発は、一度基本を理解すれば驚くほど効率的に進められます。この記事が、皆さんのLaravelの学習に役立つことを願っています！

## 参考リンク

- [Laravel公式ドキュメント](https://laravel.com/docs)
- [Laravel日本語ドキュメント](https://readouble.com/laravel/)
- [Blade テンプレートエンジン](https://laravel.com/docs/blade)
- [Eloquent ORM](https://laravel.com/docs/eloquent)
