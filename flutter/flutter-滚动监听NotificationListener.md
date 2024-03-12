# 📚 目录

1. [介绍](#1-介绍)
2. [用法](#2-用法)
---

# 1. 介绍

NotificationListener是flutter的通知监听器，当需要对滚动组件进行监听时，需要用NotificationListener将组件包起来，然后通过NotificationListener的onNotification方法来处理监听到的通知。它支持监听以下滚动通知：

- UserScrollNotification：用户滑动
- ScrollStartNotification：滚动开始
- ScrollUpdateNotification：滑动中
- ScrollEndNotification：滑动结束
- OverscrollNotification：滑动位置未改变，一般只有在滑动到列表边界才会回调，且需要设置不可越界，即physics为ClampingScrollPhysics，这里要注意安卓默认是这样，但是ios平台默认是弹性边界

通知的回调参数Notification.metrics里，有以下属性：

- pixels：当前绝对位置
- atEdge：是否在滚动边界
- axis：垂直或水平滚动
- axisDirection：滚动方向描述是down还是up，这个受列表reverse影响，正序就是down倒序就是up，并不代表列表是上滑还是下滑
- extentAfter：视口底部距离列表底部有多大
- extentBefore：视口顶部距离列表顶部有多大
- extentInside：视口范围内的列表长度
- maxScrollExtent：最大滚动距离，列表长度-视口长度
- minScrollExtent：最小滚动距离
- viewportDimension：沿滚动方向视口的长度
- outOfRange：是否越过边界

# 2. 用法

```dart
import 'package:flutter/material.dart';

/// 滚动通知类
class ScrollNotificationDemo extends StatelessWidget {
  const ScrollNotificationDemo({super.key});

  @override
  Widget build(BuildContext context) {
    double maxW = MediaQuery.of(context).size.width;
    double maxH = MediaQuery.of(context).size.height;

    return Scaffold(
        appBar: AppBar(
            title:
                const Text('BlurBox', style: TextStyle(color: Colors.black))),
        backgroundColor: Colors.white,
        body: SizedBox(
            width: maxW,
            height: maxH,
            child: NotificationListener<ScrollNotification>(
              onNotification: (notification) {
                if (notification is ScrollStartNotification) {
                  print('用户开始滚动');
                } else if (notification is ScrollStartNotification) {
                  print('滚动开始');
                } else if (notification is ScrollUpdateNotification) {
                  print('滚动中');
                } else if (notification is OverscrollNotification) {
                  print('滚动超出了可滚动区域的边界，并且边界处没有越界：通常是没设置滚动physics');
                } else if (notification is ScrollEndNotification) {
                  print('滚动结束');
                }
                // 滚动距离
                var offset = notification.metrics.pixels;
                // 是否在滚动边界
                var isEnd = notification.metrics.atEdge;
                // 是否越界
                var isOut = notification.metrics.outOfRange;
                // 垂直滚动还是水平滚动
                var axis = notification.metrics.axis;
                // 如果是垂直滚动
                if (axis == Axis.vertical) {
                  // 可视区的顶部与全部内容的顶部的距离
                  var extentBefore = notification.metrics.extentBefore;
                  // 可视区的底部与全部内容的底部的距离
                  var extentAfter = notification.metrics.extentAfter;
                  // 可视区的高度
                  var extentInside = notification.metrics.extentInside;
                }

                /// return true 代表此通知已处理，也就是说通知不会冒泡
                return true;
              },
              child: SingleChildScrollView(
                physics: const BouncingScrollPhysics(),
                child: Column(
                  children: List.generate(50, (index) {
                    return Container(
                      width: maxW,
                      height: 60,
                      alignment: Alignment.center,
                      decoration: const BoxDecoration(
                          border: Border(
                              bottom:
                                  BorderSide(width: 1, color: Colors.black))),
                      child: Text('${index + 1}',
                          style: const TextStyle(color: Colors.black)),
                    );
                  }),
                ),
              ),
            )));
  }
}
```