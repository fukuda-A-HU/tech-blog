---
title: "GASで実現するカレンダー間の時間帯同期：予定の衝突を防ぐシンプルなソリューション"
emoji: "📅"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["gas", "googleappsscript", "googlecalendar", "javascript"]
published: false
---

## ゴール(はじめに)：GASを利用して複数のカレンダーを効率的に管理する

Googleカレンダーは複数のカレンダーを持てる便利なツールですが、仕事用と個人用など複数のカレンダーを使い分けていると、予定の重複チェックが大変になります。この記事では、Google Apps Script（GAS）を使って、特定のカレンダー（例：仕事用）の予定を別のカレンダー（例：プライベート用）に「時間帯のみ」自動的に追記する方法を紹介します。これにより、予定の詳細を共有せずに、時間の衝突だけを簡単に確認できるようになります。

## やってみた結果

この記事の内容を実践することで、次のような効果が得られます：

- 複数のカレンダーを使い分けながらも、予定の衝突を未然に防げるようになる
- プライベートな予定の詳細を共有せずに、時間の確保だけを表示できる
- カレンダーが自動的に同期されるため、手動で予定をコピーする手間が不要になる
- コード数十行の簡単なスクリプトで、カレンダー管理の効率が大幅に向上する

## 開発環境
- Googleアカウント（2つ以上のカレンダーにアクセス可能なもの）
- ウェブブラウザ（Google Apps Scriptエディタにアクセスできるもの）
- JavaScriptの基本的な知識（初心者でも理解できるレベル）

## 事前準備

1. 2つのGoogleカレンダーを用意します
   - ソースカレンダー：同期元となる予定が入っているカレンダー
   - ターゲットカレンダー：時間帯のみを追記する先のカレンダー

2. 両方のカレンダーに対する編集権限があることを確認します

## やったこと

### 1. Google Apps Scriptプロジェクトの作成

まず、Google Apps Scriptプロジェクトを作成します。

