# Lessons Learned

## 2026-06-22（Power Apps / Power Automate 平台踩坑彙整）
- **Power Automate 運算式必須用「fx 插入成 token」，不可在欄位直接打字**：直接打 `formatDateTime/concat/toLower/...` 會被當「字面字串」不計算（症狀：執行輸出出現整段運算式文字、或 Select 報「請輸入有效的 JSON」）。這是本次 0945 反覆卡關的真正主因。判斷：欄位裡是灰字＝字面(錯)、彩色 chip＝運算式(對)。**指南一律提供可整段貼的運算式，並註明「用 fx 插入」**。
- **SharePoint 清單欄位「內部名」≠ 顯示名**：CSV 匯入/中文欄常變 `field_N`（Email→field_3、姓名→field_4…）；運算式用顯示名 `item()?['Email']` 會回 null。**從「取得項目」一次執行的原始輸出 JSON key 即可確認對照**，別猜。Title、Email(若 ASCII)有時保留原名，但本案連 Email 都成 field_3。
- **Filter array 是「相對於迴圈值」過濾，不是原樣輸出**：「輸入有列、輸出卻空」＝where 對該列不成立（常是右側 `items('loop')` 值錯），不是該列有問題。**最快定位＝把比較一側寫死**（如 `equals(item()?['field_3'],'某email')`）跑一次：過＝右側(迴圈值)錯、不過＝左側(欄位)錯。
- **Select 文字模式(字串陣列) vs 鍵/值模式(物件陣列) 會改變下游 `items()` 型別**：兩者與消費端引用必須一致（字串用 `items('loop')`、物件用 `items('loop')?['key']`）。模式不確定時，`@or(equals(...,items()), contains(toLower(replace(string(items()),' ','')), '"key":"val"'))` 可同時涵蓋兩種。**但 Select 的值仍須是被計算的運算式**，否則兩種都不中。
- **低代碼某元件反覆搞不定時，換成更可靠的等價做法**：0945 的「Select 去重 email」屢卡 → 改用既有 team_member(Excel) 當人員名冊跑迴圈（姓名/Email 走 Excel header、清單只引 field_3），直接移除會壞的 Select。與行政頁簽同模式。
- **ai-team 繞圈時找第二個 agent（Codex headless）拿獨立診斷有用，但真正卡點常是缺執行期資料**：先用「寫死測試」取得 run output 證據再下判斷，勝過反覆改公式空想。
- **pa.yaml 貼上是「控件層級、一次一畫面」**：把兩個畫面塞同一檔整段貼會 PA1001（YamlInvalidSyntax）。維持「一檔＝一畫面、全選整段貼」。官方舊「preview 格式」貼上已淘汰，但單一畫面的控件清單仍可貼。
- **GroupBy v3 語法用識別碼非字串**：群組欄名 `"明細"`(字串)→`'明細'`(識別碼)，否則「預期的識別碼名稱／引數無效」連鎖報錯。**更穩的是根本不用 GroupBy**：行政頁簽人員名冊直接用 team_member 表＋`LookUp` 取當日狀態，避開 GroupBy 的委派與語法坑。
- **Power Automate OData `$filter` 用「內部欄名」**：中文欄或改名欄的內部名與顯示名不符 → 報「欄不存在」。`Title` 內部名固定，可用 `startswith(Title,'yyyyMMdd_')`。且 `$filter` 欄**不能手打運算式文字**（不會被計算、變字面字串），要嘛用動態內容引用 Compose、要嘛巢狀單引號會把內層字串(如時區)截壞。**能用 Filter array＋動態內容就別硬塞 `$filter`**。
- **convertFromUtc 的 Windows 時區 ID 可能不被吃**：某些 PA 環境 `convertFromUtc(utcNow(),'Taipei Standard Time')` 報「時區無效」。台北固定 UTC+8 無日光節約 → 用 `addHours(utcNow(),8)` 最穩，完全不碰時區庫。
- **SharePoint「Email」欄別建成「個人或群組」**：作為比對鍵的 email/識別欄要用「單行文字」，否則 `Email = User().Email` 比對失敗（個人欄是 record 非字串）、清單顯示成人名。識別/join 欄一律單行文字。
- **Power Fx `=` 文字比對本就不分大小寫**：`Lower()` 多餘且會觸發委派警告，去掉即可。
- **改名後的內建 Title**：清單檢視別名（如「ID (yyyyMMdd_Email)」）在 Power Automate「建立項目」仍顯示為 `Title`；唯一鍵填 `Title`。若報「`Item/Title` 已不存在」是舊動作殘留孤兒參照，刪舊動作重建即可。
- **Canvas 垂直 gallery 列高固定**：無法每列依內容動態高。內容多寡不一時，用內嵌 gallery＋捲軸，或固定夠高（無完美的 per-row 自動高）。
- **App 忠實呈現資料源、欄位不會自己錯開**：使用者回報「編號跟姓名對不上」，比對後是來源 team_member 的員工編號本身排錯（3 個林姓對調），非程式 bug。同一列的多欄不可能被程式錯開——先用 Email join 比對來源資料找出處，別預設是 UI 問題。
- **沙盒產出的檔帶 `com.apple.provenance`（受保護、清不掉）**：macOS 會每次開檔重新隔離、跳 Gatekeeper。沙盒內（含關閉 sandbox 的 cp -X）都移不掉；只能由使用者在自己的 Terminal.app 重建檔案。匯入 SharePoint 不經 Gatekeeper、不受影響。

