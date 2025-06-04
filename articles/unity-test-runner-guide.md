---
title: "Unity Test Runnerの使い方：初心者向け実践ガイド"
emoji: "🧪"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Unity", "テスト", "C#", "TDD", "GameDev"]
published: false
---

## ゴール(はじめに)：Unity Test Runnerでゲーム開発のテストを効率化

この記事では、Unity Test Runnerというツールを使って、Unityプロジェクトにテストを導入する方法を初心者向けに解説します。テストを書くことで、バグを早期に発見し、安定したゲーム開発が可能になります。特に複雑なゲームロジックや長期的なプロジェクトでは、テストの自動化は非常に重要です。この記事を読んで、Unity Test Runnerの基本を身につけましょう。

## やってみた結果
- ゲームロジックのバグを早期発見できるようになった
- リファクタリングが安全に行えるようになった
- チーム開発での品質管理が容易になった
- 継続的インテグレーション(CI)との連携が可能になった

この記事を読めば、Unity Test Runnerの基本的な使い方を理解し、自分のプロジェクトにテストを導入できるようになります。

## 開発環境
- Unity 2020.3以上（Unity Test Runnerは2017.3以降のバージョンに標準搭載）
- Visual Studio または Visual Studio Code
- C#の基本的な知識

## 事前準備
- Unityのインストール
- 新規または既存のUnityプロジェクト
- C#の基本構文の理解

## やったこと

### 1. Unity Test Runnerとは何か

Unity Test Runnerは、Unity内蔵のテストフレームワークで、以下の2種類のテストをサポートしています：

1. **PlayModeテスト**：ゲームの動作中に行うテスト。UIやゲームプレイの流れなど、実際のゲーム動作に近い形でテストできます。
2. **EditModeテスト**：エディタ上で行うテスト。ゲームを実行せずにロジックやデータ処理などの単体テストが可能です。

これらのテストを利用することで、コードの正確性を確認し、変更による予期せぬ影響を早期に発見できます。

### 2. Unity Test Runnerウィンドウを開く

まず、Unity Test Runnerウィンドウを開きましょう：

1. Unityエディタのメニューから「Window」→「General」→「Test Runner」を選択
2. Test Runnerウィンドウが開きます

ウィンドウには「EditMode」と「PlayMode」の2つのタブがあります。それぞれのモードでテストを作成・実行できます。

### 3. テスト用アセンブリの設定

テストコードは通常、メインのコードとは別のアセンブリに配置します。Unity Test Runnerでは、テスト用のアセンブリ定義が自動的に作成されます：

1. Test Runnerウィンドウで「Create EditMode Test Assembly Folder」または「Create PlayMode Test Assembly Folder」ボタンをクリック
2. プロジェクト内に「Tests」フォルダと対応するアセンブリ定義ファイル（.asmdef）が作成されます

この設定により、テストコードがビルド時に含まれないようになり、本番環境のパフォーマンスに影響を与えません。

### 4. 最初のEditModeテストを作成する

簡単な計算機能をテストする例を考えてみましょう。まず、テスト対象となるクラスを作成します：

```csharp
// Assets/Scripts/Calculator.cs
public class Calculator
{
    public int Add(int a, int b)
    {
        return a + b;
    }
    
    public int Subtract(int a, int b)
    {
        return a - b;
    }
}
```

次に、EditModeテストを作成します：

1. Test Runnerウィンドウの「EditMode」タブを選択
2. 「Create Test Script in current folder」ボタンをクリック
3. テストスクリプトの名前を「CalculatorTests」として保存

作成されたテストスクリプトを以下のように編集します：

```csharp
// Assets/Tests/EditMode/CalculatorTests.cs
using System.Collections;
using NUnit.Framework;
using UnityEngine.TestTools;

public class CalculatorTests
{
    [Test]
    public void Add_TwoPlusTwo_ReturnsFour()
    {
        // Arrange
        Calculator calculator = new Calculator();
        int a = 2;
        int b = 2;
        
        // Act
        int result = calculator.Add(a, b);
        
        // Assert
        Assert.AreEqual(4, result);
    }
    
    [Test]
    public void Subtract_FiveMinusThree_ReturnsTwo()
    {
        // Arrange
        Calculator calculator = new Calculator();
        int a = 5;
        int b = 3;
        
        // Act
        int result = calculator.Subtract(a, b);
        
        // Assert
        Assert.AreEqual(2, result);
    }
}
```

### 5. テストを実行する

テストを実行する方法はいくつかあります：

1. **すべてのテストを実行**：Test Runnerウィンドウの「Run All」ボタンをクリック
2. **特定のテストを実行**：実行したいテストの左側にある再生ボタンをクリック
3. **テストクラス単位で実行**：クラス名の左側にある再生ボタンをクリック

テストが成功すると緑色のチェックマークが、失敗すると赤色のバツ印が表示されます。

### 6. PlayModeテストの基本

PlayModeテストは、実際にゲームを実行した状態でテストを行います。例として、オブジェクトの動きをテストしてみましょう：

1. まず、Test Runnerウィンドウの「PlayMode」タブを選択
2. 「Create Test Script in current folder」ボタンをクリック
3. テストスクリプトの名前を「MovementTests」として保存

簡単な移動スクリプトを作成します：

```csharp
// Assets/Scripts/PlayerMovement.cs
using UnityEngine;

public class PlayerMovement : MonoBehaviour
{
    public float speed = 5f;
    
    public void MoveForward()
    {
        transform.position += Vector3.forward * speed * Time.deltaTime;
    }
}
```

PlayModeテストを以下のように編集します：

