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

## 🔗 寫入 💎 數據檔的路徑規則（2026-07-14 新增）
`content_plan.md` / `history.md` / `style_profile.md` / `affiliate_config.json` 在 skill 資料夾裡是 **symlink**，真身在 `C:\tia\vault\01 Brand\社群數據\`（同步用，故意設計成兩邊看到同一份，見 [[project_second_brain]]）。

**任何 Claude session 要編輯這四個檔案，一律直接用真身路徑（`C:\tia\vault\01 Brand\社群數據\...`），不要用 skill 資料夾的路徑去寫**——Edit 工具會拒絕透過 symlink 寫入（`Refusing to write through symlink`），這是工具內建的安全機制，不是可以用 `settings.json` 權限規則繞過的東西。若某個 session 只知道 skill 資料夾路徑撞到這個錯誤，直接改叫它讀真身路徑重試即可，不用調整任何設定。
- 讀取沒有這個限制，symlink 讀寫都正常，只有「寫入」要注意
- 不要把 symlink 改回實體檔案或 hardlink——會重新引發兩邊資料不同步的問題（hardlink 教訓見 [[feedback_symlink_not_hardlink]]，實體檔案會讓 Obsidian 儀表板跟 Discord bot/Claude session 看到不同版本）

## ⚠️ 2026-07-20 發現：generate_and_publish.md / learn_style.md / SKILL.md 已經是「混血檔」，更新時不能整檔覆蓋
之前 SOP 假設 🔧 檔案是「純追蹤上游」，但這幾週的對話已經直接在這三個檔案加了大量本地客製，跟上游版本分岔了：
- **`references/generate_and_publish.md`**：本地版本已經比上游豐富非常多（公式自我檢核步驟、內容變現/聯盟連結檢查、微調安全區、文案存檔步驟、發文靈感池 Step A/B 系統整套），上游版本其實還是原始簡化模板。**這個檔案之後更新一律不能整檔覆蓋**，要先 diff 確認上游有沒有本地沒有的真新內容才手動merge進去。
- **`references/learn_style.md`**：本地版本比上游多兩段「2026-07-17 FB DOM選擇器失效」的實測踩坑記錄，上游沒改過這個檔案。**同樣不能整檔覆蓋**。
- **`SKILL.md`**：混合情況——大部分內容跟著上游走，但速查表最後多一行本地專屬的「聯盟連結留言判斷（需另裝 claude-skill-content-commerce）」。更新時用上游版本當底，**手動把這一行局部合併回去**，不能整檔覆蓋。
- **這次更新已經正確處理**：用上游當底的 8 個乾淨檔案（`F19_DEPLOYMENT_KIT.md`＋`case_studies.md`/`evaluation.md`/`facebook.md`/`formulas.md`/`instagram.md`/`rules.md`/`threads.md`）直接整檔覆蓋；`SKILL.md` 用上游版本+手動補回本地那一行；`generate_and_publish.md`/`learn_style.md` 完全不動（上游這次沒有新內容可合併）
- **之後每次更新的第一步**：先對全部 🔧 檔案做 diff，看diff內容像不像「本地多出來的東西」（含中文操作紀錄口吻、日期是最近對話發生的、參照 vault 專屬檔案如發文靈感池.md/文案存檔.md）—— 像的話代表那個檔案已混血，要手動 merge 不能整檔覆蓋；不像的話（單純標點/數字/案例更新）才安全整檔覆蓋

## 更新紀錄
- 2026-07-20：同步到最新 main（up to commit a402683 "v1.1.2 - Case 39 plateau + partner E + borrowed-vs-own artifact split"）。新增 F19_DEPLOYMENT_KIT.md 部署包+多筆新案例(Case 39-41)+R34全形標點鐵律+R19新細節(第三方artifact帶轉發但自製artifact轉粉絲高7x)。已備份4個數據檔到 `_social-post-backup/2026-07-20_XXXX`，數據完好。**過程中發現上面那則「混血檔」問題並正確處理**，是這次更新最重要的收穫。
- 2026-06-28：同步到 v1.0.2（F1-F27、R1-R35）。差異主要是例句標點細修（R34 真實 voice 防 AI 腔）。已備份 4 個數據檔到 `_social-post-backup/2026-06-28`，數據完好。

## 連結
- 引擎節點：[[社群自動發文]]
