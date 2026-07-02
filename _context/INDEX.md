# 出勤專案 — 現況總覽 (INDEX)
> 進場先讀。最後更新：2026-06-24

## ✅ 跨機接續：Windows 端 worktree 已重建（2026-06-24 [Claude@Win] 完成）
> 三資料夾並列佈局已在 DESKTOP-7SF21LR 重建完成，與 Mac 一致：
> - `E:\Git_work\attendance-dashboard` → `main`
> - `E:\Git_work\attendance-0945` → `flow-0945`
> - `E:\Git_work\attendance-0955` → `flow-0955`
>
> 跨機紀律：每次換機前先在當前機 `commit + push`；worktree 資料夾本身不會同步，只有已提交的 commit 會過去。

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
- **最新工作紀錄**：`_context/TaskLog_2026-06-24_儀表板表格化與淺色改版.md`（表格化＋自動存 bug 鏈＋雙頁淺色＋ai-team 審查）。前一份：`_context/TaskLog_2026-06-16_架構落實與晨間流程.md`。
- 欄位已從 `_context/AttendanceHistory.csv` 確認（九欄顯示名稱）。

## 待辦 / 下一步
- **架構落實（待動工，已寫計畫）**：`_context/Plan_2026-06-15_excel-list-bridge.html` — Excel 主檔＋小清單介面＋Power Automate 橋接，含模組二（請假紀錄查詢）。
- 上游 ③ Office Scripts 寬轉長自動化（目前人工暫代）；RowKey 需在此步驟產生（橋接前置）。
- 模組三（假期餘額）、角色權限、⑥ SharePoint 嵌入。
- 請 Claude 補上它的 ai-team CLI 協作版本：登入、短 prompt、卡住處理、避免直接讀寫檔案。
- 請 Antigravity 補上它的 ai-team CLI 協作版本：權限需求、寫檔限制、編碼風險、signal 寫入慣例。
- Claude / Antigravity 版本補齊後，推回 WTF repo，系統化成共用流程。
- 整理完成後分享給同事，讓 ai-team CLI 協作可被團隊複用。

## 備註
- `_context/AI_TEAM_DISCUSSION_2026-06-08_dashboard-rebuild.md` 是原始 ai-team 討論檔，包含 Codex、Antigravity、Claude 意見。
