# TaskLog 2026-07-28｜ScrToday 抓不到今日資料（委派退回＋500 列上限）[Claude@Mac]

> todo 真相源。主題：個人查核頁 ScrToday 顯示「今天沒有需要回覆的延遲工作」但實際有延遲工作；併同事手機開不了儀表板。

## 症狀（兩件事，根因無關）
1. **使用者本人**：ScrToday 開得起來、畫面正常，但顯示「今天沒有需要回覆的延遲工作」，實際有延遲工作。已成功運作兩週以上才首次出現。
2. **同事**：手機完全打不開，畫面為「無法開啟頁面。請確認網路連線狀態後，再試一次。」

## 症狀 1：根因與處置

### 根因（已確診）
- `galToday.Items` 公式：
  ```
  Filter(AttendanceHistory, Email = User().Email, 日期 = gToday, 任務名稱 <> Blank())
  ```
- Studio 委派警告點名 **`任務名稱`**（非 `Email`，非 `gToday`）→ `任務名稱 <> Blank()` 不可委派，整條 Filter 退回本機。
- 本機模式只從清單取前 N 列（依 ID 遞增＝最舊），N＝App「非委派查詢的資料列限制」，預設 500。
- `AttendanceHistory` 已累積 **550 筆** → 最新一天的列落在 500 之外，永遠抓不到。**破線前完全無症狀**，故「跑兩週才突然壞」。

### 確診方法（決定性驗證，非推論）
App 設定→一般→資料列限制 500 改 **2000** → 重開 App → 資料立即出現。使用者實測通過。

### 處置狀態
- ✅ **暫時解已上線**：資料列限制設 2000，目前正常運作。依累積速度約可再撐至 2000 筆（推測約兩個多月）。
- ⏳ **治本未做**：已設 Google 行事曆 2026-07-28 17:00 提醒（含完整修法）。

### 治本修法（待套用）
`galToday.Items` 改巢狀 Filter——內層只留可委派條件，外層再篩不可委派的：
```
=Filter(
    Filter(AttendanceHistory, Email = User().Email, 日期 = gToday),
    任務名稱 <> Blank()
)
```
內層委派後只回當日該員數筆，外層本機處理數筆，永不觸及上限。
`Email` 與 `gToday` **不動**（Studio 未對其警告；`User().Email` 被 Power Fx 當常數先求值，可委派）。
驗收：Items 委派警告三角消失＋手機實開看到今日延遲工作。

## 症狀 2：同事開不了
- 截圖特徵（頂部綠色進度條、右上 X、底部導覽列）＝**LINE 內建瀏覽器**；使用者確認同事是從 LINE 訊息直接點連結。
- （推測，待同事回報）Power Apps player 需 Microsoft 帳號多次重導向，LINE WebView 常擋掉，表徵即「無法開啟頁面」。與資料層、與症狀 1 無關。
- 處置：請同事在 LINE 內點右下「⋯」→用 Safari／Chrome 開啟。**若換瀏覽器仍打不開**，才轉查 App 分享權限（07-23 TaskLog 待辦「確認 App 分享範圍已放寬」尚未勾銷）。

## 症狀 3（晚間追加，同事回報）：Popup 任務名稱被裁掉
- 現象：點「填寫回覆」後，Popup 內「【專案】／【工作】」兩行文字被切掉尾巴，長任務名尤其明顯。
- 根因：`lblPopMeta` 固定 `Height: =72`，文字換行超過兩行即被裁；且 `radReply.Y` 寫死 `132`，把溢出的行蓋住。
- 修法（已寫入 `ScrToday_paste.pa.yaml`，**待 Studio 貼上驗證**）：
  - `lblPopMeta`：移除固定 Height，改 `AutoHeight: =true`，`Size` 14→13
  - `radReply.Y`：`=lblPopMeta.Y + lblPopMeta.Height + 12`（跟隨實際高度）
  - `radReply.Height`：`=Parent.Height - Self.Y - 72`
  - `conPopup.Height`：`Min(Parent.Height - 64, 580)` → `Min(Parent.Height - 40, 680)`
- **備援**（若 `AutoHeight` 貼不進或 Height 引用不生效）：`lblPopMeta.Height` 固定改 `110`、`radReply.Y` 固定改 `174`，可涵蓋約 4 行。
- ⚠ **Gallery 卡片同病未修**：`lblPlan`／`lblTask` 亦為固定高度（26～28px）、`TemplateSize` 固定 280，長專案名在卡片上一樣被切。依「UI 增量修改」原則本次未動，待 Popup 驗證通過後再處理。

## 交接（2026-07-31 [Claude@Mac] session 結束，另開新 session 接手）

**新 session 進場先做這件事**：把 `workingfiles/canvas/ScrToday_paste.pa.yaml` 貼進 Studio 的 ScrToday 畫面（全選現有控件→刪除→貼上），該檔已含下列兩處改動，一次貼上到位。

| 項目 | 狀態 |
|---|---|
| 症狀 1 委派退回（抓不到今日資料） | 暫時解已上線（資料列限制 2000，正常運作）；治本巢狀 Filter **已寫入 yaml、未貼上驗證** |
| 症狀 2 同事手機打不開 | 已給處置建議（改用 Safari／Chrome），**未回報結果** |
| 症狀 3 Popup 文字被裁 | 修法**已寫入 yaml、未貼上驗證** |
| Gallery 卡片同病 | **完全未動**（依 UI 增量原則，等 Popup 驗證通過再處理） |

**貼上後逐條驗收**：
1. `Screen.OnVisible` 仍在（`UpdateContext({locShowPopup: false}); Set(gToday, Text(Today(), "yyyy/mm/dd"))`）——全選刪除會連帶掉，貼完必補。
2. `galToday.Items` 委派警告三角消失。
3. 開一筆長任務名的卡片，Popup 內「【專案】／【工作】」兩行文字完整不裁。
4. 手機實開看得到今日延遲工作。

**兩點未驗證，失敗就走備援**（見「症狀 3」段）：`AutoHeight` 屬性能否由 YAML 貼入 `Label@2.5.1`；`AutoHeight=true` 時 `lblPopMeta.Height` 被別的控件引用是否回傳計算後高度。任一不成立→`lblPopMeta.Height` 固定 `110`、`radReply.Y` 固定 `174`。

## 待辦
- [ ] **（最優先）** 貼上 yaml 並跑完上述四條驗收（原訂 07-28 17:00，未執行，順延）。
- [ ] 同事改用 Safari／Chrome 後回報結果；仍失敗則查 App 分享範圍。
- [ ] **全檔掃同型 bug**：其餘畫面（ScrAdmin 等）的 Filter／LookUp／UpdateIf 逐條檢查委派警告，別只修爆掉的這一條。
- [ ] **清單索引**：`AttendanceHistory` 破 5000 列後，未編索引欄的篩選即使可委派也會失敗。建議在清單設定給 `Email`、`日期` 兩欄加索引。
- [ ] 承 07-23：清單層硬鎖、App Owner 共用帳號漏洞，均未動。

## 檔案異動
- 新增：本 TaskLog。
- 修改：`workingfiles/canvas/ScrToday_paste.pa.yaml`（巢狀 Filter＋Popup 三處＋檔頭註解），2026-07-31 已同步 mirror。
