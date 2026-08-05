# TaskLog 2026-07-23｜行政儀表板權限設定 [Claude@Mac]

> todo 真相源。主題：行政查核獨立 App 加權限——只有行政人員可檢視／編輯，且換人／代班免改設定。

## 需求
- 行政 App 目前無任何權限判定，任何開得到的人都能看與寫 field_8。
- 目標：只有行政能看能改；行政可能更換或臨時代班，希望用 team_member 名冊驅動，免每次改 App 分享。

## 現狀盤點（session 開場調查）
- ScrAdmin 為**獨立 Power App**，`scradmin07023_origin.yml` **完全無權限閘**（無 OnVisible、無 locIsAdmin）。
- 行政判定既有 pattern 只接在 ScrToday：`locIsAdmin = 讀 team_member.出勤存取權限`；行政 App 沒接。
- 行政名冊 2 人：黃文郁 `wenyuhuang@techartgroup.com`、陳怡惠 `ihueychen@techartgroup.com`（`出勤存取權限`＝「行政管理」）。
- `team_member` 存 **Excel（TaskDB.xlsx）非清單** → 無法做欄位級硬權限，但當唯讀名冊查權限可用。
- **SharePoint 清單硬邊界從未實作**；field_8 有 3 個寫入者（行政 App、1000 米米信 flow、1010 flow）→ 若鎖清單須白名單這兩支 flow 帳號。

## 完成
- **決定採「App 內 UI 權限閘（team_member 驅動）」**：分享面板無法讀 team_member，唯一能動態換人的是畫布層閘。代價＝UI 層非硬安全、App 需分享給全員由此閘過濾。
- **交付 `workingfiles/canvas/ScrAdmin_權限閘_paste.pa.yaml`**（全選貼）：
  - 末尾加 `conNoAccess` 全螢幕遮罩（`Visible = Not(locIsAdmin)`，文字「您沒有查閱權限」）。
  - `conHeaderA`／`conColHead`／`galPeople` 均加 `Visible = locIsAdmin`（雙保險，非行政內容不渲染，不只靠遮罩 z-order）。
  - `lblEmptyA.Visible` 加 `locIsAdmin And …`。
- **Screen.OnVisible（手動設，pa.yaml 帶不進畫面屬性）**：
  ```
  Set(gToday, Text(Today(), "yyyy/mm/dd")); UpdateContext({locIsAdmin: Trim(Coalesce(LookUp(team_member, Lower(Email) = Lower(User().Email), '出勤存取權限'), "")) = "行政管理"}); ClearCollect(colToday, Filter(AttendanceHistory, 日期 = gToday))
  ```
- **關鍵除錯**：初版用 `!IsBlank(LookUp(...))` 判定，實測**每個在名冊有列的人都被判成行政**（遮罩閃一下就被收掉）。根因＝Excel 空欄回傳空字串 `""`，`!IsBlank("")`＝true。改為值比對 `= "行政管理"`。使用者實測**成功**。
- 兩份操作指南（供對照，非必留）：`行政app權限_分享限行政_設定指南.html`、`行政app權限_team_member動態閘_套用指南.html`。

## 待辦
- [ ] **清單層硬鎖（使用者要求稍晚提醒）**：SharePoint 項目層權限＋寫入導向 server-time flow，白名單 1000／1010 flow 帳號，且不擋員工寫自己的 field_7。此層本次未做，屬已知安全缺口。
- [ ] 使用者確認 App 分享範圍已放寬（否則 team_member 新增行政仍開不了 App）。
- [ ] App Owner `真核域(開會用)` 若為多人共用帳號＝權限漏洞，建議換個人帳號。

## 未提交
- 新增：`ScrAdmin_權限閘_paste.pa.yaml`、兩份 `.html` 指南、本 TaskLog。待 merge-main。
