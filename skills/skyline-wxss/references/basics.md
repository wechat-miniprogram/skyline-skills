# WXSS BNF 基础数据类型

本文档定义 Skyline WXSS 支持的基础数据类型。

> 标注说明：
> - ⚠️ **渲染层不支持** = 解析器识别但渲染层丢弃/忽略
> - ⛔ **整个属性不可用** = 渲染层不支持该属性

---

## `<length>`
```bnf
<length> = <number> <length-unit> | <percentage> | auto | <calc()> | <env()>
<length-unit> = px | rpx | vw | vh | vmin | vmax | rem | em
<env()> = env( <custom-ident> [ , <length> ]? )
```
支持的 env() 变量：`safe-area-inset-top`, `safe-area-inset-right`, `safe-area-inset-bottom`, `safe-area-inset-left`

## `<color>`
```bnf
<color> = <named-color> | <hex-color> | rgb() | rgba() | hsl() | hsla() | hwb() | currentColor
<hex-color> = '#' <hex-digit>{3,4,6,8}
```

## `<angle>` / `<time>`
```bnf
<angle> = <number> [ deg | grad | rad | turn ]
<time> = <number> [ s | ms ]
```

---

## BNF 符号说明

| 符号 | 含义 |
|------|------|
| `=` | 定义 |
| `\|` | 互斥选择 |
| `\|\|` | 可任意组合 |
| `&&` | 必须全部出现（顺序可变） |
| `?` | 可选 |
| `+` | 至少 1 次 |
| `{n,m}` | n 到 m 次 |
| `#` | 逗号分隔列表 |

---

*本文档基于 Skyline 渲染引擎官方文档与实际测试生成。*
