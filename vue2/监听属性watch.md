# 📚 目录

1. [监听属性 watch](#1-监听属性watch)
2. [常规用法](#2-常规用法)
3. [监听对象和route变化](#3-监听对象和route变化)
4. [使用场景](#4-使用场景)
---

# 1. 监听属性 watch

watch 是一个对象，键是需要观察的表达式，值是对应的回调函数。值也可以是方法名，或者包含选项的对象。Vue 实例将会在实例化时调用 $watch()，遍历 watch 对象的每一个 property。

# 2. 常规用法

watch 监听有两个形参，第一个是新值，第二个是旧值。如下例子：使用 watch 监听了 total 的值，当 total 的值改变时，控制台会打印出旧值和新值。

```javascript
<template>
  <div class="home_box">
    <h1>{{ total }}</h1>
    <button @click="handleAddTotal">增加</button>
  </div>
</template>

<script>
export default {
  name: 'Home',
  watch: {
    total(newValue, oldValue) {
      console.log('旧值：', oldValue)
      console.log('新值：', newValue)
    }
  },
  data() {
    return {
      total: 0
    }
  },
  methods: {
    handleAddTotal() {
      this.total++
    }
  }
}
</script>
```

# 3. 监听对象和route变化

watch监听的目标，可以是基本类型，也可以是对象，也可以是对象里的一个值。而监听目标的属性，可以是一个函数，也可以是一个包含handler（回调函数），immediate（是否初始化后立即执行一次）和deep（是否开启深度监听）的对象。

```javascript
<script>
export default {
  name: 'Home',
  watch: {
    // 监听基本类型
    aaa(newValue, oldValue) {
      console.log('旧值：', oldValue)
      console.log('新值：', newValue)
    },
    // 监听基本类型，并且回调函数写在methods里，且初始化加载立即执行一次
    bbb: {
      handler: 'handleBBB',
      immediate: true
    },
    // 监听对象类型，需要开启深度监听
    ccc: {
      handler: (newValue, oldValue) {
        console.log('旧值：', oldValue)
        console.log('新值：', newValue)
      },
      deep: true
    },
    // 监听对象里的某个值
    'ddd.d2.d21': {
      handler: (newValue, oldValue) {
        console.log('旧值：', oldValue)
        console.log('新值：', newValue)
      }
    },
    // 监听route变化
    '$route': {
      handler: (newValue, oldValue) {
        console.log('旧值：', oldValue)
        console.log('新值：', newValue)
      }
    }
  },
  data() {
    return {
      aaa: 0,
      bbb: 0,
      ccc: {
        c1: 0,
        c2: 0
      },
      ddd: {
        d1: 0,
        d2: {
          d21: 0
        }
      }
    }
  },
  methods: {
    handleBBB() {
      this.bbb++
    }
  }
}
</script>
```

# 4. 使用场景

watch监听属性使用场景很多。比如：

1. 即时表单验证
2. 搜索
3. 监听数据变化，做出相应改变
4. ......

如下例子，监听 keyword 的值，实时打印出来。

```javascript
<template>
  <div class="home_box">
    <input type="text" v-model="keyword">
  </div>
</template>

<script>
export default {
  name: 'Home',
  watch: {
    keyword: {
      handler: 'handleKeywordChange'
    }
  },
  data() {
    return {
      keyword: '',
    }
  },
  methods: {
    handleKeywordChange(newValue, oldValue) {
      console.log(newValue, oldValue)
    }
  }
}
</script>
```
