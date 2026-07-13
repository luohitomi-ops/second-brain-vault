---
type: hub
status: active
updated: 2026-07-08
---

# Skills 方法庫

> 跨中樞的通用 SOP / AI 提示詞 / 工作流程索引。**只做連結，不搬移原檔**——每份方法仍留在原本所屬的中樞（品牌/投資/生活），這裡只是快速找到入口，避免原本引用它的系統（DC bot、dataviewjs）路徑跟著斷掉。

## 01 AI 提示詞 / 顧問召喚
- [[顧問技能清單]]（05 Reference）— 各領域顧問人格召喚指令總覽
- 品牌顧問：`/greg-mckeown-perspective`、`/chris-do-perspective`、`/dan-koe-perspective`、`/alex-hormozi-perspective` 等（見 [[品牌中樞]] ✦ 顧問群）

## 02 工作流程（`.claude/commands/` 完整索引，2026-07-08 建立）

> 系統藍圖 Phase 4「日常工作流節點化」的核心產出：14 個自訂指令散落各處沒有集中清單，這裡統一列出**觸發時機／讀取什麼／產出到哪**，之後新增指令記得回來補一行。

### ☀️ 每日例行
| 指令 | 觸發時機 | 讀取 | 所需 skill / 顧問 | 產出位置 |
|------|---------|------|------------------|---------|
| `/每日校準` | 每天開始前，掃全局抓焦點 | 04 Projects status、三大中樞當前重點、CLAUDE.md、content_plan.md、history.md、自律計劃表.md、生活優先序、today-calendar.json | 無（純 vault 掃描，不召喚顧問人格）| 純報告，不寫檔 |
| `/發文日常` | 要規劃/生成今日發文 | content_plan.md、history.md、每日碎片化靈感隨記.md、formulas.md、style_profile.md | `social-post` skill 核心邏輯（F1-F29 公式 + 語氣模仿）| 草稿（需另外「確認」才會實際發佈）|
| `/整理` `/整理Inbox` | 碎片靈感需要分類分流 | 每日碎片化靈感隨記.md | 無 | 各中樞對應分支（投資知識庫/愛美與醫美心得/想學習的技能清單等）|

### 🔍 顧問深挖（接續式，不靠舊對話記憶）
| 指令 | 觸發時機 | 讀取 | 所需 skill / 顧問 | 產出位置 |
|------|---------|------|------------------|---------|
| `/品牌深聊 [主題]` | 想深入討論品牌策略 | 品牌中樞深挖佇列 | 動態點名品牌顧問群：`/greg-mckeown-perspective`（精要主義）、`/chris-do-perspective`（定價）、`/dan-koe-perspective`（創作者飛輪）、`/alex-hormozi-perspective`（報價系統）、`/alan-weiss-perspective`（顧問定位）、`/taiwan-creator-perspective`、`/taiwan-shortform-perspective`——依主題挑，不是全召喚 | 品牌中樞.md 當前重點 |
| `/投資深聊 [主題]` | 想深入討論長期投資 | 投資中樞深挖佇列 | 動態點名投資顧問群：`/warren-buffett-perspective`、`/charlie-munger-perspective`、`/peter-lynch-perspective`、`/morgan-housel-perspective`、`/naval-ravikant-perspective`、`/mj-demarco-perspective`、`/taiwan-investor-perspective`、`/mark-minervini-perspective` | 投資中樞.md 當前重點 |
| `/生活深聊 [主題]` | 想深入討論醫美/學習/自律 | 生活自我成長中樞深挖佇列 | 只有「技能」主題會用 `/greg-mckeown-perspective`（精要主義排優先序）；醫美/自律主題不召喚特定 skill，用內建「生活教練」人格 | 生活自我成長中樞.md 當前重點 |

