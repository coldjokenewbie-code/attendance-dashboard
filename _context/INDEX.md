# 出勤專案 — 現況總覽 (INDEX)

> ⚠️ **版控架構**：本資料夾（Google Drive）不進 git 追蹤，不要在此嘗試 `git init`／`add`／`commit`／`push`。真正版控在本機獨立資料夾 `/Users/coma/git_mirror/attendance-dashboard/`（**注意資料夾名跟本專案名不同，已於 2026-07-15 統一改用 GitHub repo 名 `attendance-dashboard`，不再用 `出勤專案`**；GitHub: `https://github.com/coldjokenewbie-code/attendance-dashboard.git`，branch `main`，完整 clone；Windows 對應路徑 `E:\git_mirror\attendance-dashboard\`，不隨 Drive 同步過去，各機各自 clone）。要 commit／push，先把 code/文字檔（依副檔名白名單：html/css/js/json/md/ts/tsx/jsx/mjs/py/txt/yaml/yml/sh）鏡像複製到 mirror 資料夾再操作；大型文檔（docx/pptx/pdf/圖片/影片）不鏡像，留 Drive 原地。機制詳見 WTF repo `wtf-config/GLOBAL.md`「Claude_cowork 專案的版控架構」段。

> 進場先讀。最後更新：2026-07-15（本日：Drive 舊快照與 mirror main 合併完成，本檔內容以 mirror main 版為準更新）

## ✅ 跨機接續：Windows 端 worktree 已重建（2026-06-24 [Claude@Win] 完成）
> 三資料夾並列佈局已在 DESKTOP-7SF21LR 重建完成，與 Mac 一致：
> - `E:\Git_work\attendance-dashboard` → `main`
> - `E:\Git_work\attendance-0945` → `flow-0945`
> - `E:\Git_work\attendance-0955` → `flow-0955`
>
> 跨機紀律：每次換機前先在當前機 `commit + push`；worktree 資料夾本身不會同步，只有已提交的 commit 會過去。
> （2026-07-15 註：Mac 端 main 已遷至 `git_mirror/attendance-dashboard`；Windows 端仍為上列 Git_work 佈局、待搬。）

## 一句話目標
MS365 Power Apps 員工出勤儀表板；先重作模組一 Popup MVP，讓員工查看今日出勤查核並回寫 `表單回覆`。

## 系統資料流（端到端）
```
① Power Automate 每日 09:30：查 Planner 延遲工作 → 通知行政 → Line 通知員工   ［自動］
② 行政人工核對 → 填寬格式 Excel（橫向：每人一列／每日一欄）                  ［人工，本質工作］
③ Office Scripts 寬轉長 → 寫入 SharePoint List AttendanceHistory            ［⚠ 自動化未完成，行政人工暫代］
④ SharePoint List：AttendanceHistory（單一真相源，每任務一列）
⑤ Power Apps Canvas App（模組一：今日查核 Gallery + Popup + Patch 回寫 表單回覆）
⑥ 嵌入 SharePoint 頁面，員工瀏覽器自助查看／填寫                            ［未做］
```
- ①②③＝上游（產資料塞進 List）；④中樞；⑤⑥下游（員工自助介面）。
- ①「09:30 / Planner / Line」為 2026-06-14 使用者口述補錄，先前文件未記；其餘與 handover 第二節一致。

## 目前狀態 / 進度
- 三方討論後決議：重作 Canvas App，不修舊 App。
- **✅ 模組一 MVP 完成**（2026-06-15 使用者確認）：響應式（手機優先）＋回覆改 4 選項單選，Gallery／Popup／回寫 `表單回覆` 實機驗證通過。已結案歸檔。
  - `workingfiles/canvas/ScrToday_paste.pa.yaml`（畫面原始碼）
  - `workingfiles/canvas/匯入指南.html`、`變更_回覆改單選.html`
  - 結案紀錄：`_context/archive/ClosedTaskLog_2026-06-13_module1-canvas-yaml.md`
- **模組一介面更新（2026-06-24 [Claude@Win]）**：行政頁重作成表格版＋自動存（`workingfiles/canvas/ScrAdmin_table.pa.yaml`）；ScrAdmin／ScrToday 統一簡約淺色風格。**待 PO 實機驗收**。
- **最新工作紀錄**：`_context/TaskLog_2026-07-06_個人行政拆分與請假整合.md`（個人/行政拆分＝行政獨立新 app、回覆 5 選項、請假整合流程指南 `Flow_請假整合_建置指南.html`；⚠ 待辦：**1000 有誤，待使用者修**；各流程待 Portal 套用）。前一份：`_context/TaskLog_2026-06-16_架構落實與晨間流程.md`（06-30~07-02 追記：0955 修判定＋發卡、1005/1030 米米信提醒指南、1000 視窗修單日、field_8 粒度待決策；07-03：ScrToday/ScrAdmin 委派警告 bug 修復，5 處公式改用 gToday 變數；**07-05 新增：日期欄格式改零填補 yyyy/MM/dd（0930 流程＋App gToday 同步）＋既有列回填指南**，以上皆**待使用者 Studio／Portal 套用**）。另一份：`_context/TaskLog_2026-06-24_儀表板表格化與淺色改版.md`（表格化＋自動存 bug 鏈＋雙頁淺色，待 PO 實機驗收）。
- 欄位已從 `_context/AttendanceHistory.csv` 確認（九欄顯示名稱）。

## 待辦 / 下一步
- **架構落實（待動工，已寫計畫）**：`_context/Plan_2026-06-15_excel-list-bridge.html` — Excel 主檔＋小清單介面＋Power Automate 橋接，含模組二（請假紀錄查詢）。
- 上游 ③ Office Scripts 寬轉長自動化（目前人工暫代）；RowKey 需在此步驟產生（橋接前置）。
- 模組三（假期餘額）、角色權限、⑥ SharePoint 嵌入。
- 請 Claude 補上它的 ai-team CLI 協作版本：登入、短 prompt、卡住處理、避免直接讀寫檔案。
- [x] Antigravity 已補上其 ai-team CLI 協作規格至參考文件中（`_context/ai-team-agent-cli-reference.html` 角色分段版，2026-07-15 已併入 mirror main）。
- Claude / Antigravity 版本補齊後，推回 WTF repo，系統化成共用流程。
- 整理完成後分享給同事，讓 ai-team CLI 協作可被團隊複用。

## AttendanceHistory 清單欄位對照（內部名 field_N ↔ 顯示名）
> 由「取得項目」原始輸出證實。Power Automate 運算式一律用內部名（顯示名回 null）。

| 內部名 | 顯示名 |
|---|---|
| Title | ID（yyyyMMdd_Email，唯一鍵） |
| field_1 | 員工編號 |
| field_2 | 日期 |
| field_3 | Email |
| field_4 | 姓名 |
| field_5 | 方案名稱 |
| field_6 | 任務名稱 |
| field_7 | 表單回覆（延遲回覆；0955 再巡檢寫回） |
| field_8 | 實際出勤狀態（米米信；1000/1010 寫入） |
| field_9 | DateString（樣本為空，不可靠；「今日」用 Title 前綴 yyyyMMdd_ 篩） |

## 備註
- `_context/AI_TEAM_DISCUSSION_2026-06-08_dashboard-rebuild.md` 是原始 ai-team 討論檔，包含 Codex、Antigravity、Claude 意見。
