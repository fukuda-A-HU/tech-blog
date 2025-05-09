---
title: "FlutterのMaterial3 Widget完全ガイド - 使えるウィジェットを全網羅"
emoji: "🎨"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Flutter", "Material3", "Dart", "Widget", "UI"]
published: false
---

## ゴール(はじめに)：Flutter開発者向けMaterial 3ウィジェット活用術

Googleが提唱するデザインシステム「Material Design」は、2021年10月に大幅アップデートされ「Material 3（Material You）」として生まれ変わりました。この新しいデザインシステムはパーソナライズ機能の強化、アクセシビリティの向上、そしてより多様なデバイスでの一貫したユーザー体験を提供することを目指しています。

この記事では、Flutter開発においてMaterial 3デザインを実現するために使用できるすべてのウィジェットを網羅的に解説します。Material 3ウィジェットの基本的な使い方から応用まで、初心者にもわかりやすく説明していきます。

## やってみた結果

この記事を読むことで以下の成果が得られます：

- Material 3デザインの基本概念を理解できる
- Flutter用のMaterial 3ウィジェットを分類別に把握できる
- 各ウィジェットの基本的な使い方をサンプルコードで学べる
- Material 3テーマをアプリに適用する方法がわかる
- ダークモードやダイナミックカラーにも対応したアプリの作り方を学べる

具体的なコードサンプルも交えながら解説するので、すぐにあなたのアプリにMaterial 3デザインを取り入れられるようになります。

## 開発環境

- Flutter 3.10.0以上
- Dart 3.0.0以上
- Android Studio / VS Code
- Android端末またはエミュレータ（API Level 31+でダイナミックカラーを体験可能）
- iOS端末またはシミュレータ（iOS 13以上）

Material 3は比較的新しい機能なので、最新のFlutterバージョンを使用することをお勧めします。

## 事前準備

Material 3を使うには、`pubspec.yaml`に以下の依存関係が必要です（Flutter 3.10以上なら標準で含まれています）。

```yaml
dependencies:
  flutter:
    sdk: flutter
  material_color_utilities: ^0.5.0  # Material 3のカラーユーティリティ
```

また、アプリのエントリーポイントでMaterial 3を有効にします：

```dart
void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Material 3 Demo',
      theme: ThemeData(
        // Material 3を有効化
        useMaterial3: true,
        // デフォルトのカラースキーム
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.deepPurple),
      ),
      home: const MyHomePage(),
    );
  }
}
```

それでは、Material 3ウィジェットの解説に入ります。

## Material 3の基本概念

Material 3は以下の3つの基本原則に基づいています：

1. **パーソナライズ可能**：ユーザーの好みに合わせたデザイン
2. **アダプティブ**：様々なデバイスサイズやプラットフォームに適応
3. **アクセシブル**：すべてのユーザーにとって使いやすいデザイン

FlutterでMaterial 3を使う最大の魅力は、これらの原則を簡単に実装できる点です。

### ダイナミックカラーシステム

Material 3の大きな特徴は「ダイナミックカラー」です。これはユーザーの壁紙などから抽出した色をベースにアプリの配色を自動生成する機能です。

```dart
// Android 12以上でダイナミックカラーを適用
ColorScheme colorScheme = ColorScheme.fromSeed(
  seedColor: Colors.blue, // フォールバックカラー
  brightness: Brightness.light,
);

// ダイナミックカラー対応デバイスか確認して適用する方法
ColorScheme? dynamicColorScheme;
if (Theme.of(context).platform == TargetPlatform.android) {
  // DynamicColorPluginなどのプラグインを使用して取得
  dynamicColorScheme = await DynamicColorPlugin.getColorScheme();
}
final colorScheme = dynamicColorScheme ?? 
    ColorScheme.fromSeed(seedColor: Colors.blue);
```

## Material 3ウィジェットカタログ

ここからは、FlutterのMaterial 3ウィジェットを機能別に分類して解説します。

### ナビゲーションコンポーネント

#### 1. NavigationBar

新しいMaterial 3スタイルのボトムナビゲーションバー。従来の`BottomNavigationBar`よりもモダンなデザインと柔軟なカスタマイズが可能です。

