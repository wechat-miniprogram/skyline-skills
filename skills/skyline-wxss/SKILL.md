---
name: skyline-wxss
description: Skyline WXSS 样式支持技能。提供 Skyline 渲染引擎实际支持的 CSS 属性、值和限制信息，包括解析层与渲染层的缺口分析。触发关键词：WXSS、CSS 支持、样式兼容性、属性支持、不支持的属性、display grid、position sticky、overflow、filter、渐变、gradient、border-style、white-space、font-weight、text-decoration。
---

# Skyline WXSS 样式支持

## 适用场景

- 检查某个 CSS 属性/值在 Skyline 渲染层是否可用
- 排查 WXSS 样式在 Skyline 下不生效的问题
- 从 WebView 迁移到 Skyline 时评估样式兼容性
- 使用渐变、滤镜、遮罩等高级样式时了解限制
- 查阅 Skyline 支持的 CSS 属性完整 BNF 语法

## 核心概念

### 三层架构

Skyline WXSS 处理经过三层，每层可能过滤不同的属性/值：

| 层级 | 引擎 | 作用 | 影响 |
|------|------|------|------|
| Layer 1 解析层 | CSS 解析器 | 解析 CSS 语法 | 不在语法定义中的属性直接丢弃 |
| Layer 2 样式应用层 | 样式处理器 | 映射到内部属性 | 部分属性被标记为无效并丢弃 |
| Layer 3 渲染层 | widget (C++) | 实际渲染 | 部分值在 ApplyValue 中无 case 分支，静默忽略 |

> 一个属性可能被解析层识别（BNF 有定义），但在样式应用层或渲染层被丢弃。**以渲染层实际支持为准。**

## 文档索引

| 我想要... | 查阅文档 |
|-----------|----------|
| 查看基础数据类型与 BNF 符号 | `references/basics.md` |
| 查看布局/定位/盒模型 BNF | `references/layout.md` |
| 查看 Flex 布局 BNF | `references/flex.md` |
| 查看背景/边框/滤镜/遮罩 BNF | `references/visual.md` |
| 查看文本/字体 BNF | `references/text.md` |
| 查看动画/过渡/变换 BNF | `references/animation.md` |
| 查看某属性渲染层是否支持 | 本文档 Quick Reference |
| 了解渐变/滤镜的具体限制 | 本文档「渐变与滤镜限制」规则 |

## 强制规则

### NEVER: 使用渲染层不支持的 CSS 属性

以下属性被解析器识别但渲染层**整体拒绝**，设置后不生效：

| 属性 | 分类 | 替代方案 |
|------|------|----------|
| `float` | 布局 | 使用 flex 布局 |
| `contain` | 布局 | 使用 `-wx-contain` |
| `resize` | 布局 | 无替代 |
| `writing-mode` | 文本 | 无替代 |
| `text-indent` | 文本 | 使用 padding-left 模拟 |
| `overflow-wrap` | 文本 | 使用 word-break |
| `text-decoration-thickness` | 文本 | 无替代 |
| `background-attachment` | 背景 | 无替代 |
| `background-clip` | 背景 | 无替代 |
| `background-origin` | 背景 | 无替代 |
| `mask-origin / mask-clip / mask-mode` | 遮罩 | 无替代 |
| `justify-items` | Flex | 无替代 |
| `list-style-type / list-style-image / list-style-position` | 列表 | 无替代 |

### MUST: 使用渲染层支持的属性值

部分属性可用但**特定值不支持**，会被静默忽略：

**Incorrect:**
```css
.container {
  display: grid;           /* grid 不支持 */
  position: sticky;        /* sticky 不支持 */
  overflow: auto;          /* auto 不支持 */
}
```

**Correct:**
```css
.container {
  display: flex;           /* 使用 flex 替代 grid */
  position: relative;      /* sticky 需使用 sticky-header 组件 */
  overflow: hidden;        /* 使用 hidden，滚动用 scroll-view */
}
```

### MUST: 注意渐变限制

**Incorrect:**
```css
.bg {
  /* radial-gradient 不支持 ellipse */
  background: radial-gradient(ellipse, red, blue);
  /* radial-gradient 尺寸仅支持 px */
  background: radial-gradient(circle 50%, red, blue);
}
```

**Correct:**
```css
.bg {
  /* 仅支持 circle + px 单位 */
  background: radial-gradient(circle 50px, red, blue);
  /* linear-gradient 停止位置仅支持 % 和固定长度 */
  background: linear-gradient(red 0%, blue 100%);
}
```

渐变限制汇总：
- `radial-gradient` 仅支持 `circle` 形状（不支持 `ellipse`）
- `radial-gradient` 尺寸仅支持 `px` 单位
- `radial-gradient` 颜色停止位置仅支持 `%` 单位
- `linear-gradient` 停止位置仅支持 `%` 和固定长度单位
- `background-image` / `mask-image` 最多支持 2 个值

### NEVER: 使用 filter 的 url() 和 drop-shadow()

**Incorrect:**
```css
.effect {
  filter: drop-shadow(2px 2px 4px black);  /* 不支持 */
  filter: url(#svg-filter);                /* 不支持 */
  backdrop-filter: drop-shadow(2px 2px 4px black); /* 不支持 */
}
```

