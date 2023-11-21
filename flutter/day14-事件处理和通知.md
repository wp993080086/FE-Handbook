# 📚 目录

1. [原始指针事件处理](#1-原始指针事件处理)
---

# 1. 原始指针事件

在移动端，各个平台或UI系统的原始指针事件模型基本都是一致，即：一次完整的事件分为三个阶段：手指按下、手指移动、和手指抬起，而更高级别的手势（如点击、双击、拖动等）都是基于这些原始事件的。当指针按下时，Flutter会对应用程序执行命中测试(Hit Test)，以确定指针与屏幕接触的位置存在哪些组件（widget）， 指针按下事件（以及该指针的后续事件）然后被分发到由命中测试发现的最内部的组件，然后从那里开始，事件会在组件树中向上冒泡，这些事件会从最内部的组件被分发到组件树根的路径上的所有组件。

## 1-1. Listener组件

Flutter中，可以通过`Listener`组件来监听指针触摸事件，有如下事件：

|事件名|描述|
|--|--|
|onPointerDown|手指按下回调|
|onPointerMove|手指移动回调|
|onPointerUp|手指抬起回调|
|onPointerCancel|触摸事件取消回调|
|behavior|子组件如何响应命中测试|

- 事件属性

|属性|描述|
|--|--|
|position|指针相对于当对于全局坐标的偏移|
|localPosition|指针相对于当对于本身布局坐标的偏移|
|delta|两次指针移动事件的距离|
|pressure|按压力度，如果手机屏幕不支持压力传感器，则始终为1|
|orientation|指针移动方向，是一个角度值|



如下例子，手指在一个容器上移动时查看手指相对于容器的位置：

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
  PointerEvent? myEventObj;
  @override
  Widget build(BuildContext context) {
    return Listener(
        child: Scaffold(
            appBar: AppBar(
              title: const Text('Flutter Home'),
            ),
            body: Container(
                alignment: Alignment.center,
                width: 300.0,
                height: 150.0,
                child: Text('点击此处：${myEventObj?.localPosition ?? ''}')),
            floatingActionButton: FloatingActionButton(
                onPressed: () async {}, child: const Icon(Icons.palette))),
        onPointerDown: (PointerDownEvent event) =>
            setState(() => myEventObj = event),
        onPointerMove: (PointerMoveEvent event) =>
            setState(() => myEventObj = event),
        onPointerUp: (PointerUpEvent event) => setState(() => myEventObj = event));
  }
}
```

## 1-2. 忽略指针事件

假如我们不想让某个子树响应PointerEvent的话，我们可以使用IgnorePointer和AbsorbPointer，这两个组件都能阻止子树接收指针事件，不同之处在于AbsorbPointer本身会参与命中测试，而IgnorePointer本身不会参与，这就意味着AbsorbPointer本身是可以接收指针事件的（但其子树不行），而IgnorePointer不可以。一个简单的例子如下：

```dart
Listener(
  child: AbsorbPointer(
    child: Listener(
      child: Container(
        color: Colors.red,
        width: 200.0,
        height: 100.0,
      ),
      onPointerDown: (event) => debugPrint("a"),
    ),
  ),
  onPointerDown: (event) => debugPrint("b"),
)
```

# 2. 手势识别

GestureDetector是一个用于手势识别的功能性组件，我们通过它可以来识别各种手势。GestureDetector 内部封装了 Listener，用以识别语义化的手势。

## 2-1. 单击双击和长按

当同时监听onTap和onDoubleTap事件时，当用户触发tap事件时，会有200毫秒左右的延时，这是因为当用户点击完之后很可能会再次点击以触发双击事件，所以GestureDetector会等一段时间来确定是否为双击事件。如果只监听了onTap（没有监听onDoubleTap）事件时，则没有延时。

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

GestureDetector对于拖动和滑动事件是没有区分的，他们本质上是一样的。GestureDetector会将要监听的组件的原点（左上角）作为本次手势的原点，当用户在监听的组件上按下手指时，手势识别就会开始。

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

GestureDetector也可以监听缩放事件，如下例子：

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

## 2-4. GestureRecognizer语义手势

GestureRecognizer的作用就是通过Listener来将原始指针事件转换为语义手势，它是一个抽象类，一种手势的识别器对应一个GestureRecognizer的子类。


例子，给一段富文本（RichText）的不同部分分别添加点击事件处理器：

```dart
import 'package:flutter/material.dart';
import 'package:flutter/gestures.dart';

