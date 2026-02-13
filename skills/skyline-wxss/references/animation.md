# WXSS BNF 变换、过渡与动画

本文档定义 Skyline WXSS 支持的变换、过渡和动画属性。

> ✅ 以下属性在渲染层**完全支持**。

---

## 变换属性

### transform
```bnf
transform = none | <transform-function>+

<transform-function> = matrix() | matrix3d()
    | translate() | translateX() | translateY() | translateZ() | translate3d()
    | scale() | scaleX() | scaleY() | scaleZ() | scale3d()
    | rotate() | rotateX() | rotateY() | rotateZ() | rotate3d()
    | skew() | skewX() | skewY()
    | perspective()
```

### transform-origin / opacity
```bnf
transform-origin = <length> && <length> && <length>?   /* 默认 50% 50% 0 */
opacity = <number>                                       /* 默认 1 */
```

---

## 过渡属性

### transition
```bnf
transition-property = none | <animatable-property>#
transition-duration = <time>#              /* 默认 0s */
transition-timing-function = <timing-function>#
transition-delay = <time>#                 /* 默认 0s */
```

---

## 动画属性

### animation
```bnf
animation-name = [ none | <custom-ident> | <string> ]#
animation-duration = <time>#
animation-timing-function = <timing-function>#
animation-delay = <time>#
animation-iteration-count = [ infinite | <number> ]#
animation-direction = [ normal | reverse | alternate | alternate-reverse ]#
animation-fill-mode = [ none | forwards | backwards | both ]#
animation-play-state = [ running | paused ]#
```

### @keyframes
```bnf
@keyframes <animation-name> {
  <keyframe-selector># { <declaration-list> }
}
<keyframe-selector> = from | to | <percentage>
```

---

## 缓动函数

```bnf
<timing-function> = linear | ease | ease-in | ease-out | ease-in-out
    | steps( <integer> [ , start | end | jump-* ]? )
    | cubic-bezier( <number> , <number> , <number> , <number> )
```

---

## 优化属性

### will-change
```bnf
will-change = auto | [ scroll-position | contents | <custom-ident> ]#
```

---

*本文档基于 Skyline 渲染引擎官方文档与实际测试生成。*