**Correct:**
```css
.effect {
  filter: blur(4px) brightness(0.8);       /* blur 等函数正常支持 */
  box-shadow: 2px 2px 4px black;           /* 用 box-shadow 替代 drop-shadow */
}
```

filter/backdrop-filter 支持的函数：`blur()`, `brightness()`, `contrast()`, `grayscale()`, `hue-rotate()`, `invert()`, `opacity()`, `saturate()`, `sepia()`。

### MUST: text-decoration-line 仅支持单值

**Incorrect:**
```css
.text {
  text-decoration-line: underline line-through; /* 多值组合不支持 */
}
```

**Correct:**
```css
.text {
  text-decoration-line: underline; /* 仅支持单个值 */
}
```

### MUST: mask-image 渐变不可用

`mask-image` 的 `linear-gradient()` 和 `radial-gradient()` 功能未实现（NOTIMPLEMENTED），仅 `url()` 形式可用。

### MUST: 注意简写属性的部分有效行为

以下 CSS 简写属性在解析后会展开为多个子属性，但**部分子属性在渲染层被静默丢弃**：

| 简写属性 | 被丢弃的子属性 |
|---------|---------------|
| `background` | `background-attachment`, `background-origin`, `background-clip` |
| `text-decoration` | `text-decoration-thickness` |
| `mask` | `mask-origin`, `mask-clip`（`mask-position` 简写自身被丢弃但 x/y 子属性有效） |

**Incorrect:**
```css
.element {
  /* background-clip: text 不会生效，因为 background-clip 被丢弃 */
  background: linear-gradient(red, blue) text;
  /* text-decoration-thickness 不会生效 */
  text-decoration: underline solid red 2px;
}
```

**Correct:**
```css
.element {
  /* 只使用渲染层支持的分量 */
  background: linear-gradient(red, blue) no-repeat center/cover;
  /* 不指定 thickness，因为它会被丢弃 */
  text-decoration: underline solid red;
}
```

## Quick Reference

### 布局属性速查

| 属性 | 支持的值 | 不支持的值 |
|------|---------|-----------|
| `display` | `none`, `block`, `inline`, `inline-block`, `flex`, `inline-flex` | `grid`, `flow-root` |
| `position` | `static`, `relative`, `absolute`, `fixed` | `sticky` |
| `overflow` | `visible`, `hidden` | `auto`, `scroll` |
| `visibility` | `visible`, `hidden` | `collapse` |
| `box-sizing` | `content-box`, `border-box` | `padding-box` |

### 文本属性速查

| 属性 | 支持的值 | 不支持的值 |
|------|---------|-----------|
| `white-space` | `normal`, `nowrap` | `pre`, `pre-wrap`, `pre-line` |
| `font-weight` | `normal`, `bold`, 数值(100-900) | `bolder`, `lighter` |
| `font-style` | `normal`, `italic` | `oblique` |
| `word-break` | `normal`, `break-all`, `keep-all` | `break-word`（映射为 normal） |
| `vertical-align` | `baseline`, `top`, `middle`, `bottom` | `text-top`, `text-bottom` |
| `text-align` | `left`, `center`, `right`, `justify`, `start`, `end` | `justify-all`, `match-parent` |
| `text-decoration-line` | `none`, `underline`, `overline`, `line-through`（仅单值） | 多值组合, `blink` |

### 边框与背景速查

| 属性 | 支持的值 | 不支持的值 |
|------|---------|-----------|
| `border-*-style` | `none`, `solid`, `dashed`, `dotted` | `hidden`, `double`, `groove`, `ridge`, `inset`, `outset` |
| `background-repeat` | `repeat`, `repeat-x`, `repeat-y`, `no-repeat` | `space`, `round` |

### Flex 属性（全部支持）

以下 Flex 属性的所有 BNF 定义值在渲染层**完全支持**：
`flex-direction`, `flex-wrap`, `align-items`, `align-self`, `align-content`, `justify-content`, `flex-grow`, `flex-shrink`, `flex-basis`, `order`

### 完全支持的其他属性

`direction`, `pointer-events`, `text-decoration-style`, `background-size`, `transform`, `opacity`, `transition-*`, `animation-*`, `border-radius`, `box-shadow`, `text-shadow`, `color`, `font-size`, `font-family`, `line-height`, `letter-spacing`, `word-spacing`, `z-index`

## 相关技能

| 场景 | 推荐技能 | 说明 |
|------|----------|------|
| Skyline JSON 配置 | `skyline-config` | rendererOptions 影响默认 display/box-sizing |
| 组件使用 | `skyline-components` | scroll-view 替代 overflow:auto |
| Worklet 动画 | `skyline-worklet` | transform/opacity 动画最佳实践 |
| Skyline 概览 | `skyline-overview` | 渲染引擎概览与迁移指南 |

## References 目录结构

```
references/
├── basics.md      # 基础数据类型 + BNF 符号说明
├── layout.md      # 布局、定位、盒模型
├── flex.md        # Flex 布局
├── visual.md      # 背景、边框、滤镜、遮罩
├── text.md        # 文本、字体
└── animation.md   # 变换、过渡、动画
```
