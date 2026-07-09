---
type: home
status: active
updated: 2026-07-02
cssclasses:
  - dashboard-v2
---

```dataviewjs
// ============================================================
// 檸子資料庫 · 主儀表板 v2
// ============================================================

// ── 0. 解除可讀行寬（DOM 操作，不依賴 CSS 載入時序）──────────
setTimeout(() => {
  const view = dv.container.closest('.markdown-source-view');
  if (!view) return;
  ['cm-sizer','cm-content','cm-contentContainer'].forEach(cls => {
    const el = view.querySelector('.' + cls);
    if (el) { el.style.maxWidth = 'none'; el.style.width = '100%'; el.style.paddingInline = '0'; }
  });
}, 80);

// ── 1. 工具函式 ──────────────────────────────────────────────
const R = dv.container; // DataviewJS 根節點
const now = new Date(); // 頂層共用，各卡片都可取用

/** 建立元素的簡寫（支援 children 陣列）*/
function el(parent, tag, opts = {}) {
  const { cls, text, html, attr = {} } = opts;
  const node = parent.createEl(tag, { cls, text });
  if (html) node.innerHTML = html;
  Object.entries(attr).forEach(([k,v]) => node.setAttribute(k, v));
  return node;
}

/** Lucide icon SVG 字串（純 path，最小化）*/
const ICONS = {
  home:      `<svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round"><path d="m3 9 9-7 9 7v11a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2z"/><polyline points="9 22 9 12 15 12 15 22"/></svg>`,
  brand:     `<svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round"><path d="M12 20h9"/><path d="M16.5 3.5a2.12 2.12 0 0 1 3 3L7 19l-4 1 1-4Z"/></svg>`,
  invest:    `<svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round"><polyline points="22 7 13.5 15.5 8.5 10.5 2 17"/><polyline points="16 7 22 7 22 13"/></svg>`,
  swing:     `<svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round"><path d="M3 3v18h18"/><path d="m7 16 4-4 4 4 4-6"/></svg>`,
  life:      `<svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round"><path d="M20.84 4.61a5.5 5.5 0 0 0-7.78 0L12 5.67l-1.06-1.06a5.5 5.5 0 0 0-7.78 7.78l1.06 1.06L12 21.23l7.78-7.78 1.06-1.06a5.5 5.5 0 0 0 0-7.78z"/></svg>`,
  capture:   `<svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round"><path d="M14.5 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V7.5L14.5 2z"/><polyline points="14 2 14 8 20 8"/><line x1="12" y1="18" x2="12" y2="12"/><line x1="9" y1="15" x2="15" y2="15"/></svg>`,
  review:    `<svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round"><rect width="18" height="18" x="3" y="3" rx="2"/><path d="M3 9h18M9 21V9"/></svg>`,
  blueprint: `<svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round"><rect width="18" height="18" x="3" y="3" rx="2" ry="2"/><line x1="3" y1="9" x2="21" y2="9"/><line x1="9" y1="21" x2="9" y2="3"/></svg>`,
  task:      `<svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round"><path d="M9 11l3 3L22 4"/><path d="M21 12v7a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2V5a2 2 0 0 1 2-2h11"/></svg>`,
  earnings:  `<svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round"><line x1="18" y1="20" x2="18" y2="10"/><line x1="12" y1="20" x2="12" y2="4"/><line x1="6" y1="20" x2="6" y2="14"/><line x1="2" y1="20" x2="22" y2="20"/></svg>`,
  chevron:   `<svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="m6 9 6 6 6-6"/></svg>`,
  mindmap:   `<svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round"><path d="M9.5 2a2.5 2.5 0 0 0-2.5 2.5v.5a2.5 2.5 0 0 0-2 2.45v.5a2.5 2.5 0 0 0 0 5v.5a2.5 2.5 0 0 0 2 2.45v.5A2.5 2.5 0 0 0 9.5 19"/><path d="M14.5 2a2.5 2.5 0 0 1 2.5 2.5v.5a2.5 2.5 0 0 1 2 2.45v.5a2.5 2.5 0 0 1 0 5v.5a2.5 2.5 0 0 1-2 2.45v.5a2.5 2.5 0 0 1-2.5 2.5"/><path d="M9.5 2h5M9.5 19h5"/><path d="M6 15a3 3 0 0 1 6 0M12 15a3 3 0 0 1 6 0"/></svg>`,
};

/** 顯示 Toast 提示 */
function showToast(msg, type = 'success') {
  const t = document.body.createEl('div', { cls: `db-toast ${type}`, text: msg });
  setTimeout(() => t.remove(), 2800);
}

/** 開啟 Obsidian 內部頁面 */
function openPage(name) {
  // 1. 先試完整 vault 路徑
  const f = app.vault.getAbstractFileByPath(name + '.md')
         || app.vault.getAbstractFileByPath(name);
  if (f) { app.workspace.getLeaf(false).openFile(f); return; }
  // 2. 路徑含 & 或中文時 getAbstractFileByPath 可能失敗 → 改用 basename 搜尋
  const basename = name.split('/').pop();
  const byName = app.vault.getFiles().find(x => x.basename === basename);
  if (byName) { app.workspace.getLeaf(false).openFile(byName); return; }
  // 3. 最後 fallback
  app.workspace.openLinkText(name, '', false);
}

// ── 2. App Shell ─────────────────────────────────────────────
const shell = R.createEl('div', { cls: 'db-shell' });

// ── 3. Sidebar ───────────────────────────────────────────────
(function buildSidebar() {
  const sb = shell.createEl('div', { cls: 'db-sidebar' });

  // Logo
  const logo = sb.createEl('div', { cls: 'db-sidebar-logo' });
  logo.createEl('div', { cls: 'db-sidebar-logo-icon', text: '🦦' });
  const lt = logo.createEl('div');
  lt.createEl('div', { cls: 'db-sidebar-logo-text', text: '檸子資料庫' });
  lt.createEl('div', { cls: 'db-sidebar-logo-sub', text: '整理生活・經營自我・創造價值' });

  // 核心模組
  el(sb, 'div', { cls: 'db-nav-group-label', text: '核心模組' });
  const coreNav = [
    ['今日主控', 'Home', 'home'],
    ['品牌中樞', '品牌中樞', 'brand'],
    ['投資中樞', '投資中樞', 'invest'],
    ['波段中樞', '波段中樞', 'swing'],
    ['生活成長', '生活自我成長中樞', 'life'],
    ['快速捕獲', '快速捕獲中樞', 'capture'],
  ];
  coreNav.forEach(([label, target, icon]) => {
    const a = sb.createEl('div', { cls: 'db-nav-item' + (target === 'Home' ? ' active' : '') });
    a.innerHTML = `<span class="db-nav-icon">${ICONS[icon]}</span>${label}`;
    a.addEventListener('click', () => openPage(target));
  });

  // 工具
  el(sb, 'div', { cls: 'db-nav-group-label', text: '工具' });
  const toolNav = [
    ['決策心智地圖', '00 Dashboard/我的決策心智地圖', 'mindmap'],
    ['靈感候選庫', '01 Brand/發文靈感池', 'capture'],
    ['股市新聞', '01 Brand/股市新聞', 'earnings'],
    ['Skills 方法庫', '08 Skills 方法庫/README', 'task'],
    ['雙週復盤', '發文戰情', 'review'],
    ['系統藍圖', '系統藍圖', 'blueprint'],
    ['財報頻道', '03 Invest-Swing/財報頻道', 'earnings'],
    ['所有待辦', '', 'task'],
  ];
  toolNav.forEach(([label, target, icon]) => {
    const a = sb.createEl('div', { cls: 'db-nav-item' });
    a.innerHTML = `<span class="db-nav-icon">${ICONS[icon]}</span>${label}`;
    if (target) a.addEventListener('click', () => openPage(target));
    else a.addEventListener('click', () => {
      const sec = document.querySelector('.db-todos-section');
      if (!sec) return;
      const hdr2 = sec.querySelector('.db-card-header');
      const body2 = sec.querySelector('div[style*="display"]');
      if (body2 && body2.style.display === 'none') hdr2?.click();
      sec.scrollIntoView({ behavior: 'smooth' });
    });
  });

  // 資料庫（所有分支內容庫，按區域分組，預設收合避免側欄過長，2026-07-10 新增）
  (function buildDataLibrary() {
    const labelRow = sb.createEl('div', {
      cls: 'db-nav-group-label',
      attr: { style: 'cursor:pointer;user-select:none;display:flex;align-items:center;justify-content:space-between;' },
    });
    labelRow.createEl('span', { text: '資料庫' });
    const chev = labelRow.createEl('span', { attr: { style: 'font-size:10px;' }, text: '▸' });

    const wrap = sb.createEl('div', { attr: { style: 'display:none;' } });

    const groups = [
      ['品牌', 'brand', [
        ['主持資料庫', '01 Brand/主持資料庫/README'],
        ['香氛工作室', '01 Brand/香氛工作室/README'],
        ['COS經營', 'COS經營'],
        ['接案執行庫', '接案執行庫'],
        ['部落格進度', '部落格進度'],
        ['每日熱門話題摘要', '每日熱門話題摘要'],
        ['撥撥獺獺IP', '撥撥獺獺IP'],
        ['社群自動發文', '社群自動發文'],
      ]],
      ['投資', 'invest', [
        ['投資知識庫', '投資知識庫'],
      ]],
      ['波段', 'swing', [
        ['波段庫存', '波段庫存'],
        ['個股總覽', '個股總覽'],
        ['觀察名單', '觀察名單'],
        ['給獺金幣程式', '給獺金幣程式'],
      ]],
      ['生活', 'life', [
        ['愛美與醫美心得', '愛美與醫美心得'],
        ['想學習的技能清單', '想學習的技能清單'],
        ['課程筆記庫', '06 Lifestyle & Self-Growth/課程筆記庫/README'],
        ['自律計劃表', '自律計劃表'],
      ]],
      ['快速捕獲', 'capture', [
        ['每日碎片隨記', '07 Quick Capture & Hub/每日碎片化靈感隨記'],
        ['靈感庫', '靈感庫'],
      ]],
    ];

    groups.forEach(([areaLabel, icon, items]) => {
      el(wrap, 'div', { attr: { style: 'font-size:10px;color:var(--db-text-muted);text-transform:uppercase;letter-spacing:.06em;padding:8px 12px 2px;' }, text: areaLabel });
      items.forEach(([label, target]) => {
        const a = wrap.createEl('div', { cls: 'db-nav-item', attr: { style: 'padding-left:24px;font-size:12px;' } });
        a.innerHTML = `<span class="db-nav-icon">${ICONS[icon]}</span>${label}`;
        a.addEventListener('click', () => openPage(target));
      });
    });

    labelRow.addEventListener('click', () => {
      const open = wrap.style.display !== 'none';
      wrap.style.display = open ? 'none' : 'block';
      chev.textContent = open ? '▸' : '▾';
    });
  })();

  // Footer
  const footer = sb.createEl('div', { cls: 'db-sidebar-footer' });
  el(footer, 'div', { cls: 'db-sidebar-footer-label', text: '外觀設定' });
  el(footer, 'div', { cls: 'db-sidebar-logo-sub', text: `v2 · ${now.getFullYear()}/${String(now.getMonth()+1).padStart(2,'0')}/${String(now.getDate()).padStart(2,'0')}` });
})();

// ── 4. Main 區域 ─────────────────────────────────────────────
const main = shell.createEl('div', { cls: 'db-main' });

// 「每週才看」內容暫存容器（尚未掛進 main，等所有每天必看區塊都建完後，統一收合掛到底部）
const weeklyBody = document.createElement('div');

// 通用收合式卡片工廠（同 財報頻道.md／個股總覽.md 風格）
function makeCollapsible(parent, title, defaultOpen = false) {
  const card = parent.createEl('div', { cls: 'db-card db-card-enter', attr: { style: 'margin-bottom: var(--db-gap);' } });
  const hdr  = card.createEl('div', { cls: 'db-card-header', attr: { style: 'cursor:pointer;user-select:none;' } });
  el(hdr, 'div', { cls: 'db-card-title', text: title });
  const chev = el(hdr, 'div', { attr: { style: 'font-size:13px;color:var(--db-text-muted);' }, text: defaultOpen ? '▾' : '▸' });
  const body = card.createEl('div', { attr: { style: `display:${defaultOpen ? 'block' : 'none'};` } });
  hdr.addEventListener('click', () => {
    const open = body.style.display !== 'none';
    body.style.display = open ? 'none' : 'block';
    chev.textContent   = open ? '▸' : '▾';
  });
  return body;
}

