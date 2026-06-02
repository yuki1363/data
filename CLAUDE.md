# CLAUDE.md — ユーティリティ日報入力アプリ
## プロジェクト概要
工場ユーティリティ設備の日常点検データをスマートフォンから入力し、SharePoint リストに自動保存する PowerApps アプリの構築プロジェクト。
- **アプリ種別**: Power Apps キャンバスアプリ（スマホ最適化）
- **データ保存先**: SharePoint Online リスト
- **入力頻度**: 毎日（日報形式）
- **主な利用者**: 工場保全技術課 点検担当者
---
## ディレクトリ構成
```
utility-daily-report/
├── CLAUDE.md                  # このファイル
├── docs/
│   ├── sharepoint-list-design.md   # SharePointリスト列定義
│   ├── screen-design.md            # 画面設計・UI仕様
│   └── deployment-guide.md         # 展開・運用手順
├── powerapps/
│   ├── formulas/
│   │   ├── submit.fx              # 送信処理 Power Fx
│   │   ├── validation.fx          # 入力バリデーション
│   │   └── navigation.fx          # 画面遷移ロジック
│   └── export/
│       └── UtilityDailyReport.msapp  # アプリエクスポートファイル
└── sharepoint/
    ├── list-schema.json           # リスト列スキーマ定義
    └── pnp-provision.ps1          # PnP PowerShell プロビジョニング
```
---
## SharePoint リスト設計
### リスト名: `UtilityDailyReport`（ユーティリティ日報）
| # | 列名（表示名） | 内部名（英語） | 型 | 必須 | 備考 |
|---|---|---|---|---|---|
| 1 | 点検日時 | InspectionDateTime | 日付と時刻 | ✅ | 既定値: 今日 |
| 2 | 灯油サブタンクレベル（液面） | KeroseneSubTankLevel | 選択肢 | | 上 / 中 / 下 |
| 3 | 温水ボイラー灯油メーター | HotWaterBoilerKerosene_L | 数値 | | 単位: L |
| 4 | 運転開始時間 | OperationStartTime | テキスト | | HH:MM 形式 |
| 5 | 運転終了時間 | OperationEndTime | テキスト | | HH:MM 形式 |
| 6 | 空気圧縮機運転号機 | AirCompressorUnit | 選択肢（**複数選択可**） | | 1号機 / 2号機 / 3号機 |
| 7 | ヘッダー圧力 | HeaderPressure_MPa | 数値 | | 単位: MPa |
| 8 | 温水ポンプ運転号機 | HotWaterPumpUnit | 選択肢（**複数選択可**） | | 1号機 / 2号機 / 3号機 |
| 9 | 温水ポンプ圧力 | HotWaterPumpPressure_MPa | 数値 | | 単位: MPa |
| 10 | 温水タンク容量 | HotWaterTankVolume_m3 | 数値 | | 単位: ㎥ |
| 11 | 温水タンク内温度 | HotWaterTankTemp_C | 数値 | | 単位: ℃ |
| 12 | 中水タンク容量 | GreyWaterTankVolume_m3 | 数値 | | 単位: ㎥ |
| 13 | 飲料水タンク容量 | DrinkingWaterTankVolume_m3 | 数値 | | 単位: ㎥ |
| 14 | PWタンク容量 | PWTankVolume_m3 | 数値 | | 単位: ㎥ |
| 15 | 市水温度 | CityWaterTemp_C | 数値 | | 単位: ℃ |
| 16 | 灯油タンク容量TK605 | KeroseTankTK605_m3 | 数値 | | 単位: ㎥ |
| 17 | 灯油メーター1 | KeroseneMeter1_L | 数値 | | 単位: L |
| 18 | 薬品タンク1 | ChemicalTank1_L | 数値 | | 単位: L |
| 19 | 灯油メーター2 | KeroseneMeter2_L | 数値 | | 単位: L |
| 20 | 薬品タンク2 | ChemicalTank2_L | 数値 | | 単位: L |
| 21 | 蒸気ボイラー運転号機 | SteamBoilerUnit | 選択肢（**複数選択可**） | | 1号機 / 2号機 |
| 22 | 蒸気ボイラー圧力 | SteamBoilerPressure_MPa | 数値 | | 単位: MPa |
| 23 | PW補給水メーター | PWSupplyMeter_L | 数値 | | 単位: L |
| 24 | ボイラー給水ポンプ圧力 | BoilerFeedPumpPressure_MPa | 数値 | | 単位: MPa |
| 25 | ドレン電導度 | DrainConductivity | 数値 | | 単位: μS/cm |
| 26 | ドレンポンプ圧力 | DrainPumpPressure_MPa | 数値 | | 単位: MPa |
| 27 | 総運転時間1 | TotalRunTime1_hr | 数値 | | 単位: hr |
| 28 | 油面確認1 | OilLevel1 | 選択肢 | | OK / 要補充 / 異常 |
| 29 | 総運転時間2 | TotalRunTime2_hr | 数値 | | 単位: hr |
| 30 | 油面確認2 | OilLevel2 | 選択肢 | | OK / 要補充 / 異常 |
| 31 | 総運転時間3 | TotalRunTime3_hr | 数値 | | 単位: hr |
| 32 | 油面確認3 | OilLevel3 | 選択肢 | | OK / 要補充 / 異常 |
| 33 | 特記事項・備考 | Notes | 複数行テキスト | | 異常・メモ等 |
---
## PowerApps 画面設計
### 共通 UI 仕様
| 項目 | 値 |
|---|---|
| 画面サイズ | 390 × 844 px（縦向き固定） |
| ヘッダー色 | #1A3A6E（ネイビー） |
| プログレスバー色 | #2196F3（ブルー） |
| フォント | Segoe UI / 游ゴシック |
| ラベルサイズ | 12pt / Color: RGBA(0,0,0,0.5) |
| 入力値サイズ | 14pt / Color: Black |
| ボタン（次へ） | Height 44px / Fill #1A3A6E / FontColor White |
| ボタン（戻る） | Height 44px / Fill Transparent / Border #999 |
| ボタン（送信） | Height 44px / Fill #1A7E4A（グリーン）|
---
### 画面一覧・詳細仕様
---
#### scrHome — ホーム画面
| 要素 | コントロール | プロパティ / Power Fx |
|---|---|---|
| 日付表示 | Label | `Text: Text(Today(),"yyyy年mm月dd日")` |
| 本日入力済バッジ | Label | `Text: If(varTodaySubmitted,"✅ 本日入力済","● 本日未入力")` / 色を条件分岐 |
| 新規入力ボタン | Button | `OnSelect: Navigate(scrInput1, ScreenTransition.Fade)` |
| 履歴ボタン | Button | `OnSelect: Navigate(scrHistory, ScreenTransition.Fade)` |
| OnVisible | — | `Set(varTodaySubmitted, CountRows(Filter(UtilityDailyReport, DateValue(Text(InspectionDateTime,"yyyy/mm/dd"))=Today()))>0)` |
---
#### scrInput1 — 入力①（灯油・ボイラー・運転時間）
担当項目: 点検日時 / 灯油サブタンクレベル / 温水ボイラー灯油メーター / 運転開始・終了時間
| 要素 | コントロール | プロパティ / Power Fx |
|---|---|---|
| プログレスバー | Rectangle | `Width: Parent.Width * 0.2` |
| 点検日時 | DatePicker | `DefaultDate: Today()` / `OnChange: Set(varDateTime, Self.SelectedDate)` |
| サブタンクレベル | Toggle/Chip | ButtonGroup コントロール: Items=["上","中","下"] / `OnChange: Set(varSubTankLevel, Self.Selected.Value)` |
| 灯油メーター | TextInput | `Format: Number` / `Hint: "L単位で入力"` / `KeyboardType: Decimal` |
| 開始時刻 | TextInput | `Hint: "HH:MM"` / `MaxLength: 5` |
| 終了時刻 | TextInput | `Hint: "HH:MM"` / `MaxLength: 5` |
| 次へボタン | Button | `OnSelect: Navigate(scrInput2, ScreenTransition.Fade)` |
---
#### scrInput2 — 入力②（空気圧縮機・温水ポンプ・温水タンク）
担当項目: 空気圧縮機号機 / ヘッダー圧力 / 温水ポンプ号機 / 温水ポンプ圧力 / 温水タンク容量・温度
| 要素 | コントロール | プロパティ / Power Fx |
|---|---|---|
| プログレスバー | Rectangle | `Width: Parent.Width * 0.4` |
| 空気圧縮機号機 | CheckBox × 3 または ToggleChip | 各号機に CheckBox / `OnChange: Set(varAirComp, Concat(Filter([{v:"1号機"},{v:"2号機"},{v:"3号機"}], chk.Value), v & ";"))` → 選択結果を ";" 区切りテキストで変数保持 |
| 選択中表示 | Label | `Text: If(IsBlank(varAirComp), "未選択", varAirComp)` / 未選択時は警告色 |
| ヘッダー圧力 | TextInput | `KeyboardType: Decimal` / `Hint: "MPa"` |
| 温水ポンプ号機 | CheckBox × 3 または ToggleChip | 同上パターン / 変数: `varHWPump` |
| 温水ポンプ圧力 | TextInput | `KeyboardType: Decimal` |
| 温水タンク容量 | TextInput | `KeyboardType: Decimal` / `Hint: "㎥"` |
| 温水タンク温度 | TextInput | `KeyboardType: Decimal` / `Hint: "℃"` |
| 戻る / 次へ | Button | `Navigate(scrInput1)` / `Navigate(scrInput3)` |
---
#### scrInput3 — 入力③（各種タンク・薬品）
担当項目: 中水・飲料水・PWタンク / 市水温度 / 灯油タンクTK605 / 灯油メーター1-2 / 薬品タンク1-2
| 要素 | コントロール | プロパティ |
|---|---|---|
| プログレスバー | Rectangle | `Width: Parent.Width * 0.6` |
| 中水タンク容量 | TextInput | `KeyboardType: Decimal` / `Hint: "㎥"` |
| 飲料水タンク容量 | TextInput | `KeyboardType: Decimal` / `Hint: "㎥"` |
| PWタンク容量 | TextInput | `KeyboardType: Decimal` / `Hint: "㎥"` |
| 市水温度 | TextInput | `KeyboardType: Decimal` / `Hint: "℃"` |
| 灯油タンクTK605 | TextInput | `KeyboardType: Decimal` / `Hint: "㎥"` |
| 灯油メーター1・2 | TextInput × 2 | 横並び配置 / `Hint: "L"` |
| 薬品タンク1・2 | TextInput × 2 | 横並び配置 / `Hint: "L"` |
| 戻る / 次へ | Button | `Navigate(scrInput2)` / `Navigate(scrInput4)` |
---
#### scrInput4 — 入力④（蒸気ボイラー・ドレン）
担当項目: 蒸気ボイラー号機・圧力 / PW補給水 / 給水ポンプ圧力 / ドレン電導度・ポンプ圧力
| 要素 | コントロール | プロパティ |
|---|---|---|
| プログレスバー | Rectangle | `Width: Parent.Width * 0.8` |
| 蒸気ボイラー号機 | CheckBox × 2 または ToggleChip | 変数: `varSteamBoiler` / 選択結果を ";" 区切りテキストで保持 |
| 蒸気ボイラー圧力 | TextInput | `KeyboardType: Decimal` / `Hint: "MPa"` |
| PW補給水メーター | TextInput | `KeyboardType: Decimal` / `Hint: "L"` |
| 給水ポンプ圧力 | TextInput | `KeyboardType: Decimal` / `Hint: "MPa"` |
| ドレン電導度 | TextInput | `KeyboardType: Decimal` / `Hint: "μS/cm"` |
| ドレンポンプ圧力 | TextInput | `KeyboardType: Decimal` / `Hint: "MPa"` |
| 戻る / 次へ | Button | `Navigate(scrInput3)` / `Navigate(scrInput5)` |
---
#### scrInput5 — 入力⑤（号機別 総運転時間・油面確認）
担当項目: 総運転時間1-3（hr）/ 油面確認1-3 / 特記事項
| 要素 | コントロール | プロパティ |
|---|---|---|
| プログレスバー | Rectangle | `Width: Parent.Width` （100%）|
| 総運転時間1・2・3 | TextInput × 3 | `KeyboardType: Decimal` / `Hint: "hr（例: 8.5）"` |
| 油面確認1・2・3 | Dropdown × 3 | `Items: ["OK","要補充","異常"]` / 横並び（総運転時間と同行） |
| 特記事項 | TextInput | `Mode: Multiline` / `Height: 80px` |
| 戻る | Button | `Navigate(scrInput4)` |
| 確認画面へ | Button | `OnSelect: Navigate(scrConfirm, ScreenTransition.Fade)` |
---
#### scrConfirm — 確認・送信画面
| 要素 | コントロール | プロパティ / Power Fx |
|---|---|---|
| 確認テーブル表示 | HTMLText or Gallery | 全33項目をグループ別に一覧表示 |
| 修正ボタン | Button | `OnSelect: Navigate(scrInput1)` |
| 送信ボタン | Button | `Fill: RGBA(26,126,74,1)` / `OnSelect: [Patch処理 → Navigate(scrComplete)]` |
| 送信 Power Fx | — | 下記「データ送信」参照 |
---
#### scrComplete — 送信完了画面
| 要素 | コントロール | プロパティ |
|---|---|---|
| 完了アイコン | Icon / Label | ✅ / サイズ 48pt |
| 完了メッセージ | Label | `Text: "送信が完了しました"` |
| 日付サブメッセージ | Label | `Text: Text(Today(),"yyyy年mm月dd日") & " 分を保存しました"` |
| ホームへ戻るボタン | Button | `OnSelect: Navigate(scrHome, ScreenTransition.Fade)` |
---
#### scrHistory — 履歴閲覧画面
| 要素 | コントロール | プロパティ / Power Fx |
|---|---|---|
| 検索ボックス | TextInput | `OnChange: Set(varSearch, Self.Text)` |
| データ一覧 | Gallery (Vertical) | `Items: Filter(UtilityDailyReport, IsBlank(varSearch) Or Text(InspectionDateTime) = varSearch)` |
| 日付ラベル | Label（Gallery内） | `Text: Text(ThisItem.InspectionDateTime,"yyyy/mm/dd")` |
| メタ情報 | Label（Gallery内） | `Text: ThisItem.AirCompressorUnit & " / " & ThisItem.SteamBoilerUnit` |
| 詳細矢印 | Icon | `OnSelect: Set(varSelectedRecord, ThisItem); Navigate(scrDetail)` |
---
## Power Fx 主要ロジック
### データ送信（scrConfirm の送信ボタン）
```powerfx
// 送信処理
Patch(
    UtilityDailyReport,
    Defaults(UtilityDailyReport),
    {
        InspectionDateTime: DateTimePicker1.SelectedDate,
        KeroseneSubTankLevel: ddKeroseneSubTank.Selected.Value,   // 上/中/下
        HotWaterBoilerKerosene_L: Value(txtHWBoilerKerosene.Text),
        OperationStartTime: txtStartTime.Text,
        OperationEndTime: txtEndTime.Text,
        AirCompressorUnit: varAirComp,   // 例: "1号機;2号機"（複数選択 ";" 区切り）
        HeaderPressure_MPa: Value(txtHeaderPressure.Text),
        HotWaterPumpUnit: varHWPump,     // 例: "2号機;3号機"
        SteamBoilerUnit: varSteamBoiler, // 例: "1号機;2号機"
        TotalRunTime1_hr: Value(txtRunTime1.Text),
        OilLevel1: ddOilLevel1.Selected.Value,
        TotalRunTime2_hr: Value(txtRunTime2.Text),
        OilLevel2: ddOilLevel2.Selected.Value,
        TotalRunTime3_hr: Value(txtRunTime3.Text),
        OilLevel3: ddOilLevel3.Selected.Value,
        // ... 残り数値列
        Notes: txtNotes.Text
    }
);
Navigate(scrComplete, ScreenTransition.Fade);
```
### 入力バリデーション（送信前チェック）
```powerfx
// 必須項目チェック
If(
    IsBlank(DateTimePicker1.SelectedDate),
    Notify("点検日時を入力してください", NotificationType.Error),
    // 数値範囲チェック例
    Value(txtHeaderPressure.Text) > 2.0,
    Notify("ヘッダー圧力が上限(2.0 MPa)を超えています", NotificationType.Warning),
    // チェック通過→送信処理
    true
)
```
### 今日のデータ確認（二重入力防止）
```powerfx
// ホーム画面 OnVisible
If(
    CountRows(
        Filter(UtilityDailyReport, 
               DateValue(InspectionDateTime) = Today())
    ) > 0,
    Set(varTodaySubmitted, true),
    Set(varTodaySubmitted, false)
)
```
---
## SharePoint リスト作成手順（PnP PowerShell）
```powershell
# sharepoint/pnp-provision.ps1
Connect-PnPOnline -Url "https://[テナント].sharepoint.com/sites/[サイト名]" -UseWebLogin
# リスト作成
New-PnPList -Title "UtilityDailyReport" -Template GenericList
# 列追加（抜粋）
Add-PnPField -List "UtilityDailyReport" -DisplayName "点検日時" -InternalName "InspectionDateTime" -Type DateTime -AddToDefaultView
Add-PnPField -List "UtilityDailyReport" -DisplayName "温水ボイラー灯油メーター(L)" -InternalName "HotWaterBoilerKerosene_L" -Type Number
Add-PnPField -List "UtilityDailyReport" -DisplayName "ヘッダー圧力(MPa)" -InternalName "HeaderPressure_MPa" -Type Number
Add-PnPField -List "UtilityDailyReport" -DisplayName "温水ポンプ圧力(MPa)" -InternalName "HotWaterPumpPressure_MPa" -Type Number
Add-PnPField -List "UtilityDailyReport" -DisplayName "総運転時間1(hr)" -InternalName "TotalRunTime1_hr" -Type Number
Add-PnPField -List "UtilityDailyReport" -DisplayName "総運転時間2(hr)" -InternalName "TotalRunTime2_hr" -Type Number
Add-PnPField -List "UtilityDailyReport" -DisplayName "総運転時間3(hr)" -InternalName "TotalRunTime3_hr" -Type Number
# ... 他の数値列を同様に追加
# 選択肢列（灯油サブタンクレベル）
Add-PnPFieldFromXml -List "UtilityDailyReport" -FieldXml '
<Field Type="Choice" DisplayName="灯油サブタンクレベル" Name="KeroseneSubTankLevel">
  <CHOICES>
    <CHOICE>上</CHOICE>
    <CHOICE>中</CHOICE>
    <CHOICE>下</CHOICE>
  </CHOICES>
</Field>'
# 複数選択可・選択肢列（空気圧縮機運転号機）
Add-PnPFieldFromXml -List "UtilityDailyReport" -FieldXml '
<Field Type="MultiChoice" DisplayName="空気圧縮機運転号機" Name="AirCompressorUnit">
  <CHOICES>
    <CHOICE>1号機</CHOICE>
    <CHOICE>2号機</CHOICE>
    <CHOICE>3号機</CHOICE>
  </CHOICES>
</Field>'
# 複数選択可・選択肢列（温水ポンプ運転号機）
Add-PnPFieldFromXml -List "UtilityDailyReport" -FieldXml '
<Field Type="MultiChoice" DisplayName="温水ポンプ運転号機" Name="HotWaterPumpUnit">
  <CHOICES>
    <CHOICE>1号機</CHOICE>
    <CHOICE>2号機</CHOICE>
    <CHOICE>3号機</CHOICE>
  </CHOICES>
</Field>'
# 複数選択可・選択肢列（蒸気ボイラー運転号機）
Add-PnPFieldFromXml -List "UtilityDailyReport" -FieldXml '
<Field Type="MultiChoice" DisplayName="蒸気ボイラー運転号機" Name="SteamBoilerUnit">
  <CHOICES>
    <CHOICE>1号機</CHOICE>
    <CHOICE>2号機</CHOICE>
  </CHOICES>
</Field>'
# 選択肢列（油面確認 × 3）
foreach ($i in 1..3) {
    Add-PnPFieldFromXml -List "UtilityDailyReport" -FieldXml "
<Field Type='Choice' DisplayName='油面確認$i' Name='OilLevel$i'>
  <CHOICES>
    <CHOICE>OK</CHOICE>
    <CHOICE>要補充</CHOICE>
    <CHOICE>異常</CHOICE>
  </CHOICES>
</Field>"
}
```
---
## PowerApps ← SharePoint 接続手順
1. PowerApps Studio を開く
2. **データ** → **データの追加** → **SharePoint**
3. サイトURL入力 → `UtilityDailyReport` リストを選択
4. アプリ内で `UtilityDailyReport` としてデータソース参照可能
---
## 開発・展開フロー
```
1. SharePointリスト作成（PnP PowerShell or 手動）
   ↓
2. PowerApps Studio でキャンバスアプリ新規作成
   ↓
3. 各画面・コントロール実装
   ↓
4. バリデーション・送信ロジック実装
   ↓
5. スマホ実機テスト（iOS / Android PowerApps アプリ）
   ↓
6. 組織に公開（Power Platform 管理センター）
   ↓
7. 利用者へ共有（AAD グループ or 個別ユーザー）
```
---
## 複数選択号機の実装詳細
### SharePoint 列型: MultiChoice（複数選択可）
`Type="MultiChoice"` を使うことで SharePoint 側が配列として保存する。PowerApps からは `;#` 区切り文字列で渡す。
### PowerApps 実装パターン（ToggleChip方式）
各号機を個別の変数（Boolean）で管理し、Patch時に結合する。
```powerfx
// 初期化（scrInput2 の OnVisible）
Set(varAC1, false); Set(varAC2, false); Set(varAC3, false);
Set(varHW1, false); Set(varHW2, false); Set(varHW3, false);
// チップボタンの OnSelect（例: 1号機ボタン）
Set(varAC1, !varAC1)
// チップの Fill（選択中=ネイビー / 非選択=白）
If(varAC1, ColorValue("#1A3A6E"), RGBA(255,255,255,1))
// 選択結果テキスト生成（Concat）
Set(varAirCompResult,
    Concat(
        Filter(
            [{unit:"1号機", sel:varAC1},
             {unit:"2号機", sel:varAC2},
             {unit:"3号機", sel:varAC3}],
            sel = true
        ),
        unit, ";"
    )
)
// 結果例: "1号機;2号機"
```
### バリデーション（未選択チェック）
```powerfx
// 送信前チェック（scrConfirm 送信ボタン OnSelect 内）
If(
    IsBlank(varAirCompResult),
    Notify("空気圧縮機の運転号機を選択してください", NotificationType.Error),
    IsBlank(varHWPumpResult),
    Notify("温水ポンプの運転号機を選択してください", NotificationType.Error),
    IsBlank(varSteamBoilerResult),
    Notify("蒸気ボイラーの運転号機を選択してください", NotificationType.Error),
    // 全チェック通過 → Patch送信
    Patch(UtilityDailyReport, Defaults(UtilityDailyReport), { ... })
)
```
### SharePoint MultiChoice への Patch
```powerfx
// MultiChoice 列へは {Value: "値"} のテーブル形式で渡す
AirCompressorUnit: ForAll(
    Split(varAirCompResult, ";"),
    {Value: Result}
),
HotWaterPumpUnit: ForAll(
    Split(varHWPumpResult, ";"),
    {Value: Result}
),
SteamBoilerUnit: ForAll(
    Split(varSteamBoilerResult, ";"),
    {Value: Result}
)
```
### 確認画面での表示
```powerfx
// MultiChoice を文字列で表示
Text: Concat(varAirCompResult_table, Value, " / ")
// または保存した文字列変数をそのまま表示
Text: Substitute(varAirCompResult, ";", " / ")
// 表示例: "1号機 / 2号機"
```
---
## 注意事項・制約
- SharePoint リスト列の **内部名は作成後に変更不可**。初回設定を慎重に行うこと。
- 数値列の小数点桁数は SharePoint 列設定で制御（例: MPa は小数2桁）。
- PowerApps の **Patch 関数は非同期**。送信後に Navigate で画面遷移すること。
- スマホからの利用は **Microsoft PowerApps アプリ（iOS/Android）** が必要。
- SharePoint サイトへのアクセス権限（投稿権限以上）を利用者全員に付与すること。
- オフライン入力は **PowerApps のオフライン機能（SaveData/LoadData）** を使う場合は別途実装が必要。
---
## 将来的な拡張案
- Power Automate でデータ入力時に管理者へ **メール通知**
- Power BI で SharePoint リストを参照した **ユーティリティダッシュボード**
- 異常値検知時の **アラート自動送信**（Teams / メール）
- 写真添付機能（PowerApps Camera コントロール → SharePoint ドキュメントライブラリ）
---
*最終更新: 2026-06-02 (rev.3 — 号機3項目を複数選択対応・MultiChoice/ToggleChip実装追加) | 担当: 保全技術課*
