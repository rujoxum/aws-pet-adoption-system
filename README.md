# ☁️ AWS 雲端寵物領養平台 (Cloud Pet Adoption System)

> **課程專題**：雲端運算之規劃與管理
> **技術關鍵字**：`AWS EC2` `Amazon DynamoDB` `Amazon S3` `Node.js` `Express` `REST API` `PuTTY`

---

## 📌 專案背景與系統情境 (System Context)

本專案旨在解決傳統領養平台在圖片傳送時對伺服器頻寬與儲存空間造成的負擔，並確保領養狀態在多使用者同時存取時能保持資料一致性。

我們設計了一個兼具高可用性與動態/靜態資料解耦的寵物領養平台：
1. **收容所端**：登記寵物的姓名、年齡、種類與領養狀態至雲端 NoSQL 資料庫，並上傳寵物大頭照至雲端物件儲存桶。
2. **領養人端**：透過 Web 介面瀏覽寵物列表與相片。點擊領養申請後，系統自動將該寵物狀態鎖定為「已領養」，防止重複媒合。若後續家訪未通過或有棄養紀錄，收容所可重置狀態再次開放領養。

---

## 🏗️ 雲端服務架構設計 (Cloud Architecture Design)

系統採用 AWS 原生服務進行架構切割與微服務化解耦：

* **AWS EC2 (Web & Application Server)**
  * 執行 Ubuntu Linux 作業系統，託管 Node.js / Express 後端應用程式。
  * 負責處理前端 HTTP / REST API 請求、執行商業邏輯判斷（如領養狀態變更），並透過 AWS SDK 與 DynamoDB 及 S3 進行資料存取與檔案寫入。
* **Amazon DynamoDB (NoSQL Database)**
  * 全託管（Fully Managed）Key-Value NoSQL 資料庫，負責儲存貓咪與狗狗的文字型資料（如 `name`、`age`、`type`、`adopted` 領養狀態）。
  * 提供毫秒級的讀寫回應速度，無須管理伺服器硬體底層，大幅提升資料擴充性。
* **Amazon S3 (Object Storage)**
  * 託管所有寵物的真實大頭照。
  * 圖片上傳後直接產生公開 URL 供前端用戶下載與渲染，極大化節省 EC2 伺服器的儲存空間與網路傳輸頻寬。
<p align="center">
<img width="591" height="515" alt="截圖 2026-10-12 下午4 38 56" src="https://github.com/user-attachments/assets/4b3df295-4857-4337-818b-3fc222cdf957" />
</p>

## 🛠️ 實務除錯與維運紀錄 (Technical Debugging & Maintenance)

### 1. DynamoDB 資料型態隱式轉換與判斷修復 (Type Mismatch Debugging)
* **問題現象**：在初期測試中，無論 DynamoDB 資料庫登記為 `true` 或 `false`，網站前端渲染出來的寵物狀態始終全部被強制顯示為「已領養」。
* **原因分析**：發現 DynamoDB 欄位型態由原始字串 (`String`) 被系統自動轉譯或編輯為布林值 (`Boolean`)。JavaScript 弱型別在執行 `pet.adopted === true` 比較時，傳入的字串 `"false"` 在條件式中被隱式轉換判定成布林真值（Truthy），導致邏輯失真。
* **解決方案**：重構後端判斷邏輯，強化資料型態相容性：
  ```javascript
  // 修正前：單一型別比較導致字串型態判斷失準
  if (pet.adopted === true) ...

  // 修正後：兼顧 Boolean 與 String 資料型態相容性
  if (pet.adopted === true || pet.adopted === "true") {
      // 正確執行鎖定領養狀態與按鈕禁用邏輯
  }

### 2. EC2 伺服器生命週期與服務啟動監控 (EC2 Lifecycle & Process Management)
* **維運問題**：當關閉 EC2 實例後，網站便無法連線，跳出 400/Timeout 錯誤。
* **維運 SOP 建立**：
  1. 使用 SSH 工具（如 PuTTY）遠端連線至 EC2 Linux 虛擬主機。
  2. 進入專案目錄 `cd pet-site` 並執行 `node app.js` 啟動背景伺服器與 Port 3000 監聽。
  3. 掌握 EC2 服務重新啟動後的環境檢測與背景進程維運。

---

## 🎓 專案收穫與技術成長 (Key Outcomes)

1. **雲端架構實作能力**：掌握 AWS EC2、DynamoDB 與 S3 等服務之間的 API 串接與資料流互動。
2. **前後端與資料庫整合**：了解雲端微服務架構中，前端 UI、後端 API 伺服器與雲端資料庫之間的溝通流程。
3. **系統除錯與問題排除**：親身經歷資料型態不一致（Type Mismatch）、伺服器部署連線等實務問題，培養獨立分析與修復 Bug 的能力。

---

## 🛠️ 技術棧總覽 (Tech Stack Summary)

* **Cloud Infrastructure**: AWS EC2, Amazon DynamoDB, Amazon S3
* **Backend & API**: Node.js, Express, AWS SDK for JavaScript
* **Database & Storage**: NoSQL Key-Value Store, Object Storage
* **DevOps & Tools**: Linux (Ubuntu), PuTTY SSH, Git
