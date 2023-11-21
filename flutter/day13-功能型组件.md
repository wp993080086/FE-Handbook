# 📚 目录

1. [导航返回拦截](#1-导航返回拦截)
2. [InheritedWidget数据共享](#2-inheritedwidget数据共享)
3. [跨组件状态共享](#3-跨组件状态共享)
    1. [事件总线EventBus](#3-1-事件总线eventbus)
    2. [依赖注入Provider](#3-2-依赖注入provider)
4. [颜色和主题](#4-颜色和主题)
    1. [颜色字符串转成color对象](#4-1-颜色字符串转成Color对象)
    2. [颜色亮度](#4-2-颜色亮度)
    3. [MaterialColor类](#4-3-materialcolor类)
    4. [主体](#4-4-主题)
5. [异步UI更新](#5-异步ui更新)
    1. [FutureBuilder](#5-1-futurebuilder)
    2. [StreamBuilder](#5-2-streambuilder)
6. [对话框](#6-对话框)
---

# 1. 导航返回拦截

为了避免用户误触返回按钮而导致 App 退出，在很多 App 中都拦截了用户点击返回键的按钮，然后进行一些防误触判断，比如当用户在某一个时间段内点击两次时，才会认为用户是要退出。Flutter中可以通过WillPopScope来实现返回按钮拦截。

- 示例如下：

```dart
import 'package:flutter/material.dart';

class HomePage extends StatefulWidget {
  const HomePage({super.key});

  @override
  State<HomePage> createState() => HomePageState();
}

class HomePageState extends State<HomePage> {
  /// 时间
  DateTime? times;

  @override
  Widget build(BuildContext context) {
    return WillPopScope(
      onWillPop: () async {
        const oneSecond = Duration(seconds: 1);
        if (times == null || DateTime.now().difference(times!) > oneSecond) {
          // 两次点击间隔超过1秒则重新计时
          times = DateTime.now();
          return false;
        }
        return true;
      },
      child: Scaffold(
        appBar: AppBar(
          title: const Text('Flutter Home'),
        ),
        body: Center(
          child: TextButton(
            child: const Text('点击我'),
            onPressed: () {},
          ),
        ),
      ),
    );
  }
}
```

# 2. InheritedWidget数据共享

InheritedWidget提供了一种在 widget 树中从上到下共享数据的方式，比如我们在应用的根 widget 中通过InheritedWidget共享了一个数据，那么我们便可以在任意子widget 中来获取该共享的数据。如Flutter SDK中正是通过 InheritedWidget 来共享应用主题和Locale信息的。如下例子：

- 计数器使用共享数据

```dart
import 'package:flutter/material.dart';

class ShareDataWidget extends InheritedWidget {
  const ShareDataWidget({super.key, required this.data, required Widget child})
      : super(child: child);

  final int data;

  static ShareDataWidget? of(BuildContext context) {
    return context.dependOnInheritedWidgetOfExactType<ShareDataWidget>();
  }

  @override
  bool updateShouldNotify(ShareDataWidget old) {
    return old.data != data;
  }
}

class MyTextWidget extends StatefulWidget {
  const MyTextWidget({super.key});

  @override
  MyTextWidgetState createState() => MyTextWidgetState();
}

class MyTextWidgetState extends State<MyTextWidget> {
  @override
  Widget build(BuildContext context) {
    /// 使用InheritedWidget中的共享数据
    var txt = ShareDataWidget.of(context)!.data.toString();
    return Text(txt);
  }

  /// 父或祖先widget中的InheritedWidget改变（updateShouldNotify返回true）时会被调用。如果build中没有依赖InheritedWidget，则此回调不会被调用。
  @override
  void didChangeDependencies() {
    super.didChangeDependencies();
    debugPrint("count修改了");
  }
}

class HomePage extends StatefulWidget {
  const HomePage({super.key});

  @override
  State<HomePage> createState() => HomePageState();
}

class HomePageState extends State<HomePage> {
  int count = 0;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Flutter Home'),
      ),
      body: Center(
        child: ShareDataWidget(
          /// 使用ShareDataWidget
          data: count,
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: <Widget>[
              const Padding(
                padding: EdgeInsets.only(bottom: 20.0),
                /// 子widget中依赖ShareDataWidget
                child: MyTextWidget(),
              ),
              ElevatedButton(
                child: const Text("点击增加"),
                /// 每点击一次，将count自增，然后重新build,ShareDataWidget的data将被更新
                onPressed: () => setState(() => ++count),
              )
            ],
          ),
        ),
      ),
    );
  }
}
```

## 2-1. 只引用数据不调用didChangeDependencies

在上面的例子中，如果我们只想在MyTextWidgetState中引用ShareDataWidget数据，但却不希望在ShareDataWidget发生变化时调用MyTextWidgetState的didChangeDependencies()方法，只需要将ShareDataWidget.of()的实现改一下即可。唯一的改动就是获取ShareDataWidget对象的方式，把`dependOnInheritedWidgetOfExactType()`方法换成了`context.getElementForInheritedWidgetOfExactType<ShareDataWidget>().widget`。他们的区别就是前者会注册依赖关系，而后者不会。

```dart
class ShareDataWidget extends InheritedWidget {
  const ShareDataWidget({super.key, required this.data, required Widget child})
      : super(child: child);

  final int data;

  static ShareDataWidget? of(BuildContext context) {
    // return context.dependOnInheritedWidgetOfExactType<ShareDataWidget>();
    return context.getElementForInheritedWidgetOfExactType<ShareDataWidget>()!.widget as ShareDataWidget;
  }

  @override
  bool updateShouldNotify(ShareDataWidget old) {
    return old.data != data;
  }
}
```

# 3. 跨组件状态共享

状态管理是一个永恒的话题。如果状态是组件私有的，则应该由组件自己管理；如果状态要跨组件共享，则该状态应该由各个组件共同的父元素来管理。常用的有事件总线EventBus（发布-订阅）和依赖注入Provider（观察者模式）。

## 3-1. 事件总线EventBus

```dart
/// 定义事件
enum Event{
  login
}

/// 发布事件
bus.emit(Event.login);

/// 
void onLoginChanged(e){
  //登录状态变化处理逻辑
}

@override
void initState() {
  /// 订阅登录状态改变事件
  bus.on(Event.login, onLoginChanged);
  super.initState();
}

@override
void dispose() {
  /// 取消订阅
  bus.off(Event.login, onLoginChanged);
  super.dispose();
}
```

## 3-2. 依赖注入Provider

现在Flutter社区已经有很多专门用于状态管理的包了，在此列出几个相对评分比较高的：

|名称|描述|
|-|-|
|Provider & Scoped Model|两个包都是基于InheritedWidget的，原理相似|
|Redux|React生态链中Redux包的Flutter实现|
|MobX|React生态链中MobX包的Flutter实现|
|BLoC|BLoC模式的Flutter实现|

# 4. 颜色和主题

Flutter里有一个Colors类，里面定义了100多个颜色常量，颜色都是以一个int值保存。而Theme组件可以为Material APP定义主题数据。

## 4-1. 颜色字符串转成Color对象

- 颜色固定

```dart
/// 直接使用整数值
Color(0xffdc380d)
```

- 字符串变量

```dart
var c = "dc380d";

/// 通过位运算符将Alpha设置为FF
Color(int.parse(c,radix:16)|0xFF000000)

/// 通过方法将Alpha设置为FF
Color(int.parse(c,radix:16)).withAlpha(255)
```

## 4-2. 颜色亮度

假如要实现一个背景颜色和Title可以自定义的导航栏，并且背景色为深色时我们应该让Title显示为浅色；背景色为浅色时，Title 显示为深色。要实现这个功能，我们就需要来计算背景色的亮度，然后动态来确定Title的颜色。Color 类中提供了一个computeLuminance()方法，它可以返回一个[0-1]的一个值，数字越大颜色就越浅，我们可以根据它来动态确定Title的颜色。

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
      body: const Center(
        child: Column(
          children: [
            NavBar(color: Colors.blue, title: '标题1'),
            NavBar(color: Colors.white, title: '标题2')
          ],
        ),
      ),
    );
  }
}

/// 定义组件
class NavBar extends StatelessWidget {
  final String title;
  final Color color;

  const NavBar({
    super.key,
    required this.color,
    required this.title,
  });

  @override
  Widget build(BuildContext context) {
    return Container(
        constraints: const BoxConstraints(
          minHeight: 52,
          minWidth: double.infinity,
        ),
        decoration: BoxDecoration(
          color: color,
          boxShadow: const [
            // 阴影
            BoxShadow(
              color: Colors.black26,
              offset: Offset(0, 3),
              blurRadius: 3,
            ),
          ],
        ),
        alignment: Alignment.center,
        child: Text(
          title,
          style: TextStyle(
            fontWeight: FontWeight.bold,
            // 根据背景色亮度来确定Title颜色
            color: color.computeLuminance() < 0.5 ? Colors.white : Colors.black,
          ),
        ));
  }
}
```

## 4-3. MaterialColor类

MaterialColor是实现Material Design中的颜色的类，它包含一种颜色的10个级别的渐变色。它通过"[]"运算符的索引值来代表颜色的深度，有效的索引有：50，100，200，…，900，数字越大，颜色越深。默认值是500。使用方式为Colors.blue.shade50。

- 例子：

```dart
class HomePageState extends State<HomePage> {

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Flutter Home'),
      ),
      body: Center(
        child: Column(
          children: [
            NavBar(color: Colors.blue.shade50, title: '标题1'),
            NavBar(color: Colors.blue.shade100, title: '标题2'),
            NavBar(color: Colors.blue.shade200, title: '标题2'),
            NavBar(color: Colors.blue.shade300, title: '标题2'),
            NavBar(color: Colors.blue.shade400, title: '标题2'),
            NavBar(color: Colors.blue.shade500, title: '标题2'),
            NavBar(color: Colors.blue.shade600, title: '标题2'),
            NavBar(color: Colors.blue.shade700, title: '标题2'),
            NavBar(color: Colors.blue.shade800, title: '标题2'),
            NavBar(color: Colors.blue.shade900, title: '标题2')
          ],
        ),
      ),
    );
  }
}
```

## 4-4. 主题

ThemeData用于保存是Material 组件库的主题数据，Material组件需要遵守相应的设计规范，而这些规范可自定义部分都定义在ThemeData中了，所以我们可以通过ThemeData来自定义应用主题。在子组件中，我们可以通过Theme.of方法来获取当前的ThemeData。（有些是不能自定义的，比如导航栏高度）。

- 定义：

```dart
ThemeData({
  Brightness? brightness, // 深色还是浅色
  MaterialColor? primarySwatch, // 主题颜色样本，见下面介绍
  Color? primaryColor, // 主色，决定导航栏颜色
  Color? cardColor, // 卡片颜色
  Color? dividerColor, // 分割线颜色
  ButtonThemeData buttonTheme, // 按钮主题
  Color dialogBackgroundColor,// 对话框背景颜色
  String fontFamily, // 文字字体
  TextTheme textTheme,// 字体主题，包括标题、body等文字样式
  IconThemeData iconTheme, // Icon的默认样式
  TargetPlatform platform, // 指定平台，应用特定平台控件风格
  ColorScheme? colorScheme,
  /// ......
})
```

下面是一个单个页面换肤的例子，如果想要对整个应用换肤，则可以去修改MaterialApp的theme属性，例子如下：

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
  var myThemeColor = Colors.teal;

  @override
  Widget build(BuildContext context) {
    ThemeData themeData = Theme.of(context);

    return Theme(
        data: ThemeData(
          // 用于导航栏、FloatingActionButton的背景色等
          primarySwatch: myThemeColor,
          // 用于Icon颜色
          iconTheme: IconThemeData(color: myThemeColor),
        ),
        child: Scaffold(
            appBar: AppBar(
              title: const Text('Flutter Home'),
            ),
            body: Center(
              child: Column(
                children: [
                  const Row(
                    mainAxisAlignment: MainAxisAlignment.center,
                    children: [
                      Icon(Icons.favorite),
                      Text(" 颜色跟随主题"),
                    ],
                  ),
                  Theme(
                    data: themeData.copyWith(
                      iconTheme:
                          themeData.iconTheme.copyWith(color: Colors.black),
                    ),
                    child: const Row(
                        mainAxisAlignment: MainAxisAlignment.center,
                        children: <Widget>[
                          Icon(Icons.favorite),
                          Text(" 颜色固定黑色")
                        ]),
                  )
                ],
              ),
            ),
            floatingActionButton: FloatingActionButton(
              // 切换主题
                onPressed: () {
                  setState(() => myThemeColor = myThemeColor == Colors.teal
                      ? Colors.blue
                      : Colors.teal);
                },
                child: const Icon(Icons.palette))));
  }
}
```

# 5. 异步UI更新

很多时候我们会依赖一些异步数据来动态更新UI，比如在打开一个页面时我们需要先从互联网上获取数据，在获取数据的过程中我们显示一个加载框，等获取到数据时我们再渲染页面；又比如我们想展示文件流、互联网数据接收流的进度。Flutter专门提供了FutureBuilder和StreamBuilder两个组件来快速实现这种功能。

## 5-1. FutureBuilder

FutureBuilder是Flutter中专门用于异步UI更新的组件，它会依赖一个Future，它会根据所依赖的Future的状态来动态构建自身。

|属性|描述|
|---|---|
|future|FutureBuilder依赖的Future，通常是一个异步耗时任务|
|initialData|初始数据，用户设置默认数据|
|builder|Widget构建器，该构建器会在Future执行的不同阶段被多次调用，第一个参数是context|
|builder.snapshot|builder的参数，包含当前异步任务的状态信息及结果信息|

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

  Future<String> mockNetworkData() async {
    return Future.delayed(const Duration(seconds: 2), () => "我是从互联网上获取的数据");
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
        appBar: AppBar(
          title: const Text('Flutter Home'),
        ),
        body: Center(
          child: FutureBuilder(
            future: mockNetworkData(),
            builder: (BuildContext context, AsyncSnapshot snapshot) {
              if (snapshot.connectionState == ConnectionState.done) {
                if (snapshot.hasError) {
                  // 请求失败，显示错误
                  return Text("Error: ${snapshot.error}");
                } else {
                  // 请求成功，显示数据
                  return Text("Contents: ${snapshot.data}");
                }
              } else {
                // 请求未结束，显示loading
                return const CircularProgressIndicator();
              }
            },
          ),
        ));
  }
}
```

## 5-2. StreamBuilder

StreamBuilder是用于配合Stream来展示流上事件（数据）变化的UI组件。

例子如下：

```dart
import 'package:flutter/material.dart';
import 'package:english_words/english_words.dart';

/// 定义
class HomePage extends StatefulWidget {
  const HomePage({super.key});

  @override
  State<HomePage> createState() => HomePageState();
}

/// 实现
class HomePageState extends State<HomePage> {
  /// 任务流
  Stream<int> myCounter() {
    return Stream.periodic(const Duration(seconds: 1), (i) {
      return i;
    });
  }
  /// 状态文案
  String str = '';

  @override
  Widget build(BuildContext context) {
    return Scaffold(
        appBar: AppBar(
          title: const Text('Flutter Home'),
        ),
        body: Center(
          child: StreamBuilder(
            stream: myCounter(),
            builder: (BuildContext context, AsyncSnapshot<int> snapshot) {
              if (snapshot.hasError) {
                str = snapshot.error.toString();
              }
              switch (snapshot.connectionState) {
                case ConnectionState.none:
                  str = '没有Stream';
                case ConnectionState.waiting:
                  str = '等待数据';
                case ConnectionState.active:
                  str = snapshot.data.toString();
                case ConnectionState.done:
                  str = '已关闭';
              }
              return Text('状态：$str');
            },
          ),
        ),
        floatingActionButton: FloatingActionButton(
            onPressed: () {}, child: const Icon(Icons.palette)));
  }
}
```

# 6. 对话框

对话框本质上也是UI布局，通常一个对话框会包含标题、内容，以及一些操作按钮，为此，Material库中提供了一些现成的对话框组件来用于快速的构建出一个完整的对话框。下面写一个AlertDialog做例子。

构造函数如下：

```dart
const AlertDialog({
  Key? key,
  this.title, // 对话框标题组件
  this.titlePadding, // 标题填充
  this.titleTextStyle, // 标题文本样式
  this.content, // 对话框内容组件
  this.contentPadding = const EdgeInsets.fromLTRB(24.0, 20.0, 24.0, 24.0), // 内容的填充
  this.contentTextStyle,// 内容文本样式
  this.actions, // 对话框操作按钮组
  this.backgroundColor, // 对话框背景色
  this.elevation,// 对话框的阴影
  this.semanticLabel, // 对话框语义化标签（用于读屏软件）
  this.shape, // 对话框外形
})
```

使用例子如下：

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
  /// 定义对话框
  Future<bool?> showDeleteConfirmDialog() {
    return showDialog<bool>(
      context: context,
      builder: (context) {
        return AlertDialog(
          title: const Text("提示"),
          content: const Text("您确定要删除当前文件吗?"),
          actions: <Widget>[
            TextButton(
              child: const Text("取消"),
              onPressed: () => Navigator.of(context).pop(), // 关闭对话框
            ),
            TextButton(
              child: const Text("删除"),
              onPressed: () {
                //关闭对话框并返回true
                Navigator.of(context).pop(true);
              },
            ),
          ],
        );
      },
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
        appBar: AppBar(
          title: const Text('Flutter Home'),
        ),
        body: const Center(
          child: Text('点击此处'),
        ),
        floatingActionButton: FloatingActionButton(
            onPressed: () async {
              bool? deleteDialog = await showDeleteConfirmDialog();
              if (deleteDialog == null) {
                debugPrint("取消");
              } else {
                debugPrint("确认");
              }
            }, child: const Icon(Icons.palette)));
  }
}
```