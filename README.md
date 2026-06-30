<p align="center">
  <img src="https://img.shields.io/badge/HTML5-E34F26?style=for-the-badge&logo=html5&logoColor=white" alt="HTML5">
  <img src="https://img.shields.io/badge/CSS3-1572B6?style=for-the-badge&logo=css3&logoColor=white" alt="CSS3">
  <img src="https://img.shields.io/badge/JavaScript-F7DF1E?style=for-the-badge&logo=javascript&logoColor=black" alt="JavaScript">
  <img src="https://img.shields.io/badge/Zero_Dependencies-✓-brightgreen?style=for-the-badge" alt="Zero Dependencies">
  <img src="https://img.shields.io/badge/GitHub_Pages-✓-blue?style=for-the-badge&logo=github" alt="GitHub Pages">
</p>

<h1 align="center">🎲 两碗饭-选择助手</h1>

<p align="center">
  <strong>帮选择困难症做决定的趣味小工具</strong><br>
  深色紫红配色 · 5 种决策模式 · CSS 3D 动画 · 预设列表管理
</p>

<p align="center">
  <a href="https://nutshell319.github.io/choice-helper/"><strong>🔗 立即体验</strong></a>
  &nbsp;·&nbsp;
  <a href="#-决策模式">决策模式</a>
  &nbsp;·&nbsp;
  <a href="#-快速开始">快速开始</a>
  &nbsp;·&nbsp;
  <a href="#-功能亮点">功能亮点</a>
</p>

---

## ✨ 为什么做这个

有时候选择比努力更难。中午吃什么？周末去哪？要不要换工作？

两碗饭-选择助手用**趣味交互**消解决策摩擦——转盘的刺激、抽签的仪式感、翻牌的悬念，让每一次选择都变成一件好玩的事。

---

## 🎯 决策模式

<table>
  <tr>
    <td width="33%" align="center">
      <h3>🎡 幸运转盘</h3>
      <p>Canvas 绘制彩色转盘<br>easeOutBack 缓停旋转<br>支持 2~20 个选项</p>
    </td>
    <td width="33%" align="center">
      <h3>🎋 抽签筒</h3>
      <p>签条抖动 + 中签弹出<br>中签高亮发光效果<br>支持 2~30 个选项</p>
    </td>
    <td width="33%" align="center">
      <h3>🃏 翻牌洗牌</h3>
      <p>3D 翻转隐藏选项<br>FLIP 滑动洗牌动画<br>点击揭晓，悬念拉满</p>
    </td>
  </tr>
  <tr>
    <td align="center">
      <h3>🎲 3D 骰子</h3>
      <p>CSS 正方体真实骰子<br>双轴随机翻滚动画<br>圆点面 1~6</p>
    </td>
    <td align="center">
      <h3>🪙 抛硬币</h3>
      <p>双面 3D 银币<br>花面 / 人像面<br>rotateY 翻转动画</p>
    </td>
    <td align="center">
      <h3>⚙ 预设管理</h3>
      <p>常用列表一键加载<br>JSON 导入/导出备份<br>localStorage 自动保存</p>
    </td>
  </tr>
</table>

---

## 🎨 视觉风格

| 元素 | 值 |
|------|-----|
| 背景 | `#08090d → #12121a` 深黑渐变 |
| 主色 | `#b98eff` 紫 |
| 强调色 | `#e85d75` 玫红 |
| 字体 | Microsoft YaHei / PingFang SC |
| 骰子 | 米白象牙质感圆角立方体 |
| 硬币 | 银灰色金属径向渐变 + 多层币缘 |

---

## 🚀 快速开始

### 在线使用

👉 **[nutshell319.github.io/choice-helper](https://nutshell319.github.io/choice-helper/)**

浏览器打开即用，无需安装。

### 本地运行

```bash
git clone https://github.com/nutshell319/choice-helper.git
cd choice-helper
# 双击 index.html 即可
```

---

## 🧱 技术栈

| 技术 | 用途 |
|------|------|
| **HTML5 Canvas** | 转盘绘制 + 指针动画 |
| **CSS 3D Transforms** | 骰子正方体、硬币双面、翻牌翻转、FLIP 洗牌 |
| **requestAnimationFrame** | 骰子翻滚、转盘旋转、抽签抖动、硬币翻转 |
| **localStorage** | 预设列表 + 当前选项持久化 |
| **零依赖** | 纯单文件，CSS/JS 全部内联，< 50KB |

---

## 📂 项目结构

```
choice-helper/
├── index.html          # 主文件（单文件，CSS/JS 内联）
├── README.md
└── docs/
    └── specs/          # 设计规格 + 实现计划
```

---

## 📝 License

MIT © 2026 nutshell319
