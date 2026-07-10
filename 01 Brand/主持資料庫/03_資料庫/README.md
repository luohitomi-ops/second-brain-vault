---
type: hub
area: brand
status: active
updated: 2026-07-10
---

# 主持課程資料庫

> **我自己上過的主持相關課程**筆記（我本人去學主持技巧的課）。捕獲頻道貼上課筆記 → `/整理` 自動判斷分類存檔到這裡。

## ⚠️ 這裡只放「我自己上的主持課」
- **客戶/合作廠商給的產品資料**（如 ploom 產品教育內容）→ **不放這裡**，改放 `01_事前準備/`。因為那是廠商給的、為了主持他們活動用的資料，不是我學到的技能（2026-07-10 釐清）
- **非主持的跨領域課程**（演員課/NLP/表達/溝通）→ 放 [[課程筆記庫/README|06 Lifestyle/課程筆記庫]]

## 📂 目前檔案（自動列出，新增檔案會自動出現）

```dataviewjs
const files = dv.pages('"01 Brand/主持資料庫/03_資料庫"').where(p => p.file.name !== "README").sort(p => p.file.mtime, 'desc').array();
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

## 整理方式建議
每份課程筆記整理完，用以下格式存成獨立檔案：
```
## 課程名稱 / 講師 / 上課日期
### 核心觀念
### 可直接套用的技巧
### 我的實戰驗證（哪一場用過、效果如何）
```
