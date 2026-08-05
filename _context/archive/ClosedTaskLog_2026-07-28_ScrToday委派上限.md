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

## 症狀 4（2026-07-31 使用者手機截圖回報）：Gallery 卡片文字被裁
- 現象：卡片內【專案】【工作】長名稱被切一半，下一個欄位還疊上來（截圖可見「更新」只顯示上半、工作名第二行被切）。與症狀 3 同病、不同位置。
- 根因：卡片內六個 Label 全是固定 `Height`（`lblPlan` 28／`lblTask` 26／`lblReplyHeader` 22／`lblReplyVal` 30／`lblStatusHeader` 22／`lblStatusVal` 46），且每個的 `Y` 都是寫死座標（16／46／78／102／136／160／btnReply 214），文字一換行就超出自己的框、又被下一個控件蓋住。
- 修法（已寫入 `ScrToday_paste.pa.yaml`，**待 Studio 貼上驗證**）：
  - 六個 Label 全開 `AutoHeight: =true`，移除固定 `Height`
  - `Y` 改串接：每個引用上一個控件的 `Y + Height + 間距`（4／10／2／10／2），`btnReply.Y = lblStatusVal.Y + lblStatusVal.Height + 14`
  - `recCard.Height`：`268` → `=btnReply.Y + btnReply.Height + 12`（白卡隨內容縮）
  - `TemplateSize`：`280` → `420`
- **為何 TemplateSize 只能給固定值**：垂直 Gallery 的列高無法逐列依內容自動（平台限制，見 lessons 2026-06-22），只能取最壞情況。代價＝短內容卡片下方留白；因 `recCard` 已改成隨內容縮，留白落在卡片外的灰底，不是一張很空的白卡。
- **若嫌留白太多**的替代方向（未做，需使用者決定）：卡片上的專案／工作名改截斷顯示（如取前 N 字加「…」），完整內容只在 Popup 呈現，`TemplateSize` 就能收回 300 上下。

## 交接（2026-07-31 [Claude@Mac] session 結束，另開新 session 接手）

| 項目 | 狀態 |
|---|---|
| 症狀 1 委派退回（抓不到今日資料） | ✅ **已上線**（巢狀 Filter 已貼、使用者 07-31 回報修好；暫時解「資料列限制 2000」保留無害） |
| 症狀 2 同事手機打不開 | 已給處置建議（改用 Safari／Chrome），**未回報結果** |
| 症狀 3 Popup 文字被裁 | ✅ **已上線**（使用者 07-31 回報修好） |
| 症狀 4 Gallery 卡片文字被裁 | ✅ **已上線**；留白是否可接受**尚未回報** |
| 症狀 5 ScrAdmin 長文字跨列重疊 | ⏳ 修法**已寫入 `ScrAdmin_權限閘_paste.pa.yaml`、未貼上驗證** ← 新 session 從這裡接 |
| Planner 卡片整合 | 使用者裁定**下一階段**處理；前置事實與坑已寫入下方待辦 |

**新 session 進場第一件事**：把 `workingfiles/canvas/ScrAdmin_權限閘_paste.pa.yaml` 貼進 Studio 的 ScrAdmin 畫面。

**貼上前**：確認基準檔——本檔以 07-23 上線版為基準改，若其後在 Studio 手動改過，先匯出現況比對再貼，否則洗掉手改部分。ScrToday 就發生過 repo 版落後 Studio 的情況。

**貼上後逐條驗收**：
1. `Screen.OnVisible` 補回（`Set(gToday, ...)`＋`locIsAdmin` 判定式＋`ClearCollect(colToday, ...)`，全文見本檔檔頭）——pa.yaml 帶不進畫面屬性，全選刪除會連帶掉。
2. 長內容（如 MI-005 紅字、MI-007 請假字串）單行截斷、不再壓到相鄰列。
3. 滑鼠停留在被截斷的儲存格上看得到全文。
4. 權限閘（非行政看不到內容）與下拉自動存仍正常。

**（未驗證）** `Wrap: =false` 與 `Tooltip` 能否由 pa.yaml 貼入 `Label@2.5.1`。若 Studio 報屬性無效，先把該兩行拿掉再貼，另尋壓住溢出的方式（例如把 `Text` 外包一層 `Left(..., N) & "…"` 手動截斷）。

**ScrToday 側殘留**：`AutoHeight` 已實證可用（症狀 3／4 已上線），先前記的備援值（`lblPopMeta.Height` 110／`radReply.Y` 174）不再需要。唯一待回報＝`TemplateSize: =420` 造成的短內容留白是否可接受；嫌多就改截斷顯示、`TemplateSize` 收回 300 上下（見症狀 4 末段）。

