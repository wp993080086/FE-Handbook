# 📚 目录

1. [ref是什么](#1-ref是什么)
2. [ref的使用](#2-ref的使用)
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