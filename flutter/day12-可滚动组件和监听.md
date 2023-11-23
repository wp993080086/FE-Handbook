# 📚 目录

1. [简介](#1-简介)
2. [可滚动组件](#2-可滚动组件)
    1. [SingleChildScrollView](#2-1-singlechildscrollview)
    2. [ListView](#2-2-listview)
        1. [separated分割线](#2-2-1-separated分割线)
        2. [无限加载列表](#2-2-2-无限加载列表)
        3. [带标题列表](#2-2-3-带标题列表)
3. [滚动监听和控制](#3-滚动监听和控制)
    1. [ScrollController滚动监听](#3-1-scrollcontroller滚动监听)
    2. [NotificationListener滚动监听](#3-2-notificationlistener滚动监听)
4. [AnimatedList动画列表](#4-animatedlist动画列表)
5. [滚动网格布局GridView](#5-滚动网格布局gridview)
    1. [横轴子元素为固定数量](#5-1-横轴子元素为固定数量)
    2. [横轴子元素为固定最大长度](#5-2-横轴子元素为固定最大长度)
    3. [动态创建](#5-3-动态创建)
6. [PageView页面切换和缓存](#6-pageview页面切换和缓存)
    1. [PageState缓存](#6-1-pagestate缓存)
7. [TabBarView](#7-tabbarview)
8. [CustomScrollView和Slivers](#8-customscrollview和slivers)
    1. [常用的Slivers](#8-1-常用的slivers)
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

### 2-2-1. separated分割线

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

### 2-2-2. 无限加载列表

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

## 2-2-3. 带标题列表

```dart
Widget build(BuildContext context) {
    return Scaffold(
        appBar: AppBar(
          title: const Text('Home Page'),
          backgroundColor: Theme.of(context).colorScheme.inversePrimary,
        ),
        body: Column(
          children: [
            // 标题
            const ListTile(
              title: Text('标题')
            ),
            // 用Expanded 占满剩余空间
            Expanded(
              child: ListView.builder(itemBuilder: (BuildContext context, int index) {
                return ListTile(
                  title: Text('行：$index'),
                );
              }),
            )
          ],
        ));
  }
```

# 3. 滚动监听和控制

可以用ScrollController来控制可滚动组件的滚动位置。而，可滚动组件在滚动时会发送ScrollNotification类型的通知，ScrollBar正是通过监听滚动通知来实现的。

## 3-1. ScrollController滚动监听

ScrollController间接继承自Listenable，我们可以根据ScrollController来监听滚动事件。如下例子：

- 滚动到1000的时候，出现回到顶部的按钮，点击即可回到顶部

```dart
import 'package:flutter/material.dart';

class HomePage extends StatefulWidget {
  const HomePage({super.key});

  @override
  State<HomePage> createState() => HomePageState();
}

class HomePageState extends State<HomePage> {
  /// 监听控制器
  ScrollController myController = ScrollController();

  /// 是否显示回到顶部按钮
  bool toTop = false;

  @override
  void initState() {
    super.initState();

    /// 开启监听
    myController.addListener(() {
      if (myController.offset < 1000 && toTop) {
        setState(() {
          toTop = false;
        });
      } else if (myController.offset >= 1000 && toTop == false) {
        setState(() {
          toTop = true;
        });
      }
    });
  }

  @override
  void dispose() {
    /// 注销监听
    myController.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Home Page'),
        backgroundColor: Theme.of(context).colorScheme.inversePrimary,
      ),
      body: Scrollbar(
          child: ListView.builder(
        /// 固定列数
        itemCount: 100,

        /// 固定列高
        itemExtent: 50.0,
        controller: myController,

        /// 渲染行
        itemBuilder: (BuildContext context, int index) {
          return ListTile(title: Text('行：$index'));
        },
      )),

      /// 返回顶部按钮
      floatingActionButton: !toTop
          ? null
          : FloatingActionButton(
              child: const Icon(Icons.arrow_upward),
              onPressed: () {
                /// 动画返回
                myController.animateTo(0.0,
                    duration: const Duration(microseconds: 200),
                    curve: Curves.ease);
              },
            ),
    );
  }
}
```

## 3-2. NotificationListener滚动监听

和ScrollController不同，NotificationListener可以在可滚动组件到widget树根之间任意位置监听，而ScrollController只能和具体的可滚动组件关联后才可以。并且，收到滚动事件时，NotificationListener通知中会携带当前滚动位置和ViewPort的一些信息，而ScrollController只能获取当前滚动位置。如下例子：

- 中间有个文字会显示滚动的进度是多少

```dart
import 'package:flutter/material.dart';

class HomePage extends StatefulWidget {
  const HomePage({super.key});

  @override
  State<HomePage> createState() => HomePageState();
}

class HomePageState extends State<HomePage> {
  /// 进度文案
  String myProgress = '0%';

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Home Page'),
        backgroundColor: Theme.of(context).colorScheme.inversePrimary,
      ),
      body: NotificationListener(
        onNotification: (ScrollNotification  notification) {
          double progress = notification.metrics.pixels / notification.metrics.maxScrollExtent;
          setState(() {
            myProgress = '${(progress * 100).toInt()}%';
          });
          return false;
        },
        child: Stack(
          alignment: Alignment.center,
          children: [
            ListView.builder(
              itemCount: 100,
              itemExtent: 50.0,
              itemBuilder: (context, index) => ListTile(title: Text('第$index行'),),
            ),
            CircleAvatar(
              radius: 30.0,
              backgroundColor: Colors.black12,
              child: Text(myProgress)
            )
          ],
        ),
      )
    );
  }
}
```

# 4. AnimatedList动画列表

AnimatedList 和 ListView 的功能大体相似，不同的是， AnimatedList 可以在列表中插入或删除节点时执行一个动画，在需要添加或删除列表项的场景中会提高用户体验。如下例子：

```dart
import 'package:flutter/material.dart';

class HomePage extends StatefulWidget {
  const HomePage({super.key});

  @override
  State<HomePage> createState() => HomePageState();
}

class HomePageState extends State<HomePage> {
  /// 数据列表
  var dataList = <String>[];
  /// 初始个数
  int counter = 6;
  /// Key
  final myKey = GlobalKey<AnimatedListState>();

  @override
  void initState() {
    super.initState();
    /// 初始化数据
    for (var i = 0; i < counter; i++) {
      dataList.add('标题：${i + 1}');
    }
  }

  Widget buildItem(context, index) {
    String str = dataList[index];
    return ListTile(
      key: ValueKey(str),
      title: Text(str),
      trailing: IconButton(
        icon: const Icon(Icons.delete),
        onPressed: () {
          /// 删除的逻辑和动画
          setState(() {
            myKey.currentState!.removeItem(index, (context, animation) {
              var item = buildItem(context, index);
              dataList.removeAt(index);
              return FadeTransition(
                opacity: CurvedAnimation(
                    parent: animation, curve: const Interval(0.5, 1.0)),
                child: SizeTransition(
                  sizeFactor: animation,
                  axisAlignment: 0.0,
                  child: item,
                ),
              );
            }, duration: const Duration(milliseconds: 200));
          });
        },
      ),
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
        appBar: AppBar(
          title: const Text('Home Page'),
          backgroundColor: Theme.of(context).colorScheme.inversePrimary,
        ),
        body: Stack(
          alignment: Alignment.center,
          children: [
            AnimatedList(
              key: myKey,
              initialItemCount: dataList.length,
              itemBuilder: (context, index, animation) {
                return FadeTransition(
                    opacity: animation, child: buildItem(context, index));
              },
            ),
            Positioned(
              left: 0,
              right: 0,
              bottom: 30.0,
              child: FloatingActionButton(
                /// 添加项
                onPressed: () {
                  dataList.add('标题：${++counter}');
                  myKey.currentState!.insertItem(dataList.length - 1);
                },
                child: const Icon(Icons.add),
              ),
            )
          ],
        ));
  }
}
```

# 5. 滚动网格布局GridView

GridView是ListView的升级版，可以构建二维网格列表，可以指定子组件的布局方式，如网格布局、列表布局等。他的参数和ListView是一致的，唯一区别是多了个gridDelegate参数，控制GridView子组件的排列。它提供了SliverGridDelegateWithFixedCrossAxisCount和SliverGridDelegateWithMaxCrossAxisExtent两个子类。

## 5-1. 横轴子元素为固定数量

使用SliverGridDelegateWithFixedCrossAxisCount，可以实现一个横轴为固定数量子元素的layout布局。子元素的大小（最大显示空间）是通过crossAxisCount和childAspectRatio两个参数共同决定的。

```dart
/// 写法一
GridView(
  gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(

      /// 横轴子元素的数量
      crossAxisCount: 3,

      /// 主轴方向的间距
      mainAxisSpacing: 10.0,

      /// 横轴方向子元素的间距
      crossAxisSpacing: 20.0,

      /// 子元素在横轴长度和主轴长度的比例
      childAspectRatio: 1.0),

  /// 元素列表
  children: const [
    Icon(Icons.ac_unit),
    Icon(Icons.airport_shuttle),
    Icon(Icons.all_inclusive),
    Icon(Icons.beach_access),
    Icon(Icons.cake),
    Icon(Icons.free_breakfast)
  ]
)
/// 写法二
GridView.count( 
  crossAxisCount: 3,
  childAspectRatio: 1.0,
  children: const [
    Icon(Icons.ac_unit),
    Icon(Icons.airport_shuttle),
    Icon(Icons.all_inclusive),
    Icon(Icons.beach_access),
    Icon(Icons.cake),
    Icon(Icons.free_breakfast),
  ],
)
```

## 5-2. 横轴子元素为固定最大长度

使用SliverGridDelegateWithMaxCrossAxisExtent，可以实现一个横轴子元素为固定最大长度的layout布局，子元素的大小（最大显示空间）是通过maxCrossAxisExtent参数决定的。

```dart
/// 写法一
GridView(
  gridDelegate: const SliverGridDelegateWithMaxCrossAxisExtent(
    /// 子元素在横轴上的最大长度
    maxCrossAxisExtent: 100.0,

    /// 子元素在横轴长度和主轴长度的比例
    childAspectRatio: 1.0,

    /// 主轴方向的间距
    mainAxisSpacing: 10.0,

    /// 横轴方向子元素的间距
    crossAxisSpacing: 20.0,
  ),
  children: const [
    Icon(Icons.ac_unit),
    Icon(Icons.airport_shuttle),
    Icon(Icons.all_inclusive),
    Icon(Icons.beach_access),
    Icon(Icons.cake),
    Icon(Icons.free_breakfast)
  ]
)

/// 写法二
GridView.extent(
  maxCrossAxisExtent: 120.0,
  childAspectRatio: 1.0,
  children: const [
    Icon(Icons.ac_unit),
    Icon(Icons.airport_shuttle),
    Icon(Icons.all_inclusive),
    Icon(Icons.beach_access),
    Icon(Icons.cake),
    Icon(Icons.free_breakfast),
  ],
)
```

## 5-3. 动态创建

当子widget比较多时，我们可以通过GridView.builder来动态创建子widget。如下例子：

```dart
import 'package:flutter/material.dart';

class HomePage extends StatefulWidget {
  const HomePage({super.key});

  @override
  State<HomePage> createState() => HomePageState();
}

class HomePageState extends State<HomePage> {
  /// Icon列表
  List<IconData> myIcons = [];
  
  @override
  void initState() {
    super.initState();
    /// 初始化数据
    handleSyncAddIcon();
  }

  /// 模拟异步获取数据
  void handleSyncAddIcon() {
    Future.delayed(const Duration(milliseconds: 2000)).then((e) {
      setState(() {
        myIcons.addAll([
          Icons.ac_unit,
          Icons.airport_shuttle,
          Icons.all_inclusive,
          Icons.beach_access,
          Icons.cake,
          Icons.free_breakfast,
        ]);
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
        body: GridView.builder(
          gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
            /// 每行三列
            crossAxisCount: 3,
            /// 显示区域宽高相等
            childAspectRatio: 1.0,
          ),
          itemCount: myIcons.length,
          itemBuilder: (context, index) {
            /// 如果显示到最后一个 并且Icon总数小于200时 继续获取数据
            if (index == myIcons.length - 1 && myIcons.length < 200) {
              handleSyncAddIcon();
            }
            return Icon(myIcons[index]);
          },
        ));
  }
}
```

# 6. PageView页面切换和缓存

如果要实现页面切换和 Tab 布局，Tab 换页效果、图片轮动以及抖音上下滑页切换视频功能等等，我们可以使用 PageView 组件。如下全屏切换的例子：

```dart
import 'package:flutter/material.dart';

class HomePage extends StatefulWidget {
  const HomePage({super.key});

  @override
  State<HomePage> createState() => HomePageState();
}

class PageItem extends StatefulWidget {
  const PageItem({
    Key? key,
    required this.text
  }) : super(key: key);

  final String text;

  @override
  PageItemState createState() => PageItemState();
}

class PageItemState extends State<PageItem> {
  @override
  Widget build(BuildContext context) {
    debugPrint('触发${widget.text}');
    return Center(child: Text("第${widget.text}页", textScaleFactor: 5));
  }
}

class HomePageState extends State<HomePage> {
  var children = <Widget>[];

  @override
  void initState() {
    super.initState();
    for (int i = 0; i < 6; ++i) {
      children.add(PageItem(text: '$i'));
    }
  }

  @override
  Widget build(BuildContext context) {
    debugPrint('触发');
    return Scaffold(
        appBar: AppBar(
          title: const Text('Home Page'),
          backgroundColor: Theme.of(context).colorScheme.inversePrimary,
        ),
        body: PageView(
          /// 每次滑动是否强制切换整个页面
          pageSnapping: true,
          padEnds: true,
          /// 滑动方向
          scrollDirection: Axis.vertical,
          /// 子元素
          children: children,
        )
    );
  }
}
```

## 6-1. PageState缓存

只需要让 PageState 混入这个 AutomaticKeepAliveClientMixin ，简单改造就可以实现缓存。

```dart
import 'package:flutter/material.dart';

class HomePage extends StatefulWidget {
  const HomePage({super.key});

  @override
  State<HomePage> createState() => HomePageState();
}

class PageItem extends StatefulWidget {
  const PageItem({
    Key? key,
    required this.text
  }) : super(key: key);

  final String text;

  @override
  PageItemState createState() => PageItemState();
}

class PageItemState extends State<PageItem> with AutomaticKeepAliveClientMixin {
  @override
  Widget build(BuildContext context) {
    super.build(context);

    debugPrint('触发${widget.text}');
    return Center(child: Text("第${widget.text}页", textScaleFactor: 5));
  }

  /// 是否需要缓存
  @override
  bool get wantKeepAlive => true;
}

class HomePageState extends State<HomePage> {
  var children = <Widget>[];

  @override
  void initState() {
    super.initState();
    for (int i = 0; i < 6; ++i) {
      children.add(PageItem(text: '$i'));
    }
  }

  @override
  Widget build(BuildContext context) {
    debugPrint('触发');
    return Scaffold(
        appBar: AppBar(
          title: const Text('Home Page'),
          backgroundColor: Theme.of(context).colorScheme.inversePrimary,
        ),
        body: PageView(
          /// 每次滑动是否强制切换整个页面
          pageSnapping: true,
          padEnds: true,
          /// 滑动方向
          scrollDirection: Axis.vertical,
          /// 子元素
          children: children,
        )
    );
  }
}
```

# 7. TabBarView

abBarView 封装了 PageView，而abController 用于监听和控制 TabBarView 的页面切换，通常和 TabBar 联动。如果没有指定，则会在组件树中向上查找并使用最近的一个 DefaultTabController。如下例子：

```dart
import 'package:flutter/material.dart';

class HomePage extends StatefulWidget {
  const HomePage({super.key});

  @override
  State<HomePage> createState() => HomePageState();
}

class PageItem extends StatefulWidget {
  const PageItem({Key? key, required this.text}) : super(key: key);

  final String text;

  @override
  PageItemState createState() => PageItemState();
}

class PageItemState extends State<PageItem> with AutomaticKeepAliveClientMixin {
  @override
  Widget build(BuildContext context) {
    super.build(context);

    debugPrint('触发${widget.text}');
    return Center(child: Text("${widget.text}", textScaleFactor: 5));
  }

  /// 是否需要缓存
  @override
  bool get wantKeepAlive => true;
}

class HomePageState extends State<HomePage> {
  var children = <Widget>[];

  List tabs = ["新闻", "历史", "图片"];

  @override
  void initState() {
    super.initState();
    for (int i = 0; i < tabs.length; ++i) {
      children.add(PageItem(text: '${tabs[i]}'));
    }
  }

  @override
  Widget build(BuildContext context) {
    return DefaultTabController(
      length: children.length,
      child: Scaffold(
        appBar: AppBar(
          title: const Text("App Name"),
          bottom: TabBar(
            tabs: tabs.map((e) => Tab(text: e)).toList(),
          ),
        ),
        body: TabBarView(
          /// tab子页面
          children: children,
        ),
      ),
    );
  }
}
```

# 8. CustomScrollView和Slivers

CustomScrollView的 slivers 参数接受一个 Sliver 数组，这样就可以在一个页面中，同时包含多个可滚动组件，且使它们的滑动效果能统一起来。如下例子：

```dart
import 'package:flutter/material.dart';

class HomePage extends StatefulWidget {
  const HomePage({super.key});

  @override
  State<HomePage> createState() => HomePageState();
}

class HomePageState extends State<HomePage> {

  var listView = SliverFixedExtentList(
    itemExtent: 56,
      delegate: SliverChildBuilderDelegate((_, index) => ListTile(
        title: Text('$index')),
        childCount: 10,
      )
  );

  @override
  Widget build(BuildContext context) {
    return Scaffold(
        appBar: AppBar(
          title: const Text('Home Page'),
          backgroundColor: Theme.of(context).colorScheme.inversePrimary,
        ),
        body: Center(
          child: CustomScrollView(
            slivers: [
              listView,
              listView,
            ],
          ),
        )
    );
  }
}
```

## 8-1. 常用的Slivers

- 可滚动组件对应的Sliver

|名称|功能|对应组件|
|--|--|--|
|SliverList|列表|ListView|
|SliverFixedExtentList|高度固定的列表|ListView，指定itemExtent后|
|SliverAnimatedList|添加/删除列表项可以执行动画|AnimatedList|
|SliverGrid|网格|GridView|
|SliverPrototypeExtentList|根据原型生成高度固定的列表|ListView，指定prototypeItem后|
|SliverFillViewport|包含多个子组件，每个都可以填满屏幕|PageView|

- 子组件必须是Sliver

|名称|对应RenderBox容器|
|--|--|
|SliverPadding|Padding|
|SliverVisibility、SliverOpacity|Visibility、Opacity|
|SliverFadeTransition|FadeTransition|
|SliverLayoutBuilder|LayoutBuilder|

- 其他常用的Sliver

|名称|功能|
|--|--|
|SliverAppBar|对应 AppBar，主要是为了在 CustomScrollView 中使用|
|SliverToBoxAdapter|一个适配器，可以将 RenderBox 适配为 Sliver|
|SliverPersistentHeader|滑动到顶部时可以固定住|