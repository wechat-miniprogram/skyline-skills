# WXSS BNF Flex 布局属性

本文档定义 Skyline WXSS 支持的 Flex 布局属性。

> ✅ 以下 Flex 属性的所有值在渲染层**完全支持**（除特别标注外）。

---

## 容器属性

### flex-direction / flex-wrap
```bnf
flex-direction = row | row-reverse | column | column-reverse
flex-wrap = nowrap | wrap | wrap-reverse
```

### align-items / align-self / align-content / justify-content
```bnf
align-items = stretch | center | flex-start | flex-end | baseline | normal | start | end
align-self = auto | stretch | center | flex-start | flex-end | baseline | start | end | normal
align-content = stretch | center | flex-start | flex-end | space-between | space-around | space-evenly | normal | start | end
justify-content = center | flex-start | flex-end | space-between | space-around | space-evenly | start | end
```

---

## 项目属性

### flex / flex-grow / flex-shrink / flex-basis / order
```bnf
flex = none | auto | [ <flex-grow> <flex-shrink>? || <flex-basis> ]
flex-grow = <number>          /* 默认 0 */
flex-shrink = <number>        /* 默认 1 */
flex-basis = <length>         /* 默认 auto */
order = <integer>             /* 默认 0 */
```

---

## 不可用属性

### justify-items ⛔
```bnf
justify-items = stretch | center | flex-start | flex-end | start | end | self-start | self-end | left | right
```
> ⛔ 整个属性不可用

---

*本文档基于 Skyline 渲染引擎官方文档与实际测试生成。*
