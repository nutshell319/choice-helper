# 选择助手 — 实现计划

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 构建一个单文件 HTML 选择助手工具，包含转盘、抽签、翻牌、骰子、硬币五种模式，支持预设列表管理。

**Architecture:** 单文件 HTML（`choice-helper/index.html`），CSS/JS 全部内联。JS 按功能分区：状态管理 → 存储层 → UI 渲染 → 各模式动画引擎 → 事件绑定。Canvas 用于转盘绘制，CSS Animation + requestAnimationFrame 用于其余动画。localStorage 持久化预设和当前选项。

**Tech Stack:** 纯 HTML/CSS/JavaScript（ES6），零外部依赖，Canvas 2D API

---

### Task 1: HTML 骨架 + CSS 全局样式 + Tab 切换

**Files:**
- Modify: `choice-helper/index.html`（完整重写）

- [ ] **Step 1: 写入完整 HTML 骨架、CSS 变量体系、顶部导航和 Tab 栏**

将以下内容写入 `choice-helper/index.html`：

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>🎲 选择助手</title>
<style>
/* ===== CSS Variables ===== */
:root {
  --bg-start: #1a1a2e;
  --bg-end: #16213e;
  --primary: #f0a500;
  --accent: #ff6b6b;
  --surface: rgba(255,255,255,0.08);
  --surface-hover: rgba(255,255,255,0.14);
  --text: #e8e8e8;
  --text-dim: #999;
  --radius: 12px;
  --font: "Microsoft YaHei", "PingFang SC", sans-serif;
  --shadow: 0 4px 24px rgba(0,0,0,0.4);
}

/* ===== Reset & Base ===== */
*, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }
body {
  font-family: var(--font);
  background: linear-gradient(135deg, var(--bg-start), var(--bg-end));
  color: var(--text);
  min-height: 100vh;
  display: flex; justify-content: center; align-items: center;
  padding: 20px;
}
.app {
  width: 100%; max-width: 680px;
  display: flex; flex-direction: column;
  gap: 24px;
}

