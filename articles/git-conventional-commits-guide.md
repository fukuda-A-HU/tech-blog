---
title: "ConventionalCommitsの使い方：初心者向け実践ガイド"
emoji: "📝"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["git", "github", "conventionalcommits", "開発フロー", "チーム開発"]
published: false
---

## ゴール(はじめに)：ConventionalCommitsを理解し実践する

この記事では、ConventionalCommitsという規約に基づいたコミットメッセージの書き方について初心者向けに解説します。ConventionalCommitsを使うことで、コミットの意図が明確になり、自動でCHANGELOGを生成するなど、より体系的なバージョン管理が可能になります。チーム開発でも個人開発でも役立つこの手法を身につけましょう。

## やってみた結果
- コミットログが統一された形式で整理された
- コミットの目的が一目で分かるようになった
- セマンティックバージョニングとの連携が容易になった
- 自動CHANGELOGの生成が可能になった

この記事を読めば、ConventionalCommitsの基本的な書き方を理解し、すぐに実践できるようになります。

## 開発環境
- Git 2.20以上
- Node.js 14.0以上（オプションツールを使用する場合）
- 任意のテキストエディタ/IDE

## 事前準備
- Gitの基本的な使い方を理解していること
- Gitがインストールされていること

## やったこと

### 1. ConventionalCommitsとは何か

ConventionalCommitsとは、コミットメッセージに関する規約で、人間と機械の両方が読みやすい標準的な形式を定義しています。この規約は以下の形式をベースとしています：

```
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

主な要素：
- **type**: コミットの種類（feat, fix, docs, styleなど）
- **scope**: コミットの影響範囲（オプション）
- **description**: 変更内容の簡潔な説明
- **body**: 詳細な変更内容（オプション）
- **footer**: 追加情報やBreaking Changesへの言及（オプション）

### 2. 基本的なタイプ(type)とその意味

ConventionalCommitsで一般的に使用されるタイプとその意味を説明します：

- **feat**: 新機能の追加
  ```
  feat: ユーザー登録機能を追加
  ```

- **fix**: バグ修正
  ```
  fix: ログイン時のエラーを修正
  ```

- **docs**: ドキュメントのみの変更
  ```
  docs: READMEの使用方法を更新
  ```

- **style**: コードの意味に影響を与えない変更（フォーマットなど）
  ```
  style: インデントを修正
  ```

- **refactor**: バグ修正や機能追加を含まないコードの変更
  ```
  refactor: ユーザー認証処理をリファクタリング
  ```

- **test**: テストの追加・修正
  ```
  test: ユーザー登録のユニットテストを追加
  ```

- **chore**: ビルドプロセスやツールの変更、ライブラリの更新など
  ```
  chore: package.jsonの依存関係を更新
  ```

### 3. スコープ(scope)の活用

スコープを追加することで、変更の影響範囲をより明確に示すことができます：

```
feat(auth): ソーシャルログイン機能を追加
fix(database): 接続切れ時の自動再接続機能を修正
```

スコープは括弧で囲み、システムの一部、機能、モジュール名などを指定します。プロジェクトに合わせてチームで統一したスコープを決めておくと良いでしょう。

### 4. Breaking Changesの表現方法

後方互換性を破る変更（Breaking Changes）は、タイプの後に「!」をつけるか、フッターに「BREAKING CHANGE:」を記述します：

```
feat!: APIレスポンス形式を変更

BREAKING CHANGE: レスポンスのJSONフォーマットを新しい標準に合わせて変更
```

または

```
feat(api): APIレスポンス形式を変更

BREAKING CHANGE: レスポンスのJSONフォーマットを新しい標準に合わせて変更
```

### 5. 実践的なコミットメッセージの例

以下にConventionalCommitsを使った実践的なコミットメッセージの例を紹介します：

**シンプルな新機能追加**:
```
feat: 商品検索機能を追加
```

**特定の領域のバグ修正**:
```
fix(checkout): カートに商品を追加できないバグを修正
```

**詳細な説明を含む機能追加**:
```
feat(user): パスワードリセット機能を追加

ユーザーが自分のパスワードを忘れた場合に、
メールアドレスを使用してリセットできるようにしました。
セキュリティのため、リンクは1時間で期限切れになります。
```

**Breaking Changeを含む変更**:
```
feat!: 認証APIのエンドポイントを変更

