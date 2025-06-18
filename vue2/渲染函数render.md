# 📚 目录

1. [渲染函数Render](#1-渲染函数render)
2. [参数和语法](#2-参数和语法)
    1. [参数](#2-1-参数)
    2. [Option数据对象](#2-2-option数据对象)
    3. [指令变化](#2-3-指令变化)
        1. [v-if和else](#2-3-1-v-if和else)
        2. [v-model](#2-3-2-v-model)
        3. [事件按键修饰符](#2-3-3-事件按键修饰符)
        4. [插槽](#2-3-4-插槽)
3. [编译JSX](#3-编译jsx)
4. [典型应用场景](#4-典型应用场景)
    1. [高阶组件示例](#4-1-高阶组件示例)
    2. [复杂的动态ui示例](#4-2-复杂的动态ui示例)
5. [性能](#5-性能)
---

# 1. 渲染函数Render

Vue 推荐在绝大多数情况下使用模板来创建你的 HTML。然而在一些场景中，你真的需要 JavaScript 的完全编程的能力。这时你可以用[渲染函数 render](https://v2.cn.vuejs.org/v2/guide/render-function.html)，它比模板更接近编译器，直接生成 VNode 虚拟 DOM。

下面是一个对比例子，通过 level prop 来动态生成标题的组件

- template 写法

```html
<template>
	<h1 :class="[`title${level}`]" v-if="level === 1">
		<slot></slot>
	</h1>
	<h2 :class="[`title${level}`]" v-else-if="level === 2">
		<slot></slot>
	</h2>
	<h3 :class="[`title${level}`]" v-else-if="level === 3">
		<slot></slot>
	</h3>
	<h4 :class="[`title${level}`]" v-else-if="level === 4">
		<slot></slot>
	</h4>
	<h5 :class="[`title${level}`]" v-else-if="level === 5">
		<slot></slot>
	</h5>
	<h6 :class="[`title${level}`]" v-else-if="level === 6">
		<slot></slot>
	</h6>
</template>

<script>
	export default {
		name: 'Title',
		props: {
			level: {
				type: Number,
				required: true,
			},
		},
	}
</script>
```

- render 写法

```javascript
export default {
	name: 'Title',
	props: {
		level: {
			type: Number,
			required: true,
		},
	},
	render(createElement) {
		const children = this.$slots.default || [];
		return createElement(`h${this.level}`, { class: [`title${this.level}`] }, children)
	},
}
```

# 2. 参数和语法

当使用 render 函数描述虚拟 DOM 时，vue 提供一个构建虚拟 DOM 的函数，叫 createElement，约定的简写为 h。

## 2-1. 参数

createElement 函数有三个参数：

1. 必填。一个 HTML 标签名、组件名，类型：{String | Object | Function}（也可以是组件选项对象或返回组件选项的函数）
2. 可选。一个与模板中属性对应的数据对象，也就是与模板中属性对应的数据对象，包含组件属性、DOM 属性、事件等。
3. 可选。子级虚拟节点 (VNodes)，由 `createElement()` 构建而成，也可以使用字符串来生成"文本虚拟"节点。

**语法：**

```javascript
createElement(TagName，Option，Content)
```

**第一个参数也可以是组件选项对象或返回组件选项的函数，例如：**

```javascript
// ：
createElement({
  template: '<div>{{ msg }}</div>',
  data() {
    return { msg: 'Hello' };
  }
});
```

## 2-2. Option数据对象

createElement 函数的第二个参数，是一个与模板中属性对应的数据对象，也就是组件的属性。

```javascript
{
  // 与 `v-bind:class` 的 API 相同，接受一个字符串、对象或字符串和对象组成的数组
  'class': {
    foo: true,
    bar: false
  },
  // 与 `v-bind:style` 的 API 相同，接受一个字符串、对象，或对象组成的数组
  style: {
    color: 'red',
    fontSize: '14px'
  },
  // 普通的 HTML attribute
  attrs: {
    id: 'foo'
  },
  // 组件 prop
  props: {
    myProp: 'bar'
  },
  // DOM property
  domProps: {
    innerHTML: 'baz'
  },
  // 事件监听器在 `on` 内，但不再支持如 `v-on:keyup.enter` 这样的修饰器。需要在处理函数中手动检查 keyCode。
  on: {
    click: this.clickHandler
  },
  // 仅用于组件，用于监听原生事件，而不是组件内部使用`vm.$emit` 触发的事件。
  nativeOn: {
    click: this.nativeClickHandler
  },
  // 自定义指令。注意，你无法对 `binding` 中的 `oldValue`赋值，因为 Vue 已经自动为你进行了同步。
  directives: [
    {
      name: 'my-custom-directive',
      value: '2',
      expression: '1 + 1',
      arg: 'foo',
      modifiers: {
        bar: true
      }
    }
  ],
  // 作用域插槽的格式为{ name: props => VNode | Array<VNode> }
  scopedSlots: {
    default: props => createElement('span', props.text)
  },
  // 如果组件是其它组件的子组件，需为插槽指定名称
  slot: 'name-of-slot',
  // 其它特殊顶层 property
  key: 'myKey',
  ref: 'myRef',
  // 如果你在渲染函数中给多个元素都应用了相同的 ref 名，那么 `$refs.myRef` 会变成一个数组。
  refInFor: true
}
```

下面是一个 Button 按钮的例子：

```javascript
export default {
	name: 'Button',
	props: {
		text: {
			type: String,
			required: true,
		},
	},
	methods: {
    handleClick() {
      console.log('按钮被点击了！');
    }
  },
	render(h) {
		return h(
			'div',
			{
				class: 'button-wrapper',
				on: {
					click: handleClick,
				},
			},
			[h('span', { class: 'button-text' }, this.text)]
		)
	},
}
```

## 2-3. 指令变化

指令的写法发生了变化，常用的 v-if/else，还有 v-for，v-model，事件修饰符等都有变化。

### 2-3-1. v-if和else

- 模板写法：

```javascript
<ul v-if="items.length">
  <li v-for="item in items" :key="item">{{ item }}</li>
</ul>
<p v-else>空空如也</p>
```

- render 函数写法：

```javascript
export default {
	// 省略......
	render(h) {
		if (this.items.length) {
			return h('ul', this.items.map((item) => h('li', { key: item }, item)));
		} else {
			return h('p', '空空如也');
		}
	}
}
```

### 2-3-2. v-model

render 函数中没有与 v-model 的直接对应，需要自己实现相应的逻辑。

- 模板写法

```javascript
<input v-model="message" placeholder="请输入" />
```

- render 函数写法

```javascript
export default {
	// 省略......
	props: ['message'],
	render(h) {
		const self = this
		return h('input', {
			domProps: {
				value: self.message,
			},
			on: {
				input: (event) => {
					// 组件绑定使用了sync语法糖
					self.$emit('update:message', event.target.value)
				},
			},
		})
	},
}
```

### 2-3-3. 事件按键修饰符

对于事件修饰符，vue 官方提供了部分的特殊前缀来处理，其余的，则需要自己在函数中处理。

| 修饰符        | 前缀 | 说明                                                       |
| ------------- | ---- | ---------------------------------------------------------- |
| .passive      | &    | 滚动事件的默认行为将会立即触发，而不是等到事件触发完再触发 |
| .capture      | !    | 捕获模式                                                   |
| .once         | ~    | 只触发一次回调                                             |
| .capture.once | ~!   | 只触发一次回调 的 捕获模式                                                   |

例子如下：

```javascript
on: {
  '!click': this.func,
  '~keyup': this.func,
  '~!mouseover': this.func,
  keyup: (event) => {
    // 如果触发事件的元素不是事件绑定的元素
    if (event.target !== event.currentTarget) return
    // 如果按下去的不是 enter 键或者没有同时按下 shift 键
    f (!event.shiftKey || event.keyCode !== 13) return
    // 阻止 事件冒泡
    event.stopPropagation()
    // 阻止该元素默认的 keyup 事件
    event.preventDefault()
  }
}
```

| 修饰符   | 操作                                              | 说明                   |
| -------- | ------------------------------------------------ | ---------------------- |
| .stop    | event.stopPropagation()                          | 阻止冒泡               |
| .prevent | event.preventDefault()                           | 阻止元素发生默认的行为 |
| .self    | if (event.target !== event.currentTarget) return | 自身触发               |
| .enter   | if (event.keyCode !== 13) return                 | 按键匹配               |
| .shift   | if (!event.shiftKey) return              				| 按键匹配               |

### 2-3-4. 插槽

可以通过 this.$slots 访问静态插槽的内容，每个插槽都是一个 VNode 数组：

```javascript
render: function (h) {
  return h('div', this.$slots.default)
}
```

也可以通过 this.$scopedSlots 访问作用域插槽，每个作用域插槽都是一个返回若干 VNode 的函数：

```javascript
props: ['message'],
render: function (h) {
  return h('div', [
    this.$scopedSlots.default({
      text: this.message
    })
  ])
}
```

如果要用渲染函数向子组件中传递作用域插槽，可以利用 VNode 数据对象中的 scopedSlots 字段：

```javascript
render: function (h) {
  return h('div', [
    h('child', {
      // 在数据对象中传递 `scopedSlots` 格式为 { name: props => VNode | Array<VNode> }
      scopedSlots: {
        default: function (props) {
          return h('span', props.text)
        }
      }
    })
  ])
}
```

例子如下：

- list.js

```javascript
const handleClick = (index) => {
	return () => {
		console.log(`${index}按钮被点击了！`)
	}
}

export default {
	name: 'List',
	props: {
		data: {
			type: Array,
			required: true,
			default: () => [],
		},
	},
	render(h) {
		if (this.data.length > 0) {
			return h(
				'ul',
				{
					class: 'ul-box',
				},
				this.data.map((item, i) => {
					return h(
						'li',
						{
							class: 'li-box',
							on: {
								click: handleClick(i),
							},
						},
						this.$scopedSlots.default({
							data: `${item}+${i}`,
						})
					)
				})
			)
		}
		return h('div', '暂无数据')
	},
}
```

- home.vue

```javascript
List<template>
  <div class="home_box">
    <List :data="list">
      <template slot-scope="scope">
        <span>{{ scope.data }}</span>
      </template>
    </List>
  </div>
</template>

<script>
import List from '@/views/render/list.js'

export default {
  name: 'Home',
  components: { List },
  data() {
    return {
      list: [100, 200, 300, 400, 500]
    }
  }
}
</script>
```

# 3. 编译jsx

如果你写了很多 render 函数，可能会觉得这样的代码写起来很痛苦，并且可读性也不好。这时候可以使用 JSX 插件：[传送门](https://github.com/vuejs/jsx-vue2)

- 使用JSX的render函数示例:

```javascript
render() {
  return (
    <div class={`title${this.level}`}>
      {this.$slots.default}
    </div>
  );
}
```

# 4. 典型应用场景

render 函数非常适合实现高阶组件以及复杂的动态 UI，因为它可以动态创建和组合组件。

## 4-1 高阶组件示例

```javascript
// 动态加载组件
export default function asyncComponentLoader(componentName) {
  return {
    name: `AsyncLoader(${componentName})`,
    data() {
      return {
        Component: null,
        loading: true,
        error: null,
      };
    },
    async created() {
      try {
        const componentModule = await import(`@/components/${componentName}.vue`);
        this.Component = componentModule.default || componentModule;
      } catch (err) {
        this.error = err.message;
      } finally {
        this.loading = false;
      }
    },
    render(h) {
      if (this.loading) return h('div', 'Loading...');
      if (this.error) return h('div', { style: { color: 'red' } }, this.error);
      if (this.Component) return h(this.Component, { props: this.$props });
      return h('div', 'Component not found');
    },
  };
}
```

## 4-2 复杂的动态ui示例

```javascript
// 动态表单生成器
export default {
  name: 'DynamicForm',
  props: {
    formConfig: {
      type: Array,
      required: true,
    },
    formData: {
      type: Object,
      default: () => ({}),
    },
  },
  methods: {
    handleInput(field, event) {
      this.$emit('input', { ...this.formData, [field]: event.target.value });
    },
  },
  render(h) {
    const formItems = this.formConfig.map((field) => {
      switch (field.type) {
        case 'text':
          return h('div', [
            h('label', field.label),
            h('input', {
              attrs: { type: 'text', placeholder: field.placeholder },
              domProps: { value: this.formData[field.name] || '' },
              on: { input: (e) => this.handleInput(field.name, e) },
            }),
          ]);
        case 'select':
          return h('div', [
            h('label', field.label),
            h('select', {
              domProps: { value: this.formData[field.name] || '' },
              on: { input: (e) => this.handleInput(field.name, e) },
            }, field.options.map(option => 
              h('option', { attrs: { value: option.value } }, option.label)
            )),
          ]);
        case 'checkbox':
          return h('div', [
            h('label', [
              h('input', {
                attrs: { type: 'checkbox' },
                domProps: { checked: this.formData[field.name] || false },
                on: { input: (e) => this.handleInput(field.name, e) },
              }),
              field.label,
            ]),
          ]);
        default:
          return null;
      }
    });

    return h('form', {
      on: {
        submit: (e) => {
          e.preventDefault();
          this.$emit('submit', this.formData);
        },
      }
    }, [
      ...formItems,
      h('button', { attrs: { type: 'submit' } }, '提交')
    ]);
  },
};

// 使用示例
<DynamicForm 
  :form-config="formConfig" 
  :form-data="formData" 
  @input="formData = $event" 
  @submit="handleSubmit" 
/>

// 配置示例
formConfig: [
  { type: 'text', name: 'username', label: '用户名', placeholder: '请输入用户名' },
  { type: 'text', name: 'email', label: '邮箱', placeholder: '请输入邮箱' },
  { type: 'select', name: 'role', label: '角色', options: [
    { value: 'admin', label: '管理员' },
    { value: 'user', label: '普通用户' },
  ]},
  { type: 'checkbox', name: 'agreement', label: '我同意条款' },
],
```

# 5. 性能

render 函数通常比 template 性能更高，原因如下：

1. 避免了模板编译过程
2. 可以更精确地控制虚拟 DOM 的创建
3. 在复杂动态 UI 场景中减少不必要的重新渲染

但请注意，不要为了性能而过度使用 render 函数，保持代码可读性同样重要。在大多数情况下，template 的性能已经足够好，没必要为了什么原因全部使用render。