---
description: 手動立即重跑一次 Daily Brief（跟每天02:00自動版同一套邏輯，0 Claude額度）
---

幫檸子手動重新生成一份今日的熱門話題摘要（Daily Brief）。

> 2026-07-11 起改為呼叫跟 Discord bot 每天 02:00 自動排程**同一支腳本**（`gemini-brief.mjs`，Gemini API + Google 搜尋 grounding，強制真實來源URL，0 Claude額度），不再用 WebSearch 自己重寫一份不同格式的版本——避免出現兩套邏輯互相覆蓋、frontmatter 欄位對不上儀表板的問題。

執行方式：

1. 用 Bash 執行：`cd "C:\tia\brain-bot" && node gemini-brief.mjs`
2. 讀腳本輸出的結果訊息（幾則影片梗/股市新聞/熱門話題），回報給使用者。
3. 這份 Brief 會覆蓋掉今天已存在的 `01 Brand/Brief YYYY-MM-DD.md`（如果今天自動版已經跑過），適合用在：自動排程失敗、或想要更即時的一份重新抓。
4. 不要實際發文，只生成素材。它會自動出現在 [[社群自動發文]] 的靈感候選庫表格（因為帶 #發文素材 tag）。
