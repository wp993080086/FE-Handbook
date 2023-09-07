# 📚 目录

1. [transition过渡组件](#1-transition过渡组件)
2. [属性](#2-属性)
3. [事件](#3-事件)
4. [原理](#4-原理)
5. [使用场景](#5-使用场景)
---

# 1. transition过渡组件

<transition> 组件用于在元素进入或离开 DOM 时应用过渡效果。

# 2. 属性

属性名 | 类型 | 默认值| 可选值 | 说明
---|---|---|---|---
name |string |'v' | - |用于自动生成 CSS 过渡类名前缀，例如：name: 'fade' 将自动拓展为 .fade-enter，.fade-enter-active 等 |
appear | boolean |false | -| 是否在初始渲染时使用过渡 |
css | boolean |true | -|是否使用 CSS 过渡类，如果设置为 false，将只通过组件事件触发注册的 JavaScript 钩子 |
type | string |- |'transition' 'animation' |指定过渡事件类型 |
mode | boolean |true |'out-in' 'in-out' |控制离开/进入过渡的时间序列 |
duration | number |- |{ enter: number, leave: number } |指定过渡的持续时间 |
enter-class | string | -| -|类名 |
leave-class | string | -| -|类名 |
appear-class | string | |- |类名 |
enter-to-class | string |- |- |类名 |
leave-to-class | string |- |- |类名 |
appear-to-class | string | |- |类名 |
enter-active-class | string | -|- |类名 |
leave-active-class | string |- |- |类名 |
appear-active-class | string |- |- | 类名 |

# 3. 事件

事件名 | 说明 |
---|---|
 before-enter | 进入之前 |
 before-leave | 离开之前 |
 before-appear | 出现之前 |
 enter | 进入 |
 leave | 离开 |
 appear | 出现 |
 after-enter | 进入之后 |
 after-leave | 离开之后 |
 after-appear | 出现之后 |
 enter-cancelled | 卸载进入 |
 leave-cancelled | 卸载离开，只有v-show有 |
 appear-cancelled | 卸载出现 |

# 4. 原理

总结起来就是，通过添加和移除类名来触发 CSS的transition 或 animation ，来达到过渡效果。步骤如下：

1. 挂载阶段：当包裹在 <transition> 内的元素首次渲染到 DOM 时，它会立即显示，没有过渡效果。

2. DOM 更新：当触发了元素的显示或隐藏操作，比如修改了 v-if、v-show、或者条件渲染的数据，Vue 会执行一次重新渲染。

3. 过渡类名添加：在重新渲染过程中，Vue 会根据过渡效果的配置为元素添加一系列类名，这些类名用于触发 CSS 过渡效果，比如v-enter，v-enter-active，v-enter-to，v-leave等等

4. 过渡执行：通过添加上述的类名，触发了 CSS 过渡效果。

5. 过渡结束：一旦过渡效果执行完毕，Vue 会自动移除相应的过渡类名，使元素回到普通状态。

6. 钩子函数：如果需要在过渡过程中执行一些 JavaScript 逻辑，可以使用 @enter、@leave 等事件钩子函数。

# 5. 使用场景

1. 列表、按钮，文字，各种过渡效果
2. 页面切换动画等

- transition显示变化

```javascript
<template>
  <div class="home_box">
    <Transition name="fade" mode="out-in">
      <p v-show="visible">{{ name }}</p>
    </Transition>
    <button @click="handleVisible">显示隐藏</button>
  </div>
</template>

<script>
export default {
  name: 'Home',
  data() {
    return {
      name: 'abc',
      visible: true
    }
  },
  methods: {
    handleVisible() {
      this.visible = !this.visible
    }
  }
}
</script>
```

- transition-group的列表变化

```javascript
<template>
  <div class="home_box">
    <button v-on:click="handleVisible">倒序</button>
    <transition-group name="flip-list" tag="ul">
      <li v-for="item in items" v-bind:key="item">
        {{ item }}
      </li>
    </transition-group>
  </div>
</template>

<script>
export default {
  name: 'Home',
  data() {
    return {
      name: 'abc',
      visible: true,
      items: [1, 2, 3, 4, 5, 6, 7, 8, 9]
    }
  },
  methods: {
    handleVisible() {
      this.items = this.items.reverse()
    }
  }
}
</script>

<style scoped lang="scss">
.flip-list-move {
  transition: transform 1s;
}
</style>

```