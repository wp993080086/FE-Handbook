# 📚 目录1
1. [生命周期描述](#1-生命周期描述)
    1. [beforeCreate](#1-1-beforecreate)
    2. [created](#1-2-created)
    3. [beforeMount](#1-3-beforemount)
    4. [mounted](#1-4-mounted)
    5. [beforeUpdate](#1-5-beforeupdate)
    6. [updated](#1-6-updated)
    7. [activated](#1-7-activated)
    8. [deactivated](#1-8-deactivated)
    9. [beforeDestroy](#1-9-beforedestroy)
    10. [destroyed](#1-10-destroyed)
    11. [errorCaptured](#1-11-errorcaptured)
2. [父子组件生命周期的关系](#2-父子组件生命周期的关系)
2. [mixin混入后组件的生命周期](#3-mixin混入后组件的生命周期)
---

# 1. 生命周期描述

vue每个组件都是独立的，每个组件都有一个属于它的生命周期，从一个组件创建、数据初始化、挂载、更新、销毁，这就是一个组件所谓的生命周期，每个生命周期都是一个函数。

## 1-1. beforeCreate

在实例初始化之后被调用。此时组件的选项对象还未创建，`el`和`data`并未初始化，因此无法访问`methods`， `data`，`computed`，`props`等方法和数据。

此周期操作：暂无

## 1-2. created

在实例创建完成后被立即调用。在这一步，实例已完成以下的配置：数据观测(`data observer`)，属性和方法的运算，`watch/event`事件回调。但是，挂载阶段还没开始，`$el`目前尚不可用。可以改变`data`的数据，使用`methods`的方法，`computed`等。

此周期操作：Axios请求、初始化数据等

## 1-3. beforeMount

在挂载开始之前被调用：相关的render函数(虚拟dom)首次被调用。实例已完成配置： 编译模板，把`data`里面的数据和模板生成`vNode`，完成了`el`和`data`初始化，注意此时还没有挂在`html`到页面上。

此周期操作：暂无

## 1-4. mounted

实例被挂载后调用，这时`el`被新创建的`vm.$el`替换了。`mounted`不会保证所有的子组件也都一起被挂载。如需等到整个视图都渲染完毕，需要在`mounted`内部使用`vm.$nextTick`

此周期操作：获取dom、Axios请求等

## 1-5. beforeUpdate

数据更新完成之前调用，发生在虚拟`DOM`打补丁之前。这里适合在更新之前访问现有的`DOM`，比如手动移除已添加的事件监听器。

此周期操作：获取dom等

## 1-6. updated

由于数据更改导致的虚拟`DOM`重新渲染和打补丁，在这之后会调用该钩子。可以执行依赖于DOM的操作，避免在此期间更改状态，因为这可能会导致更新无限循环，最好使用`computed`或`watch`，如需等到整个视图都重绘完毕，可以在`updated`里使用`vm.$nextTick`

此周期操作：获取dom等

## 1-7. activated

被`keep-alive`缓存的组件激活时调用

此周期操作：暂无

## 1-8. deactivated

被`keep-alive`缓存的组件停用时调用

此周期操作：暂无

## 1-9. beforeDestroy

实例销毁之前调用。在这一步，实例仍然完全可用。

操作：清除定时器。事件绑定等等

## 1-10. destroyed

实例销毁后调用。该钩子被调用后，对应`Vue`实例的所有指令都被解绑，所有的事件监听器被移除，所有的子实例也都被销毁。

此周期操作：清除定时器。事件绑定等等

## 1-11. errorCaptured

当捕获一个来自子孙组件的错误时被调用。此钩子会收到三个参数：错误对象、发生错误的组件实例以及一个包含错误来源信息的字符串。此钩子可以返回`false`以阻止该错误继续向上传播。

此周期操作：暂无

# 2. 父子组件生命周期的关系

在开发中，经常是组件里面又有组件，父子组件的生命周期是不一样的，开发的时候，经常会有父组件异步调用接口的数据，传递给子组件，如果不知道生命周期的顺序，会出现问题。下面是总结的父子组件的生命周期顺序：

- 加载渲染过程

```javascript
父beforeCreate
↓
父created
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
父mounted
```

- 数据更新过程

```javascript
父beforeUpdate
↓
子beforeUpdate
↓
子updated
↓
父updated
```

- 组件销毁过程

```javascript
父beforeDestroy
↓
子beforeDestroy
↓
子destroyed
↓
父destroyed
```

# 3. mixin混入后组件的生命周期

在开发中，有时候会用到mixin混入，下面是总结的混入后的生命周期顺序：


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