// ── 4a. Header ───────────────────────────────────────────────
(function buildHeader() {
  const header = main.createEl('div', { cls: 'db-card db-card-enter', attr: { style: 'margin-bottom: var(--db-gap); padding: 22px 28px;' } });

  const row = header.createEl('div', { attr: { style: 'display:flex; align-items:flex-start; justify-content:space-between; gap:20px;' } });

  // 左側：標題 + 日期 + 語錄
  const left = row.createEl('div');
  el(left, 'div', { attr: { style: 'font-size:11px; color:var(--db-text-muted); text-transform:uppercase; letter-spacing:.1em; margin-bottom:8px;' }, text: 'Home 儀表板' });

  const weekDays = ['日','一','二','三','四','五','六'];
  const dateStr = `${now.getFullYear()} 年 ${now.getMonth()+1} 月 ${now.getDate()} 日　星期${weekDays[now.getDay()]}`;

  el(left, 'h1', { attr: { style: 'font-size:26px; font-weight:700; color:var(--db-text-primary); margin:0 0 6px; line-height:1.2;' }, text: '早安，檸子 🐾' });
  el(left, 'div', { attr: { style: 'font-size:13px; color:var(--db-text-secondary); margin-bottom:12px;' }, text: dateStr });

  const quote = el(left, 'div', { attr: { style: 'font-size:13px; color:var(--db-text-muted); font-style:italic; border-left:2px solid var(--db-purple-dim); padding-left:10px;' } });
  const quotes = [
    '今天也要當一隻努力的水獺！',
    '一點一滴，積累成自己想成為的樣子。',
    '專注、執行、成長、豐盛。',
    '慢慢來，比較快。',
  ];
  quote.textContent = quotes[now.getDate() % quotes.length];

  // 右側：水獺插畫
  const otterWrap = row.createEl('div', { attr: { style: 'flex-shrink:0; width:100px; height:100px; opacity:.85;' } });
  otterWrap.innerHTML = `<svg viewBox="0 0 100 100" xmlns="http://www.w3.org/2000/svg" fill="none" stroke="#A89EC0" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round">
  <!-- 身體 -->
  <ellipse cx="50" cy="64" rx="22" ry="14" stroke="#8B7BAD"/>
  <!-- 頭 -->
  <circle cx="50" cy="38" r="15" stroke="#8B7BAD"/>
  <!-- 耳朵 -->
  <path d="M37 26 Q33 20 36 17"/>
  <path d="M63 26 Q67 20 64 17"/>
  <!-- 眼睛 -->
  <circle cx="44" cy="36" r="2.2" fill="#A89EC0" stroke="none"/>
  <circle cx="56" cy="36" r="2.2" fill="#A89EC0" stroke="none"/>
  <!-- 鼻子 -->
  <ellipse cx="50" cy="41.5" rx="3.5" ry="2.2" fill="#8B7BAD" stroke="none"/>
  <!-- 嘴巴 -->
  <path d="M46 44 Q50 47 54 44"/>
  <!-- 鬍鬚 -->
  <line x1="35" y1="40" x2="43" y2="41"/><line x1="35" y1="43" x2="43" y2="43"/>
  <line x1="57" y1="41" x2="65" y2="40"/><line x1="57" y1="43" x2="65" y2="43"/>
  <!-- 爪子 -->
  <path d="M30 66 Q26 70 28 74"/>
  <path d="M70 66 Q74 70 72 74"/>
  <!-- 肚皮（淡色）-->
  <ellipse cx="50" cy="62" rx="13" ry="8" stroke="#7A7490" stroke-dasharray="2 2"/>
  <!-- 水波 -->
  <path d="M20 78 Q30 74 40 78 Q50 82 60 78 Q70 74 80 78" stroke="#7AA8A8" stroke-width="1.4"/>
  <path d="M25 84 Q37 80 50 84 Q63 88 75 84" stroke="#7AA8A8" stroke-width="1" opacity=".5"/>
</svg>`;
})();

// ── 4b. 今日概覽（頂層 await 取資料，再同步建 DOM）─────────────

// 0. 行事曆（PowerShell 每天 7:55 匯出，含未來 60 天）
let _calEvents = [];     // 全部事件（月曆用）
let _calTodayEvs = [];   // 僅今天（stat card / 今日待辦用）
try {
  const _calf = app.vault.getAbstractFileByPath('00 Dashboard/today-calendar.json');
  if (_calf) {
    const _calRaw = await app.vault.read(_calf);
    _calEvents = JSON.parse(_calRaw || '[]');
    const _calToday = now.toISOString().slice(0,10);
    _calTodayEvs = _calEvents.filter(e => e.start && e.start.startsWith(_calToday));
  }
} catch(e) {}

// 1. 全庫未完成任務數（所有待辦區塊用）
const pendingCount = dv.pages().file.tasks.where(t => !t.completed).length;

// 今日固定任務（每天 8:00 重置：8點前沿用昨日）
const _effectiveDate = (() => {
  const d = new Date(now);
  if (d.getHours() < 8) d.setDate(d.getDate() - 1);
  return `${d.getFullYear()}-${String(d.getMonth()+1).padStart(2,'0')}-${String(d.getDate()).padStart(2,'0')}`;
})();
const _dailyKey = `daily-tasks-${_effectiveDate}`;
const _dailyDefs = ['檢視發文計畫', '排程發文', '每日數據回填', '幫貓刷牙'];
let _dailyState = {};
try { _dailyState = JSON.parse(localStorage.getItem(_dailyKey) || '{}'); } catch(e) {}
const dailyDone  = _dailyDefs.filter(t => _dailyState[t]).length;
const dailyTotal = _dailyDefs.length;
const dailyPct   = Math.round(dailyDone / dailyTotal * 100) + '%';

// 2. 靈感碎片行數
let fragCount = '—';
try {
  const _ff = app.vault.getAbstractFileByPath('07 Quick Capture & Hub/每日碎片化靈感隨記.md');
  if (_ff) {
    const _fc = await app.vault.read(_ff);
    fragCount = _fc.split('\n').filter(l => {
      const t = l.trim();
      if (!/^[-*] ./.test(t)) return false;    // 必須是 "- x" 或 "* x"（排除 ---）
      if (t.includes('✅')) return false;       // 已整理
      if (/^[-*] \[[^\]]+\]/.test(t)) return false; // 已分類（有 [tag]）
      return true;
    }).length;
  }
} catch(e) {}

// 3. 本輪發文完成度
let weekDone = 0, weekTotal = 0;
try {
  const _cp = app.vault.getAbstractFileByPath('01 Brand/社群數據/content_plan.md');
  if (_cp) {
    const _cc = await app.vault.read(_cp);
    const _rows = _cc.split('\n').filter(l => /^\|\s*\d{1,2}\/\d{1,2}\s*\|/.test(l));
    weekTotal = _rows.length;
    weekDone  = _rows.filter(l => /✅|已發|done/i.test(l)).length;
  }
} catch(e) {}
const weekPct = weekTotal > 0 ? Math.round(weekDone / weekTotal * 100) + '%' : '—';

