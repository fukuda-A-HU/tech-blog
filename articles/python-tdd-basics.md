---
title: "Pythonを用いた、テスト駆動開発の簡単な流れ"
emoji: "🧪"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["python", "tdd", "unittest", "pytest", "テスト"]
published: false
---

# Pythonを用いた、テスト駆動開発の簡単な流れ

## はじめに

テスト駆動開発（Test-Driven Development、以下TDD）は、ソフトウェア開発の手法の一つで、テストを先に書いてから実装を進めていくアプローチです。この記事では、Pythonを使ったTDDの基本的な流れを簡単な例を通して解説します。

## TDDとは？

TDDは以下のサイクルで進められます：

1. **Red**: 失敗するテストを書く
2. **Green**: テストが通るように最小限の実装をする
3. **Refactor**: コードをリファクタリングする

このサイクルを繰り返すことで、テストが常に通る状態を保ちながら、コードを進化させていきます。

## 必要な環境

Pythonのテストには様々なライブラリがありますが、今回は標準ライブラリの`unittest`と人気のあるサードパーティ製テストフレームワーク`pytest`を使います。

### 環境のセットアップ

pytestをインストールしていない場合は、以下のコマンドでインストールしましょう：

```bash
pip install pytest
```

## シンプルな例：電卓機能の実装

では、TDDの流れに沿って簡単な電卓機能を実装してみましょう。

### ステップ1：テストを書く（Red）

まず、`test_calculator.py`という名前のファイルを作成して、テストを書きます：

```python
# test_calculator.py
import unittest
from calculator import Calculator

class TestCalculator(unittest.TestCase):
    def test_add(self):
        # 準備（Arrange）
        calc = Calculator()
        
        # 実行（Act）
        result = calc.add(3, 5)
        
        # 検証（Assert）
        self.assertEqual(result, 8)

if __name__ == '__main__':
    unittest.main()
```

この時点ではまだ`calculator.py`が存在しないため、テストを実行すると失敗します。

### ステップ2：最小限の実装をする（Green）

次に、テストが通るように`calculator.py`を作成します：

```python
# calculator.py
class Calculator:
    def add(self, a, b):
        return a + b
```

これでテストを実行すると成功するはずです：

```bash
python -m unittest test_calculator.py
```

または、pytestを使う場合：

```bash
pytest test_calculator.py
```

### ステップ3：リファクタリング（Refactor）

この例では実装がシンプルなので特にリファクタリングは必要ありませんが、必要に応じてコードの品質を向上させるための変更を行います。

### TDDサイクルを続ける

次に、引き算の機能を追加してみましょう。同様にテストから始めます：

```python
# test_calculator.py に追加
def test_subtract(self):
    calc = Calculator()
    result = calc.subtract(10, 3)
    self.assertEqual(result, 7)
```

テストが失敗することを確認した後（Red）、実装を追加します：

```python
# calculator.py に追加
def subtract(self, a, b):
    return a - b
```

これでテストが通るようになります（Green）。必要に応じてリファクタリングを行います。

## より実践的な例：ToDo管理アプリ

もう少し実践的な例として、シンプルなToDoリスト管理クラスを考えてみましょう。

### テストから始める

```python
# test_todo.py
import unittest
from todo import TodoList

class TestTodoList(unittest.TestCase):
    def test_add_todo(self):
        # 準備
        todo_list = TodoList()
        
        # 実行
        todo_list.add("牛乳を買う")
        
        # 検証
        self.assertEqual(len(todo_list.items), 1)
        self.assertEqual(todo_list.items[0]["task"], "牛乳を買う")
        self.assertFalse(todo_list.items[0]["completed"])
        
    def test_complete_todo(self):
        # 準備
        todo_list = TodoList()
        todo_list.add("牛乳を買う")
        
        # 実行
        todo_list.complete(0)
        
        # 検証
        self.assertTrue(todo_list.items[0]["completed"])

if __name__ == '__main__':
    unittest.main()
```

### 実装を書く

```python
# todo.py
class TodoList:
    def __init__(self):
        self.items = []
        
    def add(self, task):
        self.items.append({
            "task": task,
            "completed": False
        })
        
    def complete(self, index):
        if 0 <= index < len(self.items):
            self.items[index]["completed"] = True
```

### リファクタリング

必要に応じてコードを整理します。例えば、エラー処理を追加したり、タスクの検索機能を実装したりすることが考えられます。

## pytestを使ったTDD

`unittest`の代わりに`pytest`を使うと、より簡潔にテストを書けます：

```python
# test_calculator_pytest.py
from calculator import Calculator

def test_add():
    calc = Calculator()
    assert calc.add(3, 5) == 8
    
def test_subtract():
    calc = Calculator()
    assert calc.subtract(10, 3) == 7
```

pytestはアサーションが簡潔に書け、セットアップや後処理も`fixture`を使って簡単に実装できるため、多くのPythonプロジェクトで採用されています。

## TDDの利点

TDDには以下のような利点があります：

1. **設計が明確になる**: 実装前にテストを書くことで、APIやインターフェースを事前に考える必要があり、より良い設計につながります。
2. **回帰バグの防止**: 自動テストがあることで、修正によって既存の機能が壊れていないことを保証できます。
3. **安心感**: テストが通っていれば、コードが期待通りに動作していることを確認できます。
4. **ドキュメントとしての役割**: テストコードはコードの使い方の例としても機能します。

## まとめ

この記事では、Pythonを使ったテスト駆動開発の基本的な流れを説明しました。TDDは初めは少し手間に感じるかもしれませんが、習慣化することで品質の高いコードを効率的に書けるようになります。

小さなプロジェクトから始めて、徐々にTDDの考え方を取り入れていくことをお勧めします。テストが先にあることで、実装の方向性が明確になり、より自信を持ってコーディングできるようになるでしょう。

## 参考リンク

- [unittest — ユニットテストフレームワーク](https://docs.python.org/ja/3/library/unittest.html)
- [pytest公式ドキュメント](https://docs.pytest.org/en/stable/)
- [Kent Beck著「テスト駆動開発」](https://www.amazon.co.jp/dp/4274217884/)
