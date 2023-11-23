# 📚 目录

1. [简介](#1-简介)
    1. [dart包](#1-1-dart包)
    2. [插件包](#1-2-插件包)
2. [依赖配置pubspec.yaml](#2-依赖配置pubspecyaml)
3. [添加插件包](#3-添加插件包)
4. [其他导包方式](#4-其他导包方式)
5. [获取平台信息](#5-获取平台信息)
6. [插件合集](#6-插件合集)
---

# 1. 简介

Flutter的包分为两类

## 1-1. Dart包

包就是可以复用的模块化代码。其中一些可能包含Flutter的特定功能，因此对Flutter框架具有依赖性，这种包仅用于Flutter，例如fluro包。而一个最小的Package包括：

- pubspec.yaml文件：声明了Package的名称、版本、作者等的元数据文件
- lib文件夹：包括包中公开的代码，最少应有一个<package-name>.dart文件

## 1-2. 插件包

一种专用的Dart包，其中包含用Dart代码编写的API，以及针对Android（使用Java或Kotlin）和针对iOS（使用OC或Swift）平台的特定实现，也就是说插件包括原生代码，一个具体的例子是battery 插件包。因为Flutter 本质上只是一个 UI 框架，运行在宿主平台之上，Flutter 本身是无法提供一些系统能力，比如使用蓝牙、相机、GPS等，因此要在 Flutter 中调用这些能力就必须和原生平台进行通信。Flutter与原生之间的通信本质上是一个远程调用（RPC），通过消息传递实现，步骤如下：

- 应用的Flutter部分通过平台通道（platform channel）将调用消息发送到宿主应用
- 宿主监听平台通道，并接收该消息。然后它会调用该平台的API，并将响应发送回Flutter。

由于插件编写涉及具体平台的开发知识，比如 image_picker 插件需要开发者在 iOS 和 Android 平台上分别实现图片选取和拍摄的功能，因此需要开发者熟悉原生开发

# 2. 依赖配置pubspec.yaml

Flutter 使用配置文件pubspec.yaml（位于项目根目录）来管理第三方依赖包。如下示例：

- name：应用或包名称
- description: 应用或包的描述、简介
- version：应用或包的版本号
- dependencies：应用或包依赖的其他包或插件
- dev_dependencies：开发环境依赖的工具包（而不是flutter应用本身依赖的包）
- flutter：flutter相关的配置选项

```yaml
name: flutter_in_action
description: First Flutter Application.

version: 1.0.0+1

dependencies:
  flutter:
    sdk: flutter
  cupertino_icons: ^0.1.2

dev_dependencies:
  flutter_test:
    sdk: flutter
    
flutter:
  uses-material-design: true
```

如果我们的Flutter应用本身依赖某个包，我们需要将所依赖的包添加到dependencies下

# 3. 添加插件包

flutter应用类似npm的包管理仓库是：[Pub](https://pub.dev/)，是 Google 官方的 Dart Packages 仓库。下面是一个使用插件的示例：

- 先在pubspec.yaml中，添加示例包`english_words`和版本号

```yaml
dependencies:
  flutter:
    sdk: flutter
  # 新添加的依赖
  english_words: ^4.0.0
```

- 然后，在Android Studio的编辑器视图中，点击右上角的Pub get按钮，等待安装完毕

- 然后在要使用的页面引入`english_words`包

```dart
import 'package:english_words/english_words.dart';
```

- 使用

```dart
class _AboutPageState extends State<AboutPage> {
  final _title = 'About Page';
  // 使用包生成随机名字
  final wordPair = WordPair.random().toString();
  @override
  Widget build(BuildContext context) {
    var args = ModalRoute.of(context)?.settings.arguments;
    return Scaffold(
        appBar: AppBar(title: Text(_title)),
        body: Center(
            child: ElevatedButton(
              onPressed: () => {
                Navigator.pop(context, '我是返回值$args')
              },
              child: Text(widget.text + wordPair),
        )));
  }
}
```

# 4. 其他导包方式

- 本地包，绝对相对路径均可

```yaml
dependencies:
	pkg1:
        path: ../../code/pkg1
```

- git包根目录

```yaml
dependencies:
  pkg1:
    git:
      url: git://github.com/xxx/pkg1.git
```

- git包非根目录

```yaml
dependencies:
  package1:
    git:
      url: git://github.com/flutter/packages.git
      path: packages/package1        
```

# 5. 获取平台信息

在 Flutter 中我们想根据宿主平台添加一些差异化的功能，因此 Flutter 中提供了一个全局变量 defaultTargetPlatform 来获取当前应用的平台信息。

```dart
import 'package:flutter/foundation.dart';

/// 判断是那个系统
if (defaultTargetPlatform == TargetPlatform.iOS) {
  // 如果是 iOS 平台的逻辑
} else if (defaultTargetPlatform == TargetPlatform.android) {
  // 如果是 Android 平台的逻辑
} else {
  // 其他平台的处理
}
```

# 6. 插件合集

- 官方实现：https://github.com/flutter/plugins/tree/master/packages

- 社区：https://pub.dev/