// 4. 本週目標（T2）— 提前讀取供 stat card 使用
let _goals = [];
try {
  const _lf = app.vault.getAbstractFileByPath('06 Lifestyle & Self-Growth/生活自我成長中樞.md');
  if (_lf) {
    const _lc = await app.vault.read(_lf);
    const _t2 = _lc.match(/###\s*T2[^]*?(?=###\s*T3|##|$)/s);
    if (_t2) {
      const _today0g = new Date(now); _today0g.setHours(0,0,0,0);
      const _14d_out = new Date(_today0g.getTime() + 14 * 86400 * 1000); // 跟系統其他「近期」窗口（14天發文計畫、行事曆提醒）一致
      _goals = [..._t2[0].matchAll(/- \[( |x)\] (.+)/g)].map(m => {
        const raw  = m[2].trim();
        // 支援兩種日期寫法：「（截止 M/D）」括號格式，或項目開頭直接寫「M/D 標題」（可能有 ** 粗體包住）
        const dlm  = raw.match(/[（(]截止\s*(\d{1,2})[\/\-](\d{1,2})[）)]/) || raw.match(/^\*{0,2}(\d{1,2})\/(\d{1,2})/);
        let deadline = null, expired = false;
        if (dlm) {
          deadline = new Date(2026, parseInt(dlm[1])-1, parseInt(dlm[2]));
          if (deadline < _today0g) expired = true;
        }
        return { done: m[1]==='x', text: raw, deadline, expired };
      }).filter(g => !g.expired) // 截止日已過的自動隱藏
        .filter(g => !g.deadline || g.deadline <= _14d_out); // 只留 14 天內到期的，日期太遠的先不顯示（等進入週期再出現），避免「本週待辦」掛著下個月的事
    }
  }
} catch(e) {}

// 本週待辦完成度（來自 _goals）
const goalDone  = _goals.filter(g => g.done).length;
const goalTotal = _goals.length;
const goalPct   = goalTotal > 0 ? Math.round(goalDone / goalTotal * 100) + '%' : '—';

// 建 DOM
{
  const gridWrap = main.createEl('div', { attr: { style: 'margin-bottom: var(--db-gap);' } });
  const grid = gridWrap.createEl('div', { cls: 'db-grid db-grid-4col db-card-enter' });

  // 今日待辦：可展開的固定任務清單
  {
    const tdCard = grid.createEl('div', { cls: 'db-stat-card', attr: { style: 'cursor:pointer;' } });
    el(tdCard, 'div', { cls: 'db-stat-label', text: '今日待辦' });
    const tdVal = el(tdCard, 'div', { cls: 'db-stat-value', text: dailyPct });
    el(tdCard, 'div', { cls: 'db-stat-sub', text: `${dailyDone} / ${dailyTotal} 項完成` });

    const tdDrop = gridWrap.createEl('div', { attr: { style: 'display:none;background:var(--db-bg-card);border:1px solid var(--db-border);border-radius:var(--db-radius);padding:12px 16px;margin-top:6px;' } });
    el(tdDrop, 'div', { attr: { style: 'font-size:11px;color:var(--db-text-muted);text-transform:uppercase;letter-spacing:.08em;margin-bottom:8px;' }, text: '每日固定任務' });

    let _dDone = dailyDone;
    const updateStat = () => {
      tdVal.textContent = Math.round(_dDone / dailyTotal * 100) + '%';
      tdCard.querySelector('.db-stat-sub').textContent = `${_dDone} / ${dailyTotal} 項完成`;
    };

    _dailyDefs.forEach(taskName => {
      const row = tdDrop.createEl('div', { attr: { style: 'display:flex;align-items:center;gap:10px;padding:7px 0;border-bottom:1px solid var(--db-border);' } });
      const cb = row.createEl('input', { attr: { type: 'checkbox' } });
      cb.classList.add('db-checkbox');
      cb.checked = !!_dailyState[taskName];
      const lbl = el(row, 'span', { attr: { style: 'font-size:13px;' }, text: taskName });
      if (cb.checked) lbl.style.cssText += 'text-decoration:line-through;color:var(--db-text-muted);';
      cb.addEventListener('change', () => {
        _dailyState[taskName] = cb.checked;
        lbl.style.cssText = cb.checked ? 'font-size:13px;text-decoration:line-through;color:var(--db-text-muted);' : 'font-size:13px;';
        try { localStorage.setItem(_dailyKey, JSON.stringify(_dailyState)); } catch(e) {}
        _dDone = _dailyDefs.filter(t => _dailyState[t]).length;
        updateStat();
      });
    });

    // 今日行事曆
    el(tdDrop, 'div', { attr: { style: 'font-size:11px;color:var(--db-text-muted);text-transform:uppercase;letter-spacing:.08em;margin:10px 0 6px;' }, text: '📅 今日行事曆' });
    if (_calTodayEvs.length === 0) {
      el(tdDrop, 'div', { cls: 'db-empty', text: '今日無行程（或尚未匯出）' });
    } else {
      _calTodayEvs.forEach(ev => {
        const row = tdDrop.createEl('div', { attr: { style: 'display:flex;align-items:center;gap:8px;padding:5px 0;border-bottom:1px solid var(--db-border);font-size:13px;' } });
        const timeStr = ev.allDay ? '全天' : ev.start.slice(11,16);
        el(row, 'span', { attr: { style: 'color:var(--db-purple);font-size:11px;min-width:38px;flex-shrink:0;' }, text: timeStr });
        el(row, 'span', { attr: { style: 'flex:1;' }, text: ev.title });
        if (ev.location) el(row, 'span', { attr: { style: 'font-size:11px;color:var(--db-text-muted);' }, text: '📍' + ev.location.slice(0,12) });
      });
    }

    tdCard.addEventListener('click', () => {
      tdDrop.style.display = tdDrop.style.display === 'none' ? 'block' : 'none';
    });
  }

  // 今日行程 stat card
  {
    const card = grid.createEl('div', { cls: 'db-stat-card' });
    el(card, 'div', { cls: 'db-stat-label', text: '今日行程' });
    el(card, 'div', { cls: 'db-stat-value', text: _calTodayEvs.length > 0 ? String(_calTodayEvs.length) : '—' });
    el(card, 'div', { cls: 'db-stat-sub', text: _calTodayEvs.length > 0 ? '個行程' : '今日無行程' });
  }

  // 靈感碎片
  {
    const card = grid.createEl('div', { cls: 'db-stat-card', attr: { style: 'cursor:pointer;' } });
    el(card, 'div', { cls: 'db-stat-label', text: '靈感碎片' });
    el(card, 'div', { cls: 'db-stat-value', text: fragCount === 0 ? '✓' : String(fragCount) });
    el(card, 'div', { cls: 'db-stat-sub', text: fragCount === 0 ? '全部整理完畢' : '則待整理' });
    card.addEventListener('click', () => openPage('07 Quick Capture & Hub/靈感庫'));
  }

  // 第四格：近期待辦（14天內），點擊展開清單
  const todoCard = grid.createEl('div', { cls: 'db-stat-card', attr: { style: 'cursor:pointer;' } });
  el(todoCard, 'div', { cls: 'db-stat-label', text: '近期待辦（14天內）' });
  el(todoCard, 'div', { cls: 'db-stat-value', text: goalPct });
  el(todoCard, 'div', { cls: 'db-stat-sub',   text: `${goalDone} / ${goalTotal} 項完成` });

  // 展開清單（初始收合）
  const todoDropdown = gridWrap.createEl('div', { attr: { style: 'display:none; background:var(--db-bg-card); border:1px solid var(--db-border); border-radius:var(--db-radius); padding:12px 16px; margin-top:6px;' } });
  if (_goals.length === 0) {
    el(todoDropdown, 'div', { cls: 'db-empty', text: '生活中樞 T2 無資料' });
  } else {
    let _gDone = goalDone;
    const _goalValEl  = todoCard.querySelector('.db-stat-value');
    const _goalSubEl  = todoCard.querySelector('.db-stat-sub');
    const _updateGoalStat = () => {
      if (_goalValEl) _goalValEl.textContent = _goals.length > 0 ? Math.round(_gDone / _goals.length * 100) + '%' : '—';
      if (_goalSubEl) _goalSubEl.textContent = `${_gDone} / ${_goals.length} 項完成`;
    };

    _goals.forEach(({ done, text }, idx) => {
      const row = todoDropdown.createEl('div', { attr: { style: 'display:flex;align-items:center;gap:10px;padding:7px 0;border-bottom:1px solid var(--db-border);' } });
      const cb = row.createEl('input', { attr: { type: 'checkbox' } });
      cb.classList.add('db-checkbox');
      cb.checked = done;
      const lbl = el(row, 'span', { attr: { style: `font-size:13px;${done ? 'text-decoration:line-through;color:var(--db-text-muted);' : ''}` }, text });

      cb.addEventListener('change', async e => {
        e.stopPropagation();
        const checked = cb.checked;
        lbl.style.cssText = `font-size:13px;${checked ? 'text-decoration:line-through;color:var(--db-text-muted);' : ''}`;
        _gDone += checked ? 1 : -1;
        _updateGoalStat();
        // 寫回生活中樞 T2 section
        try {
          const _lf2 = app.vault.getAbstractFileByPath('06 Lifestyle & Self-Growth/生活自我成長中樞.md');
          if (_lf2) {
            const _lc2 = await app.vault.read(_lf2);
            const _from = checked ? `- [ ] ${text}` : `- [x] ${text}`;
            const _to   = checked ? `- [x] ${text}` : `- [ ] ${text}`;
            await app.vault.modify(_lf2, _lc2.replace(_from, _to));
          }
        } catch(e2) {}
      });
    });
  }
  // 週一清除已完成按鈕
  const _isMonday = now.getDay() === 1;
  const _clearBtn = todoDropdown.createEl('button', {
    attr: { style: `margin-top:10px;padding:5px 12px;font-size:12px;border-radius:6px;border:1px solid var(--db-border);background:${_isMonday ? 'var(--db-purple)' : 'var(--db-bg-deep)'};color:${_isMonday ? '#fff' : 'var(--db-text-muted)'};cursor:pointer;` },
    text: _isMonday ? '🗑 週一更新：清除已完成' : '🗑 清除已完成（週一執行）',
  });
  _clearBtn.addEventListener('click', async e => {
    e.stopPropagation();
    if (!_isMonday && !confirm('現在不是週一，確定要清除本週已完成的項目？')) return;
    try {
      const _lf3 = app.vault.getAbstractFileByPath('06 Lifestyle & Self-Growth/生活自我成長中樞.md');
      if (_lf3) {
        const _lc3 = await app.vault.read(_lf3);
        // 移除 T2 裡所有 [x] 行（已完成），保留 [ ] 未完成
        const _newContent = _lc3.replace(/^- \[x\] .+\n?/gm, '');
        await app.vault.modify(_lf3, _newContent);
        showToast('✅ 已完成項目已清除，未完成自動帶入下週');
      }
    } catch(e2) { showToast('⚠️ 清除失敗', 'error'); }
  });

  todoCard.addEventListener('click', () => {
    todoDropdown.style.display = todoDropdown.style.display === 'none' ? 'block' : 'none';
  });
}

// ── Step 5 資料 ───────────────────────────────────────────────

// 今天日期字串（共用）
const _todayStr = `${now.getFullYear()}-${String(now.getMonth()+1).padStart(2,'0')}-${String(now.getDate()).padStart(2,'0')}`;

// 寫入檔案工具
const writeAppend = async (path, line) => {
  try {
    const f = app.vault.getAbstractFileByPath(path);
    if (!f) return false;
    const c = await app.vault.read(f);
    await app.vault.modify(f, c.trimEnd() + '\n' + line + '\n');
    return true;
  } catch(e) { return false; }
};

// DC 通知橋接（呼叫 brain-bot HTTP server）
const notifyDC = async (channel, message) => {
  try {
    await requestUrl({
      url: 'http://localhost:3456/notify',
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ channel, message }),
      throw: false,
    });
  } catch(e) { /* bot 未啟動時靜默失敗 */ }
};

// 習慣清單（從自律計劃表讀取）
let _habits = [];
try {
  const _sp = app.vault.getAbstractFileByPath('06 Lifestyle & Self-Growth/自律計劃表.md');
  if (_sp) {
    const _sc = await app.vault.read(_sp);
    _habits = [..._sc.matchAll(/^\|\s*([^|]+?)\s*\|\s*每日/gm)].map(m => m[1].trim()).filter(Boolean);
  }
} catch(e) {}
if (_habits.length === 0) _habits = ['喝水 2000cc','運動','早睡（00:00 前）','社群維護互動'];

// 今日已打卡的習慣（讀自律日誌，避免重新渲染後 checkbox 消失）
const _todayDone = new Set();
try {
  const _jf = app.vault.getAbstractFileByPath('06 Lifestyle & Self-Growth/自律日誌.md');
  if (_jf) {
    const _jc = await app.vault.read(_jf);
    for (const m of _jc.matchAll(new RegExp(_todayStr + ' ✅ ([^\\n]+)', 'g'))) {
      _todayDone.add(m[1].trim());
    }
  }
} catch(e) {}

// 待補數據的貼文（history.md + content_plan.md 中 >48H、讚=— 且備註含已發的列）
let _pendingBackfill = [];
try {
  const _48h = new Date(now.getTime() - 48 * 3600 * 1000);
  const _14d  = new Date(now.getTime() - 14 * 86400 * 1000);
  const _seen = new Set(); // 去重（date+platform+formula）

  const _parseBackfillLines = (content, requirePublished) => {
    for (const line of content.split('\n')) {
      if (!/^\|\s*\d{4}-\d{2}-\d{2}/.test(line)) continue;
      const cols = line.split('|').map(c => c.trim()).filter(Boolean);
      if (cols.length < 4) continue;
      const [date, formula, platform, likes] = cols;
      const note = cols[6] || '';
      const d = new Date(date);
      if (isNaN(d.getTime()) || d > _48h || d < _14d) continue;
      const normPlat = (platform || '').split(/\s+@/)[0].trim();
      const key = `${date}|${normPlat}|${formula}`;
      // 不論有無數據，都標記為已見過（防止 content_plan 重複撈）
      if (_seen.has(key)) continue;
      _seen.add(key);
      if (likes && likes !== '—') continue; // 已有數字 → 不加入待補清單
      if (requirePublished && !note.includes('已發') && !note.includes('數據待補')) continue;
      _pendingBackfill.push({ date, formula, platform });
    }
  };

  const _hbf = app.vault.getAbstractFileByPath('01 Brand/社群數據/history.md');
  if (_hbf) _parseBackfillLines(await app.vault.read(_hbf), false);


  _pendingBackfill.sort((a, b) => b.date.localeCompare(a.date)); // 最新在前
} catch(e) {}

// 側邊欄狀態徽章（靈感碎片待整理數 / 待回填數）— 資料算好後才補上，不影響 sidebar 原本渲染順序
(function addSidebarBadges() {
  const addBadge = (labelText, count) => {
    if (!count) return;
    const items = shell.querySelectorAll('.db-nav-item');
    for (const item of items) {
      if (item.textContent.trim() === labelText) {
        el(item, 'span', { cls: 'db-badge', attr: { style: 'margin-left:auto;font-size:9px;background:var(--db-purple);color:#fff;' }, text: String(count) });
        item.style.display = 'flex';
        item.style.alignItems = 'center';
        break;
      }
    }
  };
  addBadge('快速捕獲', typeof fragCount === 'number' ? fragCount : 0);
  addBadge('品牌中樞', _pendingBackfill.length);
})();

