# 員工出勤儀表板 — Claude Code 交接文件

版本：v1 | 建立：2026-06-07 本文件供 Claude Code 新 session 開場使用，包含完整背景、現況、待辦與技術限制。

---

## 一、專案背景（快速版）

**環境**：MS365 Apps 商務版，10–50 人

**目標**：員工開啟 SharePoint 頁面，看到自己的出勤查核紀錄、請假紀錄、假期餘額，並可在儀表板直接填寫出勤回覆。

**上班制度**：預設 10–19，有 Planner 延遲工作者須 09:30 到。出勤查核由行政人員人工核對後填入系統。

---

## 二、系統架構

寬格式 Excel（行政每日填）

    ↓ \[人工，目標自動化\]

Office Scripts 轉長格式

    ↓

SharePoint List: AttendanceHistory

    ↓

Power Apps Canvas App

    ↓

嵌入 SharePoint 頁面（員工瀏覽器存取）

Office Scripts 自動化尚未完成，目前人工將資料寫入 AttendanceHistory。

---

## 三、資料結構

### SharePoint List：`AttendanceHistory`

| 欄位 | 類型 | 說明 |
| :---- | :---- | :---- |
| 日期 | 日期 | 格式 yyyy/mm/dd |
| Email | 單行文字 | 員工登入帳號，Filter 主鍵 |
| 員工編號 | 單行文字 |  |
| 姓名 | 單行文字 |  |
| 方案名稱 | 單行文字 | 有延遲工作才有值，無則空白 |
| 任務名稱 | 單行文字 | 有延遲工作才有值，無則空白 |
| 表單回覆 | 單行文字 | **員工填寫**（已處理 / 待確認 / 其他：自填） |
| 實際出勤狀態 | 單行文字 | **行政填寫，員工唯讀** |
| DateString | 單行文字 | 格式 `yyyy/m/d`（不補零），委派 Filter 用 |

**狀態選項**（實際出勤狀態）： 0930已到 / 1000已到 / 10:00前已發\*\* / 1000未到已提醒 / 1000未到已發米米信 / 異地 / 已發\*\*先去場館後進公司 / 當日休假

### SharePoint Excel 檔（兩個）

- **請假申請表**：欄位含 假別 / 日期 / 狀態 / 員工 Email  
- **假期餘額表**：欄位含 假別餘額 / 員工 Email

---

## 四、Power Apps 開發現況

### App OnStart（已確認）

Set(varMonth, Today());

Set(varShowPopup, false);

Set(varSelectedRecord, First(Filter(AttendanceHistory, false)))

### Screen2 Gallery（已確認）

**Items：**

Filter(

    AttendanceHistory,

    Email \= User().Email,

    DateString \= Text(Today(), "yyyy/m/d")

)

**OnSelect：**

If(

    \!IsBlank(ThisItem.方案名稱),

    Set(varSelectedRecord, ThisItem);

    Set(varShowPopup, true)

)

---

## 五、模組開發狀態

| 模組 | 狀態 | 說明 |
| :---- | :---- | :---- |
| 模組一：出勤查核 | **卡關** | Popup 結構尚未成功建立（見第六節） |
| 模組二：請假紀錄 | 未開始 |  |
| 模組三：假期餘額 | 未開始 |  |
| 角色權限 | 未開始 | 行政 vs 一般員工 |
| SharePoint 嵌入 | 未完成 | 需先完成 App |

---

## 六、模組一當前卡關點

### 目標：Popup 彈窗

員工點擊 Gallery 中有延遲工作的列 → 彈出 Popup，顯示：

- 方案名稱（唯讀 Label）  
- 任務名稱（唯讀 Label）  
- 下拉選單（已處理 / 待確認 / 其他）  
- 文字輸入框（只在選「其他」時顯示）  
- 關閉按鈕（`Set(varShowPopup, false)`）  
- 儲存按鈕（Patch 回 `AttendanceHistory.表單回覆`）

### 已知問題

1. **YAML 無法貼入** — Power Apps 的 YAML 貼入在這個環境無效，不走 YAML 路線  
2. **X/Y 定位無效** — 已關閉「縮放以符合視窗大小」後，容器的 X/Y 屬性在響應式模式下無效  
3. **Popup 置中失效** — PopupContainer 放在 ScreenContainer 內時位置跑掉

### 正確的 Popup 結構（需手動逐步建立）

Screen2（根層級）

├── ScreenContainer1（主畫面內容）

│   └── Gallery1（出勤列表）

└── OverlayContainer ← 必須在根層級，與 ScreenContainer1 同層

    └── PopupCard（垂直容器）

        ├── LabelPlanName（方案名稱）

        ├── LabelTaskName（任務名稱）

        ├── DropdownReply（下拉選單）

        ├── TextInputOther（文字輸入，Visible 條件顯示）

        └── ButtonRow（水平容器）

            ├── ButtonClose

            └── ButtonSave

