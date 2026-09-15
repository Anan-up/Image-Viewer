[English](README.md) | [简体中文](README_Simplified_Chinese.md) | [繁體中文](README_Classical_Chinese.md)

## 一、總綱

此乃**絕依賴、純原生 JS、單文件**之本地圖像觀覽器。凡所處理，皆行於瀏覽器之內，未嘗上傳一紙。其界面用簡體中文，尚極簡白底之風（彷彿 macOS 預覽、Google Photos 之輕量者）。

---

## 二、功能列目

| 類 | 能 |
|---|---|
| 開 | 拖曳至窗／資料夾（遞歸展開）、擇文件、擇資料夾（`webkitdirectory`） |
| 覽 | 前張／次張、縮略條（逾一張方現）、循環往復 |
| 縮 | 滾輪（以指為錨）、雙指捏合、雙擊切 fit／一對一、按鈕／捷徑鍵 |
| 視 | 適窗（fit）、原大（百分百）、旋九十度、全屏 |
| 理 | 刪當前、Esc 清空、去重（路徑＋大小＋末改時） |
| 隱 | 放大逾 fit 則上下欄自隱，鼠移近上下緣四十四像素內暫現 |
| 式 | JPG／PNG／GIF／WebP／SVG／BMP／ICO／AVIF／TIF 之屬 |

---

## 三、架構與數流

```
狀態 state {scale, tx, ty, rot, index}   數據 items[{file,url,name,path,key}]
        ↓                                        ↓
    apply() 書 stage.transform            繪 img ／ thumbs ／ 欄文
```

- **單一 `<img>` 復用**：易圖唯易其 `src`，不重建 DOM。
- **stage 裹層**：`transform-origin:0 0`，圖以 `translate(-50%,-50%)` 自中心對齊 stage 之原點。故 `(tx,ty)` 之義甚明——**圖心在視口中之坐標**也。

---

## 四、核心要術

### 一、變換之模（其數甚潔）
```js
stage.transform = `translate(tx,ty) scale(s) rotate(rot)`
```
凡縮放、平移，唯改 `tx/ty/scale`，圖身不重排。

### 二、以指為錨之縮
```js
T' = m - k·(m - T)   // k = newScale/oldScale
```
`zoomAt` 與 `updatePinch` 共用一公式，捏合復疊雙指整移。如是則縮時「指下之像素」不動，其感自然。

### 三、⭐ P0 要術：fitScale 與 immersive 相解耦
此乃全篇最可稱道之修正。其患如下：

- 隱模式下視口由 `h-54` 變為 `h`；
- 若 fitScale 以**實際視口之高**計，則放大觸隱 → 視口增高 → fitScale 增 → 相對之 scale 減 → 退隱 → 視口減矮 → ……**死循環而震**。

其解：令 `computeFitScale()` 恒用 `win.h - BAR`，不繫於實際視口之高；`fit()` 之居中仍用真 `vp.h/2`（視覺即正）。判準既一，震自絕。

```js
// 基高恒為 win.h - BAR，與 immersive 解耦
return Math.min(win.w / w, Math.max(0, win.h - BAR) / h);
```

### 四、幾何之緩存，免強制同步佈局
```js
let vp  = {left,top,w,h}   // ResizeObserver 更之
let win = {w,h}            // resize 更之
let imgDim = {w,h}         // onload 更一次
```
`pointermove`、捏合、`checkEdgeReveal` 自始至終讀緩存，不讀 `getBoundingClientRect()`／`innerHeight`。此乃拖曳跟手流暢之要。

### 五、多點觸控之狀態機
以 `Map<pointerId, {x,y}>` 理之：

| 事 | 處 |
|---|---|
| 第一指按 | 入拖 |
| 第二指按 | 止拖，立捏合基線 |
| 第三指以上按 | 忽之，`pinch = null` 防抖 |
| 由二指鬆至一指 | 以餘指為新始續拖 |
| 由三指鬆回二指 | **無條件重建捏合基線**（P0，否則「既不拖亦不縮」而凍） |

佐以 `setPointerCapture`，指移出視口亦不遺其事。

### 六、過渡之控
拖曳／捏合／易圖定位時 `stage.style.transition='none'`，釋手或以 `requestAnimationFrame` 復之，以免「橡皮筋之滯」與「圖飛入」。

---

## 五、文件之處

- **去重之鍵**：`(webkitRelativePath||name) + size + lastModified`——免異目錄同名之誤判。
- **排序**：唯於**本批**依路徑 `localeCompare('zh')` 序之（累次拖入，全局之序或未善）。
- **拖曳目錄**：`webkitGetAsEntry()` → 遞歸 `createReader().readEntries()`，用 `do...while` 之環者，以 `readEntries` **每召至多返百條**，必屢召至空陣而後已。

---

## 六、無礙（A11y）之工，勝於常例

- 縮略 `role="button"` ＋ `tabIndex=0` ＋ `aria-label` ＋ `:focus-visible` 之框。
- 縮略內 `<img alt="">` 標為飾，免讀屏重報。
- 刪除鈕 `aria-hidden="true"`（刪統走 Delete 鍵，免 a11y 樹中義之歧）。
- `pct` 唯於百分比**實變**時方書於 DOM，且附 `aria-label`——免讀屏於拖曳時逐幀而報。
- 刪／清後，主移焦於當者（新當前縮略／「擇圖」之鈕），不令焦落 `body`。
- 空格鍵於焦在 `button/[role=button]/a/textarea/select` 時**不劫**，還其原生觸發。
- 圖標鈕自 `title` 補 `aria-label`。

---

## 七、內存與生死

- `URL.createObjectURL` ＋ 刪／清時 `revokeObjectURL`。
- **以 `pagehide` 易 `beforeunload`**，且察 `e.persisted`：若頁入 bfcache 則不收，否則前後返復後圖皆壞。此乃極常之正法。

---

## 八、註中 P0–P3 之標

此標識示其碼嘗歷一輪**系統之審／迭代**，註言「所以然」而非「所為」，其可讀遠勝常例：

- **P0**：致功能之疵者（fit／immersive 死循環、捏合基線凍）
- **P1**：傷用者（空格劫鈕、bfcache 收）
- **P2**：感／效之細（DOM 書之守、緩存讀幾何、隱態之遺）
- **P3**：琢磨者（焦之管、寸之復）

---

## 
![項目截圖](image-viewer.png)
---
## 許可

[MIT](LICENSE)
