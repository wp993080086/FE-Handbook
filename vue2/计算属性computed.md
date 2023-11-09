# 📚 目录

1. [计算属性computed](#1-计算属性computed)
2. [仅读取](#2-仅读取)
3. [get读取和set设置](#3-get读取和set设置)
4. [参数传递](#4-参数传递)
5. [使用场景](#5-使用场景)
6. [computed和methods的区别](#6-computed和methods的区别)
7. [computed和watch的区别](#7-computed和watch的区别)
---

# 1. 计算属性computed

计算属性必须返回一个值，值的结果会被缓存，除非依赖的响应式 property 变化才会重新计算，如果某个依赖在该实例范畴之外，则计算属性是不会被更新的：[例子](#2-仅读取)。

# 2. 仅读取

如下例子：handleDouble依赖了total，只有total更新后，handleDouble才会重新计算，并返回一个新值。

```javascript
<template>
  <div class="home_box">
    <h1>{{ handleDouble }}</h1>
  </div>
</template>

<script>
export default {
  name: 'Home',
  computed: {
    handleDouble() {
      return this.total * 2
    }
  },
  data() {
    return {
      total: 10
    }
  }
}
</script>
```

# 3. get读取和set设置

如下例子：在mounted生命周期中，我给handleDouble赋值了数字6，在template模板中，最终显示的却是数字14，这是因为handleDouble的set方法，会触发total的set方法，从而触发handleDouble的get方法，重新计算handleDouble的值。

```javascript
<template>
  <div class="home_box">
    <h1>{{ handleDouble }}</h1>
  </div>
</template>

<script>
export default {
  name: 'Home',
  computed: {
    handleDouble: {
      get() {
        return this.total * 2
      },
      set(value) {
        this.total = value + 1
      }
    }
  },
  data() {
    return {
      total: 10
    }
  },
  mounted() {
    this.handleDouble = 6
  }
}
</script>
```

# 4. 参数传递

如下例子：computed也是可以传递参数的，采用闭包的方式，返回一个函数，该函数接收的参数就是你传递的参数。

```javascript
<template>
  <div class="home_box">
    <h1>{{ checkcCloudless(false) }}</h1>
  </div>
</template>

<script>
export default {
  name: 'Home',
  computed: {
    checkcCloudless() {
      return (value) => {
        return value ? '晴天' : '阴天'
      }
    }
  }
}
</script>
```

# 5. 使用场景

计算属性使用场景很多。比如：

1. 后端返回了一个状态，值为false/true或0/1，需要你转成文字是和否
2. 数字/日期需要格式化
3. v-if、v-show的判断条件复杂时

如下例子，根据result的值，显示不同的文字。

```javascript
<template>
  <div class="home_box">
    <h1>{{ checkcCloudless }}</h1>
    <button @click="switchCloudless">切换天气</button>
  </div>
</template>

<script>
export default {
  name: 'Home',
  computed: {
    checkcCloudless() {
      return this.result ? '晴天' : '阴天'
    }
  },
  data() {
    return {
      result: false
    }
  },
  methods: {
    switchCloudless() {
      this.result = !this.result
    }
  }
}
</script>
```

# 6. computed和methods的区别

- computed：注重结果
    - 主要用来逻辑运算，防止模板过重
    - 有缓存
    - 监听，有get和set两个方法，get必须return
- methods：注重过程
    - 主要用做事件的处理函数，return不是必须的
    - 没有缓存

# 7. computed和watch的区别

- computed：
    - 主要用来逻辑运算，防止模板过重
    - 依赖其它数据
    - 有缓存，是基于它们的响应式依赖进行缓存的
    - 只会在其依赖项发生改变时才会重新计算
    - 监听，有get和set两个方法，get必须return
- watch：
    - 主要用于监听数据的变化，并执行函数
    - 监听的函数接收两个参数，第一个参数是最新的值，第二个参数是变化之前的值
    - 监听的函数执行次数取决于监听的属性值变化多少次
    - 监听的函数是立即执行的，监听的函数只执行一次
    - 监听的函数中，可以配置2个属性
        - immediate：是否立即执行
        - deep：是否深度监听