## 症狀 5（2026-07-31）：行政頁 ScrAdmin 長文字跨列重疊
- 現象：「今日延遲工作｜同事回覆」欄與「出勤狀態（已存）」欄的長內容壓到相鄰列上（截圖 MI-005 紅字、MI-007 請假字串）。
- 根因：列高寫死 `TemplateSize: =(Parent.Height - 116) / 25`（一頁塞 25 列），Label 的 `Height` 等於列高；`lblReply` 用 `Char(10)` 把多筆延遲工作串成多行，超出的行沒被裁在框內，直接畫到隔壁列。
- **取捨已由使用者定案（2026-07-31）：維持一頁 25 列全覽，長內容單行截斷，滑鼠停留看全文。** 否決的兩案：加大列高自動換行（一頁看不完要捲）、點列展開浮層（工比較大，可留作日後）。
- 修法（已寫入 `ScrAdmin_權限閘_paste.pa.yaml`，**待 Studio 貼上驗證**）：
  - `lblReply`：`Concat` 分隔符 `Char(10)` → 全形「　／　」；`Wrap: =false`；新增 `Tooltip`（保留 `Char(10)` 多行版）
  - `lblStatusView`：`Wrap: =false`；新增 `Tooltip`
- **與 ScrToday 的修法方向相反、是刻意的**：ScrToday 是手機、單欄、可捲，所以放開 `AutoHeight` 讓它長高；ScrAdmin 是桌機表格、要一頁全覽，所以反過來鎖成單行。同一個「文字被切」的表徵，兩頁的正解不同。
- 代價：行政頁只能桌機用（手機沒有滑鼠停留）。此頁本來就是桌機作業。
- ⚠ **基準檔待確認**：以 `ScrAdmin_權限閘_paste.pa.yaml`（07-23 上線版）為基準修改，截圖上的元素與該檔一致。若 07-23 後有在 Studio 手動改過，貼上前先匯出現況比對，否則會把手改的部分洗掉。

## 待辦
- [x] ~~ScrToday：貼上 yaml 並驗收~~ **2026-07-31 使用者回報已修好**（症狀 1／3／4 的修法皆已上線）。
- [ ] **（最優先）** ScrAdmin：貼上 `ScrAdmin_權限閘_paste.pa.yaml` 驗收——長內容單行不再壓到隔壁列、滑鼠停留看得到全文、權限閘與自動存仍正常。貼上前先確認基準檔（見症狀 5 末段）。
- [ ] **下一階段（使用者 2026-07-31 提出）：儀表板帶入 Planner 卡片，讓同事就地改卡片狀態**。已確認清單 `Title` 第三段就是 Planner task id（樣本 `vXu-m6GEu0amsGMCTVsn5skAK6t_`，28 字元 base64url ＝ Graph `plannerTask`，非 Project for the web 的 GUID）。**注意結尾底線是 id 本身的字元，用 `Split` 取最後一段會拿到空字串**；要取就用 `Mid(Title, Len(Email) + 11)`，但更建議讓 09:30 flow 直接多寫一欄 `PlannerTaskId`，別在 App 端解析字串。尚待確認：同事是否都是該 Planner 計畫的成員（非成員會拿到權限錯誤）。做法取捨＝App 直接接 Planner 連接器（修改者顯示為本人，但連接器不可委派、每人首次要授權） vs 呼叫 flow 代改（異動全掛同一帳號、稽核失真）。**先做最小驗證**：加連接器＋一顆測試按鈕寫死 task id 設完成度 100，確認能改、修改者是本人、非 owner 的同事不被擋。
- [ ] 同事改用 Safari／Chrome 後回報結果；仍失敗則查 App 分享範圍。
- [ ] **全檔掃同型 bug**：其餘畫面（ScrAdmin 等）的 Filter／LookUp／UpdateIf 逐條檢查委派警告，別只修爆掉的這一條。
- [ ] **清單索引**：`AttendanceHistory` 破 5000 列後，未編索引欄的篩選即使可委派也會失敗。建議在清單設定給 `Email`、`日期` 兩欄加索引。
- [ ] 承 07-23：清單層硬鎖、App Owner 共用帳號漏洞，均未動。

## 檔案異動
- 新增：本 TaskLog。
- 修改：`workingfiles/canvas/ScrToday_paste.pa.yaml`（巢狀 Filter＋Popup 三處＋檔頭註解），2026-07-31 已同步 mirror。
