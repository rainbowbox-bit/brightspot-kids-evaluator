# BrightSpot Kids Evaluator Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** 將 React 評語生成器重構為純 HTML 單檔，整合 Gemini API，並部署至 GitHub Pages。

**Architecture:** 單一 `index.html`，透過 CDN 載入 React/Babel/Tailwind/Lucide/Mammoth。新增 Gemini API Key 設定面板（含申請引導）與 Toast 通知系統。API 呼叫失敗時自動降級為模板模式。

**Tech Stack:** React 18 (CDN), Babel Standalone, Tailwind CSS (CDN), Lucide React (CDN), Mammoth.js (CDN), Gemini API (gemini-1.5-flash)

---

### Task 1：初始化 Git 倉庫並建立 HTML 骨架

**Files:**
- Create: `index.html`

**Step 1：初始化 git**

```bash
cd "C:/Users/rainb/vbtools/BrightSpot Kids Evaluator"
git init
git branch -M main
```

**Step 2：建立 index.html 骨架（CDN 全部就位）**

```html
<!DOCTYPE html>
<html lang="zh-TW">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>看見孩子身上的星星 — 評語生成器</title>
  <meta name="description" content="幼教老師總結性評語 AI 生成工具，支援 Gemini AI 與離線模板雙模式。" />
  <script src="https://cdn.tailwindcss.com"></script>
  <script src="https://unpkg.com/react@18/umd/react.production.min.js"></script>
  <script src="https://unpkg.com/react-dom@18/umd/react-dom.production.min.js"></script>
  <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/mammoth/1.6.0/mammoth.browser.min.js"></script>
  <script src="https://unpkg.com/lucide-react@latest/dist/umd/lucide-react.js"></script>
  <style>
    @keyframes fadeIn { from { opacity: 0; transform: translateY(8px); } to { opacity: 1; transform: translateY(0); } }
    .animate-fade-in { animation: fadeIn 0.3s ease-out; }
    @keyframes fadeInUp { from { opacity: 0; transform: translateY(16px); } to { opacity: 1; transform: translateY(0); } }
    .animate-fade-in-up { animation: fadeInUp 0.4s ease-out; }
    @keyframes toastIn { from { opacity: 0; transform: translateX(100%); } to { opacity: 1; transform: translateX(0); } }
    .animate-toast { animation: toastIn 0.3s ease-out; }
  </style>
</head>
<body class="bg-stone-50">
  <div id="root"></div>
  <script type="text/babel">
    // ★ 元件程式碼放這裡
    const App = () => <div>載入中...</div>;
    const root = ReactDOM.createRoot(document.getElementById('root'));
    root.render(<App />);
  </script>
</body>
</html>
```

**Step 3：瀏覽器確認骨架正常**

打開 http://localhost:3000，確認畫面顯示「載入中...」。

**Step 4：第一次 commit**

```bash
git add index.html docs/
git commit -m "chore: init project with HTML skeleton and design docs"
```

---

### Task 2：移植核心元件（狀態管理 + 工具函式）

**Files:**
- Modify: `index.html`（在 `<script type="text/babel">` 內）

**Step 1：移植所有 useState / useEffect / useRef 狀態宣告**

將原元件中的狀態直接貼入，注意 Lucide 圖示需改為 `LucideReact.Star` 等命名空間格式：

```jsx
const {
  Star, Heart, AlertTriangle, FileText, Check, PenTool, Edit3,
  ChevronRight, ChevronDown, ChevronUp, Sparkles, Copy, RefreshCw,
  User, School, Plus, X, AlertCircle, Save, Trash2, List,
  Mic, MicOff, History, Clock, Upload, Settings, Eye, EyeOff
} = LucideReact;
```

**Step 2：移植所有工具函式（cleanTerm、extractAssessmentItems、analyzeSymbols 等）**

修正 `extractAssessmentItems` 的已知問題：
- 空行造成的解析錯誤：加入 `if (!line || line.length < 2) continue;`
- 確保 `analyzeSymbols` 只在 `handleContentChange` 呼叫，不在 `useEffect` 重複呼叫