## 2026-06-16
- **儲存選型準則：唯讀小表走 Excel 直連，多人寫＋需權限走清單**：Power Apps 可直連 SharePoint/OneDrive 上的 Excel（須格式化 Table），但僅適合「唯讀＋小量（<2000 列）＋低頻」——如請假餘額、team_member，這種建清單反而多餘。反之，會被多人並發寫、需欄位／項目級權限、或資料可能累積的，必須用 SharePoint 清單：如出勤 AttendanceHistory（員工填回覆＋行政填狀態＋0930 自動寫入三方並寫、`實際出勤狀態` 要員工唯讀），Excel 會鎖檔／互相覆蓋、做不到欄位權限、又非委派受 2000 列上限。**按讀寫特性分流，不是全清單或全 Excel。**
- **要列的清單若上游已同步全體，就從現有資料去重，別反射性再接一個來源**：行政頁簽要「列當天所有員工」，第一直覺想接 team_member，但 09:40 flow 已把全體同步進 AttendanceHistory，直接 `GroupBy` 當日資料去重即可；team_member 只留作判權限。少接一個資料源＝少一層委派/同步風險。
- **Power Apps 的 `Visible` 是畫面層、非安全邊界**：用 Visible 隱藏行政頁簽只擋一般員工的眼睛，敏感欄位（實際出勤狀態）的寫入硬保障必須靠 SharePoint 清單欄位／項目權限，不能只靠前端隱藏。設計權限時 UI 與資料源兩層都要顧。
- **舊自動化的實際機制要先讀碼再設計，別照「架構名詞」猜**：我原以為發 Line 是 Power Automate 的 Line 連接器，規劃了「§3-5 段 C 發 Line」。實際發 Line 是獨立的 `Planner2Line`（Windows 桌機程式監聽 Outlook 延遲通知信→LINE Desktop 轉發，連結取自郵件正文＋自動縮網址）。已註冊專案本機就有 clone（見 projects-registry），花兩分鐘讀碼即釐清，勝過閉門規劃。接手涉及既有系統時，先定位並讀現行程式碼。
- **欄位名可能與實際內容不符（舊名沿用）**：清單 RowKey 欄名是 `yyyyMMdd_Email`，但實際內容含任務 ID（`日期_email_任務ID`）。我據欄名推斷「同人同日會碰撞」其實是誤判。別照欄名推內容，向使用者確認實際值。
- **過程工作檔放 `workingfiles/`，不放 `outputs/`**（被使用者糾正）：YAML、匯入指南、變更說明、flow 建置規格都是開發中工作檔，`outputs/` 只放正式交付成果。動手放檔前先對齊 GLOBAL 檔案規範。
- **效益溝通：禁止附和語與無實質鋪陳**（被使用者糾正）：「你的提醒對」「好的」、可有可無的比較表都要砍，回應只給資訊／推論／判斷。

## 2026-06-14
- **舊交接記的環境限制要再驗，別當永久前提**：handover 斬釘截鐵寫「Power Apps YAML 無法貼入，不走 YAML 路線」，三個 AI 據此全規劃成「逐步手動建立」。2026-06-14 實測官方 `pa.yaml` paste code 可貼入並執行——舊限制是**當時舊格式**的結論，新功能上線後即失效。接手時對「環境做不到 X」類限制先快速實測再採信，過時前提會讓整條路線繞遠路。
- **上游自動化流程（Power Automate 09:30 查 Planner→通知行政→Line 通知員工）先前完全未落檔**，靠使用者口述才補錄到 INDEX。關鍵架構即使「不是這次要做的部分」也該記，否則每次接手都要重問。

## 2026-06-08
- `claude -p` 在使用者 Terminal 可用，不代表 Codex 沙盒內可用；若回 `Not logged in`，需用核准後的沙盒外執行測試。
- 讓外部 agent 直接讀寫檔案可能卡住或產生編碼問題；較穩定做法是請 agent 只輸出文字意見，再由 Tech Lead 寫入專案檔。
- ai-team 討論稿若屬過程文件，應放 `workingfiles/`，正式成果才放 `outputs/`。
