---
title: "UnityでUPMパッケージを用いて別のプロジェクトとコードを共有する方法"
emoji: "📦"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Unity", "UPM", "GameDev", "CSharp", "パッケージ管理"]
published: false
---

## ゴール(はじめに)：UnityプロジェクトでUPMパッケージを活用してコード共有を効率化する

この記事では、Unity Package Manager（UPM）を使って、複数のUnityプロジェクト間でコードやアセットを簡単に共有する方法を解説します。大規模な開発やチーム開発では、共通のコードやツールを複数のプロジェクトで再利用することが重要です。UPMパッケージを作成・公開することで、コードの重複を避け、メンテナンスの手間を減らし、効率的な開発ワークフローを構築できます。

## やってみた結果
- 異なるプロジェクト間で共通のコードを簡単に共有できるようになった
- コードの変更が全プロジェクトに自動的に反映されるようになった
- コードの再利用性と管理性が大幅に向上した
- チーム内での共同開発が効率化された
- 開発プロセスが標準化され、品質が向上した

この記事を読めば、パッケージの作成からGitHubでの公開、他のプロジェクトでの利用まで、UPMパッケージの基本的なワークフローを理解できます。

## 開発環境
- Unity 2020.3以上（2019.3以上でも基本機能は利用可能）
- Git（バージョン管理用）
- GitHub（パッケージの公開・共有用）
- 基本的なC#知識

## 事前準備
- Unityのインストール
- Gitのインストールと基本操作の理解
- GitHubアカウント

## やったこと

### 1. UPMパッケージとは何か

Unity Package Manager（UPM）は、Unityが提供するパッケージ管理システムで、以下のような特徴があります：

- プロジェクト間でコードやアセットを簡単に共有できる
- バージョン管理ができ、更新や後方互換性の維持が容易
- 依存関係を管理できる
- Unity本体のアップデートとは独立して、個別のパッケージを更新できる

UPMパッケージは、公式の「Unity Registry」に公開されているものだけでなく、GitHubなどの外部リポジトリからも直接インポートできます。このことを利用して、独自のパッケージを作成・共有することができます。

### 2. UPMパッケージの基本構造

UPMパッケージには、以下のような基本的なファイル構造が必要です：

```
MyPackage/
├── package.json      # パッケージの情報を定義
├── README.md         # パッケージの説明書
├── CHANGELOG.md      # 変更履歴
├── LICENSE.md        # ライセンス情報
├── Runtime/          # 実行時に必要なスクリプト
├── Editor/           # エディタ拡張スクリプト
├── Tests/            # テストコード
└── Documentation~/   # ドキュメント
```

これらのうち、最低限必要なのは`package.json`ファイルのみです。このファイルにはパッケージの基本情報が含まれます。

### 3. パッケージ用のGitHubリポジトリを作成する

まず、パッケージ用のGitHubリポジトリを作成します：

1. GitHubにログイン
2. 「New repository」をクリックして新しいリポジトリを作成
3. リポジトリ名を入力（例：`unity-utility-package`）
4. 「Create repository」をクリック

リポジトリが作成されたら、ローカルにクローンします：

```bash
git clone https://github.com/あなたのユーザー名/unity-utility-package.git
cd unity-utility-package
```

### 4. パッケージの基本ファイルを作成する

まず、`package.json`ファイルを作成します。これはパッケージの中心となるファイルで、名前やバージョンなどの重要な情報を含みます：

```json
{
  "name": "com.yourcompany.utilities",
  "version": "1.0.0",
  "displayName": "Utility Tools",
  "description": "A collection of utility scripts for Unity projects",
  "unity": "2020.3",
  "author": {
    "name": "Your Name",
    "email": "your.email@example.com",
    "url": "https://yourwebsite.com"
  },
  "keywords": [
    "utility",
    "tools"
  ],
  "license": "MIT",
  "documentationUrl": "https://github.com/yourname/unity-utility-package",
  "changelogUrl": "https://github.com/yourname/unity-utility-package/blob/main/CHANGELOG.md",
  "licensesUrl": "https://github.com/yourname/unity-utility-package/blob/main/LICENSE.md"
}
```

次に、基本的なフォルダ構造を作成します：

