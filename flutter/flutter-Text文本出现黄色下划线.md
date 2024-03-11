# 📚 目录

1. [现象](#1-现象)
2. [原因](#2-原因)
3. [解决方法](#3-解决方法)
---

# 1. 现象

这天我正在写页面功能，突然发现我的 Text 文字出现了奇怪的样式，具体如下：

- 文字下面出现了双层黄色下划线
- 文字的空格变得很大，文字的间距也变得很大

我百思不得其解，经过搜索，发现原因如下，特此记录

# 2. 原因

原来，因为我的页面不是 Material 风格的组件，所以 Text 的样式是默认的，所以出现了上述问题。解决方法也很简单，只需要在页面中使用 Scaffold 做为根节点，或者使用 Material 小部件包裹一下即可。这里注意，Material 小部件自带背景色，记得设置为透明色。

# 3. 解决方法

- 方法一：使用 Scaffold 做为根节点

```dart
Scaffold(body: Center(child: Text('Hello World')));
```

- 方法二：使用 Material 小部件包裹

```dart
Material(color: Colors.transparent, child: Text('Hello World'));
```
