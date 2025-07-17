# Gemini-Pool - Gemini API 金鑰池

[繁體中文](README.md) | [English](README_EN.md)

## 概覽

Gemini-Pool 是一個Google Gemini API 金鑰池專案，使用 FastAPI 框架開發。它旨在幫助開發者更有效地管理和使用多個 Gemini API 金鑰，並提供自動負載平衡、金鑰輪換、失敗重試和請求日誌的監控面板。

### ✨ 特點
- **金鑰池管理**: 集中管理所有金鑰。
- **自動輪換與負載平衡**: 在可用的金鑰之間輪換，將請求壓力平均分配。
- **失敗轉移 (Failover)**: 當某個金鑰失效時，自動切換到下一個可用金鑰，確保服務可用。
- **定期檢查**: 自動定期檢查失效金鑰，並在恢復後自動將其加回運行池。
- **詳細儀表板**: 提供網頁介面，即時監控金鑰狀態、請求流量和詳細日誌。
- **動態設定**: 可透過儀表板動態管理金鑰與設定，多數操作無需重啟服務。
- **存取控制**: 可設定允許訪問此代理服務的客戶端 API 金鑰，並進行流量的細緻管理。
- **日誌管理**: 詳細的請求日誌。

## 🚀 工作流程

本專案的核心工作流程如下：

1.  **請求攔截**：您的應用程式將 API 請求發送至 `Gemini-Pool`，而非直接發送至 Google。
2.  **金鑰選取**：`Gemini-Pool` 從其管理的金鑰池中，選取當前可用且負載較低的 API 金鑰。
3.  **請求轉發**：使用選定的金鑰，將原始請求轉發至 Google Gemini API 端點。
4.  **成功回傳**：若請求成功，`Gemini-Pool` 會將來自 Google 的回覆原封不動地傳回給您的應用程式。
5.  **失敗處理**：若請求失敗（例如，因金鑰額度用盡或失效），系統會自動將該金鑰標記為暫時不可用，並立即選取**下一個**可用金鑰重試請求。此過程對您的應用程式完全透明。
6.  **日誌與監控**：所有請求的詳細資訊，包括使用的金鑰、延遲、成功或失敗狀態，都會被詳細記錄，並可在管理儀表板上進行即時監控與分析。



<blockquote style="color: red;">
  <p><strong>警告！</strong><br>
  我們無法保證Gemini-Pool的進階安全性和穩定性，建議僅以測試與體驗之目的使用。</p>
</blockquote>


## 🛠️ 安裝

### 條件
- Python 3.8 或更高版本。
- 一個或多個有效的 Google Gemini API 金鑰。

### 步驟
1. **安裝依賴**
   ```bash
   pip install -r requirements.txt
   ```

2. **設定環境變數**
   - 在`gemini_key_pooler`目錄下，複製 `.env.example` 為 `.env` (若無，請手動建立)。
   - 編輯 `.env` 檔案，填寫必要資訊。詳見下一節「自訂選項」。

4. **啟動伺服器**
   ```bash
   python -m gemini_key_pooler.main
   ```
   - 控制台將顯示 `Uvicorn running on http://0.0.0.0:8000`
   - 在瀏覽器中訪問 `http://localhost:8000` 即可進入登入頁面。

## 📖 使用說明

### 1. 登入管理後台
- **存取位址**: `http://localhost:8000`
- **登入**: 使用您在 `.env` 中設定的 `ADMIN_USERNAME` 和 `ADMIN_PASSWORD` 登入。成功後會導向到 `/status` 儀表板。

### 2. API 代理使用
您的應用程式需要將原本直接請求 Google Gemini API 的地方，改為請求本代理服務的端點。

- **代理端點**: `http://localhost:8000/v1beta/models/{model_id}:generateContent`
- **驗證方式**: 在請求的 `Authorization` 標頭中帶上您在 `ALLOWED_API_KEYS` 中設定的其中一個金鑰。

#### cURL 範例
```bash
# 假設您的 ALLOWED_API_KEYS 中包含 "CLIENT_KEY_1"
curl -X POST http://localhost:8000/v1beta/models/gemini-1.5-flash:generateContent \
-H "Authorization: Bearer CLIENT_KEY_1" \
-H "Content-Type: application/json" \
-d '{
  "contents": [{
    "parts":[{
      "text": "Hi,:D"
    }]
  }]
}'
```

## ⚙️ 自訂選項

