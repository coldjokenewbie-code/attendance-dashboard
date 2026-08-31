# Plan_2026-06-13_module1-canvas-yaml
> [Claude@Mac] 2026-06-13 04:31 開工。使用者已核准「我寫、你匯入」路線 A。

## 目標
模組一 MVP：員工開 App 看到「今日、自己」的出勤查核列表，點列開 Popup，填寫後 Patch 回寫 `AttendanceHistory.表單回覆`。

## 交付方式（路線 A 調整版）
本機無 `pac`/`dotnet`，不從零打包 `.msapp`（無 Studio 基準檔風險高）。
改交付 **pa.yaml 原始碼**：使用者在 Power Apps Studio 建空白 App＋加 `AttendanceHistory` SharePoint 資料來源，貼入控件 YAML（官方 paste code 支援）。

## 依據
- 欄位（CSV 標頭，Power Fx 用顯示名稱）：員工編號、日期、Email、姓名、方案名稱、任務名稱、表單回覆、實際出勤狀態、DateString
- `DateString` 格式 `yyyy/m/d`（無前導零）→ `Text(Today(), "yyyy/m/d")`
- 同人同日多列（每任務一列）；Popup 主方案＝Screen 根層 Overlay 容器置中（TaskLog 2026-06-08 決議）

## 預計產出
1. `workingfiles/canvas/ScrToday.pa.yaml` — 完整畫面原始碼（含 Gallery、Overlay Popup、Patch）
2. `workingfiles/canvas/匯入指南.html` — 建 App、加資料來源、貼 YAML、驗證步驟
3. YAML 經本機 parser 驗證語法

## 已知風險
- Studio paste 對 YAML schema 驗證嚴格；控件版本號省略以用最新版。貼入若報錯，依錯誤訊息迭代。
- `DateString` 多為空值 → 今日列表可能空。畫面加「顯示我的全部紀錄」切換做 fallback。
- 一般員工對 list 的編輯權限未驗證（Patch 會失敗時 Notify 錯誤）。

## Checklist (task)
- [ ] ScrToday.pa.yaml：標題列＋使用者資訊
- [ ] Gallery：Filter(Email=User().Email, DateString=今日)＋fallback 切換
- [ ] Overlay Popup：明細＋多行輸入＋取消/送出（Patch＋Notify）
- [ ] yaml 語法驗證（python yaml.safe_load）
- [ ] 匯入指南 HTML
- [ ] TaskLog／INDEX 更新
