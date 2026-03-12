# react-ssg

基於 [vite-react-ssg](https://github.com/Daydreamer-riri/vite-react-ssg) + [vite-plugin-pages](https://github.com/hannoeru/vite-plugin-pages) 的 React SSG 基礎模板，支援檔案系統路由、頁面級 CSS 隔離、靜態預渲染，可部署至 GitHub Pages。

## 技術棧

| 套件 | 用途 |
|------|------|
| [rolldown-vite](https://vite.dev/guide/rolldown) | 建置工具（Rolldown 後端） |
| [@vitejs/plugin-react](https://github.com/vitejs/vite-plugin-react) | React + Babel + React Compiler |
| [vite-react-ssg](https://github.com/Daydreamer-riri/vite-react-ssg) | 靜態預渲染（SSG） |
| [vite-plugin-pages](https://github.com/hannoeru/vite-plugin-pages) | 檔案系統路由自動生成 |
| [react-router-dom](https://reactrouter.com/) | 客戶端路由（由 vite-react-ssg 管理） |
| [styled-components](https://styled-components.com/) | 頁面級 CSS 注入 |

## 指令

```bash
npm run dev        # SSG 開發模式（vite-react-ssg dev）
npm run build      # SSG 靜態建置輸出至 dist/
npm run preview    # 預覽 dist/
npm run deploy     # 建置並部署至 GitHub Pages

npm run vite-dev   # 純 Vite 開發模式（不含 SSG，用於對比測試）
npm run vite-build # 純 Vite 建置（不含 SSG）
```

---

## 路由規則

路由由 `vite-plugin-pages` 根據 `src/pages/` 目錄結構自動生成，對應規則如下：

```
src/pages/
├── index.jsx          →  /
├── about.jsx          →  /about
└── app/
    └── index.jsx      →  /app
```

生成的路由會自動傳入 `vite-react-ssg`，無需手動維護 route config。

### 頁面間導航

使用 `react-router-dom` 的 `<Link>` 元件進行 SPA 導航：

```jsx
import { Link } from "react-router-dom";

<Link to="/app">前往 App 頁</Link>
```

---

## CSS 用法

本模板採用**頁面級 CSS 隔離**方案：每個頁面的樣式透過 `styled-components` 的 `createGlobalStyle` 注入，頁面切換時自動卸載。

### 使用方式

在頁面的 CSS 檔案旁建立 `style.css`，以 `?raw` 方式引入：

```jsx
import { createGlobalStyle } from "styled-components";
import css from "./style.css?raw";

const PageStyles = createGlobalStyle`${css}`;

export default function MyPage() {
  return (
    <>
      <PageStyles />
      {/* 頁面內容 */}
    </>
  );
}
```

`?raw` 將 CSS 作為純字串引入，`createGlobalStyle` 負責在 `<head>` 動態注入與移除。

### 全域樣式

`src/global.css` 在 `main.jsx` 中統一引入，適合放 reset、字型、`:root` 變數等全站共用樣式。

---

## 圖片載入

本模板示範兩種圖片載入方式：

### 方式一：ES Module import（`src/assets/`）

適合需要讓 Vite 處理 hash 檔名、最佳化的靜態資源：

```jsx
import reactLogo from "../../assets/react.svg";

<img src={reactLogo} alt="React logo" />
```

建置後檔名會帶 hash（如 `react-BUBnl1aA.svg`），適合長期快取。

### 方式二：`public/` 目錄（絕對路徑）

適合不需要 hash、必須保持固定路徑的資源（如 favicon、Open Graph 圖片）：

```jsx
import viteLogo from "/vite.svg";   // 對應 public/vite.svg

<img src={viteLogo} alt="Vite logo" />
```

建置後路徑不變，直接複製到 `dist/` 根目錄。

---

## SSG / SSR

### SSG（靜態預渲染）

`npm run build` 會為每個路由生成對應的 `.html` 檔案：

```
dist/
├── index.html      ←  /
└── app/
    └── index.html  ←  /app
```

每個 HTML 已包含預渲染的 HTML 結構，對 SEO 和首屏速度有利。

### 運作原理

`src/main.jsx` 使用 `ViteReactSSG` 取代標準的 `ReactDOM.createRoot`：

```jsx
import { ViteReactSSG } from "vite-react-ssg";
import routes from "~react-pages";
import "./global.css";

export const createRoot = ViteReactSSG(
  { routes, basename: import.meta.env.BASE_URL },
  ({ router, isClient, initialState }) => {
    // 可在此進行 plugin 初始化（如 pinia、i18n）
  }
);
```

- `~react-pages`：由 `vite-plugin-pages` 自動生成的路由陣列
- `basename`：自動讀取 `vite.config.js` 的 `base` 設定，部署子路徑時不需額外調整
- 建置時為 Node.js 環境執行（SSR），請避免在模組頂層直接存取 `window` / `document`

### 判斷執行環境

```jsx
import { useSSRContext } from "vite-react-ssg";

// 或直接判斷
if (typeof window !== "undefined") {
  // 僅在客戶端執行
}
```

---

## 部署至 GitHub Pages

1. 在 `vite.config.js` 設定 `base`：
   ```js
   base: "/your-repo-name",
   ```

2. 在 `package.json` 確認 `homepage`：
   ```json
   "homepage": "https://your-name.github.io/your-repo-name/"
   ```

3. 執行部署：
   ```bash
   npm run deploy
   ```
