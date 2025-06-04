---
title: "Unity C#でのファクトリーパターン実装ガイド：初心者向け解説"
emoji: "🏭"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["unity", "csharp", "デザインパターン", "gamedev", "programming"]
published: false
---

## ゴール(はじめに)：Unity C#でファクトリーパターンを理解して実装する

この記事では、Unity C#を使ったファクトリーパターンの実装方法を初心者向けに解説します。ゲーム開発においてオブジェクトの生成を柔軟に行うためのデザインパターンとして、ファクトリーパターンは非常に有用です。敵キャラクター、アイテム、効果などの多様なオブジェクトを効率的に生成する方法を学びましょう。

## やってみた結果
- オブジェクト生成のコードを一箇所に集約できた
- 新しい敵やアイテムの追加が容易になった
- コードの可読性と保守性が向上した
- 実行時にオブジェクトを柔軟に切り替えられるようになった

この記事を読めば、ファクトリーパターンの基本概念を理解し、Unityプロジェクトで活用できるようになります。

## 開発環境
- Unity 2022.3以降（より古いバージョンでも概念は適用可能）
- C#の基本的な知識
- Visual Studioまたは Visual Studio Code

プロジェクト構成イメージ：
```
Assets/
  ├── Scripts/
  │   ├── Enemies/
  │   │   ├── Enemy.cs (基底クラス)
  │   │   ├── Slime.cs
  │   │   ├── Goblin.cs
  │   │   └── Dragon.cs
  │   ├── Factories/
  │   │   ├── EnemyFactory.cs
  │   │   └── ItemFactory.cs
  │   └── GameManager.cs
  └── Scenes/
      └── Main.cs
```

## 事前準備
- Unityプロジェクトの作成
- 基本的なC#スクリプト作成の知識

## やったこと

### 1. ファクトリーパターンの基本概念を理解する

ファクトリーパターンとは、オブジェクトの生成処理をカプセル化し、クライアントコードから分離するデザインパターンです。このパターンの主な利点は：

- オブジェクト生成を一箇所に集約する
- 具体的なクラスの実装からクライアントを切り離す
- 新しい種類のオブジェクト追加が容易になる
- コードの再利用性と保守性の向上

### 2. 基底クラス（インターフェース）の作成

まず、敵キャラクターの基底クラスを作成します。

```csharp
// Enemy.cs
using UnityEngine;

public abstract class Enemy : MonoBehaviour
{
    public float health;
    public float damage;
    
    public abstract void Attack();
    public abstract void Move();
    
    public virtual void TakeDamage(float amount)
    {
        health -= amount;
        if (health <= 0)
        {
            Die();
        }
    }
    
    protected virtual void Die()
    {
        Destroy(gameObject);
    }
}
```

### 3. 具体的なサブクラスの実装

次に、具体的な敵キャラクタークラスを実装します。

```csharp
// Slime.cs
using UnityEngine;

public class Slime : Enemy
{
    private void Awake()
    {
        health = 50f;
        damage = 5f;
    }
    
    public override void Attack()
    {
        Debug.Log("スライムが攻撃! ダメージ: " + damage);
    }
    
    public override void Move()
    {
        Debug.Log("スライムがゆっくり動く");
    }
}
```

```csharp
// Goblin.cs
using UnityEngine;

public class Goblin : Enemy
{
    private void Awake()
    {
        health = 80f;
        damage = 15f;
    }
    
    public override void Attack()
    {
        Debug.Log("ゴブリンが武器で攻撃! ダメージ: " + damage);
    }
    
    public override void Move()
    {
        Debug.Log("ゴブリンが素早く走る");
    }
}
```

```csharp
// Dragon.cs
using UnityEngine;

public class Dragon : Enemy
{
    private void Awake()
    {
        health = 200f;
        damage = 40f;
    }
    
    public override void Attack()
    {
        Debug.Log("ドラゴンが炎を吐く! ダメージ: " + damage);
    }
    
    public override void Move()
    {
        Debug.Log("ドラゴンが飛行する");
    }
}
```

### 4. ファクトリークラスの実装

敵を生成するファクトリークラスを作成します。

