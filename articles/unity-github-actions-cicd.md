---
title: "UnityのCI/CDをGithub Actionsで実現する方法：初心者向けガイド"
emoji: "🚀"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["unity", "github", "githubactions", "cicd", "devops"]
published: false
---

## ゴール(はじめに)：UnityプロジェクトをGitHub Actionsで自動化する

この記事では、GitHub Actionsを使ってUnityプロジェクトの
- 自動ビルド
- 自動テスト
- 自動デプロイ
を行う方法を初心者向けに解説します。手動での作業を減らし、品質を向上させながら開発サイクルを加速させましょう。

## やってみた結果
- プロジェクトが毎回自動的にビルドされるようになった
- プルリクエスト時に自動テストが走るようになった
- ミスが早期に発見できて品質が向上した
- リリース作業が大幅に簡素化された

この記事を読めば、UnityプロジェクトのCI/CD（継続的インテグレーション/継続的デリバリー）環境をGitHub Actionsで構築できるようになります。

## 開発環境
- Unity 2022.3 LTS以降
- GitHub（リポジトリ）
- 基本的なGitの知識

フォルダ構造：
```
UnityProject/
  ├── Assets/
  ├── ProjectSettings/
  ├── .github/
  │   └── workflows/
  │       ├── unity-build.yml
  │       ├── unity-test.yml
  │       └── unity-deploy.yml
  └── ...その他Unityプロジェクトファイル
```

## 事前準備
1. UnityプロジェクトがGitHubリポジトリにプッシュされていること
2. Unityライセンス（個人版でも可）を取得していること

## やったこと

### 1. GitHub Actionsのワークフロー用フォルダを作成

まず、GitHub Actionsのワークフローファイルを格納するためのフォルダを作成します。

```bash
mkdir -p .github/workflows
```

### 2. Unity用のライセンス認証とシークレットの設定

GitHub ActionsでUnityを実行するにはライセンス認証が必要です。以下のステップで設定します。

#### 2-1. Unityライセンスファイルの取得

以下のコマンドをローカルで実行して、ライセンスファイルを生成します（Unityがインストールされている環境で実行）。

```bash
# Unityを起動してライセンス認証用ファイルを生成
/path/to/Unity -batchmode -createManualActivationFile -logfile -
```

これにより`Unity_v20XX.X.alf`というファイルが生成されます。

#### 2-2. Unity手動認証ページでライセンスを取得

