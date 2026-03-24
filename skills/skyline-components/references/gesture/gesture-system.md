# 手势系统

## 概述

Skyline 内置了一套手势系统，直接在 UI 线程响应用户的手势操作，避免了传统 WXS 方案的跨线程通信延迟，使拖拽、缩放等交互动画更加流畅。

手势组件是"虚组件"，不参与布局渲染，事件由其直接子节点响应。

**版本要求**：需使用最新的 Nightly 工具进行调试。

## 手势组件列表

| 组件名 | 触发条件 | 典型用途 |
|--------|----------|----------|
| `tap-gesture-handler` | 点击 | 按钮点击、菜单选择 |
| `double-tap-gesture-handler` | 双击 | 图片放大、点赞 |
| `long-press-gesture-handler` | 长按 | 菜单弹出、删除确认 |
| `pan-gesture-handler` | 拖动（横向/纵向） | 拖拽排序、滑动删除 |
| `horizontal-drag-gesture-handler` | 横向滑动 | 左滑删除、抽屉菜单 |
| `vertical-drag-gesture-handler` | 纵向滑动 | 下拉刷新、页面返回 |
| `scale-gesture-handler` | 多指缩放 | 图片缩放、地图操作 |
| `force-press-gesture-handler` | iPhone 重按 | 3D Touch 快捷操作 |

## 手势状态

手势识别过程包含以下状态，通过回调参数中的 `state` 字段判断：

| 状态 | 值 | 说明 |
|------|-----|------|
| `POSSIBLE` | 0 | 手势未识别（手指刚接触屏幕） |
| `BEGIN` | 1 | 手势已识别（开始生效） |
| `ACTIVE` | 2 | 连续手势活跃状态（移动中） |
| `END` | 3 | 手势终止（手指离开） |
| `CANCELLED` | 4 | 手势取消（被中断或识别失败） |

**使用示例**：

```js
Page({
  handlePan(evt) {
    'worklet'
    if (evt.state === GestureState.BEGIN) {
      // 手势开始
    } else if (evt.state === GestureState.ACTIVE) {
      // 手势进行中
      this._offset.value += evt.deltaX
    } else if (evt.state === GestureState.END) {
      // 手势结束
    } else if (evt.state === GestureState.CANCELLED) {
      // 手势取消
    }
  }
})
```

## 通用属性

所有手势组件共享以下属性：

| 属性名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| `tag` | string | 否 | 手势协商时的组件标识 |
| `worklet:ongesture` | eventhandler | 否 | 手势识别成功的回调（必须为 Worklet） |
| `worklet:should-response-on-move` | callback | 否 | 手指移动过程中是否响应该手势 |
| `worklet:should-accept-gesture` | callback | 否 | 手势是否应该被识别 |
| `simultaneous-handlers` | Array&lt;string&gt; | 否 | 声明可同时触发的手势节点 tag 列表 |
| `native-view` | string | 否 | 代理的原生节点类型（如 `scroll-view`） |

## 基本用法

### pan-gesture-handler 拖拽

```html
<pan-gesture-handler worklet:ongesture="handlePan">
  <view class="draggable">可拖拽元素</view>
</pan-gesture-handler>
```

```js
Page({
  onLoad() {
    const offset = wx.worklet.shared(0)
    this.applyAnimatedStyle('.draggable', () => {
      'worklet'
      return {
        transform: `translateX(${offset.value}px)`
      }
    })
    this._offset = offset
  },
  handlePan(evt) {
    'worklet'
    if (evt.state === GestureState.ACTIVE) {
      this._offset.value += evt.deltaX
    }
  }
})
```

### scale-gesture-handler 缩放

```html
<scale-gesture-handler worklet:ongesture="handleScale">
  <image class="photo" src="{{url}}" />
</scale-gesture-handler>
```

```js
Page({
  onLoad() {
    const scale = wx.worklet.shared(1)
    this._scale = scale
    this.applyAnimatedStyle('.photo', () => {
      'worklet'
      return { transform: `scale(${scale.value})` }
    })
  },
  handleScale(evt) {
    'worklet'
    if (evt.state === GestureState.ACTIVE) {
      this._scale.value = evt.scale
    }
  }
})
```

## 回调参数

### pan/horizontal-drag/vertical-drag-gesture-handler

| 参数名 | 类型 | 说明 |
|--------|------|------|
| state | number | 手势状态（0-4） |
| absoluteX | number | 相对于全局（屏幕）的 X 坐标 |
| absoluteY | number | 相对于全局（屏幕）的 Y 坐标 |
| deltaX | number | 相对上一次回调，X 轴方向移动距离 |
| deltaY | number | 相对上一次回调，Y 轴方向移动距离 |
| velocityX | number | 手指离开时的横向速度（px/s） |
| velocityY | number | 手指离开时的纵向速度（px/s） |

### scale-gesture-handler

| 参数名 | 类型 | 说明 |
|--------|------|------|
| state | number | 手势状态 |
| focalX | number | 中心点相对于全局的 X 坐标 |
| focalY | number | 中心点相对于全局的 Y 坐标 |
| focalDeltaX | number | 相对上一次，中心点 X 轴移动距离 |
| focalDeltaY | number | 相对上一次，中心点 Y 轴移动距离 |
| scale | number | 累计缩放比例（1 为原始大小） |
| rotation | number | 旋转角度（单位：弧度） |
| pointerCount | number | 当前跟踪的手指数 |

### tap/double-tap-gesture-handler

| 参数名 | 类型 | 说明 |
|--------|------|------|
| state | number | 手势状态 |
| absoluteX | number | 相对于全局的 X 坐标 |
| absoluteY | number | 相对于全局的 Y 坐标 |

### long-press-gesture-handler

| 参数名 | 类型 | 说明 |
|--------|------|------|
| state | number | 手势状态 |
| absoluteX | number | 相对于全局的 X 坐标 |
| absoluteY | number | 相对于全局的 Y 坐标 |
| translationX | number | 相对于初始触摸点的 X 轴偏移量 |
| translationY | number | 相对于初始触摸点的 Y 轴偏移量 |

### force-press-gesture-handler

| 参数名 | 类型 | 说明 |
|--------|------|------|
| state | number | 手势状态 |
| absoluteX | number | 相对于全局的 X 坐标 |
| absoluteY | number | 相对于全局的 Y 坐标 |
| pressure | number | 压力大小（0-1 之间） |

## 使用规则

### MUST（必须遵守）

- **必须声明 `'worklet'` 指令**：所有手势回调函数开头必须声明
- **必须在 Skyline 模式下使用**：手势组件仅支持 Skyline 渲染
- **必须使用单子节点**：手势组件只能包含一个直接子节点

### NEVER（禁止行为）

- **禁止在 WebView 模式使用**：手势组件不支持 WebView
- **禁止在回调中执行耗时操作**：回调运行在 UI 线程，阻塞会导致卡顿
- **禁止忘记处理 CANCELLED 状态**：手势被中断时需要正确恢复状态

## 注意事项

1. **手势不冒泡**：手势事件与普通 touch 事件不同，不会发生冒泡
2. **回调在 UI 线程**：`worklet:ongesture` 等回调运行在 UI 线程，可直接操作 `shared` 值
3. **坐标参考系**：`absoluteX/Y` 相对于屏幕，`deltaX/Y` 相对于上一次回调
4. **速度单位**：`velocityX/Y` 单位为 pixels per second（每秒像素数）

## 示例代码片段

- [手势系统示例](https://developers.weixin.qq.com/s/A0ToYFmD79dA)
