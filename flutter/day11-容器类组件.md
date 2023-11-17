# 📚 目录

1. [填充组件Padding和EdgeInsets](#1-填充组件padding和edgeinsets)
2. [装饰容器DecoratedBox和BoxDecoration](#2-装饰容器decoratedbox和boxdecoration)
3. [变换Transform](#3-变换transform)
    1. [RotatedBox](#3-1-rotatedbox)
---

# 1. 填充组件Padding和EdgeInsets

Flutter里的padding和EdgeInsets是用来控制组件的填充和边距的。Padding一般包含一个child，以及一个EdgeInsetsGeometry抽象类，开发中，我们一般都使用EdgeInsets类，它是EdgeInsetsGeometry的一个子类，定义了一些设置填充的便捷方法，如下：

|方法|值|描述|
|---|---|---|
|fromLTRB|double：left、top、right、bottom|分别指定四个方向的填充|
|all|double|所有方向均使用相同数值的填充|
|only|double：{left, top, right ,bottom }|设置具体某个方向的填充|
|symmetric|double：{ vertical, horizontal }|设置对称方向的填充|

```dart
@override
Widget build(BuildContext context) {
  return Scaffold(
    appBar: AppBar(
      title: Text(title),
      backgroundColor: Theme.of(context).colorScheme.inversePrimary,
    ),
    body: const SizedBox(
      width: double.infinity,
      child: Padding(
        padding: EdgeInsets.all(16.16),
        child: Column(
          children: [
            Padding(
                padding: EdgeInsets.only(left: 20.0),
                child: Text('Padding-Left：20')),
            Padding(
                padding: EdgeInsets.symmetric(vertical: 8),
                child: Text('Padding-vertical：8'))
          ],
        ),
      ),
    ),
  );
}
```

# 2. 装饰容器DecoratedBox和BoxDecoration

DecoratedBox可以在其子组件绘制前(或后)绘制一些装饰，如背景、边框、渐变等。但是通常会直接使用BoxDecoration类，它是一个Decoration的子类，实现了常用的装饰元素的绘制。

|方法|值|描述|
|---|---|---|
|decoration|BoxDecoration|代表将要绘制的装饰|
|position|background、foreground|决定在哪里绘制|

- 渐变色按钮的例子：

```dart
@override
  Widget build(BuildContext context) {
    return Scaffold(
        appBar: AppBar(
          title: Text(title),
          backgroundColor: Theme.of(context).colorScheme.inversePrimary,
        ),
        body: DecoratedBox(
            decoration: BoxDecoration(
                gradient:
                    const LinearGradient(colors: [Colors.red, Colors.green]),
                borderRadius: BorderRadius.circular(3.0),
                boxShadow: const [
                  BoxShadow(
                      color: Colors.black,
                      offset: Offset(2.0, 2.0),
                      blurRadius: 4.0)
                ]),
            child: const Padding(
              padding: EdgeInsets.symmetric(horizontal: 80.0, vertical: 18.0),
              child: Text(
                "Login",
                style: TextStyle(color: Colors.white),
              ),
            )));
  }
```

# 3. 变换Transform

Transform可以在其子组件绘制时对其应用一些矩阵变换来实现一些特效。Matrix4是一个4D矩阵，通过它我们可以实现各种矩阵操作。

> Transform的变换是应用在绘制阶段，而并不是应用在布局阶段，所以无论对子组件应用何种变化，其占用空间的大小和在屏幕上的位置都是固定不变的，因为这些是在布局阶段就确定的。

|属性|值|描述|
|---|---|---|
|Transform.translate|offset: Offset(-20.0, -5.0)|在绘制时沿x、y轴对子组件平移指定的距离|
|Transform.rotate|angle:math.pi/2|对子组件进行旋转变换（需要先导包：import 'dart:math' as math;）|
|Transform.scale|offset: Offset(-20.0, -5.0)|对子组件进行缩小或放大|

- 矩阵变换

```dart
Container(
  color: Colors.black,
  child: Transform(
    // 相对于坐标系原点的对齐方式
    alignment: Alignment.topRight,
    // 沿Y轴倾斜0.3弧度
    transform: Matrix4.skewY(0.3),
    child: Container(
      padding: const EdgeInsets.all(8.0),
      color: Colors.red,
      child: const Text('Hello world!'),
    ),
  ),
)
```

- 平移

```dart
DecoratedBox(
  decoration:BoxDecoration(color: Colors.red),
  //默认原点为左上角，左移20像素，向上平移5像素  
  child: Transform.translate(
    offset: Offset(-20.0, -5.0),
    child: Text("Hello world"),
  ),
)
```

- 旋转

```dart
import 'dart:math' as math;  

DecoratedBox(
  decoration:BoxDecoration(color: Colors.red),
  child: Transform.rotate(
    //旋转90度
    angle:math.pi/2 ,
    child: Text("Hello world"),
  ),
)
```

- 缩放

```dart
DecoratedBox(
  decoration:BoxDecoration(color: Colors.red),
  child: Transform.scale(
    scale: 1.5, //放大到1.5倍
    child: Text("Hello world")
  )
);
```

## 3-1. RotatedBox

RotatedBox和Transform.rotate功能相似，它们都可以对子组件进行旋转变换，但是有一点不同：RotatedBox的变换是在layout阶段，会影响在子组件的位置和大小。

```dart
Row(
  mainAxisAlignment: MainAxisAlignment.center,
  children: <Widget>[
    DecoratedBox(
      decoration: BoxDecoration(color: Colors.red),
      //将Transform.rotate换成RotatedBox  
      child: RotatedBox(
        quarterTurns: 1, //旋转90度(1/4圈)
        child: Text("Hello world"),
      ),
    ),
    Text("你好", style: TextStyle(color: Colors.green, fontSize: 18.0),)
  ],
)
```

# 4. 容器组件Container

Container是一个组合类容器，它本身不对应具体的RenderObject，它是DecoratedBox、ConstrainedBox、Transform、Padding、Align等组件组合的一个多功能容器，所以我们只需通过一个Container组件可以实现同时需要装饰、变换、限制的场景

|属性|值|描述|
|---|---|---|
|alignment|-|对齐|
|padding|-|内边距|
|color|-|背景色，和decoration互斥|
|decoration|-|背景装饰|
|foregroundDecoration|-|前景装饰|
|width|-|宽|
|height|-|高|
|constraints|-|约束，和width和height互斥|
|margin|-|外边距|
|transform|-|变换|
|child|-|子组件|


```dart
@override
Widget build(BuildContext context) {
  return Scaffold(
      appBar: AppBar(
        title: Text(title),
        backgroundColor: Theme.of(context).colorScheme.inversePrimary,
      ),
      body: Container(
        margin: const EdgeInsets.only(top: 50.0, left: 120.0),
        // 卡片大小约束
        constraints: const BoxConstraints.tightFor(width: 200.0, height: 150.0),
        // 装饰
        decoration: const BoxDecoration(
          // 背景装饰
          gradient: RadialGradient(
            // 背景径向渐变
            colors: [Colors.red, Colors.orange],
            center: Alignment.topLeft,
            radius: .98,
          ),
          boxShadow: [
            // 卡片阴影
            BoxShadow(
              color: Colors.black54,
              offset: Offset(2.0, 2.0),
              blurRadius: 4.0,
            )
          ],
        ),
        // 卡片倾斜变换
        transform: Matrix4.rotationZ(.2),
        // 卡片内文字居中
        alignment: Alignment.center,
        child: const Text(
          // 卡片文字
          "Hello World!", style: TextStyle(color: Colors.white, fontSize: 28.0),
        ),
      ));
}
```

# 5. 剪裁类组件

Flutter中提供了一些剪裁组件，用于对组件进行剪裁。（类似CSS里的overflow）

|属性|描述|
|---|---|
|CliOval|子组件为正方形时剪裁成内贴圆形；为矩形时，剪裁成内贴椭圆|
|ClipRRect|将子组件剪裁为圆角矩形|
|ClipRect|默认剪裁掉子组件布局空间之外的绘制内容（溢出部分剪裁）|
|ClipPath|按照自定义的路径剪裁|

```dart
Widget avatar = Image.asset('static/portrait.png', width: 120.0);

@override
Widget build(BuildContext context) {
  return Scaffold(
      appBar: AppBar(
        title: Text(title),
        backgroundColor: Theme.of(context).colorScheme.inversePrimary,
      ),
      body: Container(
        margin: const EdgeInsets.all(16.0),
        child: Column(
          children: [
            // 原图不剪裁
            avatar,
            // 剪裁为圆形
            ClipOval(child: avatar),
            // 剪裁为圆角矩形
            ClipRRect(
                borderRadius: BorderRadius.circular(5.0), child: avatar),
            Row(
                mainAxisAlignment: MainAxisAlignment.center,
                children: <Widget>[
                  ClipRect(
                    // 将溢出部分剪裁
                    child: Align(
                      alignment: Alignment.topLeft,
                      // 宽度设为原来宽度一半
                      widthFactor: .5,
                      child: avatar,
                    ),
                  ),
                  const Text("Hello!", style: TextStyle(color: Colors.green))
                ])
          ],
        ),
      ));
}
```

- 自定义裁剪

> 图片所占用的空间大小仍然不变 这是因为组件大小是是在layout阶段确定的，而剪裁是在之后的绘制阶段进行的

```dart
class MyClipper extends CustomClipper<Rect> {
  // 用于获取剪裁区域 由于图片大小是120x120 我们返回剪裁区域为图片右上侧60像素的范围
  @override
  Rect getClip(Size size) => Rect.fromLTWH(60.0, 0.0, 60.0, 60.0);

  // 是否重新剪裁 剪裁区域始终不会发生变化时应该返回false 否则返回true
  @override
  bool shouldReclip(CustomClipper<Rect> oldClipper) => false;
}

class AboutPageState extends State<AboutPage> {
  final title = 'About Page';

  Widget avatar = Image.asset('static/portrait.png', width: 120.0);

  @override
  Widget build(BuildContext context) {
    return Scaffold(
        appBar: AppBar(
          title: Text(title),
          backgroundColor: Theme.of(context).colorScheme.inversePrimary,
        ),
        body: Container(
          margin: const EdgeInsets.all(16.0),
          child: Column(
            children: [
              // 原图不剪裁
              avatar,
              // 
              ClipRect(
                  clipper: MyClipper(), //使用自定义的clipper
                  child: avatar
              )
            ],
          ),
        ));
  }
}
```

# 6. 空间适配

根据Flutter 的布局协议，父组件会将自身的最大显示空间作为约束传递给子组件，子组件应该遵守父组件的约束，如果子组件原始大小超过了父组件的约束区域，则需要进行一些缩小、裁剪或其他处理，如果不经过处理的话 Flutter 中就会显示一个溢出警告并在控制台打印错误日志。为了方便开发者自定义适配规则，Flutter 提供了一个 FittedBox 组件。

- 适配原理
    - FittedBox 在布局子组件时会忽略其父组件传递的约束，可以允许子组件无限大，即FittedBox 传递给子组件的约束为（0<=width<=double.infinity, 0<= height <=double.infinity）
    - FittedBox 对子组件布局结束后就可以获得子组件真实的大小。
    - FittedBox 知道子组件的真实大小也知道他父组件的约束，那么FittedBox 就可以通过指定的适配方式（BoxFit 枚举中指定），让起子组件在 FittedBox 父组件的约束范围内按照指定的方式显示

如下代码，因为父Container要比子Container小，然后子Container的模式是等比缩放，因为子组件的长宽并不相同，缩放后，露出了父组件的颜色：

```dart
Widget build(BuildContext context) {
  return Scaffold(
      appBar: AppBar(
        title: Text(title),
        backgroundColor: Theme.of(context).colorScheme.inversePrimary,
      ),
      body: Container(
        margin: const EdgeInsets.all(16.0),
        width: 90.0,
        height: 90.0,
        color: Colors.green,
        child: FittedBox(
          fit: BoxFit.contain,
          child: Container(
            width: 100.0,
            height: 80.0,
            color: Colors.black,
          ),
        ),
      ));
}
```

# 7. 页面脚手架

一个完整的路由页可能会包含导航栏、抽屉菜单(Drawer)以及底部 Tab 导航菜单等。Scaffold 是一个路由页的骨架，我们使用它可以很容易地拼装出一个完整的页面。

|组件名|解释|
|---|---|
|AppBar|导航栏骨架|
|MyDrawer|抽屉菜单|
|BottomNavigationBar|底部导航栏|
|FloatingActionButton|漂浮按钮|

- 一个带抽屉和Tabbar的页面框架例子：

```dart
// 抽屉类
class MyDrawer extends StatelessWidget {
  const MyDrawer({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Drawer(
      child: MediaQuery.removePadding(
        context: context,
        // 移除抽屉菜单顶部默认留白
        removeTop: true,
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Padding(
              padding: const EdgeInsets.only(top: 38.0),
              child: Row(
                children: [
                  Padding(
                    padding: const EdgeInsets.symmetric(horizontal: 16.0),
                    child: ClipOval(
                      child: Image.asset(
                        "static/portrait.png",
                        width: 80,
                      ),
                    ),
                  ),
                  const Text(
                    "Hello",
                    style: TextStyle(fontWeight: FontWeight.bold),
                  )
                ],
              ),
            ),
            Expanded(
              child: ListView(
                children: const [
                  ListTile(
                    leading: Icon(Icons.add),
                    title: Text('添加内容'),
                  ),
                  ListTile(
                    leading: Icon(Icons.settings),
                    title: Text('管理账号'),
                  ),
                ],
              ),
            ),
          ],
        ),
      ),
    );
  }
}

// 页面
class MyMainPageState extends State<MyMainPage> {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      // 头部
      appBar: AppBar(
        backgroundColor: Theme.of(context).colorScheme.inversePrimary,
        title: Text(widget.title),
        actions: [IconButton(onPressed: () {}, icon: const Icon(Icons.share))],
        leading: Builder(builder: (context) {
          return IconButton(
              onPressed: () {
                Scaffold.of(context).openDrawer();
              },
              icon: const Icon(Icons.add));
        }),
      ),
      // 身体
      body: Center(
        child: Image.asset('static/portrait.png', width: 240.0),
      ),
      // 抽屉
      drawer: const MyDrawer(),
      // Tabbar
      bottomNavigationBar: BottomNavigationBar(
        items: const [
          BottomNavigationBarItem(icon: Icon(Icons.home), label: '首页'),
          BottomNavigationBarItem(
              icon: Icon(Icons.business), label: '办公'),
          BottomNavigationBarItem(icon: Icon(Icons.school), label: '学习')
        ],
        // 切换事件 索引从0开始
        onTap: (int type) {
          debugPrint(type.toString());
        },
      ),
      // 悬浮按钮
      floatingActionButton: FloatingActionButton(
        onPressed: () {},
        backgroundColor: Colors.green,
        child: const Icon(Icons.add),
      ),
    );
  }
}
```