# TaskLog_2026-06-22_0955再巡檢流程
> [Claude@Mac] ｜ 分支 `flow-0955`（worktree `/Users/coma/Git_work/attendance-0955`）｜ 與 `flow-0945`（另一個 Claude）並行，互不干擾
> ai-team：Claude 為 Tech Lead、與 Codex 五輪討論定案；不含 Antigravity

## 任務
做 Power Automate 排程流程「0955 再巡檢與同步」：員工 09:45 收信回覆／改 Planner 後，09:55 重掃 Planner，收斂清單 AttendanceHistory（端到端步驟 6）。外加：儀表板回覆 10:00 後唯讀。
**使用者硬性要求**：盡量複製 0930 流程元件、不要改太多、不動 0930 本體。

## 最終定案（使用者多次迭代後）
### 0955 流程：不刪任何列，只「覆寫 field_7」標註
- **做法**：`0930每日巡檢Planer進清單_v1`「另存複本」成 0955，掃描段一字不動；只刪寫入/占位段、加讀清單→標註段。0930 不碰。
- **比對**：用 `endsWith(Title,'_'+id)` 巢狀 Filter array（不解析 Title、不加欄、不動 0930）。
- **直接沿用 0930 既有的 `逾期_篩選陣列`** 建「仍逾期集合」（比上一版更徹底複用 0930）。
- **處置（不刪列，以 Planner 實況校正 field_7）**（2026-06-23 使用者修正：勾 1/4 只是「宣稱」，必須看 Planner 卡片實況）：
  - 原 1./4. 且「卡片已退出逾期」（真的做了）→ field_7＝`原勾選「<原回覆>」，同事已修改狀態`。
  - 原 1./4. 但「卡片仍逾期」（宣稱但沒真做，含勾1卻沒勾掉卡片、勾4卻沒改卡片）→ field_7＝`3. 延遲（…假卡）（原勾選「<原回覆>」，卡片仍延遲）`＝**歸到 3、要請假**。
  - 空白／未回覆 → field_7＝`3. 延遲（09:30沒到公司，請填寫假卡）`。
  - 2./3. 不動。
- **結構**：條件_原1或4 →（是）巢狀 條件_仍逾期（true＝歸3註記卡片仍延遲／false＝已修改狀態）；（否）條件_未回覆 → 回填3。布林先用 Compose 算、條件只比 `=true`（新版條件無進階模式）；累加用「附加至陣列變數」（Set+union 自我參照被擋）。
- **field_7 實際存完整標籤**（"1. 已完成（請自行勾掉卡片）"…），比對一律 `startsWith(trim(coalesce(f7,'')),'1.')`，不用 equals。
- **再跑防呆**：條件A 要 startsWith 1./4.、條件B 有 `not(startsWith '原勾選')`＋"3." 被擋 → 已標註列再跑不會二次覆寫（Codex 驗）。

### ScrToday：表單回覆 10:00 後唯讀（一般同事），行政可改
- Screen.OnVisible 多存 `locIsAdmin`；`radReply.DisplayMode` 與 `btnSubmit.DisplayMode`＝`If(locIsAdmin Or Hour(Now())<10, Edit, View)`；`btnSubmit.OnSelect` 內再加同條件自衛（DisplayMode 只擋畫面，Patch 須自衛 — Codex F）。
- **UI 鎖非硬安全**（lessons）：真硬保障需走受控流程/清單權限+伺服器時間，成本較高，列 PO 評估。
- 表單題目「請於今天『10點前』回覆」與此鎖一致（使用者貼圖佐證）。

## ai-team 討論軌跡（5 輪 Codex）
1. 認同 Save As 複製 0930、移除 for_each_task 子樹、關並行、開 pagination。
2. 提 endsWith 比對取代「加純任務id欄」→ Codex 認同，不動 0930、不加欄。
3. 簡化為「只刪真完成」單集合 → Codex「設計通過」（此版後被使用者改為不刪）。
4. 追加需求：空白回填3、10點後唯讀 → Codex 認同；指出 btnSubmit.OnSelect 要自衛、用 coalesce。
5. 翻轉成「不刪只標註、覆寫 field_7 註記原回覆」→ Codex「設計通過」，建議 startsWith 前 trim。
6. 使用者修正核心邏輯（勾1/4只是宣稱、卡片仍逾期要歸3＋註記卡片仍延遲）→ Codex「邏輯通過」：五情境全覆蓋、再跑防呆成立、歸3保留「3.」開頭供下游 startsWith 判假卡（唯一未來注意：若要分辨「原本選3」vs「翻轉成3」需解析括號註記，現階段可接受）。

## 產出（flow-0955 worktree）
- `workingfiles/flows/Flow_0955_再巡檢與同步_建置指南.html`（最終：複製0930、不刪只標註）。
- `workingfiles/canvas/變更_回覆10點後唯讀.html`（ScrToday 唯讀變更指南）。
- `workingfiles/canvas/ScrToday_paste.pa.yaml`（已改：radReply/btnSubmit DisplayMode＋OnSelect 自衛＋OnVisible locIsAdmin）。
- 移除 `workingfiles/flows/Flow_0955_再巡檢與同步_建置規格.html`（舊草稿，被指南取代）。
- `informaiton/0930每日巡檢Planer進清單_v1_20260621230005.zip`（複本到本 worktree，比對來源）。

## 0930 v1 實際結構（讀 definition.json 確認）
觸發 Recurrence(09:30) → 列出我擁有及我所屬的群組 → 測試用_篩選陣列 → 列出資料表中的資料列_team_member → 初始化變數(延遲Emails,array) → for_each_group_list_plans{ 列出群組的方案 → for_each_plan_list_tasks{ 列出工作 → 逾期_篩選陣列 → for_each_task{ For_each(逐_assignments){ 取得使用者設定檔_(V2) → 依Email篩員工編號 → 建立項目(寫清單) → 附加至陣列變數 }}}} → Apply_to_each(team_member){ 條件 → 建立項目_1(占位) }

## Power Automate 踩坑（本輪實建回報，宜 merge 後升級進 lessons-learned）
- **新版設計工具「條件(Condition)」沒有進階模式**（不能貼整段運算式）。解法：布林先用「撰寫(Compose)」算（Compose 輸入框可貼整段 fx），條件只比 `outputs(...) 等於 true`；混合 And/Or 也靠 Compose 拆成純 And 列。
- **Set variable 禁止自我參照**：`union(variables('X'), ...)` 寫回 X 會存檔失敗 `WorkflowRunActionInputsInvalidProperty / Self reference is not supported`（新版設計工具擋）。累加陣列改用「附加至陣列變數(AppendToArrayVariable)」逐筆附加單一 id（扁平、不報錯）——即 0930 既有 `附加至陣列變數` 同款；跨方案 task id 不重複免去重。

## 上線前必確認
- field_7「3.」回填字串以員工實際入口(儀表板 radio)為準。
- 更新項目帶原 Title（必填欄）；用整數 ID。
- 兩層 Foreach 關並行、取得清單項目開 pagination。
- Title 結尾穩定 `_<taskId>`（勿手改）。

## 狀態
- 0955 建置指南＋ScrToday 唯讀變更完成，Codex 五輪驗收通過。**未 commit**（ai-team：交 PO 驗收後才 commit）。
- 下一步：PO 看指南→Portal 照建/改→§9 測試→回饋→merge `flow-0955` 至 main。
