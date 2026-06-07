# Lessons Learned

## 2026-06-08
- `claude -p` 在使用者 Terminal 可用，不代表 Codex 沙盒內可用；若回 `Not logged in`，需用核准後的沙盒外執行測試。
- 讓外部 agent 直接讀寫檔案可能卡住或產生編碼問題；較穩定做法是請 agent 只輸出文字意見，再由 Tech Lead 寫入專案檔。
- ai-team 討論稿若屬過程文件，應放 `workingfiles/`，正式成果才放 `outputs/`。
