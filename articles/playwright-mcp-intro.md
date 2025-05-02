---
title: "playwright-mcpの概要と使い方入門"
emoji: "🧑‍💻"
type: "tech"
topics: ["Playwright", "MCP", "自動化", "初心者"]
published: false
---

## ゴール(はじめに)：playwright-mcpの概要と使い方を理解する

この記事では、Webブラウザ自動化ツール「playwright-mcp」について、初心者にも分かりやすく解説します。playwright-mcpの特徴や使い方、Linuxでの起動例などを紹介し、実際に使い始めるための手順をまとめます。

## やってみた結果

- playwright-mcpの基本的な仕組みと特徴が分かる
- Linux環境でのインストール・起動方法が分かる
- サンプル設定やコマンド例を参考に、すぐに試せる

## 開発環境
- Linux（他OSでも利用可能）
- Node.js（npxコマンドを利用）
- VS Code（オプション）

フォルダ構成例：
```
my-mcp-project/
├── config.json  # 設定ファイル（任意）
└── ...
```

## 事前準備
- Node.jsがインストールされていることを確認してください。
  - 確認コマンド：
    ```bash
    node -v
    npm -v
    ```
- npxコマンドが使えること（npmに同梱）

## やったこと

### playwright-mcpとは？

- [Playwright](https://playwright.dev)を使ったブラウザ自動化サーバーです。
- 画像（スクリーンショット）ではなく、アクセシビリティツリー（構造化データ）を使ってWebページを操作します。
- LLM（大規模言語モデル）やエージェントがWebページを操作・データ取得できるように設計されています。

### 主な特徴
- **高速・軽量**：画像処理不要で動作が速い
- **LLMフレンドリー**：構造化データのみで操作可能
- **再現性が高い**：スクリーンショットベースの曖昧さがない

### 代表的な用途
- Webページの自動操作・フォーム入力
- 構造化データの抽出
- LLMによる自動テストやデータ収集

## playwright-mcpのインストールと起動

### 1. コマンドラインからの起動

npxを使って、最新バージョンのplaywright-mcpサーバーを起動できます。

```bash
npx @playwright/mcp@latest --port 8931
```

- `--port`で待ち受けポートを指定します（例：8931）。
- デフォルトはヘッドレス（画面なし）モードです。

### 2. 設定ファイル例

MCPクライアントから接続する場合の設定例：

```json
{
  "mcpServers": {
    "playwright": {
      "url": "http://localhost:8931/sse"
    }
  }
}
```

### 3. Dockerでの利用

Dockerでも簡単に起動できます（Chromiumのみ対応）。

```bash
docker run -i --rm --init mcp/playwright
```

自分でイメージをビルドする場合：
```bash
docker build -t mcp/playwright .
```

## 便利なオプション
- `--browser <browser>`: 使用するブラウザを指定（chrome, firefox, webkitなど）
- `--headless`: ヘッドレスモードで起動（デフォルト）
- `--vision`: スクリーンショットベースの操作も可能

## 結論に対しての補足
- 公式リポジトリ: [https://github.com/microsoft/playwright-mcp](https://github.com/microsoft/playwright-mcp)
- Playwright公式: [https://playwright.dev/](https://playwright.dev/)
- MCPプロトコル: [https://modelcontextprotocol.org/](https://modelcontextprotocol.org/)

## おわりに

playwright-mcpは、初心者でも簡単にWeb自動化やデータ取得を始められる強力なツールです。npxコマンド一発で試せるので、ぜひ一度触ってみてください！