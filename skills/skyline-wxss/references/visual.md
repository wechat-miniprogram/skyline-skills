# WXSS BNF 背景、边框与视觉效果

本文档定义 Skyline WXSS 支持的背景、边框、滤镜和遮罩属性。

> 标注说明：⚠️ 渲染层不支持 | ⛔ 整个属性不可用

---

## 背景属性

### background-color / background-image
```bnf
background-color = <color>
background-image = <bg-image>#
<bg-image> = none | <url> | <gradient>
```

### background-position / background-size
```bnf
background-position = <bg-position>#
background-size = <bg-size>#
<bg-size> = cover | contain | <length>{1,2}
```

### background-repeat
```bnf
background-repeat = <repeat-style>#
<repeat-style> = repeat-x | repeat-y | [ repeat | no-repeat | space | round ]{1,2}
```
> ⚠️ 渲染层不支持：`space`, `round`

### background-attachment ⛔ / background-clip ⛔ / background-origin ⛔
> ⛔ 以上三个属性整体不可用
>
> ⚠️ **`background` 简写部分有效**：解析器会将 `background` 简写展开为 10 个子属性，其中 `background-attachment`/`background-origin`/`background-clip` 被渲染层丢弃，其余 7 个子属性（color/image/repeat/position/position-x/position-y/size）正常生效。

---

## 渐变语法

```bnf
<linear-gradient> = linear-gradient( [ <angle> | to <side-or-corner> , ]? <color-stop-list> )
<radial-gradient> = radial-gradient( [ <ending-shape> || <size> ]? [ at <position> ]? , <color-stop-list> )
<conic-gradient> = conic-gradient( [ from <angle> ]? [ at <position> ]? , <angular-color-stop-list> )
```
> ⚠️ 渲染层渐变限制：
> - `linear-gradient` 颜色停止位置仅支持 `%` 和固定长度单位
> - `radial-gradient` 仅支持 `circle` 形状（不支持 `ellipse`）
> - `radial-gradient` 尺寸仅支持 `px` 单位
> - `radial-gradient` 颜色停止位置仅支持 `%` 单位
> - `background-image` / `mask-image` 最多支持 2 个值

---

## 边框属性

### border-width / border-style / border-color
```bnf
border-width = <line-width>{1,4}
<line-width> = thin | medium | thick | <length>

border-style = <line-style>{1,4}
<line-style> = none | solid | dotted | dashed | hidden | double | groove | ridge | inset | outset

border-color = <color>{1,4}
```
> ⚠️ border-style 渲染层不支持：`hidden`, `double`, `groove`, `ridge`, `inset`, `outset`

### border-radius / box-shadow
```bnf
border-radius = <length>{1,4} [ / <length>{1,4} ]?
box-shadow = none | <shadow>#
<shadow> = inset? && <length>{2,4} && <color>?
```

---

## 滤镜属性

### filter / backdrop-filter
```bnf
filter = none | <filter-function>+
backdrop-filter = none | <filter-function>+

<filter-function> = blur() | brightness() | contrast() | grayscale()
    | hue-rotate() | invert() | opacity() | saturate() | sepia()
```
> ⚠️ 渲染层不支持：`url()`, `drop-shadow()`（filter 和 backdrop-filter 均不支持）

---

## 遮罩属性

### mask-image / mask-size / mask-repeat / mask-position
```bnf
mask-image = <bg-image>#          /* ⚠️ 渐变未实现，仅 url() 可用 */
mask-size = <bg-size>#
mask-repeat = <repeat-style>#
mask-position = <bg-position>#    /* shorthand，解析时展开为 mask-position-x + mask-position-y */
mask-position-x = <bg-position-value>#
mask-position-y = <bg-position-value>#
```

### mask-origin ⛔ / mask-clip ⛔ / mask-mode ⛔
> ⛔ 以上属性整体不可用
>
> ⚠️ **`mask` 简写部分有效**：解析器会将 `mask` 简写展开为 8 个子属性，其中 `mask-origin`/`mask-clip`/`mask-position`（简写自身）被渲染层丢弃，其余子属性（mask-image/repeat/position-x/position-y/size）正常生效。

---

*本文档基于 Skyline 渲染引擎官方文档与实际测试生成。*
