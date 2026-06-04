# Power Automate 設定ガイド
## メール受信 → SharePoint 自動保存フロー
---
## 全体フロー
```
GitHub Pages からメール送信（mailto）
    ↓
指定 Outlook アドレスにメール着信
    ↓
Power Automate トリガー起動
    ↓
メール本文から JSON を抽出
    ↓
SharePoint リストに1件追加
    ↓
（任意）完了通知メール返信
```
---
## STEP 1 — 専用メールアドレスの準備
Power Automate が監視する Outlook アドレスを決める。
| 方法 | 例 | 備考 |
|---|---|---|
| 既存のメールアドレス | `jibun@company.com` | 受信トレイが混在する |
| **共有メールボックス（推奨）** | `utility-report@company.com` | 専用で管理しやすい |
> 共有メールボックスは Microsoft 365 管理センター → Exchange → 共有メールボックスから作成できる。
---
## STEP 2 — index.html の送信先アドレスを設定
`index.html` の先頭 CONFIG 部分を編集：
```javascript
const CONFIG = {
  SEND_TO: 'utility-report@company.com', // ← ここを実際のアドレスに変更
  SUBJECT_PREFIX: 'ユーティリティ日報',
};
```
---
## STEP 3 — Power Automate フロー作成
### 3-1. フロー新規作成
1. `make.powerautomate.com` を開く
2. **「作成」→「自動化クラウドフロー」**
3. フロー名: `ユーティリティ日報_メール受信→SharePoint`
4. トリガー検索: `新しいメールが届いたとき (V3)`（Outlook）
### 3-2. トリガー設定
| 設定項目 | 値 |
|---|---|
| フォルダー | 受信トレイ |
| 宛先 | `utility-report@company.com` |
| 件名フィルター | `ユーティリティ日報` |
| 添付ファイルを含める | いいえ |
| 重要度 | 標準 |
### 3-3. アクション①「変数の初期化」（JSON文字列抽出用）
**アクション名:** `変数の初期化 — JSONテキスト`
| 設定 | 値 |
|---|---|
| 名前 | `jsonText` |
| 型 | 文字列 |
| 値 | （空白） |
### 3-4. アクション②「変数の設定」（JSON抽出）
**アクション名:** `JSON本文を抽出`
| 設定 | 値 |
|---|---|
| 名前 | `jsonText` |
| 値（式） | 以下の式を貼り付け |
```
// Power Automate 式エディタに貼り付け
substring(
  body('新しいメールが届いたとき_(V3)')?['body'],
  add(indexOf(body('新しいメールが届いたとき_(V3)')?['body'], '===JSON_START==='), 15),
  sub(
    indexOf(body('新しいメールが届いたとき_(V3)')?['body'], '===JSON_END==='),
    add(indexOf(body('新しいメールが届いたとき_(V3)')?['body'], '===JSON_START==='), 15)
  )
)
```
### 3-5. アクション③「JSONの解析」
**アクション名:** `JSONを解析`
| 設定 | 値 |
|---|---|
| コンテンツ | `変数 jsonText` |
| スキーマ | 以下のスキーマを貼り付け |
```json
{
  "type": "object",
  "properties": {
    "datetime": { "type": "string" },
    "subTankLevel": { "type": "string" },
    "hwBoilerKerosene": { "type": "string" },
    "startTime": { "type": "string" },
    "endTime": { "type": "string" },
    "airCompressorUnit": { "type": "string" },
    "headerPressure": { "type": "string" },
    "hwPumpUnit": { "type": "string" },
    "hwPumpPressure": { "type": "string" },
    "hwTankVol": { "type": "string" },
    "hwTankTemp": { "type": "string" },
    "greyWaterVol": { "type": "string" },
    "drinkWaterVol": { "type": "string" },
    "pwTankVol": { "type": "string" },
    "cityWaterTemp": { "type": "string" },
    "tk605Vol": { "type": "string" },
    "keroMeter1": { "type": "string" },
    "chemTank1": { "type": "string" },
    "keroMeter2": { "type": "string" },
    "chemTank2": { "type": "string" },
    "steamBoilerUnit": { "type": "string" },
    "steamBoilerPressure": { "type": "string" },
    "pwSupplyMeter": { "type": "string" },
    "boilerFeedPressure": { "type": "string" },
    "drainConductivity": { "type": "string" },
    "drainPumpPressure": { "type": "string" },
    "runTime1": { "type": "string" },
    "oilLevel1": { "type": "string" },
    "runTime2": { "type": "string" },
    "oilLevel2": { "type": "string" },
    "runTime3": { "type": "string" },
    "oilLevel3": { "type": "string" },
    "notes": { "type": "string" }
  }
}
```
### 3-6. アクション④「アイテムの作成」（SharePoint）
**アクション名:** `SharePoint にアイテムを作成`
| 設定 | 値 |
|---|---|
| サイトのアドレス | `https://[テナント].sharepoint.com/sites/[サイト名]` |
| リスト名 | `UtilityDailyReport`（または日本語リスト名） |
**列マッピング（SharePoint列 → JSONフィールド）:**
> ※ 列名はSharePointで作成した日本語名をそのまま選ぶ
| SharePoint列名 | 動的コンテンツ（JSON解析の出力） |
|---|---|
| 点検日時 | `datetime` |
| 灯油サブタンクレベル（液面） | `subTankLevel` |
| 温水ボイラー灯油メーター | `hwBoilerKerosene` |
| 運転開始時間 | `startTime` |
| 運転終了時間 | `endTime` |
| 空気圧縮機運転号機 | `airCompressorUnit` |
| ヘッダー圧力（Mpa) | `headerPressure` |
| 温水ポンプ運転号機 | `hwPumpUnit` |
| 温水ポンプ圧力(Mpa) | `hwPumpPressure` |
| 温水タンク容量(㎥） | `hwTankVol` |
| 温水タンク内温度（℃） | `hwTankTemp` |
| 中水タンク容量(㎥) | `greyWaterVol` |
| 飲料水タンク容量（㎥） | `drinkWaterVol` |
| PWタンク容量(㎥） | `pwTankVol` |
| 市水温度（℃） | `cityWaterTemp` |
| 灯油タンク容量TK６０５（㎥） | `tk605Vol` |
| 灯油メーター１（L) | `keroMeter1` |
| 薬品タンク１（L) | `chemTank1` |
| 灯油メーター2（L) | `keroMeter2` |
| 薬品タンク2（L) | `chemTank2` |
| 蒸気ボイラー運転号機 | `steamBoilerUnit` |
| 蒸気ボイラー圧力（Mpa） | `steamBoilerPressure` |
| PW補給水メーター（L) | `pwSupplyMeter` |
| ボイラー給水ポンプ圧力（Mpa） | `boilerFeedPressure` |
| ドレン電導度 | `drainConductivity` |
| ドレンポンプ圧力 | `drainPumpPressure` |
| 総運転時間１ | `runTime1` |
| 油面確認１ | `oilLevel1` |
| 総運転時間2 | `runTime2` |
| 油面確認2 | `oilLevel2` |
| 総運転時間3 | `runTime3` |
| 油面確認3 | `oilLevel3` |
### 3-7. アクション⑤「メールの送信」（任意・完了通知）
送信者に処理完了を通知する場合に追加。
| 設定 | 値 |
|---|---|
| 宛先 | トリガー送信者のメール |
| 件名 | `✅ 日報をSharePointに保存しました` |
| 本文 | `点検日時: [datetime] のデータを保存しました。` |
---
## STEP 4 — GitHub Pages へのデプロイ
```bash
# 既存のリポジトリにファイルを追加
git clone https://github.com/[ユーザー名]/[リポジトリ名].git
cd [リポジトリ名]
# index.html をコピー
cp /path/to/index.html ./index.html   # または直接アップロード
git add index.html
git commit -m "feat: ユーティリティ日報アプリを追加"
git push origin main
```
### GitHub Pages 有効化
1. リポジトリ → **Settings** → **Pages**
2. Source: **Deploy from a branch**
3. Branch: `main` / `/ (root)`
4. **Save**
5. 数分後に `https://[ユーザー名].github.io/[リポジトリ名]/` でアクセス可能
### スマホのホーム画面に追加（推奨）
**iOS Safari:**
共有ボタン → 「ホーム画面に追加」
**Android Chrome:**
メニュー → 「ホーム画面に追加」
---
## STEP 5 — 動作確認
| # | 確認項目 | 期待結果 |
|---|---|---|
| 1 | GitHub Pages の URL にアクセス | アプリが表示される |
| 2 | 全項目を入力して「✓ メール送信」タップ | メールアプリが開く |
| 3 | メールを送信 | `utility-report@` に届く |
| 4 | Power Automate の実行履歴を確認 | 成功（緑チェック）|
| 5 | SharePoint リストを確認 | 1件追加されている |
---
## トラブルシューティング
| 症状 | 原因 | 対処 |
|---|---|---|
| メールアプリが開かない | mailto リンクのブロック | ブラウザの設定でmailtoを許可 |
| Power Automate が起動しない | 件名フィルターの不一致 | フィルター文字列を確認 |
| JSONの解析に失敗 | 本文の改行コードの違い | `===JSON_START===` の前後を `trim()` で処理 |
| SharePointへの保存に失敗 | 列名の不一致 | SharePointの列表示名を再確認 |
| 数値列にエラー | 文字列→数値の型変換 | アイテム作成時に `float(body('JSONを解析')?['headerPressure'])` に変換 |
### 数値列の型変換（SharePoint数値型の場合）
SharePoint の列が「数値」型の場合、文字列のまま渡すとエラーになる。
アイテム作成アクションの該当フィールドで式を使う：
```
float(body('JSONを解析')?['headerPressure'])
```
空値対策：
```
if(equals(body('JSONを解析')?['headerPressure'], ''), null, float(body('JSONを解析')?['headerPressure']))
```
---
*作成日: 2026-06-02 | 担当: 保全技術課*