```csharp
// Assets/Tests/PlayMode/MovementTests.cs
using System.Collections;
using NUnit.Framework;
using UnityEngine;
using UnityEngine.TestTools;

public class MovementTests
{
    [UnityTest]
    public IEnumerator PlayerMovesForward_WhenMoveForwardCalled()
    {
        // Arrange
        GameObject playerObject = new GameObject("Player");
        PlayerMovement movement = playerObject.AddComponent<PlayerMovement>();
        Vector3 startPosition = playerObject.transform.position;
        
        // Act
        movement.MoveForward();
        
        // Wait for one frame
        yield return null;
        
        // Assert
        Assert.Greater(playerObject.transform.position.z, startPosition.z);
        
        // Clean up
        Object.Destroy(playerObject);
    }
}
```

PlayModeテストでは、`[UnityTest]`属性を使用し、`IEnumerator`を返す必要があります。これにより、`yield return null`でフレームの経過を待つなど、時間を要する処理のテストが可能になります。

### 7. テストのベストプラクティス

効果的なテストを書くためのいくつかのヒント：

1. **AAA（Arrange-Act-Assert）パターン**：テストを「準備」「実行」「検証」の3つのセクションに分けて書くことで、読みやすく、メンテナンスしやすいテストになります。

2. **テスト名を明確に**：テスト名は「テスト対象_条件_期待結果」の形式で書くと、何をテストしているのかが一目瞭然になります。例：`Add_NegativeNumbers_ReturnsCorrectSum`

3. **SetUpとTearDown**：複数のテストで共通の初期化・終了処理がある場合は、`[SetUp]`と`[TearDown]`属性を使用できます：

```csharp
[SetUp]
public void SetUp()
{
    // 各テスト前に実行される処理
}

[TearDown]
public void TearDown()
{
    // 各テスト後に実行される処理
}
```

4. **テストのカテゴリ分け**：`[Category("Category Name")]`属性を使用して、テストをカテゴリ分けできます。これにより、特定のカテゴリのテストのみを実行することが可能になります。

### 8. モックとスタブの活用

外部依存性（例：ネットワーク、データベース）を持つコードをテストする場合、モックやスタブが役立ちます。NSubstituteやMoqなどのライブラリをUnityプロジェクトに導入することで、依存性を模倣したテストが可能になります。

例えば、NSubstituteを使用したテスト：

```csharp
// テスト用のインターフェース
public interface IDataService
{
    int GetHighScore();
}

// テスト対象のクラス
public class ScoreManager
{
    private readonly IDataService _dataService;
    
    public ScoreManager(IDataService dataService)
    {
        _dataService = dataService;
    }
    
    public bool IsNewHighScore(int score)
    {
        return score > _dataService.GetHighScore();
    }
}

// テストコード（NSubstituteを使用）
[Test]
public void IsNewHighScore_ScoreHigherThanCurrentHighScore_ReturnsTrue()
{
    // Arrange
    var mockDataService = Substitute.For<IDataService>();
    mockDataService.GetHighScore().Returns(100);
    
    var scoreManager = new ScoreManager(mockDataService);
    
    // Act
    bool result = scoreManager.IsNewHighScore(101);
    
    // Assert
    Assert.IsTrue(result);
}
```

### 9. CI/CDとの連携

Unity Test Runnerは、コマンドラインからも実行できるため、GitHub ActionsやJenkinsなどのCI/CDツールと連携できます：

```bash
Unity -batchmode -projectPath "プロジェクトパス" -runTests -testPlatform EditMode -testResults "results.xml" -quit
```

この機能を活用すれば、プルリクエスト時に自動的にテストを実行し、品質を確保することができます。

## 補足・参考

### Unity Test Runnerを導入するメリット

1. **バグの早期発見**：コードを変更するたびにテストを実行することで、バグを早期に発見できます。

2. **リファクタリングの安全性**：既存のコードを改善する際、テストがあれば機能が損なわれていないことを確認できます。

3. **ドキュメント代わりの役割**：テストコードは、クラスやメソッドの使い方を示す良い例となります。

4. **チーム開発の効率化**：チームメンバーが書いたコードの品質を客観的に評価できます。

### 関連ツール・リソース

- [Unity Test Framework ドキュメント](https://docs.unity3d.com/Packages/com.unity.test-framework@1.1/manual/index.html)
- [NUnit ドキュメント](https://docs.nunit.org/)
- [NSubstitute (モックライブラリ)](https://nsubstitute.github.io/)
- [Unity Learn - テストドリブン開発](https://learn.unity.com/tutorial/test-driven-development-in-unity)

### テスト駆動開発(TDD)とUnity

Unity Test Runnerを使えば、テスト駆動開発(TDD)のアプローチも可能です：

1. 失敗するテストを書く
2. テストが通るように最小限のコードを書く
3. コードをリファクタリングする
4. 繰り返す

このサイクルを繰り返すことで、品質の高いコードを段階的に構築できます。

## おわりに

Unity Test Runnerを使ったテスト導入は、最初は少し手間に感じるかもしれませんが、プロジェクトが大きくなるにつれてその価値は明らかになります。バグの早期発見、安全なリファクタリング、コードの品質向上など、多くのメリットがあります。

小さなテストから始めて、徐々にテストカバレッジを増やしていきましょう。すべてをテストする必要はありませんが、重要なロジックやバグが発生しやすい部分にテストを書くことで、開発の効率と品質を大きく向上させることができます。

ゲーム開発は複雑ですが、テストを導入することでその複雑さを管理し、より安定した開発プロセスを構築できます。ぜひ、あなたの次のUnityプロジェクトでTest Runnerを活用してみてください。
