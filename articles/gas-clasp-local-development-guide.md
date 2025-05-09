---
title: "GASプロジェクトをaside, claspを用いてローカルで作成・テストする方法"
emoji: "📝"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["GAS", "clasp", "JavaScript", "TypeScript", "GoogleAppsScript"]
published: false
---

## ゴール(はじめに)：GoogleAppsScriptをローカル環境で効率的に開発する

Google Apps Script（GAS）はGoogleのサービスと連携できる強力なスクリプト環境ですが、デフォルトではブラウザ上のスクリプトエディタでしか開発できません。この記事では、Google公式ツールの「clasp」とテストフレームワーク「aside」を使って、VS CodeなどのIDEでGASプロジェクトを作成し、ローカル環境でテスト・デバッグする方法を紹介します。これにより、バージョン管理やモジュール分割、型チェックなど、モダンな開発手法をGAS開発に取り入れることができます。

## やってみた結果

この記事の内容を実践することで、以下のメリットを得られます：

- お気に入りのエディタ（VS Code等）でGASの開発ができる
- GitHubなどでプロジェクトをバージョン管理できる
- TypeScriptによる型チェックで堅牢なコードを書ける
- ローカル環境でのテストが可能になり、開発効率が向上する
- モジュール分割による再利用可能なコードの作成ができる

私自身、この方法を採用してから、GAS開発の効率と品質が大幅に向上しました。特に複雑なGASプロジェクトや、チームでの開発では大きな違いを実感できるでしょう。

## 開発環境

- Node.js（v12以上）
- npm または yarn
- VS Code（または任意のコードエディタ）
- Gitクライアント（オプション）

フォルダ構造の例：
```
my-gas-project/
├── .clasp.json      # claspの設定ファイル
├── appsscript.json  # GASのマニフェストファイル
├── package.json     # npm設定ファイル
├── tsconfig.json    # TypeScript設定ファイル
├── src/
│   ├── Code.ts      # メインのソースコード
│   └── utils/       # ユーティリティ関数等
└── tests/
    └── Code.test.ts # テストコード
```

## 事前準備

この記事の手順を進める前に、以下のものが必要です：

1. Googleアカウント
2. Node.jsとnpmがインストールされていること
3. GASを使用するGoogleサービス（Sheets, Docs, Gmailなど）へのアクセス