**Step 3：移植 localStorage 讀寫，加強 try/catch**

```js
const safeLocalStorageGet = (key, fallback) => {
  try { return JSON.parse(localStorage.getItem(key)) ?? fallback; }
  catch { return fallback; }
};
const safeLocalStorageSet = (key, value) => {
  try { localStorage.setItem(key, JSON.stringify(value)); }
  catch (e) { console.warn('localStorage 寫入失敗', e); }
};
```

**Step 4：瀏覽器確認無 console 錯誤**

打開 http://localhost:3000，F12 確認 console 無紅色錯誤。

**Step 5：Commit**

```bash
git add index.html
git commit -m "feat: port core state and utility functions to single HTML"
```

---

### Task 3：Toast 通知系統

**Files:**
- Modify: `index.html`

**Step 1：建立 Toast 元件與 useToast hook**

```jsx
const useToast = () => {
  const [toasts, setToasts] = React.useState([]);
  const addToast = (message, type = 'success') => {
    const id = Date.now();
    setToasts(prev => [...prev, { id, message, type }]);
    setTimeout(() => setToasts(prev => prev.filter(t => t.id !== id)), 3000);
  };
  return { toasts, addToast };
};

const ToastContainer = ({ toasts }) => (
  <div className="fixed top-4 right-4 z-[100] space-y-2 pointer-events-none">
    {toasts.map(t => (
      <div key={t.id} className={`animate-toast px-4 py-3 rounded-xl shadow-lg text-white text-sm font-medium flex items-center gap-2 pointer-events-auto ${
        t.type === 'success' ? 'bg-green-500' :
        t.type === 'error'   ? 'bg-red-500' : 'bg-orange-500'
      }`}>
        {t.type === 'success' ? '✓' : t.type === 'error' ? '✕' : 'ℹ'} {t.message}
      </div>
    ))}
  </div>
);
```

**Step 2：將所有 `alert()` 取代為 `addToast()`**

- `navigator.clipboard.writeText` 成功 → `addToast('已複製到剪貼簿')`
- 語音不支援 → `addToast('請使用 Chrome 或 Edge', 'error')`
- Word 解析失敗 → `addToast('Word 解析失敗，請改用 .docx', 'error')`
- 舊版 .doc → `addToast('不支援 .doc，請另存為 .docx', 'error')`

**Step 3：瀏覽器手動測試**

點擊複製按鈕，確認右上角出現綠色 Toast，3 秒後消失。

**Step 4：Commit**

```bash
git add index.html
git commit -m "feat: replace alert() with Toast notification system"
```

---

### Task 4：Gemini API Key 設定面板

**Files:**
- Modify: `index.html`

**Step 1：新增 API Key 相關狀態**

```jsx
const [apiKey, setApiKey] = React.useState(
  () => safeLocalStorageGet('gemini_api_key', '')
);
const [showSettings, setShowSettings] = React.useState(false);
const [showApiKey, setShowApiKey] = React.useState(false);
const isAiMode = apiKey.trim().length > 10;
```

**Step 2：建立 SettingsPanel 元件**

