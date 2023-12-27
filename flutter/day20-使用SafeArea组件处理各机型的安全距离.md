# 📚 目录

1. [介绍](#1-介绍)
2. [分析](#2-分析)
3. [示例和效果图](#3-示例和效果图)
4. [特殊情况](#4-特殊情况)
---

# 1. 介绍

安全区域，指的是移动端设备的可视窗口范围。处于安全区域的内容不受圆角、刘海屏、iPhone 小黑条、状态栏等的影响，也就是说，我们要做好适配，必须保证页面可视、可操作区域是在安全区域内，而 SafeArea 组件会自动进行屏幕适配，帮你空出所占高度。如下图所示：

# 2. 解析

实际上 SafeArea 也是一个 StatelessWidget，它本身不用主动去更新 ui，它仅仅是包裹了一层 Padding 和 MediaQuery.removePadding 的容器。它外层的 Padding 填充了被异形屏所遮挡的部分，而内部的 MediaQuery.removePadding 则是把这部分的填充距离给去掉，这样就保证了 SafeArea 包裹的子元素不会被异形屏所遮挡，且后续其 child 获取外部 Padding 的时候就不会再重复获取多余的 Padding 了。

SafeArea 的属性如下：

| 属性                      | 说明                   | 默认值          |
| ------------------------- | ---------------------- | --------------- |
| top                       | 左侧是否需要有安全距离 | false           |
| left                      | 左侧是否需要有安全距离 | false           |
| right                     | 左侧是否需要有安全距离 | false           |
| bottom                    | 左侧是否需要有安全距离 | false           |
| minimum                   | 最小填充距离           | EdgeInsets.zero |
| maintainBottomViewPadding | 是否需要底部视图填充   | false           |
| child                     | 子元素                 | Widget          |

```dart
const SafeArea({
  super.key,
  this.left = true,
  this.top = true,
  this.right = true,
  this.bottom = true,
  this.minimum = EdgeInsets.zero,
  this.maintainBottomViewPadding = false,
  required this.child,
});
```

# 3. 示例和效果图

如下，是一个没有设置安全距离的页面，可以看到，由于没有设置安全区域，所以页面顶部的一部分文字被状态栏给遮挡了：

```dart
Widget build(BuildContext context) {
  return Scaffold(
    body: Container(
      alignment: Alignment.center,
      child: Column(
        children: [
          Container(
            width: double.maxFinite,
            height: 60,
            color: Colors.black.withOpacity(0.15),
            child: const Text('我是一段文字',
                style: TextStyle(color: Colors.black, fontSize: 16)),
          ),
          Expanded(
              child: Container(
                  constraints: const BoxConstraints.expand(),
                  color: Colors.white))
        ],
      ),
    )
  );
}
```

如下，是设置了 SafeArea 的，可以看到，顶部页面内容没有被遮挡：

```dart
Widget build(BuildContext context) {
  return Scaffold(
    body: SafeArea(
      child: Container(
        alignment: Alignment.center,
        child: Column(
          children: [
            Container(
              width: double.maxFinite,
              height: 60,
              color: Colors.black.withOpacity(0.15),
              child: const Text('我是一段文字',
                  style: TextStyle(color: Colors.black, fontSize: 16)),
            ),
            Expanded(
                child: Container(
                    constraints: const BoxConstraints.expand(),
                    color: Colors.white))
          ],
        ),
      ),
    )
  );
}
```

# 4. 特殊情况

有的时候，我们的老手机等设备是没有底部栏的，这时获取的安全距离为 0。这种情况我们需要使用minimum属性定一个默认高度，来保证我们的内容不被遮挡。当设备有安全距离的时候，会从默认高度和安全距离中，取最大的那个。

```dart
Widget build(BuildContext context) {
  return Scaffold(
    body: SafeArea(
      // 当有底部安全距离的时候，会从默认高度和安全距离里取最大的那个
      minimum: const EdgeInsets.only(bottom: 20),
      child: Container(
        alignment: Alignment.center,
        child: Column(
          children: [
            Container(
              width: double.maxFinite,
              height: 60,
              color: Colors.black.withOpacity(0.15),
              child: const Text('我是一段文字',
                  style: TextStyle(color: Colors.black, fontSize: 16)),
            ),
            Expanded(
                child: Container(
                    constraints: const BoxConstraints.expand(),
                    color: Colors.white))
          ],
        ),
      ),
    )
  );
}
```