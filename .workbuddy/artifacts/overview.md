# 博客 UI 三模块优化 — 完成报告

## 改动概览

| 模块 | 文件 | 核心改动 |
|------|------|---------|
| 01 全局配色 | `skins/_default.scss` | 主色天空蓝 + CSS 变量双色板 |
| 02 导航毛玻璃 | `_masthead.scss` + `masthead.html` | 毛玻璃 + sticky + 暗色切换按钮 |
| 03 文章卡片 | `_archive.scss` | 浮层卡片 + 色标条 + hover 上浮 |

---

## 模块01 — 全局配色重塑

**文件：** `_sass/minimal-mistakes/skins/_default.scss`

- 主色 `$primary-color` 从灰 `#6f777d` → 天空蓝 `#4A90D9`
- 圆角 `$border-radius` 从 `4px` → `12px`
- 注入 `:root`（亮色）和 `[data-theme="dark"]`（暗色）两套 CSS 自定义属性：

```
亮色：
  --bg #ffffff  --surface #f4f8fd  --text #1E3A5F
  --primary #4A90D9  --primary-light #93C5FD

暗色：
  --bg #0f172a  --surface #1e293b  --text #e2eaf4
  --primary #60A5FA  --card-bg #1e293b
```

- `.theme-toggle` 按钮样式（月亮/太阳图标，hover 旋转缩放）

---

## 模块02 — 导航毛玻璃化

**文件：** `_masthead.scss` / `masthead.html`

- `position: sticky; top: 0; z-index: 100` — 导航栏吸顶
- `backdrop-filter: blur(20px) saturate(180%)` — 毛玻璃效果
- `background: var(--masthead-bg)` — 亮色 `rgba(255,255,255,0.82)`，暗色 `rgba(15,23,42,0.88)`
- 滚动 10px 后自动添加 `.is-scrolled` → 增强投影
- 导航项 hover → 蓝色背景 pill 效果

**主题切换按钮逻辑（内嵌 JS）：**
1. 页面加载时读取 `localStorage` 或系统偏好，无闪白
2. 点击按钮：切换 `html[data-theme]`，写入 `localStorage`
3. 滚动监听：被动事件监听，性能无损

---

## 模块03 — 文章卡片浮层化

**文件：** `_archive.scss`

`.list__item` 新增：
- 圆角卡片 `border-radius: 14px`、边框 `var(--border)`、投影 `var(--card-shadow)`
- 左侧色标条 `::before` — 天空蓝渐变竖条（4px 宽）
- `transition: transform 0.25s cubic-bezier(0.16, 1, 0.3, 1)` — 弹性上浮动效
- hover：`translateY(-4px)` + 增强投影 + 边框高亮 + 标题变蓝
- 标签/meta 颜色跟随 CSS 变量，暗模式自动适配

---

## 后续可选优化

- [ ] 文章正文页 `.page__content` 配色调优（代码块、引用块）
- [ ] 侧边栏 `.sidebar` 卡片化（`author-profile`）
- [ ] 首页 Hero 区背景色 `var(--primary-fill)` 应用
- [ ] `_page.scss` 暗模式下标题 `h1-h3` 颜色调整
