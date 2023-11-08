[toc]

# 1、前言

使用Vue-Router时，会将一些字段信息附加到路由的Meta对象里面，比如图标icon，标题，权限等，如下：

```javascript
{
  path: '/billboard/board/:boardId',
  name: 'billboardBoard',
  props: true,
  component: () => import('@/views/billboard/board.vue'),
  meta: {
    title: 'message.router.billboard',
    isHide: true,
    isKeepAlive: false,
    isAffix: false,
    isIframe: false,
    icon: 'iconfont icon-board'
  }
}
```

但是在使用的过程中，TS会认为这些字段是unknown类型，从而导致赋值或者使用的时候会报错：

```javascript
router.beforeEach((to) => {
	document.title = to.meta.title || '默认标题'
})
```

如图：

# 2、解决

为了避免报错和标红（虽然不影响程序运行），我们可以通过扩展RouteMeta接口，声明Meta的字段，这样在使用过程中不仅不会报错标红，还会有代码提示，方法如下：

- 在根目录或者types目录下，新建一个`router-meta.d.ts`文件，文件内容如下：

```typescript
/**
 * @description 扩展ruoter-meta的类型 此处必须要export {} 不然找不到类型
 */
declare module 'vue-router' {
	interface RouteMeta {
		permission?: Array<string>
		title?: string
		icon?: string
		affix?: boolean
		hidden?: boolean
		keepAlive?: boolean
	}
}

export {}
```

- 在根目录tsconfig.json的include选项中，能够包含到这个文件即可。

```json
{
  "include": ["src/**/*.ts", "src/**/*.d.ts", "src/**/*.tsx", "src/**/*.vue", "types/**/*.d.ts", "types/**/*.ts"],
  "exclude": ["node_modules/**", "dist", "**/*.js"]
}
```

如图：