古いエンドポイント /api/v1/login は非推奨となり、
新しいエンドポイント /api/v2/auth/login に変更されました。

BREAKING CHANGE: 認証APIのエンドポイントが変更されたため、
クライアントコードの更新が必要です。
```

### 6. コミットメッセージ作成の補助ツール

ConventionalCommitsに従ったメッセージを簡単に作成するためのツールがあります。

**commitizen**のインストールと使用方法：

```bash
# グローバルにインストール
npm install -g commitizen

# プロジェクトをcz-conventionalに対応させる
npm install --save-dev cz-conventional-changelog
```

package.jsonに設定を追加します：

```json
{
  "config": {
    "commitizen": {
      "path": "cz-conventional-changelog"
    }
  }
}
```

これで、`git commit`の代わりに`git cz`を使用することで対話形式でCommitメッセージを作成できます：

```bash
git cz
```

実行すると、以下のような対話形式の質問が表示され、それに答えるだけでConventionalCommitsに準拠したコミットメッセージを作成できます：

1. コミットタイプの選択
2. スコープの入力（オプション）
3. 簡単な説明の入力
4. 詳細な説明の入力（オプション）
5. Breaking Changeの有無
6. 関連するIssueの参照（オプション）

### 7. 自動CHANGELOG生成

ConventionalCommitsを使うと、自動的にCHANGELOGを生成できます。例えば、`standard-version`というツールを使用する方法を紹介します：

```bash
# インストール
npm install --save-dev standard-version

# package.jsonにスクリプトを追加
```

package.jsonに以下のスクリプトを追加します：

```json
{
  "scripts": {
    "release": "standard-version"
  }
}
```

リリース時に以下のコマンドを実行するだけで、コミット履歴からCHANGELOGが生成され、タグが作成されます：

```bash
npm run release
```

これにより、CHANGELOG.mdファイルが自動的に更新されます。

## 補足・参考

### ConventionalCommitsを採用するメリット

1. **コミット履歴の可読性向上**：統一された形式でコミット履歴が整理され、プロジェクトの変更履歴が追いやすくなります。

2. **自動化の促進**：セマンティックバージョニングと連携した自動リリースノート作成などの自動化が容易になります。

3. **チーム開発の効率化**：チームメンバー全員が同じ形式でコミットすることで、レビューや変更の理解が容易になります。

4. **CIプロセスとの統合**：コミットタイプに基づいた自動テストやデプロイのフローを構築できます。

### 関連ツール・リソース

- [Conventional Commits 公式サイト](https://www.conventionalcommits.org/ja/)
- [commitlint](https://github.com/conventional-changelog/commitlint): コミットメッセージをリントするツール
- [husky](https://github.com/typicode/husky): Gitフックを簡単に使えるようにするツール
- [semantic-release](https://github.com/semantic-release/semantic-release): リリース自動化ツール

### セマンティックバージョニングとの関連性

ConventionalCommitsはセマンティックバージョニング（SemVer）と密接に関連しています。

- **feat** → マイナーバージョンアップ（1.0.0 → 1.1.0）
- **fix** → パッチバージョンアップ（1.0.0 → 1.0.1）
- **BREAKING CHANGE** を含む → メジャーバージョンアップ（1.0.0 → 2.0.0）

これにより、コミットメッセージから次のバージョン番号を自動的に決定できます。

## おわりに

ConventionalCommitsは、シンプルながらも強力なコミットメッセージの規約です。最初は慣れるのに時間がかかるかもしれませんが、一度習得すれば、プロジェクト管理が格段に楽になります。個人開発でも、チーム開発でも、一貫性のあるコミット履歴を残すことで、将来の自分や他の開発者が変更の履歴を理解しやすくなります。

また、自動化ツールとの親和性が高いため、CI/CDパイプラインの構築やリリース管理にも役立ちます。ぜひあなたの次のプロジェクトでConventionalCommitsを試してみてください。

シンプルな規約から始めて、徐々にプロジェクトに合わせたカスタマイズを加えていくことで、無理なく導入できるでしょう。何よりも一貫性を保つことが重要です。
