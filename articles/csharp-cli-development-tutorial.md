---
title: "C#を用いたCLIツールの作成方法"
emoji: "🛠️"
type: "tech"
topics: ["csharp", "dotnet", "cli", "tutorial"]
published: false
---

## ゴール(はじめに)：C#でコマンドラインツールを作成する

この記事では、C#言語を使ってコマンドラインインターフェース（CLI）ツールを作成する方法を解説します。コマンドライン引数の処理から、シンプルな対話型インターフェースの実装まで、初心者でも理解しやすいように段階的に説明します。

## やってみた結果

- C#でのCLIツール開発の基本が分かる
- コマンドライン引数の処理方法を習得できる
- シンプルなファイル処理ツールが作れるようになる
- .NET Core CLIツールとして配布する方法が分かる

## 開発環境
- .NET 6.0以上（.NET Coreベース）
- Visual Studio Code または Visual Studio
- dotnet CLI

フォルダ構造：
```
MyCliTool/
├── Program.cs          # メインプログラム
├── MyCliTool.csproj    # プロジェクトファイル
├── Properties/         # プロジェクト設定
└── Commands/           # コマンド実装クラス
```

## 事前準備

1. .NET SDKのインストール
   - [.NET公式サイト](https://dotnet.microsoft.com/download)から最新版をダウンロード
   - インストール後、ターミナルで確認：
     ```bash
     dotnet --version
     ```

2. 開発環境の準備
   - VS Codeを使用する場合は、C#拡張機能をインストール

## やったこと

### 1. プロジェクトの作成

まずはコンソールアプリケーションのプロジェクトを作成します：

```bash
dotnet new console -n MyCliTool
cd MyCliTool
```

### 2. 基本的なコマンドライン引数の処理

Program.cs を開き、シンプルな引数処理を実装します：

```csharp
using System;

namespace MyCliTool
{
    class Program
    {
        static void Main(string[] args)
        {
            if (args.Length == 0)
            {
                Console.WriteLine("使い方: MyCliTool <コマンド> [オプション]");
                Console.WriteLine("コマンド一覧:");
                Console.WriteLine("  hello    - 挨拶を表示します");
                Console.WriteLine("  version  - バージョンを表示します");
                return;
            }

            string command = args[0].ToLower();
            
            switch (command)
            {
                case "hello":
                    string name = args.Length > 1 ? args[1] : "ゲスト";
                    Console.WriteLine($"こんにちは、{name}さん！");
                    break;
                
                case "version":
                    Console.WriteLine("MyCliTool バージョン 1.0.0");
                    break;
                
                default:
                    Console.WriteLine($"不明なコマンド: {command}");
                    break;
            }
        }
    }
}
```

### 3. コマンドラインパーサーの導入

より高度な引数処理には、`System.CommandLine`ライブラリを使います：

```bash
dotnet add package System.CommandLine --prerelease
```

そして、Program.csを更新します：

```csharp
using System;
using System.CommandLine;
using System.CommandLine.Invocation;
using System.Threading.Tasks;

namespace MyCliTool
{
    class Program
    {
        static async Task<int> Main(string[] args)
        {
            // ルートコマンドの作成
            var rootCommand = new RootCommand("シンプルなCLIツールのサンプル");

            // hello コマンドの設定
            var helloCommand = new Command("hello", "挨拶を表示します");
            var nameOption = new Option<string>(
                "--name",
                description: "挨拶する相手の名前を指定します") { IsRequired = false };
            nameOption.AddAlias("-n");
            nameOption.SetDefaultValue("ゲスト");
            helloCommand.AddOption(nameOption);
            
            helloCommand.SetHandler((string name) =>
            {
                Console.WriteLine($"こんにちは、{name}さん！");
            }, nameOption);

            // ファイル処理コマンドの設定
            var fileCommand = new Command("file", "ファイル操作を行います");
            var pathArgument = new Argument<string>("path", "対象ファイルのパス");
            fileCommand.AddArgument(pathArgument);
            
            var countOption = new Option<bool>(
                "--count",
                description: "行数をカウントします");
            countOption.AddAlias("-c");
            fileCommand.AddOption(countOption);

            fileCommand.SetHandler((string path, bool count) =>
            {
                if (!System.IO.File.Exists(path))
                {
                    Console.WriteLine($"エラー: ファイル '{path}' が見つかりません");
                    return;
                }

                if (count)
                {
                    int lineCount = System.IO.File.ReadAllLines(path).Length;
                    Console.WriteLine($"ファイル '{path}' の行数: {lineCount}");
                }
                else
                {
                    string content = System.IO.File.ReadAllText(path);
                    Console.WriteLine(content);
                }
            }, pathArgument, countOption);

            // コマンドの登録
            rootCommand.AddCommand(helloCommand);
            rootCommand.AddCommand(fileCommand);

            // コマンドの実行
            return await rootCommand.InvokeAsync(args);
        }
    }
}
```

### 4. ツールのビルドと実行

ツールをビルドして実行します：

```bash
dotnet build
dotnet run -- hello --name 太郎
dotnet run -- file Program.cs --count
```

### 5. グローバルツールとしての配布

.csprojファイルを編集して、グローバルツールとしてパッケージ化します：

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net6.0</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
    
    <!-- グローバルツール設定 -->
    <PackAsTool>true</PackAsTool>
    <ToolCommandName>mytool</ToolCommandName>
    <PackageOutputPath>./nupkg</PackageOutputPath>
    <Version>1.0.0</Version>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="System.CommandLine" Version="2.0.0-beta*" />
  </ItemGroup>

</Project>
```

ツールのパッケージ化とインストール：

```bash
dotnet pack
dotnet tool install --global --add-source ./nupkg MyCliTool
```

インストール後は、どこからでもコマンドが実行できます：

```bash
mytool hello --name 花子
```

## 結論に対しての補足
- [Microsoft.Extensions.CommandLineUtils](https://www.nuget.org/packages/Microsoft.Extensions.CommandLineUtils/) - 別の引数パーサーライブラリ
- [Spectre.Console](https://spectreconsole.net/) - 高度なコンソールUI作成ライブラリ
- [System.CommandLine (GitHub)](https://github.com/dotnet/command-line-api) - 公式リポジトリ
- [.NET公式ドキュメント](https://docs.microsoft.com/ja-jp/dotnet/core/tools/global-tools) - グローバルツールの作成

## おわりに

C#でCLIツールを作成することは、思ったより簡単です。基本的なコンソールアプリケーションから始めて、引数のパースや対話的な要素を加えることで、実用的なツールに進化させることができます。ぜひ自分のアイデアを形にして、業務効率化や日常のタスク自動化に活用してみてください！
