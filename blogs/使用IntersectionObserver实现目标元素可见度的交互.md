[toc]

# 1、前言

IntersectionObserver 是浏览器原生提供的一个Api。可以"观察"我们的元素是否可见，原理是判断目标元素与可见区域的交叉比例，所以也被称为"交叉观察器"。本文通过一个Demo代码，讲解如何使用IntersectionObserver，来实现根据目标元素可见度的交互效果。并附有兼容性，使用场景，完整代码实现，以及成熟的Hooks推荐。

# 2、代码实现

下面代码（React 18）是使用 IntersectionObserver 来监听列表元素的可见度，当卡片元素完全进入浏览器视口时，会标记该元素为活动状态；当元素离开视口时，取消活动状态。

- 代码

```tsx
import { useEffect, useRef, useState } from 'react'
import moduleStyle from './index.module.scss'

/**
 * @description Demo
 */
export default function DemoPage() {
	// #region 数据
	const [dataList, setDataList] = useState<T_Any[]>([])
	const observer = useRef<T_Any>(null)
	// #endregion

	// #region 逻辑
	/**
	 * @description 初始化模拟请求数据
	 */
	useEffect(() => {
		setTimeout(() => {
			const list = [...Array(10)].map((_, index) => {
				return { value: (index + 1).toString(), isActive: false }
			})
			setDataList(list)
		}, 1000)
	}, [])

	/**
	 * @description 监听
	 */
	useEffect(() => {
		observer.current = new IntersectionObserver(
			(entries) => {
				entries.forEach((entry, index) => {
					if (entry.isIntersecting) {
						const tempList = [...dataList]
						tempList[index].isActive = true
						setDataList(tempList)
					} else {
						const tempList = [...dataList]
						tempList[index].isActive = false
						setDataList(tempList)
					}
				})
			},
			{
				root: null,
				threshold: 1.0 // 当可见度达到100%时触发
			}
		)

		const cardElements = document.querySelectorAll('.card')
		cardElements.forEach((el) => observer.current.observe(el))

		return () => {
			if (observer.current) {
				observer.current.disconnect()
			}
		}
	}, [dataList])
	// #endregion

	// #region 视图
	return (
		<div className={moduleStyle['demo-wrapper']}>
			<div className={`scrollbar-none ${moduleStyle['list-wrapper']}`}>
				{dataList.map((item) => (
					<div key={item.value} className={`card ${moduleStyle['item-wrapper']} ${item.isActive ? moduleStyle['active'] : ''}`}>
						{item.value}
					</div>
				))}
			</div>
		</div>
	)
	// #endregion
}
```

- dataList：是一个状态变量，用于存储模拟请求得到的数据列表。每个数据项包含 value 和 isActive 两个属性，value 是元素的标识，isActive 表示元素是否处于活动状态。
- observer：是一个 useRef 创建的引用，用于存储 IntersectionObserver 实例。
- 第一个 useEffect 钩子：用于模拟异步请求数据。在组件挂载后，通过 setTimeout 模拟接口 1 秒的延迟，生成包含 10 个元素的列表，并更新 dataList 状态。
- 第二个 useEffect 钩子用于初始化 IntersectionObserver。当 dataList 发生变化时，创建一个新的 IntersectionObserver 实例，该实例会监听所有具有 card 类名的元素。当元素完全进入视口（可见度达到 100%）时，将其 isActive 属性设置为 true；当元素离开视口时，将 isActive 属性设置为 false。同时，在组件卸载时，断开 IntersectionObserver 的连接，避免内存泄漏。

- 样式

```scss
.demo-wrapper {
	width: 100%;
	height: 100%;
	padding: 16px;
	.list-wrapper {
		display: flex;
		align-items: center;
		justify-content: flex-start;
		flex-wrap: wrap;
		gap: 16px;
		width: 100%;
		height: 100%;
		overflow-y: auto;
		.item-wrapper {
			display: flex;
			align-items: center;
			justify-content: center;
			width: calc(50% - 8px);
			height: 320px;
			color: #ffffff;
			font-size: 24px;
			background-color: #cccccc;
			transition: ease 0.3s;
			border-radius: 8px;
			overflow: hidden;
		}
		.active {
			background-color: #6212d6 !important;
		}
	}
}
```

# 3、使用场景

- 懒加载图片：在图片较多的页面中，可以使用 IntersectionObserver 监听图片元素的可见性。当图片元素进入视口时，再加载图片，从而减少初始加载时的资源消耗，提高页面性能。
- 无限滚动加载：在社交网站、新闻列表等场景中，当用户滚动页面接近底部时，通过监听底部元素的可见性，触发加载更多数据的操作，实现无限滚动加载的效果。
- 动画触发：当某个元素进入视口时，触发动画效果，增强页面的交互性和视觉效果。例如，元素淡入、滑动等动画。

# 4、兼容性

|环境|版本|支持情况|
|--|--|--|
|Chrome|51+|支持|
|Firefox|55+|支持|
|Edge|79+（新版本基于Chromium）|支持|
|Safari|12.1+|支持|
|iOS Safari|13+|支持|
|Opera|38+|支持|
|Safari on macOS|12.1+|支持|
|Android Browser|通常跟随WebView版本|需要检查具体设备|
|Chrome for Android|51+|支持|

如果不支持，可以使用 [IntersectionObserver polyfill](https://www.npmjs.com/package/intersection-observer) 来实现兼容性支持。

# 5、成熟的Hooks推荐

- Vue
  - [useElementVisibility](https://vueuse.nodejs.cn/core/useElementVisibility/)

- React
  - [useInViewport](https://ahooks.js.org/hooks/use-in-viewport)