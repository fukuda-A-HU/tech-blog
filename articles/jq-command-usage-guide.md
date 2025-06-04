---
title: "初心者でもわかる！jqコマンドの使い方入門"
emoji: "🔍"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["jq", "JSON", "CLI", "Linux", "Shell"]
published: false
---

## ゴール(はじめに)：JSONデータを自在に操るjqコマンドをマスターする

コマンドライン上でJSONデータを扱う必要があるとき、単純な`grep`や`sed`では複雑な構造を持つJSONを適切に処理するのは困難です。そこで役立つのが「jq」コマンドです。jqはJSONデータを効率的に処理できる強力なツールで、複雑なJSONからデータを抽出、加工、整形することができます。

この記事では、jqコマンドの基本的な使い方から実践的な例まで、JSONデータ操作の基礎を初心者にもわかりやすく解説します。これを読めば、コマンドライン上でのJSONデータ処理が格段に効率化されるでしょう。

## やってみた結果

この記事を読むことで、次のことができるようになります：

- jqコマンドの基本的な使い方を理解できる
- JSONデータからの特定の値の抽出方法を習得できる
- フィルタリングやマッピングなどの操作が行えるようになる
- 複雑なJSON変換処理をコマンドライン上で実現できる
- APIレスポンスなどのJSONデータを効率的に処理できる

## 開発環境

- Linux、macOS、またはWindowsのコマンドライン環境（WSL、Git Bashなど）
- jqコマンド（インストール方法は後述）
- 基本的なコマンドラインの知識

## 事前準備

この記事を読み進める前に、以下の知識があると理解が深まります：

1. 基本的なコマンドライン操作の知識
2. JSONの基本構造についての理解

JSONについては、キー・バリューのペアで構成される`{}`と配列を表す`[]`の違いを理解していれば十分です。

## jqコマンドのインストール

まずはjqコマンドをインストールしましょう。

### Linux (Debian/Ubuntu)

```bash
sudo apt-get update
sudo apt-get install jq
```

### Linux (CentOS/RHEL)

```bash
sudo yum install jq
```

### macOS (Homebrew)

```bash
brew install jq
```

### Windows

