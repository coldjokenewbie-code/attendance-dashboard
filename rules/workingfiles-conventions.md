# workingfiles 命名規範
`workingfiles/` 存放暫時性工作檔案與素材，不納入正式輸出。
## 子資料夾
| 資料夾 | 用途 |
|--------|------|
| `_screenshots/` | AI 擷圖存放處，供視覺驗收使用 |
| `_scripts/` | AI 撰寫的本專案處理腳本 |
## 原則
- 內容為暫時性，驗收或任務完成後可清除
- `_scripts/` 腳本由 AI 產生，用途明確後可移至 `tools/` 或刪除
- `_screenshots/` 截圖驗收完成後可清除

## 版控白名單的專案例外（2026-08-05 PO 裁示）
Drive → `git_mirror` 同步的副檔名白名單，本專案**加收 `.zip`**——Power Automate 流程套件是可解壓的 JSON、每個約 3–6 KB，且 `definition.json` 是流程設定的真相源，值得進版控。

- **納入**：`workingfiles/automate_import/*.zip`（待匯入）、`workingfiles/automate_export/*.zip`（Portal 匯出）
- **排除**：`informaiton/archive/*.zip`（早期歷史備份，16 個共 116 KB，已寫進 mirror 的 `.gitignore`）
- rsync 指令要多帶一條 `--include='*.zip'`；xlsx、docx、pptx、圖片等大型二進位仍不複製