```jsx
const SettingsPanel = () => (
  <div className={`fixed inset-y-0 right-0 w-80 bg-white shadow-2xl transform transition-transform duration-300 z-50 overflow-y-auto border-l border-stone-100 ${showSettings ? 'translate-x-0' : 'translate-x-full'}`}>
    <div className="p-6 space-y-6">
      {/* Header */}
      <div className="flex justify-between items-center">
        <h3 className="text-lg font-bold text-stone-700 flex items-center gap-2">
          <Settings className="w-5 h-5 text-orange-500" /> Gemini AI 設定
        </h3>
        <button onClick={() => setShowSettings(false)}><X className="w-5 h-5 text-stone-400" /></button>
      </div>

      {/* 模式指示器 */}
      <div className={`flex items-center gap-2 px-4 py-3 rounded-xl text-sm font-bold ${
        isAiMode ? 'bg-green-50 text-green-700 border border-green-200'
                 : 'bg-yellow-50 text-yellow-700 border border-yellow-200'
      }`}>
        <span className={`w-2.5 h-2.5 rounded-full ${isAiMode ? 'bg-green-500' : 'bg-yellow-400'}`}></span>
        {isAiMode ? '🟢 AI 模式已啟用' : '🟡 離線模式（使用模板）'}
      </div>

      {/* 申請引導 */}
      <details className="bg-blue-50 rounded-xl border border-blue-100">
        <summary className="cursor-pointer p-4 font-bold text-blue-800 list-none flex items-center justify-between">
          📋 如何申請免費 API Key？ <ChevronDown className="w-4 h-4" />
        </summary>
        <div className="px-4 pb-4 space-y-3 text-sm text-blue-900">
          <div className="flex gap-3"><span className="w-6 h-6 bg-blue-200 rounded-full flex items-center justify-center font-bold text-xs flex-shrink-0">1</span><p>前往 <a href="https://aistudio.google.com" target="_blank" rel="noopener" className="underline font-bold hover:text-blue-600">Google AI Studio</a>（aistudio.google.com）</p></div>
          <div className="flex gap-3"><span className="w-6 h-6 bg-blue-200 rounded-full flex items-center justify-center font-bold text-xs flex-shrink-0">2</span><p>使用 Google 帳號登入</p></div>
          <div className="flex gap-3"><span className="w-6 h-6 bg-blue-200 rounded-full flex items-center justify-center font-bold text-xs flex-shrink-0">3</span><p>左側選單點選「Get API Key」→「Create API Key」</p></div>
          <div className="flex gap-3"><span className="w-6 h-6 bg-blue-200 rounded-full flex items-center justify-center font-bold text-xs flex-shrink-0">4</span><p>複製產生的 Key，貼入下方欄位即可</p></div>
          <div className="mt-2 p-3 bg-white rounded-lg border border-blue-200 text-xs text-blue-700">
            💡 免費額度：Gemini 1.5 Flash 每天 1,500 次請求，完全免費，不需綁定信用卡。
          </div>
        </div>
      </details>

      {/* Key 輸入 */}
      <div>
        <label className="block text-sm font-bold text-stone-600 mb-2">Gemini API Key</label>
        <div className="relative">
          <input
            type={showApiKey ? 'text' : 'password'}
            value={apiKey}
            onChange={(e) => {
              setApiKey(e.target.value);
              safeLocalStorageSet('gemini_api_key', e.target.value);
            }}
            placeholder="請貼上您的 API Key..."
            className="w-full p-3 pr-10 border border-stone-200 rounded-xl focus:ring-2 focus:ring-orange-400 outline-none text-sm font-mono"
          />
          <button onClick={() => setShowApiKey(p => !p)} className="absolute right-3 top-1/2 -translate-y-1/2 text-stone-400 hover:text-stone-600">
            {showApiKey ? <EyeOff className="w-4 h-4" /> : <Eye className="w-4 h-4" />}
          </button>
        </div>
        {apiKey && (
          <button onClick={() => { setApiKey(''); safeLocalStorageSet('gemini_api_key', ''); }} className="mt-2 text-xs text-red-400 hover:text-red-600">清除 Key</button>
        )}
      </div>
    </div>
  </div>
);
```

**Step 3：在 Header 加入設定按鈕與模式 badge**

```jsx
{/* Header 右上角按鈕區 */}
<div className="absolute top-6 right-6 flex gap-2 items-center">
  <span className={`hidden md:flex items-center gap-1 text-xs px-2 py-1 rounded-full font-bold ${
    isAiMode ? 'bg-green-100 text-green-700' : 'bg-yellow-100 text-yellow-700'
  }`}>
    <span className={`w-1.5 h-1.5 rounded-full ${isAiMode ? 'bg-green-500' : 'bg-yellow-400'}`}></span>
    {isAiMode ? 'AI 模式' : '離線模式'}
  </span>
  <button onClick={() => setShowSettings(true)} className="flex items-center gap-1 px-3 py-2 bg-white hover:bg-orange-50 text-stone-500 hover:text-orange-600 rounded-lg text-sm border border-stone-200">
    <Settings className="w-4 h-4" /><span className="hidden md:inline">設定</span>
  </button>
  {/* ... 原有的紀錄、下一位學生按鈕 ... */}
</div>
```

