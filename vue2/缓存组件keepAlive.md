# 📚 目录

1. [keep-alive缓存组件](#1-keep-alive缓存组件)
2. [属性](#2-属性)
3. [原理](#3-原理)
4. [使用场景](#4-使用场景)
---

# 1. keep-alive缓存组件

keep-alive是Vue中内置的一个抽象组件，它自身不会渲染一个DOM元素，也不会出现在父组件链中。当它包裹组件时，会缓存不活动的组件实例，而不是销毁它们。主要用于保留组件状态或避免重新渲染。

> 当组件在 <keep-alive> 内被切换，它的 activated 和 deactivated 这两个生命周期钩子函数将会被对应执行

- 用于router-view标签

```javascript
<keep-alive>
  <router-view></router-view>
</keep-alive>
```

- 用于动态组件

```javascript
<keep-alive>
  <component :is="xxx"/>
</keep-alive>
```

# 2. 属性

属性名 | 类型 | 说明
---|---|---
include | string/正则表达式/数组 | 只有名称匹配的组件会被缓存
exclude | string/正则表达式/数组 | 任何名称匹配的组件都不会被缓存
max | number |最多可以缓存多少组件实例

如下例子，只会缓存aaa，bbb，ccc三个组件
```javascript
<keep-alive :include=['aaa', 'bbb', 'ccc']>
  <router-view></router-view>
</keep-alive>
```

# 3. 原理

1. vue的组件在内部是一个vnode的虚拟dom对象
2. 当一个包裹在 <keep-alive> 标签内的组件第一次被渲染时，Vue 会将这个组件的实例vnode缓存起来
3. 当被包裹在 <keep-alive> 中的组件被销毁时，该组件的vnode会被缓存，而不是销毁组件实例
4. 当要加入缓存时，将组件先删除再添加进去，比如用c时,先将c删除,再添加到d后边
5. Vue 会在内部维护一个缓存对象，将被缓存的组件实例以组件名为键进行存储。
6. 当这个组件再次需要被渲染时，Vue 会首先检查缓存中是否有该组件的实例。
    - 如果有，则直接将缓存中的实例返回，插入到目标元素中，并触发 activated 钩子
    - 如果没有，就会创建一个新的组件实例，并将其挂载到 DOM 上，触发正常的生命周期钩子 

# 使用场景

有些页面或组件需要缓存状态的时候，比如输入框，表单等，通过vuex，动态管理include属性即可。