```dart
NavigationBar(
  onDestinationSelected: (int index) {
    setState(() {
      currentPageIndex = index;
    });
  },
  selectedIndex: currentPageIndex,
  destinations: const <Widget>[
    NavigationDestination(
      icon: Icon(Icons.home_outlined),
      selectedIcon: Icon(Icons.home),
      label: 'ホーム',
    ),
    NavigationDestination(
      icon: Icon(Icons.favorite_outline),
      selectedIcon: Icon(Icons.favorite),
      label: 'お気に入り',
    ),
    NavigationDestination(
      icon: Icon(Icons.person_outline),
      selectedIcon: Icon(Icons.person),
      label: 'プロフィール',
    ),
  ],
)
```

#### 2. NavigationRail

タブレットやデスクトップアプリ向けのサイドナビゲーション。画面幅に応じてレスポンシブに表示を切り替えることができます。

```dart
NavigationRail(
  selectedIndex: selectedIndex,
  onDestinationSelected: (int index) {
    setState(() {
      selectedIndex = index;
    });
  },
  labelType: NavigationRailLabelType.selected,
  destinations: const <NavigationRailDestination>[
    NavigationRailDestination(
      icon: Icon(Icons.home_outlined),
      selectedIcon: Icon(Icons.home),
      label: Text('ホーム'),
    ),
    NavigationRailDestination(
      icon: Icon(Icons.bookmark_outline),
      selectedIcon: Icon(Icons.book),
      label: Text('保存済み'),
    ),
  ],
)
```

#### 3. NavigationDrawer

スライドインするドロワーメニュー。従来の`Drawer`の代わりに使用できます。

```dart
NavigationDrawer(
  onDestinationSelected: (int index) {
    setState(() {
      selectedIndex = index;
    });
    // ドロワーを閉じる
    Navigator.pop(context);
  },
  selectedIndex: selectedIndex,
  children: <Widget>[
    const DrawerHeader(
      child: Text('アプリメニュー'),
    ),
    NavigationDrawerDestination(
      icon: Icon(Icons.home_outlined),
      label: Text('ホーム'),
    ),
    NavigationDrawerDestination(
      icon: Icon(Icons.settings_outlined),
      label: Text('設定'),
    ),
    // 区切り線
    const Divider(),
    NavigationDrawerDestination(
      icon: Icon(Icons.info_outline),
      label: Text('アプリについて'),
    ),
  ],
)
```

### アプリバー

#### 1. AppBar

アプリの上部に表示するバー。Material 3では、より洗練されたデザインになっています。

```dart
AppBar(
  title: const Text('Material 3 AppBar'),
  actions: [
    IconButton(
      icon: const Icon(Icons.search),
      onPressed: () {},
    ),
    IconButton(
      icon: const Icon(Icons.more_vert),
      onPressed: () {},
    ),
  ],
)
```

#### 2. SliverAppBar

スクロールに応じて動的に変化するAppBarです。

```dart
CustomScrollView(
  slivers: <Widget>[
    SliverAppBar.medium(
      title: const Text('Material 3 SliverAppBar'),
      actions: [
        IconButton(icon: const Icon(Icons.filter_list), onPressed: () {}),
      ],
    ),
    // 他のスライバーウィジェット
  ],
)
```

Material 3では、以下のような種類が追加されています：
- `SliverAppBar.medium` - タイトルが大きく表示されるスタイル
- `SliverAppBar.large` - さらに大きなタイトル表示

### ボタン類

#### 1. ElevatedButton

Material 3では立体感が控えめになり、角丸のデザインになりました。

```dart
ElevatedButton(
  child: const Text('標準ボタン'),
  onPressed: () {},
),
ElevatedButton.icon(
  icon: const Icon(Icons.add),
  label: const Text('アイコン付きボタン'),
  onPressed: () {},
)
```

#### 2. FilledButton

背景色が塗りつぶされたボタン。プライマリアクションに適しています。

```dart
FilledButton(
  child: const Text('塗りつぶしボタン'),
  onPressed: () {},
),
FilledButton.tonal(
  child: const Text('トーン付きボタン'),
  onPressed: () {},
)
```

#### 3. OutlinedButton

アウトラインのあるボタン。セカンダリアクションに適しています。

```dart
OutlinedButton(
  child: const Text('アウトラインボタン'),
  onPressed: () {},
)
```

#### 4. TextButton

テキストのみのボタン。低優先度のアクションに適しています。

