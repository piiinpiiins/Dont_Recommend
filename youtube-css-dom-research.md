# YouTube.com CSS 與 DOM 結構研究報告

## 1. 技術架構

YouTube 的前端基於 **Google Polymer** 框架構建，使用 **Web Components** 標準（Custom Elements）。所有自訂元素遵循 `ytd-*`、`tp-yt-*`、`yt-*` 的命名慣例。

YouTube 使用 **Shady DOM**（而非原生 Shadow DOM），即使瀏覽器支援原生 Shadow DOM，YouTube 仍選擇使用 polyfill 來管理封裝。這是一個 **SPA（單頁應用程式）**，內容透過動態載入。

---

## 2. DOM 層級結構

```
<html>
├── <head>
└── <body>
    └── <ytd-app>                          ← 根應用元件
        ├── <ytd-masthead>                 ← 頂部導航欄 (固定定位, 高度 56px)
        │   ├── #masthead-container
        │   ├── ytd-topbar-logo-renderer   ← YouTube Logo
        │   ├── ytd-searchbox              ← 搜尋框
        │   ├── #voice-search-button
        │   └── #guide-button              ← 漢堡選單按鈕
        │
        ├── tp-yt-app-drawer#guide         ← 側邊導航抽屜
        │   ├── ytd-guide-entry-renderer   ← 完整側邊欄項目
        │   └── ytd-mini-guide-entry-renderer ← 迷你側邊欄項目
        │
        ├── ytd-mini-guide-renderer        ← 摺疊狀態的迷你側邊欄
        │
        └── ytd-page-manager#page-manager  ← 主要內容管理器
            │
            ├── [首頁] ytd-browse
            │   └── ytd-two-column-browse-results-renderer
            │       └── ytd-rich-grid-renderer     ← 影片網格容器
            │           ├── yt-chip-cloud-chip-renderer ← 分類標籤
            │           ├── ytd-rich-section-renderer   ← 區段
            │           └── #contents
            │               └── ytd-rich-item-renderer  ← 影片卡片
            │
            └── [觀看頁] ytd-watch-flexy
                ├── #primary
                │   ├── #player-container-outer    ← 播放器外層
                │   │   └── .html5-video-player
                │   │       └── .video-stream      ← <video> 元素
                │   ├── ytd-video-primary-info-renderer   ← 標題/按讚
                │   ├── ytd-video-secondary-info-renderer ← 頻道/描述
                │   └── ytd-comments#comments      ← 留言區
                │
                └── #secondary
                    ├── #related                   ← 推薦影片
                    ├── ytd-watch-next-secondary-results-renderer
                    ├── ytd-compact-video-renderer ← 側邊影片項目
                    └── ytd-playlist-panel-renderer
```

---

## 3. 核心 Custom Elements 清單

| 元素名稱 | 用途 |
|---|---|
| `ytd-app` | 根應用容器 |
| `ytd-masthead` | 頂部導航欄 |
| `ytd-page-manager` | 頁面路由管理 |
| `ytd-watch-flexy` | 觀看頁面 (支援劇院/全螢幕模式) |
| `ytd-rich-grid-renderer` | 首頁影片網格 |
| `ytd-rich-item-renderer` | 網格中的影片卡片 |
| `ytd-video-renderer` | 影片渲染器 |
| `ytd-compact-video-renderer` | 側邊欄緊湊影片 |
| `ytd-rich-section-renderer` | 網格區段 |
| `ytd-shelf-renderer` | 推薦影片列 |
| `ytd-reel-shelf-renderer` | Shorts 區塊 |
| `ytd-channel-renderer` | 頻道卡片 |
| `ytd-playlist-renderer` | 播放清單卡片 |
| `ytd-comments-header-renderer` | 留言標頭 |
| `ytd-menu-renderer` | 選單元件 |
| `yt-lockup-view-model` | 新式卡片封裝 |
| `tp-yt-app-drawer` | 側邊欄抽屜 |

---

## 4. CSS Custom Properties (設計 Tokens)

### 色彩系統 (`--yt-spec-*`)

```css
/* 品牌/背景 */
--yt-spec-brand-background-solid
--yt-spec-brand-background-primary
--yt-spec-brand-background-secondary
--yt-spec-general-background-a    /* 主要背景 */
--yt-spec-general-background-b
--yt-spec-general-background-c
--yt-spec-error-background

/* 文字 */
--yt-spec-text-primary            /* 主要文字色 */
--yt-spec-text-secondary          /* 次要文字色 */
--yt-spec-text-disabled           /* 禁用文字 */

/* 圖示 */
--yt-spec-icon-active-other
--yt-spec-icon-inactive
--yt-spec-icon-disabled

/* 互動 */
--yt-spec-call-to-action          /* CTA 按鈕色 */
--yt-spec-call-to-action-inverse
--yt-spec-brand-subscribe-button-background
```

