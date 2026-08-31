# TaskLog_2026-06-08_attendance-dashboard-rebuild
> [Codex@Mac] 2026-06-08

## Session Summary
- 載入專案交接檔 `_context/attendance-dashboard-handover.md`，確認本案是 MS365 Power Apps 員工出勤儀表板。
- 判斷舊 App 卡在模組一 Popup，且使用者已不清楚原製作步驟；三方討論後決議重作 Canvas App，沿用 SharePoint List 資料模型。
- 建立 ai-team 討論檔 `_context/AI_TEAM_DISCUSSION_2026-06-08_dashboard-rebuild.md`。
- 透過 `agy --print` 取得 Antigravity 意見；透過 `claude -p` 取得 Claude 意見，並彙整進討論檔。
- 產出 HTML 工作討論稿 `workingfiles/attendance-dashboard-rebuild-recommendation.html`。

## Decisions
- 重作，不修舊 App。
- 第一階段只做模組一 MVP：今日出勤列表、Popup 回覆、Patch 寫回 `AttendanceHistory.表單回覆`。
- 主方案：Screen 根層級 `OverlayContainer`，內放 `PopupCard`，用容器屬性置中。
- 備案：Screen 根層級 `Rectangle` 遮罩 + `Vertical Container`，用 `App.Width` / `App.Height` 計算置中。

## Open Issues
- 尚未進 Power Apps GUI 實作。
- 尚未確認 SharePoint List 實際內部欄位名稱。
- 尚未確認一般員工對 `AttendanceHistory` 是否有足夠權限更新 `表單回覆`。
- Office Scripts 自動化尚未完成；目前 `DateString` 若人工填錯，Gallery 可能撈不到資料。

## Next Steps
1. 依 `workingfiles/attendance-dashboard-rebuild-recommendation.html` 進 Power Apps Studio 建新 Canvas App。
2. 先檢查 `AttendanceHistory` 欄位與範例資料，尤其 `DateString` 格式為 `yyyy/m/d`。
3. 完成模組一 MVP 後，再開始請假紀錄、假期餘額、角色權限、SharePoint 嵌入。

## Files
- `_context/attendance-dashboard-handover.md`
- `_context/AI_TEAM_DISCUSSION_2026-06-08_dashboard-rebuild.md`
- `AGENT_SIGNAL.log`
- `workingfiles/attendance-dashboard-rebuild-recommendation.html`
