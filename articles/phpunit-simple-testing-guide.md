---
title: "PHPUnitで始める簡単なテストの書き方"
emoji: "🧪"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["php", "phpunit", "testing", "unittest"]
published: false
---

# PHPUnitで始める簡単なテストの書き方

## はじめに

PHPUnitはPHPで最も広く使われているテストフレームワークです。このフレームワークを使うことで、コードの品質を保ちながら開発を進めることができます。この記事では、PHPUnitを使った簡単なテストの書き方を紹介します。

## 環境準備

まずは、PHPUnitをインストールしましょう。Composerを使うのが最も簡単です。

```bash
composer require --dev phpunit/phpunit
```

## 基本的なテストの書き方

PHPUnitでは、テストクラスを作成し、その中にテストメソッドを定義します。各テストメソッドは、特定の機能や動作をテストします。

### シンプルな例：計算機クラスのテスト

まず、テスト対象となるCalculatorクラスを作成します。

```php
<?php
// src/Calculator.php

class Calculator
{
    public function add($a, $b)
    {
        return $a + $b;
    }
    
    public function subtract($a, $b)
    {
        return $a - $b;
    }
    
    public function multiply($a, $b)
    {
        return $a * $b;
    }
    
    public function divide($a, $b)
    {
        if ($b == 0) {
            throw new InvalidArgumentException("Division by zero");
        }
        return $a / $b;
    }
}
```

次に、このCalculatorクラスをテストするためのテストクラスを作成します。

```php
<?php
// tests/CalculatorTest.php

use PHPUnit\Framework\TestCase;

class CalculatorTest extends TestCase
{
    private $calculator;
    
    protected function setUp(): void
    {
        // 各テストの前に実行される
        $this->calculator = new Calculator();
    }
    
    public function testAdd()
    {
        $result = $this->calculator->add(5, 3);
        $this->assertEquals(8, $result);
    }
    
    public function testSubtract()
    {
        $result = $this->calculator->subtract(5, 3);
        $this->assertEquals(2, $result);
    }
    
    public function testMultiply()
    {
        $result = $this->calculator->multiply(5, 3);
        $this->assertEquals(15, $result);
    }
    
    public function testDivide()
    {
        $result = $this->calculator->divide(6, 3);
        $this->assertEquals(2, $result);
    }
    
    public function testDivideByZero()
    {
        $this->expectException(InvalidArgumentException::class);
        $this->calculator->divide(6, 0);
    }
}
```

## テストの実行方法

テストを実行するには、以下のコマンドを使用します。

```bash
./vendor/bin/phpunit tests/CalculatorTest.php
```

あるいは、PHPUnit設定ファイル（phpunit.xml）を作成している場合は、単に以下のように実行できます。

```bash
./vendor/bin/phpunit
```

## アサーションメソッド

PHPUnitには様々なアサーションメソッドがあります。以下に代表的なものをいくつか紹介します。

- `assertEquals($expected, $actual)`: 値が等しいかチェック
- `assertNotEquals($expected, $actual)`: 値が等しくないかチェック
- `assertTrue($condition)`: 条件が真かチェック
- `assertFalse($condition)`: 条件が偽かチェック
- `assertNull($actual)`: 値がnullかチェック
- `assertNotNull($actual)`: 値がnullでないかチェック
- `assertInstanceOf($expected, $actual)`: オブジェクトが期待したクラスのインスタンスかチェック
- `assertCount($expectedCount, $haystack)`: 配列やコレクションの要素数をチェック

## モックとスタブの簡単な使い方

依存関係をモック化して、テスト対象のクラスを分離することも可能です。

```php
<?php
// 依存関係のあるクラスの例
class UserRepository
{
    public function findById($id)
    {
        // 実際のデータベース処理など
    }
}

class UserService
{
    private $repository;
    
    public function __construct(UserRepository $repository)
    {
        $this->repository = $repository;
    }
    
    public function getUserName($id)
    {
        $user = $this->repository->findById($id);
        return $user ? $user->name : null;
    }
}

// モックを使ったテスト
class UserServiceTest extends TestCase
{
    public function testGetUserName()
    {
        // UserRepositoryのモックを作成
        $repositoryMock = $this->createMock(UserRepository::class);
        
        // モックの振る舞いを設定
        $user = new stdClass();
        $user->name = 'テストユーザー';
        $repositoryMock->method('findById')
                      ->with(123)
                      ->willReturn($user);
        
        // テスト対象のクラスにモックを注入
        $service = new UserService($repositoryMock);
        
        // テスト実行
        $result = $service->getUserName(123);
        
        // 検証
        $this->assertEquals('テストユーザー', $result);
    }
}
```

## まとめ

PHPUnitを使うことで、簡単にPHPコードのテストを書くことができます。基本的なアサーションから、モックやスタブを使った複雑なテストまで、様々なシナリオに対応できます。テストを書くことで、コードの品質を保ち、リファクタリングも安全に行うことができるようになります。

この記事で紹介した内容はPHPUnitの機能のごく一部です。さらに詳しく知りたい方は、[PHPUnit公式ドキュメント](https://phpunit.de/documentation.html)を参照してください。