**Step 4：瀏覽器測試**

點設定 → 展開申請引導 → 輸入假 Key → 確認 badge 從黃變綠。

**Step 5：Commit**

```bash
git add index.html
git commit -m "feat: add Gemini API key settings panel with onboarding guide"
```

---

### Task 5：Gemini API 呼叫與降級邏輯

**Files:**
- Modify: `index.html`

**Step 1：建立 buildGeminiPrompt 函式**

```js
const buildGeminiPrompt = ({ childInfo, analysis, parsedItems, selectedTags, learningAreaInfo, anecdote }) => {
  const name = childInfo.name.length <= 2 ? childInfo.name : childInfo.name.slice(-2);
  const heShe = childInfo.gender === 'female' ? '她' : '他';
  const grade = childInfo.grade;
  const isYounger = ['幼幼班', '小班'].includes(grade);

  return `你是一位富有愛心且文筆優美的幼兒園老師，請依據以下資料，為家長撰寫一段約 400-500 字的繁體中文總結性評語。

## 孩子資料
- 姓名（稱呼）：${name}
- 代名詞：${heShe}
- 年段：${grade}（${isYounger ? '較小年齡，描述以生活自理、感官探索為主' : '大班/中班，可強調邏輯、合作、創意'}）

## 評量結果
- ☆ 一級棒項目（${analysis.star} 項）：${parsedItems.stars.filter(i=>i.trim()).join('、') || '無'}
- ○ 很不錯項目（${analysis.circle} 項）：${parsedItems.circles.filter(i=>i.trim()).join('、') || '無'}
- △ 加加油項目（${analysis.triangle} 項）：${parsedItems.triangles.filter(i=>i.trim()).join('、') || '無'}

## 特質與能力
- 優勢特質：${[...selectedTags.strengthTraits, ...selectedTags.strengthAbilities].join('、') || '未填寫'}
- 需引導特質：${[...selectedTags.weaknessTraits, ...selectedTags.weaknessAbilities].join('、') || '未填寫'}

## 學習區觀察
- 常去：${learningAreaInfo.often || '未填寫'}，操作項目：${learningAreaInfo.oftenItems || '未填寫'}
- 較少去：${learningAreaInfo.less || '未填寫'}

## 印象深刻的小故事
${anecdote || '（無）'}

## 撰寫要求
1. 語氣溫暖、正向，充滿對孩子的愛與期待
2. 段落分明：開場 → 能力分析 → 特質描述 → 小故事（若有）→ 給家長的建議 → 溫馨結尾
3. 對於△項目，以「需要更多時間與陪伴」的正向框架描述，並給予具體的家庭引導建議
4. 全文使用繁體中文，語句流暢自然
5. 不要使用 Markdown 格式，輸出純文字段落`;
};
```

**Step 2：修改 generateComment 函式，加入 AI 路徑**

```js
const generateComment = async () => {
  setIsGenerating(true);

  if (isAiMode) {
    try {
      const prompt = buildGeminiPrompt({ childInfo, analysis, parsedItems, selectedTags, learningAreaInfo, anecdote });
      const res = await fetch(
        `https://generativelanguage.googleapis.com/v1beta/models/gemini-1.5-flash:generateContent?key=${apiKey}`,
        {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify({
            contents: [{ parts: [{ text: prompt }] }],
            generationConfig: { temperature: 0.85, maxOutputTokens: 1024 }
          })
        }
      );
      if (!res.ok) throw new Error(`API 錯誤 ${res.status}`);
      const data = await res.json();
      const text = data.candidates?.[0]?.content?.parts?.[0]?.text;
      if (!text) throw new Error('回應格式異常');
      setGeneratedComment(text);
      saveToHistory(text);
      setIsGenerating(false);
      setStep(5);
      return;
    } catch (err) {
      addToast(`AI 生成失敗（${err.message}），改用離線模板`, 'error');
    }
  }

  // 離線模板降級（原有邏輯）
  setTimeout(() => {
    const comment = generateTemplateComment(); // 抽出原模板邏輯
    setGeneratedComment(comment);
    saveToHistory(comment);
    setIsGenerating(false);
    setStep(5);
  }, 1200);
};
```

**Step 3：生成按鈕加入 AI/離線模式提示**

```jsx
<button onClick={generateComment} disabled={isGenerating} className="...">
  {isGenerating
    ? <><RefreshCw className="w-4 h-4 animate-spin" /> {isAiMode ? 'AI 思考中...' : '運算愛與溫度中...'}</>
    : <><Sparkles className="w-4 h-4" /> {isAiMode ? '用 AI 生成評語' : '生成評語（離線模板）'}</>
  }
