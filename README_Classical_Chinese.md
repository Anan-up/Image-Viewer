[English](README.md) | [简体中文](README_Simplified_Chinese.md) | [繁體中文](README_Classical_Chinese.md)

## 一、定位

純本機、無倚賴、孤篇之圖覽器（網頁之應用也）。諸務咸行於瀏覽器之中，不傳片紙於外，不假第三方之庫，無營造之序；但雙擊 HTML，即能運轉。

---

## 二、功用

### 一、取圖之方（凡三）

| 方 | 術 |
|---|---|
| 選文件 | `<input type="file" multiple>` |
| 選文件夾 | `<input webkitdirectory>`（Chrome/Edge 可） |
| 曳入窗 | 全局 dragenter/drop 之會，掩以示意 |

- 以 `URL.createObjectURL()` 立本機之 URL，依 `webkitRelativePath`，以**華文 locale** 為序。
- 所容之式：JPG/PNG/GIF/WebP/SVG/BMP/ICO/AVIF/TIF 等（以 MIME 辨之，兼以擴展名備之）。

### 二、觀圖之變（樞機之態）

```js
state = { scale, tx, ty, rot, index, fitScale }
```

- **平移**：Pointer 之會拖曳（`setPointerCapture`，兼容觸屏）
- **縮放**：滾輪以**指針所在為錨**（其式 `T' = m - k(m - T)`），自 0.02 至 60 倍
- **適窗**：度視口之比而算；旋 90°／270° 則**易其廣袤**
- **實大**：歸於 1:1
- **旋轉**：每轉 +90°
- **雙擊**：於「適窗」與「百分」間往復

### 三、行遊

- 前圖／後圖（環行，取模）
- 縮略之條（`items.length > 1` 方顯，自滾至當前，可刪其一張）
- 鍵捷：`←/→/空格/PageUp/PageDown` 翻頁，`+/-/0/1/r/Delete/Esc` 諸用

### 四、沉浸之境（妙筆也）

- 當 `scale > fitScale`（放大而溢於視口），則自入 `immersive`：
  - 上下列之欄淡隱，視口充乎全屏
  - 鼠標近窗之**上／下緣 44px** 內，則暫現欄（`reveal`）
  - 縮歸適窗之度，則自出沉浸之境

### 五、雜項

- 全屏（Fullscreen API，圖標隨態而易）
- 刪當前之圖（`URL.revokeObjectURL` 以釋內存）
- `Esc` 之先：先退全屏；若無全屏，則盡清諸圖而歸空

---

## 三、章法

```
IIFE 閉之（嚴式）
├── 常量：MIN_SCALE / MAX_SCALE
├── state 之象
├── 變之函：apply / zoomAt / zoomCenter / fit / actual / rotate
├── 繪：renderCurrent / updateInfo / renderThumbs / updateThumbsActive / select
├── 文：addFiles / removeCurrent / clearAll / showViewer / showEmpty
├── 交：拖曳平移 / 滾輪縮放 / 雙擊 / 工具欄 / 縮略 / 曳入窗
└── 全：mousemove（緣而現）+ keydown 捷
```

**DOM 之構**：空態 → 欄 → 視口（stage＋img）→ 縮略條 → 曳掩 → 二隱 input。

---

## 四、匠意之美

一、**無洩內存**：刪／清之時，正用 `revokeObjectURL`。

二、**變合清爽**：`translate + scale + rotate` 一施於 stage；圖本以 `translate(-50%,-50%)`，以中心為錨。

三、**工於效**：`will-change:transform`、`pointer-events:none` 以防圖擾拖曳、`touch-action:none` 以遏移動端之常勢。

四、**UI 至簡**：白底之飾、細線為界、tabular-nums 以齊數、留白充裕。

五、**交備三端**：鼠、觸（Pointer Events 一之）、鍵，三者皆具。

---

## 圖鑑

---

## 版稅

[MIT](LICENSE)