### 各元件設定

**OverlayContainer（垂直容器，Screen 根層級）**

- Width：`Parent.Width`  
- Height：`Parent.Height`  
- X：`0`，Y：`0`  
- Fill：`RGBA(0, 0, 0, 0.4)`  
- LayoutAlignItems：`LayoutAlignItems.Center`  
- LayoutJustifyContent：`LayoutJustifyContent.Center`  
- Visible：`varShowPopup`

**PopupCard（垂直容器，OverlayContainer 內）**

- Width：`Parent.Width * 0.9`  
- LayoutGap：`12`  
- Padding：四邊 `16`  
- Fill：`RGBA(255, 255, 255, 1)`  
- BorderColor：`RGBA(200, 200, 200, 1)`，BorderThickness：`1`

**LabelPlanName**

- Text：`varSelectedRecord.方案名稱`  
- Width：`Parent.Width - 32`  
- FontWeight：`FontWeight.Semibold`

**LabelTaskName**

- Text：`varSelectedRecord.任務名稱`  
- Width：`Parent.Width - 32`

**DropdownReply（命名 DropdownReply）**

- Items：`["已處理", "待確認", "其他"]`  
- Width：`Parent.Width - 32`

**TextInputOther（命名 TextInputOther）**

- Mode：`TextMode.MultiLine`  
- Width：`Parent.Width - 32`  
- Visible：`DropdownReply.Selected.Value = "其他"`  
- HintText：`"請說明原因"`

**ButtonRow（水平容器）**

- LayoutJustifyContent：`LayoutJustifyContent.End`  
- Width：`Parent.Width - 32`

**ButtonClose**

- Text：`"關閉"`  
- Width：`(Parent.Width - 12) * 0.4`  
- OnSelect：`Set(varShowPopup, false)`

**ButtonSave**

- Text：`"儲存"`  
- Width：`(Parent.Width - 12) * 0.4`  
- OnSelect：

Patch(

    AttendanceHistory,

    varSelectedRecord,

    {

        表單回覆: If(

            DropdownReply.Selected.Value \= "其他",

            TextInputOther.Text,

            DropdownReply.Selected.Value

        )

    }

);

Set(varShowPopup, false)

---

## 七、技術限制與已知地雷

| 項目 | 錯誤做法 | 正確做法 |
| :---- | :---- | :---- |
| 委派 Filter | `Year(日期)` / `Month(日期)` | `DateString = Text(Today(), "yyyy/m/d")` |
| 月份排序 | `Months` | `TimeUnit.Months` |
| 排序方向 | `Descending` | `SortOrder.Descending` |
| 對齊語法 | `LayoutJustify.SpaceBetween` | `LayoutJustifyContent.SpaceBetween` |
| 響應式寬高 | 固定數值（如 `400`） | 一律用 `Parent.Width` / `Parent.Height` 比例 |
| Popup 定位 | X/Y 屬性 | OverlayContainer 根層級 \+ LayoutJustifyContent.Center |
| DateString 格式 | `yyyy/mm/dd`（補零） | `yyyy/m/d`（不補零） |
| YAML 匯入 | 貼入 YAML | 此環境無效，改逐步手動建立 |

---

## 八、角色權限設計（尚未開發）

| 角色 | 判斷方式 | 權限 |
| :---- | :---- | :---- |
| 一般員工 | 預設 | 填寫表單回覆、查看自己紀錄 |
| 行政人員 | 聯絡人表中特定 Email | 另可編輯實際出勤狀態 |

判斷方式：App OnStart 時查聯絡人表，`Set(varIsAdmin, !IsBlank(LookUp(聯絡人表, Email = User().Email, 角色)))` 之類，具體公式待開發時確認欄位名稱。

---

## 九、待辦清單（Claude Code 執行順序）

1. **模組一 Popup** — 按第六節逐步手動建立，確認可運作  
2. **Office Scripts** — 寬格式 Excel → 長格式寫入 AttendanceHistory（自動化）  
3. **模組二** — 請假紀錄（月份切換 \+ Filter by Email）  
4. **模組三** — 假期餘額（讀假期餘額 Excel）  
5. **角色權限** — 行政帳號可編輯實際出勤狀態  
6. **SharePoint 嵌入** — 最後一步

---

## 十、新 session 開場指示（給 Claude Code）

本次任務：繼續開發 MS365 員工出勤儀表板 Power Apps Canvas App

環境限制：

\- YAML 無法貼入，所有元件設定需提供逐步手動操作步驟

\- 響應式模式，所有寬高用 Parent.Width/Parent.Height 比例，禁止固定數值

\- DateString 欄位格式 yyyy/m/d（不補零）

當前任務：完成模組一 Popup 彈窗（見交接文件第六節）

請先確認我目前 Screen2 的樹狀結構，再開始指導。  
