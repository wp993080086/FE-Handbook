# 📚 目录

1. [填充组件Padding和EdgeInsets](#1-填充组件padding和edgeinsets)
2. [装饰容器DecoratedBox和BoxDecoration](#2-装饰容器decoratedbox和boxdecoration)
3. [变换Transform](#3-变换transform)
    1. [RotatedBox](#3-1-rotatedbox)
---

# 1. 简介

通常可滚动组件的子组件可能会非常多、占用的总高度也会非常大；如果要一次性将子组件全部构建出将会非常昂贵！为此，Flutter中提出一个Sliver（中文为“薄片”的意思）概念，Sliver 可以包含一个或多个子组件。Sliver 的主要作用是配合：加载子组件并确定每一个子组件的布局和绘制信息，如果 Sliver 可以包含多个子组件时，通常会实现按需加载模型。只有当 Sliver 出现在视口中时才会去构建它，这种模型也称为“基于Sliver的列表按需加载模型”。可滚动组件中有很多都支持基于Sliver的按需加载模型，如ListView、GridView，但是也有不支持该模型的，如SingleChildScrollView。

Flutter 中的可滚动组件主要由三个角色组成：

- Scrollable ：用于处理滑动手势，确定滑动偏移，滑动偏移变化时构建 Viewport。

|方法|值|描述|
|---|---|---|
|axisDirection|-|滚动方向|
|physics|ClampingScrollPhysics、BouncingScrollPhysics|决定可滚动组件如何响应用户操作|
|controller|-|控制滚动位置和监听滚动事件|
|viewportBuilder|-|当用户滑动时，Scrollable 会调用此回调构建新的 Viewport，同时传递一个 ViewportOffset 类型的 offset 参数，该参数描述 Viewport 应该显示那一部分内容|

- Viewport：显示的视窗，即列表的可视区域。

|方法|值|描述|
|---|---|---|
|axisDirection|-|滚动方向|
|offset|-|用户的滚动偏移|
|cacheExtent|-|预渲染区域|
|CacheExtentStyle|-|用于配合解释cacheExtent的含义，也可以为主轴长度的乘数|
|slivers|-|需要显示的 Sliver 列表|

- Sliver：视窗里显示的元素，对子组件进行构建和布局。

> 如果要给可滚动组件添加滚动条，只需将Scrollbar作为可滚动组件的任意一个父级组件即可

# 2. 可滚动组件

本节讲一下SingleChildScrollView和ListView。

## 2-1 SingleChildScrollView

SingleChildScrollView类似于Android中的ScrollView，它只能接收一个子组件。通常SingleChildScrollView只应在期望的内容不会超过屏幕太多时使用，因为不支持基于 Sliver 的延迟加载模型，可能性能差。

|方法|值|描述|
|---|---|---|
|scrollDirection|-|滚动方向|
|reverse|-||
|padding|-|内边距|
|primary|-|是否使用 widget 树中默认的PrimaryScrollControlle|
|physics|-|决定可滚动组件如何响应用户操作|
|controller|-|控制器|
|child|-|子组件|

竖向排列的字母表例子：

```dart
import 'package:flutter/material.dart';

class HomePage extends StatefulWidget {
  const HomePage({super.key});

  @override
  State<HomePage> createState() => HomePageState();
}

class HomePageState extends State<HomePage> {
  String alphabet = 'ABCDEFGHIJKMLNOPKRSTUVWSYZ';
  @override
  Widget build(BuildContext context) {
    return Scaffold(
        appBar: AppBar(
          title: const Text('Home Page'),
          backgroundColor: Theme.of(context).colorScheme.inversePrimary,
        ),
        body: Scrollbar(
          child: SingleChildScrollView(
            padding: const EdgeInsets.all(16.0),
            child: Center(
                child: Column(
              children: alphabet
                  .split('')
                  .map((e) => Text(
                        e,
                        textScaleFactor: 2.0,
                      ))
                  .toList(),
            )),
          ),
        ));
  }
}
```

## 2-2. ListView

ListView是最常用的可滚动组件之一，内部组合了 Scrollable、Viewport 和 Sliver。它可以沿一个方向线性排布所有子组件，并且它也支持列表项懒加载（在需要时才会创建）。它有一个children参数，它接受一个Widget列表。这种方式适合只有少量的子组件数量已知且比较少的情况，反之则应该使用ListView.builder 按需动态构建列表项。

|方法|值|描述|
|---|---|---|
|itemExtent|-|子组件长度，如果指定了该值，就不用动态计算listview高度，性能更好|
|prototypeItem|-|指定一个列表项，可滚动组件会在 layout 时计算一次它延主轴方向的长度，和itemExtent互斥|
|shrinkWrap|false|是否根据子组件的总长度来设置ListView的长度|
|addRepaintBoundaries|-|是否将列表项（子组件）包裹在RepaintBoundary组件中，看list复杂情况设置|

- 数量固定的情况：

```dart
ListView(
  shrinkWrap: true, 
  padding: const EdgeInsets.all(16.0),
  children: <Widget>[
    const Text('a'),
    const Text('b'),
    const Text('c'),
    const Text('d'),
  ],
);
```

- 数量不固定的情况：

```dart
ListView.builder(
  itemCount: 100,
  // 强制高度为50.0
  itemExtent: 50.0,
  itemBuilder: (BuildContext context, int index) {
    return ListTile(title: Text("$index"));
  }
);
```

## 2-1. separated分割线

ListView.separated可以在生成的列表项之间添加一个分割组件，它比ListView.builder多了一个separatorBuilder参数，该参数是一个分割组件生成器。

- 奇数行添加一条红色下划线，偶数行添加一条绿色下划线：

```dart
import 'package:flutter/material.dart';

class HomePage extends StatefulWidget {
  const HomePage({super.key});

  @override
  State<HomePage> createState() => HomePageState();
}

class HomePageState extends State<HomePage> {
  Widget divider1 = const Divider(color: Colors.red);
  Widget divider2 = const Divider(color: Colors.green);

  @override
  Widget build(BuildContext context) {
    return Scaffold(
        appBar: AppBar(
          title: const Text('Home Page'),
          backgroundColor: Theme.of(context).colorScheme.inversePrimary,
        ),
        body: ListView.separated(
          itemCount: 100,
          // 列表项构造器
          itemBuilder: (BuildContext context, int index) {
            return ListTile(title: Text("$index"));
          },
          // 分割器构造器
          separatorBuilder: (BuildContext context, int index) {
            return index % 2 == 0 ? divider1 : divider2;
          },
        ));
  }
}
```

## 2-2. 无限加载列表

在实际开发中，我们经常需要实现一个无限加载列表，即当列表滚动到底部时，再向服务器请求数据，并添加到列表，例子如下：

```dart
import 'package:flutter/material.dart';
import 'package:english_words/english_words.dart';

class HomePage extends StatefulWidget {
  const HomePage({super.key});

  @override
  State<HomePage> createState() => HomePageState();
}

class HomePageState extends State<HomePage> {
  static const loadingTag = "##loading##"; //表尾标记
  var words = <String>[loadingTag];

  @override
  void initState() {
    super.initState();
    _retrieveData();
  }
    // 加载数据
  void _retrieveData() {
    Future.delayed(const Duration(seconds: 2)).then((e) {
      setState(() {
        //重新构建列表
        words.insertAll(
          words.length - 1,
          //每次生成20个单词
          generateWordPairs().take(20).map((e) => e.asPascalCase).toList(),
        );
      });
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
        appBar: AppBar(
          title: const Text('Home Page'),
          backgroundColor: Theme.of(context).colorScheme.inversePrimary,
        ),
        body: ListView.separated(
          itemCount: words.length,
          itemBuilder: (context, index) {
            // 如果到了表尾
            if (words[index] == loadingTag) {
              // 不足100条，继续获取数据
              if (words.length - 1 < 100) {
                // 获取数据
                _retrieveData();
                // 加载时显示loading
                return Container(
                  padding: const EdgeInsets.all(16.0),
                  alignment: Alignment.center,
                  child: const SizedBox(
                    width: 24.0,
                    height: 24.0,
                    child: CircularProgressIndicator(strokeWidth: 2.0),
                  ),
                );
              } else {
                // 已经加载了100条数据，不再获取数据
                return Container(
                  alignment: Alignment.center,
                  padding: const EdgeInsets.all(16.0),
                  child: const Text(
                    "没有更多了",
                    style: TextStyle(color: Colors.grey),
                  ),
                );
              }
            }
            // 显示单词列表项
            return ListTile(title: Text(words[index]));
          },
          separatorBuilder: (context, index) => const Divider(height: .0),
        ));
  }
}
```

