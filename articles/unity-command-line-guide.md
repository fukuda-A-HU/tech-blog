---
title: "Unityをコマンドラインからプロジェクト作成＆プロジェクトを開く方法"
emoji: "🎮"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Unity", "CommandLine", "CLI", "GameDev", "Automation"]
published: false
---

## ゴール(はじめに)：UnityをGUIなしでコマンドラインから操作する

この記事では、Unityエディタをコマンドラインから操作し、プロジェクトの作成や既存プロジェクトを開く方法について解説します。コマンドラインからの操作は、CI/CDパイプラインの構築やバッチ処理の自動化、ヘッドレスサーバーでの作業など、様々な場面で役立ちます。特にチーム開発やビルド自動化の現場では、この知識が開発効率を大きく向上させます。

## やってみた結果
- Unityプロジェクトの作成を自動化できるようになった
- スクリプトからUnityプロジェクトを開いて編集できるようになった
- CI/CDパイプラインにUnityプロセスを組み込めるようになった
- バッチ処理による複数プロジェクトの一括更新ができるようになった
- GUIを使わずにUnityの様々な機能を利用できるようになった

## 開発環境
- Unity 2020.3以上（他のバージョンでもコマンド体系は似ています）
- Windows、macOS、またはLinux（コマンドの一部は OS によって異なります）
- コマンドライン（ターミナル、PowerShell、コマンドプロンプトなど）

## 事前準備
- Unityのインストール
- Unityのインストール場所の確認
- （オプション）Unityのバージョン管理ツール「Unity Hub」のインストール

## やったこと

### 1. Unityコマンドラインインターフェースの基本

Unityは、GUIエディタだけでなく、コマンドラインからも操作できます。これを「Unity Command Line Interface (CLI)」と呼びます。

Unityのコマンドラインインターフェースを使用するには、Unity Editorの実行可能ファイルに直接コマンドライン引数を渡します。

#### Unityの実行ファイルの場所

OS別のデフォルトの場所は以下の通りです：

- **Windows**: `C:\Program Files\Unity\Hub\Editor\{バージョン}\Editor\Unity.exe`
- **macOS**: `/Applications/Unity/Hub/Editor/{バージョン}/Unity.app/Contents/MacOS/Unity`
- **Linux**: `/opt/unity/editor/{バージョン}/Editor/Unity`

Unity Hubを使用している場合は、インストールしたバージョン別のフォルダにUnityが配置されています。

### 2. コマンドラインからUnityプロジェクトを作成する

新しいUnityプロジェクトを作成するには、`-createProject`引数を使用します。

```bash
# 基本的な構文
Unity -createProject "プロジェクトパス" -projectName "プロジェクト名"

# Windows での例
"C:\Program Files\Unity\Hub\Editor\2021.3.10f1\Editor\Unity.exe" -createProject "C:\Projects\MyNewGame"

# macOS での例
/Applications/Unity/Hub/Editor/2021.3.10f1/Unity.app/Contents/MacOS/Unity -createProject ~/Projects/MyNewGame

# Linux での例
/opt/unity/editor/2021.3.10f1/Editor/Unity -createProject ~/Projects/MyNewGame
```

#### テンプレートを指定してプロジェクトを作成

特定のテンプレートを使用してプロジェクトを作成することもできます：

```bash
Unity -createProject "プロジェクトパス" -templatePath "テンプレートパス"
```

例えば、3Dテンプレートを使用する場合：

```bash
Unity -createProject "C:\Projects\My3DGame" -templatePath "Assets/Templates/3D"
```

### 3. 既存のUnityプロジェクトを開く

既存のプロジェクトをコマンドラインから開くには、`-projectPath`引数を使用します：

```bash
# 基本構文
Unity -projectPath "プロジェクトパス"

# Windows での例
"C:\Program Files\Unity\Hub\Editor\2021.3.10f1\Editor\Unity.exe" -projectPath "C:\Projects\MyExistingGame"

# macOS での例
/Applications/Unity/Hub/Editor/2021.3.10f1/Unity.app/Contents/MacOS/Unity -projectPath ~/Projects/MyExistingGame

# Linux での例
/opt/unity/editor/2021.3.10f1/Editor/Unity -projectPath ~/Projects/MyExistingGame
```

### 4. バッチモードでUnityを実行する

GUIを表示せずにUnityを実行する「バッチモード」は、自動化やCIなどで特に役立ちます。`-batchmode`引数を使用します：