// 第一排：行事曆 + 每日校準
{
  const row = main.createEl('div', { cls: 'db-grid db-grid-2col db-card-enter', attr: { style: 'margin-bottom: var(--db-gap);' } });

  // ── 左卡：月曆 ─────────────────────────────────────────────
  {
    const card = row.createEl('div', { cls: 'db-card' });
    const hdr = el(card, 'div', { cls: 'db-card-header' });
    el(hdr, 'div', { cls: 'db-card-title', text: '📅  行事曆' });
    el(hdr, 'div', { cls: 'db-card-action', attr: { style: 'font-size:10px;color:var(--db-text-muted);' }, text: '每天 7:55 更新' });

    // 事件 index：key = "YYYY-MM-DD"（跨日全天事件展開到每一天）
    const _evByDate = {};
    const _addEvDate = (dk, ev) => {
      if (!_evByDate[dk]) _evByDate[dk] = [];
      _evByDate[dk].push(ev);
    };
    _calEvents.forEach(ev => {
      const dk = ev.start ? ev.start.slice(0,10) : null;
      if (!dk) return;
      const endDk = ev.end ? ev.end.slice(0,10) : dk;
      if (endDk > dk) {
        // 跨日事件：展開到每一天（全天 end 為 exclusive，有時間的 end 當天也算）
        let cur = new Date(dk + 'T00:00:00');
        const limitDk = ev.allDay ? endDk : new Date(endDk + 'T23:59:59').toISOString().slice(0,10);
        while (true) {
          const curStr = `${cur.getFullYear()}-${String(cur.getMonth()+1).padStart(2,'0')}-${String(cur.getDate()).padStart(2,'0')}`;
          if (ev.allDay ? curStr >= limitDk : curStr > limitDk) break;
          _addEvDate(curStr, ev);
          cur.setDate(cur.getDate() + 1);
        }
      } else {
        _addEvDate(dk, ev);
      }
    });

    let _calViewYear  = now.getFullYear();
    let _calViewMonth = now.getMonth();
    let _calSelectedDate = null;

    const calArea    = card.createEl('div');
    const detailArea = card.createEl('div', { attr: { style: 'padding:0 2px;' } });

    function renderCalendar() {
      calArea.empty();
      detailArea.empty();

      // 月份導航列
      const mHdr = calArea.createEl('div', { attr: { style: 'display:flex;align-items:center;justify-content:space-between;padding:8px 4px 4px;' } });
      const btnPrev = mHdr.createEl('div', { attr: { style: 'cursor:pointer;padding:2px 10px;font-size:16px;color:var(--db-text-muted);user-select:none;' }, text: '‹' });
      el(mHdr, 'div', { attr: { style: 'font-size:13px;font-weight:600;color:var(--db-text-primary);' }, text: `${_calViewYear} 年 ${_calViewMonth+1} 月` });
      const btnNext = mHdr.createEl('div', { attr: { style: 'cursor:pointer;padding:2px 10px;font-size:16px;color:var(--db-text-muted);user-select:none;' }, text: '›' });

      btnPrev.addEventListener('click', () => {
        _calViewMonth--; if (_calViewMonth < 0) { _calViewMonth = 11; _calViewYear--; }
        _calSelectedDate = null; renderCalendar();
      });
      btnNext.addEventListener('click', () => {
        _calViewMonth++; if (_calViewMonth > 11) { _calViewMonth = 0; _calViewYear++; }
        _calSelectedDate = null; renderCalendar();
      });

      // 星期標頭
      const grid = calArea.createEl('div', { attr: { style: 'display:grid;grid-template-columns:repeat(7,1fr);gap:2px;padding:0 2px;' } });
      ['日','一','二','三','四','五','六'].forEach(d => {
        el(grid, 'div', { attr: { style: 'text-align:center;font-size:10px;color:var(--db-text-muted);padding:2px 0 4px;' }, text: d });
      });

      // 第一天星期幾、月天數
      const firstDay    = new Date(_calViewYear, _calViewMonth, 1).getDay();
      const daysInMonth = new Date(_calViewYear, _calViewMonth+1, 0).getDate();
      const todayD = now.getDate(), todayM = now.getMonth(), todayY = now.getFullYear();

      for (let i = 0; i < firstDay; i++) el(grid, 'div', {});

      for (let d = 1; d <= daysInMonth; d++) {
        const dk = `${_calViewYear}-${String(_calViewMonth+1).padStart(2,'0')}-${String(d).padStart(2,'0')}`;
        const isToday    = d === todayD && _calViewMonth === todayM && _calViewYear === todayY;
        const isSelected = dk === _calSelectedDate;
        const hasEv      = !!_evByDate[dk];

        const cell = grid.createEl('div', { attr: { style:
          'text-align:center;padding:5px 2px 3px;border-radius:6px;font-size:12px;cursor:pointer;position:relative;' +
          (isSelected ? 'background:var(--db-purple);color:#fff;font-weight:700;' :
           isToday    ? 'background:var(--db-purple-subtle);color:var(--db-cream);font-weight:700;' :
                        'color:var(--db-text-primary);')
        } });
        el(cell, 'span', { text: String(d) });
        // 事件小點
        if (hasEv) {
          el(cell, 'div', { attr: { style:
            'width:4px;height:4px;border-radius:50%;margin:1px auto 0;' +
            (isSelected ? 'background:#fff;' : 'background:var(--db-purple);')
          } });
        }
        cell.addEventListener('click', () => {
          _calSelectedDate = isSelected ? null : dk;
          renderCalendar();
        });
      }

      // 詳情區
      const todayKey = `${now.getFullYear()}-${String(now.getMonth()+1).padStart(2,'0')}-${String(now.getDate()).padStart(2,'0')}`;
      const showKey  = _calSelectedDate || (_evByDate[todayKey] ? todayKey : null);

      if (showKey && _evByDate[showKey]) {
        const label = showKey === todayKey && !_calSelectedDate ? '今日行程' : `${showKey.slice(5).replace('-','/')} 行程`;
        el(detailArea, 'div', { attr: { style: 'font-size:10px;color:var(--db-text-muted);text-transform:uppercase;letter-spacing:.06em;padding:7px 0 3px;border-top:1px solid var(--db-border);margin-top:4px;' }, text: label });
        _evByDate[showKey].forEach(ev => {
          const er = detailArea.createEl('div', { attr: { style: 'display:flex;align-items:flex-start;gap:8px;padding:5px 0;border-bottom:1px solid var(--db-border);' } });
          const timeStr = ev.allDay ? '全天' : ev.start.slice(11,16);
          el(er, 'span', { attr: { style: 'font-size:11px;color:var(--db-purple);min-width:34px;flex-shrink:0;font-weight:600;' }, text: timeStr });
          const info = er.createEl('div');
          el(info, 'div', { attr: { style: 'font-size:13px;' }, text: ev.title });
          if (ev.location) el(info, 'div', { attr: { style: 'font-size:11px;color:var(--db-text-muted);margin-top:2px;' }, text: '📍 ' + ev.location });
        });
      } else if (_calSelectedDate) {
        el(detailArea, 'div', { attr: { style: 'font-size:12px;color:var(--db-text-muted);padding:8px 0 4px;border-top:1px solid var(--db-border);margin-top:4px;text-align:center;' }, text: '此日無行程' });
      } else {
        el(detailArea, 'div', { attr: { style: 'font-size:12px;color:var(--db-text-muted);padding:8px 0 4px;border-top:1px solid var(--db-border);margin-top:6px;text-align:center;' }, text: '今日無行程' });
      }
    }

    renderCalendar();
  }

  // ── 右卡：每日校準 + 發文數據（Tab 切換）──────────────────
  {
    const card = row.createEl('div', { cls: 'db-card' });
    const hdr = el(card, 'div', { cls: 'db-card-header' });
    el(hdr, 'div', { cls: 'db-card-title', text: '✦  每日校準' });

    // Tab 列（狀態存 window，讓寫檔觸發 DataviewJS 整塊重繪時仍留在原分頁，不強制跳回習慣打卡）
    if (!window.__dbTabState) window.__dbTabState = {};
    const _calibTabActive = window.__dbTabState.calib !== 'data';
    const tabs = el(card, 'div', { cls: 'db-tabs' });
    const tabCalib = el(tabs, 'div', { cls: 'db-tab' + (_calibTabActive ? ' active' : ''), text: '📋 習慣打卡' });
    const tabData  = el(tabs, 'div', { cls: 'db-tab' + (_calibTabActive ? '' : ' active'), text: '📊 發文數據' });

    // ── Panel 1：習慣打卡 ──
    const panelCalib = el(card, 'div', { cls: 'db-tab-panel' + (_calibTabActive ? ' active' : '') });
    el(panelCalib, 'div', { attr: { style: 'font-size:11px; color:var(--db-text-muted); margin-bottom:10px;' }, text: `${_todayStr} 今日打卡` });

    _habits.forEach(habit => {
      const alreadyDone = _todayDone.has(habit);
      const row2 = el(panelCalib, 'div', { cls: 'db-checkbox-row' });
      const cb = row2.createEl('input', { attr: { type: 'checkbox' } });
      cb.classList.add('db-checkbox');
      if (alreadyDone) cb.checked = true;
      const lbl = el(row2, 'span', { cls: 'db-checkbox-label' + (alreadyDone ? ' done' : ''), text: habit });
      cb.addEventListener('change', async () => {
        if (cb.checked) {
          lbl.classList.add('done');
          if (!_todayDone.has(habit)) { // 防止重複寫入
            const ok = await writeAppend('06 Lifestyle & Self-Growth/自律日誌.md', `- ${_todayStr} ✅ ${habit}`);
            if (ok) _todayDone.add(habit);
            showToast(ok ? `✅ 已記錄：${habit}` : '⚠️ 寫入失敗', ok ? 'success' : 'error');
          }
        } else {
          lbl.classList.remove('done');
        }
      });
    });

    // ── Panel 2：發文數據填寫 ──
    const panelData = el(card, 'div', { cls: 'db-tab-panel' + (_calibTabActive ? '' : ' active') });

    const form = el(panelData, 'div', { attr: { style: 'display:flex; flex-direction:column; gap:8px;' } });

    // 日期 + 平台
    const row1 = el(form, 'div', { attr: { style: 'display:grid; grid-template-columns:1fr 1fr; gap:8px;' } });
    const inDate = row1.createEl('input', { cls: 'db-input', attr: { type: 'date', value: _todayStr } });
    const inPlat = row1.createEl('select', { cls: 'db-select' });
    ['FB個人','Gracie粉專','個人IG','撥撥IG','個人Threads','TikTok','X','COS IG','COS Threads'].forEach(p => {
      inPlat.createEl('option', { text: p, attr: { value: p } });
    });

    // 待補數據下拉（history.md >48H 未填）+ 公式手填輸入
    const inPost = form.createEl('select', { cls: 'db-select' });
    inPost.createEl('option', { text: _pendingBackfill.length > 0 ? `── 選擇已記錄貼文（${_pendingBackfill.length} 篇待補）──` : '── 無 history.md 待補（請手填公式）──', attr: { value: '' } });
    _pendingBackfill.forEach((p, i) => {
      inPost.createEl('option', { text: `${p.date}  ${p.platform}  ${p.formula}`, attr: { value: String(i) } });
    });

    // 公式 autocomplete datalist
    const dlId = 'db-formula-list';
    const dl = form.createEl('datalist', { attr: { id: dlId } });
    const formulaOptions = [
      'Mode A — 日常廢文碎念吐槽（帶圖）',
      'Mode B — 工作曝光+照片',
      'Mode C — 個人脆弱/反思/行業洞察長文',
      'F2 工作照 — 前125字鉤子+hashtag',
      'F7 POV — Threads短吐槽60-150字',
      'F12 預告 — 聲播/直播預告',
      'F18 撥撥Reels — 吉祥物語氣15-30秒',
      'F19 立場宣言 — Threads表態',
      'F22 AI大腦資料庫 — tag賀少穎',
      'F23 Mode C行業洞察 — AI/社群洞察',
    ];
    formulaOptions.forEach(f => dl.createEl('option', { attr: { value: f } }));

    // 公式/主題 手填欄（選下拉時自動帶入，也可直接手打）
    const inFormula = form.createEl('input', { cls: 'db-input', attr: { placeholder: '公式/主題（打字自動提示，或選上方下拉）', list: dlId } });

    // 可折疊公式速查
    const cheatToggle = el(form, 'div', { attr: { style: 'font-size:11px;color:var(--db-teal);cursor:pointer;user-select:none;margin-top:-2px;' }, text: '📋 公式速查 ▸' });
    const cheatBody = el(form, 'div', { attr: { style: 'display:none;background:var(--db-bg-deep);border-radius:var(--db-radius-inner);padding:8px 10px;font-size:11px;line-height:1.8;color:var(--db-text-secondary);' } });
    [
      ['Mode A', '日常廢文碎念吐槽（帶圖）'],
      ['Mode B', '工作曝光 + 照片'],
      ['Mode C', '個人脆弱/反思/行業洞察深度長文'],
      ['F2', '工作照，前 125 字鉤子，hashtag 5-8 個'],
      ['F7', 'Threads 短吐槽 60-150 字（POV）'],
      ['F12', '直播/聲播預告'],
      ['F18', '撥撥 Reels，吉祥物語氣，15-30 秒'],
      ['F19', 'Threads 表態/立場宣言'],
      ['F22', 'AI 大腦資料庫，tag 賀少穎'],
      ['F23', 'Mode C 行業洞察特別篇'],
    ].forEach(([code, desc]) => {
      const row = el(cheatBody, 'div', { attr: { style: 'display:flex;gap:8px;cursor:pointer;padding:1px 0;' } });
      el(row, 'span', { attr: { style: 'color:var(--db-purple);font-weight:600;min-width:52px;' }, text: code });
      el(row, 'span', { text: desc });
      row.addEventListener('click', () => { inFormula.value = code + ' — ' + desc; cheatBody.style.display = 'none'; cheatToggle.textContent = '📋 公式速查 ▸'; });
    });
    let _cheatOpen = false;
    cheatToggle.addEventListener('click', () => {
      _cheatOpen = !_cheatOpen;
      cheatBody.style.display = _cheatOpen ? 'block' : 'none';
      cheatToggle.textContent = _cheatOpen ? '📋 公式速查 ▾' : '📋 公式速查 ▸';
    });

    // 選擇後自動填入日期、平台、公式
    inPost.addEventListener('change', () => {
      const idx = parseInt(inPost.value);
      if (isNaN(idx)) return;
      const p = _pendingBackfill[idx];
      inDate.value = p.date;
      inFormula.value = p.formula;
      const matchPlat = [...inPlat.options].find(o => o.value === p.platform || p.platform.includes(o.value) || o.value.includes(p.platform));
      if (matchPlat) inPlat.value = matchPlat.value;
    });

    // 數字欄
    const row2m = el(form, 'div', { attr: { style: 'display:grid; grid-template-columns:1fr 1fr 1fr; gap:8px;' } });
    const inLike = row2m.createEl('input', { cls: 'db-input', attr: { type: 'number', placeholder: '讚', min: '0' } });
    const inComm = row2m.createEl('input', { cls: 'db-input', attr: { type: 'number', placeholder: '留言', min: '0' } });
    const inShar = row2m.createEl('input', { cls: 'db-input', attr: { type: 'number', placeholder: '分享', min: '0' } });

    // 備註（觸及等）
    const inNote = form.createEl('input', { cls: 'db-input', attr: { placeholder: '備註（觸及、觀看次數等）' } });

    // 送出
    const btnSubmit = el(form, 'button', { cls: 'db-btn db-btn-primary', text: '✓  寫入戰績表' });
    btnSubmit.style.cssText += ';width:100%;margin-top:4px;justify-content:center;';

    btnSubmit.addEventListener('click', async () => {
      const date    = inDate.value || _todayStr;
      const plat    = inPlat.value;
      const formula = inFormula.value.trim() || '未標記';
      const likes   = inLike.value || '—';
      const comms   = inComm.value || '—';
      const shares  = inShar.value || '—';
      const notes   = inNote.value.trim() || '';
      if (!formula || formula === '未標記') { showToast('請填入公式或主題', 'error'); return; }
      const newRow = `| ${date} | ${formula} | ${plat} | ${likes} | ${comms} | ${shares} | ${notes} |`;

      // 若從下拉選取，原地取代 — 行（避免 re-render 後 — 行仍在，下拉重複出現）
      const isFromDropdown = inPost.selectedIndex > 0;
      const selIdx = inPost.selectedIndex;
      const selPost = isFromDropdown ? _pendingBackfill[selIdx - 1] : null;

      let ok = false;
      try {
        const hf = app.vault.getAbstractFileByPath('01 Brand/社群數據/history.md');
        if (!hf) throw new Error('no file');
        const hc = await app.vault.read(hf);
        let replaced = false;

        if (selPost) {
          const updatedLines = hc.split('\n').map(line => {
            if (replaced || !line.startsWith('|')) return line;
            const cols = line.split('|').map(c => c.trim()).filter(Boolean);
            if (cols.length >= 4 && cols[0] === selPost.date && cols[1] === selPost.formula && cols[2] === selPost.platform && cols[3] === '—') {
              replaced = true;
              return newRow;
            }
            return line;
          });
          if (replaced) { await app.vault.modify(hf, updatedLines.join('\n')); ok = true; }
        }

        // fallback：手填或找不到對應行 → 追加
        if (!ok) {
          await app.vault.modify(hf, hc.trimEnd() + '\n' + newRow + '\n');
          ok = true;
        }
      } catch(e) { ok = false; }

      if (ok) {
        window.__dbTabState.calib = 'data'; // 寫檔會觸發整塊重繪，確保重繪後仍停在發文數據分頁
        showToast(`✅ 已寫入：${date} ${plat}`);
        notifyDC('品牌', `📊 數據回填｜${date} ${plat} ${formula}｜讚${likes} 留${comms} 分${shares}${notes ? ' '+notes : ''}`);
        if (isFromDropdown) {
          inPost.remove(selIdx);
          _pendingBackfill.splice(selIdx - 1, 1);
          const rem = inPost.options.length - 1;
          inPost.options[0].text = rem > 0 ? `── 選擇已記錄貼文（${rem} 篇待補）──` : '── 無待補貼文（請手填公式）──';
        }
        inPost.selectedIndex = 0; inFormula.value = ''; inLike.value = ''; inComm.value = ''; inShar.value = ''; inNote.value = '';
      } else {
        showToast('⚠️ 寫入失敗，請確認路徑', 'error');
      }
    });

    // Tab 切換邏輯（同步記到 window，寫檔重繪後才不會跳回習慣打卡）
    tabCalib.addEventListener('click', () => {
      window.__dbTabState.calib = 'calib';
      tabCalib.classList.add('active'); tabData.classList.remove('active');
      panelCalib.classList.add('active'); panelData.classList.remove('active');
    });
    tabData.addEventListener('click', () => {
      window.__dbTabState.calib = 'data';
      tabData.classList.add('active'); tabCalib.classList.remove('active');
      panelData.classList.add('active'); panelCalib.classList.remove('active');
    });
  }
}