```dart
TextButton(
  child: const Text('テキストボタン'),
  onPressed: () {},
)
```

#### 5. IconButton

アイコンのみのボタン。Material 3では複数のバリエーションが追加されました。

```dart
// 標準アイコンボタン
IconButton(
  icon: const Icon(Icons.favorite),
  onPressed: () {},
),
// 塗りつぶしアイコンボタン
IconButton.filled(
  icon: const Icon(Icons.favorite),
  onPressed: () {},
),
// トーン付きアイコンボタン
IconButton.filledTonal(
  icon: const Icon(Icons.favorite),
  onPressed: () {},
),
// アウトラインアイコンボタン
IconButton.outlined(
  icon: const Icon(Icons.favorite),
  onPressed: () {},
)
```

#### 6. FloatingActionButton

Material 3ではデザインが刷新され、小さめのサイズやアイコンとラベルを組み合わせた拡張FABも用意されています。

```dart
// 標準FAB
FloatingActionButton(
  child: const Icon(Icons.add),
  onPressed: () {},
),
// 拡張FAB
FloatingActionButton.extended(
  label: const Text('新規作成'),
  icon: const Icon(Icons.add),
  onPressed: () {},
),
// 小型FAB
FloatingActionButton.small(
  child: const Icon(Icons.add),
  onPressed: () {},
)
```

#### 7. SegmentedButton

新しく追加されたセグメント選択ボタン。複数の選択肢から選ばせるUIに便利です。

```dart
SegmentedButton<int>(
  segments: const <ButtonSegment<int>>[
    ButtonSegment<int>(value: 0, label: Text('日')),
    ButtonSegment<int>(value: 1, label: Text('週')),
    ButtonSegment<int>(value: 2, label: Text('月')),
  ],
  selected: {selectedValue},
  onSelectionChanged: (Set<int> newSelection) {
    setState(() {
      selectedValue = newSelection.first;
    });
  },
)
```

### 選択コントロール

#### 1. Checkbox

チェックボックス。Material 3ではアニメーションや色がアップデートされています。

```dart
Checkbox(
  value: isChecked,
  onChanged: (bool? value) {
    setState(() {
      isChecked = value!;
    });
  },
)
```

#### 2. Radio

ラジオボタン。排他的な選択に使用します。

```dart
Radio<String>(
  value: 'option1',
  groupValue: selectedOption,
  onChanged: (String? value) {
    setState(() {
      selectedOption = value!;
    });
  },
)
```

#### 3. Switch

オン・オフを切り替えるスイッチ。Material 3では形状が変更されています。

```dart
Switch(
  value: isOn,
  onChanged: (bool value) {
    setState(() {
      isOn = value;
    });
  },
)
```

### 入力フィールド

#### 1. TextField

テキスト入力フィールド。Material 3ではアウトラインスタイルがデフォルトになりました。

```dart
TextField(
  decoration: const InputDecoration(
    labelText: 'ユーザー名',
    hintText: 'ユーザー名を入力してください',
  ),
  onChanged: (value) {
    // 入力値の処理
  },
)
```

#### 2. FilledTextField

背景色が塗りつぶされたテキストフィールド。

```dart
TextField(
  decoration: const InputDecoration(
    labelText: 'メールアドレス',
    hintText: 'メールアドレスを入力',
    filled: true,
    border: OutlineInputBorder(
      borderRadius: BorderRadius.all(Radius.circular(8.0)),
    ),
  ),
)
```

### バッジとチップ

#### 1. Badge

通知やカウンターを表示するためのバッジ。Material 3で新しく追加されました。

```dart
Badge(
  backgroundColor: Colors.red,
  label: const Text('3'),
  child: const Icon(Icons.notifications),
)
```

#### 2. Chip

情報のかたまりを表示するためのチップ。Material 3では複数の種類が用意されています。

```dart
// 標準チップ
Chip(
  label: const Text('標準チップ'),
  avatar: const Icon(Icons.tag),
)

// フィルタチップ
FilterChip(
  label: const Text('フィルタ'),
  selected: isSelected,
  onSelected: (bool selected) {
    setState(() {
      isSelected = selected;
    });
  },
)

// 選択チップ
ChoiceChip(
  label: const Text('選択肢'),
  selected: isSelected,
  onSelected: (bool selected) {
    setState(() {
      isSelected = selected;
    });
  },
)

// インプットチップ
InputChip(
  label: const Text('タグ'),
  onDeleted: () {
    // チップを削除
  },
)
```

