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

## 📂 目前檔案（自動列出，新增檔案會自動出現）

```dataviewjs
const files = dv.pages('"01 Brand/主持資料庫/01_事前準備"').where(p => p.file.name !== "README").sort(p => p.file.mtime, 'desc').array();
if (files.length === 0) {
  dv.paragraph("目前沒有檔案。");
} else {
  dv.table(["檔案", "類型", "建立日期"], files.map(f => [
    f.file.link,
    f.type || "-",
    f.created || f.file.cday.toFormat("yyyy-MM-dd")
  ]));
}
```

## 連結
- 所屬：[[主持資料庫/README|主持資料庫]]