// 第二排：本週目標（_goals 已於上方 stat card 區塊讀取）

let _upcomingPosts = [];
let _schedule14 = []; // 未來 14 天（含今天）發文計畫
let _suggestions = []; // 本輪主題建議（⬜待發）
try {
  const _cpf = app.vault.getAbstractFileByPath('01 Brand/社群數據/content_plan.md');
  if (_cpf) {
    const _cpc = await app.vault.read(_cpf);
    const _today0 = new Date(now); _today0.setHours(0,0,0,0);
    const _14d_end = new Date(_today0.getTime() + 14 * 86400 * 1000);

    for (const ln of _cpc.split('\n')) {
      if (!/^\|/.test(ln)) continue;
      const cols = ln.split('|').map(c => c.trim()).filter(Boolean);
      if (cols.length < 5) continue;
      // 第一欄提取 M/DD（忽略 emoji/粗體/備注）
      const dm = cols[0].replace(/\*/g,'').match(/(\d{1,2})\/(\d{1,2})/);
      if (!dm) continue;
      const dateStr = `2026-${dm[1].padStart(2,'0')}-${dm[2].padStart(2,'0')}`;
      const d = new Date(dateStr); d.setHours(0,0,0,0);
      if (d < _today0 || d > _14d_end) continue;
      const [, platform, formula, time, topic, status=''] = cols;
      if (platform === '平台' || formula === '公式') continue; // 表頭
      _schedule14.push({ dateStr, d, platform, formula, time, topic, status: status.trim() });
    }
    _schedule14.sort((a,b) => a.d - b.d);

    // _upcomingPosts：content_plan.md 表格用 M/DD 格式（個人品牌卡「近期已發」用）
    _upcomingPosts = _cpc.split('\n')
      .filter(l => /^\|\s*\d{1,2}\/\d{1,2}\s*\|/.test(l) && l.includes('已發'))
      .map(l => { const c = l.split('|').map(s=>s.trim()).filter(Boolean); return c.length>=5 ? {date:c[0], topic:(c[4]||c[2]||'').trim(), platform:c[1]} : null; })
      .filter(Boolean)
      .slice(-5)
      .reverse();

    // 本輪⬜待發主題建議（今天起 7 天，M/DD 格式行）
    const _7d_end = new Date(_today0.getTime() + 7 * 86400 * 1000);
    _suggestions = _schedule14.filter(r => {
      const st = r.status;
      return st.includes('⬜') || (!st.includes('✅') && !st.includes('❌') && !st.includes('📌') && st !== '');
    }).concat(
      // 也抓「文案已定/待排/等素材」等文字狀態
      _schedule14.filter(r => /待排|待命|文案已定|等素材|待確認/.test(r.status))
    ).filter((r, i, arr) => arr.findIndex(x => x.dateStr===r.dateStr && x.platform===r.platform) === i)
     .slice(0, 8);
  }
} catch(e) {}

{
  const card = weeklyBody.createEl('div', { cls: 'db-card db-card-enter', attr: { style: 'margin-bottom: var(--db-gap);' } });
  const hdr = el(card, 'div', { cls: 'db-card-header' });
  el(hdr, 'div', { cls: 'db-card-title', text: '🎯  近期待辦（14天內）' });
  el(hdr, 'div', { cls: 'db-card-action', text: '→ 生活中樞' }).addEventListener('click', () => openPage('生活自我成長中樞'));

  const goalGrid = el(card, 'div', { attr: { style: 'display:grid; grid-template-columns:1fr 1fr; gap:16px;' } });

  // 左欄：T2 有截止日
  const lc = goalGrid.createEl('div');
  el(lc, 'div', { attr: { style: 'font-size:11px; color:var(--db-text-muted); text-transform:uppercase; letter-spacing:.08em; margin-bottom:8px;' }, text: '有截止日的事' });
  if (_goals.length === 0) {
    el(lc, 'div', { cls: 'db-empty', text: '生活中樞 T2 無項目' });
  } else {
    _goals.forEach(({ done, text }) => {
      const row = el(lc, 'div', { cls: 'db-checkbox-row' });
      const cb = row.createEl('input', { attr: { type: 'checkbox' } });
      cb.classList.add('db-checkbox');
      if (done) cb.checked = true;
      el(row, 'span', { cls: 'db-checkbox-label' + (done ? ' done' : ''), text });
    });
  }

  // 右欄：本輪發文主題建議（⬜待發）
  const rc = goalGrid.createEl('div');
  el(rc, 'div', { attr: { style: 'font-size:11px; color:var(--db-text-muted); text-transform:uppercase; letter-spacing:.08em; margin-bottom:8px;' }, text: '📌 本輪發文主題建議' });
  if (_suggestions.length === 0) {
    el(rc, 'div', { cls: 'db-empty', text: '本輪 ⬜ 無待發項目' });
  } else {
    _suggestions.forEach(({ dateStr, platform, formula, topic }) => {
      const row = el(rc, 'div', { cls: 'db-list-item', attr: { style: 'align-items:flex-start;gap:6px;' } });
      const dateShort = dateStr.slice(5).replace('-', '/');
      el(row, 'span', { attr: { style: 'color:var(--db-text-muted);font-size:11px;min-width:38px;flex-shrink:0;' }, text: dateShort });
      const platColor = /撥撥|TikTok/.test(platform) ? 'var(--db-teal)' : /Gracie|FB/.test(platform) ? 'var(--db-purple)' : /IG/.test(platform) ? 'var(--db-rose)' : 'var(--db-cream)';
      el(row, 'span', { attr: { style: `font-size:10px;color:${platColor};flex-shrink:0;min-width:52px;` }, text: platform.replace(/個人|粉專/g,'').slice(0,6) });
      el(row, 'span', { attr: { style: 'font-size:12px;' }, text: (topic||formula||'').slice(0,22) + ((topic||formula||'').length>22?'…':'') });
    });
  }
}