### カード

#### 1. Card

情報をグループ化して表示するカード。Material 3では標高（elevation）のデザインが変更され、さらに`outlined`プロパティが追加されました。

```dart
Card(
  child: Padding(
    padding: const EdgeInsets.all(16.0),
    child: Column(
      children: const <Widget>[
        ListTile(
          leading: Icon(Icons.album),
          title: Text('カードのタイトル'),
          subtitle: Text('カードのサブタイトル'),
        ),
        Divider(),
        Text('カードの本文をここに書きます。様々な情報をまとめて表示するのに適しています。'),
      ],
    ),
  ),
)
```

#### 2. ElevatedCard

影付きのカード。

```dart
Card(
  elevation: 3.0,
  child: // カードの中身
)
```

#### 3. OutlinedCard

アウトライン付きのカード（境界線があるカード）。

```dart
Card(
  elevation: 0,
  shape: RoundedRectangleBorder(
    side: BorderSide(color: Theme.of(context).colorScheme.outline),
    borderRadius: BorderRadius.circular(12),
  ),
  child: // カードの中身
)
```

### ダイアログ

#### 1. AlertDialog

ユーザーに情報を伝えたり確認を求めるダイアログ。Material 3では角丸や配色などが変更されています。

```dart
showDialog<void>(
  context: context,
  builder: (BuildContext context) {
    return AlertDialog(
      title: const Text('ダイアログタイトル'),
      content: const Text('これはMaterial 3スタイルのAlertDialogです。'),
      actions: <Widget>[
        TextButton(
          onPressed: () => Navigator.pop(context),
          child: const Text('キャンセル'),
        ),
        TextButton(
          onPressed: () {
            // 確定処理
            Navigator.pop(context);
          },
          child: const Text('OK'),
        ),
      ],
    );
  },
);
```

#### 2. SimpleDialog

複数の選択肢を提示するシンプルなダイアログ。

```dart
showDialog<void>(
  context: context,
  builder: (BuildContext context) {
    return SimpleDialog(
      title: const Text('オプションを選択'),
      children: <Widget>[
        SimpleDialogOption(
          onPressed: () { Navigator.pop(context, 'option1'); },
          child: const Text('オプション 1'),
        ),
        SimpleDialogOption(
          onPressed: () { Navigator.pop(context, 'option2'); },
          child: const Text('オプション 2'),
        ),
      ],
    );
  },
);
```

#### 3. DatePickerDialog & TimePickerDialog

日付や時間を選択するためのダイアログ。Material 3では視覚的なデザインが大きく変更されています。

```dart
// 日付選択ダイアログ
showDatePicker(
  context: context,
  initialDate: DateTime.now(),
  firstDate: DateTime(2020),
  lastDate: DateTime(2025),
);

// 時間選択ダイアログ
showTimePicker(
  context: context,
  initialTime: TimeOfDay.now(),
);
```

### スナックバーとトースト

#### 1. SnackBar

画面下部に一時的に表示される通知。Material 3では角丸と配色が変更されています。

```dart
ScaffoldMessenger.of(context).showSnackBar(
  SnackBar(
    content: const Text('これはSnackBarです'),
    action: SnackBarAction(
      label: '閉じる',
      onPressed: () {
        // アクション処理
      },
    ),
  ),
);
```

### 進行状況表示

#### 1. CircularProgressIndicator

円形のローディングインジケータ。Material 3では色や太さがアップデートされています。

```dart
const CircularProgressIndicator(),
```

#### 2. LinearProgressIndicator

直線のローディングインジケータ。

```dart
const LinearProgressIndicator(),
// 進捗率を指定する場合
LinearProgressIndicator(value: 0.7),
```

### スライダー

#### 1. Slider

値の範囲から値を選択するためのスライダー。Material 3ではスライダーのつまみのデザインが変更されています。

```dart
Slider(
  value: _currentSliderValue,
  max: 100,
  divisions: 10,
  label: _currentSliderValue.round().toString(),
  onChanged: (double value) {
    setState(() {
      _currentSliderValue = value;
    });
  },
)
```

#### 2. RangeSlider

範囲を選択するためのスライダー。

