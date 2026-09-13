[English](README.md) | [简体中文](README_Simplified_Chinese.md) | [繁體中文](README_Classical_Chinese.md)


## 一、整体定位

一个**纯本地、零依赖、单文件**的图片查看器（Web 应用）。所有处理都在浏览器内完成，不上传任何文件，无第三方库、无构建步骤，直接双击 HTML 即可运行。

---

## 二、核心功能

### 1. 图片加载方式（三种）
| 方式 | 实现 |
|---|---|
| 选择文件 | `<input type="file" multiple>` |
| 选择文件夹 | `<input webkitdirectory>`（Chrome/Edge 支持） |
| 拖拽到窗口 | 全局 dragenter/drop 事件 + 遮罩提示 |

- 用 `URL.createObjectURL()` 生成本地 URL，按 `webkitRelativePath` 用**中文 locale**排序。
- 支持格式：JPG/PNG/GIF/WebP/SVG/BMP/ICO/AVIF/TIF 等（MIME 判断 + 扩展名兜底）。

### 2. 视图变换（核心状态机）
```js
state = { scale, tx, ty, rot, index, fitScale }
```
- **平移**：Pointer 事件拖拽（`setPointerCapture`，支持触摸）
- **缩放**：滚轮以**指针位置为锚点**（公式 `T' = m - k(m - T)`），范围 0.02 ~ 60 倍
- **适应窗口**：按视口比例计算，且旋转 90°/270° 时**交换宽高**
- **实际大小**：回到 1:1
- **旋转**：每次 +90°
- **双击**：在「适应」与「100%」之间切换

### 3. 导航
- 上一张 / 下一张（循环，取模）
- 缩略图条（`items.length > 1` 时显示，自动滚动到当前项，可删除单张）
- 键盘快捷键：`←/→/空格/PageUp/PageDown` 翻页，`+/-/0/1/r/Delete/Esc` 等操作

### 4. 沉浸模式（亮点设计）
- 当 `scale > fitScale`（放大到溢出视口）时自动进入 `immersive`：
  - 上下工具栏淡出隐藏，视口铺满全屏
  - 鼠标移到窗口**上/下边缘 44px** 内临时唤出工具栏（`reveal`）
  - 缩回适应尺寸后自动退出沉浸模式

### 5. 其他
- 全屏（Fullscreen API，图标随状态切换）
- 删除当前图（`URL.revokeObjectURL` 释放内存）
- `Esc` 优先级：先退全屏，无全屏则清空所有图片回到空状态

---

## 三、代码结构

```
IIFE 闭包（严格模式）
├── 常量：MIN_SCALE / MAX_SCALE
├── state 状态对象
├── 变换函数：apply / zoomAt / zoomCenter / fit / actual / rotate
├── 渲染：renderCurrent / updateInfo / renderThumbs / updateThumbsActive / select
├── 文件：addFiles / removeCurrent / clearAll / showViewer / showEmpty
├── 交互：拖拽平移 / 滚轮缩放 / 双击 / 工具栏 / 缩略图 / 拖拽入窗
└── 全局：mousemove（边缘唤出）+ keydown 快捷键
```

**DOM 结构**：空状态 → 工具栏 → 视口(stage+img) → 缩略图条 → 拖拽遮罩 → 两个隐藏 input。

---

## 四、设计上的优点

1. **无内存泄漏**：删除/清空时正确 `revokeObjectURL`。
2. **变换组合干净**：`translate + scale + rotate` 统一作用于 stage，图片本身用 `translate(-50%,-50%)` 以中心为锚点。
3. **性能优化**：`will-change:transform`、`pointer-events:none` 防止图片干扰拖拽、`touch-action:none` 防止移动端默认手势。
4. **UI 极简克制**：白色主题、细线分隔、tabular-nums 数字对齐、留白充足。
5. **交互完整**：鼠标、触摸（Pointer Events 统一）、键盘三套输入都覆盖。

---
## 项目截图
---
## 许可证

[MIT](LICENSE)
