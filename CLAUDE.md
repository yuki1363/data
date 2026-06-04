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
---
## 全画面 コントロール配置コード（PowerApps Studio）
### 共通設定値
```
アプリサイズ: 幅 390 × 高さ 844（電話レイアウト）
ヘッダー色: #1A3A6E
完了ボタン色: #1A7E4A
標準フォントサイズ: 14pt
ラベル色: RGBA(80,80,80,1)
入力枠 BorderColor: RGBA(180,180,180,1)
画面余白（X）: 16
コントロール幅: Parent.Width - 32
```
---
### scrHome — ホーム画面
#### lbl_Date（日付表示）
| プロパティ | 値 |
|---|---|
| Text | `Text(Today(), "yyyy年mm月dd日（aaa）")` |
| X | 16 |
| Y | 16 |
| Width | `Parent.Width - 32` |
| Height | 28 |
| Size | 13 |
| Color | `RGBA(80,80,80,1)` |
#### lbl_Title（タイトル）
| プロパティ | 値 |
|---|---|
| Text | `"ユーティリティ日報"` |
| X | 16 |
| Y | 44 |
| Width | `Parent.Width - 32` |
| Height | 40 |
| Size | 22 |
| FontWeight | `FontWeight.Bold` |
| Color | `RGBA(26,58,110,1)` |
#### lbl_TodayStatus（本日入力済みバッジ）
| プロパティ | 値 |
|---|---|
| Text | `If(varTodaySubmitted, "✅ 本日入力済み", "● 本日未入力")` |
| X | 16 |
| Y | 90 |
| Width | 160 |
| Height | 28 |
| Size | 12 |
| Fill | `If(varTodaySubmitted, RGBA(232,248,238,1), RGBA(255,235,235,1))` |
| Color | `If(varTodaySubmitted, RGBA(26,126,74,1), RGBA(180,30,30,1))` |
| BorderColor | `If(varTodaySubmitted, ColorValue("#1A7E4A"), ColorValue("#cc3333"))` |
| BorderThickness | 1 |
| RadiusTopLeft / TopRight / BottomLeft / BottomRight | 12 |
| PaddingLeft | 10 |
| Align | `Align.Left` |
#### btn_NewEntry（本日分を入力するボタン）
| プロパティ | 値 |
|---|---|
| Text | `"📋 本日分を入力する"` |
| X | 16 |
| Y | 150 |
| Width | `Parent.Width - 32` |
| Height | 60 |
| Fill | `ColorValue("#1A3A6E")` |
| Color | White |
| RadiusTopLeft / TopRight / BottomLeft / BottomRight | 10 |
| FontWeight | `FontWeight.Semibold` |
| Size | 16 |
| OnSelect | `Navigate(scrInput1, ScreenTransition.Fade)` |
#### btn_History（履歴ボタン）
| プロパティ | 値 |
|---|---|
| Text | `"📂 過去データを見る"` |
| X | 16 |
| Y | 224 |
| Width | `Parent.Width - 32` |
| Height | 56 |
| Fill | White |
| Color | `RGBA(50,50,50,1)` |
| BorderColor | `RGBA(180,180,180,1)` |
| BorderThickness | 1 |
| RadiusTopLeft / TopRight / BottomLeft / BottomRight | 10 |
| Size | 15 |
| OnSelect | `Navigate(scrHistory, ScreenTransition.Fade)` |
#### OnVisible（ホーム画面）
```powerfx
Set(varTodaySubmitted,
    CountRows(
        Filter(UtilityDailyReport,
            Text(DateValue(Text('点検日時',"yyyy/mm/dd")),"yyyy/mm/dd") = Text(Today(),"yyyy/mm/dd")
        )
    ) > 0
)
```
---
### scrInput1 — 入力①（灯油・ボイラー・運転時間）
#### rect_Header（ヘッダー背景）
| プロパティ | 値 |
|---|---|
| X | 0 |
| Y | 0 |
| Width | `Parent.Width` |
| Height | 50 |
| Fill | `ColorValue("#1A3A6E")` |
#### lbl_HeaderTitle
| プロパティ | 値 |
|---|---|
| Text | `"灯油・ボイラー・運転時間"` |
| X | 16 |
| Y | 0 |
| Width | `Parent.Width - 80` |
| Height | 50 |
| Color | White |
| Size | 14 |
| FontWeight | `FontWeight.Semibold` |
#### lbl_Step
| プロパティ | 値 |
|---|---|
| Text | `"1 / 5"` |
| X | `Parent.Width - 60` |
| Y | 0 |
| Width | 50 |
| Height | 50 |
| Color | `RGBA(255,255,255,0.7,1)` |
| Size | 12 |
| Align | `Align.Right` |
#### rect_Progress（プログレスバー）
| プロパティ | 値 |
|---|---|
| X | 0 |
| Y | 50 |
| Width | `Parent.Width * 0.2` |
| Height | 4 |
| Fill | `ColorValue("#2196F3")` |
#### lbl_Section1（セクションラベル — 灯油系統）
| プロパティ | 値 |
|---|---|
| Text | `"■ 灯油系統"` |
| X | 16 |
| Y | 68 |
| Width | `Parent.Width - 32` |
| Height | 24 |
| Color | `ColorValue("#1A3A6E")` |
| Size | 12 |
| FontWeight | `FontWeight.Bold` |
#### lbl_SubTankLevel（ラベル）
| プロパティ | 値 |
|---|---|
| Text | `"灯油サブタンクレベル（液面）"` |
| X | 16 |
| Y | 96 |
| Height | 24 |
| Size | 12 |
| Color | `RGBA(80,80,80,1)` |
#### btn_SubTank_High / Mid / Low（上・中・下チップ）
| プロパティ | 上 | 中 | 下 |
|---|---|---|---|
| Text | `"上"` | `"中"` | `"下"` |
| X | 16 | 100 | 184 |
| Y | 124 | 124 | 124 |
| Width | 76 | 76 | 76 |
| Height | 40 | 40 | 40 |
| Fill | `If(varSubTank="上", ColorValue("#1A3A6E"), White)` | `If(varSubTank="中", ColorValue("#1A3A6E"), White)` | `If(varSubTank="下", ColorValue("#1A3A6E"), White)` |
| Color | `If(varSubTank="上", White, RGBA(0,0,0,0.7,1))` | 同左 | 同左 |
| BorderColor | `ColorValue("#1A3A6E")` | 同左 | 同左 |
| BorderThickness | 1 | 1 | 1 |
| RadiusTopLeft 他 | 6 | 6 | 6 |
| OnSelect | `Set(varSubTank,"上")` | `Set(varSubTank,"中")` | `Set(varSubTank,"下")` |
> ※ サブタンクレベルは単一選択なので変数1つ（varSubTank）で管理
#### txt_HWBoilerKerosene（温水ボイラー灯油メーター）
| プロパティ | 値 |
|---|---|
| HintText | `"温水ボイラー灯油メーター（L）"` |
| X | 16 |
| Y | 178 |
| Width | `Parent.Width - 32` |
| Height | 44 |
| KeyboardType | `KeyboardType.DecimalNumber` |
| Format | `TextFormat.Number` |
| BorderColor | `RGBA(180,180,180,1)` |
| BorderThickness | 1 |
| RadiusTopLeft 他 | 6 |
#### lbl_Section2（運転時間）
| プロパティ | 値 |
|---|---|
| Text | `"■ 運転時間"` |
| X | 16 |
| Y | 238 |
| Height | 24 |
| Color | `ColorValue("#1A3A6E")` |
| Size | 12 |
| FontWeight | `FontWeight.Bold` |
#### txt_StartTime / txt_EndTime（横並び）
| プロパティ | 開始時刻 | 終了時刻 |
|---|---|---|
| HintText | `"開始 HH:MM"` | `"終了 HH:MM"` |
| X | 16 | `Parent.Width/2 + 4` |
| Y | 268 | 268 |
| Width | `Parent.Width/2 - 20` | `Parent.Width/2 - 20` |
| Height | 44 | 44 |
| MaxLength | 5 | 5 |
| KeyboardType | `KeyboardType.Number` | `KeyboardType.Number` |
| BorderColor | `RGBA(180,180,180,1)` | 同左 |
| BorderThickness | 1 | 1 |
| RadiusTopLeft 他 | 6 | 6 |
#### btn_Next1（次へ）
| プロパティ | 値 |
|---|---|
| Text | `"次へ →"` |
| X | 16 |
| Y | `Parent.Height - 60` |
| Width | `Parent.Width - 32` |
| Height | 44 |
| Fill | `ColorValue("#1A3A6E")` |
| Color | White |
| RadiusTopLeft 他 | 6 |
| FontWeight | `FontWeight.Semibold` |
| OnSelect | `Navigate(scrInput2, ScreenTransition.None)` |
#### OnVisible（scrInput1）
```powerfx
Set(varSubTank, "中")
```
---
### scrInput2 — 入力②（空気圧縮機・温水ポンプ・温水タンク）
#### ヘッダー・プログレスバー（共通）
| コントロール | 変更箇所 |
|---|---|
| lbl_HeaderTitle.Text | `"圧縮機・温水ポンプ"` |
| lbl_Step.Text | `"2 / 5"` |
| rect_Progress.Width | `Parent.Width * 0.4` |
#### 空気圧縮機チップ（btn_AC1 / btn_AC2 / btn_AC3）
| プロパティ | 1号機 | 2号機 | 3号機 |
|---|---|---|---|
| Text | `"1号機"` | `"2号機"` | `"3号機"` |
| X | 16 | 109 | 202 |
| Y | 100 | 100 | 100 |
| Width | 85 | 85 | 85 |
| Height | 40 | 40 | 40 |
| Fill | `If(varAC1, ColorValue("#1A3A6E"), White)` | `If(varAC2, ColorValue("#1A3A6E"), White)` | `If(varAC3, ColorValue("#1A3A6E"), White)` |
| Color | `If(varAC1, White, RGBA(0,0,0,0.7,1))` | 同左 | 同左 |
| BorderColor | `ColorValue("#1A3A6E")` | 同左 | 同左 |
| BorderThickness | 1 | 1 | 1 |
| RadiusTopLeft 他 | 6 | 6 | 6 |
| OnSelect | `Set(varAC1,!varAC1)` | `Set(varAC2,!varAC2)` | `Set(varAC3,!varAC3)` |
#### lbl_ACResult（空気圧縮機 選択結果）
| プロパティ | 値 |
|---|---|
| Text | `If(varAC1\|\|varAC2\|\|varAC3, "選択中: "&Substitute(Concat(Filter([{u:"1号機",s:varAC1},{u:"2号機",s:varAC2},{u:"3号機",s:varAC3}],s=true),u,";"),";",", "), "⚠ 1つ以上選択してください")` |
| X | 16 |
| Y | 148 |
| Width | `Parent.Width - 32` |
| Height | 30 |
| Fill | `If(varAC1\|\|varAC2\|\|varAC3, RGBA(232,240,252,1), RGBA(255,243,224,1))` |
| Color | `If(varAC1\|\|varAC2\|\|varAC3, RGBA(26,58,110,1), RGBA(146,64,14,1))` |
| BorderColor | `If(varAC1\|\|varAC2\|\|varAC3, ColorValue("#93b4e8"), ColorValue("#f59e0b"))` |
| BorderThickness | 1 |
| RadiusTopLeft 他 | 4 |
| PaddingLeft | 8 |
| Size | 11 |
#### txt_HeaderPressure（ヘッダー圧力）
| プロパティ | 値 |
|---|---|
| HintText | `"ヘッダー圧力（MPa）"` |
| X | 16 |
| Y | 190 |
| Width | `Parent.Width - 32` |
| Height | 44 |
| KeyboardType | `KeyboardType.DecimalNumber` |
| Format | `TextFormat.Number` |
| BorderColor | `RGBA(180,180,180,1)` |
| BorderThickness | 1 |
| RadiusTopLeft 他 | 6 |
#### 温水ポンプチップ（btn_HW1 / btn_HW2 / btn_HW3）
> 空気圧縮機チップと同パターン。Y=270, varHW1〜3 を使用
#### lbl_HWResult（温水ポンプ 選択結果）
> lbl_ACResult と同パターン。Y=318, varHW1〜3 を参照
#### txt_HWPumpPressure / txt_HWTankVol / txt_HWTankTemp
| コントロール | HintText | Y |
|---|---|---|
| txt_HWPumpPressure | `"温水ポンプ圧力（MPa）"` | 360 |
| txt_HWTankVol | `"温水タンク容量（㎥）"` | 416 |
| txt_HWTankTemp | `"温水タンク内温度（℃）"` | 472 |
> 全て Width=`Parent.Width-32`, Height=44, KeyboardType=DecimalNumber
#### btn_Back2 / btn_Next2（戻る・次へ）
| プロパティ | 戻る | 次へ |
|---|---|---|
| Text | `"← 戻る"` | `"次へ →"` |
| X | 16 | `Parent.Width/2 + 8` |
| Y | `Parent.Height - 60` | 同左 |
| Width | `Parent.Width/2 - 24` | `Parent.Width/2 - 8` |
| Height | 44 | 44 |
| Fill | White | `ColorValue("#1A3A6E")` |
| Color | `RGBA(80,80,80,1)` | White |
| BorderColor | `RGBA(180,180,180,1)` | `ColorValue("#1A3A6E")` |
| BorderThickness | 1 | 0 |
| RadiusTopLeft 他 | 6 | 6 |
| OnSelect | `Navigate(scrInput1, ScreenTransition.None)` | `Navigate(scrInput3, ScreenTransition.None)` |
#### OnVisible（scrInput2）
```powerfx
Set(varAC1,false); Set(varAC2,false); Set(varAC3,false);
Set(varHW1,false); Set(varHW2,false); Set(varHW3,false)
```
---
### scrInput3 — 入力③（各種タンク・薬品）
#### ヘッダー・プログレスバー
| コントロール | 変更箇所 |
|---|---|
| lbl_HeaderTitle.Text | `"各種タンク"` |
| lbl_Step.Text | `"3 / 5"` |
| rect_Progress.Width | `Parent.Width * 0.6` |
#### 水系タンク — TextInput 一覧
| コントロール名 | HintText | Y |
|---|---|---|
| txt_GreyWaterVol | `"中水タンク容量（㎥）"` | 80 |
| txt_DrinkWaterVol | `"飲料水タンク容量（㎥）"` | 136 |
| txt_PWTankVol | `"PWタンク容量（㎥）"` | 192 |
| txt_CityWaterTemp | `"市水温度（℃）"` | 248 |
| txt_TK605Vol | `"灯油タンク容量 TK605（㎥）"` | 304 |
> 全て X=16, Width=`Parent.Width-32`, Height=44, KeyboardType=DecimalNumber
#### 灯油メーター・薬品タンク（横並び2列）
| コントロール | HintText | X | Y |
|---|---|---|---|
| txt_KeroMeter1 | `"灯油メーター1（L）"` | 16 | 368 |
| txt_ChemTank1 | `"薬品タンク1（L）"` | `Parent.Width/2+4` | 368 |
| txt_KeroMeter2 | `"灯油メーター2（L）"` | 16 | 424 |
| txt_ChemTank2 | `"薬品タンク2（L）"` | `Parent.Width/2+4` | 424 |
```
Width（横並び）: Parent.Width/2 - 20
```
#### btn_Back3 / btn_Next3
> scrInput2と同パターン。OnSelect: Back→scrInput2, Next→scrInput4
---
### scrInput4 — 入力④（蒸気ボイラー・ドレン）
#### ヘッダー・プログレスバー
| コントロール | 変更箇所 |
|---|---|
| lbl_HeaderTitle.Text | `"蒸気ボイラー・ドレン"` |
| lbl_Step.Text | `"4 / 5"` |
| rect_Progress.Width | `Parent.Width * 0.8` |
#### 蒸気ボイラーチップ（btn_SB1 / btn_SB2）
| プロパティ | 1号機 | 2号機 |
|---|---|---|
| Text | `"1号機"` | `"2号機"` |
| X | 16 | 130 |
| Y | 100 | 100 |
| Width | 106 | 106 |
| Height | 40 | 40 |
| Fill | `If(varSB1, ColorValue("#1A3A6E"), White)` | `If(varSB2, ColorValue("#1A3A6E"), White)` |
| Color | `If(varSB1, White, RGBA(0,0,0,0.7,1))` | 同左 |
| BorderColor | `ColorValue("#1A3A6E")` | 同左 |
| BorderThickness | 1 | 1 |
| RadiusTopLeft 他 | 6 | 6 |
| OnSelect | `Set(varSB1,!varSB1)` | `Set(varSB2,!varSB2)` |
#### lbl_SBResult（蒸気ボイラー 選択結果）
> lbl_ACResultと同パターン。Y=148, varSB1〜2 参照
#### 蒸気ボイラー・ドレン — TextInput 一覧
| コントロール名 | HintText | Y |
|---|---|---|
| txt_SteamBoilerPressure | `"蒸気ボイラー圧力（MPa）"` | 190 |
| txt_PWSupplyMeter | `"PW補給水メーター（L）"` | 246 |
| txt_BoilerFeedPressure | `"ボイラー給水ポンプ圧力（MPa）"` | 302 |
| txt_DrainConductivity | `"ドレン電導度（μS/cm）"` | 368 |
| txt_DrainPumpPressure | `"ドレンポンプ圧力（MPa）"` | 424 |
> 全て X=16, Width=`Parent.Width-32`, Height=44, KeyboardType=DecimalNumber
#### btn_Back4 / btn_Next4
> OnSelect: Back→scrInput3, Next→scrInput5
#### OnVisible（scrInput4）
```powerfx
Set(varSB1,false); Set(varSB2,false)
```
---
### scrInput5 — 入力⑤（号機別 運転時間・油面確認・備考）
#### ヘッダー・プログレスバー
| コントロール | 変更箇所 |
|---|---|
| lbl_HeaderTitle.Text | `"運転時間・油面確認"` |
| lbl_Step.Text | `"5 / 5"` |
| rect_Progress.Width | `Parent.Width` |
#### 号機カード × 3（1号機を例に記載）
##### lbl_Unit1Header（1号機カードヘッダー）
| プロパティ | 値 |
|---|---|
| Text | `"1号機"` |
| X | 16 |
| Y | 68 |
| Width | `Parent.Width - 32` |
| Height | 28 |
| Fill | `RGBA(26,58,110,0.12,1)` |
| Color | `ColorValue("#1A3A6E")` |
| FontWeight | `FontWeight.Bold` |
| Size | 12 |
| PaddingLeft | 10 |
| RadiusTopLeft / TopRight | 6 |
| RadiusBottomLeft / BottomRight | 0 |
##### txt_RunTime1 / dd_OilLevel1（横並び）
| プロパティ | 総運転時間 | 油面確認 |
|---|---|---|
| コントロール | TextInput | Dropdown |
| HintText / Items | `"総運転時間（hr）"` | `["OK","要補充","異常"]` |
| X | 16 | `Parent.Width/2 + 4` |
| Y | 100 | 100 |
| Width | `Parent.Width/2 - 20` | `Parent.Width/2 - 20` |
| Height | 44 | 44 |
| KeyboardType | `KeyboardType.DecimalNumber` | — |
| Format | `TextFormat.Number` | — |
| BorderColor | `RGBA(180,180,180,1)` | 同左 |
| BorderThickness | 1 | 1 |
| RadiusTopLeft 他 | 6 | 6 |
> 2号機カード（Y=160〜210）・3号機カード（Y=220〜270）を同パターンで作成
> 変数: txt_RunTime2, dd_OilLevel2 / txt_RunTime3, dd_OilLevel3
#### txt_Notes（特記事項・備考）
| プロパティ | 値 |
|---|---|
| HintText | `"異常・特記事項があれば入力"` |
| X | 16 |
| Y | 296 |
| Width | `Parent.Width - 32` |
| Height | 80 |
| Mode | `TextMode.MultiLine` |
| BorderColor | `RGBA(180,180,180,1)` |
| BorderThickness | 1 |
| RadiusTopLeft 他 | 6 |
#### btn_Back5 / btn_Next5
| プロパティ | 戻る | 確認画面へ |
|---|---|---|
| Text | `"← 戻る"` | `"確認画面へ →"` |
| OnSelect | `Navigate(scrInput4, ScreenTransition.None)` | `Navigate(scrConfirm, ScreenTransition.Fade)` |
> その他プロパティは scrInput2 の戻る/次へボタンと同じ
---
### scrConfirm — 確認・送信画面
#### ヘッダー
| プロパティ | 値 |
|---|---|
| lbl_HeaderTitle.Text | `"送信前確認"` |
| rect_Progress 非表示 | `Visible: false` |
#### 確認グループ（グループ × 5）
各グループは背景ラベル + 項目ラベル × 数の構成。
**グループ背景 rect_Group1〜5**
| プロパティ | 値 |
|---|---|
| Fill | `RGBA(245,247,250,1)` |
| BorderColor | `RGBA(200,210,225,1)` |
| BorderThickness | 1 |
| RadiusTopLeft 他 | 8 |
| Width | `Parent.Width - 32` |
| X | 16 |
**確認行ラベルパターン（キー）**
| プロパティ | 値 |
|---|---|
| Size | 11 |
| Color | `RGBA(100,100,100,1)` |
| Align | `Align.Left` |
**確認行ラベルパターン（値）**
| プロパティ | 値 |
|---|---|
| Size | 12 |
| Color | `RGBA(20,20,20,1)` |
| FontWeight | `FontWeight.Semibold` |
| Align | `Align.Right` |
**各グループのラベル値一覧**
```
グループ1: 基本情報
  点検日時: Text(dtpInspection.SelectedDate,"yyyy/mm/dd")
  運転時間: txt_StartTime.Text & " 〜 " & txt_EndTime.Text
グループ2: 灯油・ボイラー
  サブタンクレベル: varSubTank
  灯油メーター: txt_HWBoilerKerosene.Text & " L"
グループ3: 圧縮機・温水
  空気圧縮機: Substitute(Concat(Filter([{u:"1号機",s:varAC1},{u:"2号機",s:varAC2},{u:"3号機",s:varAC3}],s=true),u,";"),";",", ")
  ヘッダー圧力: txt_HeaderPressure.Text & " MPa"
  温水ポンプ: Substitute(Concat(Filter([{u:"1号機",s:varHW1},{u:"2号機",s:varHW2},{u:"3号機",s:varHW3}],s=true),u,";"),";",", ")
グループ4: 蒸気ボイラー
  蒸気ボイラー: Substitute(Concat(Filter([{u:"1号機",s:varSB1},{u:"2号機",s:varSB2}],s=true),u,";"),";",", ")
  蒸気ボイラー圧力: txt_SteamBoilerPressure.Text & " MPa"
グループ5: 運転時間・油面
  1号機: txt_RunTime1.Text & " hr / " & dd_OilLevel1.Selected.Value
  2号機: txt_RunTime2.Text & " hr / " & dd_OilLevel2.Selected.Value
  3号機: txt_RunTime3.Text & " hr / " & dd_OilLevel3.Selected.Value
```
#### btn_Edit（修正ボタン）
| プロパティ | 値 |
|---|---|
| Text | `"← 修正する"` |
| X | 16 |
| Y | `Parent.Height - 120` |
| Width | `Parent.Width - 32` |
| Height | 44 |
| Fill | White |
| Color | `RGBA(80,80,80,1)` |
| BorderColor | `RGBA(180,180,180,1)` |
| BorderThickness | 1 |
| RadiusTopLeft 他 | 6 |
| OnSelect | `Navigate(scrInput1, ScreenTransition.None)` |
#### btn_Submit（送信ボタン）
| プロパティ | 値 |
|---|---|
| Text | `"✓ SharePointに送信する"` |
| X | 16 |
| Y | `Parent.Height - 66` |
| Width | `Parent.Width - 32` |
| Height | 50 |
| Fill | `ColorValue("#1A7E4A")` |
| Color | White |
| RadiusTopLeft 他 | 8 |
| FontWeight | `FontWeight.Semibold` |
| Size | 16 |
**OnSelect 完全版:**
```powerfx
Set(varAirCompResult, Concat(Filter([{u:"1号機",s:varAC1},{u:"2号機",s:varAC2},{u:"3号機",s:varAC3}],s=true),u,";"));
Set(varHWPumpResult,  Concat(Filter([{u:"1号機",s:varHW1},{u:"2号機",s:varHW2},{u:"3号機",s:varHW3}],s=true),u,";"));
Set(varSteamBoilerResult, Concat(Filter([{u:"1号機",s:varSB1},{u:"2号機",s:varSB2}],s=true),u,";"));
If(
    IsBlank(varAirCompResult), Notify("空気圧縮機の運転号機を選択してください", NotificationType.Error),
    IsBlank(varHWPumpResult),  Notify("温水ポンプの運転号機を選択してください", NotificationType.Error),
    IsBlank(varSteamBoilerResult), Notify("蒸気ボイラーの運転号機を選択してください", NotificationType.Error),
    Patch(
        UtilityDailyReport,
        Defaults(UtilityDailyReport),
        {
            '点検日時': dtpInspection.SelectedDate,
            '灯油サブタンクレベル（液面）': {Value: varSubTank},
            '温水ボイラー灯油メーター': Value(txt_HWBoilerKerosene.Text),
            '運転開始時間': txt_StartTime.Text,
            '運転終了時間': txt_EndTime.Text,
            '空気圧縮機運転号機': ForAll(Split(varAirCompResult,";"),{Value:Result}),
            'ヘッダー圧力（Mpa)': Value(txt_HeaderPressure.Text),
            '温水ポンプ運転号機': ForAll(Split(varHWPumpResult,";"),{Value:Result}),
            '温水ポンプ圧力(Mpa)': Value(txt_HWPumpPressure.Text),
            '温水タンク容量(㎥）': Value(txt_HWTankVol.Text),
            '温水タンク内温度（℃）': Value(txt_HWTankTemp.Text),
            '中水タンク容量(㎥)': Value(txt_GreyWaterVol.Text),
            '飲料水タンク容量（㎥）': Value(txt_DrinkWaterVol.Text),
            'PWタンク容量(㎥）': Value(txt_PWTankVol.Text),
            '市水温度（℃）': Value(txt_CityWaterTemp.Text),
            '灯油タンク容量TK６０５（㎥）': Value(txt_TK605Vol.Text),
            '灯油メーター１（L)': Value(txt_KeroMeter1.Text),
            '薬品タンク１（L)': Value(txt_ChemTank1.Text),
            '灯油メーター2（L)': Value(txt_KeroMeter2.Text),
            '薬品タンク2（L)': Value(txt_ChemTank2.Text),
            '蒸気ボイラー運転号機': ForAll(Split(varSteamBoilerResult,";"),{Value:Result}),
            '蒸気ボイラー圧力（Mpa）': Value(txt_SteamBoilerPressure.Text),
            'PW補給水メーター（L)': Value(txt_PWSupplyMeter.Text),
            'ボイラー給水ポンプ圧力（Mpa）': Value(txt_BoilerFeedPressure.Text),
            'ドレン電導度': Value(txt_DrainConductivity.Text),
            'ドレンポンプ圧力': Value(txt_DrainPumpPressure.Text),
            '総運転時間１': Value(txt_RunTime1.Text),
            '油面確認１': dd_OilLevel1.Selected.Value,
            '総運転時間2': Value(txt_RunTime2.Text),
            '油面確認2': dd_OilLevel2.Selected.Value,
            '総運転時間3': Value(txt_RunTime3.Text),
            '油面確認3': dd_OilLevel3.Selected.Value
        }
    );
    Navigate(scrComplete, ScreenTransition.Fade)
)
```
---
### scrComplete — 送信完了画面
#### lbl_CompleteIcon
| プロパティ | 値 |
|---|---|
| Text | `"✅"` |
| X | `Parent.Width/2 - 32` |
| Y | 220 |
| Width | 64 |
| Height | 64 |
| Size | 48 |
| Align | `Align.Center` |
#### lbl_CompleteTitle
| プロパティ | 値 |
|---|---|
| Text | `"送信が完了しました"` |
| X | 16 |
| Y | 296 |
| Width | `Parent.Width - 32` |
| Height | 40 |
| Size | 18 |
| FontWeight | `FontWeight.Bold` |
| Align | `Align.Center` |
| Color | `RGBA(20,20,20,1)` |
#### lbl_CompleteSub
| プロパティ | 値 |
|---|---|
| Text | `Text(Today(),"yyyy年mm月dd日") & " 分の" & Char(10) & "ユーティリティ日報をSharePointに保存しました"` |
| X | 16 |
| Y | 344 |
| Width | `Parent.Width - 32` |
| Height | 60 |
| Size | 13 |
| Color | `RGBA(80,80,80,1)` |
| Align | `Align.Center` |
#### btn_BackHome
| プロパティ | 値 |
|---|---|
| Text | `"ホームへ戻る"` |
| X | 16 |
| Y | 430 |
| Width | `Parent.Width - 32` |
| Height | 50 |
| Fill | `ColorValue("#1A7E4A")` |
| Color | White |
| RadiusTopLeft 他 | 8 |
| FontWeight | `FontWeight.Semibold` |
| Size | 16 |
| OnSelect | `Navigate(scrHome, ScreenTransition.Fade)` |
---
### scrHistory — 履歴閲覧画面
#### ヘッダー
| プロパティ | 値 |
|---|---|
| lbl_HeaderTitle.Text | `"過去データ履歴"` |
| プログレスバー | 非表示（Visible: false） |
#### txt_Search（検索ボックス）
| プロパティ | 値 |
|---|---|
| HintText | `"日付で絞り込み（例: 2026/06）"` |
| X | 16 |
| Y | 60 |
| Width | `Parent.Width - 32` |
| Height | 40 |
| BorderColor | `RGBA(180,180,180,1)` |
| BorderThickness | 1 |
| RadiusTopLeft 他 | 20 |
| OnChange | `Set(varSearch, Self.Text)` |
#### gal_History（履歴Gallery）
| プロパティ | 値 |
|---|---|
| Items | `Sort(Filter(UtilityDailyReport, IsBlank(varSearch) \|\| Text('点検日時',"yyyy/mm") = varSearch), '点検日時', Descending)` |
| X | 0 |
| Y | 112 |
| Width | `Parent.Width` |
| Height | `Parent.Height - 112` |
| TemplateSize | 64 |
| TemplatePadding | 0 |
**Gallery 内コントロール:**
| コントロール | Text / プロパティ |
|---|---|
| lbl_HistDate | `Text(ThisItem.'点検日時', "yyyy/mm/dd（aaa）")` / Size:14, FontWeight:Bold |
| lbl_HistMeta | `ThisItem.'空気圧縮機運転号機' & " / 蒸気" & ThisItem.'蒸気ボイラー運転号機'` / Size:11, Color:Gray |
| lbl_Arrow | `"›"` / Size:20, Align:Right, Color:Gray |
| rect_Divider | Height:1, Fill:RGBA(220,220,220,1), Y:TemplateHeight-1 |
#### btn_BackFromHistory
| プロパティ | 値 |
|---|---|
| Text | `"← ホームへ"` |
| X | 16 |
| Y | 8 |
| Width | 80 |
| Height | 34 |
| Fill | Transparent |
| Color | White |
| Size | 13 |
| OnSelect | `Navigate(scrHome, ScreenTransition.Fade)` |
---
*最終更新: 2026-06-02 (rev.4 — 全9画面コントロール配置コード追加) | 担当: 保全技術課*

---

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