```dart
RangeSlider(
  values: _currentRangeValues,
  min: 0,
  max: 100,
  divisions: 10,
  labels: RangeLabels(
    _currentRangeValues.start.round().toString(),
    _currentRangeValues.end.round().toString(),
  ),
  onChanged: (RangeValues values) {
    setState(() {
      _currentRangeValues = values;
    });
  },
)
```

### リスト

#### 1. ListTile

リストの項目として使用されるタイル。Material 3ではレイアウトと視覚効果が更新されています。

```dart
ListTile(
  leading: const Icon(Icons.flight),
  title: const Text('東京 - 大阪'),
  subtitle: const Text('2時間30分・¥14,000'),
  trailing: const Icon(Icons.more_vert),
  onTap: () {
    // タップ時の処理
  },
)
```

### 拡張パネル

#### 1. ExpansionPanel

折りたたみ可能なコンテンツパネル。

```dart
ExpansionPanelList(
  expansionCallback: (int index, bool isExpanded) {
    setState(() {
      items[index].isExpanded = !isExpanded;
    });
  },
  children: items.map<ExpansionPanel>((Item item) {
    return ExpansionPanel(
      headerBuilder: (BuildContext context, bool isExpanded) {
        return ListTile(
          title: Text(item.headerText),
        );
      },
      body: ListTile(
        title: Text(item.expandedText),
      ),
      isExpanded: item.isExpanded,
    );
  }).toList(),
)
```

#### 2. ExpansionTile

リスト内で使用できる折りたたみ可能なタイル。

```dart
ExpansionTile(
  title: const Text('詳細を表示'),
  children: <Widget>[
    Container(
      padding: const EdgeInsets.all(16.0),
      child: const Text('ここに詳細情報を表示します。'),
    ),
  ],
)
```

### 検索

#### 1. SearchBar

Material 3で新しく追加された検索バー。従来のSearchDelegateより柔軟に使用できます。

```dart
SearchBar(
  hintText: '検索...',
  onTap: () {
    // タップ時の処理
  },
  onChanged: (String value) {
    // 検索クエリ変更時の処理
  },
  leading: const Icon(Icons.search),
)
```

#### 2. SearchView

検索のフルスクリーン表示。

```dart
SearchAnchor(
  builder: (BuildContext context, SearchController controller) {
    return SearchBar(
      controller: controller,
      onTap: () {
        controller.openView();
      },
      onChanged: () {
        controller.openView();
      },
      hintText: '検索...',
      leading: const Icon(Icons.search),
    );
  },
  suggestionsBuilder: (BuildContext context, SearchController controller) {
    // 検索候補の生成
    return List<ListTile>.generate(5, (int index) {
      final String item = 'item $index';
      return ListTile(
        title: Text(item),
        onTap: () {
          // 候補選択時の処理
          controller.closeView(item);
        },
      );
    });
  },
)
```

### その他のコンポーネント

#### 1. BottomSheet

画面下部から表示されるシート。Material 3では角丸のデザインが特徴的です。

```dart
showModalBottomSheet<void>(
  context: context,
  builder: (BuildContext context) {
    return Container(
      padding: const EdgeInsets.all(16.0),
      child: Column(
        mainAxisSize: MainAxisSize.min,
        children: <Widget>[
          const Text('ボトムシートのタイトル'),
          const SizedBox(height: 16),
          ElevatedButton(
            child: const Text('閉じる'),
            onPressed: () => Navigator.pop(context),
          ),
        ],
      ),
    );
  },
);
```

## Material 3テーマのカスタマイズ

Material 3テーマをカスタマイズする方法を紹介します。

### 1. カラースキームの設定

```dart
ThemeData(
  useMaterial3: true,
  colorScheme: ColorScheme.fromSeed(
    seedColor: Colors.green, // シードカラーを指定
    brightness: Brightness.light, // 明るさ（ライト/ダークモード）
  ),
)
```

### 2. テキストテーマの設定

```dart
ThemeData(
  useMaterial3: true,
  // カラースキーム設定
  textTheme: TextTheme(
    displayLarge: TextStyle(fontSize: 32, fontWeight: FontWeight.bold),
    bodyLarge: TextStyle(fontSize: 18, height: 1.5),
    // その他のテキストスタイル
  ),
)
```

### 3. ダークモードの対応

