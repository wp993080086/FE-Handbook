# 📚 目录

1. [transition过渡组件](#1-transition过渡组件)
2. [属性](#2-属性)
3. [事件](#3-事件)
4. [原理](#4-原理)
5. [使用场景](#5-使用场景)
6. [实用技巧](#6-实用技巧)
7. [注意事项](#7-注意事项)
8. [性能优化](#8-性能优化)
---

# 1. transition过渡组件

<transition> 组件用于在元素进入或离开 DOM 时应用过渡效果。它是 Vue 内置的组件，无需额外安装即可使用。

# 2. 属性

属性名 | 类型 | 默认值| 可选值 | 说明
---|---|---|---|---
name |string |'v' | - |用于自动生成 CSS 过渡类名前缀，例如：name: 'fade' 将自动拓展为 .fade-enter，.fade-enter-active 等 |
appear | boolean |false | -| 是否在初始渲染时使用过渡 |
css | boolean |true | -|是否使用 CSS 过渡类，如果设置为 false，将只通过组件事件触发注册的 JavaScript 钩子 |
type | string |- |'transition' 'animation' |指定过渡事件类型，默认会自动检测 |
mode | string |- |'out-in' 'in-out' |控制离开/进入过渡的时间序列，默认情况下两个元素会同时执行过渡 |
duration | number/object |- |{ enter: number, leave: number } |指定过渡的持续时间，单位为毫秒 |
enter-class | string | -| -|自定义进入过渡的初始类名 |
leave-class | string | -| -|自定义离开过渡的初始类名 |
appear-class | string | |- |自定义初始渲染的过渡类名 |
enter-to-class | string |- |- |自定义进入过渡结束时的类名 |
leave-to-class | string |- |- |自定义离开过渡结束时的类名 |
enter-active-class | string | -|- |自定义进入过渡的活跃状态类名 |
leave-active-class | string |- |- |自定义离开过渡的活跃状态类名 |
appear-active-class | string |- |- | 自定义初始渲染的过渡活跃状态类名 |

# 3. 事件

事件名 | 说明 |
---|---|
 before-enter | 进入过渡开始前触发 |
 before-leave | 离开过渡开始前触发 |
 before-appear | 初始渲染过渡开始前触发 |
 enter | 进入过渡开始时触发 |
 leave | 离开过渡开始时触发 |
 appear | 初始渲染过渡开始时触发 |
 after-enter | 进入过渡完成后触发 |
 after-leave | 离开过渡完成后触发 |
 after-appear | 初始渲染过渡完成后触发 |
 enter-cancelled | 进入过渡被取消时触发 |
 leave-cancelled | 离开过渡被取消时触发，只有v-show有 |
 appear-cancelled | 初始渲染过渡被取消时触发 |

# 4. 原理

总结起来就是，通过添加和移除类名来触发 CSS 的 transition 或 animation ，来达到过渡效果。步骤如下：

1. 挂载阶段：当包裹在 <transition> 内的元素首次渲染到 DOM 时，它会立即显示，没有过渡效果。

2. DOM 更新：当触发了元素的显示或隐藏操作，比如修改了 v-if、v-show、或者条件渲染的数据，Vue 会执行一次重新渲染。

3. 过渡类名添加：在重新渲染过程中，Vue 会根据过渡效果的配置为元素添加一系列类名，这些类名用于触发 CSS 过渡效果，比如v-enter，v-enter-active，v-enter-to，v-leave等等

4. 过渡执行：通过添加上述的类名，触发了 CSS 过渡效果。

5. 过渡结束：一旦过渡效果执行完毕，Vue 会自动移除相应的过渡类名，使元素回到普通状态。

6. 钩子函数：如果需要在过渡过程中执行一些 JavaScript 逻辑，可以使用 @enter、@leave 等事件钩子函数。

# 5. 使用场景

1. 列表、按钮，文字，各种过渡效果
2. 页面切换动画等
3. 表单验证反馈动画
4. 模态框/抽屉的显示隐藏
5. 图片懒加载时的淡入效果

# 6. 实用技巧

### 6.1 自定义过渡类

通过自定义过渡类，可以实现更复杂的过渡效果：

```javascript
<template>
  <div>
    <transition name="bounce" mode="out-in">
      <div v-show="show">Bounce!</div>
    </transition>
    <button @click="show = !show">Toggle</button>
  </div>
</template>

<style>
.bounce-enter-active {
  animation: bounce-in 0.5s;
}
.bounce-leave-active {
  animation: bounce-in 0.5s reverse;
}
@keyframes bounce-in {
  0% { transform: scale(0); }
  50% { transform: scale(1.2); }
  100% { transform: scale(1); }
}
</style>
```

### 6.2 JavaScript 钩子

使用 JavaScript 钩子可以实现更精细的动画控制：

```javascript
<template>
  <transition
    @enter="enter"
    @leave="leave"
    :css="false"
  >
    <div v-show="show">JavaScript Animation</div>
  </transition>
</template>

<script>
export default {
  methods: {
    enter(el, done) {
      // 设置初始状态
      el.style.opacity = 0
      el.style.transform = 'translateY(20px)'
      
      // 使用 requestAnimationFrame 确保浏览器渲染初始状态
      requestAnimationFrame(() => {
        requestAnimationFrame(() => {
          el.style.transition = 'opacity 300ms, transform 300ms'
          el.style.opacity = 1
          el.style.transform = 'translateY(0)'
          
          // 动画结束后调用 done
          setTimeout(done, 300)
        })
      })
    },
    leave(el, done) {
      el.style.transition = 'opacity 300ms, transform 300ms'
      el.style.opacity = 0
      el.style.transform = 'translateY(20px)'
      setTimeout(done, 300)
    }
  }
}
</script>
```

# 7. 注意事项

1. <transition> 组件只包裹一个元素或组件，如果需要过渡多个元素，需要使用 <transition-group>
2. 过渡模式（mode）很重要，特别是在切换两个元素时，使用 mode="out-in" 可以避免两个元素同时显示
3. 确保 CSS 过渡属性正确设置，否则过渡可能不会生效
4. 使用 v-show 而不是 v-if 可以避免元素的频繁创建和销毁，提高性能
5. 对于复杂的动画效果，考虑使用 JavaScript 钩子配合 requestAnimationFrame 实现

# 8. 性能优化

1. 优先使用 transform 和 opacity 进行过渡，因为它们不会触发重排（reflow）
2. 使用 will-change 属性提前告知浏览器哪些属性会发生变化
3. 避免在过渡过程中修改布局属性（如 width, height, margin 等）
4. 对于大型列表过渡，考虑使用 Vue 的虚拟滚动库（如 vue-virtual-scroller）
5. 对于复杂动画，考虑使用 Web Animation API 替代 CSS 动画