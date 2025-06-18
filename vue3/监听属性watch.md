# 📚 目录

1. [前言](#1-前言)
2. [常规用法](#2-常规用法)
3. [监听对象和route变化](#3-监听对象和route变化)
4. [使用场景](#4-使用场景)
5. [watchEffect详解](#5-watcheffect详解)
   - [基本概念](#5-1-基本概念)
   - [核心用法](#5-2-核心用法)
   - [高级场景应用](#5-3-高级场景应用)
6. [watch与watchEffect的对比](#6-watch与watcheffect的对比)
---

# 1. 前言

在 Vue3 中，watch 是一个独立的函数，用于响应式地监听数据变化并执行回调。与 Vue2 的选项式 API 不同，Vue3 的 watch 主要在 Composition API 中使用，需要从 `@vue/composition-api` 或 Vue3 核心包中导入。它接收三个参数：监听的数据源、回调函数和可选的配置对象。

Vue3 的 watch 具有以下特点：
- 更灵活的监听方式，支持监听 ref、reactive 对象和函数返回值
- 明确的依赖声明，避免依赖收集问题
- 更强大的配置选项，如深度监听、立即执行等

# 2. 常规用法

在 Vue3 中使用 watch 需要先导入相关函数，通常在 setup 函数中使用。watch 第一个参数可以是 ref 类型的数据、响应式对象的属性，或者一个返回值的函数。回调函数接收新值和旧值作为参数。

```javascript
<template>
  <div class="home_box">
    <h1>{{ total }}</h1>
    <button @click="handleAddTotal">增加</button>
  </div>
</template>

<script>
import { ref, watch } from 'vue';

export default {
  name: 'Home',
  setup() {
    const total = ref(0);
    
    // 常规监听 ref 类型数据
    watch(total, (newValue, oldValue) => {
      console.log('旧值：', oldValue);
      console.log('新值：', newValue);
    });
    
    const handleAddTotal = () => {
      total.value++;
    };
    
    return {
      total,
      handleAddTotal
    };
  }
}
</script>
```

# 3. 监听对象和route变化

Vue3 的 watch 可以监听多种类型的数据，包括响应式对象、嵌套属性和路由变化。对于对象类型的监听，需要注意深度监听的配置；对于路由变化，需要使用特定的监听方式。

```javascript
<template>
  <div class="home_box">
    <!-- 页面内容 -->
  </div>
</template>

<script>
import { ref, reactive, watch } from 'vue';
import { useRoute } from 'vue-router';

export default {
  name: 'Home',
  setup() {
    // 基本类型
    const aaa = ref(0);
    
    // 响应式对象
    const state = reactive({
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
    });
    
    // 获取路由实例
    const route = useRoute();
    
    // 监听基本类型
    watch(aaa, (newValue, oldValue) => {
      console.log('aaa 旧值：', oldValue);
      console.log('aaa 新值：', newValue);
    });
    
    // 监听基本类型，初始化立即执行
    watch(() => state.bbb, (newValue, oldValue) => {
      console.log('bbb 旧值：', oldValue);
      console.log('bbb 新值：', newValue);
    }, { immediate: true });
    
    // 监听对象，需要深度监听
    watch(
      () => state.ccc, 
      (newValue, oldValue) => {
        console.log('ccc 旧值：', oldValue);
        console.log('ccc 新值：', newValue);
      }, 
      { deep: true }
    );
    
    // 监听对象嵌套属性
    watch(
      () => state.ddd.d2.d21, 
      (newValue, oldValue) => {
        console.log('d21 旧值：', oldValue);
        console.log('d21 新值：', newValue);
      }
    );
    
    // 监听路由变化
    watch(
      () => route.path, 
      (newPath, oldPath) => {
        console.log('路由旧路径：', oldPath);
        console.log('路由新路径：', newPath);
      }
    );
    
    return {
      aaa,
      state
    };
  }
}
</script>
```

# 4. 使用场景

Vue3 的 watch 在很多场景中都有重要应用，特别是在需要响应数据变化并执行副作用的场景中。以下是几个典型使用场景：

### 4.1 即时表单验证

```javascript
<template>
  <div class="home_box">
    <input type="text" v-model="username">
    <div v-if="validationError" class="error-message">{{ validationError }}</div>
  </div>
</template>

<script>
import { ref, watch } from 'vue';

export default {
  name: 'FormValidation',
  setup() {
    const username = ref('');
    const validationError = ref('');
    
    // 监听输入框内容变化，即时验证
    watch(username, (newValue) => {
      if (newValue.length < 3) {
        validationError.value = '用户名至少需要3个字符';
      } else if (newValue.length > 20) {
        validationError.value = '用户名不能超过20个字符';
      } else {
        validationError.value = '';
      }
    });
    
    return {
      username,
      validationError
    };
  }
}
</script>
```

### 4.2 搜索联想功能

```javascript
<template>
  <div class="home_box">
    <input type="text" v-model="searchKeyword">
    <div v-if="searchResults.length > 0" class="suggestions">
      <div v-for="result in searchResults" :key="result">{{ result }}</div>
    </div>
  </div>
</template>

<script>
import { ref, watch } from 'vue';

export default {
  name: 'SearchSuggestions',
  setup() {
    const searchKeyword = ref('');
    const searchResults = ref([]);
    
    // 监听搜索关键词变化，延迟发送请求
    let searchTimeout;
    watch(searchKeyword, (newValue) => {
      if (newValue.length < 2) {
        searchResults.value = [];
        return;
      }
      
      // 防抖处理，避免频繁请求
      clearTimeout(searchTimeout);
      searchTimeout = setTimeout(() => {
        // 模拟API请求
        searchResults.value = [
          `${newValue} 相关结果1`,
          `${newValue} 相关结果2`,
          `${newValue} 相关结果3`
        ];
      }, 500);
    });
    
    return {
      searchKeyword,
      searchResults
    };
  }
}
</script>
```

### 4.3 数据变化联动处理

```javascript
<template>
  <div class="home_box">
    <div>
      <label>宽度：</label>
      <input type="number" v-model="width">
    </div>
    <div>
      <label>高度：</label>
      <input type="number" v-model="height">
    </div>
    <div>面积：{{ area }}</div>
  </div>
</template>

<script>
import { ref, watch } from 'vue';

export default {
  name: 'DimensionCalculator',
  setup() {
    const width = ref(10);
    const height = ref(20);
    const area = ref(0);
    
    // 监听宽度和高度变化，计算面积
    watch([width, height], ([newWidth, newHeight]) => {
      area.value = newWidth * newHeight;
    });
    
    return {
      width,
      height,
      area
    };
  }
}
</script>
```

# 5. watchEffect详解

## 5-1 基本概念

`watchEffect` 是 Vue3 中 Composition API 提供的响应式副作用监听函数，与 `watch` 的主动监听不同，它会**自动追踪回调函数中使用的响应式依赖**，当依赖变化时触发回调。其核心特点包括：

- **自动依赖收集**：无需显式声明监听目标，回调内使用的响应式数据会被自动追踪
- **初始化立即执行**：默认在组件挂载后立即执行一次回调，用于处理初始依赖的副作用
- **简洁的语法**：适用于依赖多个响应式数据的场景，避免重复声明监听源

## 5-2 核心用法

### 基础示例：自动响应依赖变化

```javascript
<template>
  <div class="home_box">
    <h3>计数器：{{ count }}</h3>
    <h3>翻倍值：{{ doubleCount }}</h3>
    <button @click="count++">+1</button>
  </div>
</template>

<script>
import { ref, watchEffect } from 'vue';

export default {
  name: 'WatchEffectDemo',
  setup() {
    const count = ref(0);
    const doubleCount = ref(0);
    
    // watchEffect 会自动监听 count 的变化
    watchEffect(() => {
      console.log('依赖变化触发回调');
      doubleCount.value = count.value * 2;
    });
    
    // 初始化时立即输出
    // 控制台打印：依赖变化触发回调
    
    return {
      count,
      doubleCount
    };
  }
}
</script>
```

### 处理异步副作用

```javascript
<template>
  <div class="home_box">
    <input v-model="searchText" placeholder="搜索...">
    <div v-if="loading">加载中...</div>
    <div v-else-if="searchResults.length > 0">
      <ul>
        <li v-for="item in searchResults" :key="item">{{ item }}</li>
      </ul>
    </div>
    <div v-else>无搜索结果</div>
  </div>
</template>

<script>
import { ref, watchEffect } from 'vue';

export default {
  name: 'SearchEffect',
  setup() {
    const searchText = ref('');
    const searchResults = ref([]);
    const loading = ref(false);
    
    watchEffect(async onInvalidate => {
      // 当 searchText 变化时，先清除之前的定时器
      let timeout;
      onInvalidate(() => clearTimeout(timeout));
      
      if (searchText.value.length < 2) {
        searchResults.value = [];
        return;
      }
      
      loading.value = true;
      // 模拟异步搜索
      timeout = setTimeout(async () => {
        try {
          // 模拟API请求
          const response = await new Promise(resolve => {
            setTimeout(() => {
              resolve([
                `${searchText.value} 结果1`,
                `${searchText.value} 结果2`,
                `${searchText.value} 结果3`
              ]);
            }, 500);
          });
          searchResults.value = response;
        } catch (error) {
          console.error('搜索失败', error);
        } finally {
          loading.value = false;
        }
      }, 300);
    });
    
    return {
      searchText,
      searchResults,
      loading
    };
  }
}
</script>
```

### 停止监听与清理副作用

```javascript
<template>
  <div class="home_box">
    <button @click="toggleWatch">
      {{ isWatching ? '停止监听' : '开始监听' }}
    </button>
    <div>计数器：{{ count }}</div>
  </div>
</template>

<script>
import { ref, watchEffect } from 'vue';

export default {
  name: 'WatchEffectControl',
  setup() {
    const count = ref(0);
    const isWatching = ref(true);
    let watcher = null;
    
    // 条件性创建 watchEffect
    const toggleWatch = () => {
      if (isWatching.value) {
        // 启动监听
        watcher = watchEffect(() => {
          console.log('监听中，count:', count.value);
        });
      } else {
        // 停止监听
        watcher && watcher();
      }
      isWatching.value = !isWatching.value;
    };
    
    return {
      count,
      isWatching,
      toggleWatch
    };
  }
}
</script>
```

## 5-3 高级场景应用

### 监听多个响应式依赖

```javascript
<template>
  <div class="home_box">
    <div>
      <label>宽度：</label>
      <input type="number" v-model="width">
    </div>
    <div>
      <label>高度：</label>
      <input type="number" v-model="height">
    </div>
    <div>面积：{{ area }}</div>
    <div>周长：{{ perimeter }}</div>
  </div>
</template>

<script>
import { ref, watchEffect } from 'vue';

export default {
  name: 'DimensionCalculator',
  setup() {
    const width = ref(10);
    const height = ref(20);
    const area = ref(0);
    const perimeter = ref(0);
    
    // 同时监听 width 和 height 的变化
    watchEffect(() => {
      area.value = width.value * height.value;
      perimeter.value = 2 * (width.value + height.value);
    });
    
    return {
      width,
      height,
      area,
      perimeter
    };
  }
}
</script>
```

### 处理 DOM 副作用

```javascript
<template>
  <div class="home_box" ref="container">
    <h3>滚动位置：{{ scrollY }}</h3>
  </div>
</template>

<script>
import { ref, onMounted, watchEffect } from 'vue';

export default {
  name: 'ScrollEffect',
  setup() {
    const container = ref(null);
    const scrollY = ref(0);
    
    onMounted(() => {
      // 监听容器滚动事件
      watchEffect(onInvalidate => {
        if (!container.value) return;
        
        const handleScroll = () => {
          scrollY.value = container.value.scrollTop;
        };
        
        container.value.addEventListener('scroll', handleScroll);
        
        // 组件卸载或依赖变化时清理事件
        onInvalidate(() => {
          container.value.removeEventListener('scroll', handleScroll);
        });
      });
    });
    
    return {
      container,
      scrollY
    };
  }
}
</script>
```

# 6. watch与watchEffect的对比

| 特性                | watch                                                                 | watchEffect                                                          |
|---------------------|-----------------------------------------------------------------------|----------------------------------------------------------------------|
| **依赖声明方式**    | 显式指定监听的数据源（ref、reactive 属性、函数返回值）                | 隐式收集回调函数中使用的响应式依赖                                  |
| **初始化执行**      | 默认不执行，需通过 `immediate: true` 配置                             | 初始化时立即执行一次                                                 |
| **回调参数**        | 接收新值和旧值 `(newValue, oldValue)`                                | 不接收参数，通过闭包访问最新值                                       |
| **适用场景**        | 监听特定数据的变化，需要获取新旧值对比                                | 处理与多个响应式数据相关的副作用，无需明确依赖                       |
| **性能优化**        | 可精确控制监听目标，避免无效触发                                      | 依赖收集可能包含不必要的响应式数据，需注意副作用清理                 |
| **语法复杂度**      | 语法相对繁琐，需声明监听源和配置项                                    | 语法简洁，适合快速实现响应式副作用                                   |
| **异步处理**        | 更适合处理需要等待特定数据变化的异步操作                              | 适合处理随依赖变化自动更新的异步副作用（如API请求、DOM操作）         |

## 选择建议：
- **使用 watch**：
  - 需要明确知道监听的数据源
  - 只在特定数据变化时执行回调
  - 需要获取变化前后的新旧值对比
- **使用 watchEffect**：
  - 处理与多个响应式数据相关的副作用
  - 初始化时需要立即执行一次副作用
  - 希望以更简洁的方式实现响应式逻辑

```javascript
// 场景：监听用户登录状态并请求用户数据
setup() {
  const isLoggedIn = ref(false);
  const userData = ref(null);
  
  // 使用 watch 监听登录状态变化
  watch(isLoggedIn, (newVal) => {
    if (newVal) {
      fetchUserData(); // 只在登录状态变化时请求
    }
  });
  
  // 使用 watchEffect 自动响应视图更新
  watchEffect(() => {
    document.title = `用户状态: ${isLoggedIn.value ? '已登录' : '未登录'}`;
  });
}
```