```bash
mkdir -p Runtime/Scripts
mkdir -p Editor/Scripts
mkdir -p Tests
```

そして、READMEファイルなどの基本的なドキュメントも作成します：

```markdown
// README.md
# Unity Utility Package

A collection of utility scripts for Unity projects.

## Installation

To install this package, follow these steps:

1. Open the Unity Package Manager
2. Click the "+" button and select "Add package from git URL..."
3. Enter: `https://github.com/あなたのユーザー名/unity-utility-package.git`
4. Click "Add"

## Features

- Feature 1
- Feature 2
- Feature 3

## Documentation

See the [documentation](Documentation~/index.md) for more information.
```

### 5. パッケージにコードを追加する

例として、簡単なユーティリティクラスを作成してみましょう。以下のファイルを`Runtime/Scripts`フォルダに作成します：

`Runtime/Scripts/StringUtility.cs`:
```csharp
using UnityEngine;

namespace YourCompany.Utilities
{
    /// <summary>
    /// 文字列操作のためのユーティリティクラス
    /// </summary>
    public static class StringUtility
    {
        /// <summary>
        /// 文字列の先頭を大文字にします
        /// </summary>
        /// <param name="input">入力文字列</param>
        /// <returns>先頭が大文字の文字列</returns>
        public static string Capitalize(string input)
        {
            if (string.IsNullOrEmpty(input))
                return input;
                
            return char.ToUpper(input[0]) + input.Substring(1);
        }
        
        /// <summary>
        /// 文字列を指定された回数繰り返します
        /// </summary>
        /// <param name="input">入力文字列</param>
        /// <param name="count">繰り返し回数</param>
        /// <returns>繰り返された文字列</returns>
        public static string Repeat(string input, int count)
        {
            if (string.IsNullOrEmpty(input) || count <= 0)
                return string.Empty;
                
            string result = string.Empty;
            for (int i = 0; i < count; i++)
            {
                result += input;
            }
            
            return result;
        }
    }
}
```

`Runtime/Scripts/MathUtility.cs`:
```csharp
using UnityEngine;

namespace YourCompany.Utilities
{
    /// <summary>
    /// 数学関連のユーティリティクラス
    /// </summary>
    public static class MathUtility
    {
        /// <summary>
        /// 値を指定範囲内に制限します
        /// </summary>
        /// <param name="value">元の値</param>
        /// <param name="min">最小値</param>
        /// <param name="max">最大値</param>
        /// <returns>制限された値</returns>
        public static float Clamp(float value, float min, float max)
        {
            return (value < min) ? min : (value > max) ? max : value;
        }
        
        /// <summary>
        /// 2点間の距離を計算します
        /// </summary>
        /// <param name="a">点A</param>
        /// <param name="b">点B</param>
        /// <returns>距離</returns>
        public static float Distance(Vector2 a, Vector2 b)
        {
            float dx = b.x - a.x;
            float dy = b.y - a.y;
            return Mathf.Sqrt(dx * dx + dy * dy);
        }
    }
}
```

Runtime/Scripts/YourCompany.Utilities.asmdefファイルも作成して、アセンブリ定義も追加しましょう：

```json
{
    "name": "YourCompany.Utilities",
    "rootNamespace": "YourCompany.Utilities",
    "references": [],
    "includePlatforms": [],
    "excludePlatforms": [],
    "allowUnsafeCode": false,
    "overrideReferences": false,
    "precompiledReferences": [],
    "autoReferenced": true,
    "defineConstraints": [],
    "versionDefines": [],
    "noEngineReferences": false
}
```

### 6. パッケージをGitHubにプッシュする

作成したパッケージファイルをGitHubにプッシュします：

```bash
git add .
git commit -m "Initial package setup"
git tag v1.0.0
git push origin main
git push origin v1.0.0
```

タグを付けることで、特定のバージョンをUnity Package Managerで簡単に参照できるようになります。

### 7. Unity ProjectでUPMパッケージを使用する

作成したパッケージを別のUnityプロジェクトで使用するには：

1. Unityを起動し、プロジェクトを開く
2. 「Window」→「Package Manager」を選択
3. 左上の「+」ボタンをクリック
4. 「Add package from git URL...」を選択
5. 以下のURLを入力：`https://github.com/あなたのユーザー名/unity-utility-package.git`

