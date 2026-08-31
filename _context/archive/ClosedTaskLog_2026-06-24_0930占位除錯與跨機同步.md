# TaskLog_2026-06-24_0930 占位除錯 + 檢核表 + 跨機同步
> [Claude@Mac]｜worktree `flow-0945`｜延續 0945 session，主軸轉到 0930 占位除錯與跨機佈署

## 背景
0945 名冊版寄信已上線（前一 session）。本 session 由「0945 延遲列為何只看到延遲人」追到上游 **0930 補占位流程實際沒寫入清單**，逐一定位並修正；另完成指南檢核表化、清單排序解法、跨機 worktree 同步、0955 檢核表 prompt。

## 完成項目
- **0945 延遲判斷釐清**：`延遲列` 用反向篩 `not(endsWith(Title,'_TaskOnSchedule'))`。占位缺席時此式退化為「全過」＝全部延遲列，**0945 仍正確、不需改**；占位只影響「行政頁簽列全體」。
- **0930 占位不寫入 — 解壓實際匯出 definition.json 坐實兩個病灶**（`/Downloads/0930_0623_…zip`）：
  1. **段B 條件左值填錯**：左值竟是「是」分支建立項目的 Title `concat(...,'_TaskOnSchedule')`（字串），與 `@false` 比 → `equals(字串,false)` 恆 false → 是分支永不執行 → 占位一張不寫（右值 `@false` 其實正確）。正解：左值改 `contains(variables('延遲Emails'), toLower(items('Apply_to_each')?['Email']))`。
  2. **附加至陣列變數值缺 `@`**：值為字面字串 `toLower(...)`（非 `@toLower(...)`）→ `延遲Emails` 不含真 email、`contains` 永不中，修①後延遲者反被重複補占位。正解：值改 fx 插入成 `@toLower(...)`。
- **0930 指南更新**：§5/§4-3 警告改精簡專業敘述；新增 **§8 排序**（快解 §8-1 檢視依 Title 遞減置頂；正解 §8-2 加真「日期及時間」欄 `日期D`＋流程補寫＋回填）。
- **浮動修改檢核表**（嵌進 0930 指南）：固定面板、localStorage 持久打勾、點項平滑跳修改區＋黃光閃、進度計數、可收合、重設鍵；5 項（含可選的日期欄）。檔已 `open` 給使用者。
- **清單排序問題解**：`日期` 欄單行文字＋`yyyy/M/d` 非零填補 → 文字排序錯位。快解＝改用 Title 排序（`yyyyMMdd` 前綴固定寬度零填補，文字序＝時間序），優於依「建立時間」（回填列建立時間擠在匯入當下）。
- **跨機佈署（→Windows）**：澄清 **worktree 資料夾不隨 git 同步、只有分支/commit 會**。使用者選「保留三分支、Windows 重建 worktree」。已 `commit + push` flow-0945 並 FF 到 origin/main（含 0930 指南＋檢核表＋INDEX 跨機待辦）。
- **INDEX 跨機待辦**：寫入 `_context/INDEX.md` 頂段（main 上），Windows clone 後 Claude 即讀到並協助 `git worktree add ../attendance-0945 flow-0945`／`...0955 flow-0955`。
- **0955 檢核表 prompt**：產出可直接貼給 0955 agent 的 prompt（以 0930 指南為範本複製三段元件、改 KEY `todo0955_v1`、依 0955 步驟自擬項目）。
- **ScrAdmin YAML 匯出說明**：方法 1 樹狀檢視選 Screen 節點 Ctrl+C 貼純文字即得 YAML（一檔一畫面、選整個 Screen 帶子控件）；方法 2 下載 .msapp 用 pac canvas 解包。

## 待 PO / 待辦
1. **0955 分支尚未上遠端**：origin 僅 `main`、`flow-0945`。需 0955 CLs 在 `attendance-0955` 執行 `commit → merge origin/main → push -u origin flow-0955`，Windows 才能重建 `attendance-0955`。
2. **0930 占位正解（§8-2 日期欄）**：使用者標記為可選；若要依日期區間篩選/分組再做（加 `日期D`＋流程兩個建立項目補寫＋回填）。
3. **使用者實作驗收**：0930 改①②後清測試列手動跑，驗 `延遲Emails` 為真 email、非延遲者各一占位、延遲者不重複。

## 關鍵檔
- 0930 指南（含檢核表）：`workingfiles/flows/0930_Planner改寫_寫入清單_建置指南.html`
- 0955 指南：`workingfiles/flows/Flow_0955_再巡檢與同步_建置規格.html`
- 實際匯出比對：`~/Downloads/0930_0623_20260623163923.zip`（解壓於 scratchpad）
- 清單 AttendanceHistory：field_1 員工編號／field_2 日期(文字)／field_3 Email／field_4 姓名…；Title＝`yyyyMMdd_email_…` 唯一鍵、可排序
- repo：`github.com/coldjokenewbie-code/attendance-dashboard`（main/flow-0945 = 99a3d00）
