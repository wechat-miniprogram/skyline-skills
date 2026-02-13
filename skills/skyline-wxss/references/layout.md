# WXSS BNF 布局与定位属性

本文档定义 Skyline WXSS 支持的布局、定位和盒模型属性。

> 标注说明：⚠️ 渲染层不支持 | ⛔ 整个属性不可用

---

## 布局属性

### display
```bnf
display = none | block | inline | inline-block | flex | inline-flex | grid | flow-root
```
默认值：`inline`
> ⚠️ 渲染层不支持：`grid`, `flow-root`

### position
```bnf
position = static | relative | absolute | fixed | sticky
```
默认值：`static`
> ⚠️ 渲染层不支持：`sticky`

### overflow / overflow-x / overflow-y
```bnf
overflow = <overflow-value>{1,2}
<overflow-value> = visible | hidden | auto | scroll
```
默认值：`visible`
> ⚠️ 渲染层不支持：`auto`, `scroll`（滚动请使用 scroll-view 组件）

### visibility
```bnf
visibility = visible | hidden | collapse
```
> ⚠️ 渲染层不支持：`collapse`

### box-sizing
```bnf
box-sizing = content-box | padding-box | border-box
```
默认值：`content-box`
> ⚠️ 渲染层不支持：`padding-box`

### float ⛔ / contain ⛔ / resize ⛔
> ⛔ 整个属性不可用（contain 可用 `-wx-contain` 替代）

### aspect-ratio / content
```bnf
aspect-ratio = auto | <number> [ '/' <number> ]?
content = none | normal | <string> | <url>
```

---

## 定位属性

### top / right / bottom / left
```bnf
top | right | bottom | left = <length>
```
默认值：`auto`

### z-index
```bnf
z-index = auto | <integer>
```

---

## 盒模型属性

### width / height / min-* / max-*
```bnf
width | height | min-width | min-height | max-width | max-height = <length>
```

### padding / margin
```bnf
padding = <length>{1,4}
margin = <length>{1,4}
```
margin 支持 `auto` 值

---

*本文档基于 Skyline 渲染引擎官方文档与实际测试生成。*
