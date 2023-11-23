# 📚 目录

1. [介绍](#1-介绍)
2. [APP目录](#2-app目录)
3. [使用path_provider存储](#3-使用path_provider存储)
4. [使用shared_preferences存储](#4-使用shared_preferences存储)
---

# 1. 介绍

Dart的 IO 库包含了文件读写的相关类，它属于 Dart 语法标准的一部分，所以通过 Dart IO 库，无论是 Dart VM 下的脚本还是 Flutter，都是通过 Dart IO 库来操作文件的，不过和 Dart VM 相比，Flutter 有一个重要差异是文件系统路径不同，这是因为Dart VM 是运行在 PC 或服务器操作系统下，而 Flutter 是运行在移动操作系统中，他们的文件系统会有一些差异。

# 2. APP目录

Android 和 iOS 的应用存储目录不同，PathProvider插件提供了一种平台透明的方式来访问设备文件系统上的常用位置。该类当前支持访问两个文件系统位置：

- 临时目录：使用 getTemporaryDirectory() 来获取临时目录； 系统可随时清除临时目录的文件。在 iOS 上，这对应于NSTemporaryDirectory()返回的值。在 Android上，这是getCacheDir() 返回的值。
- 文档目录：使用getApplicationDocumentsDirectory()来获取应用程序的文档目录，该目录用于存储只有自己可以访问的文件。只有当应用程序被卸载时，系统才会清除该目录。在 iOS 上，这对应于NSDocumentDirectory。在 Android 上，这是AppData目录。
- 外部存储目录：使用getExternalStorageDirectory()来获取外部存储目录，如 SD 卡；由于 iOS不支持外部目录，所以在 iOS 下调用该方法会抛出UnsupportedError异常，而在 Android 下结果是Android SDK 中getExternalStorageDirectory的返回值。

一旦Flutter 应用程序有一个文件位置的引用，就可以使用 dart:io API来执行对文件系统的读/写操作。

# 3. 使用path_provider存储

先导入依赖，然后运行flutter pub get

```yaml
dependencies:
  flutter:
    sdk: flutter
  path_provider: ^2.0.14  # 版本号可以根据需要选择
```

- 例子：

```dart
import 'dart:io';
import 'dart:async';
import 'package:flutter/material.dart';
import 'package:path_provider/path_provider.dart';

/// 定义
class HomePage extends StatefulWidget {
  const HomePage({super.key});

  @override
  State<HomePage> createState() => HomePageState();
}

/// 实现
class HomePageState extends State<HomePage> {
  int size = 0;

  @override
  void initState() {
    super.initState();
    // 从文件读取点击次数
    handleReadCounter().then((int value) {
      setState(() {
        size = value;
      });
    });
  }
  /// 获取应用目录
  Future<File> handleGetLocalFile() async {
    String dir = (await getApplicationDocumentsDirectory()).path;
    debugPrint(dir);
    return File('$dir/counter.txt');
  }
  /// 读取文件 获取点击次数
  Future<int> handleReadCounter() async {
    try {
      File file = await handleGetLocalFile();
      String contents = await file.readAsString();
      return int.parse(contents);
    } on FileSystemException {
      return 0;
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
        appBar: AppBar(
          title: const Text('Flutter Home'),
        ),
        body: Container(
          alignment: Alignment.center,
          child: Padding(
            padding: const EdgeInsets.all(16.0),
            child: Column(
              children: [
                Text(size.toString(), style: const TextStyle(fontSize: 24.0),)
              ],
            ),
          ),
        ),
    floatingActionButton: FloatingActionButton(
      onPressed: () async {
        setState(() {
          size++;
        });
        // 将点击次数以字符串类型写到文件中
        await (await handleGetLocalFile()).writeAsString('$size');
      },
      child:  const Text('添加'),
    ),);
  }
}
```

# 4. 使用shared_preferences存储

在实际开发中，如果要存储一些简单的数据，使用shared_preferences插件会比较简单。需要先安装：flutter pub add shared_preferences

```dart
import 'package:flutter/material.dart';
import 'package:shared_preferences/shared_preferences.dart';

/// 定义
class HomePage extends StatefulWidget {
  const HomePage({super.key});

  @override
  State<HomePage> createState() => HomePageState();
}

/// 实现
class HomePageState extends State<HomePage> {
  int size = 0;

  @override
  void initState() {
    super.initState();
    // 从文件读取点击次数
    handleLoadCounter();
  }
  
  /// 读取数据
  Future<void> handleLoadCounter() async {
    final prefs = await SharedPreferences.getInstance();
    setState(() {
      size = prefs.getInt('counter') ?? 0;
    });
  }
  
  /// 存储数据
  Future<void> handleIncrementCounter() async {
    final prefs = await SharedPreferences.getInstance();
    setState(() {
      size = (prefs.getInt('counter') ?? 0) + 1;
      prefs.setInt('counter', size);
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
        appBar: AppBar(
          title: const Text('Flutter Home'),
        ),
        body: Container(
          alignment: Alignment.center,
          child: Padding(
            padding: const EdgeInsets.all(16.0),
            child: Column(
              children: [
                Text(size.toString(), style: const TextStyle(fontSize: 24.0),)
              ],
            ),
          ),
        ),
    floatingActionButton: FloatingActionButton(
      onPressed: () async {
        setState(() {
          size++;
        });
        // 将点击次数存储
        handleIncrementCounter();
      },
      child:  const Text('添加'),
    ),);
  }
}
```