# 📚 目录

1. [简介](#1-简介)
2. [StatelessWidget](#2-statelesswidget)
3. [Context上下文](#3-context上下文)
4. [StatefulWidget](#4-statefulwidget)
5. [State](#5-state)
6. [State的生命周期](#6-state的生命周期)
7. [在widget树中获取State对象](#7-在widget树中获取state对象)
    1. [通过Context](#7-1-通过context)
    2. [通过GlobalKey](#7-2-通过globalkey)
8. [通过RenderObject自定义Widget](#8-通过RenderObject自定义Widget)
8. [常用基础组件](#9-常用基础组件)
---

# 1. 简介

widget的功能是`描述一个UI元素的配置信息`。它不仅可以表示UI元素，也可以表示一些功能性的组件如：用于手势检测的 GestureDetector 、用于APP主题数据传递的 Theme 等等。Flutter 中是通过 Widget 嵌套 Widget 的方式来构建UI和进行事件处理的，所以，Flutter 中万物皆为Widget。

- 根据 Widget 树生成一个 Element 树
- 根据 Element 树生成 Render 树
- 根据渲染树生成 Layer 树
- Flutter 真正的布局和渲染逻辑在 Render 树中
- Element 是 Widget 和 RenderObject 的粘合剂，可以理解为一个中间代理

# 2. StatelessWidget

无状态组件tatelessWidget，用于不需要维护状态的场景，它通常在build方法中通过嵌套其他 widget 来构建UI，在构建过程中会递归的构建其嵌套的 widget。它继承自widget类，重写了createElement()方法：

- StatefulWidget的类定义

```dart
// 不可变的 属性必须是 final
@immutable
// 继承DiagnosticableTree诊断树 提供调试信息
abstract class Widget extends DiagnosticableTree {
  // key属性类似于 React/Vue 中的key
  const Widget({ this.key });

  final Key? key;

  // Flutter 框架隐式调用createElement() 开发者无需关注
  @protected
  @factory
  Element createElement();

  @override
  String toStringShort() {
    final String type = objectRuntimeType(this, 'Widget');
    return key == null ? type : '$type-$key';
  }

  // 复写父类的方法，主要是设置诊断树的一些特性
  @override
  void debugFillProperties(DiagnosticPropertiesBuilder properties) {
    super.debugFillProperties(properties);
    properties.defaultDiagnosticsTreeStyle = DiagnosticsTreeStyle.dense;
  }

  @override
  @nonVirtual
  bool operator ==(Object other) => super == other;

  @override
  @nonVirtual
  int get hashCode => super.hashCode;

  // 更新 决定是否在下一次build时复用旧的 widget
  static bool canUpdate(Widget oldWidget, Widget newWidget) {
    return oldWidget.runtimeType == newWidget.runtimeType
        && oldWidget.key == newWidget.key;
  }
  // ......
}
```

```dart
@override
StatelessElement createElement() => StatelessElement(this);
```

- 例子

```dart
class Echo extends StatelessWidget  {
  const Echo({
    Key? key,  
    required this.text,
    this.backgroundColor = Colors.grey,
  }):super(key:key);
    
  final String text;
  final Color backgroundColor;

  @override
  Widget build(BuildContext context) {
    return Center(
      child: Container(
        color: backgroundColor,
        child: Text(text),
      ),
    );
  }
}

Widget build(BuildContext context) {
  return Echo(text: "hello world");
}
```

# 3. Context上下文

build方法有一个context参数，它是BuildContext类的一个实例，表示当前 widget 在 widget 树中的上下文，每一个 widget 都会对应一个 context 对象。如下：

```dart
// 在子树中获取父级 widget 的 title 属性
class ContextRoute extends StatelessWidget  {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text("Context测试"),
      ),
      body: Container(
        child: Builder(builder: (context) {
          // 在 widget 树中向上查找最近的父级`Scaffold`  widget 
          Scaffold scaffold = context.findAncestorWidgetOfExactType<Scaffold>();
          // 直接返回 AppBar的title， 此处实际上是Text("Context测试")
          return (scaffold.appBar as AppBar).title;
        }),
      ),
    );
  }
}
```

# 4. StatefulWidget

StatefulWidget也是继承自widget类，并重写了createElement()方法，但是返回的Element 对象并不相同；另外StatefulWidget类中添加了一个新的接口createState()。

- StatefulWidget的类定义

```dart
abstract class StatefulWidget extends Widget {
  const StatefulWidget({ Key key }) : super(key: key);
    
  // 配置数据
  @override
  StatefulElement createElement() => StatefulElement(this);
    
  // 创建和 StatefulWidget 相关的状态
  @protected
  State createState();
}
```

# 5. State

一个 StatefulWidget 类会对应一个 State 类，State表示与其对应的 StatefulWidget 要维护的状态，State 中的保存的状态信息可以做如下操作：

- 在 widget 构建时可以被同步读取
- 在 widget 生命周期中可以被改变，当State被改变时，可以手动调用其setState()方法通知Flutter 框架状态发生改变，Flutter 框架会重新调用其build方法重新构建widget树

State 中有两个常用属性：

- widget，它表示与该 State 实例关联的 widget 实例，由Flutter 框架动态设置，重新构建时，也会更新。
- context。StatefulWidget对应的 BuildContext，示当前 widget 在 widget 树中的上下文

# 6. State生命周期

- initState：当 widget 第一次插入到 widget 树时会被调用，只会调用一次。
- didChangeDependencies：当State对象的依赖发生变化时会被调用。
- build：主要是用于构建 widget 子树，在调用initState、didUpdateWidget、setState、didChangeDependencies，以及移除后重新插入之后会调用。
- reassemble：热重载时会被调用，生产环境不会调用。
- didUpdateWidget：在 widget 重新构建时，在新旧 widget 的key和runtimeType同时相等时didUpdateWidget()就会被调用。
- deactivate：当 State 对象从树中被移除时，会调用此回调。
- dispose：当 State 对象从树中被永久移除时调用，通常在此回调中释放资源。

```dart
// 继承StatefulWidget
class CounterWidget extends StatefulWidget {
  const CounterWidget({Key? key, this.initValue = 0});

  final int initValue;

  @override
  _CounterWidgetState createState() => _CounterWidgetState();
}

// 定义状态
class _CounterWidgetState extends State<CounterWidget> {
  int _counter = 0;

  // 当 widget 第一次插入到 widget 树时会被调用 只会调用一次
  @override
  void initState() {
    super.initState();
    //初始化状态
    _counter = widget.initValue;
    print("initState");
  }

  // 主要是用于构建 widget 子树，在调用initState、didUpdateWidget、setState、didChangeDependencies，以及移除后重新插入之后会调用。
  @override
  Widget build(BuildContext context) {
    print("build");
    return Scaffold(
      body: Center(
        child: TextButton(
          child: Text('$_counter'),
          //点击后计数器自增
          onPressed: () => setState(
            () => ++_counter,
          ),
        ),
      ),
    );
  }

  // 在 widget 重新构建时，在新旧 widget 的key和runtimeType同时相等时didUpdateWidget()就会被调用。
  @override
  void didUpdateWidget(CounterWidget oldWidget) {
    super.didUpdateWidget(oldWidget);
    print("didUpdateWidget ");
  }

  // 当 State 对象从树中被移除时，会调用此回调。
  @override
  void deactivate() {
    super.deactivate();
    print("deactivate");
  }

  // 当 State 对象从树中被永久移除时调用，通常在此回调中释放资源
  @override
  void dispose() {
    super.dispose();
    print("dispose");
  }

  // 热重载时会被调用，生产环境不会调用。
  @override
  void reassemble() {
    super.reassemble();
    print("reassemble");
  }

  // 当State对象的依赖发生变化时会被调用。
  @override
  void didChangeDependencies() {
    super.didChangeDependencies();
    print("didChangeDependencies");
  }
}
```

# 7. 在widget树中获取State对象

由于 StatefulWidget 的具体逻辑都在其 State 中，所以很多时候，我们需要获取 StatefulWidget 对应的State 对象来调用一些方法。

## 7-1. 通过Context

context对象有一个findAncestorStateOfType()方法，该方法可以从当前节点沿着 widget 树向上查找指定类型的 StatefulWidget 对应的 State 对象。下面是实现打开 SnackBar 的示例：

```dart
class GetStateObjectRoute extends StatefulWidget {
  const GetStateObjectRoute({Key? key}) : super(key: key);

  @override
  State<GetStateObjectRoute> createState() => _GetStateObjectRouteState();
}

class _GetStateObjectRouteState extends State<GetStateObjectRoute> {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text("子树中获取State对象"),
      ),
      body: Center(
        child: Column(
          children: [
            Builder(builder: (context) {
              return ElevatedButton(
                onPressed: () {
                  // 查找父级最近的Scaffold对应的ScaffoldState对象
                  ScaffoldState _state = context.findAncestorStateOfType<ScaffoldState>()!;
                  // 打开抽屉菜单
                  _state.openDrawer();
                },
                child: Text('打开抽屉菜单1'),
              );
            }),
          ],
        ),
      ),
      drawer: Drawer(),
    );
  }
}
```

> 在 Flutter 开发中有一个默认的约定：如果 StatefulWidget 的状态是希望暴露出的，应当在 StatefulWidget 中提供一个of 静态方法来获取其 State 对象，开发者便可直接通过该方法来获取；如果 State不希望暴露，则不提供of方法。这个约定在 Flutter SDK 里随处可见。

```dart
Builder(builder: (context) {
  return ElevatedButton(
    onPressed: () {
      // 直接通过of静态方法来获取ScaffoldState
      ScaffoldState _state = Scaffold.of(context);
      // 打开抽屉菜单
      _state.openDrawer();
    },
    child: Text('打开抽屉菜单2'),
  );
})
```

## 7-2. 通过GlobalKey

Flutter还有一种通用的获取State对象的方法——通过GlobalKey来获取！ 步骤分两步：

- 给目标StatefulWidget添加GlobalKey。

```dart
//定义一个globalKey, 由于GlobalKey要保持全局唯一性，我们使用静态变量存储
static GlobalKey<ScaffoldState> _globalKey= GlobalKey();

Scaffold(
  // 设置key
  key: _globalKey,
  // ...... 
)
```

- 通过GlobalKey来获取State对象

```dart
_globalKey.currentState.openDrawer()
```

GlobalKey 是 Flutter 提供的一种在整个 App 中引用 element 的机制。如果一个 widget 设置了GlobalKey，那么我们便可以通过globalKey.currentWidget获得该 widget 对象、globalKey.currentElement来获得 widget 对应的element对象，如果当前 widget 是StatefulWidget，则可以通过globalKey.currentState来获得该 widget 对应的state对象。

# 8. 通过RenderObject自定义Widget

StatelessWidget 和 StatefulWidget 都是用于组合其他组件的，二自定义组件的方式就是通过定义RenderObject 来实现。

```dart
class CustomWidget extends LeafRenderObjectWidget{
  @override
  RenderObject createRenderObject(BuildContext context) {
    // 创建 RenderObject
    return RenderCustomObject();
  }
  @override
  void updateRenderObject(BuildContext context, RenderCustomObject  renderObject) {
    // 更新 RenderObject
    super.updateRenderObject(context, renderObject);
  }
}

class RenderCustomObject extends RenderBox{

  @override
  void performLayout() {
    // 实现布局逻辑
  }

  @override
  void paint(PaintingContext context, Offset offset) {
    // 实现绘制
  }
}
```

# 9. 常用基础组件

有Material（Android视觉风格）和Cupertino（iOS视觉风格）组件库

- Text：创建一个带格式的文本。
- Row：水平行。
- Column：垂直列。
- Stack：类似绝对定位，允许子 widget 堆叠。
- Container：创建矩形视觉元素。可以装饰一个BoxDecoration，比如背景、边框、渐变、阴影等。