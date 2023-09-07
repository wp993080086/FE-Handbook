# 📚 目录

1. [nextTick的语法](#1-nextTick的语法)
2. [使用场景](#2-使用场景)
3. [原理](#3-原理)
---

# 1. nextTick的语法

官网说法是：在下次 DOM 更新循环结束之后执行延迟回调。在修改数据之后立即使用这个方法，可以获取更新后的 DOM。（如果没有提供回调且在支持 Promise 的环境中，则返回一个 Promise），语法如下：

```jsvascript
this.$nextTick(function () {
  // DOM 更新了
})

// 作为一个 Promise 使用
this.$nextTick().then(function () {
  // DOM 更新了
})
```

# 2. 使用场景

nextTick主要用于处理数据动态变化后，DOM还未及时更新的问题，用nextTick就可以获取数据更新后最新DOM的变化。

1. 数据改变后，获取改变之后的dom
2. created()生命周期里进行DOM的操作
3. 等等...

如下例子，使用nextTick获取到了v-if为true的dom节点，如果不使用nextTick的话，会获取到一个null。

```javascript
<template>
  <div class="home_box">
    <div v-if="visible" id="box"></div>
    <button @click="getDom">Get DOM</button>
  </div>
</template>

<script>
export default {
  name: 'Home',
  data() {
    return {
      visible: false
    }
  },
  methods: {
    getDom() {
      this.visible = true
      this.$nextTick(() => {
        const dom = document.querySelector('#box')
        console.log(dom)
      })
    }
  }
}
</script>
```

# 3. 原理

这个API是怎么实现的的呢，原理如下：

**JS 运⾏机制**

- JS 执⾏是单线程的， 它是基于事件循环的。 事件循环⼤致分为以下⼏个步骤
  - 1. 所有同步任务都在主线程上执⾏， 形成⼀个执⾏栈（execution context stack）。
  - 2. 主线程之外， 还存在⼀个"任务队列"（task queue），只要异步任务有了运⾏结果，就在"任务队列"之中放置⼀个事件。
  - 3. ⼀旦"执⾏栈"中的所有同步任务执⾏完毕，系统就会读取"任务队列"，看看⾥⾯有哪些事件，那些对应的异步任务，于是结束等待状态，任务队列的任务进⼊执⾏栈，开始执⾏。
  - 4. 主线程不断重复上⾯的第三步
  - 5. 同步的任务，按照顺序进行执行。异步的任务，分为宏任务（参与了事件循环的异步任务）和微任务（直接在Javascript引擎中的执行的，没有参与事件循环的异步任务）
  - 6. 浏览器环境中：当宏任务和当前宏任务产生的微任务全部执行完毕后，才会执行下一个宏任务。每遇到生成的微任务就放到微任务队列中，当前宏任务代码全部执行后开始执行微任务队列中的任务
  - 7. Node.js中：它会先执行所有的宏任务，然后再执行所有的微任务，最后再执行 I/O 操作的回调函数，所以I/O有更好的性能。

**nextTick**

1. 由于Vue是异步渲染Dom的，所以修改数据后，页面不会立即更新，而是设置一个回调函数放入callbacks队列等待执行，如果这个更新触发多次，只会被推送到队列一次。
2. 根据环境判断，尝试使用原生的Promise.then（微任务），MutationObserver（h5新增 监听dom结构改变的 微任务），setImmediate（浏览器兼容性较差 宏任务），setTimeout（宏任务）等等做降级处理，将执行函数放到微任务或者宏任务中。
3. 事件循环走到了微任务或者宏任务，执行函数依次执行callbacks中的回调