</button>
```

**Step 4：測試 AI 模式**

1. 設定面板輸入有效 Gemini API Key
2. 填寫所有欄位後按生成
3. 確認回傳繁體中文評語，無 console 錯誤

**Step 5：測試降級模式**

1. 設定面板輸入錯誤 Key
2. 按生成，確認出現 Toast 錯誤訊息並自動用模板生成

**Step 6：Commit**

```bash
git add index.html
git commit -m "feat: integrate Gemini API with offline template fallback"
```

---

### Task 6：移植全部 UI 步驟並優化 UX

**Files:**
- Modify: `index.html`

**Step 1：移植 Step 1–5 所有 JSX，確認五步驟流程正常**

重點修正：
- 步驟 1「下一步」按鈕的 disabled 條件加入提示文字：
```jsx
{(!childInfo.name || !childInfo.grade || !childInfo.gender || !assessmentContent) && (
  <p className="text-xs text-red-400 mt-2 text-right">
    請填寫：{[!childInfo.name && '姓名', !childInfo.grade && '年段', !childInfo.gender && '性別', !assessmentContent && '評量內容'].filter(Boolean).join('、')}
  </p>
)}
```

**Step 2：移植 HistoryPanel、renderTagSection 等子元件**

**Step 3：手機版 Header 按鈕確認 icon-only 顯示正常**

縮小視窗至 375px，確認按鈕文字隱藏、icon 保留。

**Step 4：Commit**

```bash
git add index.html
git commit -m "feat: complete all 5-step UI with UX improvements"
```

---

### Task 7：建立 GitHub Repo 並部署 GitHub Pages

**Step 1：建立 GitHub 公開倉庫**

```bash
gh repo create brightspot-kids-evaluator \
  --public \
  --description "幼兒園總結性評語 AI 生成器 | 支援 Gemini AI 與離線模板" \
  --source . \
  --remote origin \
  --push
```

**Step 2：確認推送成功**

```bash
git log --oneline -5
gh repo view --web
```

**Step 3：啟用 GitHub Pages**

```bash
gh api repos/rainbowbox-bit/brightspot-kids-evaluator/pages \
  --method POST \
  --field source='{"branch":"main","path":"/"}' \
  2>&1 || echo "Pages 可能已啟用或需手動設定"
```

若 CLI 無法啟用，手動步驟：
1. 開啟 `https://github.com/rainbowbox-bit/brightspot-kids-evaluator/settings/pages`
2. Source 選「Deploy from a branch」→ Branch: `main` / `/ (root)`
3. 點 Save

**Step 4：等待部署並確認網址**

約 1-2 分鐘後，造訪：
`https://rainbowbox-bit.github.io/brightspot-kids-evaluator/`

**Step 5：最終 commit（加入 README）**

```bash
cat > README.md << 'EOF'
# 看見孩子身上的星星 — 評語生成器

幼教老師總結性評語 AI 生成工具。支援 Gemini AI 與離線模板雙模式。

🌐 **線上使用**：https://rainbowbox-bit.github.io/brightspot-kids-evaluator/

## 功能
- 上傳/貼上評量內容（支援 .docx、.txt）
- Gemini AI 生成自然語言評語（需自備免費 API Key）
- 離線模板模式（無需 API Key）
- 歷史紀錄、語音輸入、自訂標籤庫
EOF
git add README.md
git commit -m "docs: add README with usage instructions"
git push
```
