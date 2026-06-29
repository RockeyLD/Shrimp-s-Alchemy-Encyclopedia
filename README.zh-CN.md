[![en](https://img.shields.io/badge/docs-English-blue.svg)](README.md)
[![zh-CN](https://img.shields.io/badge/docs-%E4%B8%AD%E6%96%87-red.svg)](README.zh-CN.md)

# Shrimp's Alchemy 百科全书
Shrimp's Alchemy 合并游戏中所有配方的官方百科全书！



官方链接：[https://rockeyld.github.io/Shrimp-s-Alchemy-Encyclopedia](https://rockeyld.github.io/Shrimp-s-Alchemy-Encyclopedia/)

---

## 给 AI 助手和 Mod 开发者的笔记

### 1. 项目架构
- **单文件应用**：所有代码都在 `index.html` 中（内嵌 CSS + JavaScript）。没有构建步骤，没有使用框架。
- **数据源**：页面加载时，从 `https://raw.githubusercontent.com/shrimp0913/Shrimp-s-Alchemy/main/app.js` 获取原始 `app.js`。
- **回退机制**：如果获取失败，则回退到 `localStorage` 缓存键 `shrimp_alchemy_cache`。
- **解析**：`parseAppJs(source)` 函数（约第 412 行）通过正则表达式从原始 JS 文本中解析配方、加密配方、元素名称/图标和 `finalItems`。
- **数据源指示器**：右上角有一个小徽章，显示数据状态（Live / Cached / Error）。5 秒后自动淡出（见 `mainInit`，约第 1755 行）。

### 2. 图形与画布系统
- **渲染器**：HTML5 Canvas，使用自定义力导向布局。
- **节点**：每个元素是一个节点。属性包括 `id`、`name`、`color`、`depth`、`isBase`、`isFinal`、`isGhost`、`isImplicitFinal`、`isDefaultColor`。
- **边**：每个配方 `A + B = Result` 创建两条有向边（`A→Result`，`B→Result`）。
- **布局**：`runOrganicLayout()` 运行带斥力和边引力的力模拟。稳定后，`startFloating()` 运行持续的轻微浮动动画循环。
- **相机**：拖拽平移，滚轮或捏合缩放。变量：`camX`、`camY`、`camZoom`。

### 3. 如何修改元素颜色
颜色定义在 `CATEGORY_COLORS` 对象中（约第 840 行）。如果新元素被抓取但未在此列出，默认颜色为**透明绿色**（`rgba(102,187,106,0.3)`），边框为半透明（`rgba(102,187,106,0.5)`）。这使得未映射的元素在视觉上与众不同。

```js
const CATEGORY_COLORS = {
  'lake': '#4fc3f7',
  'glass-water': '#4dd0e1',  // 带连字符的 ID 必须使用引号
  'heat': '#ff5722',
  // ...
};
```

基础元素（water, fire, earth, air）有它们自己的 `BASE_COLORS` 对象在其上方。

### 4. 如何添加 / 修改图标
应用对 UI 元素使用 **CSS mask 图标**（不是从获取数据中来的元素图标，那些是内联 SVG）。要添加新的 UI 图标：

1. 将 SVG 文件放在 `Image/` 文件夹中（例如 `Image/my-icon.svg`）。
2. 在 `<style>` 块中添加 CSS 规则（约第 260 行）：
   ```css
   .icon-my-icon {
     -webkit-mask-image: url('Image/my-icon.svg');
     mask-image: url('Image/my-icon.svg');
   }
   ```
3. 在 HTML 或 JS 中使用：`<span class="icon icon-my-icon"></span>`

**重要**：新的 SVG 文件必须执行 `git add` 并推送；否则 GitHub Pages 会返回 404，图标会显示为空白方块。

### 5. 交互与高亮
- **悬停 / 点击**：高亮元素的直接上游原料（橙色）和下游产物（青色）。在 `handleHover()` 和 `canvas.click` 中实现。
- **搜索**：搜索框按名称/ID 过滤元素，并高亮其一层连接。
- **Dragon Studio**：硬编码集合 `DRAGON_STUDIO_ELEMENTS`（约第 541 行）标记特殊元素。点击龙按钮仅高亮这些节点。
- **主题**：通过 CSS `body.light` 和 CSS 变量在深色和浅色主题之间切换。`getThemeCanvasColors()` 返回当前主题的画布特定颜色。

### 6. 模态框与 UI
- **致谢模态框**：`#credit-modal` 通过 `.open` 类切换。显示 `SHRIMP'S ALCHEMY.svg` 标志（带主题感知 `filter` CSS）和原作者链接。打开时，画布上的悬停高亮被禁用（见 `handleHover()`）。
- **统计**：右下角面板显示元素总数和配方总数。
- **图例**：左下角面板解释节点/边颜色含义。

### 7. 幽灵与最终物品
- **幽灵元素**：在配方中出现但在获取数据中无名/图标定义的 ID。它们自动创建为带问号图标和虚线青色边框的节点。
- **最终物品**：由原始游戏明确标记（`finalItems.add(...)`）。它们获得金色虚线边框和工具提示中的星形徽章。
- **隐式最终物品**：从未在任何配方中作为原料使用的元素。它们获得红色虚线边框。
- **默认颜色元素**：未在 `BASE_COLORS` 或 `CATEGORY_COLORS` 中列出的元素渲染为透明绿色节点（`rgba(102,187,106,0.3)` 填充，`rgba(102,187,106,0.5)` 边框）以区别于分类元素。通过 `initNodes()` 中的 `node.isDefaultColor` 跟踪（约第 925 行）。

### 8. 常见陷阱
- **引号不一致**：`CATEGORY_COLORS` 键始终使用单引号，特别是带连字符的 ID（例如 `'glass-water'`）。普通单词键如 `lake` 可以不加引号，但加引号更安全且一致。
- **Git 跟踪**：放在 `Image/` 中的任何新文件必须添加到 git（`git add Image/...`），否则在 GitHub Pages 上不可用。
- **缓存失效**：如果获取的 `app.js` 格式更改，`parseAppJs()` 可能需要更新。注意浏览器控制台中的解析日志。
