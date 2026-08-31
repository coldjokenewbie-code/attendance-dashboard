# TaskLog_2026-06-22_0945名冊版排程寄信
> [Claude@Mac]｜ai-team：Claude=Tech Lead、Codex=peer（3 輪審）、不用 Antigravity｜worktree `flow-0945`

## 背景
0945 排程寄信「未完成」收尾：舊版用「Select 去重」反覆卡（清單內部名 field_3，Select 寫成 `item()?['Email']` 映成 null）。定案改 **team_member(Excel) 名冊版**取代 Select。

## 本次完成（核心已上線，使用者回報「成功」）
- **worktree 隔離**：兩 CLI 各自 worktree＋分支——本 session `attendance-0945`/`flow-0945` 做 0945；另一 CLI `attendance-0955`/`flow-0955` 做 0955，git index／working tree 完全隔離。
- **讀通實際匯出 definition.json**（`informaiton/0945_未完成_..._v1.zip`），指南改以**真實 action 名稱**寫，不再靠猜：
  - 坐實 bug：`選取` = `select{Email: item()?['Email']}`（應 field_3）→ null；`篩選_延遲任務數量` where 寫死 `ihueychen@…`（除錯殘留）。
  - 取得真實儀表板 URL（已內嵌 Body，沿用）；清單 GUID `9bd1cdf3…`；原始版有 Excel Online Business 連線讀 TaskDB.xlsx（可重用改指 team_member）。
- **Codex 三輪審查**收斂：① 名冊版方向對、比修 Select 穩；② 比對式加 `string(coalesce(...,''))` 防 null；③ Wait 放 Condition True 分支內；④ 主旨姓名用名冊『姓名』非清單 field_4；⑤ Condition True 分支 runAfter 要重接（傳送 runAfter `{}`、延遲接傳送）；⑥ foreach 用 `@body('列出表格中的列_名冊')?['value']`；⑦ Excel 開分頁、header 精確。
- **Tech Lead 否決 Codex 一點**：Codex 建議「To 改個人 Email」——**錯**。Planner2Line 監聽 domain.e、靠**主旨姓名**對 contacts.json 路由 LINE，**To 必須維持 `domain.e`**。用 Planner2Line 架構知識否決。
- **產出指南**：`workingfiles/flows/Flow_0945_延遲通知信_改讀清單_建置指南.html`（名冊版就地編輯六步 A–F、§5 實跑檢查、附錄缺漏告警）。
- **即時排障（使用者實作中）**：
  - `篩選_延遲任務數量` 卡 → 病灶是改造有**順序依賴**：必須先加 Excel 名冊＋foreach 改名冊 value，`items(...)?['Email']` 才有物件可取；單改 where 必爆。
  - `條件 要寄嗎` 無「進階模式」（新版 designer 只有「篩選陣列」有）→ 改：布林運算式放**左值(fx)**、運算子**等於**、右值**用 fx 打 `true`**（直接打字 true 變字串致永遠 false）。✅ 使用者套用後成功。
- team_member 欄名已對 CSV：`員工編號/姓名/Email/Line 顯示名稱/出勤存取權限`（無尾空白）。

## 待 PO 裁示
1. **名冊缺漏告警**（名冊漏人→延遲者漏寄且無感知，regression 中偏高）：v1 內建，還是先上核心、之後再開？設計見指南附錄（純篩選＋附加陣列＋join，無 Select 坑；告警主旨用 `0945排程告警_名冊缺漏` 避開 Planner2Line `工作延遲_` 前綴）。
2. Excel team_member 表 header 是否精確 `姓名`/`Email`（存檔前用測試輸出確認 key）。

## 關鍵檔
- 指南：`workingfiles/flows/Flow_0945_延遲通知信_改讀清單_建置指南.html`
- 實際匯出：`informaiton/0945_未完成_清單每日專案延誤通知EMAIL_v1_20260621230243.zip`（解壓比對用）
- 名冊：team_member（TaskDB.xlsx 內）；清單 AttendanceHistory（GUID 9bd1cdf3…）
- 發 Line：`/Users/coma/Git_work/Planner2Line`
