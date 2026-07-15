---
description: 手動立即重跑一次 Daily Brief（跟每天02:00自動版同一套邏輯，0 Claude額度）
---

幫檸子手動重新生成一份今日的熱門話題摘要（Daily Brief）。

> 2026-07-11 起改為呼叫跟 Discord bot 自動排程**同一支腳本**，不再用 WebSearch 自己重寫一份不同格式的版本，避免兩套邏輯互相覆蓋。
> **2026-07-15 起預設用免費版**（`news-brief-free.mjs`，Google News RSS + 一般 Gemini 摘要，不掛 google_search grounding，0成本）——原本的 grounding 版（`gemini-brief.mjs`）因免費額度長期見底（連續5天無法生成，見系統藍圖已知問題）暫停排程，範圍也縮小成只收「股票新聞」+「美妝流行熱門話題」兩類（原本還有影片迷因/泛用熱門話題，已拿掉）。之後若使用者決定開 Google Cloud 計費，再切回 `gemini-brief.mjs`。

執行方式：

1. 用 Bash 執行：`cd "C:\tia\brain-bot" && node news-brief-free.mjs`
2. 讀腳本輸出的結果訊息（股票新聞/美妝流行話題各幾則），回報給使用者。
3. 這份 Brief 會覆蓋掉今天已存在的 `01 Brand/Brief YYYY-MM-DD.md`（如果今天自動版已經跑過），適合用在：想要更即時的一份重新抓。
4. 不要實際發文，只生成素材。它會自動出現在 [[社群自動發文]] 的靈感候選庫表格（因為帶 #發文素材 tag）。