```csharp
// EnemyFactory.cs
using UnityEngine;

public enum EnemyType
{
    Slime,
    Goblin,
    Dragon
}

public class EnemyFactory : MonoBehaviour
{
    [SerializeField] private GameObject slimePrefab;
    [SerializeField] private GameObject goblinPrefab;
    [SerializeField] private GameObject dragonPrefab;
    
    // プレハブが設定されていることを確認
    private void Awake()
    {
        if (slimePrefab == null || goblinPrefab == null || dragonPrefab == null)
        {
            Debug.LogError("EnemyFactoryにプレハブが設定されていません");
        }
    }
    
    // 敵タイプに基づいて敵を生成するメソッド
    public Enemy CreateEnemy(EnemyType type, Vector3 position)
    {
        GameObject enemyObject = null;
        
        switch (type)
        {
            case EnemyType.Slime:
                enemyObject = Instantiate(slimePrefab, position, Quaternion.identity);
                break;
            case EnemyType.Goblin:
                enemyObject = Instantiate(goblinPrefab, position, Quaternion.identity);
                break;
            case EnemyType.Dragon:
                enemyObject = Instantiate(dragonPrefab, position, Quaternion.identity);
                break;
            default:
                Debug.LogError("不明な敵タイプ: " + type);
                return null;
        }
        
        return enemyObject.GetComponent<Enemy>();
    }
    
    // レベルに応じてランダムな敵を生成するメソッド
    public Enemy CreateRandomEnemy(int level, Vector3 position)
    {
        EnemyType type;
        
        // レベルに応じて出現確率を調整
        float random = Random.value;
        
        if (level < 5)
        {
            // レベルが低い場合はスライムが多く、ドラゴンは出現しない
            type = random < 0.7f ? EnemyType.Slime : EnemyType.Goblin;
        }
        else if (level < 10)
        {
            // 中レベルではゴブリンが多く、ドラゴンが少し出現
            if (random < 0.5f)
                type = EnemyType.Goblin;
            else if (random < 0.8f)
                type = EnemyType.Slime;
            else
                type = EnemyType.Dragon;
        }
        else
        {
            // 高レベルではドラゴンがより多く出現
            if (random < 0.4f)
                type = EnemyType.Dragon;
            else if (random < 0.7f)
                type = EnemyType.Goblin;
            else
                type = EnemyType.Slime;
        }
        
        return CreateEnemy(type, position);
    }
}
```

### 5. GameManagerでファクトリーを使用

ゲームマネージャークラスを作成し、ファクトリーを使って敵を生成します。

```csharp
// GameManager.cs
using UnityEngine;

public class GameManager : MonoBehaviour
{
    [SerializeField] private EnemyFactory enemyFactory;
    [SerializeField] private int playerLevel = 1;
    
    // UI用ボタン
    public void SpawnSlime()
    {
        Vector3 spawnPos = GetRandomSpawnPosition();
        Enemy slime = enemyFactory.CreateEnemy(EnemyType.Slime, spawnPos);
        slime.Attack();
    }
    
    public void SpawnGoblin()
    {
        Vector3 spawnPos = GetRandomSpawnPosition();
        Enemy goblin = enemyFactory.CreateEnemy(EnemyType.Goblin, spawnPos);
        goblin.Attack();
    }
    
    public void SpawnDragon()
    {
        Vector3 spawnPos = GetRandomSpawnPosition();
        Enemy dragon = enemyFactory.CreateEnemy(EnemyType.Dragon, spawnPos);
        dragon.Attack();
    }
    
    public void SpawnRandomEnemy()
    {
        Vector3 spawnPos = GetRandomSpawnPosition();
        Enemy enemy = enemyFactory.CreateRandomEnemy(playerLevel, spawnPos);
        enemy.Attack();
    }
    
    // ランダムな出現位置を取得
    private Vector3 GetRandomSpawnPosition()
    {
        float x = Random.Range(-8f, 8f);
        float z = Random.Range(-8f, 8f);
        return new Vector3(x, 0, z);
    }
    
    // プレイヤーレベルアップ（デモ用）
    public void LevelUp()
    {
        playerLevel++;
        Debug.Log("プレイヤーレベルアップ! 現在のレベル: " + playerLevel);
    }
}
```

