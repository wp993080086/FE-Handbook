# 📚 目录

1. [介绍](#1-介绍)
2. [使用](#2-使用)
   1. [单击双击和长按](#2-1-单击双击和长按)
   2. [拖动和滑动](#2-2-拖动和滑动)
   3. [缩放](#2-3-缩放)
3. [注意点](#3-注意点)
---

# 1. 介绍

在 flutter 中，GestureDetector 是手势识别的组件，可以识别点击、双击、长按、拖动、缩放等手势事件，并且可以与子组件进行交互，构造函数属性如下：

```dart
(new) GestureDetector GestureDetector({
    // 可选的Key属性，用于标识该组件
    Key? key,
    // 可选的子组件，将被包裹在GestureDetector中
    Widget? child,
    // 当用户按下手指时触发的事件处理函数
    void Function(TapDownDetails)? onTapDown,
    // 当用户抬起手指时触发的事件处理函数
    void Function(TapUpDetails)? onTapUp,
    // 当用户轻触屏幕时触发的事件处理函数
    void Function()? onTap,
    // 当用户取消触摸屏幕时触发的事件处理函数
    void Function()? onTapCancel,
    // 当用户轻触屏幕的次级区域时触发的事件处理函数
    void Function()? onSecondaryTap,
    // 当用户按下次级区域的手指时触发的事件处理函数
    void Function(TapDownDetails)? onSecondaryTapDown,
    // 当用户抬起次级区域的手指时触发的事件处理函数
    void Function(TapUpDetails)? onSecondaryTapUp,
    // 当用户取消触摸次级区域的屏幕时触发的事件处理函数
    void Function()? onSecondaryTapCancel,
    // 当用户轻触屏幕的三级区域时触发的事件处理函数
    void Function()? onTertiaryTap,
    // 当用户按下三级区域的手指时触发的事件处理函数
    void Function(TapDownDetails)? onTertiaryTapDown,
    // 当用户抬起三级区域的手指时触发的事件处理函数
    void Function(TapUpDetails)? onTertiaryTapUp,
    // 当用户取消触摸三级区域的屏幕时触发的事件处理函数
    void Function()? onTertiaryTapCancel,
    // 当用户双击屏幕时触发的事件处理函数
    void Function()? onDoubleTap,
    // 当用户双击屏幕时触发的事件处理函数
    void Function()? onDoubleTapCancel,
    // 当用户长按屏幕时触发的事件处理函数
    void Function(LongPressDownDetails)? onLongPressDown,
    // 当用户取消长按屏幕时触发的事件处理函数
    void Function()? onLongPressCancel,
    // 当用户长按屏幕时触发的事件处理函数
    void Function()? onLongPress,
    // 当用户开始长按屏幕时触发的事件处理函数
    void Function(LongPressStartDetails)? onLongPressStart,
    // 当用户移动手指以更新长按位置时触发的事件处理函数
    void Function(LongPressMoveUpdateDetails)? onLongPressMoveUpdate,
    // 当用户抬起手指以结束长按时触发的事件处理函数
    void Function()? onLongPressUp,
    // 当用户结束长按屏幕时触发的事件处理函数
    void Function(LongPressEndDetails)? onLongPressEnd,
    // 当用户长按屏幕的次级区域时触发的事件处理函数
    void Function()? onSecondaryLongPress,
    // 当用户长按屏幕的次级区域时触发的事件处理函数
    void Function()? onSecondaryLongPressCancel,
    // 当用户长按屏幕的次级区域时触发的事件处理函数
    void Function(LongPressStartDetails)? onSecondaryLongPressStart,
    // 当用户移动手指以更新次级长按位置时触发的事件处理函数
    void Function(LongPressMoveUpdateDetails)? onSecondaryLongPressMoveUpdate,
    // 当用户抬起手指以结束次级长按时触发的事件处理函数
    void Function()? onSecondaryLongPressUp,
    // 当用户结束次级长按屏幕时触发的事件处理函数
    void Function(LongPressEndDetails)? onSecondaryLongPressEnd,
    // 当用户长按屏幕的三级区域时触发的事件处理函数
    void Function()? onTertiaryLongPress,
    // 当用户长按屏幕的三级区域时触发的事件处理函数
    void Function()? onTertiaryLongPressCancel,
    // 当用户长按屏幕的三级区域时触发的事件处理函数
    void Function(LongPressStartDetails)? onTertiaryLongPressStart,
    // 当用户移动手指以更新三级长按位置时触发的事件处理函数
    void Function(LongPressMoveUpdateDetails)? onTertiaryLongPressMoveUpdate,
    // 当用户抬起手指以结束三级长按时触发的事件处理函数
    void Function()? onTertiaryLongPressUp,
    // 当用户结束三级长按屏幕时触发的事件处理函数
    void Function(LongPressEndDetails)? onTertiaryLongPressEnd,
    // 当用户垂直拖动屏幕时触发的事件处理函数
    void Function(DragDownDetails)? onVerticalDragDown,
    // 当用户开始垂直拖动屏幕时触发的事件处理函数
    void Function(DragStartDetails)? onVerticalDragStart,
    // 当用户更新垂直拖动位置时触发的事件处理函数
    void Function(DragUpdateDetails)? onVerticalDragUpdate,
    // 当用户结束垂直拖动屏幕时触发的事件处理函数
    void Function(DragEndDetails)? onVerticalDragEnd,
    // 当用户取消垂直拖动屏幕时触发的事件处理函数
    void Function()? onVerticalDragCancel,
    // 当用户水平拖动屏幕时触发的事件处理函数
    void Function(DragDownDetails)? onHorizontalDragDown,
    // 当用户开始水平拖动屏幕时触发的事件处理函数
    void Function(DragStartDetails)? onHorizontalDragStart,
    // 当用户更新水平拖动位置时触发的事件处理函数
    void Function(DragUpdateDetails)? onHorizontalDragUpdate,
    // 当用户结束水平拖动屏幕时触发的事件处理函数
    void Function(DragEndDetails)? onHorizontalDragEnd,
    // 当用户取消水平拖动屏幕时触发的事件处理函数
    void Function()? onHorizontalDragCancel,
    // 当用户开始强制按压屏幕时触发的事件处理函数
    void Function(ForcePressDetails)? onForcePressStart,
    // 当用户达到最大按压力时触发的事件处理函数
    void Function(ForcePressDetails)? onForcePressPeak,
    // 当用户更新按压力度时触发的事件处理函数
    void Function(ForcePressDetails)? onForcePressUpdate,
    // 当用户结束按压屏幕时触发的事件处理函数
    void Function(ForcePressDetails)? onForcePressEnd,
    // 当用户开始平移屏幕时触发的事件处理函数
    void Function(DragDownDetails)? onPanDown,
    // 当用户开始平移屏幕时触发的事件处理函数
    void Function(DragStartDetails)? onPanStart,
    // 当用户更新平移位置时触发的事件处理函数
    void Function(DragUpdateDetails)? onPanUpdate,
    // 当用户结束平移屏幕时触发的事件处理函数
    void Function(DragEndDetails)? onPanEnd,
    // 当用户取消平移屏幕时触发的事件处理函数
    void Function()? onPanCancel,
    // 当用户开始缩放屏幕时触发的事件处理函数
    void Function(ScaleStartDetails)? onScaleStart,
    // 当用户更新缩放比例时触发的事件处理函数
    void Function(ScaleUpdateDetails)? onScaleUpdate,
    // 当用户结束缩放屏幕时触发的事件处理函数
    void Function(ScaleEndDetails)? onScaleEnd,
    // 当用户的指针设备类型被识别时触发的事件处理函数
    HitTestBehavior? behavior,
    // 是否从语义中排除此组件，默认为false
    bool excludeFromSemantics = false,
    // 拖动开始时的手势行为，默认为start
    DragStartBehavior dragStartBehavior = DragStartBehavior.start,
    // 是否由跟踪板滚动引起缩放，默认为false
    bool trackpadScrollCausesScale = false,
    // 跟踪板滚动到缩放因子的值，默认为kDefaultTrackpadScrollToScaleFactor
    Offset trackpadScrollToScaleFactor = kDefaultTrackpadScrollToScaleFactor,
    // 支持的设备类型集合，默认为空集
    Set<PointerDeviceKind>? supportedDevices,
})
```

# 2. 使用

GestureDetector 内部封装了 Listener，用以识别语义化的手势。

## 2-1. 单击双击和长按

当同时监听 onTap 和 onDoubleTap 事件时，当用户触发 tap 事件时，会有 200 毫秒左右的延时，这是因为当用户点击完之后很可能会再次点击以触发双击事件，所以 GestureDetector 会等一段时间来确定是否为双击事件。如果只监听了 onTap（没有监听 onDoubleTap）事件时，则没有延时。

```dart
import 'package:flutter/material.dart';

/// 定义
class HomePage extends StatefulWidget {
  const HomePage({super.key});

  @override
  State<HomePage> createState() => HomePageState();
}

/// 实现
class HomePageState extends State<HomePage> {
  String msg = '';

  @override
  Widget build(BuildContext context) {
    return Scaffold(
        appBar: AppBar(
          title: const Text('Flutter Home'),
        ),
        body: Center(
          child: GestureDetector(
            child: Container(
              alignment: Alignment.center,
              color: Colors.blue,
              width: 200.0,
              height: 100.0,
              child: Text(
                msg,
                style: const TextStyle(color: Colors.white),
              ),
            ),
            onTap: () {
              setState(() {
                msg = '单击';
              });
            },
            onDoubleTap: () {
              setState(() {
                msg = '双击';
              });
            },
            onLongPress: () {
              msg = '长按';
            },
          ),
        ),
        floatingActionButton: FloatingActionButton(
            onPressed: () async {}, child: const Icon(Icons.palette)));
  }
}
```

## 2-2. 拖动和滑动

GestureDetector 对于拖动和滑动事件是没有区分的，他们本质上是一样的。GestureDetector 会将要监听的组件的原点（左上角）作为本次手势的原点，当用户在监听的组件上按下手指时，手势识别就会开始。

```dart
import 'package:flutter/material.dart';

/// 定义
class HomePage extends StatefulWidget {
  const HomePage({super.key});

  @override
  State<HomePage> createState() => HomePageState();
}

/// 实现
class HomePageState extends State<HomePage> {
  double topOffset = 0.0;
  double leftOffset = 0.0;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
        appBar: AppBar(
          title: const Text('Flutter Home'),
        ),
        body: Stack(
          children: [
            Positioned(
              top: topOffset,
              left: leftOffset,
              child: GestureDetector(
                onPanDown: (DragDownDetails ev) {
                  debugPrint('手指按下');
                },
                onPanUpdate: (DragUpdateDetails ev) {
                  setState(() {
                    topOffset += ev.delta.dy;
                    leftOffset += ev.delta.dx;
                  });
                },
                onPanEnd: (DragEndDetails  ev) {
                  debugPrint('手指拿开');
                },
                child: const CircleAvatar(
                  child: Text('拖'),
                ),
              ),
            )
          ],
        ),
        floatingActionButton: FloatingActionButton(
            onPressed: () async {}, child: const Icon(Icons.palette)));
  }
}
```

## 2-3. 缩放

GestureDetector 也可以监听缩放事件，如下例子：

```dart
import 'package:flutter/material.dart';

/// 定义
class HomePage extends StatefulWidget {
  const HomePage({super.key});

  @override
  State<HomePage> createState() => HomePageState();
}

/// 实现
class HomePageState extends State<HomePage> {
  double imgW = 200;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
        appBar: AppBar(
          title: const Text('Flutter Home'),
        ),
        body: Center(
          child: GestureDetector(
            child: Image.asset('static/portrait.png', width: imgW,),
            onScaleUpdate: (ScaleUpdateDetails details) {
              setState(() {
                imgW = 200 * details.scale.clamp(.8, 10.0);
              });
            },
          ),
        ),
        floatingActionButton: FloatingActionButton(
            onPressed: () async {}, child: const Icon(Icons.palette)));
  }
}
```

# 3. 注意点

有时候 GestureDetector 实现点击事件时，点击空白区域不能响应，这是因为子元素没有占满全部内容，此时，需要设置 behavior 属性，它有三个值，如下例子：

| 属性         | 说明                                                                                                                                                                     |
| ------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| deferToChild | 只有当前容器中的 child 被点击时才会响应点击事件。                                                                                                                        |
| opaque       | 点击整个区域都会响应点击事件，但是点击事件不可穿透向下传递，注释翻译：阻止视觉上位于其后方的目标接收事件。                                                               |
| translucent  | 同样是点击整个区域都会响应点击事件，和 opaque 的区别是点击事件是否可以向下传递，注释翻译：半透明目标既可以在其范围内接受事件，也可以允许视觉上位于其后方的目标接收事件。 |

```dart
Column(children: [
  GestureDetector(
    behavior: HitTestBehavior.opaque,
    onTap: () {},
    child: Container(
      width: double.infinity,
      height: 64,
      alignment: Alignment.center,
      child: Text('Delete',
          style: TextStyle(
              color: Color(0xFFFB4056),
              fontSize: 18,
              fontWeight: FontWeight.w600)),
    ),
  )
])
```
