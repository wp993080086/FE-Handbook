# 📚 目录

1. [mixins混入介绍](#1-mixins混入介绍)
2. [为什么使用mixins](#2-为什么使用mixins)
3. [局部混入用法](#3-局部混入用法)
4. [全局混入用法](#4-全局混入用法)
4. [生命周期顺序](#4-生命周期顺序)
---

# 1. mixins混入介绍

mixins 选项接收一个混入对象的数组。一个混入对象可以包含任意的组件选项（vue里script的部分），这些选项将会被合并到最终的选项中，使用的是和 Vue.extend() 一样的选项合并逻辑。也就是说，如果你的混入包含一个 created 钩子，而创建组件本身也有一个，那么两个函数都会被调用。混入对象也可以包含data、created、mounted、methods、filters等等，它们将会被合并到组件自己的选项上。

# 2. 为什么使用mixins

可以理解为一个扩展包，引入之后，可以给组件扩展一些功能，比如组件的初始化，组件的销毁，组件的监听，组件的过滤等，可以减少组件的代码量，提高代码的复用率。相当于在引入后，父组件的各种属性方法都被扩充。

> 注意：minxins里面同名的方法和属性，会被父组件覆盖。

# 3. 局部混入用法

先定义一个minxins.js文件，里面导出一个包含vue option选项的对象，然后在需要的页面里面使用mixins选项引入即可。

- minxins.js

```javascript
export const commonMixins = {
  data() {
    return {
      name: '张三'
    }
  },
  mounted() {
    this.name = '李四'
    this.consolelog()
  },
  methods: {
    consolelog() {
      console.log(this.name)
    }
  }
}
```

- home.vue

```javascript
<template>
  <div class="home_box">
    <p>{{ this.name }}</p>
  </div>
</template>

<script>
import { commonMixins } from '@/utils/mixins.js'

export default {
  name: 'Home',
  mixins: [commonMixins],
  data() {
    return {
      keyword: '',
    }
  }
}
</script>
```

# 4. 全局混入用法

先先定义一个minxins.js文件，里面导出一个包含vue option选项的对象。然后在main.js中使用，其余页面就不用再引入了。

- main.js

```javascript
import commonMixins from '文件路径/mixin';
Vue.mixin(commonMixins)

const app = new Vue({
  render: h => h(App),
  router,
  store
})

app.$mount('#app')
```

# 5. 生命周期顺序

混入之后，生命周期执行顺序如下：

```javascript
mixin的beforeCreate 
↓
父beforeCreate
↓
mixin的created
↓
父created
↓
mixin的beforeMount
↓
父beforeMount
↓
子beforeCreate
↓
子created
↓
子beforeMount
↓
子mounted
↓
mixin的mounted
↓
父mounted
```