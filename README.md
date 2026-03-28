# 看見孩子身上的星星 — 評語生成器

幼教老師總結性評語 AI 生成工具。支援 Gemini AI 與離線模板雙模式，無需安裝，開啟瀏覽器即可使用。

🌐 **線上使用**：https://rainbowbox-bit.github.io/brightspot-kids-evaluator/

## 功能特色

- 📄 上傳/貼上評量內容（支援 Word .docx、.txt）
- 🤖 Gemini AI 生成自然語言評語（需自備免費 API Key）
- 📝 離線模板模式（無需 API Key，隨時可用）
- ✏️ 解析結果手動微調（☆ ○ △ 三等級）
- 🏷️ 特質與能力標籤庫（可自訂並永久保存）
- 🏫 學習區觀察記錄
- 🎤 語音輸入小故事（Chrome / Edge）
- 📋 歷史紀錄（最近 10 筆，可一鍵複製）
- 💾 自動暫存，切換頁面不怕資料遺失

## 如何取得免費 Gemini API Key

1. 前往 [Google AI Studio](https://aistudio.google.com)
2. 使用 Google 帳號登入
3. 左側選單點選「Get API Key」→「Create API Key」
4. 複製 Key，貼入應用程式右上角「設定」即可

> 💡 建議在 Google AI Studio 建立最新可用的 Gemini API Key；系統會自動嘗試可用模型（2.5/2.0/1.5 Flash）。

## 技術架構

純 HTML 單檔，無需建置步驟：

- React 18（CDN）+ Babel Standalone（JSX 轉譯）
- Tailwind CSS（CDN）
- Lucide React（圖示）
- Mammoth.js（Word 檔解析）
- Gemini API（自動 fallback 可用模型）

---

Designed by 袋鼠老師陪你幼教有愛
