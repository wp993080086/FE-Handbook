[toc]

# 1、前言

最近在做一个文件夹管理的功能，要实现一个树状的文件夹面板。里面包含两种元素，文件夹以及文件。交互要求如下：

1. 创建、删除，重命名文件夹和文件
2. 可以拖拽，拖拽文件到文件夹中，或着拖拽文件夹到文件夹中
3. 文件夹可展开，显示里面全部文件
4. 拖拽的时候要有辅助线显示

# 2、分析

根据这个要求，我先找到了[vue.draggable.next](https://www.itxst.com/vue-draggable-next/tutorial.html)这个库，结合[elementPlus](https://element-plus.org/zh-CN/component/collapse.html)的Collapse折叠面板，以及Vue 3的[递归组件](https://cn.vuejs.org/api/sfc-script-setup.html#using-components)封装了一个组件`drag-folder`，结果测试发现，这个库太久没维护了，很多事件不支持，导致功能很难实现。比如，拖动的时候拿不到拖动对象所选中的目标、没有辅助线、Collapse折叠面板关闭后无法拖入等问题。

就在头疼之时，我不小心看到了elementPlus的[Tree](https://element-plus.org/zh-CN/component/tree.html#%E5%8F%AF%E6%8B%96%E6%8B%BD%E8%8A%82%E7%82%B9)组件，发现它也支持拖拽，而且有辅助线，有丰富的回调事件，于是，我准备魔改一下。

# 3、 实现

Tree组件只需要准备一个树状数据，然后根据数据渲染出Tree组件即可，可以自定义子节点的键名，也可以使用插槽自定义内容，于是一番操作后，我完成了第二版的`drag-folder`组件：

```html
<template>
	<div class="drag_folder_box">
		<el-tree
			draggable
			node-key="uid"
			:default-expanded-keys="defaultExpanded"
			:data="interiorList"
			:allow-drop="handleDragBehavior"
			:allow-drag="handleAllowDrag"
			@node-drag-start="handleDragStart"
			@node-drag-enter="handleDragEnter"
			@node-drag-leave="handleDragLeave"
			@node-drag-over="handleDragOver"
			@node-drag-end="handleDragEnd"
			@node-drop="handleDrop"
			@node-click="handleSwitchBillboard"
		>
			<template #default="{ data }">
				<div class="tree_item">
					<div class="item_title">{{ data.label }}</div>
					<div class="item_operate">
						<div class="operate_item" title="编辑" @click.stop="editBillboard(data)">
							<el-icon :size="16">
								<ele-Edit />
							</el-icon>
						</div>
						<div class="operate_item" title="删除" @click.stop="deleteBillboard(data)">
							<el-icon :size="16">
								<ele-Delete />
							</el-icon>
						</div>
					</div>
				</div>
			</template>
		</el-tree>
	</div>
</template>

<script setup lang="ts">
import { ref, onMounted, watch } from 'vue'
import type Node from 'element-plus/es/components/tree/src/model/node'
import type { DragEvents } from 'element-plus/es/components/tree/src/model/useDragNode'
import type { AllowDropType, NodeDropType } from 'element-plus/es/components/tree/src/tree.type'
import type { IGetBillboardListTreeItem } from '@/types/data-billboard'

// #region 组件逻辑
interface IProps {
	list?: Array<IGetBillboardListTreeItem>
}

const props = withDefaults(defineProps<IProps>(), {
	list: () => []
})

const emit = defineEmits(['edit', 'delete', 'switch', 'change'])

const interiorList = ref<Array<IGetBillboardListTreeItem>>([])
// #endregion

// #region 拖拽逻辑
watch(
	() => props.list,
	(newValue) => {
		interiorList.value = newValue
	},
	{
		deep: true,
		immediate: true
	}
)
// 节点开始拖拽时
const handleDragStart = (node: Node, ev: DragEvents) => {}

// 拖拽进入其他节点时
const handleDragEnter = (draggingNode: Node, dropNode: Node, ev: DragEvents) => {}

// 拖拽离开某个节点时
const handleDragLeave = (draggingNode: Node, dropNode: Node, ev: DragEvents) => {}

// 在拖拽节点时
const handleDragOver = (draggingNode: Node, dropNode: Node, ev: DragEvents) => {}

// 拖拽结束时
const handleDragEnd = (draggingNode: Node, dropNode: Node, dropType: NodeDropType, ev: DragEvents) => {}

// 拖拽成功时
const handleDrop = (draggingNode: Node, dropNode: Node, dropType: NodeDropType, ev: DragEvents) => {
	emit('change', interiorList.value)
}
// 是否允许拖拽
const handleAllowDrag = (draggingNode: Node) => {
	return true
}
// 拖拽行为判断
const handleDragBehavior = (draggingNode: Node, dropNode: Node, type: AllowDropType) => {
	// 如果选中的节点不是看板 则不允许拖动到内部，只能是 'prev' 或 'next'
	if (dropNode.data.type === 'billboard') {
		return type !== 'inner'
	}
	return true
}

// 编辑看板/文件夹
const editBillboard = (data: IGetBillboardListTreeItem) => {
	emit('edit', data)
}

// 删除看板/文件夹
const deleteBillboard = (data: IGetBillboardListTreeItem) => {
	emit('delete', data)
}

// 选择看板
const handleSwitchBillboard = (data: IGetBillboardListTreeItem) => {
	if (data.type === 'dir') return
	if (data.id === props.billboardId) return
	emit('switch', data)
}
// #endregion

// #region 生命周期
onMounted(() => {
	interiorList.value = props?.list || []
})
// #endregion
</script>
```

这个组件，使用起来很简单，只需要引入组件，然后绑定list就行了，下面我讲一下这里面的一些坑。

# 4、踩坑

这里面有几个坑，但是基本都解决了。

## 4.1、拖拽辅助线的坑

Tree组件没有itemSize属性，它的辅助线，默认是26px，而我的每一项，是36px的高度，所以就会导致辅助线不能对准。最开始我想着修改样式，给height和line-height，发现不起作用。于是我去翻源码，发现源码：node_modules\element-plus\lib\components\tree\src\model\useDragNode.js里，treeNodeDragOver方法是给辅助线设置top的，这个top是根据前面的iconPosition的高度来的，所以我设置了icon的height和line-height，一顿操纵如下：

```css
.el-tree-node__expand-icon {
	line-height: 36px !important;
	height: 36px !important;
	padding: 0px 2px !important;
}
```

改完发现，面板折叠起来是正常的，但是打开后就还是不正常，审查元素发现，这个icon会旋转，打开面板后会添加一个expanded的类名，该类名添加了transform: rotate(90deg)的属性，导致高度不对了，于是我又一顿操作：

```css
.el-tree-node__expand-icon {
	line-height: 36px !important;
	height: 36px !important;
	padding: 0px 2px !important;
}
.el-tree-node__expand-icon.expanded {
	transform: rotate(0deg);
	& svg {
		transform: rotate(90deg);
	}
}
```

## 4.2、数据的坑

这个是我们后端设计的锅。文件夹和文件的ID，会出现一样的，没有唯一ID，没办法，谁让我们前端就是这么善解人意，写个递归函数，拼接一个唯一ID出来咯。

```typescript
const formatBillboardList = (list: Array<IBillboardTreeItem>, pid: number): Array<IBillboardTreeItem> => {
	return list.map((item, index) => {
		// 唯一ID
		const uid = `${item.type}_${item.id}`
		// 父ID
		const parentId = pid === 0 ? item.id : pid
		// 子层
		const children = Array.isArray(item.children) ? formatBillboardList(item.children, item.id) : []
		return {
			...item,
			uid,
			order: index,
			parentId,
			children
		}
	})
}

const list = formatBillboardList(res.data, 0)
```

## 4.3、限制拖拽

文件夹和文件，都是可以拖拽的，但是文件可以拖拽到文件夹上，文件夹不能拖拽到文件里。这里我们用到了Tree组件的allow-drop处理函数，它又三个参数，分别是拖拽对象，目标对象，拖拽类型。

```typescript
const handleDragBehavior = (draggingNode: Node, dropNode: Node, type: AllowDropType) => {
	// 如果选中的节点不是看板 则不允许拖动到内部，只能是 'prev' 或 'next'
	if (dropNode.data.type === 'billboard') {
		return type !== 'inner'
	}
	return true
}
```

## 4.4、样式调整

- 文件夹和文件的样式不一样，要区分，这里我们用Tree组件的props属性，定制class实现

```typescript
const treeOption = {
	class: (data: IGetBillboardListTreeItem, node: Node) => {
		if (data.id === props.billboardId && data.type === 'billboard') {
			return 'on_tree_item'
		} else if (data.type === 'dir') {
			return 'folder'
		} else {
			return 'billboard'
		}
	}
}
```

- 每一层，要有不同的缩进，比如第一层缩进10，第二层20，以此类推，这里我们用Tree组件的indent属性实现，直接绑定即可。

```html
<el-tree :indent="10"></el-tree>
```

- 修改hover和focus时候的背景色

```css
.drag_folder_box {
  &:deep(.el-tree-node__content) {
		width: 100%;
		height: auto;
		border-bottom: 1px solid #c1c1c1;
		&:hover {
			background-color: #e4f2ff !important;
		}
		&:focus {
			background-color: #e4f2ff !important;
		}
	}
}
```

如上，基本的样式和功能就全部完成了。

---