```bash
Unity -batchmode -projectPath "プロジェクトパス" -logFile "ログファイルパス"
```

バッチモードでは通常、何らかの処理を実行させたいので、追加の引数を指定します：

```bash
# プロジェクトをビルドする例
Unity -batchmode -projectPath "C:\Projects\MyGame" -executeMethod BuildScript.BuildForWindows -logFile build.log -quit
```

この例では、`BuildScript`クラスの`BuildForWindows`メソッドを実行しています。

### 5. エディタスクリプトを実行する

特定のエディタスクリプトのメソッドを実行するには、`-executeMethod`引数を使用します：

```bash
Unity -projectPath "プロジェクトパス" -executeMethod MyEditorClass.MyStaticMethod
```

この機能を使うには、`Editor`フォルダ内に静的メソッドを持つスクリプトを作成する必要があります。例えば：

```csharp
// Assets/Editor/BuildScript.cs
using UnityEditor;
using UnityEngine;

public class BuildScript
{
    // 静的メソッドにする必要があります
    public static void BuildForWindows()
    {
        Debug.Log("Windows向けビルドを開始します");
        
        // ビルド設定
        BuildPlayerOptions buildPlayerOptions = new BuildPlayerOptions
        {
            scenes = new[] { "Assets/Scenes/MainScene.unity" },
            targetGroup = BuildTargetGroup.Standalone,
            target = BuildTarget.StandaloneWindows64,
            locationPathName = "Builds/Windows/MyGame.exe",
            options = BuildOptions.None
        };
        
        // ビルド実行
        BuildPipeline.BuildPlayer(buildPlayerOptions);
        
        Debug.Log("ビルドが完了しました");
    }
}
```

このスクリプトを以下のコマンドで実行します：

```bash
Unity -batchmode -projectPath "C:\Projects\MyGame" -executeMethod BuildScript.BuildForWindows -quit
```

### 6. よく使用するコマンドラインオプション

Unityで頻繁に使われるコマンドラインオプションをまとめました：

| オプション | 説明 |
|------------|------|
| `-batchmode` | GUIなしでUnityを実行します |
| `-projectPath` | 開くUnityプロジェクトのパスを指定します |
| `-createProject` | 新しいプロジェクトを作成します |
| `-quit` | 処理完了後にUnityを終了します |
| `-logFile` | ログ出力先のファイルパスを指定します |
| `-executeMethod` | 実行する静的メソッドを指定します |
| `-buildTarget` | ビルドターゲットを指定します（iOS, Android, WebGLなど） |
| `-nographics` | グラフィックスAPIを使用せずに起動します |
| `-username` | Unityアカウントのユーザー名を指定します |
| `-password` | Unityアカウントのパスワードを指定します |
| `-serial` | Unityライセンスのシリアルナンバーを指定します |

### 7. 実践的なコマンドラインスクリプト例

日常的な作業を自動化するスクリプト例をいくつか紹介します。

#### 複数のプラットフォーム向けに連続ビルドするスクリプト（Windows）

```batch
@echo off
echo Unityプロジェクトの複数プラットフォーム向けビルドを開始します...

set UNITY_PATH="C:\Program Files\Unity\Hub\Editor\2021.3.10f1\Editor\Unity.exe"
set PROJECT_PATH="C:\Projects\MyGame"

echo Windows向けビルドを開始...
%UNITY_PATH% -batchmode -projectPath %PROJECT_PATH% -executeMethod BuildScript.BuildForWindows -logFile build_win.log -quit

echo WebGL向けビルドを開始...
%UNITY_PATH% -batchmode -projectPath %PROJECT_PATH% -executeMethod BuildScript.BuildForWebGL -logFile build_webgl.log -quit

echo Android向けビルドを開始...
%UNITY_PATH% -batchmode -projectPath %PROJECT_PATH% -executeMethod BuildScript.BuildForAndroid -logFile build_android.log -quit

echo 全てのビルドが完了しました。
```

#### 複数のプラットフォーム向けに連続ビルドするスクリプト（macOS/Linux）

