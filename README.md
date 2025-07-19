# Gemini-Pool - Gemini API 金鑰池

[繁體中文](README.md) | [English](README_EN.md)

<blockquote style="color: red;">
  <p><strong>警告！</strong><br>
  Gemini-Pool v0.17a版本存在重大漏洞，建議升級至Gemini-Pool v0.59a來修復已知漏洞</p>
</blockquote>

你可以前往此地查看最新版本的README
https://himservice-docs.himserver.com/#Gemini-Pool/intro

## 概覽

Gemini-Pool 是一個Google Gemini API 金鑰池專案，使用 FastAPI 框架開發。它旨在幫助開發者更有效地管理和使用多個 Gemini API 金鑰，並提供自動負載平衡、金鑰輪換、失敗重試和請求日誌的監控面板。

### 特點
- **金鑰池管理**: 集中管理所有金鑰。
- **自動輪換與負載平衡**: 在可用的金鑰之間輪換，將請求壓力平均分配。
- **失敗轉移 (Failover)**: 當某個金鑰失效時，自動切換到下一個可用金鑰，確保服務可用。
- **定期檢查**: 自動定期檢查失效金鑰，並在恢復後自動將其加回運行池。
- **詳細儀表板**: 提供網頁介面，即時監控金鑰狀態、請求流量和詳細日誌。
- **動態設定**: 可透過儀表板動態管理金鑰與設定，多數操作無需重啟服務。
- **存取控制**: 可設定允許訪問此代理服務的客戶端 API 金鑰，並進行流量的細緻管理。
- **日誌管理**: 詳細的請求日誌。

## 工作流程

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

## 使用說明

### 登入管理後台
- **存取位址**: `http://localhost:8000`
- **登入**: 使用您在 `.env` 中設定的 `ADMIN_USERNAME` 和 `ADMIN_PASSWORD` 登入。成功後會導向到 `/status` 儀表板。

### API 代理使用
您的應用程式需要將原本直接請求 Google Gemini API 的地方，改為請求本代理服務的端點。

- **代理端點**: `http://localhost:8000/v1beta/models/{model_id}:generateContent`
- **驗證方式**: 在請求的 `Authorization` 標頭中帶上您在 `ALLOWED_API_KEYS` 中設定的其中一個金鑰。

### cURL 範例
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

## ⚙️ 基礎設定

本專案的所有設定都透過 `.env` 檔案進行管理。

[前往DOCS查看配置](https://himservice-docs.himserver.com/#Gemini-Pool/set-env)
網站目前僅支持繁體中文，請見諒!!!
## API 端點

本專案提供兩種 API 端點：管理端點和代理端點。

[前往DOCS查看說明](https://himservice-docs.himserver.com/#Gemini-Pool/api)
網站目前僅支持繁體中文，請見諒!!!

## 報告問題

## 開發與貢獻

## 許可證
請閱讀 `LICENSE` 文件。
