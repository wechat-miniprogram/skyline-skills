# 手势协商

## 概述

手势协商是 Skyline 手势系统中解决手势冲突的核心机制。当多个手势组件嵌套时，默认行为是内层手势优先识别，一旦内层手势被激活，外层手势将失效。通过手势协商，可以让多个手势同时生效，并精确控制它们的触发条件。

## 冲突场景分析

### 典型冲突 1：嵌套滚动

**场景**：底部弹窗中包含可滚动列表
- 期望行为：列表滚动到顶部后，继续下拉可关闭弹窗
- 默认行为：列表始终响应滚动，弹窗无法通过下拉关闭

### 典型冲突 2：缩放与拖动

**场景**：图片查看器同时支持缩放和拖动
- 期望行为：单指拖动移动图片，双指缩放
- 默认行为：双指操作时只有缩放或拖动生效

### 典型冲突 3：横向滑动与页面返回

**场景**：列表项左滑删除功能
- 期望行为：左滑显示删除按钮
- 默认行为：可能与系统的右滑返回手势冲突

## simultaneous-handlers 属性

### 基本用法

通过 `simultaneous-handlers` 属性声明允许同时触发的手势：

```html
<!-- 外层手势 -->
<vertical-drag-gesture-handler 
  tag="outer" 
  simultaneous-handlers="{{['inner']}}"
>
  <!-- 内层手势 -->
  <vertical-drag-gesture-handler 
    tag="inner" 
    simultaneous-handlers="{{['outer']}}"
  >
    <view>内容</view>
  </vertical-drag-gesture-handler>
</vertical-drag-gesture-handler>
```

**关键点**：
1. 使用 `tag` 属性为手势组件命名
2. 双方都需要在 `simultaneous-handlers` 中声明对方的 tag
3. 声明后，两个手势的回调都会被触发

### 三层及以上嵌套

```html
<pan-gesture-handler tag="a" simultaneous-handlers="{{['b', 'c']}}">
  <pan-gesture-handler tag="b" simultaneous-handlers="{{['a', 'c']}}">
    <pan-gesture-handler tag="c" simultaneous-handlers="{{['a', 'b']}}">
      <view>内容</view>
    </pan-gesture-handler>
  </pan-gesture-handler>
</pan-gesture-handler>
```

## worklet:should-accept-gesture

在手势识别早期阶段决定是否接受该手势：

```js
Page({
  shouldAcceptGesture(evt) {
    'worklet'
    // 只在特定区域接受手势
    if (evt.absoluteX < 100) {
      return false  // 不接受
    }
    return true  // 接受
  }
})
```

**调用时机**：手势刚开始识别时，仅调用一次。

**适用场景**：
- 根据触摸位置判断是否响应
- 根据当前状态（如是否在编辑模式）判断

## worklet:should-response-on-move

在手势进行中动态控制是否响应：

```js
Page({
  shouldResponseOnMove(evt) {
    'worklet'
    // 滚动到顶部后不再响应列表滚动
    if (this._scrollTop.value <= 0 && evt.deltaY > 0) {
      return false
    }
    return true
  }
})
```

**调用时机**：每次手指移动时都会调用。

**适用场景**：
- 列表滚动到边界后切换行为
- 根据滑动方向动态判断

## native-view 原生组件代理

对于 `scroll-view`、`swiper` 等原生组件，需要使用 `native-view` 属性代理其内部手势：

```html
<vertical-drag-gesture-handler 
  native-view="scroll-view"
  worklet:should-response-on-move="shouldRespond"
>
  <scroll-view scroll-y>
    <!-- 内容 -->
  </scroll-view>
</vertical-drag-gesture-handler>
```

**支持的 native-view 值**：
- `scroll-view`：代理滚动视图手势
- `swiper`：代理轮播组件手势

## 完整示例：评论区效果

实现类似视频号的评论区——列表滚动到顶部后继续下拉可关闭面板：

```html
<!-- 外层：控制面板拖动 -->
<vertical-drag-gesture-handler 
  tag="sheet" 
  simultaneous-handlers="{{['list']}}"
  worklet:ongesture="handleSheetDrag"
>
  <view class="sheet">
    <!-- 内层：控制列表滚动 -->
    <vertical-drag-gesture-handler 
      tag="list" 
      simultaneous-handlers="{{['sheet']}}"
      native-view="scroll-view"
      worklet:should-response-on-move="shouldListRespond"
    >
      <scroll-view 
        scroll-y 
        type="list"
        worklet:onscrollupdate="handleScrollUpdate"
      >
        <!-- 评论列表 -->
      </scroll-view>
    </vertical-drag-gesture-handler>
  </view>
</vertical-drag-gesture-handler>
```

```js
Page({
  onLoad() {
    this._scrollTop = wx.worklet.shared(0)
    this._sheetOffset = wx.worklet.shared(0)
    
    this.applyAnimatedStyle('.sheet', () => {
      'worklet'
      return { transform: `translateY(${this._sheetOffset.value}px)` }
    })
  },
  
  handleScrollUpdate(evt) {
    'worklet'
    this._scrollTop.value = evt.detail.scrollTop
  },
  
  shouldListRespond(evt) {
    'worklet'
    // 在顶部且下拉时，不响应列表滚动
    if (this._scrollTop.value <= 0 && evt.deltaY > 0) {
      return false
    }
    // 面板已拉下时，不响应列表滚动
    if (this._sheetOffset.value > 0) {
      return false
    }
    return true
  },
  
  handleSheetDrag(evt) {
    'worklet'
    const { spring } = wx.worklet
    
    if (evt.state === GestureState.ACTIVE) {
      // 只在列表顶部或面板已拉下时响应
      if (this._scrollTop.value <= 0 || this._sheetOffset.value > 0) {
        const newOffset = this._sheetOffset.value + evt.deltaY
        this._sheetOffset.value = Math.max(0, newOffset)
      }
    } else if (evt.state === GestureState.END) {
      // 下拉超过阈值或速度够快则关闭
      if (this._sheetOffset.value > 200 || evt.velocityY > 1000) {
        this._sheetOffset.value = spring(500)
        wx.worklet.runOnJS(this.closeSheet.bind(this))()
      } else {
        this._sheetOffset.value = spring(0)
      }
    }
  },
  
  closeSheet() {
    wx.navigateBack()
  }
})
```

## 最佳实践

### 1. 明确手势职责

为每个手势组件设定清晰的职责：
- 外层手势：控制整体容器
- 内层手势：控制内部交互

### 2. 使用 shared 值同步状态

通过 `shared` 值在多个手势回调间共享状态：

```js
this._isListAtTop = wx.worklet.shared(true)
this._isDraggingSheet = wx.worklet.shared(false)
```

### 3. 处理边界情况

确保在各种边界情况下都有正确行为：
- 快速滑动
- 滑动方向改变
- 手势取消

### 4. 提供视觉反馈

手势切换时提供平滑的视觉过渡：

```js
// 使用 spring 动画而非直接设置值
this._offset.value = spring(targetValue)
```

## 注意事项

1. **性能考虑**：`should-response-on-move` 在每次移动时调用，避免复杂计算
2. **状态同步**：使用 `shared` 值确保多个回调间状态一致
3. **降级处理**：为不支持手势系统的环境提供替代方案
4. **调试技巧**：使用 `console.log` 在 worklet 函数中输出状态，便于调试