### 6. Unityエディタでの設定

1. 各敵キャラクターのプレハブを作成します
   - 各敵クラス（Slime.cs、Goblin.cs、Dragon.cs）をアタッチした空のゲームオブジェクトを作成
   - それぞれに見た目を付けるため、プリミティブな3DオブジェクトやSprite（2D用）をアタッチ
   - プレハブとして保存

2. シーンにEnemyFactoryとGameManagerを追加します
   - 空のゲームオブジェクトを作成し、EnemyFactory.csをアタッチ
   - Inspector上で作成した敵プレハブをそれぞれスロットに割り当て
   - 別の空のゲームオブジェクトにGameManager.csをアタッチ
   - GameManagerのenemyFactoryフィールドに、作成したEnemyFactoryオブジェクトをアサイン

3. UIボタンの設定（オプション）
   - Canvas内にボタンを作成し、それぞれSpawnSlime、SpawnGoblin、SpawnDragon、SpawnRandomEnemy、LevelUpメソッドを呼び出すように設定

### 7. 実行・テスト

プレイモードで実行し、ボタンクリックで敵が生成されることを確認します。コンソールウィンドウには、生成された敵のAttackメソッドによるログが表示されるはずです。

## ファクトリーパターンの拡張

### 抽象ファクトリーパターンの実装

より複雑なシナリオでは、抽象ファクトリーパターンを使用することも検討できます。これは関連するオブジェクトのファミリーを生成するためのパターンです。

例えば、敵の難易度レベル（簡単、普通、難しい）ごとに異なるファクトリーを用意する場合：

```csharp
// IEnemyFactory.cs - 抽象ファクトリーインターフェース
using UnityEngine;

public interface IEnemyFactory
{
    Enemy CreateSlime(Vector3 position);
    Enemy CreateGoblin(Vector3 position);
    Enemy CreateDragon(Vector3 position);
}

// EasyEnemyFactory.cs - 簡単な敵を生成するファクトリー
using UnityEngine;

public class EasyEnemyFactory : MonoBehaviour, IEnemyFactory
{
    [SerializeField] private GameObject slimePrefab;
    [SerializeField] private GameObject goblinPrefab;
    [SerializeField] private GameObject dragonPrefab;
    
    public Enemy CreateSlime(Vector3 position)
    {
        GameObject enemy = Instantiate(slimePrefab, position, Quaternion.identity);
        Slime slime = enemy.GetComponent<Slime>();
        // 簡単モード用の設定
        slime.health = 30f;
        slime.damage = 3f;
        return slime;
    }
    
    public Enemy CreateGoblin(Vector3 position)
    {
        GameObject enemy = Instantiate(goblinPrefab, position, Quaternion.identity);
        Goblin goblin = enemy.GetComponent<Goblin>();
        // 簡単モード用の設定
        goblin.health = 50f;
        goblin.damage = 8f;
        return goblin;
    }
    
    public Enemy CreateDragon(Vector3 position)
    {
        GameObject enemy = Instantiate(dragonPrefab, position, Quaternion.identity);
        Dragon dragon = enemy.GetComponent<Dragon>();
        // 簡単モード用の設定
        dragon.health = 150f;
        dragon.damage = 25f;
        return dragon;
    }
}

// HardEnemyFactory.cs - 難しい敵を生成するファクトリー
// EasyEnemyFactoryと同様に実装し、パラメータを調整
```

### ファクトリーメソッドパターンの活用

ファクトリーメソッドパターンは、サブクラスにオブジェクト生成の決定を委ねるパターンです。これはゲームの波（Wave）やレベルに応じて異なる敵を生成する場合に便利です。

