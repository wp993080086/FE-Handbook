# 📚 目录

1. [attrs](#1-attrs)
2. [listeners](#2-listeners)
3. [高级用法和注意事项](#3-高级用法和注意事项)
4. [应用场景](#4-应用场景)
---

在 Vue 开发中，$attrs和$listeners是两个非常实用的实例属性，它们为组件间的通信和组件封装提供了极大的便利。

# 1. attrs

`$attrs`包含了父作用域中不作为 prop 被识别 (且获取) 的 attribute 绑定 (class 和 style 除外)。当一个组件没有声明任何 prop 时，这里会包含所有父作用域的绑定 (class 和 style 除外)，并且可以通过 v-bind="$attrs" 传入内部组件。也就是说，再创建子组件的时候，有的属性绑定没有写在 props 里，则可以通过 attrs 传递过去。

**特性解析**

- 范围：所有未被组件 props 声明的属性
- 排除项：class 和 style 属性
- 传递方式：通过v-bind="$attrs"实现属性透传

如下例子：父组件给子组件绑定了 percentage 属性等于 70，子组件通过 v-bind="$attrs"，把 percentage 属性直接透传给了 el-progress 组件，这在做一些多级组件嵌套（例如 A 嵌套 B ， B 嵌套 C ）封装时很方便。

- 父组件：

```javascript
<template>
  <div>
    <h1>父组件</h1>
    <child-component 
      :percentage="70" 
      :color="'#409EFF'" 
      :stroke-width="10" 
      :text-inside="true"
    />
  </div>
</template>

<script>
import ChildComponent from './child.vue';

export default {
  name: 'Parent',
  components: {
    ChildComponent
  }
}
</script>
```

- 子组件：

```javascript
<template>
  <div>
    <h3>子组件</h3>
    <el-progress v-bind="$attrs"></el-progress>
  </div>
</template>

<script>
export default {
  name: 'Child',
  // 声明了一些props，其他未声明的属性会被包含在$attrs中
  props: {
    percentage: {
      type: Number,
      default: 0
    }
  },
  created() {
    // 打印$attrs中的内容
    console.log('$attrs:', this.$attrs);
    // 输出: { color: '#409EFF', stroke-width: 10, text-inside: true }
  }
}
</script>
```

在这个例子中，父组件向子组件传递了多个属性，但子组件只声明了percentage作为 prop，其他属性（color、stroke-width、text-inside）都会被包含在$attrs中，并通过v-bind="$attrs"传递给了el-progress组件。

# 2. listeners

`$listeners`包含了父作用域中的 (不含 .native 修饰器的) v-on 事件监听器。它可以通过 v-on="$listeners" 传入内部组件，在创建更高层次的组件时非常有用。

**特性解析**

- 范围：所有未被组件明确处理的事件监听器
- 排除项：使用.native 修饰器的事件
- 传递方式：通过v-on="$listeners"实现事件透传

$listeners 相当于一个中间容器。可以用于解决组件嵌套，孙组件传递数据给 祖父组件。当出现多级组件嵌套时（例如 A 嵌套 B ， B 嵌套 C ），C 想传递数据 A，就需要在 B 中给 C 设置 v-on=“$listeners”，然后子组件C就可以通过$listeners.xxx 来传递数据给 A 了。

- 祖父组件：

```javascript
<template>
  <div>
    <h1>祖父组件</h1>
    <p>接收到的数据: {{ receivedData }}</p>
    <parent-component @getData="handleGetData" />
  </div>
</template>

<script>
import ParentComponent from './parent.vue';

export default {
  name: 'Grandparent',
  components: {
    ParentComponent
  },
  data() {
    return {
      receivedData: ''
    };
  },
  methods: {
    handleGetData(data) {
      this.receivedData = data;
      console.log('从孙子组件接收到的数据:', data);
    }
  }
}
</script>
```

- 父组件：

```javascript
<template>
  <div>
    <h3>父组件</h3>
    <child-component v-on="$listeners" />
  </div>
</template>

<script>
import ChildComponent from './child.vue';

export default {
  name: 'Parent',
  components: {
    ChildComponent
  },
  created() {
    // 打印$listeners中的内容
    console.log('$listeners:', this.$listeners);
    // 输出: { getData: ƒ }
  }
}
</script>
```

- 子组件：

```javascript
<template>
  <div>
    <h5>子组件</h5>
    <button @click="postData">传递数据给祖父组件</button>
  </div>
</template>

<script>
export default {
  name: 'Child',
  methods: {
    postData() {
      this.$emit('getData', '这是从子组件传递给祖父组件的数据');
    }
  }
}
</script>
```

在这个例子中，祖父组件定义了getData事件处理函数，父组件使用v-on="$listeners"将所有事件监听器传递给子组件，子组件通过$emit('getData', data)触发事件，最终祖父组件的事件处理函数被调用。

# 3. 高级用法和注意事项

在实际开发中，我们经常会同时使用$attrs和$listeners来实现组件的高级封装：

- $attrs

1. 父组件传递给子组件的属性，在子组件中，可以通过 $attrs 获取到，而不用一个个的定义 props。
2. 所有非 props 属性绑定到相应标签，如果是组件，则绑定到组件标签上。

- $listeners

1. 父组件传递给子组件的事件，在子组件中，可以通过 $listeners 获取到，而不用一个个的定义 props。
2. 所有非 props 事件绑定到相应标签，如果是组件，则绑定到组件标签上。

**二次封装组件例子：**

- custom-checkbox.vue

```javascript
<template>
  <div class="custom-checkbox-wrapper">
    <el-checkbox 
      v-bind="$attrs" 
      v-on="$listeners" 
      @change="handleChange"
    >
      <slot></slot>
    </el-checkbox>
    <span v-if="showLabel" class="custom-label">{{ label }}</span>
  </div>
</template>

<script>
export default {
  name: 'CustomCheckbox',
  props: {
    label: String,
    showLabel: {
      type: Boolean,
      default: true
    }
  },
  methods: {
    handleChange(value) {
      // 在触发原生change事件前可以做一些额外处理
      console.log('复选框状态变化:', value);
      // 触发change事件，保持与原生组件行为一致
      this.$emit('change', value);
    }
  }
}
</script>

<style scoped>
.custom-label {
  margin-left: 8px;
  color: #606266;
}
</style>
```

- usage.vue

```javascript
<template>
  <div>
    <custom-checkbox 
      v-model="checked" 
      label="记住我" 
      @change="onChange"
      :disabled="isDisabled"
    />
  </div>
</template>

<script>
import CustomCheckbox from './custom-checkbox.vue';

export default {
  components: {
    CustomCheckbox
  },
  data() {
    return {
      checked: false,
      isDisabled: false
    };
  },
  methods: {
    onChange(value) {
      console.log('复选框状态:', value);
    }
  }
}
</script>
```

**在上面这个例子中：**

- v-bind="$attrs"将父组件传递的所有非 prop 属性传递给内部的el-checkbox组件
- v-on="$listeners"将父组件定义的所有事件监听器传递给内部组件
- 组件可以定义自己的 props 和 methods，实现额外的功能

**注意事项：**

- 优先级问题：如果子组件定义了与父组件相同的 prop 或事件处理，子组件的定义会覆盖父组件的
- 性能考虑：在大型应用中过度使用属性透传可能会影响性能
- Vue3中的变化：Vue3中$listeners已被移除，其功能合并到了$attrs中

# 4. 应用场景

- 组件库封装：在封装第三方组件库时非常有用，可以保持原有组件的 API 不变
- 高阶组件开发：创建能够增强现有组件功能的高阶组件
- 跨级组件通信：在不使用 Vuex 或 Event Bus 的情况下实现跨级组件通信
- 表单组件封装：封装表单组件时，可以方便地传递各种表单属性和事件