1. [Google Apps Script](https://script.google.com/) にアクセスします
2. 「新しいプロジェクト」をクリックします
3. プロジェクト名を「CalendarTimeSync」などわかりやすい名前に変更します

### 2. カレンダーIDの取得

スクリプトで使用するカレンダーのIDを取得します。

1. [Googleカレンダー](https://calendar.google.com/)にアクセスします
2. 左側のカレンダー一覧からソースカレンダー（同期元）の「その他」メニュー（3つの点）をクリックします
3. 「設定と共有」を選択します
4. 「カレンダーの統合」セクションにある「カレンダーID」をコピーしておきます
5. 同様の手順でターゲットカレンダー（同期先）のIDもコピーしておきます

### 3. 同期スクリプトの実装

Google Apps Scriptエディタに以下のコードを入力します：

```javascript
// カレンダーの設定
const SOURCE_CALENDAR_ID = 'your_source_calendar_id@group.calendar.google.com'; // ソースカレンダーのID
const TARGET_CALENDAR_ID = 'your_target_calendar_id@group.calendar.google.com'; // ターゲットカレンダーのID
const SYNC_EVENT_PREFIX = '[予約済] '; // ターゲットカレンダーに作成するイベントのタイトル接頭辞

/**
 * メイン関数：ソースカレンダーからターゲットカレンダーへ時間帯のみを同期する
 */
function syncCalendarTimes() {
  try {
    // 1. 同期状態を初期化（前回の同期イベントを削除）
    clearPreviousSyncEvents();
    
    // 2. ソースカレンダーから今後の予定を取得（2週間分）
    const now = new Date();
    const twoWeeksLater = new Date(now.getTime() + 14 * 24 * 60 * 60 * 1000);
    const events = CalendarApp.getCalendarById(SOURCE_CALENDAR_ID)
      .getEvents(now, twoWeeksLater);
    
    Logger.log(`同期対象イベント数: ${events.length}`);
    
    // 3. 各イベントをターゲットカレンダーに時間帯のみコピー
    for (const event of events) {
      // イベントの基本情報を取得
      const startTime = event.getStartTime();
      const endTime = event.getEndTime();
      
      // ターゲットカレンダーに新しいイベントを作成（詳細なし）
      const targetCalendar = CalendarApp.getCalendarById(TARGET_CALENDAR_ID);
      const newEvent = targetCalendar.createEvent(
        SYNC_EVENT_PREFIX + '時間確保', // タイトル
        startTime,                     // 開始時間
        endTime,                       // 終了時間
        {
          description: '他のカレンダーから同期された予約済み時間です',
          // オプション: 元のイベントの場所を引き継ぎたい場合
          // location: event.getLocation()
        }
      );
      
      // イベントを識別しやすくするための設定
      // 特定の色を設定（8:グレー）
      newEvent.setColor('8');
      
      Logger.log(`同期完了: ${startTime} - ${endTime}`);
    }
    
    Logger.log('カレンダー同期が完了しました');
  } catch (error) {
    Logger.log(`エラーが発生しました: ${error.message}`);
  }
}

/**
 * 前回の同期で作成したイベントをクリアする
 */
function clearPreviousSyncEvents() {
  const now = new Date();
  const oneMonthLater = new Date(now.getTime() + 30 * 24 * 60 * 60 * 1000);
  
  // ターゲットカレンダーから既存の同期イベントを検索
  const targetCalendar = CalendarApp.getCalendarById(TARGET_CALENDAR_ID);
  const existingEvents = targetCalendar.getEvents(now, oneMonthLater);
  
  let deletedCount = 0;
  
  // 同期イベントを識別して削除
  for (const event of existingEvents) {
    if (event.getTitle().startsWith(SYNC_EVENT_PREFIX)) {
      event.deleteEvent();
      deletedCount++;
    }
  }
  
  Logger.log(`以前の同期イベント ${deletedCount} 件を削除しました`);
}
```

### 4. カレンダーIDの設定

スクリプトの冒頭にある以下の変数の値を、先ほど取得したカレンダーIDに書き換えます：

```javascript
const SOURCE_CALENDAR_ID = 'your_source_calendar_id@group.calendar.google.com'; // ここを実際のIDに変更
const TARGET_CALENDAR_ID = 'your_target_calendar_id@group.calendar.google.com'; // ここを実際のIDに変更
```

### 5. スクリプトの動作テスト

1. スクリプトエディタで「実行」ボタンをクリックします
2. 初回実行時は権限の承認を求められるので、承認します
   - 「詳細」→「安全ではないページに移動」→「許可」の順にクリックします
3. 実行結果はログに表示されます（「表示」→「ログ」）
4. Googleカレンダーを開いて、ターゲットカレンダーに予定が正しくコピーされたかを確認します

### 6. 定期実行の設定

カレンダーを自動的に定期同期するように設定します。

1. スクリプトエディタで「トリガー」アイコン（時計マーク）をクリックします
2. 「トリガーを追加」をクリックします
3. 以下のように設定します：
   - 実行する関数: `syncCalendarTimes`
   - イベントのソース: `時間主導型`
   - 時間ベースのトリガーのタイプ: `時間ベースのタイマー`
   - 時間の間隔: 任意（例: `1時間おき`）
4. 「保存」をクリックします

これで、設定した時間間隔で自動的にカレンダーの同期が行われるようになります。

## 結論に対しての補足

### カスタマイズの可能性

- イベントのタイトルや説明をカスタマイズすることができます
- 特定の条件（例：特定のキーワードを含む予定のみ）を満たすイベントだけを同期することも可能です
- 同期の対象期間を変更することもできます（現在は2週間先までを同期）

### 注意点

- スクリプトを実行するGoogleアカウントが両方のカレンダーにアクセスできる必要があります
- 同期は一方向のみです（ソースカレンダー→ターゲットカレンダー）
- ソースカレンダーのイベントが更新された場合、次回の同期タイミングまで反映されません

### 関連サービス

- [Google カレンダーAPI](https://developers.google.com/calendar) - より高度なカレンダー連携を行いたい場合
- [Zapier](https://zapier.com/) や [IFTTT](https://ifttt.com/) - GASを使わずにカレンダー連携を行いたい場合

## おわりに

Google Apps Script（GAS）を使うことで、数十行のコードだけで複数のカレンダーを効率的に連携させることができました。このスクリプトにより、予定の詳細を共有せずに時間の衝突を防ぐという課題を解決できます。

GASは無料で使用でき、プログラミングの初心者でも比較的簡単に始められるのが魅力です。今回紹介したカレンダー連携以外にも、さまざまなGoogle製品と連携したワークフロー自動化に応用できます。ぜひ、自分のニーズに合わせてカスタマイズしてみてください。
