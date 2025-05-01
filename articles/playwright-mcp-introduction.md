---
title: "PlayWrightMCPを使ってテスト自動化とCopilotを連携させる方法"
emoji: "🎭"
type: "tech"
topics: ["playwright", "mcp", "testing", "vscode", "copilot"]
published: false
---

## ゴール(はじめに)：PlayWrightMCPの基礎を理解し、VSCodeで効率的にテスト自動化を行う

PlayWright Model Context Protocol（MCP）は、テスト自動化とAIアシスタント（GitHub Copilot）を連携させるための新しいフレームワークです。この記事では、PlayWrightMCPの基本的な使い方を紹介し、VSCodeにインストールして、Copilotと連携する方法を解説します。

## やってみた結果

PlayWrightMCPを使うことで以下のことが可能になりました：
- Copilotに対してブラウザの状態を認識させ、より的確なテストコード生成が可能に
- テストの記述・実行・デバッグが一つの環境で完結
- 自然言語での指示によるテスト作成の効率化

この記事を読むことで、あなたもPlayWrightとCopilotを連携させ、テスト自動化の効率を大幅に向上させることができるでしょう。

## 開発環境
- Windows/macOS/Linux
- Visual Studio Code
- Node.js (バージョン14以上推奨)

フォルダ構造：
```
my-playwright-project/
├── node_modules/
├── tests/
│   └── example.spec.ts
├── playwright.config.ts
└── package.json
```

## 事前準備

PlayWrightMCPを使用するには以下が必要です：
- Visual Studio Codeのインストール
- Node.jsのインストール（npmを含む）
- GitHub Copilotのサブスクリプション（推奨）

## PlayWrightのインストール

まず、新しいプロジェクトを作成し、PlayWrightをインストールします：

```bash
# 新しいフォルダを作成
mkdir my-playwright-project
cd my-playwright-project

# package.jsonの初期化
npm init -y

# PlayWrightのインストール
npm init playwright@latest
```

インストールウィザードに従って設定を完了します：
- TypeScriptを使用するか選択（推奨：Yes）
- テストフォルダの名前を指定（デフォルト：tests）
- GitHub Actionsのワークフローを追加するか選択

## PlayWright MCPのインストール

次に、Visual Studio CodeでPlayWright MCPの拡張機能をインストールします：

1. VSCodeの拡張機能タブを開く（Ctrl+Shift+X または Command+Shift+X）
2. 検索バーに「Playwright」と入力
3. 「Playwright Test for VSCode」をインストール
4. MCPサポートを有効にするために、VSCodeの設定で以下を追加：

```json
{
  "playwright.experimental.mcp.enabled": true
}
```

## Copilotとの連携設定

PlayWright MCPとCopilotを連携させるには：

1. VSCodeに「GitHub Copilot」拡張機能をインストール（既にインストール済みの場合はスキップ）
2. GitHub Copilotにログインして認証を完了
3. テストファイルを開いた状態でコマンドパレットを開き（Ctrl+Shift+P または Command+Shift+P）、「Playwright: Start MCP Session」を選択

これにより、PlayWrightとCopilotの連携セッションが開始され、ブラウザの状態情報がCopilotに提供されるようになります。

## PlayWright MCPの使用方法

### 基本的なテスト実行

1. VSCodeでプロジェクトフォルダを開く
2. テストファイル（例：tests/example.spec.ts）を開く
3. テストファイルの横にある実行ボタンをクリック、またはコマンドパレットから「Playwright: Run Tests」を選択

### MCPセッションの開始

```bash
# VS Codeのターミナルから以下を実行
npx playwright test --ui
```

または、VSCodeのコマンドパレットから「Playwright: Start MCP Session」を選択します。

### Copilotとの対話例

MCPセッションが開始されると、Copilotに次のような指示を与えることができます：

1. 「ログインページのテストを作成して」
2. 「商品検索機能をテストするコードを書いて」
3. 「現在表示されているフォームの全項目を入力するテストを作成して」

Copilotは現在のブラウザ状態を把握しているため、より正確なコード提案が可能になります。

## PlayWright MCPの主要機能

### 1. ブラウザコンテキスト認識

PlayWright MCPは、Copilotにブラウザの現在の状態（DOM構造、CSS、JavaScript実行状態など）を提供します。これにより、Copilotはページの実際の状態に基づいたより正確なテストコードを提案できます。

### 2. インタラクティブなテスト作成

VSCode内でテストを作成しながら、リアルタイムでブラウザ上での実行結果を確認できます。テストステップを追加・修正するたびに、ブラウザの反応を見ることができます。

### 3. 自然言語によるテスト生成

例えば、「ログインページにユーザー名とパスワードを入力してログインボタンをクリックするテストを作成」といった自然言語の指示から、適切なテストコードを生成できます。

### 4. レコーディング機能との連携

Playwrightの標準機能であるレコーディングと組み合わせることで、手動で行った操作をもとに、Copilotがより洗練されたテストコードを提案できます。

```typescript
// PlayWrightでの典型的なテスト例
import { test, expect } from '@playwright/test';

test('ログインテスト', async ({ page }) => {
  await page.goto('https://example.com/login');
  await page.fill('input[name="username"]', 'testuser');
  await page.fill('input[name="password"]', 'password123');
  await page.click('button[type="submit"]');
  
  // ログイン後のアサーション
  await expect(page.locator('.welcome-message')).toContainText('ようこそ');
});
```

## 結論に対しての補足

### PlayWright MCPのメリット

- コードの品質向上：AIによる提案が実際のDOM構造に基づいているため、より堅牢なセレクタを使用できる
- 開発速度の向上：テストコードの記述が大幅に効率化される
- 学習曲線の緩和：Playwrightの詳細な知識がなくても効果的なテストを作成可能

### 関連リンク
- [PlayWright MCP GitHub リポジトリ](https://github.com/microsoft/playwright-mcp)
- [PlayWright 公式ドキュメント](https://playwright.dev/)
- [Visual Studio Code 拡張機能マーケットプレイス](https://marketplace.visualstudio.com/items?itemName=ms-playwright.playwright)
- [GitHub Copilot ドキュメント](https://docs.github.com/ja/copilot)

## おわりに

PlayWright MCPは、テスト自動化とAI支援開発の融合という新しい可能性を開く技術です。テスト作成の効率が大幅に向上するだけでなく、テストの品質も向上させることができます。特に大規模なウェブアプリケーションのテスト自動化において、その威力を発揮するでしょう。

今後もPlayWright MCPは進化を続け、より多くの機能が追加される予定です。テスト自動化の効率を高めたい開発者は、ぜひPlayWright MCPを試してみてください。自動テストの作成がこれまでよりもずっと簡単で楽しくなるはずです。