### 修改 `.env` 檔案
`gemini_key_pooler/.env` 是主要的設定檔。

```ini
# --- 核心設定 ---
# 您的 Google Gemini API 金鑰，用逗號分隔
GEMINI_API_KEYS="YOUR_GEMINI_KEY_1,YOUR_GEMINI_KEY_2"

# --- 資料庫設定 ---
# 預設使用 SQLite。若要使用 PostgreSQL，請修改此 URL
# 例如: DATABASE_URL="postgresql+asyncpg://user:password@host:port/dbname"
DATABASE_URL="sqlite+aiosqlite:///./gemini_pool.db"

# --- 管理後台設定 ---
ADMIN_USERNAME="admin"
ADMIN_PASSWORD="your_secure_password" # !!! 請務必修改為一個安全的密碼 !!!

# --- 安全性設定 ---
# 用於 JWT 簽名的密鑰，請修改為一個隨機且複雜的字串
SECRET_KEY="a_very_secret_key_please_change_me"

# --- 存取控制 ---
# 允許呼叫此代理服務的 API 金鑰，用逗號分隔。
# 如果留空，則任何客戶端都可以存取（不建議）。
ALLOWED_API_KEYS="CLIENT_KEY_1,CLIENT_KEY_2"

# --- 可選設定 ---
MAX_FAILURES_PER_KEY=3          # 每個金鑰連續失敗多少次後會被標記為失效
KEY_RECHECK_INTERVAL=3600       # 失效金鑰的健康檢查間隔（秒）
REQUEST_TIMEOUT=120             # 請求 Gemini API 的超時時間（秒）
ACCESS_TOKEN_EXPIRE_MINUTES=43200 # 管理員登入 Token 的有效期限（分鐘）
LOG_RETENTION_DAYS=30           # 請求日誌保留天數（天），設為 0 表示不清理
PORT=8000                       # 應用程式運行的埠號
DEBUG=False                     # 是否啟用除錯模式
```

## 📜 API 端點

本專案提供兩種 API 端點：管理端點和代理端點。

### 管理端點
主要透過管理後台介面操作。以下是完整的端點列表：

- **狀態與日誌**:
  - `GET /api/status`: 獲取金鑰池和請求的統計數據。
  - `GET /api/request_logs`: 獲取請求日誌列表。
  - `GET /api/request_logs/{log_id}`: 獲取單一請求日誌的詳細資訊。
- **Gemini 金鑰管理**:
  - `GET /api/keys/{key_id}`: 獲取一個 Gemini 金鑰的完整字串。
  - `POST /api/keys`: 新增一個 Gemini 金鑰。
  - `POST /api/keys/bulk_add`: 批次新增多個 Gemini 金鑰。
  - `DELETE /api/keys/{key_id}`: 刪除一個 Gemini 金鑰。
  - `POST /api/keys/bulk_delete`: 批次刪除多個 Gemini 金鑰。
  - `POST /api/keys/{key_id}/reset`: 重設一個 Gemini 金鑰的錯誤計數。
  - `POST /api/keys/bulk_reset`: 批次重設多個 Gemini 金鑰的錯誤計數。
  - `POST /api/keys/{key_id}/test`: 測試一個 Gemini 金鑰的有效性。
- **客戶端存取金鑰管理**:
  - `GET /api/allowed_keys`: 獲取所有允許的客戶端 API 金鑰列表。
  - `POST /api/allowed_keys`: 新增一個允許的客戶端 API 金鑰。
  - `PUT /api/allowed_keys/{key_id}`: 更新一個允許的客戶端 API 金鑰的設定。
  - `DELETE /api/allowed_keys/{key_id}`: 刪除一個允許的客戶端 API 金鑰。

### Gemini 代理端點
- `POST /v1beta/models/{model_id}:generateContent`: 代理標準的 Gemini `generateContent` 請求。
- `POST /v1beta/models/{model_id}:streamGenerateContent`: 代理支援串流的 Gemini `streamGenerateContent` 請求。

## 🐞 報告問題
如果發現 bug 或有功能建議，請在 GitHub 創建 Issue，提供詳細描述和重現步驟。

## 🤝 開發與貢獻
歡迎任何形式的貢獻！如果您有興趣，請 Fork 本專案並提交 Pull Request。

## 📄 許可證
本專案採用 [MIT License](LICENSE) 授權。請閱讀 `LICENSE` 文件以獲取更多資訊。
