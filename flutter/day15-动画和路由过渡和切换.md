# 📚 目录

1. [介绍](#1-介绍)
2. [介绍](#2-flutter中动画抽象)
    1. [Animation](#2-1-animation)
    2. [Curve](#2-2-curve)
    3. [AnimationController](#2-3-animationcontroller)
    4. [Tween](#2-4-tween)
    5. [监听动画](#2-5-监听动画)
3. [自定义路由切换动画](#3-自定义路由切换动画)
4. [Hero飞行动画](#4-hero飞行动画)
5. [交织动画](#5-交织动画)
6. [动画切换组件](#6-动画切换组件)
    1. [AnimatedSwitcher](#6-1-animatedswitcher)
    2. [AnimatedSwitcher封装](#6-2-animatedswitcher封装)
7. [动画过渡组件](#7-动画过渡组件)
---

# 1. 介绍

在任何系统的UI框架中，动画实现的原理都是相同的，即：在一段时间内，快速地多次改变UI外观；由于人眼会产生视觉暂留，所以最终看到的就是一个“连续”的动画。我们将UI的一次改变称为一个动画帧，对应一次屏幕刷新，而决定动画流畅度的一个重要指标就是帧率FPS，即每秒的动画帧数。帧率越高则动画就会越流畅！一般情况下，对于人眼来说，动画帧率超过16 FPS，就基本能看了，超过 32 FPS就会感觉相对平滑，而超过 32 FPS，大多数人基本上就感受不到差别了。由于动画的每一帧都是要改变UI输出，是比较耗资源的，而在Flutter中，理想情况下是可以实现 60FPS 的，这和原生应用能达到的帧率是基本是持平的。

# 2. Flutter中动画抽象

为了方便开发者创建动画，不同的UI系统对动画都进行了一些抽象，比如在 Android 中可以通过XML来描述一个动画然后设置给View。Flutter中也对动画进行了抽象，主要涉及 Animation、Curve、Controller、Tween这四个角色，它们一起配合来完成一个完整动画

## 2-1. Animation

Animation是一个抽象类，它本身和UI渲染没有任何关系，它主要的功能是保存动画的插值和状态。Animation对象是一个在一段时间内依次生成一个区间（Tween）之间值的类。Animation对象在整个动画执行过程中输出的值可以是线性的、曲线的、一个步进函数或者任何其他曲线函数等等，这由Curve来决定。 根据Animation对象的控制方式，动画可以正向运行（从起始状态开始，到终止状态结束），也可以反向运行，甚至可以在中间切换方向。我们可以通过Animation来监听动画每一帧以及执行状态的变化：

- addListener()：用于给Animation添加帧监听器，在每一帧都会被调用。帧监听器中最常见的行为是改变状态后调用setState()来触发UI重建。

- addStatusListener()：给Animation添加“动画状态改变”监听器；动画开始、结束、正向或反向时会调用状态改变的监听器。

## 2-2. Curve

动画过程可以是匀速的、匀加速的或者先加速后减速等。Flutter中通过Curve（曲线）来描述动画过程，我们把匀速动画称为线性的，而非匀速动画称为非线性的。

|曲线|动画过程|
|--|--|
|linear|匀速的|
|decelerate|匀减速|
|ease|开始加速，后面减速|
|easeIn|开始慢，后面快|
|easeOut|开始快，后面慢|
|easeInOut|开始慢，然后加速，最后再减速|

- 指定动画曲线

```dart
final CurvedAnimation curve = CurvedAnimation(parent: controller, curve: Curves.easeIn);
```

- 定义一个正弦曲线

```dart
class ShakeCurve extends Curve {
  @override
  double transform(double t) {
    return math.sin(t * math.PI * 2);
  }
}
```
## 2-3. AnimationController

AnimationController用于控制动画，它包含动画的启动forward()、停止stop() 、反向播放 reverse()等方法。AnimationController会在动画的每一帧，就会生成一个新的值。默认情况下，AnimationController在给定的时间段内线性的生成从 0.0 到1.0（默认区间）的数字。 

- 创建animation对象

```dart
final AnimationController controller = AnimationController(
  // 动画时长
  duration: const Duration(milliseconds: 2000),
  // 指定生成数字的区间
  lowerBound: 10.0,
  upperBound: 20.0,
  // 绑定State对象
  vsync: this
);
```

## 2-4. Tween

默认情况下，AnimationController对象值的范围是[0.0，1.0]。如果我们需要构建UI的动画值在不同的范围或不同的数据类型，则可以使用Tween来添加映射以生成不同的范围或数据类型的值。

```dart
final Tween doubleTween = Tween<double>(begin: -200.0, end: 0.0);
```

## 2-5. 监听动画

我们可以通过Animation的addStatusListener()方法来添加动画状态改变监听器。Flutter中，有四种动画状态，在AnimationStatus枚举类中定义。

|属性值|描述|
|--|--|
|dismissed|动画在起始点停止|
|forward|动画正在正向执行|
|reverse|动画正在反向执行|
|completed|动画在终点停止|

- 完整动画例子：

```dart
import 'package:flutter/material.dart';

/// 定义
class HomePage extends StatefulWidget {
  const HomePage({super.key});

  @override
  State<HomePage> createState() => HomePageState();
}

/// 实现 需要继承TickerProvider，如果有多个AnimationController，则应该继承TickerProviderStateMixin
class HomePageState extends State<HomePage> with SingleTickerProviderStateMixin {
  late Animation<double> animation;
  late AnimationController controller;

  @override
  void initState() {
    super.initState();
    controller = AnimationController(
      duration: const Duration(seconds: 3),
      vsync: this
    );

    // 使用弹性曲线
    animation = CurvedAnimation(parent: controller, curve: Curves.bounceIn);

    // 匀速图片宽高从0变到300
    animation = Tween(begin: 0.0, end: 300.0).animate(controller)
      ..addListener(() {
        setState(() => {});
      });

    animation.addStatusListener((status) {
      if (status == AnimationStatus.completed) {
        // 动画执行结束时反向执行动画
        controller.reverse();
      } else if (status == AnimationStatus.dismissed) {
        // 动画恢复到初始状态时执行动画（正向）
        controller.forward();
      }
    });

    // 启动动画（正向执行）
    controller.forward();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
        appBar: AppBar(
          title: const Text('Flutter Home'),
        ),
        body: Container(
          alignment: Alignment.center,
          child: Image.asset('static/portrait.png',width: animation.value,height: animation.value),
        ));
  }

  @override
  void dispose() {
    controller.dispose();
    super.dispose();
  }
}
```

# 3. 自定义路由切换动画

Material组件库中提供了一个MaterialPageRoute组件，它可以使用和平台风格一致的路由切换动画，如在iOS上会左右滑动切换，而在Android上会上下滑动切换。现在，我们如果在Android上也想使用左右切换风格，该怎么做？一个简单的作法是可以直接使用CupertinoPageRoute。

```dart
Navigator.push(context, CupertinoPageRoute(  
   builder: (context)=>PageB(),
 ));
```

- 使用PageRouteBuilder来自定义路由切换动画

```dart
import 'package:flutter/material.dart';
import 'package:demo1/views/login/view.dart';

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
        body: Container(
          alignment: Alignment.center,
          child: Image.asset('static/portrait.png',width: 200.0,height: 200.0),
        ),
    floatingActionButton: FloatingActionButton(
      onPressed: () {
        Navigator.push(
          context,
          PageRouteBuilder(
            transitionDuration: const Duration(milliseconds: 500),
            pageBuilder: (BuildContext context, Animation<double> animation,
                Animation secondaryAnimation) {
              return FadeTransition(
                // 使用渐隐渐入过渡,
                opacity: animation,
                // 其他页面
                child: const LoginPage(),
              );
            },
          ),
        );
      },
      child: const Text('跳'),
    ),);
  }
}
```

# 4. Hero飞行动画

在Flutter中将图片从一个路由“飞”到另一个路由称为hero动画，尽管相同的动作有时也称为 共享元素转换。实现 Hero 动画只需要用Hero组件将要共享的 widget 包装起来，并提供一个相同的 tag 即可。

```dart
import 'package:flutter/material.dart';
import 'package:english_words/english_words.dart';
import 'package:demo1/views/login/view.dart';

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
        body: Container(
          alignment: Alignment.center,
          child: InkWell(
            onTap: () {
              Navigator.push(context, PageRouteBuilder(
                pageBuilder: (
                  BuildContext context,
                  animation,
                  secondaryAnimation,
                ) {
                  return FadeTransition(
                    opacity: animation,
                    child: Scaffold(
                      appBar: AppBar(
                        title: const Text("原图"),
                      ),
                      body: HeroAnimationB(),
                    ),
                  );
                },
              ));
            },
            child: Hero(
              tag: 'xxxxx',
              child: ClipOval(
                child: Image.asset('static/portrait.png', width: 50.0),
              ),
            ),
          ),
        ));
  }
}

/// 大图
class HeroAnimationB extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Center(
      child: Hero(
        tag: 'xxxxx',
        child: Image.asset('static/portrait.png'),
      ),
    );
  }
}
```

# 5. 交织动画

有些时候我们可能会需要一些复杂的动画，这些动画可能由一个动画序列或重叠的动画组成。要实现这种效果，使用交织动画比较简单。要创建交织动画，需要使用多个动画对象，并且使用一个AnimationController控制所有的动画对象，给每一个动画对象指定时间间隔。

```dart
import 'package:flutter/material.dart';

/// 定义
class HomePage extends StatefulWidget {
  const HomePage({super.key});

  @override
  State<HomePage> createState() => HomePageState();
}

/// 实现
class HomePageState extends State<HomePage> with TickerProviderStateMixin {
  late AnimationController myController;

  @override
  void initState() {
    super.initState();

    myController = AnimationController(
      duration: const Duration(milliseconds: 2000),
      vsync: this,
    );
  }

  handlePlayAnimation() async {
    try {
      //先正向执行动画
      await myController.forward().orCancel;
      //再反向执行动画
      await myController.reverse().orCancel;
    } on TickerCanceled {
      //捕获异常。可能发生在组件销毁时，计时器会被取消。
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
        appBar: AppBar(
          title: const Text('Flutter Home'),
        ),
        body: Container(
          alignment: Alignment.center,
          child: Column(
            children: [
              ElevatedButton(
                onPressed: () => handlePlayAnimation(),
                child: const Text("Start Animation"),
              ),
              Container(
                width: 300.0,
                height: 300.0,
                decoration: BoxDecoration(
                  color: Colors.black.withOpacity(0.1),
                  border: Border.all(
                    color: Colors.black.withOpacity(0.5),
                  ),
                ),
                //调用我们定义的交错动画Widget
                child: StaggerAnimation(controller: myController),
              ),
            ],
          ),
        ));
  }
}

/// 动画组件
class StaggerAnimation extends StatelessWidget {
  StaggerAnimation({
    Key? key,
    required this.controller,
  }) : super(key: key) {
    // 高度动画
    height = Tween<double>(
      begin: .0,
      end: 300.0,
    ).animate(
      CurvedAnimation(
        parent: controller,
        curve: const Interval(
          0.0, 0.6, //间隔，前60%的动画时间
          curve: Curves.ease,
        ),
      ),
    );
    // 颜色动画
    color = ColorTween(
      begin: Colors.green,
      end: Colors.red,
    ).animate(
      CurvedAnimation(
        parent: controller,
        curve: const Interval(
          0.0, 0.6, //间隔，前60%的动画时间
          curve: Curves.ease,
        ),
      ),
    );
    // 偏移动画
    padding = Tween<EdgeInsets>(
      begin: const EdgeInsets.only(left: .0),
      end: const EdgeInsets.only(left: 100.0),
    ).animate(
      CurvedAnimation(
        parent: controller,
        curve: const Interval(
          0.6, 1.0, //间隔，后40%的动画时间
          curve: Curves.ease,
        ),
      ),
    );
  }

  late final Animation<double> controller;
  late final Animation<double> height;
  late final Animation<EdgeInsets> padding;
  late final Animation<Color?> color;

  Widget handleBuildAnimation(BuildContext context, child) {
    return Container(
      alignment: Alignment.bottomCenter,
      padding: padding.value,
      child: Container(
        color: color.value,
        width: 50.0,
        height: height.value,
      ),
    );
  }

  @override
  Widget build(BuildContext context) {
    return AnimatedBuilder(
      builder: handleBuildAnimation,
      animation: controller,
    );
  }
}
```

# 6. 动画切换组件

开发中，我们经常会遇到切换UI元素的场景，比如Tab切换、路由切换。为了增强用户体验，通常在切换时都会指定一个动画，以使切换过程显得平滑。Flutter SDK中提供了一个AnimatedSwitcher组件，它定义了一种通用的UI切换抽象。

## 6-1. AnimatedSwitcher

AnimatedSwitcher 可以同时对其新、旧子元素添加显示、隐藏动画。也就是说在AnimatedSwitcher的子元素发生变化时，会对其旧元素和新元素做动画。当AnimatedSwitcher的 child 发生变化时（类型或 Key 不同），旧 child 会执行隐藏动画，新 child 会执行执行显示动画。默认情况，AnimatedSwitcher会对新旧child执行“渐隐”和“渐显”动画。


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
  int myCount = 0;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
        appBar: AppBar(
          title: const Text('Flutter Home'),
        ),
        body: Container(
          alignment: Alignment.center,
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              AnimatedSwitcher(
                duration: const Duration(milliseconds: 500),
                transitionBuilder: (Widget child, Animation<double> animation) {
                  // 执行缩放动画
                  return ScaleTransition(scale: animation, child: child);
                },
                child: Text(
                  '$myCount',
                  // 显示指定key，不同的key会被认为是不同的Text，这样才能执行动画
                  key: ValueKey<int>(myCount),
                  style: Theme.of(context).textTheme.headline4,
                ),
              ),
              ElevatedButton(
                child: const Text('+1'),
                onPressed: () {
                  setState(() {
                    myCount += 1;
                  });
                },
              )
            ],
          ),
        ));
  }
}
```

## 6-2. AnimatedSwitcher封装

封装一个通用的SlideTransitionX 来实现左出右入，上入下出等 出入动画。修改direction的值即可修改方向。

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
  int myCount = 0;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
        appBar: AppBar(
          title: const Text('Flutter Home'),
        ),
        body: Container(
          alignment: Alignment.center,
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              AnimatedSwitcher(
                duration: const Duration(milliseconds: 500),
                transitionBuilder: (Widget child, Animation<double> animation) {
                  // 执行缩放动画
                  return SlideTransitionX(
                    direction: AxisDirection.down, // 上入下出
                    position: animation,
                    child: child
                  );
                },
                child: Text(
                  '$myCount',
                  // 显示指定key，不同的key会被认为是不同的Text，这样才能执行动画
                  key: ValueKey<int>(myCount),
                  style: Theme.of(context).textTheme.headline4,
                ),
              ),
              ElevatedButton(
                child: const Text('+1'),
                onPressed: () {
                  setState(() {
                    myCount += 1;
                  });
                },
              )
            ],
          ),
        ));
  }
}

/// 封装的动画容器
class SlideTransitionX extends AnimatedWidget {
  SlideTransitionX({
    Key? key,
    required Animation<double> position,
    this.transformHitTests = true,
    this.direction = AxisDirection.down,
    required this.child,
  }) : super(key: key, listenable: position) {
    switch (direction) {
      case AxisDirection.up:
        _tween = Tween(begin: const Offset(0, 1), end: const Offset(0, 0));
        break;
      case AxisDirection.right:
        _tween = Tween(begin: const Offset(-1, 0), end: const Offset(0, 0));
        break;
      case AxisDirection.down:
        _tween = Tween(begin: const Offset(0, -1), end: const Offset(0, 0));
        break;
      case AxisDirection.left:
        _tween = Tween(begin: const Offset(1, 0), end: const Offset(0, 0));
        break;
    }
  }

  final bool transformHitTests;

  final Widget child;

  final AxisDirection direction;

  late final Tween<Offset> _tween;

  @override
  Widget build(BuildContext context) {
    final position = listenable as Animation<double>;
    Offset offset = _tween.evaluate(position);
    if (position.status == AnimationStatus.reverse) {
      switch (direction) {
        case AxisDirection.up:
          offset = Offset(offset.dx, -offset.dy);
          break;
        case AxisDirection.right:
          offset = Offset(-offset.dx, offset.dy);
          break;
        case AxisDirection.down:
          offset = Offset(offset.dx, -offset.dy);
          break;
        case AxisDirection.left:
          offset = Offset(-offset.dx, offset.dy);
          break;
      }
    }
    return FractionalTranslation(
      translation: offset,
      transformHitTests: transformHitTests,
      child: child,
    );
  }
}
```

# 7. 动画过渡组件

在Widget属性发生变化时会执行过渡动画的组件统称为”动画过渡组件“，而动画过渡组件最明显的一个特征就是它会在内部自管理AnimationController。而为了方便使用者可以自定义动画的曲线、执行时长、方向等，通常都需要使用者自己提供一个AnimationController对象来自定义这些属性值。如此一来，使用者就必须得手动管理AnimationController，这又会增加使用的复杂性。因此，如果也能将AnimationController进行封装，则会大大提高动画组件的易用性。Flutter SDK中也预置了很多动画过渡组件，如下：

|组件名|功能|
|---|---|
|AnimatedPadding|在padding发生变化时会执行过渡动画到新状态|
|AnimatedPositioned|配合Stack一起使用，当定位状态发生变化时会执行过渡动画到新的状态。|
|AnimatedOpacity|在透明度opacity发生变化时执行过渡动画到新状态|
|AnimatedAlign|当alignment发生变化时会执行过渡动画到新的状态|
|AnimatedContainer|当Container属性发生变化时会执行过渡动画到新的状态|
|AnimatedDefaultTextStyle|当字体样式发生变化时，子组件中继承了该样式的文本组件会动态过渡到新样式|

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
  double myHeight = 100;
  Color myColor = Colors.red;

  @override
  Widget build(BuildContext context) {
    var duration = const Duration(milliseconds: 400);

    return Scaffold(
        appBar: AppBar(
          title: const Text('Flutter Home'),
        ),
        body: Container(
          alignment: Alignment.center,
          child: AnimatedContainer(
              duration: duration,
              height: myHeight,
              color: myColor,
              child: TextButton(
                  onPressed: () {
                    if (myHeight < 300) {
                      setState(() {
                        myHeight = 300;
                        myColor = Colors.blue;
                      });
                    } else {
                      setState(() {
                        myHeight = 100;
                        myColor = Colors.red;
                      });
                    }
                  },
                  child: const Text(
                    '点击执行变化',
                    style: TextStyle(color: Colors.white),
                  ))),
        ));
  }
}
```