または、`https://github.com/あなたのユーザー名/unity-utility-package.git#v1.0.0`のように、特定のバージョンやブランチを指定することもできます。

また、`Packages/manifest.json`を直接編集して追加することもできます：

```json
{
  "dependencies": {
    "com.yourcompany.utilities": "https://github.com/あなたのユーザー名/unity-utility-package.git",
    // 他の依存関係...
  }
}
```

### 8. 特定のバージョンを指定する

開発が進むと、パッケージのバージョンを更新する必要があります。バージョンを指定するには：

1. `package.json`ファイルの"version"フィールドを更新（例："1.1.0"）
2. 変更をコミットし、新しいタグを付ける：

```bash
git add package.json
git commit -m "Update to version 1.1.0"
git tag v1.1.0
git push origin main
git push origin v1.1.0
```

利用側では、以下のようにバージョンを指定できます：
- `https://github.com/あなたのユーザー名/unity-utility-package.git#v1.1.0`
- `https://github.com/あなたのユーザー名/unity-utility-package.git#v1.0.0`

### 9. パッケージの更新と管理

パッケージを継続的に改善するためのヒント：

1. **セマンティックバージョニング**：バージョン番号はメジャー.マイナー.パッチ（例：1.2.3）の形式で管理し：
   - メジャー：互換性のない変更
   - マイナー：後方互換性のある新機能
   - パッチ：バグ修正

2. **CHANGELOG.mdの更新**：各バージョンでの変更内容を記録

3. **アセンブリ定義の適切な設定**：依存関係を明確にし、必要なプロジェクトのみが参照できるようにする

4. **サンプルの提供**：`Samples~`フォルダにサンプルコードを含めることで、パッケージの使い方を示す

## 補足・参考

### UPMパッケージを使うメリット

1. **コードの再利用**：一度作成したコードを複数のプロジェクトで簡単に使い回せる

2. **メンテナンスの容易さ**：修正や更新を一箇所で行うだけで、全プロジェクトに適用される

3. **バージョン管理**：各プロジェクトが必要とするバージョンを選択できる

4. **カプセル化**：機能単位でパッケージ化することで、コードの理解とテストが容易になる

5. **配布の簡易化**：チーム内やコミュニティへのコード共有が容易になる

### 関連リソース

- [Unity Package Manager のドキュメント](https://docs.unity3d.com/Manual/upm-ui.html)
- [カスタムパッケージの作成](https://docs.unity3d.com/Manual/CustomPackages.html)
- [パッケージのフォーマット](https://docs.unity3d.com/Manual/upm-manifestPkg.html)
- [OpenUPM](https://openupm.com/) - オープンソースのUnityパッケージレジストリ

### ベストプラクティス

- **明確な命名規則**：命名の衝突を避けるため、会社名や独自の接頭辞を使用する（例：`com.yourcompany.utilities`）

- **適切な粒度**：パッケージは単一の責任を持つべき。大きすぎるパッケージは分割を検討

- **ドキュメント**：使用方法や例を丁寧に記述

- **テスト**：UPMパッケージにテストを含め、品質を確保

- **オープンソース**：可能であれば、コミュニティに貢献するためオープンソースとして公開

## おわりに

UPMパッケージを活用することで、Unityプロジェクト間のコード共有が格段に効率化されます。初めは少し手間に感じるかもしれませんが、一度パッケージ化のワークフローを理解すれば、開発効率は大きく向上します。

特に、複数のプロジェクトを同時に開発している場合や、チームで開発している場合は、共通のコードベースを維持する手段として非常に有効です。また、コードをパッケージ化することで、より整理された設計思考が身につき、再利用可能なコンポーネントを作る習慣が身に付きます。

ぜひ、自分の便利な機能をパッケージ化して、プロジェクト間で共有してみてください。さらに一歩進んで、GitHub上で公開すれば、世界中のUnity開発者があなたのコードを利用できるようになります。

パッケージの作成と共有を通じて、Unity開発のワークフローを改善し、より効率的で品質の高い開発を実現しましょう。
