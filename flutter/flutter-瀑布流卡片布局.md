# 📚 目录

1. [前言](#1-前言)
1. [导包](#2-导包)
1. [配置](#3-配置)
1. [完整代码](#4-完整代码)
---

# 1. 前言

在一些 ToC 应用的开发过程中，经常会碰到需要上拉加载+下拉刷新的瀑布流列表布局，而 Flutter 的 ListView 组件并不支持上拉加载和下拉刷新的功能，因此需要自己实现，且瀑布流一般都是块与块之间高度不一样，因此，本文将使用(pull_to_refresh)[https://pub.dev/packages/pull_to_refresh]和(waterfall_flow)[https://pub.dev/packages/waterfall_flow]这两个插件，带你实现一个完整的瀑布流列表。

# 2. 导包

一共要用安装两个插件，分别如下：

- pull_to_refresh：上拉加载+下拉刷新

```shell
flutter pub add pull_to_refresh
```

- waterfall_flow：瀑布流布局

```shell
flutter pub add waterfall_flow
```

# 3. 插件属性

- 上拉加载+下拉刷新需要用到 SmartRefresher 组件，该组件提供了下拉刷新和上拉加载的功能，并且可以自定义头部和底部视图，属性如下：

| 属性名             | 描述                                                                 | 默认值 |
| ------------------ | -------------------------------------------------------------------- | ------ |
| key                | 组件的 Key，用于在父组件中唯一标识该组件。                           | -      |
| controller         | RefreshController 对象，用于控制刷新状态和加载状态。                 | -      |
| child              | 子组件，即需要被包裹的列表组件。                                     | -      |
| header             | 头部组件，用于自定义下拉刷新时的头部视图。                           | -      |
| footer             | 底部组件，用于自定义上拉加载时的底部视图。                           | -      |
| enablePullDown     | 是否启用下拉刷新功能。                                               | true   |
| enablePullUp       | 是否启用上拉加载功能。                                               | false  |
| enableTwoLevel     | 是否启用二级刷新功能。                                               | false  |
| onRefresh          | 当下拉刷新触发时执行的回调函数。                                     | -      |
| onLoading          | 当上拉加载触发时执行的回调函数。                                     | -      |
| onTwoLevel         | 当二级刷新触发时执行的回调函数，参数为布尔值，表示是否允许二级刷新。 | -      |
| dragStartBehavior  | 拖拽开始行为。                                                       | null   |
| primary            | 是否为主滚动视图。                                                   | null   |
| cacheExtent        | 缓存区域的大小。                                                     | null   |
| semanticChildCount | 语义子组件的数量。                                                   | null   |
| reverse            | 是否反向滚动。                                                       | null   |
| physics            | 滚动物理效果。                                                       | null   |
| scrollDirection    | 滚动方向。                                                           | null   |
| scrollController   | 滚动控制器。                                                         | null   |

- 瀑布流布局需要用到 WaterfallFlow 组件，该组件提供了瀑布流布局的功能，并且可以自定义子组件的布局方式，属性如下：

| 属性名                  | 描述                                                       | 默认值                                   |
| ----------------------- | ---------------------------------------------------------- | ---------------------------------------- |
| key                     | 组件的 Key，用于在父组件中唯一标识该组件。                 | -                                        |
| scrollDirection         | 滚动方向。                                                 | 垂直方向                                 |
| reverse                 | 是否反向滚动。                                             | false                                    |
| controller              | 滚动控制器。                                               | null                                     |
| primary                 | 是否为主滚动视图。                                         | null                                     |
| physics                 | 滚动物理效果。                                             | null                                     |
| shrinkWrap              | 是否根据子组件的总长度自动调整组件的长度。                 | false                                    |
| padding                 | 内边距。                                                   | null                                     |
| gridDelegate            | SliverWaterfallFlowDelegate 对象，用于控制瀑布流的布局。   | -                                        |
| itemBuilder             | 构建每个子组件的函数，接收 BuildContext 和索引值作为参数。 | -                                        |
| itemCount               | 子组件的数量。                                             | null                                     |
| addAutomaticKeepAlives  | 是否添加自动保持活跃的组件。                               | true                                     |
| addRepaintBoundaries    | 是否添加重绘边界。                                         | true                                     |
| addSemanticIndexes      | 是否添加语义索引。                                         | true                                     |
| cacheExtent             | 缓存区域的大小。                                           | null                                     |
| semanticChildCount      | 语义子组件的数量。                                         | null                                     |
| dragStartBehavior       | 拖拽开始行为。                                             | DragStartBehavior.start                  |
| keyboardDismissBehavior | 键盘收起行为。                                             | ScrollViewKeyboardDismissBehavior.manual |
| restorationId           | 用于恢复状态的 ID。                                        | null                                     |
| clipBehavior            | 裁剪行为。                                                 | Clip.hardEdge                            |

# 4. 完整代码

- 数据定义

```dart
/// 上下拉控制器
final RefreshController myRefreshController = RefreshController();

/// 数据列表
List<Map<String, dynamic>> dataList = [];

/// 上下拉加载的Loading 动图
final Widget loadingWidget = SizedBox();
```

- 上拉下拉容器

```dart
/// isBottomPadding：是否需要底部内边距
Widget _refreshWidget(bool isBottomPadding) {
  return SmartRefresher(
    enablePullDown: true,
    enablePullUp: true,
    primary: true,
    controller: myRefreshController,
    onRefresh: () {},
    onLoading: () {},
    header: ClassicHeader(
        // 刷新样式
        refreshingIcon: loadingWidget,
        refreshingText: '',
        // 等待样式
        idleIcon: loadingWidget,
        idleText: '',
        // 成功样式
        completeIcon: loadingWidget,
        completeText: '',
        // 释放样式
        releaseIcon: loadingWidget,
        releaseText: '',
        // 失败样式
        failedIcon: null,
        failedText: 'Loading failed',
        textStyle: TextStyle(
            fontSize: 12,
            fontWeight: FontWeight.w500,
            color: Colors.white.withOpacity(0.4)),
        spacing: 0),
    footer: ClassicFooter(
        outerBuilder: isBottomPadding
            ? (footerWidget) {
                return Container(
                    height: 112,
                    alignment: Alignment.center,
                    padding: EdgeInsets.only(bottom: 56),
                    child: footerWidget);
              }
            : null,
        height: 56,
        // 加载样式
        loadingIcon: loadingWidget,
        loadingText: '',
        // 等待样式
        idleIcon: loadingWidget,
        idleText: '',
        // 放开样式
        canLoadingIcon: loadingWidget,
        canLoadingText: '',
        // 失败样式
        failedIcon: null,
        failedText: 'Loading failed',
        textStyle: TextStyle(
            fontSize: 12,
            fontWeight: FontWeight.w500,
            color: Colors.white.withOpacity(0.4)),
        spacing: 0),
    child: listWidget(),
  );
}
```

- 瀑布流容器

```dart
Widget listContentWidget() {
  // 数据个数
  int dataLength = dataList.length;
  // 构建布局
  return WaterfallFlow.builder(
      itemCount: dataLength,
      physics: const BouncingScrollPhysics(),
      padding: EdgeInsets.all(16.w),
      gridDelegate: SliverWaterfallFlowDelegateWithFixedCrossAxisCount(
        crossAxisCount: 2,
        crossAxisSpacing: 16.w,
        mainAxisSpacing: 16.w,
        // 最后一个元素
        lastChildLayoutTypeBuilder: (index) => index == dataLength
            ? LastChildLayoutType.foot
            : LastChildLayoutType.none,
      ),
      itemBuilder: (BuildContext context, int index) {
        // 获取高度值
        int size = dataList[index].size;
        return Container(
            color: Colors.black,
            height: size * 100,
            child: SizedBox());
      });
}
```
