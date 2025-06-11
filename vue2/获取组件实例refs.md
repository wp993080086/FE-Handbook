# 📚 目录

1. [ref是什么](#1-ref是什么)
2. [ref的使用](#2-ref的使用)
3. [ref的特性](#3-ref的特性)
4. [使用场景](#4-使用场景)
5. [注意事项](#5-注意事项)
6. [与React的对比](#6-与React的对比)
7. [动态ref](#7-动态ref)
8. [函数式组件中的ref](#8-函数式组件中的ref)
9. [组合式API中的ref](#9-组合式API中的ref)
10. [总结](#10-总结)
---

# 1. ref是什么

ref 被用来给元素或子组件注册引用信息。引用信息将会注册在父组件的 $refs 对象上。可以通过实例对象获取到后进行一些操作。

- 如果在普通的 DOM 元素上使用，引用指向的就是 DOM 元素

```html
<div ref="divRef">hello</div>
```

- 如果用在子组件上，引用就指向组件实例

```html
<list-component ref="listRef"></list-component>
```

- 如果在v-for循环中使用了ref，引用指向的就是数组类型。

```html
<div v-for="item in list" :ref="item.xxx"></div>
```

# 2. ref的使用

在给组件绑定ref=xxx的属性后，通过this.$refs.xxx，就可以获取到组件的实例，进而可以操作组件的方法，获取到属性，如下例子，通过获取到list组件的实例，得到了组件内部的xxx属性。

- 普通ref

```javascript
<list-component ref="listRef"></list-component>

console.log(this.$refs.listRef.xxx)
```

- 循环里的ref

如果在v-for循环中使用了ref，引用指向的就是数组类型，为了确保唯一性，可以使用id拼接的方式。

```javascript
<div v-for="item in list" :ref="`ref${item.id}`"></div>

console.log(this.$refs[`ref${item.id}`])
```

# 3. ref的特性

- **非响应式**：`$refs` 对象不会触发视图更新，即便引用的元素或者组件有了变化。
- **延迟更新**：`$refs` 是在 DOM 更新结束之后才会进行更新的，所以在初始化阶段或者数据变更时，直接访问 `$refs` 可能无法获取到最新的引用。
- **生命周期钩子**：在 `mounted` 钩子函数里访问 `$refs` 是比较稳妥的，因为这时 DOM 已经完成了首次渲染。

# 4. 使用场景

- **DOM 操作**：

```javascript
  // 聚焦输入框
  this.$refs.inputRef.focus()
  
  // 滚动到特定元素
  this.$refs.scrollContainer.scrollTop = 100
  ```

- **调用子组件方法**：
```javascript
  // 调用子组件的刷新方法
  this.$refs.childComponent.refreshData()
```

- **访问子组件属性**：
```javascript
  // 获取子组件的内部状态
  const count = this.$refs.counterComponent.count
```

# 5. 注意事项

- **避免滥用**：应当优先考虑使用数据绑定或者事件机制来实现组件间的通信，只有在确实需要直接操作 DOM 或者组件实例时才使用 `ref`。
- **异步更新**：由于 Vue 的 DOM 更新是异步进行的，所以在修改数据之后立即访问 `$refs` 可能无法获取到更新后的 DOM。这种情况下，可以使用 `this.$nextTick()` 来确保 DOM 已经更新完毕。
  ```javascript
  this.list = [...newList]
  this.$nextTick(() => {
    // 此时可以获取到更新后的 ref 列表
    console.log(this.$refs.itemRefs)
  })
  ```

# 6. 与 React 的对比

在 React 中，`ref` 的使用方式和 Vue 既有相似之处，也存在差异：

- **创建方式**：
  ```javascript
  // Vue
  <div ref="myDiv"></div>
  
  // React
  <div ref={this.myRef}></div>
  ```

- **类型区别**：
  - 在 Vue 里，`ref` 可以直接指向 DOM 元素或者组件实例。
  - 在 React 中，`ref` 可以是一个回调函数，也可以是通过 `createRef()` 创建的对象。

- **访问方法**：
  ```javascript
  // Vue
  this.$refs.myDiv
  
  // React
  this.myRef.current
  ```

# 7. 动态 ref

借助动态绑定的方式，可以实现 `ref` 的动态管理：
```javascript
<component :ref="dynamicRefName"></component>

// 在不同条件下使用不同的 ref
this.dynamicRefName = condition ? 'refA' : 'refB'
```

# 8. 函数式组件中的 ref

函数式组件没有自己的实例，所以在使用 `ref` 时会指向其渲染的根节点：
```javascript
// 函数式组件
const FunctionalComp = {
  functional: true,
  render(h, context) {
    return <div>Functional Component</div>
  }
}

// 使用 ref 获取根 DOM
<FunctionalComp ref="funcRef" />
this.$refs.funcRef // 指向 <div> 元素
```

# 9. 组合式 API 中的 ref

在组合式 API 里，可以通过 `defineExpose` 来暴露组件的属性和方法：
```javascript
<template>
  <button ref="btnRef" @click="handleClick">Click</button>
</template>

<script setup>
import { ref, onMounted } from 'vue'

const btnRef = ref(null)

const handleClick = () => {
  console.log('Button clicked')
}

// 暴露给父组件
defineExpose({
  handleClick
})

onMounted(() => {
  btnRef.value.focus()
})
</script>
```

# 10. 总结

- `ref` 是 Vue 提供的一种直接操作 DOM 或者组件实例的方式。
- 要注意 `ref` 的非响应式特性以及延迟更新的特点。
- 建议在必要的场景下使用 `ref`，并优先采用声明式的数据绑定。
- 在组合式 API 中，可以使用 `defineExpose` 有选择性地暴露组件的方法和属性。