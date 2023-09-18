# 📚 目录
1. [实例配置](#1-实例配置)
2. [获取实例或组件](#2-获取实例或组件)
2. [设置属性或方法](#3-设置属性或方法)
---

# 1. 实例配置

1. 取消Vue所有日志与警告

```javascript
Vue.config.silent = true
```

2. 配置是否允许 vue-devtools 检查代码，开发版本默认为 true，生产版本默认为 false。

```javascript
Vue.config.devtools = true
```

3. vue在启动时生成生产提示

```javascript
Vue.config.productionTip = false
```

# 2. 获取实例或组件

1. 获取当前组件树的根 Vue 实例。如果当前实例没有父实例，此实例将会是其自己。

```javascript
this.$root
```

2. 获取当前实例的直接子组件

```javascript
this.$children
```

3. 获取当前组件的父实例

```javascript
this.$parent
```

4. 获取注册过ref属性的组件

```javascript
this.$refs
```

5. 获取当前Vue实例的初始化选项

```javascript
this.$options
```

6. 获取当前组件接收到的props对象

```javascript
this.$props
```

7. 获取父组件不作为prop被识别的属性

```javascript
this.$attrs
```

8. 获取父组件的v-on

```javascript
this.$listeners
```

9. 获取组件的插槽

```javascript
this.$slots
```

10. 获取组件的作用域插槽

```javascript
this.$scopedSlots
```

# 3. 设置属性或方法

1. $set

向响应式对象中添加一个属性，并触发视图更新，参数为：要改变的对象或者数组，要添加的属性名或者索引，要添加的值。

语法：

```javascript
this.$set( target, propertyName / index, value )
```

例子：

```javascript
<script>
export default {
  name: 'Home',
  data() {
    return {
      userObj: {
        name: '张三',
        age: 18
      },
      userList: [
        { name: '张三', age: 18, gender: '男' },
        { name: '李四', age: 19, gender: '女' }
      ]
    }
  },
  methods: {
    setGender() {
      this.$set(this.userObj, 'gender', '男')
      this.$set(this.userList, 1, { name: '王五', age: 20, gender: '男' })
    }
  }
}
</script>
```

2. $delete

向响应式对象中删除一个属性，并触发视图更新，参数为：要删除的对象或者数组，要删除的属性名或者索引。

语法：
```javascript
this.$delete( target, propertyName / index )
```

例子：

```javascript
export default {
  name: 'Home',
  data() {
    return {
      userObj: {
        name: '张三',
        age: 18
      },
      userList: [
        { name: '张三', age: 18, gender: '男' },
        { name: '李四', age: 19, gender: '女' }
      ]
    }
  },
  methods: {
    setGender() {
      this.$delete(this.userObj, 'age')
      this.$delete(this.userList, 1)
    }
  }
}
</script>
```