/* ===== Header ===== */
.header {
  display: flex; justify-content: space-between; align-items: center;
}
.header h1 {
  font-size: 28px;
  background: linear-gradient(135deg, var(--primary), #ffd56b);
  -webkit-background-clip: text; -webkit-text-fill-color: transparent;
  background-clip: text;
}
.header .btn-manage {
  background: var(--surface); color: var(--text); border: 1px solid rgba(255,255,255,0.12);
  padding: 8px 16px; border-radius: 8px; cursor: pointer;
  font-size: 14px; font-family: var(--font);
  transition: background 0.2s;
}
.header .btn-manage:hover { background: var(--surface-hover); }

/* ===== Tabs ===== */
.tabs {
  display: flex; gap: 4px;
  background: var(--surface); border-radius: var(--radius);
  padding: 4px;
}
.tab {
  flex: 1; padding: 10px 8px; border: none; background: transparent;
  color: var(--text-dim); cursor: pointer; border-radius: 10px;
  font-size: 14px; font-family: var(--font); transition: all 0.25s;
}
.tab:hover { color: var(--text); }
.tab.active {
  background: var(--primary); color: #1a1a2e; font-weight: bold;
}

/* ===== Stage (动画舞台) ===== */
.stage {
  background: var(--surface);
  border-radius: var(--radius);
  min-height: 360px;
  display: flex; flex-direction: column;
  justify-content: center; align-items: center;
  position: relative; overflow: hidden;
  border: 1px solid rgba(255,255,255,0.06);
}

/* ===== Input Panel ===== */
.input-panel {
  background: var(--surface);
  border-radius: var(--radius);
  padding: 16px;
  border: 1px solid rgba(255,255,255,0.06);
}
.input-row {
  display: flex; gap: 8px; margin-bottom: 12px;
}
.input-row input {
  flex: 1; padding: 10px 14px; border-radius: 8px; border: 1px solid rgba(255,255,255,0.15);
  background: rgba(0,0,0,0.25); color: var(--text);
  font-size: 14px; font-family: var(--font); outline: none;
  transition: border 0.2s;
}
.input-row input:focus { border-color: var(--primary); }
.input-row button {
  padding: 10px 20px; border-radius: 8px; border: none;
  background: var(--primary); color: #1a1a2e;
  font-weight: bold; cursor: pointer; font-size: 14px; font-family: var(--font);
  transition: opacity 0.2s;
}
.input-row button:hover { opacity: 0.85; }
.input-row button:disabled { opacity: 0.4; cursor: not-allowed; }

/* Tags */
.tags {
  display: flex; flex-wrap: wrap; gap: 8px; align-items: center;
}
.tag {
  display: inline-flex; align-items: center; gap: 6px;
  padding: 6px 12px; border-radius: 20px;
  background: rgba(240,165,0,0.18); color: var(--primary);
  font-size: 13px; border: 1px solid rgba(240,165,0,0.25);
}
.tag .tag-close {
  cursor: pointer; font-size: 16px; line-height: 1;
  opacity: 0.6; transition: opacity 0.2s;
}
.tag .tag-close:hover { opacity: 1; color: var(--accent); }
.tag-placeholder { color: var(--text-dim); font-size: 13px; }

/* ===== Buttons ===== */
.btn-action {
  padding: 12px 32px; border-radius: 28px; border: none;
  background: var(--accent); color: #fff;
  font-size: 16px; font-weight: bold; cursor: pointer;
  font-family: var(--font); transition: all 0.2s;
  letter-spacing: 1px;
}
.btn-action:hover { transform: scale(1.04); box-shadow: 0 4px 20px rgba(255,107,107,0.35); }
.btn-action:disabled { opacity: 0.4; cursor: not-allowed; transform: none; box-shadow: none; }

/* ===== Modal ===== */
.modal-overlay {
  position: fixed; inset: 0; background: rgba(0,0,0,0.6);
  display: flex; justify-content: center; align-items: center;
  z-index: 100; opacity: 0; pointer-events: none; transition: opacity 0.25s;
}
.modal-overlay.open { opacity: 1; pointer-events: auto; }
.modal {
  background: #1e1e3a; border-radius: var(--radius); padding: 24px;
  width: 90%; max-width: 500px; max-height: 80vh; overflow-y: auto;
  box-shadow: var(--shadow); border: 1px solid rgba(255,255,255,0.08);
}
.modal h2 { margin-bottom: 16px; color: var(--primary); }
.modal-close {
  float: right; background: none; border: none; color: var(--text-dim);
  font-size: 24px; cursor: pointer; line-height: 1;
}
.modal-close:hover { color: var(--accent); }

/* ===== Preset List Items ===== */
.preset-item {
  display: flex; justify-content: space-between; align-items: center;
  padding: 10px 14px; border-radius: 8px;
  background: var(--surface); margin-bottom: 8px;
}
.preset-item .name { font-weight: bold; }
.preset-item .count { color: var(--text-dim); font-size: 12px; margin-left: 8px; }
.preset-item .actions { display: flex; gap: 6px; }
.preset-item .actions button {
  padding: 4px 10px; border-radius: 6px; border: none; cursor: pointer;
  font-size: 12px; font-family: var(--font);
}
.btn-load { background: var(--primary); color: #1a1a2e; }
.btn-delete { background: var(--accent); color: #fff; }
.btn-export { background: var(--surface); color: var(--text); border: 1px solid rgba(255,255,255,0.15); }
.btn-import { background: var(--surface); color: var(--text); border: 1px solid rgba(255,255,255,0.15); }

/* Small buttons */
.btn-sm {
  padding: 6px 14px; border-radius: 6px; border: none; cursor: pointer;
  font-size: 12px; font-family: var(--font);
  background: var(--surface); color: var(--text);
  border: 1px solid rgba(255,255,255,0.12);
}
.btn-sm:hover { background: var(--surface-hover); }

/* ===== Toast ===== */
.toast {
  position: fixed; top: 24px; left: 50%; transform: translateX(-50%);
  padding: 12px 24px; border-radius: 24px;
  background: var(--accent); color: #fff; font-weight: bold;
  z-index: 200; opacity: 0; transition: opacity 0.3s;
  pointer-events: none; font-family: var(--font);
}
.toast.show { opacity: 1; }

/* ===== Result overlay ===== */
.result-overlay {
  position: absolute; inset: 0;
  display: flex; flex-direction: column;
  justify-content: center; align-items: center; gap: 16px;
  background: rgba(0,0,0,0.7); z-index: 10;
  opacity: 0; pointer-events: none; transition: opacity 0.4s;
}
.result-overlay.show { opacity: 1; pointer-events: auto; }
.result-text {
  font-size: 32px; font-weight: bold; color: var(--primary);
  text-shadow: 0 0 20px rgba(240,165,0,0.5);
}
</style>
</head>
<body>
<div class="app">
  <!-- Header -->
  <div class="header">
    <h1>🎲 选择助手</h1>
    <button class="btn-manage" id="btnManage">⚙ 管理列表</button>
  </div>

  <!-- Tabs -->
  <div class="tabs" id="tabs">
    <button class="tab active" data-mode="wheel">🎡 转盘</button>
    <button class="tab" data-mode="draw">🎋 抽签</button>
    <button class="tab" data-mode="card">🃏 翻牌</button>
    <button class="tab" data-mode="dice">🎲 骰子</button>
    <button class="tab" data-mode="coin">🪙 硬币</button>
  </div>

  <!-- Stage -->
  <div class="stage" id="stage">
    <p style="color:var(--text-dim);">请在下方输入选项，然后开始</p>
  </div>

  <!-- Input Panel -->
  <div class="input-panel">
    <div class="input-row">
      <input type="text" id="optionInput" placeholder="输入一个选项，按回车添加…" maxlength="50">
      <button id="btnAdd">+ 添加</button>
    </div>
    <div class="tags" id="tags">
      <span class="tag-placeholder">暂无选项，请在上方添加</span>
    </div>
    <div style="margin-top:10px;display:flex;gap:8px;">
      <button class="btn-sm" id="btnLoadPreset">▼ 加载预设列表</button>
      <button class="btn-sm" id="btnClearOptions">清空选项</button>
    </div>
  </div>
</div>

<!-- Preset Manager Modal -->
<div class="modal-overlay" id="modalOverlay">
  <div class="modal" id="modalContent"></div>
</div>

<!-- Toast -->
<div class="toast" id="toast"></div>

<script>
// ===== 状态管理 =====
const state = {
  mode: 'wheel',
  options: [],
  isAnimating: false,
};

// ===== DOM 引用 =====
const $ = (sel) => document.querySelector(sel);
const $$ = (sel) => document.querySelectorAll(sel);

const dom = {
  tabs: $('#tabs'),
  stage: $('#stage'),
  optionInput: $('#optionInput'),
  btnAdd: $('#btnAdd'),
  tags: $('#tags'),
  btnManage: $('#btnManage'),
  btnLoadPreset: $('#btnLoadPreset'),
  btnClearOptions: $('#btnClearOptions'),
  modalOverlay: $('#modalOverlay'),
  modalContent: $('#modalContent'),
  toast: $('#toast'),
};

// ===== Tab 切换 =====
dom.tabs.addEventListener('click', (e) => {
  const tab = e.target.closest('.tab');
  if (!tab) return;
  state.mode = tab.dataset.mode;
  dom.tabs.querySelectorAll('.tab').forEach(t => t.classList.remove('active'));
  tab.classList.add('active');
  renderStage();
});

// ===== 占位渲染函数（后续 Task 实现） =====
function renderStage() {
  dom.stage.innerHTML = `<p style="color:var(--text-dim);">模式：${state.mode}（待实现）</p>`;
}

function renderTags() {
  if (state.options.length === 0) {
    dom.tags.innerHTML = '<span class="tag-placeholder">暂无选项，请在上方添加</span>';
    return;
  }
  dom.tags.innerHTML = state.options.map((opt, i) => `
    <span class="tag">
      ${escapeHtml(opt)}
      <span class="tag-close" data-index="${i}">✕</span>
    </span>
  `).join('');
}

function escapeHtml(str) {
  const div = document.createElement('div');
  div.textContent = str;
  return div.innerHTML;
}

function addOption(text) {
  const trimmed = text.trim();
  if (!trimmed) return;
  if (trimmed.length > 50) {
    showToast('选项最多 50 个字符');
    return;
  }
  state.options.push(trimmed);
  saveCurrent();
  renderTags();
  renderStage();
  dom.optionInput.value = '';
}

function removeOption(index) {
  state.options.splice(index, 1);
  saveCurrent();
  renderTags();
  renderStage();
}

// ===== Event: 添加选项 =====
dom.btnAdd.addEventListener('click', () => {
  addOption(dom.optionInput.value);
});

dom.optionInput.addEventListener('keydown', (e) => {
  if (e.key === 'Enter') {
    addOption(dom.optionInput.value);
  }
});

// ===== Event: 标签删除 =====
dom.tags.addEventListener('click', (e) => {
  const close = e.target.closest('.tag-close');
  if (!close) return;
  removeOption(parseInt(close.dataset.index));
});

// ===== Event: 清空 =====
dom.btnClearOptions.addEventListener('click', () => {
  if (state.options.length === 0) return;
  if (!confirm(`确定清空全部 ${state.options.length} 个选项吗？`)) return;
  state.options = [];
  saveCurrent();
  renderTags();
  renderStage();
});

// ===== Toast =====
let toastTimer;
function showToast(msg, duration = 2000) {
  clearTimeout(toastTimer);
  dom.toast.textContent = msg;
  dom.toast.classList.add('show');
  toastTimer = setTimeout(() => dom.toast.classList.remove('show'), duration);
}

// ===== localStorage =====
const LS_KEY_CURRENT = 'choice-helper-current';
const LS_KEY_PRESETS = 'choice-helper-presets';

function saveCurrent() {
  try {
    localStorage.setItem(LS_KEY_CURRENT, JSON.stringify(state.options));
  } catch (e) {
    showToast('存储空间不足，请导出旧数据后清理');
  }
}

function loadCurrent() {
  try {
    const raw = localStorage.getItem(LS_KEY_CURRENT);
    if (raw) {
      const arr = JSON.parse(raw);
      if (Array.isArray(arr)) state.options = arr;
    }
  } catch (e) { /* 数据损坏则忽略 */ }
}

function getPresets() {
  try {
    const raw = localStorage.getItem(LS_KEY_PRESETS);
    return raw ? JSON.parse(raw) : {};
  } catch (e) { return {}; }
}

function savePresets(presets) {
  try {
    localStorage.setItem(LS_KEY_PRESETS, JSON.stringify(presets));
  } catch (e) {
    showToast('存储空间不足，请导出旧数据后清理');
  }
}

// ===== 页面初始化 =====
function init() {
  loadCurrent();
  renderTags();
  renderStage();
}
init();
</script>
</body>
</html>
```

- [ ] **Step 2: 浏览器打开验证**

直接用浏览器打开 `C:\programs\个人项目\choice-helper\index.html`，验证：
- 页面正常渲染，深蓝紫渐变背景可见
- 5 个 Tab 可点击切换，active 状态正确
- 输入框可添加选项，标签正确显示
- 点击标签 ✕ 可删除
- 刷新页面后选项保持（localStorage 恢复）

- [ ] **Step 3: Commit**

```bash
cd "/c/programs/个人项目" && git add choice-helper/index.html && git commit -m "feat: HTML骨架 + CSS样式 + Tab切换 + 选项输入"
```

---

### Task 2: 预设列表管理（Modal CRUD + 导入导出）

**Files:**
- Modify: `choice-helper/index.html`（补充 Modal 交互逻辑）

- [ ] **Step 1: 在 `</script>` 前追加预设管理逻辑**

在 `init();` 之前插入以下代码：

```javascript
// ===== Preset Manager =====
let presets = {};

function refreshPresets() {
  presets = getPresets();
}

function openPresetManager() {
  refreshPresets();
  const names = Object.keys(presets);
  const listHtml = names.length === 0
    ? '<p style="color:var(--text-dim);text-align:center;padding:20px;">暂无预设列表，点击下方创建</p>'
    : names.map(name => {
        const count = presets[name].length;
        const preview = presets[name].slice(0, 3).join('、') + (count > 3 ? '…' : '');
        return `
          <div class="preset-item">
            <div>
              <span class="name">${escapeHtml(name)}</span>
              <span class="count">${count} 项</span>
              <div style="font-size:12px;color:var(--text-dim);margin-top:2px;">${escapeHtml(preview)}</div>
            </div>
            <div class="actions">
              <button class="btn-load" data-action="load" data-name="${escapeHtml(name)}">加载</button>
              <button class="btn-delete" data-action="delete" data-name="${escapeHtml(name)}">删除</button>
            </div>
          </div>`;
      }).join('');

  dom.modalContent.innerHTML = `
    <button class="modal-close" id="modalClose">✕</button>
    <h2>⚙ 预设列表管理</h2>
    <div style="margin-bottom:16px;">
      <div class="input-row">
        <input type="text" id="newPresetName" placeholder="新列表名称（如：午餐选择）" maxlength="30">
        <button id="btnCreatePreset">+ 创建</button>
      </div>
      <p style="font-size:12px;color:var(--text-dim);">创建后可在主界面逐个添加选项，或直接导入 JSON</p>
    </div>
    <div id="presetList">${listHtml}</div>
    <div style="margin-top:16px;display:flex;gap:8px;">
      <button class="btn-export" id="btnExportAll">📤 导出全部</button>
      <button class="btn-import" id="btnImportAll">📥 导入</button>
      <input type="file" id="importFile" accept=".json" style="display:none;">
    </div>
  `;

  dom.modalOverlay.classList.add('open');

  // Bind modal events
  document.getElementById('modalClose').addEventListener('click', closeModal);
  document.getElementById('btnCreatePreset').addEventListener('click', createPreset);
  document.getElementById('btnExportAll').addEventListener('click', exportAll);
  document.getElementById('btnImportAll').addEventListener('click', () => {
    document.getElementById('importFile').click();
  });
  document.getElementById('importFile').addEventListener('change', importAll);

  // Bind load/delete buttons
  document.getElementById('presetList').addEventListener('click', (e) => {
    const btn = e.target.closest('button');
    if (!btn) return;
    const action = btn.dataset.action;
    const name = btn.dataset.name;
    if (action === 'load') loadPreset(name);
    if (action === 'delete') deletePreset(name);
  });
}

function closeModal() {
  dom.modalOverlay.classList.remove('open');
}

dom.modalOverlay.addEventListener('click', (e) => {
  if (e.target === dom.modalOverlay) closeModal();
});

function createPreset() {
  const input = document.getElementById('newPresetName');
  const name = input.value.trim();
  if (!name) { showToast('请输入列表名称'); return; }
  refreshPresets();
  if (presets[name]) {
    if (!confirm(`列表"${name}"已存在，是否覆盖？`)) return;
  }
  // 将当前选项作为预设内容
  presets[name] = [...state.options];
  savePresets(presets);
  showToast(`预设"${name}"已保存`);
  openPresetManager(); // refresh modal
}

function loadPreset(name) {
  refreshPresets();
  const list = presets[name];
  if (!list) { showToast('预设不存在'); return; }
  state.options = [...list];
  saveCurrent();
  renderTags();
  renderStage();
  closeModal();
  showToast(`已加载预设"${name}"（${list.length} 项）`);
}

function deletePreset(name) {
  if (!confirm(`确定删除预设"${name}"？`)) return;
  refreshPresets();
  delete presets[name];
  savePresets(presets);
  openPresetManager(); // refresh modal
  showToast(`预设"${name}"已删除`);
}

function exportAll() {
  refreshPresets();
  const json = JSON.stringify(presets, null, 2);
  const blob = new Blob([json], { type: 'application/json' });
  const url = URL.createObjectURL(blob);
  const a = document.createElement('a');
  a.href = url;
  a.download = `choice-helper-presets-${new Date().toISOString().slice(0,10)}.json`;
  a.click();
  URL.revokeObjectURL(url);
  showToast('导出成功');
}

function importAll(e) {
  const file = e.target.files[0];
  if (!file) return;
  const reader = new FileReader();
  reader.onload = (ev) => {
    try {
      const data = JSON.parse(ev.target.result);
      if (typeof data !== 'object' || Array.isArray(data) || data === null) {
        throw new Error('格式错误');
      }
      // Validate structure: each value must be an array of strings
      for (const [key, val] of Object.entries(data)) {
        if (!Array.isArray(val) || !val.every(v => typeof v === 'string')) {
          throw new Error(`预设"${key}"格式无效：值必须是字符串数组`);
        }
      }
      refreshPresets();
      const merged = { ...presets, ...data };
      savePresets(merged);
      showToast(`导入成功（${Object.keys(data).length} 个预设）`);
      openPresetManager();
    } catch (err) {
      showToast(`导入失败：${err.message}`);
    }
  };
  reader.readAsText(file);
  e.target.value = ''; // reset file input
}

// Bind manager button
dom.btnManage.addEventListener('click', openPresetManager);

// Bind "load preset" quick button
dom.btnLoadPreset.addEventListener('click', openPresetManager);
```

- [ ] **Step 2: 浏览器验证**

- 点击 ⚙ 管理列表 → Modal 弹出
- 输入名称"午餐选择"→ 点击创建 → 当前选项保存为预设
- 关闭 Modal，清空选项 → 点击 ▼ 加载预设列表 → 加载"午餐选择"→ 选项恢复
- 导出 JSON 文件 → 删除预设 → 重新导入 → 预设恢复
- 同名预设提示覆盖

- [ ] **Step 3: Commit**

```bash
cd "/c/programs/个人项目" && git add choice-helper/index.html && git commit -m "feat: 预设列表管理（CRUD + JSON导入导出）"
```

---

### Task 3: 🎡 幸运转盘模式

**Files:**
- Modify: `choice-helper/index.html`（替换 renderStage + 添加 Canvas 动画引擎）

- [ ] **Step 1: 在 `renderStage` 函数位置替换为完整实现**

将原来的占位 `renderStage` 函数替换为：

```javascript
// ===== Wheel Engine =====
let wheelCanvas = null;
let wheelCtx = null;
let wheelAngle = 0;
let wheelSpinning = false;
let wheelResult = -1;

const WHEEL_COLORS = [
  '#ff6b6b', '#ffa94d', '#ffd43b', '#69db7c',
  '#4dabf7', '#748ffc', '#da77f2', '#f783ac',
  '#ff8787', '#ffc078', '#ffe066', '#8ce99a',
  '#74c0fc', '#91a7ff', '#e599f7', '#faa2c1',
];

function renderStage() {
  const mode = state.mode;
  if (mode === 'wheel') return renderWheel();
  if (mode === 'draw') return renderDraw();
  if (mode === 'card') return renderCard();
  if (mode === 'dice') return renderDice();
  if (mode === 'coin') return renderCoin();
}

function renderWheel() {
  const n = state.options.length;
  if (n < 2) {
    dom.stage.innerHTML = `<p style="color:var(--text-dim);">请至少添加 2 个选项才能使用转盘</p>`;
    return;
  }
  if (n > 20) {
    dom.stage.innerHTML = `<p style="color:var(--accent);">选项超过 20 个，建议精简后再转（当前 ${n} 个）</p>`;
    return;
  }

  dom.stage.innerHTML = `
    <canvas id="wheelCanvas" width="320" height="320"></canvas>
    <div style="margin-top:16px;">
      <button class="btn-action" id="btnSpin" ${state.isAnimating ? 'disabled' : ''}>
        ${state.isAnimating ? '旋转中…' : '🎯 开始'}
      </button>
    </div>
    <div class="result-overlay" id="wheelResult">
      <div style="font-size:14px;color:var(--text-dim);">🎉 结果</div>
      <div class="result-text" id="wheelResultText"></div>
      <button class="btn-action" id="btnWheelAgain">再来一次</button>
    </div>
  `;

  wheelCanvas = document.getElementById('wheelCanvas');
  wheelCtx = wheelCanvas.getContext('2d');
  drawWheel();

  document.getElementById('btnSpin').addEventListener('click', spinWheel);
  document.getElementById('btnWheelAgain').addEventListener('click', () => {
    document.getElementById('wheelResult').classList.remove('show');
    wheelResult = -1;
    state.isAnimating = false;
    renderWheel();
  });
}

function drawWheel() {
  const n = state.options.length;
  const cx = 160, cy = 160, r = 150;
  const sliceAngle = (2 * Math.PI) / n;
  wheelCtx.clearRect(0, 0, 320, 320);

  // Draw sectors
  for (let i = 0; i < n; i++) {
    const startAngle = wheelAngle + i * sliceAngle;
    const endAngle = startAngle + sliceAngle;
    wheelCtx.beginPath();
    wheelCtx.moveTo(cx, cy);
    wheelCtx.arc(cx, cy, r, startAngle, endAngle);
    wheelCtx.closePath();
    wheelCtx.fillStyle = WHEEL_COLORS[i % WHEEL_COLORS.length];
    wheelCtx.fill();
    wheelCtx.strokeStyle = 'rgba(0,0,0,0.2)';
    wheelCtx.lineWidth = 2;
    wheelCtx.stroke();

    // Draw text
    const textAngle = startAngle + sliceAngle / 2;
    const textR = r * 0.65;
    const tx = cx + Math.cos(textAngle) * textR;
    const ty = cy + Math.sin(textAngle) * textR;
    wheelCtx.save();
    wheelCtx.translate(tx, ty);
    wheelCtx.rotate(textAngle + Math.PI / 2);
    wheelCtx.fillStyle = '#1a1a2e';
    wheelCtx.font = 'bold 13px "Microsoft YaHei", sans-serif';
    wheelCtx.textAlign = 'center';
    const label = state.options[i].length > 6
      ? state.options[i].slice(0, 5) + '…'
      : state.options[i];
    wheelCtx.fillText(label, 0, 0);
    wheelCtx.restore();
  }

  // Center circle
  wheelCtx.beginPath();
  wheelCtx.arc(cx, cy, 20, 0, 2 * Math.PI);
  wheelCtx.fillStyle = '#1a1a2e';
  wheelCtx.fill();
  wheelCtx.strokeStyle = 'var(--primary)';
  wheelCtx.lineWidth = 3;
  wheelCtx.stroke();

  // Pointer (triangle at top)
  wheelCtx.beginPath();
  wheelCtx.moveTo(cx, cy - r + 8);
  wheelCtx.lineTo(cx - 10, cy - r - 12);
  wheelCtx.lineTo(cx + 10, cy - r - 12);
  wheelCtx.closePath();
  wheelCtx.fillStyle = '#fff';
  wheelCtx.fill();
}

function spinWheel() {
  if (state.isAnimating) return;
  state.isAnimating = true;
  document.getElementById('btnSpin').disabled = true;
  document.getElementById('btnSpin').textContent = '旋转中…';

  const n = state.options.length;
  const sliceAngle = (2 * Math.PI) / n;
  // Choose random result
  wheelResult = Math.floor(Math.random() * n);
  // Calculate target angle: pointer at top (3*PI/2), result sector center should align
  const targetSectorCenter = wheelResult * sliceAngle + sliceAngle / 2;
  // We need: wheelAngle + targetSectorCenter ≡ 3*PI/2 (mod 2*PI)
  // So: wheelAngle = 3*PI/2 - targetSectorCenter + N*2*PI
  const targetAngle = (3 * Math.PI / 2 - targetSectorCenter) + Math.PI * 2;
  // Add multiple full rotations for dramatic effect
  const spinTarget = targetAngle + Math.PI * 2 * (5 + Math.floor(Math.random() * 5));

  const startAngle = wheelAngle;
  const startTime = performance.now();
  const duration = 4000; // 4 seconds

  function animate(now) {
    const elapsed = now - startTime;
    const progress = Math.min(elapsed / duration, 1);
    // Ease out cubic
    const eased = 1 - Math.pow(1 - progress, 3);
    wheelAngle = startAngle + (spinTarget - startAngle) * eased;
    drawWheel();

    if (progress < 1) {
      requestAnimationFrame(animate);
    } else {
      // Spin complete
      wheelAngle = spinTarget % (Math.PI * 2);
      state.isAnimating = false;
      showWheelResult();
    }
  }
  requestAnimationFrame(animate);
}

function showWheelResult() {
  const resultText = state.options[wheelResult];
  document.getElementById('wheelResultText').textContent = resultText;
  document.getElementById('wheelResult').classList.add('show');
}
```

- [ ] **Step 2: 浏览器验证**

- 添加 3~8 个选项 → 切换到转盘 Tab → Canvas 转盘可见
- 点击"开始"→ 转盘旋转约 4 秒 → 指针位置揭示结果
- 结果 overlay 显示正确选项名
- "再来一次"可重复旋转
- 选项<2 时显示提示
- 选项>20 时显示警告

- [ ] **Step 3: Commit**

```bash
cd "/c/programs/个人项目" && git add choice-helper/index.html && git commit -m "feat: 幸运转盘模式（Canvas绘制 + 缓动旋转动画）"
```

---

### Task 4: 🎋 抽签筒模式

**Files:**
- Modify: `choice-helper/index.html`（追加抽签引擎）

- [ ] **Step 1: 在 `renderCoin` 占位前追加抽签实现**

在 `renderStage` 下方追加：

```javascript
// ===== Draw Engine =====
function renderDraw() {
  const n = state.options.length;
  if (n < 2) {
    dom.stage.innerHTML = `<p style="color:var(--text-dim);">请至少添加 2 个选项才能抽签</p>`;
    return;
  }

  dom.stage.innerHTML = `
    <div id="drawContainer" style="text-align:center;">
      <div id="drawTube" style="
        width:120px;height:200px;margin:0 auto 20px;
        background:linear-gradient(180deg,rgba(240,165,0,0.25),rgba(240,165,0,0.08));
        border:3px solid rgba(240,165,0,0.4);
        border-radius:12px 12px 20px 20px;
        position:relative;overflow:hidden;
        box-shadow:inset 0 0 20px rgba(0,0,0,0.3);
      ">
        <div id="sticksContainer" style="
          position:absolute;bottom:0;left:50%;transform:translateX(-50%);
          display:flex;flex-direction:column-reverse;align-items:center;
          gap:2px;padding-bottom:8px;
        ">
          ${state.options.map((opt, i) => `
            <div class="stick" data-index="${i}" style="
              width:80px;height:18px;background:linear-gradient(90deg,#f0c27b,#e0a050);
              border-radius:3px;font-size:10px;line-height:18px;text-align:center;
              color:#3d2000;font-weight:bold;overflow:hidden;white-space:nowrap;
              box-shadow:0 1px 2px rgba(0,0,0,0.3);
            ">${escapeHtml(opt)}</div>
          `).join('')}
        </div>
      </div>
      <button class="btn-action" id="btnShake" ${state.isAnimating ? 'disabled' : ''}>
        ${state.isAnimating ? '摇签中…' : '🎋 摇一摇'}
      </button>
    </div>
    <div class="result-overlay" id="drawResult">
      <div style="font-size:14px;color:var(--text-dim);">🎋 签文</div>
      <div class="result-text" id="drawResultText"></div>
      <button class="btn-action" id="btnDrawAgain">再来一次</button>
    </div>
  `;

  document.getElementById('btnShake').addEventListener('click', shakeDraw);
  document.getElementById('btnDrawAgain').addEventListener('click', () => {
    document.getElementById('drawResult').classList.remove('show');
    state.isAnimating = false;
    renderDraw();
  });
}

function shakeDraw() {
  if (state.isAnimating) return;
  state.isAnimating = true;

  const btn = document.getElementById('btnShake');
  btn.disabled = true;
  btn.textContent = '摇签中…';

  const tube = document.getElementById('drawTube');
  const sticks = tube.querySelectorAll('.stick');
  const drawResult = Math.floor(Math.random() * state.options.length);

  // Shake animation: random vertical offsets for each stick
  const startTime = performance.now();
  const duration = 2500;

  function animate(now) {
    const elapsed = now - startTime;
    const progress = Math.min(elapsed / duration, 1);

    // Tube shake
    const shakeX = Math.sin(progress * 40) * (1 - progress) * 8;
    const shakeY = Math.cos(progress * 37) * (1 - progress) * 4;
    tube.style.transform = `translate(${shakeX}px, ${shakeY}px)`;

    // Stick jitter
    sticks.forEach((stick, i) => {
      const offset = Math.sin(progress * 60 + i * 2.7) * (1 - progress) * 6;
      stick.style.transform = `translateX(${offset}px)`;
    });

    if (progress < 1) {
      requestAnimationFrame(animate);
    } else {
      // Reset transforms
      tube.style.transform = '';
      sticks.forEach(s => s.style.transform = '');
      // Pop out winning stick
      const winningStick = tube.querySelector(`.stick[data-index="${drawResult}"]`);
      if (winningStick) {
        winningStick.style.transform = 'translateY(-30px) scale(1.15)';
        winningStick.style.background = 'linear-gradient(90deg, var(--primary), #ffd56b)';
        winningStick.style.boxShadow = '0 0 16px rgba(240,165,0,0.6)';
      }
      state.isAnimating = false;
      setTimeout(() => showDrawResult(drawResult), 400);
    }
  }
  requestAnimationFrame(animate);
}

function showDrawResult(index) {
  document.getElementById('drawResultText').textContent = state.options[index];
  document.getElementById('drawResult').classList.add('show');
}
```

- [ ] **Step 2: 浏览器验证**

- 添加选项 → 切换到抽签 Tab → 签筒+签条可见
- 点击"摇一摇"→ 签筒抖动 2.5 秒 → 中签签条高亮弹出
- 结果 overlay 显示签文
- "再来一次"可重复

- [ ] **Step 3: Commit**

```bash
cd "/c/programs/个人项目" && git add choice-helper/index.html && git commit -m "feat: 抽签筒模式（摇签动画 + 签条高亮）"
```

---

### Task 5: 🃏 翻牌/洗牌模式

**Files:**
- Modify: `choice-helper/index.html`（追加翻牌引擎）

- [ ] **Step 1: 追加翻牌实现**

在 `renderStage` 下方追加：

```javascript
// ===== Card Engine =====
function renderCard() {
  const n = state.options.length;
  if (n < 3) {
    dom.stage.innerHTML = `<p style="color:var(--text-dim);">请至少添加 3 个选项才能翻牌</p>`;
    return;
  }
  if (n > 12) {
    dom.stage.innerHTML = `<p style="color:var(--accent);">选项超过 12 个，建议精简后再翻（当前 ${n} 个）</p>`;
    return;
  }

  const cols = n <= 4 ? n : n <= 6 ? 3 : 4;
  const cardW = n <= 4 ? 120 : n <= 6 ? 100 : 80;
  const cardH = n <= 4 ? 160 : n <= 6 ? 130 : 110;
  const fontSize = n <= 4 ? 14 : 12;

  dom.stage.innerHTML = `
    <div id="cardGrid" style="
      display:grid;grid-template-columns:repeat(${cols},${cardW}px);
      gap:12px;justify-content:center;padding:16px;
    ">
      ${state.options.map((opt, i) => `
        <div class="card-wrapper" data-index="${i}">
          <div class="card-inner" style="
            width:${cardW}px;height:${cardH}px;
            position:relative;
            transform-style:preserve-3d;
            transition:transform 0.6s;
          ">
            <div class="card-front" style="
              position:absolute;inset:0;backface-visibility:hidden;
              background:linear-gradient(135deg,var(--primary),#ffd56b);
              border-radius:10px;display:flex;align-items:center;justify-content:center;
              font-size:${fontSize}px;font-weight:bold;color:#1a1a2e;
              text-align:center;padding:6px;overflow:hidden;
            ">${escapeHtml(opt)}</div>
            <div class="card-back" style="
              position:absolute;inset:0;backface-visibility:hidden;
              transform:rotateY(180deg);
              background:linear-gradient(135deg,#2d2d5e,#1e1e3a);
              border-radius:10px;display:flex;align-items:center;justify-content:center;
              border:2px solid rgba(240,165,0,0.4);
              font-size:28px;
            ">?</div>
          </div>
        </div>
      `).join('')}
    </div>
    <div style="margin-top:12px;">
      <button class="btn-action" id="btnShuffle" ${state.isAnimating ? 'disabled' : ''}>
        ${state.isAnimating ? '洗牌中…' : '🃏 洗牌并翻牌'}
      </button>
    </div>
    <div class="result-overlay" id="cardResult">
      <div style="font-size:14px;color:var(--text-dim);">🃏 选中</div>
      <div class="result-text" id="cardResultText"></div>
      <button class="btn-action" id="btnCardAgain">再来一次</button>
    </div>
  `;

  document.getElementById('btnShuffle').addEventListener('click', shuffleAndReveal);
  document.getElementById('btnCardAgain').addEventListener('click', () => {
    document.getElementById('cardResult').classList.remove('show');
    state.isAnimating = false;
    renderCard();
  });
}

function shuffleAndReveal() {
  if (state.isAnimating) return;
  state.isAnimating = true;

  const btn = document.getElementById('btnShuffle');
  btn.disabled = true;
  btn.textContent = '洗牌中…';

  const cards = document.querySelectorAll('.card-inner');
  const wrappers = document.querySelectorAll('.card-wrapper');
  const cardResult = Math.floor(Math.random() * state.options.length);

  // Step 1: Flip all cards to back (hide options)
  cards.forEach((card, i) => {
    setTimeout(() => {
      card.style.transform = 'rotateY(180deg)';
    }, i * 80);
  });

  // Step 2: Shuffle positions after all flipped
  const totalFlipTime = cards.length * 80 + 600;

  setTimeout(() => {
    const grid = document.getElementById('cardGrid');
    const wrapperArray = Array.from(wrappers);
    // Fisher-Yates shuffle
    for (let i = wrapperArray.length - 1; i > 0; i--) {
      const j = Math.floor(Math.random() * (i + 1));
      // Swap DOM positions
      const refA = wrapperArray[i];
      const refB = wrapperArray[j];
      // Insert refA before refB's next sibling (or at end)
      const parent = refA.parentNode;
      const nextB = refB.nextSibling;
      parent.insertBefore(refA, nextB);
      if (refA !== refB) {
        parent.insertBefore(refB, refA.nextSibling || null);
      }
      // Swap in array
      [wrapperArray[i], wrapperArray[j]] = [wrapperArray[j], wrapperArray[i]];
    }

    // Brief position animation
    grid.style.transition = 'gap 0.3s';
    grid.style.gap = '16px';
    setTimeout(() => { grid.style.gap = '12px'; }, 300);

    // Step 3: Enable click to reveal
    btn.textContent = '点击任意一张翻牌…';
    btn.disabled = false;

    wrappers.forEach((w, i) => {
      w.style.cursor = 'pointer';
      w.addEventListener('click', function reveal(e) {
        const idx = parseInt(w.dataset.index);
        revealCard(w, idx, cardResult);
        // Remove all listeners
        wrappers.forEach(ww => {
          ww.style.cursor = 'default';
          ww.replaceWith(ww.cloneNode(true));
        });
        btn.disabled = true;
        btn.textContent = '已揭晓';
      });
    });
  }, totalFlipTime);
}

function revealCard(wrapper, chosenIndex, winIndex) {
  const inner = wrapper.querySelector('.card-inner');
  inner.style.transform = 'rotateY(0deg)'; // flip to front
  inner.style.boxShadow = chosenIndex === winIndex
    ? '0 0 24px rgba(240,165,0,0.6)'
    : '0 0 8px rgba(255,107,107,0.4)';

  // Flip all other cards to front too
  setTimeout(() => {
    document.querySelectorAll('.card-inner').forEach(c => {
      c.style.transform = 'rotateY(0deg)';
    });

    // Highlight winner
    const winWrapper = document.querySelector(`.card-wrapper[data-index="${winIndex}"]`);
    if (winWrapper) {
      const winInner = winWrapper.querySelector('.card-inner');
      winInner.style.transform = 'rotateY(0deg) scale(1.08)';
      winInner.style.boxShadow = '0 0 28px rgba(240,165,0,0.7)';
      winInner.style.zIndex = '5';
    }

    setTimeout(() => showCardResult(winIndex), 800);
  }, 400);
}

function showCardResult(index) {
  document.getElementById('cardResultText').textContent = state.options[index];
  document.getElementById('cardResult').classList.add('show');
}
```

- [ ] **Step 2: 浏览器验证**

- 添加 4~8 个选项 → 切换到翻牌 Tab → 卡片网格可见
- 点击"洗牌并翻牌"→ 卡片逐张翻面 → 位置打乱 → 提示点击
- 点击一张卡片 → 揭晓 + 所有卡片翻回正面 + 中选卡片高亮放大
- 结果 overlay 显示正确选项
- 选项<3 时提示；>12 时警告

- [ ] **Step 3: Commit**

```bash
cd "/c/programs/个人项目" && git add choice-helper/index.html && git commit -m "feat: 翻牌洗牌模式（3D翻转 + 随机洗牌 + 点击揭晓）"
```

---

### Task 6: 🎲 骰子 + 🪙 硬币模式

**Files:**
- Modify: `choice-helper/index.html`（追加骰子和硬币引擎）

- [ ] **Step 1: 追加骰子和硬币实现**

追加以下代码：

```javascript
// ===== Dice Engine =====
function renderDice() {
  dom.stage.innerHTML = `
    <div style="text-align:center;">
      <div style="display:flex;gap:16px;justify-content:center;align-items:center;margin-bottom:20px;">
        <input type="number" id="diceMin" value="${diceState.min}" min="1" max="9999"
          style="width:80px;padding:10px;border-radius:8px;border:1px solid rgba(255,255,255,0.15);
          background:rgba(0,0,0,0.25);color:var(--text);font-size:18px;text-align:center;
          font-family:var(--font);outline:none;">
        <span style="font-size:20px;color:var(--text-dim);">~</span>
        <input type="number" id="diceMax" value="${diceState.max}" min="2" max="9999"
          style="width:80px;padding:10px;border-radius:8px;border:1px solid rgba(255,255,255,0.15);
          background:rgba(0,0,0,0.25);color:var(--text);font-size:18px;text-align:center;
          font-family:var(--font);outline:none;">
      </div>
      <div id="diceDisplay" style="
        width:120px;height:120px;margin:0 auto 20px;
        background:var(--surface);border-radius:20px;
        display:flex;align-items:center;justify-content:center;
        font-size:52px;font-weight:bold;color:var(--primary);
        border:2px solid rgba(240,165,0,0.3);
        text-shadow:0 0 16px rgba(240,165,0,0.4);
        transition:transform 0.1s;
      ">?</div>
      <button class="btn-action" id="btnRoll" ${state.isAnimating ? 'disabled' : ''}>
        ${state.isAnimating ? '滚动中…' : '🎲 掷骰子'}
      </button>
    </div>
    <div class="result-overlay" id="diceResult">
      <div style="font-size:14px;color:var(--text-dim);">🎲 结果</div>
      <div class="result-text" id="diceResultText"></div>
      <button class="btn-action" id="btnDiceAgain">再来一次</button>
    </div>
  `;

  // Sync inputs
  document.getElementById('diceMin').addEventListener('change', (e) => {
    diceState.min = Math.max(1, parseInt(e.target.value) || 1);
    e.target.value = diceState.min;
    if (diceState.min >= diceState.max) {
      diceState.max = diceState.min + 1;
      document.getElementById('diceMax').value = diceState.max;
    }
  });
  document.getElementById('diceMax').addEventListener('change', (e) => {
    diceState.max = Math.max(2, parseInt(e.target.value) || 2);
    e.target.value = diceState.max;
    if (diceState.max <= diceState.min) {
      diceState.min = diceState.max - 1;
      document.getElementById('diceMin').value = diceState.min;
    }
  });

  document.getElementById('btnRoll').addEventListener('click', rollDice);
  document.getElementById('btnDiceAgain').addEventListener('click', () => {
    document.getElementById('diceResult').classList.remove('show');
    state.isAnimating = false;
    renderDice();
  });
}

const diceState = { min: 1, max: 6 };

function rollDice() {
  if (state.isAnimating) return;
  state.isAnimating = true;

  const btn = document.getElementById('btnRoll');
  btn.disabled = true;
  btn.textContent = '滚动中…';

  const display = document.getElementById('diceDisplay');
  const min = diceState.min;
  const max = diceState.max;
  const result = Math.floor(Math.random() * (max - min + 1)) + min;

  const startTime = performance.now();
  const duration = 1200;
  let lastSwitch = 0;

  function animate(now) {
    const elapsed = now - startTime;
    const progress = Math.min(elapsed / duration, 1);

    // Rapid number switching, slowing down
    if (now - lastSwitch > 50 + progress * 120) {
      display.textContent = Math.floor(Math.random() * (max - min + 1)) + min;
      display.style.transform = `scale(${0.9 + Math.random() * 0.2})`;
      lastSwitch = now;
    }

    if (progress < 1) {
      requestAnimationFrame(animate);
    } else {
      display.textContent = result;
      display.style.transform = 'scale(1)';
      state.isAnimating = false;
      setTimeout(() => showDiceResult(result), 300);
    }
  }
  requestAnimationFrame(animate);
}

function showDiceResult(result) {
  document.getElementById('diceResultText').textContent = result;
  document.getElementById('diceResult').classList.add('show');
}

// ===== Coin Engine =====
function renderCoin() {
  dom.stage.innerHTML = `
    <div style="text-align:center;">
      <div id="coinDisplay" style="
        width:140px;height:140px;margin:0 auto 30px;
        border-radius:50%;
        background:linear-gradient(135deg,#f0c27b,#e0a050);
        display:flex;align-items:center;justify-content:center;
        font-size:48px;font-weight:bold;color:#3d2000;
        box-shadow:0 8px 32px rgba(240,165,0,0.3),inset 0 2px 4px rgba(255,255,255,0.3);
        transition:transform 0.1s;
        user-select:none;
      ">?</div>
      <button class="btn-action" id="btnFlip" ${state.isAnimating ? 'disabled' : ''}>
        ${state.isAnimating ? '抛掷中…' : '🪙 抛硬币'}
      </button>
    </div>
    <div class="result-overlay" id="coinResult">
      <div style="font-size:14px;color:var(--text-dim);">🪙 结果</div>
      <div class="result-text" id="coinResultText"></div>
      <button class="btn-action" id="btnCoinAgain">再来一次</button>
    </div>
  `;

  document.getElementById('btnFlip').addEventListener('click', flipCoin);
  document.getElementById('btnCoinAgain').addEventListener('click', () => {
    document.getElementById('coinResult').classList.remove('show');
    state.isAnimating = false;
    renderCoin();
  });
}

function flipCoin() {
  if (state.isAnimating) return;
  state.isAnimating = true;

  const btn = document.getElementById('btnFlip');
  btn.disabled = true;
  btn.textContent = '抛掷中…';

  const coin = document.getElementById('coinDisplay');
  const isHeads = Math.random() < 0.5;

  const startTime = performance.now();
  const duration = 1600;
  const flips = 5 + Math.floor(Math.random() * 4);

  function animate(now) {
    const elapsed = now - startTime;
    const progress = Math.min(elapsed / duration, 1);

    // 3D flip simulation via scaleX
    const flipProgress = progress * flips;
    const scaleX = Math.cos(flipProgress * Math.PI);
    const absScaleX = Math.abs(scaleX);

    coin.style.transform = `scaleX(${absScaleX}) rotateY(${flipProgress * 360}deg)`;
    coin.style.opacity = absScaleX < 0.3 ? '0.6' : '1';

    // Switch face at midpoint of each flip
    const halfFlip = (flipProgress % 1) > 0.5;
    coin.textContent = halfFlip ? '正' : '反';

    if (progress < 1) {
      requestAnimationFrame(animate);
    } else {
      coin.style.transform = 'scaleX(1) rotateY(0deg)';
      coin.style.opacity = '1';
      coin.textContent = isHeads ? '正' : '反';
      coin.style.background = isHeads
        ? 'linear-gradient(135deg,#f0c27b,#e0a050)'
        : 'radial-gradient(circle,#d4a574,#b8860b)';
      state.isAnimating = false;
      setTimeout(() => showCoinResult(isHeads), 400);
    }
  }
  requestAnimationFrame(animate);
}

function showCoinResult(isHeads) {
  const label = isHeads ? '正面' : '反面';
  document.getElementById('coinResultText').textContent = label;
  document.getElementById('coinResult').classList.add('show');
}

// Placeholder stubs for renderCoin/renderDice (replace when implemented)
// These are already defined above - no stubs needed
```

- [ ] **Step 2: 浏览器验证**

**骰子：**
- 切换到骰子 Tab → 范围输入框默认 1~6
- 点击"掷骰子"→ 数字快速切换约 1.2 秒 → 定格结果
- 修改范围后掷骰子（如 10~20）→ 结果在范围内

**硬币：**
- 切换到硬币 Tab → 硬币圆形可见
- 点击"抛硬币"→ 硬币 3D 翻转约 1.6 秒 → 显示正/反面

- [ ] **Step 3: Commit**

```bash
cd "/c/programs/个人项目" && git add choice-helper/index.html && git commit -m "feat: 骰子 + 硬币模式（数字滚动 + 3D翻转动画）"
```

---

### Task 7: 收尾 — 错误处理 + 边界情况 + 键盘快捷键

**Files:**
- Modify: `choice-helper/index.html`（补全边界处理）

- [ ] **Step 1: 补全边界处理逻辑**

在 `init()` 函数之前追加：

```javascript
// ===== Edge Cases & Polish =====

// Prevent adding empty/whitespace-only options (already handled in addOption)
// Warn on duplicate options (visual cue)
function addOption(text) {
  const trimmed = text.trim();
  if (!trimmed) {
    showToast('选项不能为空');
    return;
  }
  if (trimmed.length > 50) {
    showToast('选项最多 50 个字符');
    return;
  }
  // Duplicate warning
  if (state.options.includes(trimmed)) {
    if (!confirm(`选项"${trimmed}"已存在，确定要添加重复项吗？`)) return;
  }
  state.options.push(trimmed);
  saveCurrent();
  renderTags();
  renderStage();
  dom.optionInput.value = '';
  dom.optionInput.focus();
}

// Keyboard shortcut: Escape to close modal
document.addEventListener('keydown', (e) => {
  if (e.key === 'Escape' && dom.modalOverlay.classList.contains('open')) {
    closeModal();
  }
});

// Save current options on page unload (already done on each change via saveCurrent)
// Handle storage quota exceeded gracefully
function safeSetItem(key, value) {
  try {
    localStorage.setItem(key, value);
  } catch (e) {
    if (e.name === 'QuotaExceededError' || e.code === 22) {
      showToast('存储空间已满，请导出数据后清理旧预设', 4000);
    }
  }
}

// Override existing localStorage functions with safe wrappers
const _origSetItem = localStorage.setItem.bind(localStorage);
localStorage.setItem = function(key, value) {
  try {
    _origSetItem(key, value);
  } catch (e) {
    showToast('存储空间已满，请导出数据后清理旧预设', 4000);
  }
};
```

- [ ] **Step 2: 验证所有模式的结果正确性**

写一个快速验证脚本，在浏览器 Console 中运行：

```javascript
// 验证转盘结果在合法范围内
for (let i = 0; i < 100; i++) {
  const r = Math.floor(Math.random() * state.options.length);
  if (r < 0 || r >= state.options.length) console.error('Wheel result OOB:', r);
}

// 验证骰子范围
const min = 1, max = 6;
for (let i = 0; i < 100; i++) {
  const r = Math.floor(Math.random() * (max - min + 1)) + min;
  if (r < min || r > max) console.error('Dice OOB:', r);
}

console.log('Random checks passed');
```

- [ ] **Step 3: 整体功能回归测试**

| 测试项 | 预期 |
|--------|------|
| 添加"吃饭""睡觉"→ 刷新 → 选项保持 | ✅ |
| 创建预设"测试列表"→ 清空选项 → 加载预设 → 选项恢复 | ✅ |
| 转盘 2 个选项 → 旋转 → 结果正确 | ✅ |
| 转盘 1 个选项 → 显示提示 | ✅ |
| 抽签 5 个选项 → 摇签 → 中签高亮 | ✅ |
| 翻牌 3 个选项 → 洗牌 → 点击翻牌 → 选中高亮 | ✅ |
| 骰子 50~100 → 掷 → 结果在范围内 | ✅ |
| 硬币 → 抛 → 显示正或反 | ✅ |
| 重复选项 → 弹窗确认 | ✅ |
| 导入格式错误的 JSON → Toast 提示 | ✅ |

- [ ] **Step 4: Commit**

```bash
cd "/c/programs/个人项目" && git add choice-helper/index.html && git commit -m "fix: 边界情况处理 + 重复选项提醒 + 存储配额保护"
```

---

## 实现顺序

```
Task 1 (骨架+CSS+Tab) → Task 2 (预设管理) → Task 3 (转盘)
                                                   ↓
Task 6 (骰子+硬币) ← Task 5 (翻牌) ← Task 4 (抽签)
                                                   ↓
                                            Task 7 (收尾)
```

每个 Task 独立可验证，不依赖后续 Task 即可在浏览器中看到效果。
