---
type: reference
status: active
updated: 2026-06-26
---

# social-post Skill 更新規則（保護個人數據）

> 作者：駱君昊 (Hao)｜Repo：https://github.com/Hao0321/claude-skill-social-post｜MIT
> 用途：作者之後在 GitHub 更新 skill 時，安全合併新功能，**絕不覆蓋檸子的數據**。

## 🔑 核心原則：邏輯可更新、數據要保留

Skill 資料夾 `C:\Users\USER\.claude\skills\social-post\` 內兩類檔案分開處理：

| 🔧 可更新（用作者新版覆蓋）| 💎 絕不可覆蓋（檸子數據）|
|---|---|
| `SKILL.md` | `content_plan.md` |
| `references/`（全部，含 formulas.md F 公式）| `history.md`（戰績表）|
| `social-post/*.example.md`（範本）| `style_profile.md`（語氣檔）|
| | `affiliate_config.json` |

## 📋 更新 SOP（要更新時叫 Claude 執行）

跟 Claude 說：「幫我更新 social-post skill，照更新規則。」Claude 會：

1. **先備份** 💎 四個數據檔到 `C:\tia\vault\05 Reference\_social-post-backup\`（時間戳）
2. 讀 GitHub repo 最新 `SKILL.md` + `references/*`（用 WebFetch 讀 raw 檔）
3. **比對差異**：列出新增/修改的公式、規則、功能，給檸子看
4. 確認後，**只覆蓋 🔧 邏輯檔**，💎 數據檔原封不動
5. 若新版改了數據檔的「格式」（例如 history 欄位變了）→ 不自動改，先告知，由檸子決定怎麼遷移
6. 回報：更新了哪些、新增什麼功能、數據檔確認未動

## ⚠️ 風險提醒
- 永遠不要對整個 skill 資料夾做 `git pull` 或整包覆蓋——會洗掉數據
- 一律走上面 SOP，逐檔處理

## 更新紀錄
- 2026-06-28：同步到 v1.0.2（F1-F27、R1-R35）。差異主要是例句標點細修（R34 真實 voice 防 AI 腔）。已備份 4 個數據檔到 `_social-post-backup/2026-06-28`，數據完好。

## 連結
- 引擎節點：[[社群自動發文]]