```bash
#!/bin/bash
echo "Unityプロジェクトの複数プラットフォーム向けビルドを開始します..."

UNITY_PATH="/Applications/Unity/Hub/Editor/2021.3.10f1/Unity.app/Contents/MacOS/Unity"
PROJECT_PATH="$HOME/Projects/MyGame"

echo "macOS向けビルドを開始..."
"$UNITY_PATH" -batchmode -projectPath "$PROJECT_PATH" -executeMethod BuildScript.BuildForMac -logFile build_mac.log -quit

echo "WebGL向けビルドを開始..."
"$UNITY_PATH" -batchmode -projectPath "$PROJECT_PATH" -executeMethod BuildScript.BuildForWebGL -logFile build_webgl.log -quit

echo "iOS向けビルドを開始..."
"$UNITY_PATH" -batchmode -projectPath "$PROJECT_PATH" -executeMethod BuildScript.BuildForIOS -logFile build_ios.log -quit

echo "全てのビルドが完了しました。"
```

### 8. Unity Hubを使ったコマンドライン操作

Unity Hubもコマンドラインから操作できます。これにより、さらに柔軟なUnityのバージョン管理が可能になります。

#### Unity Hubの実行ファイルの場所

- **Windows**: `C:\Program Files\Unity Hub\Unity Hub.exe`
- **macOS**: `/Applications/Unity Hub.app/Contents/MacOS/Unity Hub`
- **Linux**: `/opt/unityhub/unityhub`

#### Unity Hubでエディタを起動する

```bash
# Windows
"C:\Program Files\Unity Hub\Unity Hub.exe" -- --headless editors -i

# macOS/Linux
/Applications/Unity\ Hub.app/Contents/MacOS/Unity\ Hub -- --headless editors -i
```

#### Unity Hubで特定バージョンのエディタをインストールする

```bash
# Windows
"C:\Program Files\Unity Hub\Unity Hub.exe" -- --headless install --version 2021.3.10f1 --changeset f03b68e11893

# macOS/Linux
/Applications/Unity\ Hub.app/Contents/MacOS/Unity\ Hub -- --headless install --version 2021.3.10f1 --changeset f03b68e11893
```

## 補足・参考

### コマンドライン操作のメリット

1. **自動化**: ビルドやテスト、アセットのインポートなどを自動化できます
2. **CI/CD統合**: Jenkins、GitHub Actions、GitLab CIなどと統合が容易です
3. **バッチ処理**: 多数のプロジェクトに対して同じ操作を一括適用できます
4. **ヘッドレス環境**: GUIのないサーバー環境でも操作できます
5. **再現性**: コマンドをスクリプト化することで、同じ環境を簡単に再現できます

### セキュリティに関する注意点

1. パスワードやライセンスキーをコマンドラインで直接指定する場合は注意が必要です。環境変数や安全な認証情報管理を検討しましょう。
2. CI/CD環境では、シークレット変数としてパスワードを保存することをおすすめします。

### トラブルシューティング

1. **コマンドが見つからない**: Unityのパスが正しく設定されているか確認してください。
2. **メソッドが見つからない**: 指定した静的メソッドの名前とクラス名が正しいか、そのクラスがEditorフォルダ内にあるかを確認してください。
3. **ライセンスエラー**: バッチモードでも有効なUnityライセンスが必要です。ライセンスの認証状況を確認してください。
4. **ログを確認**: `-logFile`オプションで指定したファイルにエラー情報が記録されているはずです。

### 関連リソース

- [Unity公式マニュアル - Command line arguments](https://docs.unity3d.com/Manual/CommandLineArguments.html)
- [Unity公式マニュアル - Building from the command line](https://docs.unity3d.com/Manual/CommandLineArguments.html)
- [Unity Learn - Continuous integration](https://learn.unity.com/tutorial/continuous-integration)
- [GitHub - Unity-CI](https://github.com/Unity-CI/unity3d-ci-example) - UnityプロジェクトのCI設定例

## おわりに

コマンドラインからUnityを操作することで、開発プロセスの多くの部分を自動化できることがわかりました。特に、チーム開発やCI/CD環境では、この知識が大きな時間節約につながります。

最初はコマンドラインオプションを覚えるのが大変に感じるかもしれませんが、一度スクリプト化してしまえば繰り返し利用でき、長い目で見ると大きな効率化になります。

また、今回紹介した基本的なコマンドから発展させて、より複雑な自動化スクリプトを作成することも可能です。例えば、テスト自動化やアセットの自動処理、複数環境でのクロスプラットフォームビルドなど、様々なケースに応用できます。

Unityの開発においてコマンドライン操作をマスターすることで、より効率的で再現性の高い開発ワークフローを構築しましょう。
