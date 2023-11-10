# 📚 目录

1. [简介](#1-简介)
2. [StatelessWidget](#2-statelesswidget)
3. [Context上下文](#3-context上下文)
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

Navigator是一个路由管理的组件，它提供了打开和退出路由页方法。Navigator通过一个栈来管理活动路由集合。通常当前屏幕显示的页面就是栈顶的路由。