# BrightSpot Kids Evaluator — 設計文件
**日期：** 2026-03-25
**狀態：** 已確認

---

## 目標
將現有 React 元件重構為純 HTML 單檔，整合 Gemini API 讓老師用自己的 API Key 生成 AI 評語，並部署至 GitHub Pages 供幼教老師免費使用。

---

## 技術架構

### 單檔結構（index.html）
- React 18（CDN）
- Babel Standalone（瀏覽器端 JSX 轉換）
- Tailwind CSS（CDN）
- Lucide React（CDN）
- Mammoth.js（CDN，Word 解析）
- Gemini API（`gemini-1.5-flash`，使用者自備 Key）

### 生成模式
| 模式 | 觸發條件 | 說明 |
|------|----------|------|
| 🟢 AI 模式 | 已設定 Gemini API Key | 呼叫 Gemini API 生成自然語言評語 |
| 🟡 離線模式 | 未設定 Key 或 API 失敗 | 使用原有模板邏輯自動降級 |

---

## 新增功能：API Key 管理

### 入口
Header 右上角新增「⚙️ 設定」按鈕，開啟設定面板。

### 設定面板內容
1. **申請引導**（Step-by-step 折疊區）
   - Step 1：前往 Google AI Studio（aistudio.google.com，附可點擊連結）
   - Step 2：用 Google 帳號登入
   - Step 3：點選「Get API Key」→「Create API Key」
   - Step 4：複製 Key 貼入下方欄位

2. **API Key 輸入欄**
   - 密碼格式（`type="password"`）
   - 右側附「顯示/隱藏」切換按鈕
   - Key 儲存於 `localStorage`（key: `gemini_api_key`）

3. **模式狀態指示器**
   - 🟢 AI 模式已啟用（Key 已設定）
   - 🟡 離線模式（未設定 Key）

### Gemini API 呼叫
- Endpoint：`https://generativelanguage.googleapis.com/v1beta/models/gemini-1.5-flash:generateContent?key={API_KEY}`
- 方法：POST，JSON body
- Prompt：將所有評量資料組成結構化 prompt，要求以繁體中文輸出幼教評語
- 失敗時：自動降級至模板模式並顯示 Toast 提示

---

## UX 優化項目

- **Toast 通知**取代 `alert()`（複製成功、API 錯誤等）
- **手機版 Header** 按鈕改為 icon-only
- **步驟 1** 未填完整資料時，「下一步」按鈕顯示具體缺少項目提示
- **生成中** 加入 spinner 動畫與 AI 思考狀態文字

---

## 程式碼品質優化

- 消除重複的 `analyzeSymbols` 呼叫
- 修正 `extractAssessmentItems` 邊緣案例（空行、特殊符號混排）
- `localStorage` 讀寫統一加入 try/catch

---

## GitHub 部署

- **Repo：** `rainbowbox-bit/brightspot-kids-evaluator`（公開）
- **Branch：** `main`
- **Pages Source：** `main` branch / root（`index.html`）
- **URL：** `https://rainbowbox-bit.github.io/brightspot-kids-evaluator/`