// 第二排半：14 天發文計畫（全寬，收合式，預設收合）
{
  const card = main.createEl('div', { cls: 'db-card db-card-enter', attr: { style: 'margin-bottom: var(--db-gap);' } });
  const hdr = el(card, 'div', { cls: 'db-card-header', attr: { style: 'cursor:pointer;user-select:none;' } });
  el(hdr, 'div', { cls: 'db-card-title', text: '📅  未來 14 天發文計畫' });
  el(hdr, 'div', { cls: 'db-card-action', text: '→ 內容計畫' }).addEventListener('click', (e) => { e.stopPropagation(); openPage('01 Brand/社群數據/content_plan'); });
  const schedChev = el(hdr, 'div', { attr: { style: 'font-size:13px;color:var(--db-text-muted);margin-left:8px;' }, text: '▸' });
  const schedWrap = card.createEl('div', { attr: { style: 'display:none;' } });
  hdr.addEventListener('click', () => {
    const open = schedWrap.style.display !== 'none';
    schedWrap.style.display = open ? 'none' : 'block';
    schedChev.textContent   = open ? '▸' : '▾';
  });

  if (_schedule14.length === 0) {
    el(schedWrap, 'div', { cls: 'db-empty', text: '無排程資料（content_plan.md 尚無 M/DD 格式行）' });
  } else {
    const body = el(schedWrap, 'div', { cls: 'db-card-body db-card-body-tall', attr: { style: 'max-height:320px;' } });
    let lastDate = '';
    _schedule14.forEach(({ dateStr, d, platform, formula, time, topic, status }) => {
      const isDone      = status.includes('✅');
      const isCancelled = status.includes('❌');
      const isPinned    = status.includes('📌');
      const isToday     = dateStr === _todayStr;

      if (dateStr !== lastDate) {
        lastDate = dateStr;
        const dLabel = d.toLocaleDateString('zh-TW', { month:'numeric', day:'numeric', weekday:'short' });
        const dateLine = el(body, 'div', { attr: { style: `display:flex;align-items:center;gap:8px;padding:8px 0 4px;border-top:1px solid var(--db-border);margin-top:4px;font-size:11px;font-weight:600;color:${isToday?'var(--db-cream)':'var(--db-text-muted)'};` } });
        if (isToday) el(dateLine, 'span', { cls: 'db-badge', attr: { style: 'font-size:9px;background:var(--db-purple-subtle);' }, text: '今日' });
        el(dateLine, 'span', { text: dLabel });
      }

      const row = el(body, 'div', { attr: { style: `display:flex;align-items:baseline;gap:8px;padding:4px 6px 4px 12px;border-radius:6px;font-size:12px;opacity:${isCancelled?'0.35':isDone?'0.6':'1'};${isCancelled?'text-decoration:line-through;':''}` } });

      // 狀態圖示
      el(row, 'span', { attr: { style: 'flex-shrink:0;font-size:11px;' }, text: isDone?'✅':isCancelled?'❌':isPinned?'📌':'⬜' });

      // 平台 badge
      const platColor = platform.includes('FB')||platform.includes('Gracie') ? 'var(--db-purple)' : platform.includes('IG') ? 'var(--db-rose)' : platform.includes('Threads')||platform.includes('X') ? 'var(--db-teal)' : 'var(--db-text-muted)';
      el(row, 'span', { attr: { style: `font-size:10px;color:${platColor};font-weight:600;min-width:80px;flex-shrink:0;` }, text: platform.replace(' + X','').replace('個人','') });

      // 公式
      el(row, 'span', { attr: { style: 'font-size:10px;color:var(--db-text-muted);min-width:52px;flex-shrink:0;' }, text: formula });

      // 題材
      el(row, 'span', { attr: { style: 'flex:1;color:var(--db-text-primary);' }, text: topic.length>40 ? topic.slice(0,40)+'…' : topic });

      // 時間
      if (time && time !== '—') el(row, 'span', { attr: { style: 'font-size:10px;color:var(--db-text-muted);flex-shrink:0;' }, text: time });
    });
  }
}

// ── Step 6 資料：最近碎片 ─────────────────────────────────────
let _recentFrags = [];
try {
  const _rfFile = app.vault.getAbstractFileByPath('07 Quick Capture & Hub/每日碎片化靈感隨記.md');
  if (_rfFile) {
    const _rfc = await app.vault.read(_rfFile);
    _recentFrags = _rfc.split('\n')
      .filter(l => /^- /.test(l.trim()) && l.trim().length > 4)
      .slice(-10).reverse(); // 最新 10 筆，倒序
  }
} catch(e) {}

// 第四排：靈感碎片（兩欄，align-items:start 避免收合卡片被對面卡片撐高）
{
  const row = main.createEl('div', { cls: 'db-grid db-grid-2col db-card-enter', attr: { style: 'margin-bottom: var(--db-gap);align-items:start;' } });

  // ── 左卡：碎片輸入 ──
  {
    const card = row.createEl('div', { cls: 'db-card' });
    const hdr = el(card, 'div', { cls: 'db-card-header' });
    el(hdr, 'div', { cls: 'db-card-title', text: '💡  靈感碎片' });
    el(hdr, 'span', { attr: { style: 'font-size:10px;color:var(--db-teal);' }, text: '→ DC #快速捕獲' });

    // 分類選單
    const cats = [
      { val: '靈感',    label: '💡 靈感' },
      { val: '發文素材', label: '✍️ 發文素材' },
      { val: '自律',    label: '🏃 自律' },
      { val: '投資',    label: '📈 投資' },
      { val: '生活',    label: '🌱 生活' },
      { val: '接案',    label: '💼 接案' },
      { val: '雜念',    label: '💭 雜念' },
    ];
    const selCat = card.createEl('select', { cls: 'db-select', attr: { style: 'width:100%; margin-bottom:8px;' } });
    cats.forEach(c => selCat.createEl('option', { text: c.label, attr: { value: c.val } }));

    // 輸入框
    const taFrag = card.createEl('textarea', { cls: 'db-input db-textarea', attr: { placeholder: '隨手打，不用分類，打完按送出…', rows: '3' } });
    taFrag.style.marginBottom = '8px';

    // 送出鈕
    const btnFrag = el(card, 'button', { cls: 'db-btn db-btn-primary', text: '⚡ 捕獲碎片' });
    btnFrag.style.cssText += ';width:100%;justify-content:center;margin-bottom:12px;';

    btnFrag.addEventListener('click', async () => {
      const content = taFrag.value.trim();
      if (!content) { showToast('請輸入內容', 'error'); return; }
      const cat = selCat.value;
      const ts  = new Date();
      const tsStr = `${ts.getFullYear()}-${String(ts.getMonth()+1).padStart(2,'0')}-${String(ts.getDate()).padStart(2,'0')} ${String(ts.getHours()).padStart(2,'0')}:${String(ts.getMinutes()).padStart(2,'0')}`;
      const line = `- [${cat}] ${tsStr} ${content}`;
      const ok = await writeAppend('07 Quick Capture & Hub/每日碎片化靈感隨記.md', line);
      if (ok) {
        showToast(`✅ 已捕獲 [${cat}]`);
        notifyDC('快速捕獲', `💡 [${cat}] ${content}`);
        taFrag.value = '';
        // 即時新增到清單最頂
        const ni = fragList.createEl('div', { cls: 'db-list-item', attr: { style: 'font-size:12px; align-items:flex-start;' } });
        el(ni, 'span', { attr: { style: 'color:var(--db-teal);font-size:10px;min-width:36px;flex-shrink:0;' }, text: `[${cat}]` });
        el(ni, 'span', { text: content.length > 50 ? content.slice(0,50)+'…' : content });
        fragList.insertBefore(ni, fragList.firstChild);
      } else {
        showToast('⚠️ 寫入失敗', 'error');
      }
    });

    // 快速鍵：Ctrl+Enter 送出
    taFrag.addEventListener('keydown', e => { if ((e.ctrlKey || e.metaKey) && e.key === 'Enter') btnFrag.click(); });

    // 最近碎片清單（收合式，預設收合避免佔版面）
    const fragHdr = el(card, 'div', { attr: { style: 'display:flex;align-items:center;justify-content:space-between;cursor:pointer;user-select:none;margin-bottom:6px;' } });
    el(fragHdr, 'div', { attr: { style: 'font-size:10px;color:var(--db-text-muted);text-transform:uppercase;letter-spacing:.08em;' }, text: `最近捕獲（${_recentFrags.length} 則）` });
    const fragChev = el(fragHdr, 'div', { attr: { style: 'font-size:12px;color:var(--db-text-muted);' }, text: '▸' });
    const fragWrap = card.createEl('div', { attr: { style: 'display:none;' } });
    fragHdr.addEventListener('click', () => {
      const open = fragWrap.style.display !== 'none';
      fragWrap.style.display = open ? 'none' : 'block';
      fragChev.textContent   = open ? '▸' : '▾';
    });
    const fragList = el(fragWrap, 'div', { cls: 'db-card-body' });
    if (_recentFrags.length === 0) {
      el(fragList, 'div', { cls: 'db-empty', text: '還沒有碎片' });
    } else {
      _recentFrags.forEach(raw => {
        const m = raw.match(/^-\s*(?:\[([^\]]+)\])?\s*(\d{4}-\d{2}-\d{2}\s+\d{2}:\d{2})?\s*(.+)/);
        if (!m) return;
        const [,cat,ts,text] = m;
        const item = el(fragList, 'div', { cls: 'db-list-item', attr: { style: 'font-size:12px; align-items:flex-start;' } });
        if (cat) el(item, 'span', { attr: { style: 'color:var(--db-teal);font-size:10px;min-width:52px;flex-shrink:0;' }, text: `[${cat}]` });
        el(item, 'span', { text: (text||'').trim().slice(0,50) + ((text||'').trim().length > 50 ? '…' : '') });
      });
    }
  }

  // ── 右卡：靈感圖書館（只放蒸餾過的知識，收合式，預設收合）──
  {
    const card = row.createEl('div', { cls: 'db-card' });
    const hdr = el(card, 'div', { cls: 'db-card-header', attr: { style: 'cursor:pointer;user-select:none;' } });
    el(hdr, 'div', { cls: 'db-card-title', text: '📚  靈感圖書館' });
    el(hdr, 'div', { cls: 'db-card-action', text: '→ 整理 Inbox' }).addEventListener('click', (e) => { e.stopPropagation(); openPage('快速捕獲中樞'); });
    const libChev = el(hdr, 'div', { attr: { style: 'font-size:13px;color:var(--db-text-muted);margin-left:8px;' }, text: '▸' });
    const libWrap = card.createEl('div', { attr: { style: 'display:none;' } });
    hdr.addEventListener('click', () => {
      const open = libWrap.style.display !== 'none';
      libWrap.style.display = open ? 'none' : 'block';
      libChev.textContent   = open ? '▸' : '▾';
    });

    el(libWrap, 'div', { attr: { style: 'font-size:11px;color:var(--db-text-muted);margin-bottom:12px;' }, text: '已蒸餾的知識與策略，不是原始紀錄（原始碎片見側邊欄「快速捕獲」）' });

    const hubs = [
      { icon: '✍️', label: '發文靈感池',    page: '01 Brand/發文靈感池',                        sub: '已分類可用的發文素材' },
      { icon: '🧠', label: '概念卡',        page: '08 Skills 方法庫/README',                    sub: '跨領域心法（demo>claim/90%法則等6張）' },
      { icon: '📌', label: '品牌中樞當前重點', page: '01 Brand/品牌中樞',                        sub: '定位/策略/顧問建議蒸餾結論' },
      { icon: '🦦', label: '撥撥獺獺 IP',    page: '04 Projects/撥撥獺獺IP',                     sub: '角色設定+五條主力內容線' },
      { icon: '📊', label: '個股總覽',      page: '03 Invest-Swing/個股總覽',                   sub: '持倉+估值+模型+研究聚合' },
      { icon: '💼', label: '接案執行庫',    page: '01 Brand/接案執行庫',                        sub: '報價規律+實戰紀錄' },
    ];

    hubs.forEach(({ icon, label, page, sub }) => {
      const item = el(libWrap, 'div', { cls: 'db-list-item', attr: { style: 'cursor:pointer; padding:9px 6px; flex-direction:column; align-items:flex-start; gap:2px;' } });
      const top = el(item, 'div', { attr: { style: 'display:flex; align-items:center; gap:8px; width:100%;' } });
      el(top, 'span', { text: icon });
      el(top, 'span', { attr: { style: 'font-weight:500; font-size:13px; color:var(--db-text-primary);' }, text: label });
      el(top, 'span', { attr: { style: 'margin-left:auto; font-size:10px; color:var(--db-text-muted);' }, text: '→' });
      el(item, 'div', { attr: { style: 'font-size:11px; color:var(--db-text-muted); padding-left:22px;' }, text: sub });
      item.addEventListener('click', () => openPage(page));
      item.addEventListener('mouseenter', () => item.style.background = 'rgba(139,123,173,0.08)');
      item.addEventListener('mouseleave', () => item.style.background = '');
    });
  }
}

