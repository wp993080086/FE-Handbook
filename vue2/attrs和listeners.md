# 📚 目录

1. [attrs](#1-attrs)
2. [listeners](#2-listeners)
3. [用处](#3-用处)

---

# 1. attrs

包含了父作用域中不作为 prop 被识别 (且获取) 的 attribute 绑定 (class 和 style 除外)。当一个组件没有声明任何 prop 时，这里会包含所有父作用域的绑定 (class 和 style 除外)，并且可以通过 v-bind="$attrs" 传入内部组件。也就是说，再创建子组件的时候，有的属性绑定没有写在 props 里，则可以通过 attrs 传递过去。

如下例子：父组件给子组件绑定了 percentage 属性等于 70，子组件通过 v-bind="$attrs"，把 percentage 属性直接透传给了 el-progress 组件，这在做一些多级组件嵌套（例如 A 嵌套 B ， B 嵌套 C ）封装时很方便。

- 父组件：

```javascript
<template>
  <div>
    <son-component :percentage="70" />
  </div>
</template>

<script>
import sonComponent from '@/views/attrs/son.vue'

export default {
  name: 'Attr',
  components: {
    sonComponent
  }
}
</script>
```

- 子组件：

```javascript
<template>
  <div>
    <el-progress v-bind="$attrs"></el-progress>
  </div>
</template>

<script>
export default {
  name: 'Son'
}
</script>
```

# 2. listeners

包含了父作用域中的 (不含 .native 修饰器的) v-on 事件监听器。它可以通过 v-on="$listeners" 传入内部组件——在创建更高层次的组件时非常有用。$listeners 相当于一个中间容器。可以用于解决组件嵌套，孙组件传递数据给 祖父组件。当出现多级组件嵌套时（例如 A 嵌套 B ， B 嵌套 C ），C 想传递数据 A，就需要在 B 中给 C 设置 v-on=“$listeners”，然后子组件C就可以通过$listeners.xxx 来传递数据给 A 了。

- A 组件：

```javascript
<template>
    <B @getData="getData"></B>
</template>
<script>
export default {
    methods: {
       getData(data){
         //获取c 传递的数据
      }
    }
}
</script>
```

- B 组件：

```javascript
<template>
	<C v-on="$listeners"></C>
</template>
```

- C 组件：

```javascript
<template>
    <button  @click="postData">传递数据</button>
</template>
<script>
export default {
    methods: {
       postData(){
         this.$emit('getData','传递数据给A')
      }
    }
}
```

# 3. 用处

- attrs

1. 父组件传递给子组件的属性，在子组件中，可以通过 $attrs 获取到，而不用一个个的定义 props。
2. 所有非 props 属性绑定到相应标签，如果是组件，则绑定到组件标签上。

- listeners

1. 父组件传递给子组件的事件，在子组件中，可以通过 $listeners 获取到，而不用一个个的定义 props。
2. 所有非 props 事件绑定到相应标签，如果是组件，则绑定到组件标签上。

二次封装组件例子：

```javascript
<template>
    <el-checkbox v-on="$listeners" v-bind="$attrs"  @click="myClick">
        <slot></slot>
    </el-checkbox>
</template>
<script>
```
