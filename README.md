# Shrimp's Alchemy Encyclopedia
[![en](https://img.shields.io/badge/docs-English-blue.svg)](README.md)
[![zh-CN](https://img.shields.io/badge/docs-%E4%B8%AD%E6%96%87-red.svg)](README.zh-CN.md)
The official encyclopedia of all of the recipes in the popular Shrimp's Alchemy merge game!



Official Link: [https://rockeyld.github.io/Shrimp-s-Alchemy-Encyclopedia](https://rockeyld.github.io/Shrimp-s-Alchemy-Encyclopedia/)

---

## Developer Notes for AI Assistants and Modders

### 1. Project Architecture
- **Single-file app**: Everything lives in `index.html` (embedded CSS + JavaScript). No build step, no frameworks.
- **Data source**: On load, it fetches the raw `app.js` from `https://raw.githubusercontent.com/shrimp0913/Shrimp-s-Alchemy/main/app.js`.
- **Fallback**: If the fetch fails, it falls back to `localStorage` cache key `shrimp_alchemy_cache`.
- **Parsing**: The function `parseAppJs(source)` (around line 412) regex-parses recipes, encrypted recipes, element names/icons, and `finalItems` from the raw JS text.
- **Data source indicator**: A small badge in the top-right shows data status (Live / Cached / Error). It auto-fades after 5 seconds (see `mainInit`, around line 1755).

### 2. Graph & Canvas System
- **Renderer**: HTML5 Canvas with a custom force-directed layout.
- **Nodes**: Each element is a node. Properties include `id`, `name`, `color`, `depth`, `isBase`, `isFinal`, `isGhost`, `isImplicitFinal`, `isDefaultColor`.
- **Edges**: Each recipe `A + B = Result` creates two directed edges (`A→Result`, `B→Result`).
- **Layout**: `runOrganicLayout()` runs a force simulation with repulsion and edge attraction. After settling, `startFloating()` runs a continuous gentle animation loop.
- **Camera**: Pan by dragging, zoom by scroll wheel or pinch. Variables: `camX`, `camY`, `camZoom`.

### 3. How to Change Element Colors
Colors are defined in the `CATEGORY_COLORS` object (around line 840). If a new element is fetched but not listed here, it defaults to **transparent green** (`rgba(102,187,106,0.3)`) with a semi-transparent border (`rgba(102,187,106,0.5)`). This makes unmapped elements visually distinct.

```js
const CATEGORY_COLORS = {
  'lake': '#4fc3f7',
  'glass-water': '#4dd0e1',  // IDs with hyphens MUST use quotes
  'heat': '#ff5722',
  // ...
};
```

Base elements (water, fire, earth, air) have their own `BASE_COLORS` object above it.

### 4. How to Add / Change Icons
The app uses **CSS mask icons** for UI elements (not the element icons from the fetched data, which are inline SVGs). To add a new UI icon:

1. Place the SVG file in the `Image/` folder (e.g., `Image/my-icon.svg`).
2. Add a CSS rule in the `<style>` block (around line 260):
   ```css
   .icon-my-icon {
     -webkit-mask-image: url('Image/my-icon.svg');
     mask-image: url('Image/my-icon.svg');
   }
   ```
3. Use it in HTML or JS: `<span class="icon icon-my-icon"></span>`

**Important**: New SVG files must be `git add`ed and pushed; otherwise GitHub Pages will return 404 and the icon will appear as a blank square.

### 5. Interaction & Highlighting
- **Hover / Click**: Highlights the element's direct upstream ingredients (orange) and downstream products (cyan). Implemented in `handleHover()` and `canvas.click`.
- **Search**: The search box filters elements by name/id and highlights their one-layer connections.
- **Dragon Studio**: A hardcoded set `DRAGON_STUDIO_ELEMENTS` (around line 541) marks special elements. Clicking the dragon button highlights only these nodes.
- **Theme**: Toggle between dark and light themes via CSS `body.light` and CSS variables. `getThemeCanvasColors()` returns canvas-specific colors for the current theme.

### 6. Modals & UI
- **Credit modal**: `#credit-modal` is toggled via the `.open` class. It displays the `SHRIMP'S ALCHEMY.svg` logo (with theme-aware `filter` CSS) and links to the original author. When open, hover highlights on the canvas are disabled (see `handleHover()`).
- **Stats**: Bottom-right panel shows total element and recipe counts.
- **Legend**: Bottom-left panel explains node/edge color meanings.

### 7. Ghost & Final Items
- **Ghost elements**: IDs that appear in recipes but have no name/icon definition in the fetched data. They are auto-created with a question-mark icon and dashed cyan border.
- **Final items**: Explicitly marked by the original game (`finalItems.add(...)`). They get a gold dashed border and a star badge in the tooltip.
- **Implicit finals**: Elements that are never used as ingredients in any recipe. They get a red dashed border.
- **Default-color elements**: Elements not listed in `BASE_COLORS` or `CATEGORY_COLORS` render as transparent green nodes (`rgba(102,187,106,0.3)` fill, `rgba(102,187,106,0.5)` border) to distinguish them from categorized elements. Tracked via `node.isDefaultColor` in `initNodes()` (around line 925).

### 8. Common Pitfalls
- **Quote inconsistency**: Always use single quotes for `CATEGORY_COLORS` keys, especially IDs with hyphens (e.g., `'glass-water'`). Plain word keys like `lake` are legal unquoted but quoting them is safer and consistent.
- **Git tracking**: Any new file placed in `Image/` must be added to git (`git add Image/...`) or it won't be available on GitHub Pages.
- **Cache invalidation**: If the fetched `app.js` changes format, `parseAppJs()` may need updates. Watch the browser console for parse logs.