// 第五排：工作流（兩欄）— 每週才看
{
  const row = weeklyBody.createEl('div', { cls: 'db-grid db-grid-2col db-card-enter', attr: { style: 'margin-bottom: var(--db-gap);' } });

  // ── 左卡：個人品牌 ──
  {
    const card = row.createEl('div', { cls: 'db-card' });
    const hdr = el(card, 'div', { cls: 'db-card-header' });
    el(hdr, 'div', { cls: 'db-card-title', text: '✦  個人品牌' });
    el(hdr, 'div', { cls: 'db-card-action', text: '→ 品牌中樞' }).addEventListener('click', () => openPage('品牌中樞'));

    // 快速入口列
    const btnRow = el(card, 'div', { attr: { style: 'display:flex;flex-wrap:wrap;gap:6px;margin-bottom:14px;' } });
    const brandBtns = [
      { label: '✍️ 發文日常', page: '品牌中樞' },
      { label: '📊 內容計畫', page: '01 Brand/社群數據/content_plan' },
      { label: '📈 戰績表', page: '01 Brand/社群數據/history' },
      { label: '💡 靈感池', page: '01 Brand/發文靈感池' },
      { label: '📰 Brief', page: `01 Brand/Brief ${_todayStr.slice(0,10)}` },
    ];
    brandBtns.forEach(({ label, page }) => {
      const b = el(btnRow, 'button', { cls: 'db-btn db-btn-ghost db-btn-sm', text: label });
      b.addEventListener('click', () => openPage(page));
    });

    // 本週完成率
    const progRow = el(card, 'div', { attr: { style: 'display:flex;align-items:center;gap:10px;padding:10px 12px;background:var(--db-bg-deep);border-radius:var(--db-radius-inner);margin-bottom:12px;' } });
    el(progRow, 'span', { attr: { style: 'font-size:11px;color:var(--db-text-muted);flex:1;' }, text: '本週發文完成率' });
    const progBar = el(progRow, 'div', { attr: { style: 'flex:2;height:6px;background:var(--db-border);border-radius:3px;overflow:hidden;' } });
    const pct = weekTotal > 0 ? Math.round(weekDone/weekTotal*100) : 0;
    progBar.createEl('div', { attr: { style: `height:100%;width:${pct}%;background:var(--db-teal);border-radius:3px;transition:width .4s;` } });
    el(progRow, 'span', { attr: { style: 'font-size:12px;color:var(--db-cream);font-weight:600;min-width:36px;text-align:right;' }, text: weekPct });

    // 即將上線貼文
    el(card, 'div', { attr: { style: 'font-size:10px;color:var(--db-text-muted);text-transform:uppercase;letter-spacing:.08em;margin-bottom:6px;' }, text: '近期預定貼文' });
    const postList = el(card, 'div', { cls: 'db-card-body' });
    if (_upcomingPosts.length === 0) {
      el(postList, 'div', { cls: 'db-empty', text: '無預定貼文' });
    } else {
      _upcomingPosts.slice(0, 5).forEach(p => {
        const item = el(postList, 'div', { cls: 'db-list-item' });
        el(item, 'span', { attr: { style: 'font-size:10px;color:var(--db-purple);min-width:64px;' }, text: p.date });
        el(item, 'span', { attr: { style: 'flex:1;font-size:12px;' }, text: (p.topic||'—').slice(0,30) });
      });
    }
  }

  // ── 右卡：模特主持工作 ──
  {
    const card = row.createEl('div', { cls: 'db-card' });
    const hdr = el(card, 'div', { cls: 'db-card-header' });
    el(hdr, 'div', { cls: 'db-card-title', text: '🎬  模特主持工作' });
    el(hdr, 'div', { cls: 'db-card-action', text: '→ 接案執行庫' }).addEventListener('click', () => openPage('接案執行庫'));

    // 快速入口
    const wBtnRow = el(card, 'div', { attr: { style: 'display:flex;flex-wrap:wrap;gap:6px;margin-bottom:14px;' } });
    [
      { label: '📋 接案紀錄', page: '01 Brand/接案執行庫' },
      { label: '💃 COS 經營', page: '01 Brand/COS經營' },
      { label: '🎙 主持直播', page: '01 Brand/主持與直播執行庫' },
      { label: '✏️ 部落格', page: '04 Projects/部落格' },
    ].forEach(({ label, page }) => {
      const b = el(wBtnRow, 'button', { cls: 'db-btn db-btn-ghost db-btn-sm', text: label });
      b.addEventListener('click', () => openPage(page));
    });

    // 即將接案提示
    el(card, 'div', { attr: { style: 'font-size:10px;color:var(--db-text-muted);text-transform:uppercase;letter-spacing:.08em;margin-bottom:6px;' }, text: '近期接案 / 截止' });
    const workList = el(card, 'div', { cls: 'db-card-body' });
    const upcoming = [
      { date: '7/3-4', item: '成人展（品牌發文素材）', type: 'SG' },
      { date: '7/18', item: '蝦皮帶貨首播', type: '直播' },
    ];
    upcoming.forEach(({ date, item, type }) => {
      const row2 = el(workList, 'div', { cls: 'db-list-item' });
      el(row2, 'span', { attr: { style: 'font-size:10px;color:var(--db-rose);min-width:52px;font-weight:600;' }, text: date });
      el(row2, 'span', { attr: { style: 'flex:1;font-size:12px;' }, text: item });
      el(row2, 'span', { cls: 'db-badge', attr: { style: 'font-size:9px;' }, text: type });
    });

    // 報價快查
    el(card, 'div', { attr: { style: 'margin-top:10px;padding-top:10px;border-top:1px solid var(--db-border);font-size:10px;color:var(--db-text-muted);text-transform:uppercase;letter-spacing:.08em;margin-bottom:6px;' }, text: '報價參考' });
    const quotes = [
      ['SG 展場（半天）', '3,000–6,000'],
      ['主持（半天）', '5,000–15,000'],
      ['模特外拍（2hr）', '2,000–5,000'],
    ];
    quotes.forEach(([type, range]) => {
      const qRow = el(card, 'div', { attr: { style: 'display:flex;justify-content:space-between;padding:5px 0;border-bottom:1px solid var(--db-border);font-size:11px;' } });
      el(qRow, 'span', { attr: { style: 'color:var(--db-text-secondary);' }, text: type });
      el(qRow, 'span', { attr: { style: 'color:var(--db-cream);' }, text: `NT$ ${range}` });
    });
  }
}

// 第六排：人生資料庫（兩欄）— 每週才看
{
  const row = weeklyBody.createEl('div', { cls: 'db-grid db-grid-2col db-card-enter', attr: { style: 'margin-bottom: var(--db-gap);' } });

  // ── 左卡：自律習慣 & 生活靈感 ──
  {
    const card = row.createEl('div', { cls: 'db-card' });
    const hdr = el(card, 'div', { cls: 'db-card-header' });
    el(hdr, 'div', { cls: 'db-card-title', text: '🌱  自律習慣 & 生活靈感' });
    el(hdr, 'div', { cls: 'db-card-action', text: '→ 生活中樞' }).addEventListener('click', () => openPage('生活自我成長中樞'));

    const liBtn = el(card, 'div', { attr: { style: 'display:flex;flex-wrap:wrap;gap:6px;margin-bottom:12px;' } });
    [['📋 自律計劃表', '06 Lifestyle & Self-Growth/自律計劃表'], ['📒 自律日誌', '06 Lifestyle & Self-Growth/自律日誌'], ['🌿 生活成長中樞', '生活自我成長中樞'], ['🛠 想學技能', '06 Lifestyle & Self-Growth/想學習的技能清單']].forEach(([l, p]) => {
      const b = el(liBtn, 'button', { cls: 'db-btn db-btn-ghost db-btn-sm', text: l });
      b.addEventListener('click', () => openPage(p));
    });

    // T2/T3 目標
    el(card, 'div', { attr: { style: 'font-size:10px;color:var(--db-text-muted);text-transform:uppercase;letter-spacing:.08em;margin-bottom:6px;' }, text: '近期生活目標（T2 有截止日）' });
    const goalList = el(card, 'div', { cls: 'db-card-body' });
    if (_goals.length === 0) {
      el(goalList, 'div', { cls: 'db-empty', text: '讀取中或無資料' });
    } else {
      _goals.forEach(({ done, text }) => {
        const item = el(goalList, 'div', { cls: 'db-list-item', attr: { style: 'font-size:12px;align-items:flex-start;' } });
        el(item, 'span', { attr: { style: 'color:var(--db-rose);flex-shrink:0;margin-top:1px;' }, text: done ? '✓' : '●' });
        el(item, 'span', { attr: { style: done ? 'text-decoration:line-through;color:var(--db-text-muted);' : '' }, text });
      });
    }

    // 醫美存款進度
    const noseRow = el(card, 'div', { attr: { style: 'margin-top:10px;padding:10px 12px;background:var(--db-bg-deep);border-radius:var(--db-radius-inner);' } });
    el(noseRow, 'div', { attr: { style: 'font-size:11px;color:var(--db-text-muted);margin-bottom:6px;' }, text: '💉 隆鼻基金進度（目標 NT$88,000）' });
    const noseBar = el(noseRow, 'div', { attr: { style: 'height:6px;background:var(--db-border);border-radius:3px;overflow:hidden;margin-bottom:4px;' } });
    noseBar.createEl('div', { attr: { style: 'height:100%;width:75%;background:var(--db-rose);border-radius:3px;' } });
    el(noseRow, 'div', { attr: { style: 'font-size:11px;color:var(--db-text-muted);' }, text: '已存 NT$66,000 / 差 NT$22,000' });
  }

  // ── 右卡：人生目標 & 課程學習 ──
  {
    const card = row.createEl('div', { cls: 'db-card' });
    const hdr = el(card, 'div', { cls: 'db-card-header' });
    el(hdr, 'div', { cls: 'db-card-title', text: '🌟  人生目標 & 學習' });
    el(hdr, 'div', { cls: 'db-card-action', text: '→ 技能清單' }).addEventListener('click', () => openPage('06 Lifestyle & Self-Growth/想學習的技能清單'));

    const rBtn = el(card, 'div', { attr: { style: 'display:flex;flex-wrap:wrap;gap:6px;margin-bottom:12px;' } });
    [['💰 投資中樞', '投資中樞'], ['🌊 波段中樞', '波段中樞'], ['🏠 房屋規劃', '生活自我成長中樞'], ['📚 知識庫', '02 Invest-LT/投資知識庫']].forEach(([l, p]) => {
      const b = el(rBtn, 'button', { cls: 'db-btn db-btn-ghost db-btn-sm', text: l });
      b.addEventListener('click', () => openPage(p));
    });

    el(card, 'div', { attr: { style: 'font-size:10px;color:var(--db-text-muted);text-transform:uppercase;letter-spacing:.08em;margin-bottom:6px;' }, text: '長期目標追蹤' });
    const ltGoals = [
      { icon: '🏠', label: '買房基金', sub: '房屋獨立帳戶規劃中，目標每月 15-20K', tag: '進行中' },
      { icon: '💹', label: '亞翔減碼計畫', sub: '逼近 NT$1,000 才分批減碼 → 轉 00991A', tag: '監控中' },
      { icon: '🇬🇧', label: '英文 Effortless 綁定法', sub: '唯一學習中技能', tag: '進行中' },
      { icon: '📊', label: 'AI 集中度控制', sub: 'AI ETF ≤35%（現況：00991A 入場中）', tag: '規則' },
    ];
    const gList = el(card, 'div', { cls: 'db-card-body' });
    ltGoals.forEach(({ icon, label, sub, tag }) => {
      const item = el(gList, 'div', { cls: 'db-list-item', attr: { style: 'flex-direction:column;align-items:flex-start;gap:2px;padding:8px 0;' } });
      const top = el(item, 'div', { attr: { style: 'display:flex;align-items:center;gap:7px;width:100%;' } });
      el(top, 'span', { text: icon });
      el(top, 'span', { attr: { style: 'font-size:13px;font-weight:500;flex:1;' }, text: label });
      el(top, 'span', { cls: 'db-badge', attr: { style: 'font-size:9px;' }, text: tag });
      el(item, 'div', { attr: { style: 'font-size:11px;color:var(--db-text-muted);padding-left:22px;' }, text: sub });
    });
  }
}

