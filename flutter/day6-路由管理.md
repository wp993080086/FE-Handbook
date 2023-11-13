# 📚 目录

1. [简单示例](#1-简单示例)
2. [MaterialPageRoute](#2-materialpageroute)
3. [Navigator](#3-navigator)
4. [路由传值](#4-路由传值)
5. [命名路由](#5-命名路由)
5. [路由守卫onGenerateRoute](#5-路由守卫onGenerateRoute)
---

# 1. 简单示例

```dart
// 定义abuot组件
class AboutPage extends StatefulWidget {
  const AboutPage({super.key});
  @override
  State<AboutPage> createState() => _AboutPageState();
}

class _AboutPageState extends State<AboutPage> {
  final _title = 'About Page';
  @override
  Widget build(BuildContext context) {
    return Scaffold(
        appBar: AppBar(title: Text(_title)),
        body: const Center(child: Text('这是AboutPage')));
  }
}

// 在build中
Widget build(BuildContext context) {
  return Scaffold(
    appBar: AppBar(
      backgroundColor: Theme.of(context).colorScheme.inversePrimary,
      title: Text(widget.title),
    ),
    body: Center(
      child: Column(
        mainAxisAlignment: MainAxisAlignment.center,
        children: <Widget>[
          // 点击跳转
          TextButton(
              onPressed: () {
                Navigator.push(context, MaterialPageRoute(builder: (context) {
                  return const AboutPage();
                }));
              },
              child: const Text('open'))
        ],
      ),
    )
  );
}
```

# 2. MaterialPageRoute

MaterialPageRoute继承自PageRoute类，PageRoute类是一个抽象类，表示占有整个屏幕空间的一个模态路由页面，它还定义了路由构建及切换时过渡动画的相关接口及属性。MaterialPageRoute 是 Material组件库提供的组件，它可以针对不同平台，实现与平台页面切换动画风格一致的路由切换动画。它有四个参数：

- builder：WidgetBuilder类型的回调函数。作用是构建路由页面的具体内容，返回值是一个widget 需要实现此回调，返回新路由的实例
- settings：包含路由的配置信息，如路由名称、是否初始路由（首页）
- maintainState：离开的页面是否保存在内存中，设置false则会销毁释放
- fullscreenDialog：表示新的路由页面是否是一个全屏的模态对话框

```dart
MaterialPageRoute({
  // WidgetBuilder类型的回调函数 作用是构建路由页面的具体内容，返回值是一个widget 需要实现此回调，返回新路由的实例
  WidgetBuilder builder,
  // 包含路由的配置信息，如路由名称、是否初始路由（首页）
  RouteSettings settings,
  // 离开的页面是否保存在内存中，设置false则会销毁释放
  bool maintainState = true,
  // 表示新的路由页面是否是一个全屏的模态对话框
  bool fullscreenDialog = false,
})
```

# 3. Navigator

Navigator是一个路由管理的组件，它提供了打开和退出路由页方法。Navigator通过一个栈来管理活动路由集合。通常当前屏幕显示的页面就是栈顶的路由。常用方法如下：

- push：打开一个新路由，并将其推入栈顶

```dart
// 语法
Navigator.push(BuildContext context, Route route)

// 示例
Navigator.push(context, MaterialPageRoute(builder: (context) {
  return const AboutPage();
}));
```

- pop：退出当前路由，并从栈顶弹出

```dart
// 语法
Navigator.pop(BuildContext context, [ result ])

// 示例
Navigator.pop(context);
```

# 4. 路由传值

很多时候，在路由跳转时我们需要带一些参数。

- About组件页面

```dart
class AboutPage extends StatefulWidget {
  final String text;
  const AboutPage({super.key, required this.text});

  @override
  State<AboutPage> createState() => _AboutPageState();
}

class _AboutPageState extends State<AboutPage> {
  final _title = 'About Page';
  @override
  Widget build(BuildContext context) {
    return Scaffold(
        appBar: AppBar(title: Text(_title)),
        body: Center(
            child: ElevatedButton(
              onPressed: () => Navigator.pop(context, '我是返回值'),
              child: Text(widget.text),
        )));
  }
}
```

- 前一个页面

```dart
TextButton(
  onPressed: () async {
    var result = await Navigator.push(context, MaterialPageRoute(builder: (context) {
      return const AboutPage(text: '哈哈哈');
    }));
    print("路由返回值: $result");
  },
  child: const Text('open');
)
```

# 5. 命名路由

所谓命名路由（Named Route），即有名字的路由，我们可以先给路由起一个名字，然后就可以通过路由名字直接打开新的路由了，这为路由管理带来了一种直观、简单的方式。
要想使用命名路由，我们必须先提供并注册一个路由表，路由表是一个字典Map，键为路由名字，值为路由对应的builder回调函数，用于生成相应的Widget构建器。

```dart
Map<String, WidgetBuilder> routes;
```

- 注册路由表

```dart
routes:{
  // 注册首页路由
  "/":(context) => MyHomePage(title: 'Flutter Demo Home Page'),
  "new_page":(context) => NewRoute(),
} 
```

- 通过路由名打开路由页

```dart
// 语法
Navigator.pushNamed(BuildContext context, String routeName, {Object arguments})

// 例子
onPressed: () {
  Navigator.pushNamed(context, "new_page");
}

onPressed: () {
  Navigator.push(
    context,
    MaterialPageRoute(builder: (context) { return NewRoute(); })
  );  
}
```

- 命名路由传参

```dart
// 传递参数
onPressed: () async {
  var result = await Navigator.of(context).pushNamed(('about'), arguments: '哈哈嘿嘿');
  print("路由返回值: $result");
}
```

```dart
// 接收参数
class AboutPage extends StatefulWidget {
  final String text;
  const AboutPage({super.key, required this.text});

  @override
  State<AboutPage> createState() => _AboutPageState();
}

class _AboutPageState extends State<AboutPage> {
  final _title = 'About Page';
  @override
  Widget build(BuildContext context) {
    // 获取传递的参数
    var args = ModalRoute.of(context)?.settings.arguments;
    return Scaffold(
        appBar: AppBar(title: Text(_title)),
        body: Center(
            child: ElevatedButton(
              onPressed: () => {
                Navigator.pop(context, '我是返回值$args')
              },
              child: Text(widget.text),
        )));
  }
}
```

# 6. 路由守卫onGenerateRoute

类似于vue-router中的beforeEach。onGenerateRoute，它在打开命名路由时，如果路由表中已经注册，则会调用路由表中的builder函数来生成路由组件。否则，就会调用一次（只会对命名路由生效）。语法如下：

```dart
MaterialApp(
  // ...省略无关代码
  onGenerateRoute:(RouteSettings settings){
	  return MaterialPageRoute(builder: (context){
		   String routeName = settings.name;
       print(routeName)
     }
   );
  }
);
```