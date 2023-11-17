# 📚 目录

1. [文本及字体样式](#1-文本及样式)
2. [各种按钮](#2-各种按钮)
3. [图片](#3-图片)
4. [icon](#4-icon)
    1. [自定义字体图标](#4-1-自定义字体图标)
5. [单选开关和复选框](#5-单选开关和复选框)
6. [输入框和表单](#6-输入框和表单)
    1. [TextField](#6-1-textfield)
    2. [Form](#6-2-form)
    3. [登录界面例子](#6-3-登录界面例子)
7. [进度指示器](#7-进度指示器)
    1. [线形LinearProgressIndicator](#7-1-线形linearprogressindicator)
    2. [环形CircularProgressIndicator](#7-2-环形circularprogressindicator)
---

# 1. 文本及样式

- Text：用于显示简单样式文本，它包含一些控制文本显示样式的一些属性：

```dart
// 字符串重复4次
Text("Hello world"*4,
  // 左对齐
  textAlign: TextAlign.left,
  // 指定文本显示的最大行数
  maxLines: 1,
  // 文本截断后以省略符表示
  overflow: TextOverflow.ellipsis,
  // 相对于当前字体大小的缩放因子
  textScaleFactor: 1.5
);
```

- TextStyle：用于指定文本显示的样式如颜色、字体、粗细、背景等：

```dart
Text("Hello world！",
  style: TextStyle(
    color: Colors.blue,
    fontSize: 18.0,
    // 行高 等于fontSize * height
    height: 1.2,  
    // 字体
    fontFamily: "Courier",
    background: Paint()..color=Colors.yellow,
    // 下划线
    decoration:TextDecoration.underline,
    // 下划线样式
    decorationStyle: TextDecorationStyle.dashed
  ),
);
```

- TextSpan：代表文本的一个片段，可对一个 Text 内容的不同部分按照不同的样式显示

```dart
Text.rich(
  TextSpan(
    children: [
     TextSpan(
       text: "链接: "
     ),
     TextSpan(
       text: "https://pub.dev.com",
       style: TextStyle(
         color: Colors.blue
       ),  
       recognizer: () => {
        print('xxyyxx')
      }
     ),
    ]
  )
)
```

- DefaultTextStyle：文本的样式默认是可以被继承的，可以用于指定文本的默认样式，如默认字体、颜色、大小等

```dart
// 设置文本默认样式
DefaultTextStyle(
  style: TextStyle(
    color:Colors.red,
    fontSize: 20.0,
  ),
  textAlign: TextAlign.start,
  child: Column(
    crossAxisAlignment: CrossAxisAlignment.start,
    children: <Widget>[
      Text("hello world"),
      Text("I am Jack"),
      Text("I am Jack",
        style: TextStyle(
          // 不继承默认样式
          inherit: false,
          color: Colors.grey
        ),
      ),
    ],
  ),
)
```

- 字体使用

字体首先要在pubspec.yaml中声明它们，以确保它们会打包到应用程序中。然后通过TextStyle属性使用字体。

- 在asset中声明

```yaml
flutter:
  fonts:
    - family: Raleway
      fonts:
        - asset: assets/fonts/Raleway-Regular.ttf
        - asset: assets/fonts/Raleway-Medium.ttf
          weight: 500
        - asset: assets/fonts/Raleway-SemiBold.ttf
          weight: 600
    - family: AbrilFatface
      fonts:
        - asset: assets/fonts/abrilfatface/AbrilFatface-Regular.ttf
```

- 使用

```dart
// 声明文本样式
const textStyle = const TextStyle(
  fontFamily: 'Raleway',
  // 假设上面的字体声明位于my_package包中 则需要指定包名
  package: 'my_package'
);

// 使用文本样式
var buttonText = const Text(
  "Use the font for this text",
  style: textStyle,
);
```

# 2. 各种按钮

Material 组件库中提供了多种按钮组件。他们都有如下特性：

1. 点击时按钮上会出现水波扩散的“水波动画”
2. 有一个onPressed属性来设置点击回调，当按钮按下时会执行该回调，如果不提供该回调则按钮会处于禁用状态，禁用状态不响应用户点击。

- ElevatedButton漂浮按钮

```dart
// 默认带有阴影和灰色背景。按下后，阴影会变大
ElevatedButton(
  child: Text("normal"),
  onPressed: () {},
)
```

- TextButton

```dart
// 默认背景透明并不带阴影。按下后，会有背景色
TextButton(
  child: Text("normal"),
  onPressed: () {},
)
```

- OutlinedButton

```dart
// 默认有一个边框，不带阴影且背景透明。按下后，边框颜色会变亮、同时出现背景和较弱的阴影
OutlinedButton(
  child: Text("normal"),
  onPressed: () {},
)
```

- IconButton

```dart
// 可点击的Icon，不包括文字，默认没有背景，点击后会出现背景
IconButton(
  icon: Icon(Icons.thumb_up),
  onPressed: () {},
)
```

- 带图标的按钮

```dart
ElevatedButton.icon(
  icon: Icon(Icons.send),
  label: Text("发送"),
  onPressed: () {},
)

OutlinedButton.icon(
  icon: Icon(Icons.add),
  label: Text("添加"),
  onPressed: () {},
)

TextButton.icon(
  icon: Icon(Icons.info),
  label: Text("详情"),
  onPressed: () {},
)
```

# 3. 图片

Flutter 中，我们可以通过Image组件来加载并显示图片，Image的数据源可以是asset、文件、内存以及网络资源。

- ImageProvider：一个抽象类，主要定义了图片数据获取的接口load()，从不同的数据源获取图片需要实现不同的ImageProvider ，如AssetImage是实现了从Asset中加载图片的 ImageProvider，而NetworkImage 实现了从网络加载图片的 ImageProvider。

- Image：Image widget 有一个必选的image参数，它对应一个 ImageProvider。

## 3-1. 加载本地图片

先在pubspec.yaml文件中flutter部分添加assets资源路径

```dart
Image(
  image: AssetImage("static/portrait.png"),
  width: 100.0
);
// or
Image.asset("static/portrait.png",
  width: 100.0,
)
```

## 3-2. 加载网络图片

```dart
Image(
  image: NetworkImage(
      "https://avatars2.githubusercontent.com/u/20411648?s=460&v=4"),
  width: 100.0,
)
// or
Image.network(
  "https://avatars2.githubusercontent.com/u/20411648?s=460&v=4",
  width: 100.0,
)
```

## 3-3. 属性

|属性|值|描述|
|--|--|--|
|width|-|宽度|
|height|-|高度|
|fit|fill、cover、contain、fitWidth、fitHeight、none|适应模式|
|color|-|混合色|
|colorBlendMode|-|混合模式|
|repeat|-|指定图片的重复规则|

# 4. icon

Flutter 中，可以像Web开发一样使用iconfont字体图标。Flutter默认包含了一套Material Design的(字体图标)[https://material.io/tools/icons/]，在pubspec.yaml文件中的配置如下：

```yaml
flutter:
  uses-material-design: true
```

- 使用

```dart
String icons = "";
// accessible: 0xe03e
icons += "\uE03e";
// error:  0xe237
icons += " \uE237";
// fingerprint: 0xe287
icons += " \uE287";

Text(
  icons,
  style: TextStyle(
    fontFamily: "MaterialIcons",
    fontSize: 24.0,
    color: Colors.green,
  ),
);
```

- 规范使用

```dart
Row(
  mainAxisAlignment: MainAxisAlignment.center,
  children: <Widget>[
    Icon(Icons.accessible,color: Colors.green),
    Icon(Icons.error,color: Colors.green),
    Icon(Icons.fingerprint,color: Colors.green),
  ],
)
```

## 4-1. 自定义字体图标

首先导入字体图标文件：

```yaml
fonts:
  - family: myIcon  # 指定一个字体名
    fonts:
      - asset: fonts/iconfont.ttf
```

定义一个MyIcons类，功能和Icons类一样，将字体文件中的所有图标都定义成静态变量：

```dart
class MyIcons{
  // book 图标
  static const IconData book = const IconData(
      0xe614, 
      fontFamily: 'myIcon', 
      matchTextDirection: true
  );
  // 微信图标
  static const IconData wechat = const IconData(
      0xec7d,  
      fontFamily: 'myIcon', 
      matchTextDirection: true
  );
}
```

使用

```dart
Row(
  mainAxisAlignment: MainAxisAlignment.center,
  children: <Widget>[
    Icon(MyIcons.book,color: Colors.purple),
    Icon(MyIcons.wechat,color: Colors.green),
  ],
)
```

# 5. 单选开关和复选框

Material 组件库中提供了 Material 风格的单选开关Switch和复选框Checkbox。但它们本身不会保存当前选中状态，选中状态都是由父组件来管理的。

```dart
class SwitchAndCheckBoxTestRoute extends StatefulWidget {
  @override
  _SwitchAndCheckBoxTestRouteState createState() => _SwitchAndCheckBoxTestRouteState();
}

class _SwitchAndCheckBoxTestRouteState extends State<SwitchAndCheckBoxTestRoute> {
  // 维护单选开关状态
  bool _switchSelected = true;
  // 维护复选框状态
  bool _checkboxSelected=true;
  @override
  Widget build(BuildContext context) {
    return Column(
      children: <Widget>[
        Switch(
          // 当前状态
          value: _switchSelected,
          onChanged:(value){
            // 重新构建页面  
            setState(() {
              _switchSelected=value;
            });
          },
        ),
        Checkbox(
          value: _checkboxSelected,
          // 选中时的颜色
          activeColor: Colors.red,
          onChanged:(value){
            setState(() {
              _checkboxSelected=value;
            });
          },
        )
      ],
    );
  }
}
```
Checkbox的大小是固定的，无法自定义，而Switch只能定义宽度，高度也是固定的。

# 6. 输入框和表单

Material 组件库中提供了输入框组件TextField和表单组件Form

## 6-1. TextField

TextField用于文本输入

- controller：编辑框的控制器，通过它可以设置/获取编辑框的内容、选择编辑内容、监听编辑文本改变事件。大多数情况下我们都需要显式提供一个controller来与文本框交互。
- focusNode：用于控制TextField是否占有当前键盘的输入焦点。
- InputDecoration：用于控制TextField的外观显示，如提示文本、背景颜色、边框等。
- keyboardType：设置该输入框默认的键盘输入类型。
    - text：文本键盘
    - multiline：多行文本，需和maxLines配合使用（设为null或大于1）
    - number：数字键盘
    - phone：数字键盘并显示*和#
    - datetime：日期输入键盘，Android上会显示:和-
    - emailAddress：电子邮件地址 会显示@和.
    - url：url输入键盘 会显示/和.
- textInputAction：设置键盘动作按钮的显示内容，默认是next
- style：正在编辑的文本样式。
- textAlign：输入框内编辑文本在水平方向的对齐方式。
- autofocus：是否自动获取焦点
- obscureText：是否隐藏正在编辑的文本，用于输入密码的场景等，文本内容会用•替换
- maxLines：输入框的最大行数，默认为1；如果为null，则无行数限制。
- maxLength：输入框文本的最大长度 设置后输入框右下角会显示输入的文本计数
- maxLengthEnforcement：当输入文本长度超过maxLength时如何处理，如截断、超出等。
- toolbarOptions：长按或鼠标右击时出现的菜单，包括 copy、cut、paste 以及 selectAll
- onChange：输入框内容改变时的回调函数，也可以通过controller来监听。
- onEditingComplete：输入框编辑完成时的回调函数。无参数。
- onSubmitted：输入框编辑完成时的回调函数。接收当前输入内容做为参数。
- inputFormatters：指定输入格式；当用户输入内容改变时，会根据指定的格式来校验。
- enable：如果为false，则禁用输入框，不响应输入和事件，显示禁用态样式（在其decoration中定义）
- cursorWidth：光标宽度
- cursorRadius：光标圆角
- cursorColor：光标颜色

## 6-2. Form

Form 组件，它可以对输入框进行分组，然后进行一些统一操作，如输入内容校验、输入框重置以及输入内容保存。Form继承自StatefulWidget对象，它对应的状态类为FormState，Form的子孙元素必须是FormField类型，为了方便使用，Flutter 提供了一个TextFormField组件，它继承自FormField类，也是TextField的一个包装类，所以除了FormField定义的属性之外，它还包括TextField的属性。

|属性|描述|
|--|--|
|autovalidate|是否自动校验输入内容，默认为false，即只有点击操作按钮时才校验|
|onWillPop|决定Form所在的路由是否可以直接返回（如点击返回按钮），该回调返回一个Future对象，如果 Future 的最终结果是false，则当前路由不会返回；如果为true，则会返回到上一个路由。此属性通常用于拦截返回按钮|
|onChanged|Form内容改变时的回调函数，可以用于对输入内容进行校验。|

FormState是Form的State类，可以通过Form.of()或GlobalKey获得。我们可以通过它来对Form的子孙FormField进行统一操作，常用方法如下：

- FormState.validate()：调用Form子孙FormField的validate回调，如果有一个校验失败，则返回false，所有校验失败项都会返回用户返回的错误提示。
- FormState.save()：调用Form子孙FormField的save回调，用于保存表单内容
- FormState.reset()：将子孙FormField的内容清空

## 6-3. 登录界面例子

```dart
import 'package:flutter/material.dart';

class LoginPage extends StatefulWidget {
  const LoginPage({super.key});

  @override
  State<LoginPage> createState() {
    return LoginPageState();
  }
}

class LoginPageState extends State<LoginPage> {
  TextEditingController userName = TextEditingController();
  TextEditingController password = TextEditingController();
  final formKey = GlobalKey<FormState>();

  void handleSubmit() {
    if ((formKey.currentState as FormState).validate()) {
      if (userName.text.length >= 6 && password.text.length >= 6) {
        // 提示
        ScaffoldMessenger.of(context).showSnackBar(
          const SnackBar(content: Text('验证通过！')),
        );
      } else {
        // 清除内容
        formKey.currentState?.reset();
        // 提示
        ScaffoldMessenger.of(context).showSnackBar(
          const SnackBar(content: Text('账号和密码长度不能低于6位数！')),
        );
      }
    } else {
      debugPrint('校验失败！');
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        backgroundColor: Theme.of(context).colorScheme.inversePrimary,
        title: const Text('Login Page'),
      ),
      body: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          Expanded(
            child: SingleChildScrollView(
              child: Padding(
                padding: const EdgeInsets.symmetric(horizontal: 16, vertical: 20),
                child: Form(
                  key: formKey,
                  child: Column(
                    crossAxisAlignment: CrossAxisAlignment.start,
                    children: [
                      TextFormField(
                        autofocus: false,
                    controller: userName,
                        decoration: const InputDecoration(
                          labelText: "账号：",
                          hintText: "请输入账号",
                          icon: Icon(Icons.person),
                        ),
                        validator: (v) {
                          return v!.trim().isNotEmpty ? null : '账号不能为空';
                        },
                      ),
                      TextFormField(
                        autofocus: false,
                        controller: password,
                        decoration: const InputDecoration(
                          labelText: "密码：",
                          hintText: "请输入密码",
                          icon: Icon(Icons.lock),
                        ),
                        obscureText: true,
                        validator: (v) {
                          return v!.trim().isNotEmpty ? null : '密码不能为空';
                        },
                      ),
                    ],
                  ),
                ),
              ),
            ),
          ),
          Padding(
            padding: const EdgeInsets.symmetric(vertical: 16),
            child: Row(
              children: [
                Expanded(
                  child: Padding(
                    padding: const EdgeInsets.symmetric(horizontal: 16),
                    child: ElevatedButton(
                      onPressed: handleSubmit,
                      child: const Padding(
                        padding: EdgeInsets.all(16.0),
                        child: Text('登录'),
                      ),
                    ),
                  ),
                )
              ],
            ),
          )
        ],
      ),
    );
  }
}
```

# 7. 进度指示器

Material 组件库中提供了两种进度指示器：LinearProgressIndicator和CircularProgressIndicator，它们都可以同时用于精确的进度指示和模糊的进度指示。

## 7-1. 线形LinearProgressIndicator

LinearProgressIndicator 是一个线性进度条，它显示一个进度条，可以显示当前的进度和总进度。没有提供尺寸参数，都是取父容器的尺寸作为绘制的边界，也就是父容器的宽高。

```dart
// 模糊进度条（会执行一个动画）
LinearProgressIndicator(
  backgroundColor: Colors.grey[200],
  valueColor: AlwaysStoppedAnimation(Colors.blue),
),

// 进度条显示50%
LinearProgressIndicator(
  backgroundColor: Colors.grey[200],
  valueColor: AlwaysStoppedAnimation(Colors.blue),
  value: .5, 
)
```

## 7-2. 环形CircularProgressIndicator

CircularProgressIndicator 是一个环形进度条，它显示一个进度条，可以显示当前的进度和总进度。没有提供尺寸参数，都是取父容器的尺寸作为绘制的边界，也就是父容器的宽高。

```dart
// 模糊进度条（会执行一个旋转动画）
CircularProgressIndicator(
  backgroundColor: Colors.grey[200],
  valueColor: AlwaysStoppedAnimation(Colors.blue),
),

// 进度条显示50%，会显示一个半圆
CircularProgressIndicator(
  backgroundColor: Colors.grey[200],
  valueColor: AlwaysStoppedAnimation(Colors.blue),
  value: .5,
),
```