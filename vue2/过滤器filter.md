# 📚 目录

1. [过滤器filter](#1-过滤器filter)
2. [局部过滤器](#2-局部过滤器)
3. [全局过滤器](#3-全局过滤器)
4. [传递参数](#4-传递参数)
5. [过滤器串联](#5-过滤器串联)
6. [filter和computed的区别](#6-filter和computed的区别)
---

# 1. 过滤器filter

vue中的过滤器，主要用于文本的格式化，或者数组数据的过滤与排序等。其本质是一个函数，它不改变原始数据，只是对数据进行加工处理后返回过滤后的数据。过滤器可以用在两个地方：双花括号插值和v-bind表达式，使用时通过管道符 | 添加到表达式的尾部使用。语法如下：

- 花括号
```javascript
<div>{{ Id | formatId }}</div>
```

- v-bind

```javascript
<div v-bind:id="Id | formatId"></div>
```

# 2. 局部过滤器

如下例子：通过filters选项，定义了一个叫filterName的过滤器，它默认管道符前面的值作为第一个参数，filterName会返回一个格式化后的字符串。

```javascript
<template>
  <div class="home_box">
    <p>{{ this.name | filterName }}</p>
  </div>
</template>

<script>
export default {
  name: 'Home',
  filters: {
    filterName(value) {
      return `${value}-过滤器`
    }
  },
  data() {
    return {
      name: '王五'
    }
  }
}
</script>
```

# 3. 全局过滤器

首先在src目录下新建一个filters.js文件，用来装所有的filter。然后在main.js中导入，挂载到vue实例上，即可在所有组件中使用。

- filters.js

```javascript
/**
 * @description 首字母大写
 */
export const capitalize = (value) => {
  if (!value) return ''
  value = value.toString()
  return value.charAt(0).toUpperCase() + value.slice(1)
}
/**
 * @description 双倍
 */
export const double = (value) => {
  return value * 2
}
```

- main.js

```javascript
import Vue from 'vue'
import ElementUI from 'element-ui'
import App from './App.vue'
import * as filterEnum from '@/utils/filters.js'

Vue.use(ElementUI)

Object.keys(filterEnum).forEach(key => {
  Vue.filter(key, filterEnum[key])
})

const app = new Vue({
  render: h => h(App)
})

app.$mount('#app')

```

# 4. 传递参数

filter也是可以传递参数的，如下例子，过滤器filterFn接收了三个参数。其中 message 的值作为第一个参数，arg1 作为第二个参数，arg2 作为第三个参数。

```javascript
<div>{{ message | filterFn('arg1', 'arg2') }}</div>
```

# 5. 过滤器串联

过滤器可以串联，如下例子，filterA 被定义为接收单个参数的过滤器函数，表达式 message 的值将作为参数传入到函数中。然后继续调用同样被定义为接收单个参数的过滤器函数 filterB，将 filterA 的结果传递到 filterB。

```javascript
<div>{{ message | filterA | filterB }}</div>
```

# 5. 使用场景

过滤器使用场景很多。比如：

1. 后端返回了一个状态，值为false/true或0/1，需要你转成文字是和否
2. 数字/日期需要格式化等

如下例子，根据result的值，显示不同的文字。

```javascript
<template>
  <div class="home_box">
    <h1>{{ result | checkcCloudless }}</h1>
  </div>
</template>

<script>
export default {
  name: 'Home',
  filters: {
    checkcCloudless(value) {
      return value ? '晴天' : '阴天'
    }
  },
  data() {
    return {
      result: false
    }
  }
}
</script>
```

# 6. filter和computed的区别

- computed：
    - 主要用来逻辑运算，防止模板过重
    - 有缓存 只有依赖的其他数据项变化它才变
    - 监听，有get和set两个方法，get必须return
    - 只有依赖的其他数据项变化它才变
    - 在模板外面直接this.使用
- filter：
    - 主要用做数据格式化的处理
    - 无法缓存 每次渲染都会执行
    - 定义方式的区别，filter可以通过filters和vue.filter定义
    - 在模板外面使用必须用this.$options.filters['filter名字']