Windows環境では、[Chocolatey](https://chocolatey.org/)を使用してインストールできます：

```bash
choco install jq
```

または、[公式サイト](https://stedolan.github.io/jq/download/)からバイナリをダウンロードすることもできます。

### インストールの確認

インストールが完了したら、バージョンを確認して正常にインストールされているか確認しましょう：

```bash
jq --version
```

バージョン番号が表示されれば、インストールは成功です。

## jqの基本的な使い方

jqコマンドは、基本的に以下の形式で使用します：

```bash
jq 'フィルタ式' JSONファイル
```

または、パイプを使って：

```bash
cat JSONファイル | jq 'フィルタ式'
```

### シンプルな例：JSONの整形表示

最も単純なjqの使い方はJSONデータの整形です。以下のようにドット(`.`)を使用します：

```bash
echo '{"name":"山田太郎","age":30,"email":"yamada@example.com"}' | jq '.'
```

結果：
```json
{
  "name": "山田太郎",
  "age": 30,
  "email": "yamada@example.com"
}
```

これだけでも、読みやすい色付きで整形されたJSONを表示できます。

## 基本的なフィルタ操作

### 特定のフィールドを抽出する

特定のキーの値を取得するには、ドットの後にキー名を指定します：

```bash
echo '{"name":"山田太郎","age":30,"email":"yamada@example.com"}' | jq '.name'
```

結果：
```
"山田太郎"
```

### 複数のフィールドを抽出する

複数のフィールドを抽出する場合は、中括弧`{}`を使います：

```bash
echo '{"name":"山田太郎","age":30,"email":"yamada@example.com"}' | jq '{name: .name, email: .email}'
```

結果：
```json
{
  "name": "山田太郎",
  "email": "yamada@example.com"
}
```

### 配列からのデータ抽出

配列の要素にアクセスするには、インデックスを指定します：

```bash
echo '[{"id":1,"name":"アイテム1"},{"id":2,"name":"アイテム2"}]' | jq '.[0]'
```

結果：
```json
{
  "id": 1,
  "name": "アイテム1"
}
```

配列のすべての要素から特定のフィールドを抽出したい場合は、以下のようにします：

```bash
echo '[{"id":1,"name":"アイテム1"},{"id":2,"name":"アイテム2"}]' | jq '.[].name'
```

結果：
```
"アイテム1"
"アイテム2"
```

## 配列の操作

### 配列のフィルタリング

配列から条件に合う要素だけを抽出するには、`select`関数を使用します：

```bash
echo '[{"id":1,"price":100},{"id":2,"price":200},{"id":3,"price":50}]' | jq '.[] | select(.price > 100)'
```

結果：
```json
{
  "id": 2,
  "price": 200
}
```

### 配列の変換（map）

配列の各要素に対して同じ操作を適用するには、`map`関数を使います：

```bash
echo '[1,2,3,4,5]' | jq 'map(. * 2)'
```

結果：
```json
[
  2,
  4,
  6,
  8,
  10
]
```

あるいは、パイプとリスト構文を使って：

```bash
echo '[1,2,3,4,5]' | jq '[.[] | . * 2]'
```

## 実践的な例

### APIレスポンスの処理

実際のAPI呼び出しからのJSONレスポンスを処理する例を示します。以下のようなAPI呼び出しがあるとします：

```bash
curl -s 'https://jsonplaceholder.typicode.com/users' | jq '.'
```

ユーザーの名前とメールだけの配列を作成するには：

```bash
curl -s 'https://jsonplaceholder.typicode.com/users' | jq '[.[] | {name: .name, email: .email}]'
```

特定の条件に合うユーザーだけをフィルタリングするには：

```bash
curl -s 'https://jsonplaceholder.typicode.com/users' | jq '.[] | select(.username | contains("Bret"))'
```

### ファイルからのJSONデータ操作

JSONファイルがある場合は、直接ファイルパスを指定できます：

```bash
jq '.users[] | select(.age > 30) | .name' users.json
```

### データの集計

数値の合計や平均を計算する例：

```bash
echo '[{"name":"商品A","price":100},{"name":"商品B","price":200},{"name":"商品C","price":300}]' | jq '[.[].price] | add'
```

結果：
```
600
```

## 高度な使い方

### 条件分岐

`if-then-else`を使った条件分岐も可能です：

```bash
echo '[{"name":"A","stock":0},{"name":"B","stock":5}]' | jq '.[] | {name: .name, status: (if .stock > 0 then "在庫あり" else "在庫なし" end)}'
```

結果：
```json
{
  "name": "A",
  "status": "在庫なし"
}
{
  "name": "B",
  "status": "在庫あり"
}
```

### 複雑なJSONの変換

入れ子になったJSONデータも簡単に操作できます：

```bash
echo '{"user":{"personal":{"name":"田中","age":25},"office":{"department":"開発部","position":"エンジニア"}}}' | jq '.user.personal.name'
```

結果：
```
"田中"
```

### 一時変数の使用

複雑な処理では一時変数を定義すると便利です：

```bash
echo '{"items":[{"id":"A","value":10},{"id":"B","value":20}]}' | jq 'def get_value(id): .items[] | select(.id == id).value; get_value("B")'
```

結果：
```
20
```

## よく使うjqコマンドのチートシート

以下によく使うjqのパターンをまとめました：

| 操作 | コマンド例 |
|------|---------|
| JSON整形 | `jq '.'` |
| 特定キーの抽出 | `jq '.key'` |
| 配列の要素アクセス | `jq '.[0]'` |
| 配列の全要素に適用 | `jq '.[]'` |
| 配列から特定キーだけ抽出 | `jq '.[].key'` |
| 条件フィルタリング | `jq '.[] | select(.key > value)'` |
| 複数フィールドの抽出 | `jq '{key1: .key1, key2: .key2}'` |
| 配列の変換 | `jq '[.[] | .key]'` |
| 要素数のカウント | `jq 'length'` |
| キーの取得 | `jq 'keys'` |

## 結論に対しての補足

### 注意点

- 巨大なJSONファイルを処理する場合、メモリ使用量に注意が必要です
- 複雑なフィルタリングは、複数のパイプを分けて段階的に処理すると理解しやすくなります
- シングルクォート（'）とダブルクォート（"）の使い分けに注意しましょう

### 関連ツールとリソース

- [jq 公式ドキュメント](https://stedolan.github.io/jq/manual/)
- [jq Play](https://jqplay.org/) - オンラインでjqを試せるプレイグラウンド
- [gron](https://github.com/tomnomnom/gron) - JSONをよりフラットな形式に変換するツール
- [JMESPath](https://jmespath.org/) - 別のJSONクエリ言語

## おわりに

jqコマンドは、一見複雑に見えるかもしれませんが、基本的なパターンをマスターするだけでも、JSONデータ処理の効率は格段に向上します。特にAPI開発やデータ分析、システム運用などの場面では非常に役立つでしょう。

本記事で紹介した例を参考に、実際のユースケースで試してみることをお勧めします。少しずつ機能を試していくうちに、jqの強力さと使いやすさを実感できるはずです。

コマンドライン上でのJSON処理で困ったとき、「そういえばjqでできたな」と思い出してもらえれば幸いです。

次のステップとして、より複雑なJSONデータを扱うときのjqの活用法や、他のコマンドラインツールと組み合わせた使い方にもチャレンジしてみてください。jqの習得は、あなたのデータ処理スキルを確実に向上させるでしょう。
