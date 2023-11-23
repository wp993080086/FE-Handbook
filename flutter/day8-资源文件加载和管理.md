# 📚 目录

1. [简介](#1-简介)
2. [加载assets](#2-加载assets)
---

# 1. 简介

和包管理一样，Flutter 也使用pubspec.yaml文件来管理应用程序所需的资源，如下例子:

```yaml
flutter:
  assets:
    - static/
    - static/portrait.png
```

# 2. 加载assets

您的应用可以通过AssetBundle对象访问其 asset 。有两种主要方法允许从 Asset bundle 中加载字符串或图片（二进制）文件。

## 2-1. 加载文本

有两种方式都可以

- 直接使用package:flutter/services.dart中全局静态的rootBundle对象来加载

- 通过DefaultAssetBundle来获取当前 BuildContext 的AssetBundle，使父级 widget 在运行时动态替换的不同的 AssetBundle。

```dart
import 'dart:async' show Future;
import 'package:flutter/services.dart' show rootBundle;

Future<String> loadAsset() async {
  return await rootBundle.loadString('static/config.json');
}
```

## 2-2. 加载图片

```dart
class _AboutPageState extends State<AboutPage> {
  final _title = 'About Page';
  final wordPair = WordPair.random().toString();
  @override
  Widget build(BuildContext context) {
    return Scaffold(
        appBar: AppBar(title: Text(_title)),
        body: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            TextButton(
                onPressed: () {
                  print("加载文本: ")
                }
            ),
            // 加载图片
            Image.asset(
              "static/portrait.png",
              fit: BoxFit.cover,
              width: 300,
              height: 300,
            )
          ]
        )
    );
  }
}
```