まだNode.jsがインストールされていない場合は、[Node.jsの公式サイト](https://nodejs.org/)からダウンロードしてインストールしてください。

## やったこと

### 1. claspのインストールとログイン

まず、Googleのコマンドラインツールであるclaspをグローバルにインストールします：

```bash
npm install -g @google/clasp
```

次に、Googleアカウントでログインします：

```bash
clasp login
```

このコマンドを実行すると、ブラウザが開き、Googleアカウントへのアクセス許可を求められます。許可すると、ターミナルに成功メッセージが表示されます。

### 2. GASプロジェクトの作成

プロジェクトを作成するディレクトリを作り、そこに移動します：

```bash
mkdir my-gas-project
cd my-gas-project
```

新しいGASプロジェクトを作成します。ここでは、スタンドアロンのスクリプトを例に挙げます：

```bash
clasp create --title "My GAS Project" --type standalone
```

プロジェクトの種類は用途に応じて選択できます：

- `standalone`: スタンドアロンスクリプト
- `sheets`: Google Sheetsに紐づくスクリプト
- `docs`: Google Docsに紐づくスクリプト
- `forms`: Google Formsに紐づくスクリプト

### 3. TypeScriptの設定

TypeScriptを使用するための設定を行います。まず必要なパッケージをインストールします：

```bash
npm init -y  # package.jsonの作成
npm install --save-dev typescript @types/google-apps-script
```

次に、TypeScript設定ファイルを作成します。以下の内容で`tsconfig.json`を作成してください：

```json
{
  "compilerOptions": {
    "target": "ES2019",
    "moduleResolution": "node",
    "lib": ["esnext"],
    "outDir": "build",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true
  },
  "include": ["src/**/*"]
}
```

### 4. プロジェクト構造の整理

ソースコードを管理しやすくするために、`src`ディレクトリを作成します：

```bash
mkdir src
```

GASのマニフェストファイル`appsscript.json`を確認し、必要に応じて編集します：

```json
{
  "timeZone": "Asia/Tokyo",
  "dependencies": {
  },
  "exceptionLogging": "STACKDRIVER",
  "runtimeVersion": "V8"
}
```

### 5. サンプルコードの作成

`src`ディレクトリに、サンプルのTypeScriptファイル`Code.ts`を作成します：

```typescript
// Google Apps Scriptのグローバル関数を定義
// この関数はウェブアプリとして公開したときに実行される
function doGet(): GoogleAppsScript.HTML.HtmlOutput {
  const html = HtmlService.createHtmlOutput('<h1>Hello, world!</h1>');
  return html;
}

// サンプルの計算関数
function calculateSum(a: number, b: number): number {
  return a + b;
}

// SpreadsheetのデータをJSON形式で取得する関数
function getSheetDataAsJson(sheetId: string, sheetName: string): object[] {
  try {
    const ss = SpreadsheetApp.openById(sheetId);
    const sheet = ss.getSheetByName(sheetName);
    
    if (!sheet) {
      throw new Error(`Sheet "${sheetName}" not found`);
    }
    
    const data = sheet.getDataRange().getValues();
    const headers = data[0];
    const jsonData = [];
    
    // ヘッダー行をスキップしてJSONオブジェクトに変換
    for (let i = 1; i < data.length; i++) {
      const row = data[i];
      const rowData: Record<string, any> = {};
      
      for (let j = 0; j < headers.length; j++) {
        rowData[headers[j]] = row[j];
      }
      
      jsonData.push(rowData);
    }
    
    return jsonData;
  } catch (error) {
    console.error(`Error: ${error}`);
    return [];
  }
}

// グローバルにエクスポートする（GASで使用するため）
global.doGet = doGet;
global.calculateSum = calculateSum;
global.getSheetDataAsJson = getSheetDataAsJson;
```

### 6. asideのインストールとテストの設定

ローカルでGASのコードをテストするために、asideライブラリをインストールします：

```bash
npm install --save-dev @google/aside aside-gas
```

テスト用のディレクトリとサンプルテストファイルを作成します：

```bash
mkdir tests
```

`tests/Code.test.ts`ファイルを作成します：

```typescript
import { MockSpreadsheet } from 'aside-gas';
import * as assert from 'assert';

// テスト用のグローバルモック環境を設定
const mock = new MockSpreadsheet();
global.SpreadsheetApp = mock.SpreadsheetApp;
global.HtmlService = {
  createHtmlOutput: (html: string) => ({
    content: html,
    getContent: () => html
  })
};

// 実際のコード関数を読み込む
import '../src/Code';

describe('Google Apps Script Functions', () => {
  describe('calculateSum', () => {
    it('should correctly sum two numbers', () => {
      // @ts-ignore (グローバル関数をテスト)
      const result = calculateSum(5, 7);
      assert.strictEqual(result, 12);
    });
  });

  describe('doGet', () => {
    it('should return HTML content', () => {
      // @ts-ignore (グローバル関数をテスト)
      const result = doGet();
      assert.strictEqual(result.content, '<h1>Hello, world!</h1>');
    });
  });

  describe('getSheetDataAsJson', () => {
    beforeEach(() => {
      // テスト用のスプレッドシートデータをモックする
      const mockData = [
        ['名前', '年齢', '趣味'],
        ['田中', 28, '読書'],
        ['佐藤', 35, '旅行'],
      ];
      mock.setData('1234567890', 'Sheet1', mockData);
    });

    it('should convert sheet data to JSON', () => {
      // @ts-ignore (グローバル関数をテスト)
      const result = getSheetDataAsJson('1234567890', 'Sheet1');
      assert.deepStrictEqual(result, [
        { '名前': '田中', '年齢': 28, '趣味': '読書' },
        { '名前': '佐藤', '年齢': 35, '趣味': '旅行' }
      ]);
    });

    it('should return empty array when sheet not found', () => {
      // @ts-ignore (グローバル関数をテスト)
      const result = getSheetDataAsJson('1234567890', '存在しないシート');
      assert.deepStrictEqual(result, []);
    });
  });
});
```

package.jsonにテスト実行スクリプトを追加します：

```json
{
  "scripts": {
    "test": "mocha -r ts-node/register tests/**/*.test.ts"
  }
}
```

必要なテスト用パッケージをインストールします：

```bash
npm install --save-dev mocha @types/mocha ts-node assert @types/node
```

### 7. 開発とデプロイのワークフロー

開発サイクルをスムーズにするために、package.jsonにいくつかのスクリプトを追加します：

```json
{
  "scripts": {
    "build": "tsc",
    "push": "clasp push",
    "deploy": "npm run build && npm run push",
    "open": "clasp open",
    "test": "mocha -r ts-node/register tests/**/*.test.ts"
  }
}
```

これで以下のコマンドが使用可能になります：

- `npm run build`: TypeScriptコードをコンパイル
- `npm run push`: コードをGASにアップロード
- `npm run deploy`: ビルドとアップロードを一度に実行
- `npm run open`: ブラウザでGASエディタを開く
- `npm run test`: テストを実行

### 8. テストの実行とデバッグ

テストを実行して、コードが正しく動作するか確認します：

```bash
npm run test
```

正しく設定できていれば、テストが実行され、結果が表示されます。エラーがあれば修正し、再度テストを実行してください。

### 9. GASへのデプロイ

ローカルでテストが完了したら、コードをGASにプッシュします：

```bash
npm run deploy
```

ブラウザでGASエディタを開いて確認します：

```bash
npm run open
```

## 結論に対しての補足

### 便利な機能とテクニック

- **環境変数の利用**: `.env`ファイルと`dotenv`パッケージを使用して、APIキーなどの機密情報を管理できます。
- **モジュール分割**: 機能ごとにファイルを分けることで、コードの可読性と保守性が向上します。
- **Gitでのバージョン管理**: プロジェクトをGitリポジトリで管理することで、変更履歴の追跡やチーム開発が容易になります。
- **CI/CD**: GitHub Actionsなどを使用して、プッシュ時に自動テストとデプロイを行うこともできます。

### 注意点

- GASにはブラウザ環境と違い、一部のJavaScript機能が使用できません（`window`オブジェクトなど）。
- `clasp push`でアップロードできるファイルサイズには制限があります。
- GASのランタイムバージョン（Rhino/V8）によって利用できる機能が異なります。
- OAuth認証情報のセキュリティに注意が必要です。`.clasp.json`をGitリポジトリに含めないようにしましょう。

### 関連サービス・ツール

- [Google Apps Script](https://developers.google.com/apps-script): 公式ドキュメント
- [clasp](https://github.com/google/clasp): GitHub上のclaspリポジトリ
- [aside-gas](https://github.com/google/aside): GASのテストフレームワーク
- [TypeScript](https://www.typescriptlang.org/): 強力な型システムを持つJavaScriptの上位互換言語

## おわりに

ブラウザ上のスクリプトエディタだけでGAS開発をしていた頃と比べ、claspとasideを導入することで開発体験が劇的に改善します。特にTypeScriptによる型チェックと、ローカル環境でのテストが可能になることで、エラーの早期発見と修正が容易になります。

大規模なGASプロジェクトや、複数人で開発するプロジェクトでは、この記事で紹介した方法を是非試してみてください。モダンな開発ツールチェーンを活用することで、GoogleAppsScriptの可能性がさらに広がるはずです。

なお、GASの実行環境はV8エンジンへ移行していますが、依然としてブラウザJavaScriptとは異なる制約があります。公式ドキュメントを参照しながら開発することをお勧めします。
