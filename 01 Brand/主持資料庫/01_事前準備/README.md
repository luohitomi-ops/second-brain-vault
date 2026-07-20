---
type: hub
area: brand
parent: "[[主持資料庫/README|主持資料庫]]"
status: active
updated: 2026-07-10
---

# 01_事前準備

> 主持前拿到的稿子、產品資料、注意事項、自做功課 → 潤稿/備忘錄用。
> 客戶/合作廠商給的產品資料（如 ploom）也放這裡，不算「我自己上的課」（那類放 [[03_資料庫/README|03_資料庫]]）。

## 📅 即將開始（2026-07-18新增：檔名日期 ≥ 今天才顯示在這裡）

```dataviewjs
const files = dv.pages('"01 Brand/主持資料庫/01_事前準備"').where(p => p.file.name !== "README").array();
const todayStr = dv.date("today").toFormat("yyyyMMdd");

function extractDate(name) {
  const m = name.match(/_(\d{8})(?:$|\.)/);
  return m ? m[1] : null;
}

const upcoming = [];
const expired = [];
for (const f of files) {
  const d = extractDate(f.file.name);
  if (!d) { upcoming.push(f); continue; } // 抓不到日期的一律當即將開始，不要漏掉
  if (d >= todayStr) upcoming.push(f); else expired.push(f);
}

if (upcoming.length === 0) {
  dv.paragraph("目前沒有即將開始的主持稿/流程備忘。");
} else {
  dv.table(["檔案", "類型"], upcoming.map(f => [f.file.link, f.type || "-"]));
}

if (expired.length > 0) {
  dv.paragraph(`⚠️ **${expired.length} 篇已過期待歸檔**：` + expired.map(f => f.file.link).join('、') + `\n\n這幾篇活動應該已經結束，跟我說「幫我歸檔主持稿」。**長期合作客戶**（有列在 [[主持與直播執行庫]] 長期品牌日誌表的，如滿貫大亨/傑太ploom）的腳本是下次還會複用的操作範本，會移去 03_資料庫保存不刪；**單次活動**才會蒸餾進 [[主持知識備忘]] 後刪原稿。`);
}
```

## 連結
- 所屬：[[主持資料庫/README|主持資料庫]]
- 已歸檔知識：[[主持知識備忘]]
