# 📚 目录

1. [计算属性 computed](#1-计算属性computed)
2. [仅读取](#2-仅读取)
3. [读取和设置](#3-读取和设置)
4. [参数传递](#4-参数传递)
5. [使用场景](#5-使用场景)
6. [computed 和 methods 的区别](#6-computed和methods的区别)
7. [computed 和 watch 的区别](#7-computed和watch的区别)
---

# 1. 计算属性 computed

计算属性必须返回一个值，值的结果会被缓存，除非依赖的响应式 property 变化才会重新计算，如果某个依赖在该实例范畴之外，则计算属性是不会被更新的：[例子](#2-仅读取)。

# 2. 仅读取

如下例子：handleDouble 依赖了 total，只有 total 更新后，handleDouble 才会重新计算，并返回一个新值。

```vue
<template>
  <div class="home_box">
    <h1>{{ handleDouble }}</h1>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'

const total = ref(10)

const handleDouble = computed(() => {
  return total.value * 2
})
</script>
```

# 3. get 读取和 set 设置

如下例子：在setup中，我给 handleDouble 赋值了数字 6，在 template 模板中，最终显示的却是数字 14，这是因为 handleDouble 的 set 方法，会触发 total 的 set 方法，从而触发 handleDouble 的 get 方法，重新计算 handleDouble 的值。

```vue
<template>
  <div class="home_box">
    <h1>{{ handleDouble }}</h1>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'

const total = ref(10)

handleDouble.value = 6

const handleDouble = computed({
  get() {
    return total.value * 2
  },
  set(value) {
    total.value = value + 1
  }
})
</script>
```

# 4. 传递参数

如下例子：computed 也是可以传递参数的，采用闭包的方式，返回一个函数，该函数接收的参数就是你传递的参数。

```vue
<template>
  <div class="home_box">
    <h1>{{ checkcCloudless(false) }}</h1>
  </div>
</template>

<script>
import { ref, computed } from 'vue'

const checkcCloudless = computed(() => {
  return (value) => {
    return value ? '晴天' : '阴天'
  }
})
</script>
```

# 5. 使用场景

计算属性使用场景很多。比如：

1. 后端返回了一个状态，值为 false/true 或 0/1，需要你转成文字是和否
2. 数字/日期需要格式化
3. v-if、v-show 的判断条件复杂时

如下例子，根据 result 的值，显示不同的文字。

```javascript
<template>
  <div class="home_box">
    <h1>{{ checkcCloudless }}</h1>
    <button @click="switchCloudless">切换天气</button>
  </div>
</template>

<script>
import { ref, computed } from 'vue'

const result = ref(false)

const checkcCloudless = computed(() => {
  return result.value ? '晴天' : '阴天'
})

const switchCloudless = () => {
  result.value = !result.value
}
</script>
```

# 6. computed 和 methods 的区别

- computed：注重结果
  - 主要用来逻辑运算，防止模板过重
  - 有缓存
  - 监听，有 get 和 set 两个方法，get 必须 return
- methods：注重过程
  - 主要用做事件的处理函数，return 不是必须的
  - 没有缓存

# 7. computed 和 watch 的区别

- computed：
  - 主要用来逻辑运算，防止模板过重
  - 依赖其它数据
  - 有缓存，是基于它们的响应式依赖进行缓存的
  - 只会在其依赖项发生改变时才会重新计算
  - 监听，有 get 和 set 两个方法，get 必须 return
- watch：
  - 主要用于监听数据的变化，并执行函数
  - 监听的函数接收两个参数，第一个参数是最新的值，第二个参数是变化之前的值
  - 监听的函数执行次数取决于监听的属性值变化多少次
  - 监听的函数是立即执行的，监听的函数只执行一次
  - 监听的函数中，可以配置 2 个属性
    - immediate：是否立即执行
    - deep：是否深度监听