### 🔁 週期性復盤
| 指令 | 觸發時機 | 讀取 | 所需 skill / 顧問 | 產出位置 |
|------|---------|------|------------------|---------|
| `/雙週復盤` | 每 14 天 | history.md（近14天）、content_plan.md | `social-post` skill（evaluation.md 四指標框架）| content_plan.md 新增下一輪排程（確認後才寫入）|
| `/每週復盤` | 每週跨區週檢 | 04 Projects status、三大中樞 | 無 | 純報告；問要不要存 `06 Daily/週報-YYYY-WXX.md` |
| `/爆款優化` | 想重新校準爆款公式 | `social-post/history.md`（symlink 指向 `01 Brand/社群數據/history.md`，同一份資料）| `social-post` skill（evaluation.md + rules.md）| `01 Brand/爆款路線分析.md` |
| `/庫存健檢 [股票]` | 波段持倉需要體檢 | 波段庫存.md + 使用者提供籌碼/技術數據 | 固定召喚兩位：`/taiwan-intraday-signals`（籌碼面）+ `/mark-minervini-perspective`（技術面/倉位紀律）| 波段庫存.md 健檢紀錄 |

### 🏗️ 建置 / 固化
| 指令 | 觸發時機 | 讀取 | 所需 skill / 顧問 | 產出位置 |
|------|---------|------|------------------|---------|
| `/更新中樞 [腦區]` | 討論完想固化結論 | 當前對話 | 無 | 對應「○○中樞.md」+ 該區 CLAUDE.md |
| `/新工作流 [名稱]` | 有新專案要開始 | 使用者輸入 | 無 | `04 Projects/[名稱].md` + 連回對應中樞 |
| `/部落格寫作` | 要寫部落格長文 | 部落格選題庫、formulas.md（長文版） | `social-post` skill（F公式長文版邏輯，跟社群短文分開）| 部落格草稿 |
| `/生成Brief` | 手動想立即重跑一次今日熱門話題 | `gemini-brief.mjs`（Gemini API） | 無（直接呼叫跟自動排程同一支腳本）| `01 Brand/Brief YYYY-MM-DD.md` |

> ✅ **2026-07-11 已統一**：`/生成Brief` 改成直接呼叫 discord-bot 每天 02:00 自動排程用的同一支 `gemini-brief.mjs`（0 Claude 額度），不再用 WebSearch 自己重寫一份格式不同的版本。單一邏輯，不會再有格式不一致的 Brief 檔。

- [[分支用法速查]]（05 Reference）— 各分支節點怎麼用
- [[協作分工與邊界]]（05 Reference）— 人 / Claude / DC bot 分工原則
- [[social-post更新規則]]（05 Reference）— skill 版本更新時的同步 SOP

## 03 寫作方法
- [[部落格選題庫]]（01 Brand）— 選題優先序邏輯
- F1-F29 爆款公式：`C:\Users\USER\.claude\skills\social-post\references\formulas.md`（skill 本體，不搬進 vault，避免脫離版本更新）

## 04 研究方法
- [[爆款路線分析]]（01 Brand）— 演算法數據復盤方法論
- [[投資知識庫]]（02 Invest-LT）
- [[給獺金幣開盤模擬指南]]（03 Invest-Swing）

## 05 整理方法
- `/整理`、`/更新中樞` 指令（.claude/commands）
- `/瘦身`（2026-07-13）— 全庫防肥大：掃會長大的 MD 檔行數，超標的列「活狀態 vs 歷史封存」建議，確認後搬（移動不刪除）。每季或發現超標時跑
- [[重啟測試清單]]（05 Reference）— 系統改動後的驗證 SOP

## 06 概念卡（跨領域心法，2026-07-06 新增）
> 從各中樞實戰經驗蒸餾出的可重複使用心法，獨立成卡，不綁單一領域。
- [[demo大於claim]] — 具體成品展示 > 抽象宣稱
- [[value-prop開場]] — 開頭先講用途，不用懸念式鉤子
- [[90%法則篩選]] — 新項目沒到90分先暫緩，不是延後
- [[前1秒鉤子]] — 短影音開場角色/動作直接入鏡 + 題材普世度
- [[移動停損邏輯]] — 停損跟股價走，三層防線結構
- [[Effortless微動作綁定法]] — 新習慣綁進已有動作，零額外時間

---

## 使用原則
- 新的通用方法（不屬於單一中樞、多處都會用到）→ 直接寫在這裡
- 屬於單一中樞的方法（例如只有品牌會用的顧問提示詞）→ 留在原中樞，這裡只放連結
- 過期或不再使用的方法 → 移到 `05 Reference` 封存，不留在此索引
