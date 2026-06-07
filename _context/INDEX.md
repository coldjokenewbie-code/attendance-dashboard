# 出勤專案 — 現況總覽 (INDEX)
> 進場先讀。最後更新：2026-06-08

## 一句話目標
MS365 Power Apps 員工出勤儀表板；先重作模組一 Popup MVP，讓員工查看今日出勤查核並回寫 `表單回覆`。

## 目前狀態 / 進度
- 已讀取交接檔 `_context/attendance-dashboard-handover.md`。
- 三方討論後決議：重作 Canvas App，不修舊 App。
- 最新工作紀錄：`_context/TaskLog_2026-06-08_attendance-dashboard-rebuild.md`。
- 工作討論稿：`workingfiles/attendance-dashboard-rebuild-recommendation.html`。
- ai-team CLI 協作流程參考：`_context/ai-team-agent-cli-reference.html`。

## 待辦 / 下一步
- 進 Power Apps Studio，依工作討論稿建立模組一 MVP。
- 先確認 SharePoint List `AttendanceHistory` 欄位、內部欄位名稱、員工權限、`DateString` 格式。
- 請 Claude 補上它的 ai-team CLI 協作版本：登入、短 prompt、卡住處理、避免直接讀寫檔案。
- 請 Antigravity 補上它的 ai-team CLI 協作版本：權限需求、寫檔限制、編碼風險、signal 寫入慣例。
- Claude / Antigravity 版本補齊後，推回 WTF repo，系統化成共用流程。
- 整理完成後分享給同事，讓 ai-team CLI 協作可被團隊複用。

## 備註
- `_context/AI_TEAM_DISCUSSION_2026-06-08_dashboard-rebuild.md` 是原始 ai-team 討論檔，包含 Codex、Antigravity、Claude 意見。