```dart
MaterialApp(
  theme: ThemeData(
    useMaterial3: true,
    colorScheme: ColorScheme.fromSeed(
      seedColor: Colors.blue,
      brightness: Brightness.light,
    ),
  ),
  darkTheme: ThemeData(
    useMaterial3: true,
    colorScheme: ColorScheme.fromSeed(
      seedColor: Colors.blue,
      brightness: Brightness.dark,
    ),
  ),
  // システムの設定に合わせてテーマを切り替える
  themeMode: ThemeMode.system,
)
```

## 実践的なアプリ例：ToDoアプリ

最後に、Material 3ウィジェットを実際に使ったToDoアプリの例を紹介します。

```dart
class TodoApp extends StatefulWidget {
  const TodoApp({super.key});

  @override
  _TodoAppState createState() => _TodoAppState();
}

class _TodoAppState extends State<TodoApp> {
  final List<String> _todos = <String>[];
  final TextEditingController _controller = TextEditingController();

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Material 3 ToDo'),
        actions: [
          IconButton(
            icon: const Icon(Icons.brightness_6),
            onPressed: () {
              // ダークモード切り替えの実装（例）
            },
          ),
        ],
      ),
      body: ListView.builder(
        itemCount: _todos.length,
        padding: const EdgeInsets.symmetric(vertical: 8),
        itemBuilder: (BuildContext context, int index) {
          return Dismissible(
            key: Key(_todos[index]),
            background: Container(color: Colors.red),
            onDismissed: (direction) {
              setState(() {
                _todos.removeAt(index);
              });
              ScaffoldMessenger.of(context).showSnackBar(
                SnackBar(content: Text('タスクを削除しました')),
              );
            },
            child: Card(
              margin: const EdgeInsets.symmetric(horizontal: 16, vertical: 4),
              child: ListTile(
                title: Text(_todos[index]),
                trailing: IconButton(
                  icon: const Icon(Icons.check_circle_outline),
                  onPressed: () {
                    setState(() {
                      _todos.removeAt(index);
                    });
                  },
                ),
              ),
            ),
          );
        },
      ),
      floatingActionButton: FloatingActionButton.extended(
        label: const Text('タスク追加'),
        icon: const Icon(Icons.add),
        onPressed: () => _showAddTodoDialog(context),
      ),
    );
  }

  void _showAddTodoDialog(BuildContext context) {
    showDialog<void>(
      context: context,
      builder: (BuildContext context) {
        return AlertDialog(
          title: const Text('新しいタスク'),
          content: TextField(
            controller: _controller,
            decoration: const InputDecoration(
              hintText: 'タスクを入力',
            ),
            autofocus: true,
          ),
          actions: <Widget>[
            TextButton(
              onPressed: () {
                Navigator.of(context).pop();
              },
              child: const Text('キャンセル'),
            ),
            FilledButton(
              onPressed: () {
                if (_controller.text.isNotEmpty) {
                  setState(() {
                    _todos.add(_controller.text);
                    _controller.clear();
                  });
                  Navigator.of(context).pop();
                }
              },
              child: const Text('追加'),
            ),
          ],
        );
      },
    );
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }
}
```

## 結論に対しての補足

- Material 3はまだ発展途上であり、Flutterの新しいバージョンでより多くの機能やウィジェットが追加される可能性があります。
- Android 12以降ではダイナミックカラーが使用できますが、その他のプラットフォームではシードカラーからのカラースキーム生成に制限されます。
- iOS向けに開発する場合は、Cupertinoウィジェットとの併用を検討することもあります。

## 参考リソース

- [Flutter公式ドキュメント - Material 3](https://docs.flutter.dev/release/material-3)
- [Material Design公式ガイドライン](https://m3.material.io/)
- [FlutterのMaterial 3サンプルコード](https://github.com/flutter/samples/tree/main/material_3_demo)

## おわりに

この記事では、FlutterでのMaterial 3ウィジェットの使い方を網羅的に解説しました。Material 3は、モダンでパーソナライズ可能なUIを簡単に作成できる強力なデザインシステムです。

アプリのルック&フィールを次のレベルに引き上げるため、ぜひMaterial 3を試してみてください。美しいUIはユーザー体験を大きく向上させ、あなたのアプリの価値を高めることでしょう。

最新のFlutterバージョンとMaterial 3を使って、時代に合った魅力的なアプリを開発していきましょう！