// ── Step 8 資料：資產 ──────────────────────────────────────────
let _swingRows = [];
try {
  const _sf = app.vault.getAbstractFileByPath('03 Invest-Swing/波段庫存.md');
  if (_sf) {
    const _sc = await app.vault.read(_sf);
    for (const ln of _sc.split('\n')) {
      if (!/^\|\s*\d{4}\s*\|/.test(ln)) continue;
      const cols = ln.split('|').map(c => c.trim()).filter(Boolean);
      if (cols.length >= 6) _swingRows.push({ code: cols[0], name: cols[1], shares: cols[2], cost: cols[3], price: cols[4], pnl: cols[5], status: cols[6]||'', note: cols[7]||'' });
    }
  }
} catch(e) {}

// 個人資產（每週才看）
{
  const card = weeklyBody.createEl('div', { cls: 'db-card db-card-enter', attr: { style: 'margin-bottom: var(--db-gap);' } });
  const hdr = el(card, 'div', { cls: 'db-card-header' });
  el(hdr, 'div', { cls: 'db-card-title', text: '💰  個人資產' });
  el(hdr, 'div', { cls: 'db-card-action', text: '→ 投資中樞' }).addEventListener('click', () => openPage('投資中樞'));

  if (!window.__dbTabState) window.__dbTabState = {};
  const _ltTabActive = window.__dbTabState.asset !== 'sw';
  const tabs = el(card, 'div', { cls: 'db-tabs' });
  const tabLT = el(tabs, 'div', { cls: 'db-tab' + (_ltTabActive ? ' active' : ''), text: '長期資產（DCA）' });
  const tabSW = el(tabs, 'div', { cls: 'db-tab' + (_ltTabActive ? '' : ' active'), text: '波段庫存' });

  const panelLT = el(card, 'div', { cls: 'db-tab-panel' + (_ltTabActive ? ' active' : '') });
  const panelSW = el(card, 'div', { cls: 'db-tab-panel' + (_ltTabActive ? '' : ' active') });

  // ── 長期資產 Tab ──
  const ltGrid = panelLT.createEl('div', { attr: { style: 'display:grid;grid-template-columns:1fr 1fr;gap:var(--db-gap);' } });

  const ltLeft = ltGrid.createEl('div');
  el(ltLeft, 'div', { attr: { style: 'font-size:11px;color:var(--db-text-muted);text-transform:uppercase;letter-spacing:.08em;margin-bottom:8px;' }, text: '總資產概況' });
  const ltStat = el(ltLeft, 'div', { attr: { style: 'display:grid;grid-template-columns:1fr 1fr;gap:10px;margin-bottom:12px;' } });
  [['~269.5萬', '帳面總資產'], ['60萬', '緊急備用金（不動）']].forEach(([v,l]) => {
    const b = ltStat.createEl('div', { cls: 'db-stat-card', attr: { style: 'padding:12px 14px;' } });
    el(b, 'div', { cls: 'db-stat-label', text: l });
    el(b, 'div', { cls: 'db-stat-value', attr: { style: 'font-size:20px;' }, text: v });
  });
  el(ltLeft, 'div', { attr: { style: 'font-size:11px;color:var(--db-text-muted);margin-bottom:4px;' }, text: '下次主復盤：2026-12' });
  el(ltLeft, 'div', { attr: { style: 'font-size:11px;color:var(--db-text-muted);' }, text: 'ETF 獲利 >100% 可局部賣 ≤20萬' });

  const ltRight = ltGrid.createEl('div');
  el(ltRight, 'div', { attr: { style: 'font-size:11px;color:var(--db-text-muted);text-transform:uppercase;letter-spacing:.08em;margin-bottom:8px;' }, text: '每月 DCA（台股 18,000/月）' });
  const dcaItems = [
    { code: '0050',   name: '元大台灣50',        amount: 'NT$ 8,000', tag: '台股' },
    { code: '00922',  name: '國泰台灣領袖50',     amount: 'NT$ 3,000', tag: '台股' },
    { code: '00991A', name: '中信AI龍頭（累積型）', amount: 'NT$ 3,000', tag: '台股' },
    { code: '2330',   name: '台積電零股',         amount: 'NT$ 4,000', tag: '台股' },
    { code: 'VOO',    name: 'Vanguard S&P 500',   amount: 'USD 200',  tag: '美股' },
    { code: 'NVDA',   name: 'NVIDIA',             amount: 'USD 100',  tag: '美股' },
  ];
  let _lastTag = '';
  dcaItems.forEach(({ code, name, amount, tag }) => {
    if (tag !== _lastTag) {
      _lastTag = tag;
      el(ltRight, 'div', { attr: { style: 'font-size:10px;color:var(--db-text-muted);text-transform:uppercase;letter-spacing:.08em;margin:8px 0 3px;border-top:1px solid var(--db-border);padding-top:6px;' }, text: tag === '美股' ? '美股 USD 300/月' : '台股 NT$18,000/月' });
    }
    const row = el(ltRight, 'div', { cls: 'db-list-item' });
    el(row, 'span', { attr: { style: 'font-size:11px;color:var(--db-purple);min-width:58px;font-weight:600;' }, text: code });
    el(row, 'span', { attr: { style: 'flex:1;' }, text: name });
    el(row, 'span', { attr: { style: 'font-size:11px;color:var(--db-teal);' }, text: amount });
  });

  // ── 波段庫存 Tab ──
  el(panelSW, 'div', { attr: { style: 'font-size:11px;color:var(--db-text-muted);margin-bottom:8px;' }, text: '用 /庫存健檢 取最新現價；買賣後在 #波段 更新' });
  if (_swingRows.length === 0) {
    el(panelSW, 'div', { cls: 'db-empty', text: '讀取失敗或無持倉' });
  } else {
    const tbl = panelSW.createEl('div', { attr: { style: 'overflow-x:auto;' } });
    const thead = el(tbl, 'div', { attr: { style: 'display:grid;grid-template-columns:56px 80px 56px 64px 64px 80px 80px;gap:8px;padding:6px 4px;border-bottom:1px solid var(--db-border);font-size:10px;color:var(--db-text-muted);text-transform:uppercase;letter-spacing:.06em;' } });
    ['代號','名稱','股數','均價','現價','未實現','狀態'].forEach(h => el(thead, 'span', { text: h }));
    _swingRows.forEach(r => {
      const row = tbl.createEl('div', { attr: { style: 'display:grid;grid-template-columns:56px 80px 56px 64px 64px 80px 80px;gap:8px;padding:7px 4px;border-bottom:1px solid var(--db-border);font-size:12px;align-items:center;' } });
      el(row, 'span', { attr: { style: 'color:var(--db-purple);font-weight:600;' }, text: r.code });
      el(row, 'span', { text: r.name });
      el(row, 'span', { attr: { style: 'color:var(--db-text-secondary);' }, text: r.shares });
      el(row, 'span', { text: r.cost });
      el(row, 'span', { attr: { style: 'color:var(--db-text-muted);' }, text: r.price || '—' });
      const pnlColor = r.pnl && r.pnl.startsWith('+') ? 'var(--db-teal)' : r.pnl && r.pnl.startsWith('-') ? 'var(--db-priority-high)' : 'var(--db-text-muted)';
      el(row, 'span', { attr: { style: `color:${pnlColor};font-weight:600;` }, text: r.pnl || '—' });
      el(row, 'span', { attr: { style: 'font-size:10px;color:var(--db-text-muted);' }, text: r.status });
    });
  }
  el(panelSW, 'div', { attr: { style: 'margin-top:10px;font-size:11px;color:var(--db-text-muted);' }, text: `活存：NT$ 53,609｜已投入本金：NT$ 327,017` });

  tabLT.addEventListener('click', () => { window.__dbTabState.asset = 'lt'; tabLT.classList.add('active'); tabSW.classList.remove('active'); panelLT.classList.add('active'); panelSW.classList.remove('active'); });
  tabSW.addEventListener('click', () => { window.__dbTabState.asset = 'sw'; tabSW.classList.add('active'); tabLT.classList.remove('active'); panelSW.classList.add('active'); panelLT.classList.remove('active'); });
}

// 第七排：所有待辦（全寬，預設收合）
{
  const card = main.createEl('div', { cls: 'db-card db-card-enter db-todos-section', attr: { style: 'margin-bottom: var(--db-gap);' } });
  const hdr = el(card, 'div', { cls: 'db-card-header', attr: { style: 'cursor:pointer;user-select:none;' } });
  el(hdr, 'div', { cls: 'db-card-title', text: '☑️  所有待辦事項' });
  const todoChev = el(hdr, 'div', { attr: { style: 'font-size:13px;color:var(--db-text-muted);' }, text: '▸' });
  const todoBodyWrap = card.createEl('div', { attr: { style: 'display:none;' } });
  hdr.addEventListener('click', () => {
    const open = todoBodyWrap.style.display !== 'none';
    todoBodyWrap.style.display = open ? 'none' : 'block';
    todoChev.textContent = open ? '▸' : '▾';
  });

  // 篩選器
  const filterRow = el(todoBodyWrap, 'div', { attr: { style: 'display:flex;gap:8px;margin-bottom:12px;align-items:center;flex-wrap:wrap;' } });
  const searchInp = filterRow.createEl('input', { cls: 'db-input', attr: { placeholder: '🔍 搜尋待辦...', style: 'max-width:200px;padding:6px 10px;' } });
  const areaFilter = filterRow.createEl('select', { cls: 'db-select', attr: { style: 'max-width:130px;' } });
  ['全部', '品牌', '投資', '波段', '生活', '專案'].forEach(a => areaFilter.createEl('option', { text: a, value: a === '全部' ? '' : a }));
  const countBadge = el(filterRow, 'span', { cls: 'db-badge', attr: { style: 'margin-left:auto;' }, text: `${pendingCount} 項未完成` });

  // 取所有未完成任務（含檔案來源）
  const allTasks = [];
  for (const p of dv.pages().array()) {
    for (const t of p.file.tasks.where(t2 => !t2.completed).array()) {
      allTasks.push({ text: String(t.text||''), file: p.file.name, path: p.file.path });
    }
  }
  allTasks.sort((a,b) => a.file.localeCompare(b.file));

  // 渲染列表
  const todoBody = el(todoBodyWrap, 'div', { cls: 'db-card-body db-card-body-tall' });
  const renderTodos = (q, area) => {
    todoBody.empty();
    const filtered = allTasks.filter(t => {
      const matchQ = !q || t.text.toLowerCase().includes(q.toLowerCase());
      const matchA = !area || t.file.toLowerCase().includes(area.toLowerCase()) || t.path.toLowerCase().includes(area.toLowerCase());
      return matchQ && matchA;
    });
    if (filtered.length === 0) {
      el(todoBody, 'div', { cls: 'db-empty', text: q || area ? '沒有符合條件的待辦' : '🎉 全部完成！' });
      return;
    }
    let lastFile = '';
    filtered.forEach(t => {
      if (t.file !== lastFile) {
        lastFile = t.file;
        el(todoBody, 'div', { attr: { style: 'font-size:10px;color:var(--db-text-muted);text-transform:uppercase;letter-spacing:.06em;padding:8px 0 3px;margin-top:4px;border-top:1px solid var(--db-border);' }, text: t.file });
      }
      const item = el(todoBody, 'div', { cls: 'db-list-item', attr: { style: 'gap:6px;font-size:12px;cursor:pointer;' } });
      el(item, 'span', { attr: { style: 'width:12px;height:12px;border:1.5px solid var(--db-gray-purple);border-radius:3px;flex-shrink:0;' } });
      el(item, 'span', { text: t.text });
      item.addEventListener('click', () => openPage(t.path.replace('.md','')));
    });
    countBadge.textContent = `${filtered.length} 項`;
  }
  renderTodos('', '');
  let _debounce;
  searchInp.addEventListener('input', () => { clearTimeout(_debounce); _debounce = setTimeout(() => renderTodos(searchInp.value, areaFilter.value), 180); });
  areaFilter.addEventListener('change', () => renderTodos(searchInp.value, areaFilter.value));
}

// ── 📥 每週才看（本週目標／工作流／人生資料庫／個人資產，統一收合掛到最底部）──
{
  const weeklySection = makeCollapsible(main, '📥  每週才看（近期待辦／工作流／人生資料庫／個人資產）', false);
  weeklySection.appendChild(weeklyBody);
}
```