```csharp
// EnemyWave.cs - 敵の波の抽象基底クラス
using System.Collections;
using UnityEngine;

public abstract class EnemyWave : MonoBehaviour
{
    [SerializeField] protected EnemyFactory enemyFactory;
    [SerializeField] protected int numberOfEnemies = 5;
    [SerializeField] protected float spawnInterval = 1.5f;
    
    public IEnumerator SpawnWave()
    {
        for (int i = 0; i < numberOfEnemies; i++)
        {
            SpawnEnemy();
            yield return new WaitForSeconds(spawnInterval);
        }
    }
    
    // ファクトリーメソッド - サブクラスでオーバーライドして具体的な敵を生成
    protected abstract Enemy SpawnEnemy();
}

// SlimeWave.cs - スライムの波
using UnityEngine;

public class SlimeWave : EnemyWave
{
    protected override Enemy SpawnEnemy()
    {
        Vector3 position = GetRandomSpawnPosition();
        return enemyFactory.CreateEnemy(EnemyType.Slime, position);
    }
    
    private Vector3 GetRandomSpawnPosition()
    {
        // ランダムな位置を生成
        return new Vector3(Random.Range(-8f, 8f), 0, Random.Range(-8f, 8f));
    }
}

// MixedWave.cs - 混合の敵の波
public class MixedWave : EnemyWave
{
    protected override Enemy SpawnEnemy()
    {
        Vector3 position = GetRandomSpawnPosition();
        int randomType = Random.Range(0, 3);
        EnemyType type = (EnemyType)randomType;
        return enemyFactory.CreateEnemy(type, position);
    }
    
    private Vector3 GetRandomSpawnPosition()
    {
        return new Vector3(Random.Range(-8f, 8f), 0, Random.Range(-8f, 8f));
    }
}
```

## 補足・参考

### ファクトリーパターンをより効率的に使うためのヒント

1. **ScriptableObjectの活用**：敵の設定をScriptableObjectに分離するとより柔軟になります。

```csharp
// EnemyData.cs
using UnityEngine;

[CreateAssetMenu(fileName = "EnemyData", menuName = "Game/Enemy Data")]
public class EnemyData : ScriptableObject
{
    public string enemyName;
    public GameObject prefab;
    public float health;
    public float damage;
    public float moveSpeed;
    // その他の設定...
}

// 改良されたEnemyFactory
public class EnemyFactory : MonoBehaviour
{
    [SerializeField] private EnemyData[] enemyDataArray;
    
    public Enemy CreateEnemy(string enemyName, Vector3 position)
    {
        foreach (var data in enemyDataArray)
        {
            if (data.enemyName == enemyName)
            {
                GameObject enemyObject = Instantiate(data.prefab, position, Quaternion.identity);
                Enemy enemy = enemyObject.GetComponent<Enemy>();
                enemy.health = data.health;
                enemy.damage = data.damage;
                // その他の設定...
                return enemy;
            }
        }
        
        Debug.LogError("敵データが見つかりません: " + enemyName);
        return null;
    }
}
```

2. **オブジェクトプール**：頻繁に生成・破棄されるオブジェクト（弾丸や敵など）には、パフォーマンス向上のためにオブジェクトプールとファクトリーパターンを組み合わせることを検討してください。

### 関連情報

- [Unityのオブジェクト指向プログラミング基礎](https://learn.unity.com/tutorial/object-oriented-programming)
- [デザインパターン入門 - 結城浩](https://www.amazon.co.jp/dp/4797327030)
- [Game Programming Patterns - Robert Nystrom](https://gameprogrammingpatterns.com/)

## おわりに

ファクトリーパターンはUnity C#プロジェクトにおいて、オブジェクト生成の柔軟性と再利用性を高めるための強力なツールです。このパターンを使うことで:

- コードの可読性と保守性が向上する
- 新しいオブジェクトの追加が容易になる
- ゲームレベルや難易度に応じた柔軟なオブジェクト生成が可能になる
- テストしやすいコード構造が実現できる

今回紹介したシンプルな実装から始めて、自分のプロジェクトに合わせてカスタマイズしてみてください。ファクトリーパターンを理解すると、他のデザインパターンも学びやすくなり、より堅牢なゲーム設計に役立ちます。

次のステップとして、オブジェクトプールパターンやシングルトンパターンなど、ゲーム開発で便利な他のデザインパターンと組み合わせてみることをお勧めします。