/// 定义
class HomePage extends StatefulWidget {
  const HomePage({super.key});

  @override
  State<HomePage> createState() => HomePageState();
}

/// 实现
class HomePageState extends State<HomePage> {
  TapGestureRecognizer myTapGestureRecognizer = TapGestureRecognizer();
  bool myToggle = false;

  @override
  void dispose() {
    myTapGestureRecognizer.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
        appBar: AppBar(
          title: const Text('Flutter Home'),
        ),
        body: Center(
          child: Text.rich(TextSpan(
              children: [
                const TextSpan(text: 'Hello'),
                TextSpan(
                  recognizer: myTapGestureRecognizer
                    ..onTap = () {
                      debugPrint('点击$myToggle');
                      setState(() {
                        myToggle = !myToggle;
                      });
                    },
                  text: '点我变色',
                  style: TextStyle(
                      fontSize: 30.0,
                      color: myToggle ? Colors.blue : Colors.red),
                )
              ])),
        ),
        floatingActionButton: FloatingActionButton(
            onPressed: () {}, child: const Icon(Icons.palette)));
  }
}
```

# 3. 事件机制

- 组件只有通过命中测试才能响应事件。
- 一个组件是否通过命中测试取决于 hitTestChildren(...) || hitTestSelf(...) 的值
- 组件树中组件的命中测试顺序是深度优先的。
- 组件子节点命中测试的循序是倒序的，并且一旦有一个子节点的 hitTest 返回了 true，就会终止遍历，后续子节点将没有机会参与命中测试。这个原则可以结合 Stack 组件来理解。
- 大多数情况下 Listener 的 HitTestBehavior 为 opaque 或 translucent 效果是相同的，只有当其子节点的 hitTest 返回为 false 时才会有区别。
- HitTestBlocker 是一个很灵活的组件，我们可以通过它干涉命中测试的各个阶段。

# 4. 通知

通知是Flutter中一个重要的机制，在widget树中，每一个节点都可以分发通知，通知会沿着当前节点向上传递，所有父节点都可以通过NotificationListener来监听通知。Flutter中将这种由子向父的传递通知的机制称为通知冒泡（Notification Bubbling）。通知冒泡和用户触摸事件冒泡是相似的，但有一点不同：通知冒泡可以中止，但用户触摸事件不行。

## 4-1. 监听通知

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

  @override
  Widget build(BuildContext context) {
    return Scaffold(
        appBar: AppBar(
          title: const Text('Flutter Home'),
        ),
        body: NotificationListener(
          onNotification: (notification) {
            switch (notification.runtimeType){
              case ScrollStartNotification: debugPrint("开始滚动"); break;
              case ScrollUpdateNotification: debugPrint("正在滚动"); break;
              case ScrollEndNotification: debugPrint("滚动停止"); break;
              case OverscrollNotification: debugPrint("滚动到边界"); break;
            }
            return false;
          },
          child: ListView.builder(
              itemCount: 100,
              itemBuilder: (context, index) {
                return ListTile(title: Text("$index"),);
              }
          ),
        ));
  }
}
```

## 4-2. 自定义通知

除了 Flutter 内部通知，我们也可以自定义通知。

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
  String myMsg = '';

  @override
  Widget build(BuildContext context) {
    return Scaffold(
        appBar: AppBar(
          title: const Text('Flutter Home'),
        ),
        body: NotificationListener<MyNotification>(
          onNotification: (notification) {
            myMsg += notification.msg.toString();
            debugPrint(myMsg);
            return true;
          },
          child: ListView.builder(
              itemCount: 100,
              itemBuilder: (context, index) {
                return ListTile(title: Text("$index"),onTap: () {
                  MyNotification("Hi").dispatch(context);
                });
              },
          ),
        ));
  }
}

/// 定义一个通知类，继承自Notification类
class MyNotification extends Notification {
  MyNotification(this.msg);
  final String msg;
}
```