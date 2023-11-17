# 📚 目录

1. [介绍](#1-介绍)
2. [布局原理和约束](#1-布局原理和约束)
3. [盒模型布局](#3-盒模型布局)
    1. [约束容器ConstrainedBox](#3-1-约束容器constrainedbox)
    2. [非约束容器UnconstrainedBox](#3-2-非约束容器unconstrainedbox)
4. [线性布局](#4-线性布局)
    1. [行row](#4-1-行row)
    2. [列column](#4-2-列column)
5. [弹性布局](#4-弹性布局)
6. [流式布局](#6-流式布局)
    1. [Wrap](#6-1-wrap)
    1. [Flow](#6-2-flow)
7. [层叠布局](#7-层叠布局)
8. [对齐和相对定位](#8-对齐和相对定位)
9. [布局构建回调](#9-布局构建回调)
    1. [LayoutBuilder布局过程中](#9-1-layoutbuilder布局过程中)
    2. [AfterLayout布局完成后执行](#9-2-afterlayout布局完成后执行)
---

# 1. 介绍

布局类组件都会包含一个或多个子组件，不同的布局类组件对子组件排列（layout）方式不同。如下：

|属性|说明|用途|
|--|--|--|
|LeafRenderObjectWidget|非容器类组件基类|Widget树的叶子节点，用于没有子节点的widget，通常基础组件都属于这一类（比如：Image）|
|SingleChildRenderObjectWidget|单子组件基类|包含一个子Widget，如：ConstrainedBox、DecoratedBox等|
|MultiChildRenderObjectWidget|多子组件基类|包含多个子Widget，一般都有一个children参数，接受一个Widget数组。如Row、Column、Stack等|

# 2. 布局原理和约束

Flutter 中有两种布局模型，分别是基于 RenderBox 的盒模型布局、基于 Sliver ( RenderSliver ) 按需加载列表布局。布局流程如下：

- 上层组件向下层组件传递约束（constraints）条件。
- 下层组件确定自己的大小，然后告诉上层组件。注意下层组件的大小必须符合父组件的约束。
- 上层组件确定下层组件相对于自身的偏移和确定自身的大小（大多数情况下会根据子组件的大小来确定自身的大小）。

# 3. 盒模型布局

盒模型布局是 Flutter 中最常用的布局方式，它基于 RenderBox 类，所有的布局组件都继承自该类。在布局过程中父级传递给子级的约束信息由 BoxConstraints 描述，包含最大宽高信息，子组件大小需要在约束的范围内。

## 3-1. 约束容器ConstrainedBox

如下例子。虽然将Container的高度设置为5像素，但是最终却是50像素，这正是ConstrainedBox的最小高度限制生效了：

```dart
ConstrainedBox(
  constraints: BoxConstraints(
    // 宽度尽可能大
    minWidth: double.infinity,
    // 最小高度为50像素
    minHeight: 50.0
  ),
  child: Container(
    height: 5.0, 
    child: DecoratedBox(
      decoration: BoxDecoration(color: Colors.red),
    ),
  ),
)
```
当存在有多重限制时，取父子中相应数值较大的。实际上，只有这样才能保证父限制与子限制不冲突。

## 3-2. 非约束容器UnconstrainedBox

有时候想要去掉约束，则可以使用UnconstrainedBox，只不过，UnconstrainedBox对父组件限制的“去除”并非是真正的去除，UnconstrainedBox其本身还是会收到约束，只是UnconstrainedBox它内部的元素不会被约束了。因为：任何时候子组件都必须遵守其父组件的约束

```dart
ConstrainedBox(
  // 父
  constraints: BoxConstraints(minWidth: 60.0, minHeight: 100.0),
  // 去除父级限制
  child: UnconstrainedBox(
    child: ConstrainedBox(
      // 子
      constraints: BoxConstraints(minWidth: 90.0, minHeight: 20.0),
      child: redBox,
    ),
  )
)
```

# 4. 线性布局

所谓线性布局，即指沿水平或垂直方向排列子组件。Flutter 中通过Row和Column来实现线性布局，类似于Android 中的LinearLayout控件。Row和Column都继承自Flex。

## 4-1. 行Row

Row可以沿水平方向排列其子widget，也就是横向排列，Row的高度等于子组件中最高的子元素高度。

|属性|说明|默认值|
|--|--|--|
|textDirection|水平方向子组件的布局顺序是从左往右还是从右往左|系统当前Locale环境的文本方向|
|mainAxisSize|Row在主轴(水平)方向占用的空间|MainAxisSize.max 尽可能多的占用水平方向的空间|
|verticalDirection|Row纵轴（垂直）的对齐方向|VerticalDirection.down 从上到下|
|mainAxisAlignment|子组件在Row所占用的空间内水平对齐方式|-|
|crossAxisAlignment|子组件在Row所占用空间内垂直方向的对齐方式|-|
|children|子组件数组|-|

例子：

```dart
@override
Widget build(BuildContext context) {
  return Scaffold(
    appBar: AppBar(title: Text(title),),
    body: Row(
      // 水平方向居中对齐
      mainAxisAlignment: MainAxisAlignment.center,
      // 垂直方向居中对齐
      crossAxisAlignment: CrossAxisAlignment.center,
      children: [
        const Text('111'),
        const Text('222'),
        const Text('333')
      ],
    ),
  );
}
```

## 4-2. 列Column

Column可以在垂直方向排列其子组件，也就是竖向排列。

|属性|说明|默认值|
|--|--|--|
|textDirection|水平方向子组件的布局顺序是从左往右还是从右往左|系统当前Locale环境的文本方向|
|mainAxisSize|Column在主轴(水平)方向占用的空间|MainAxisSize.max 尽可能多的占用水平方向的空间|
|verticalDirection|Column纵轴（垂直）的对齐方向|VerticalDirection.down 从上到下|
|mainAxisAlignment|子组件在Column所占用的空间内水平对齐方式|-|
|crossAxisAlignment|子组件在Column所占用空间内垂直方向的对齐方式|-|
|children|子组件数组|-|

例子：

```dart
@override
Widget build(BuildContext context) {
  return Scaffold(
    appBar: AppBar(title: Text(title),),
    body: Column(
      // 水平方向居中对齐
      mainAxisAlignment: MainAxisAlignment.center,
      // 垂直方向居中对齐
      crossAxisAlignment: CrossAxisAlignment.center,
      children: [
        const Text('111'),
        const Text('222'),
        const Text('333')
      ],
    ),
  );
}
```

# 5. 弹性布局

弹性布局允许子组件按照一定比例来分配父容器空间。Flutter 中的弹性布局主要通过Flex和Expanded来配合实现。Flex组件可以沿着水平或垂直方向排列子组件，如果你知道主轴方向，使用Row或Column会方便一些，因为Row和Column都继承自Flex，属性基本相同。而Expanded 只能作为 Flex 的子组件。（因为 Row和Column 都继承自 Flex，所以 Expanded 也可以作为它们的子组件）

如下代码，最终效果是一个左右各占一半的双色长方形：

```dart
import 'package:flutter/material.dart';

class AboutPage extends StatefulWidget {
  final String userName;
  const AboutPage({super.key, required this.userName});

  @override
  State<AboutPage> createState() => AboutPageState();
}

class AboutPageState extends State<AboutPage> {
  final title = 'About Page';
  var count = 0;
  void handleAdd() {
    count++;
    debugPrint('Count：$count');
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text(title),
        backgroundColor: Theme.of(context).colorScheme.inversePrimary,
      ),
      body: ConstrainedBox(
        constraints: const BoxConstraints(minWidth: double.infinity),
        child: Column(
          children: [
            Flex(
              direction: Axis.horizontal,
              children: [
                Expanded(
                  flex: 2,
                  child: Container(
                    height: 30.0,
                    color: Colors.blue,
                    child: const Text('A'),
                  ),
                ),
                Expanded(
                  flex: 2,
                  child: Container(
                    height: 30.0,
                    color: Colors.yellow,
                    child: const Text('B'),
                  ),
                ),
              ],
            )
          ],
        ),
      ),
    );
  }
}
```

# 6. 流式布局

Row中，如果子 widget 超出屏幕范围，则会报溢出错误，这是因为Row默认只有一行，如果超出屏幕不会折行。我们把超出屏幕显示范围会自动折行的布局称为流式布局，Flutter中通过Wrap和Flow来支持流式布局。

## 6-1. Wrap

Wrap 是一个流式布局的容器，它能够根据子组件的大小自动换行。

|属性|说明|默认值|
|--|--|--|
|textDirection|水平方向子组件的布局顺序是从左往右还是从右往左|系统当前Locale环境的文本方向|
|mainAxisSize|Column在主轴(水平)方向占用的空间|MainAxisSize.max 尽可能多的占用水平方向的空间|
|verticalDirection|Column纵轴（垂直）的对齐方向|VerticalDirection.down 从上到下|
|mainAxisAlignment|子组件在Column所占用的空间内水平对齐方式|-|
|crossAxisAlignment|子组件在Column所占用空间内垂直方向的对齐方式|-|
|children|主轴方向子widget的间距|-|
|spacing|子组件数组|-|
|runSpacing|纵轴方向的间距|-|
|runAlignment|纵轴方向的对齐方式|-|

```dart
Widget build(BuildContext context) {
  return Scaffold(
    appBar: AppBar(
      title: Text(title),
      backgroundColor: Theme.of(context).colorScheme.inversePrimary,
    ),
    body: ConstrainedBox(
      constraints: const BoxConstraints(minWidth: double.infinity),
      child: const Wrap(
        spacing: 8.0, // 主轴(水平)方向间距
        runSpacing: 4.0, // 纵轴（垂直）方向间距
        alignment: WrapAlignment.center, //沿主轴方向居中
        children: <Widget>[
          Chip(
            avatar:
                CircleAvatar(backgroundColor: Colors.blue, child: Text('A')),
            label: Text('Hamilton'),
          ),
          Chip(
            avatar:
                CircleAvatar(backgroundColor: Colors.blue, child: Text('M')),
            label: Text('Lafayette'),
          ),
          Chip(
            avatar:
                CircleAvatar(backgroundColor: Colors.blue, child: Text('H')),
            label: Text('Mulligan'),
          ),
          Chip(
            avatar:
                CircleAvatar(backgroundColor: Colors.blue, child: Text('J')),
            label: Text('Laurens'),
          ),
        ],
      ),
    ),
  );
}
```

## 6-2. Flow

一般很少会使用Flow，因为其过于复杂，需要自己实现子 widget 的位置转换，在很多场景下首先要考虑的是Wrap是否满足需求。Flow主要用于一些需要自定义布局策略或性能要求较高(如动画中)的场景。

- Flow的优点：
    - 灵活：由于我们需要自己实现FlowDelegate的paintChildren()方法，所以我们需要自己计算每一个组件的位置，因此，可以自定义布局策略。
    - 性能好：Flow是一个对子组件尺寸以及位置调整非常高效的控件，在对子组件进行位置调整的时候进行了优化。Flow用转换矩阵在对子组件进行位置调整的时候进行了优化：在Flow定位过后，如果子组件的尺寸或者位置发生了变化，在FlowDelegate中的paintChildren()方法中调用context.paintChild 进行重绘，而context.paintChild在重绘时使用了转换矩阵，并没有实际调整组件位置。
- Flow的缺点：
    - 复杂：需要自己实现子 widget 的位置转换。
    - 大小固定：不能自适应子组件大小，必须通过指定父容器大小或自己实现TestFlowDelegate的getSize返回固定大小。

```dart
import 'package:flutter/material.dart';

class AboutPage extends StatefulWidget {
  final String userName;
  const AboutPage({super.key, required this.userName});

  @override
  State<AboutPage> createState() => AboutPageState();
}

class TestFlowDelegate extends FlowDelegate {
  EdgeInsets margin;

  TestFlowDelegate({this.margin = EdgeInsets.zero});

  double width = 0;
  double height = 0;

  @override
  void paintChildren(FlowPaintingContext context) {
    var x = margin.left;
    var y = margin.top;
    //计算每一个子widget的位置
    for (int i = 0; i < context.childCount; i++) {
      var w = context.getChildSize(i)!.width + x + margin.right;
      if (w < context.size.width) {
        context.paintChild(i, transform: Matrix4.translationValues(x, y, 0.0));
        x = w + margin.left;
      } else {
        x = margin.left;
        y += context.getChildSize(i)!.height + margin.top + margin.bottom;
        //绘制子widget(有优化)
        context.paintChild(i, transform: Matrix4.translationValues(x, y, 0.0));
        x += context.getChildSize(i)!.width + margin.left + margin.right;
      }
    }
  }

  @override
  Size getSize(BoxConstraints constraints) {
    // 指定Flow的大小，简单起见我们让宽度尽可能大，但高度指定为200，
    // 实际开发中我们需要根据子元素所占用的具体宽高来设置Flow大小
    return const Size(double.infinity, 200.0);
  }

  @override
  bool shouldRepaint(FlowDelegate oldDelegate) {
    return oldDelegate != this;
  }
}

class AboutPageState extends State<AboutPage> {
  final title = 'About Page';

  @override
  Widget build(BuildContext context) {

    return Scaffold(
      appBar: AppBar(
        title: Text(title),
        backgroundColor: Theme.of(context).colorScheme.inversePrimary,
      ),
      body: Flow(
        delegate: TestFlowDelegate(margin: const EdgeInsets.all(10.0)),
        children: <Widget>[
          Container(width: 80.0, height:80.0, color: Colors.red,),
          Container(width: 80.0, height:80.0, color: Colors.green,),
          Container(width: 80.0, height:80.0, color: Colors.blue,),
          Container(width: 80.0, height:80.0,  color: Colors.yellow,),
          Container(width: 80.0, height:80.0, color: Colors.brown,),
          Container(width: 80.0, height:80.0,  color: Colors.purple,),
        ],
      ),
    );
  }
}
```

# 7. 层叠布局

层叠布局和Web中的绝对定位是相似的，子组件可以根据距父容器四个角的位置来确定自身的位置。层叠布局允许子组件按照代码中声明的顺序堆叠起来。Flutter中使用Stack和Positioned这两个组件来配合实现绝对定位。Stack允许子组件堆叠，而Positioned用于根据Stack的四个角来确定子组件的位置。

- Stack

|属性|描述|
|--|--|
|alignment|决定如何去对齐没有定位（没有使用Positioned）或部分定位的子组件|
|textDirection|确定alignment对齐的参考系|
|fit|用于确定没有定位的子组件如何去适应Stack的大小|
|clipBehavior|决定对超出Stack显示空间的部分如何剪裁|

- Positioned

|属性|描述|
|--|--|
|top|离Stack上边的距离|
|left|离Stack左边的距离|
|right|离Stack右边的距离|
|bottom|离Stack下边的距离|
|width|宽度|
|height|高度|

- 例子

```dart
Widget build(BuildContext context) {
  return Scaffold(
    appBar: AppBar(
      title: Text(title),
      backgroundColor: Theme.of(context).colorScheme.inversePrimary,
    ),
    body: ConstrainedBox(
      constraints: const BoxConstraints.expand(),
      child: Stack(
        fit: StackFit.expand,
        alignment: Alignment.center,
        children: [
          Container(
            color: Colors.blue,
            child: const Text('111'),
          ),
          const Positioned(
            left: 18.0,
            child: Text('222'),
          ),
          const Positioned(
            top: 18.0,
            child: Text('333'),
          )
        ],
      ),
    ),
  );
}
```

# 8. 对齐和相对定位

如果我们只想简单的调整一个子元素在父元素中的位置的话，使用Align组件会比Stack和Positioned更简单一些。Align只能有一个子元素，不存在堆叠。

- Align

|属性|描述|
|--|--|
|alignment|一个AlignmentGeometry类型的值，表示子组件在父组件中的起始位置|
|widthFactor|是个缩放因子，会乘以子元素的宽，最终的结果就是Align 组件的宽|
|heightFactor|是个缩放因子，会乘以子元素的高，最终的结果就是Align 组件的高|

```dart
Widget build(BuildContext context) {

  return Scaffold(
    appBar: AppBar(
      title: Text(title),
      backgroundColor: Theme.of(context).colorScheme.inversePrimary,
    ),
    body: Container(
      width: double.infinity,
      height: 120.0,
      color: Colors.blue,
      child: const Align(
        // 右上角对齐
        alignment: Alignment.topRight,
        // 如果不指定宽高 则这个就生效
        // widthFactor: 2,
        // heightFactor: 2,
        child: FlutterLogo(
          size: 60,
        ),
      ),
    ),
  );
}
```

- Center

Center继承自Align，它比Align只少了一个alignment 参数，因为Center默认是居中对齐的。

```dart
Widget build(BuildContext context) {

  return Scaffold(
    appBar: AppBar(
      title: Text(title),
      backgroundColor: Theme.of(context).colorScheme.inversePrimary,
    ),
    body: Container(
      color: Colors.blue,
      child: const Center(
        widthFactor: 2,
        heightFactor: 2,
        child: FlutterLogo(
          size: 60,
        ),
      ),
    ),
  );
}
```

# 9. 布局构建回调

在Flutter中，我们可以使用LayoutBuilder和AfterLayout来处理布局构建后的回调。

## 9-1. LayoutBuilder布局过程中

通过 LayoutBuilder，我们可以在布局过程中拿到父组件传递的约束信息，然后我们可以根据约束信息动态的构建不同的布局。

```dart
import 'package:flutter/material.dart';
import 'package:flutter/rendering.dart';

class AboutPage extends StatefulWidget {
  final String userName;
  const AboutPage({super.key, required this.userName});

  @override
  State<AboutPage> createState() => AboutPageState();
}
// 处理排列函数
class ResponsiveColumn extends StatelessWidget {
  const ResponsiveColumn({Key? key, required this.children}) : super(key: key);

  final List<Widget> children;

  @override
  Widget build(BuildContext context) {
    // 通过 LayoutBuilder 拿到父组件传递的约束，然后判断 maxWidth 是否小于200
    return LayoutBuilder(
      builder: (BuildContext context, BoxConstraints constraints) {
        if (constraints.maxWidth < 200) {
          // 最大宽度小于200，显示单列
          return Column(
            mainAxisSize: MainAxisSize.min,
            children: children,
          );
        } else {
          // 大于200，显示双列
          var myChildren = <Widget>[];
          for (var i = 0; i < children.length; i += 2) {
            if (i + 1 < children.length) {
              myChildren.add(Row(
                mainAxisSize: MainAxisSize.min,
                children: [children[i], children[i + 1]],
              ));
            } else {
              myChildren.add(children[i]);
            }
          }
          return Column(
            mainAxisSize: MainAxisSize.min,
            children: myChildren,
          );
        }
      },
    );
  }
}

// 打印布局时的约束信息
class LayoutLogPrint<T> extends StatelessWidget {
  const LayoutLogPrint({
    Key? key,
    this.tag,
    required this.child,
  }) : super(key: key);

  final Widget child;
  final T? tag; //指定日志tag

  @override
  Widget build(BuildContext context) {
    return LayoutBuilder(builder: (_, constraints) {
      // assert在编译release版本时会被去除
      assert(() {
        debugPrint('${tag ?? key ?? child}: $constraints');
        return true;
      }());
      return child;
    });
  }
}

// 布局
class AboutPageState extends State<AboutPage> {
  final title = 'About Page';
  var count = 0;
  void handleAdd() {
    count++;
    debugPrint('Count：$count');
  }

  @override
  Widget build(BuildContext context) {
    var myChildren = List.filled(6, const Text("正"));
    return Scaffold(
      appBar: AppBar(
        title: Text(title),
        backgroundColor: Theme.of(context).colorScheme.inversePrimary,
      ),
      body: Column(
        children: [
          SizedBox(
            width: 240,
            child: ResponsiveColumn(
              children: myChildren,
            ),
          ),
          ResponsiveColumn(children: myChildren),
          // 打印布局时的约束信息 宽高
          const LayoutLogPrint(child:Text("xxx"))
        ],
      ),
    );
  }
}
```

## 9-2. AfterLayout布局完成后执行

AfterLayout不是一个widget，而是一个Mixin，可以用于在widget布局完成后执行代码。这对于在布局构建完成后需要进行一些计算或调整的场景非常有用。在项目中可以使用开源插件：[传送门](https://pub-web.flutter-io.cn/packages/after_layout)

- 先安装包

```shell
flutter pub add after_layout
```

- 示例

```dart
import 'package:flutter/material.dart';
import 'package:after_layout/after_layout.dart';

class AboutPage extends StatefulWidget {
  final String userName;
  const AboutPage({super.key, required this.userName});

  @override
  State<AboutPage> createState() => AboutPageState();
}

class AboutPageState extends State<AboutPage> with AfterLayoutMixin<AboutPage> {
  final title = 'About Page';

  @override
  void afterFirstLayout(BuildContext context) {
    // 获取渲染的组件
    final RenderBox renderBox = context.findRenderObject() as RenderBox;
    // 获取组件大小
    final size = renderBox.size;
    // 获取组件偏移量
    final offset = renderBox.localToGlobal(Offset.zero);
    debugPrint(size.toString());
    debugPrint(offset.toString());
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text(title),
        backgroundColor: Theme.of(context).colorScheme.inversePrimary,
      ),
      body: Container(
        color: Colors.blue,
        child: const Center(
          widthFactor: 2,
          heightFactor: 2,
          child: FlutterLogo(
            size: 60,
          ),
        ),
      ),
    );
  }
}
```