1. [Unity手動認証ページ](https://license.unity3d.com/manual)にアクセス
2. 生成した`.alf`ファイルをアップロード
3. ライセンスファイル（`.ulf`）をダウンロード

#### 2-3. GitHub Secretsにライセンス情報を登録

GitHubリポジトリの「Settings」→「Secrets and variables」→「Actions」で、以下の2つのシークレットを追加します。

1. `UNITY_LICENSE` - `.ulf`ファイルの内容をコピーペースト
2. `UNITY_EMAIL` - Unityのアカウントメールアドレス
3. `UNITY_PASSWORD` - Unityのアカウントパスワード（個人アクセストークンを推奨）

### 3. ビルドワークフローの作成

`.github/workflows/unity-build.yml`ファイルを作成し、以下の内容を記述します。

```yaml
name: Unity Build

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main, develop ]

jobs:
  build:
    name: Build Unity Project
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Repository
        uses: actions/checkout@v3
        with:
          lfs: true
          
      - name: Cache Library
        uses: actions/cache@v3
        with:
          path: Library
          key: Library-${{ hashFiles('Assets/**', 'Packages/**', 'ProjectSettings/**') }}
          restore-keys: |
            Library-
      
      - name: Build Unity Project
        uses: game-ci/unity-builder@v2
        env:
          UNITY_LICENSE: ${{ secrets.UNITY_LICENSE }}
          UNITY_EMAIL: ${{ secrets.UNITY_EMAIL }}
          UNITY_PASSWORD: ${{ secrets.UNITY_PASSWORD }}
        with:
          targetPlatform: WebGL
          
      - name: Upload Build
        uses: actions/upload-artifact@v3
        with:
          name: Build
          path: build
```

このワークフローは:
- メインブランチとdevelopブランチへのプッシュ時、およびプルリクエスト時に実行されます
- UnityプロジェクトをWebGL向けにビルドします
- ビルド結果をGitHub Artifactsに保存します

### 4. テストワークフローの作成

`.github/workflows/unity-test.yml`ファイルを作成し、以下の内容を記述します。

```yaml
name: Unity Test

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main, develop ]

jobs:
  test:
    name: Run Unity Tests
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Repository
        uses: actions/checkout@v3
        with:
          lfs: true
          
      - name: Cache Library
        uses: actions/cache@v3
        with:
          path: Library
          key: Library-Test-${{ hashFiles('Assets/**', 'Packages/**', 'ProjectSettings/**') }}
          restore-keys: |
            Library-Test-
            
      - name: Run Tests
        uses: game-ci/unity-test-runner@v2
        env:
          UNITY_LICENSE: ${{ secrets.UNITY_LICENSE }}
          UNITY_EMAIL: ${{ secrets.UNITY_EMAIL }}
          UNITY_PASSWORD: ${{ secrets.UNITY_PASSWORD }}
        with:
          testMode: all
          
      - name: Upload Test Results
        uses: actions/upload-artifact@v3
        if: always()
        with:
          name: Test Results
          path: artifacts
```

このワークフローは:
- プロジェクト内の全てのテストを実行します
- テスト結果をGitHub Artifactsに保存します

### 5. デプロイワークフローの作成

`.github/workflows/unity-deploy.yml`ファイルを作成し、以下の内容を記述します。

```yaml
name: Unity Deploy

on:
  release:
    types: [created]

jobs:
  build:
    name: Build and Deploy
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Repository
        uses: actions/checkout@v3
        with:
          lfs: true
          
      - name: Cache Library
        uses: actions/cache@v3
        with:
          path: Library
          key: Library-Deploy-${{ hashFiles('Assets/**', 'Packages/**', 'ProjectSettings/**') }}
          restore-keys: |
            Library-Deploy-
            Library-
            
      - name: Build Unity Project
        uses: game-ci/unity-builder@v2
        env:
          UNITY_LICENSE: ${{ secrets.UNITY_LICENSE }}
          UNITY_EMAIL: ${{ secrets.UNITY_EMAIL }}
          UNITY_PASSWORD: ${{ secrets.UNITY_PASSWORD }}
        with:
          targetPlatform: WebGL
          
      - name: Deploy to GitHub Pages
        uses: JamesIves/github-pages-deploy-action@v4
        with:
          folder: build/WebGL/WebGL
          branch: gh-pages
          clean: true
```

このワークフローは:
- GitHubでリリースが作成されたときに実行されます
- プロジェクトをWebGLでビルドし、GitHub Pagesに自動デプロイします

### 6. ワークフローのカスタマイズ

必要に応じてビルドプラットフォームやオプションをカスタマイズできます。例えば、Windows向けビルド:

```yaml
# unity-build.ymlのtargetPlatformを変更
targetPlatform: StandaloneWindows64
```

また、Android向けビルドも可能です（署名が必要）:

```yaml
# unity-build.ymlのwithセクションに追加
androidVersionCode: 1
androidKeystoreName: user.keystore
androidKeystoreBase64: ${{ secrets.ANDROID_KEYSTORE_BASE64 }}
androidKeystorePass: ${{ secrets.ANDROID_KEYSTORE_PASS }}
androidKeyaliasName: key
androidKeyaliasPass: ${{ secrets.ANDROID_KEYALIAS_PASS }}
```

### 7. プロジェクトをプッシュしワークフローを実行

設定が完了したら、変更をプッシュして自動ワークフローが実行されるのを確認します。

```bash
git add .
git commit -m "Add GitHub Actions CI/CD workflows"
git push
```

プッシュ後、GitHubリポジトリの「Actions」タブでワークフローの実行状況を確認できます。

## Unity CI/CDのベストプラクティス

### テストを書く習慣を身につける

CI/CDの最大の利点の一つは自動テストにあります。Unity Test Framework (UTF)を使って:

```csharp
using UnityEngine;
using UnityEngine.TestTools;
using NUnit.Framework;
using System.Collections;

public class PlayerTests
{
    [Test]
    public void PlayerInitialHealthShouldBeFull()
    {
        var player = new GameObject().AddComponent<Player>();
        Assert.AreEqual(100, player.Health);
    }
    
    [UnityTest]
    public IEnumerator PlayerTakesDamageCorrectly()
    {
        var player = new GameObject().AddComponent<Player>();
        player.TakeDamage(20);
        yield return null;
        Assert.AreEqual(80, player.Health);
    }
}
```

### キャッシュを活用して実行時間を短縮

GitHub Actionsでは、Libraryフォルダのキャッシュが非常に重要です。上記のワークフローではすでに設定していますが、キャッシュの有無でビルド時間が大幅に変わります。

### GitHubのプルリクエストを活用

プルリクエストでの自動テストとビルドにより、メインブランチに問題のあるコードがマージされるのを防ぎます。

## トラブルシューティング

### ビルドに失敗する場合

以下を確認してください:
- リポジトリに必要なアセットが含まれているか
- Git LFS（Large File Storage）が正しく設定されているか
- ライセンス情報が正しく設定されているか

### テストに失敗する場合

- エラーログを確認して具体的な問題を特定
- ローカル環境で再現するかチェック
- PlayModeとEditModeのテストを区別して実行

## 補足・参考

- [Unity用GitHub Actions公式ページ（game-ci）](https://game.ci)
- [GitHub Actions公式ドキュメント](https://docs.github.com/en/actions)
- [Unity Test Framework公式ドキュメント](https://docs.unity3d.com/Packages/com.unity.test-framework@1.1/manual/index.html)

Unity Test Frameworkの導入には、Package Managerから「Test Framework」パッケージをインストールしてください。

## おわりに

GitHub ActionsでUnityプロジェクトのCI/CDを構築することで、開発フローが大幅に改善されます。自動ビルド・テスト・デプロイにより、開発者はコードの品質向上に集中でき、リリースプロセスも簡素化されます。

初期設定は少し手間がかかりますが、一度設定すれば長期的に大きなメリットがあります。ぜひ自分のプロジェクトにもCI/CDを取り入れて、開発の効率化と品質向上を実現してみてください！
