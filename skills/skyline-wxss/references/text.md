# WXSS BNF 文本属性

本文档定义 Skyline WXSS 支持的文本和字体属性。

> 标注说明：⚠️ 渲染层不支持 | ⛔ 整个属性不可用

---

## 基础文本

### color / font-size / line-height
```bnf
color = <color>             /* 继承，默认 #000 */
font-size = <length>        /* 继承 */
line-height = normal | <number> | <length>   /* 继承 */
```

---

## 字体属性

### font-weight
```bnf
font-weight = normal | bold | bolder | lighter | <number>
```
> ⚠️ 渲染层不支持：`bolder`, `lighter`

### font-style
```bnf
font-style = normal | italic | oblique [ <angle> ]?
```
> ⚠️ 渲染层不支持：`oblique`

### font-family / font-feature-settings
```bnf
font-family = [ <family-name> | <generic-family> ]#   /* 继承 */
font-feature-settings = normal | <feature-tag-value>#  /* 继承 */
```

---

## 空白与换行

### white-space
```bnf
white-space = normal | nowrap | pre | pre-wrap | pre-line | -wx-pre-edit
```
> ⚠️ 渲染层不支持：`pre`, `pre-wrap`, `pre-line`, `-wx-pre-edit`

### word-break
```bnf
word-break = normal | break-word | break-all | keep-all
```
> ⚠️ 渲染层：`break-word` 被映射为 `normal`

---

## 对齐

### text-align
```bnf
text-align = left | center | right | justify | justify-all | start | end | match-parent
```
> ⚠️ 渲染层不支持：`justify-all`, `match-parent`

### vertical-align
```bnf
vertical-align = baseline | top | middle | bottom | text-top | text-bottom
```
> ⚠️ 渲染层不支持：`text-top`, `text-bottom`

---

## 装饰

### text-decoration-line
```bnf
text-decoration-line = none | [ underline || overline || line-through || blink ]
```
> ⚠️ 渲染层仅支持**单值**（`none`, `underline`, `overline`, `line-through`），不支持多值组合

### text-decoration-style / text-decoration-color
```bnf
text-decoration-style = solid | double | dotted | dashed | wavy   /* ✅ 全部支持 */
text-decoration-color = <color>
```

### text-overflow
```bnf
text-overflow = clip | ellipsis
```

### text-shadow / letter-spacing / word-spacing
```bnf
text-shadow = none | [ <length>{2,3} && <color>? ]#
letter-spacing = normal | <length>              /* 继承 */
word-spacing = normal | <length>                /* 继承 */
```

---

## 不可用属性

### text-indent ⛔ / overflow-wrap ⛔ / writing-mode ⛔ / text-decoration-thickness ⛔
> ⛔ 以上属性整体不可用
>
> ⚠️ **`text-decoration` 简写部分有效**：解析器会将 `text-decoration` 简写展开为 4 个子属性，其中 `text-decoration-thickness` 被渲染层丢弃，其余 3 个子属性（line/style/color）正常生效。

### list-style-type ⛔ / list-style-image ⛔ / list-style-position ⛔
> ⛔ 整个列表类别不可用

---

*本文档基于 Skyline 渲染引擎官方文档与实际测试生成。*