### 舊式色彩變數 (`--yt-*`)

```css
--yt-swatch-primary
--yt-swatch-primary-darker
--yt-swatch-text
--yt-swatch-important-text
--yt-swatch-input-text
--yt-swatch-textbox-bg
--yt-swatch-icon-color
--yt-primary-text-color
--yt-secondary-text-color
--yt-tertiary-text-color
```

### 佈局變數 (`--ytd-*`)

```css
--ytd-masthead-height: 56px           /* 頂欄高度 */
--ytd-rich-grid-items-per-row          /* 網格每行數量 */
--ytd-watch-flexy-scrollbar-width
--ytd-watch-flexy-height-ratio
--ytd-watch-flexy-width-ratio
--ytd-watch-flexy-space-below-player
```

### 暗色模式色彩參考

| 用途 | 色碼 |
|---|---|
| 主要背景 | `#181818` |
| 次要背景 | `#212121` |
| 懸停背景 | `#3d3d3d` |
| 主要文字 | `#ffffff` |
| 次要文字 | `#aaaaaa` |

---

## 5. 佈局系統

### 首頁網格

```css
/* 主要容器使用 CSS Grid */
#contents {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(310px, 1fr));
  gap: 12px;
}

/* 響應式斷點控制每行數量 */
/* > 1367px: 5 items per row */
/* > 1024px: 4 items per row */
/* > 900px:  3 items per row */
/* > 768px:  2 items per row */
/* ≤ 480px:  1 item per row  */
```

### 觀看頁面

- 使用 **Flexbox** 兩欄佈局：`#primary`（影片+資訊）+ `#secondary`（推薦列）
- `#columns` 為外層容器
- 支援 `[theatre]` 屬性切換劇院模式
- 支援 `[fullscreen]` 屬性切換全螢幕

### 關鍵 ID 選擇器

| ID | 用途 |
|---|---|
| `#columns` | 觀看頁面兩欄容器 |
| `#primary` | 主要內容區 |
| `#secondary` | 側邊欄 |
| `#related` | 推薦影片 |
| `#contents` | 網格內容容器 |
| `#guide-spacer` | 側邊欄佔位 |
| `#movie_player` | 播放器容器 |

---

## 6. 主題系統

YouTube 使用 HTML 屬性切換主題：

```css
/* 主題屬性 */
[dark]                              /* 暗色模式 */
html[dark]                          /* 全站暗色 */

/* 進階主題控制（透過擴充） */
[it-black-theme]
[it-dawn-theme]
[it-default-dark-theme]
[it-schedule=system_preference_dark]
[it-schedule=system_preference_light]
```

---

## 7. 播放器 CSS 類別

```css
.html5-video-player       /* 播放器主容器 */
.html5-video-container    /* 影片容器 */
.video-stream             /* <video> 元素 */
.ytp-chrome-bottom        /* 播放器底部控制列 */
.ytp-progress-bar         /* 進度條 */
.player-theater-container /* 劇院模式容器 */
```

---

## 8. 總結

YouTube 是一個大規模使用 **Web Components + Polymer** 的生產級應用，特點包括：

- **Custom Elements** 命名規範 (`ytd-*`, `yt-*`, `tp-yt-*`)
- **CSS Custom Properties** 實現設計 token 化的主題系統
- **CSS Grid** 用於首頁影片網格，**Flexbox** 用於觀看頁面佈局
- **Shady DOM** 替代原生 Shadow DOM
- **屬性驅動**的狀態管理（`[dark]`, `[theatre]`, `[mini-guide-visible]`）

---

## 參考資料

- [Polymer/polymer Issue #5551 - Shadow DOM usage](https://github.com/Polymer/polymer/issues/5551)
- [Hacker News - YouTube's Polymer UI](https://news.ycombinator.com/item?id=17050358)
- [minimal-youtube CSS](https://github.com/0kzh/minimal-youtube/blob/master/styles.css)
- [YouTube Grid Customizer](https://gist.github.com/boydaihungst/67c46467059cd086ec82e57f7d912266)
- [better-youtube-grid](https://github.com/RayForst/better-youtube-grid)
- [YouTube Grids Userstyle](https://userstyles.world/style/12606/youtube-grids)
- [code-charity/youtube content-styles.css](https://github.com/code-charity/youtube/blob/585957d46d79e6606b650fb1d48fdaec2a5f9e01/content-styles.css)
- [move-youtube-masthead-to-bottom](https://gist.github.com/forivall/d5efc2301669a6df4c192aeb2e27d781)
- [YouTube BlackNight CSS Theme](https://github.com/Luzkan/YouTubeBlackNightCSS)
