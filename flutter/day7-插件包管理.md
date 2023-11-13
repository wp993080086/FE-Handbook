# 📚 目录

1. [简介](#1-简介)
2. [添加插件包](#2-添加插件包)
3. [其他导包方式](#3-其他导包方式)
---

# 1. 简介

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

# 2. 添加插件包

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

# 3. 其他导包方式

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

