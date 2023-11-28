# 📚 目录

1. [介绍](#1-介绍)
2. [使用](#2-使用)
3. [获取当前区域Locale](#3-获取当前区域locale)
4. [监听语言切换](#4-监听语言切换)
5. [实现国际化](#5-实现国际化)
    1. [添加依赖](#5-1-添加依赖)
    2. [创建arb文件](#5-2-创建arb文件)
    3. [添加provider状态管理](#5-3-添加provider状态管理)
    4. [完成切换](#5-4-完成切换)
---

# 1. 介绍

默认情况下，Flutter SDK中的组件仅提供美国英语本地化资源（主要是文本）。要添加对其他语言的支持，应用程序须添加一个名为“flutter_localizations”的包依赖，然后还需要在MaterialApp中进行一些配置。

# 2. 使用

首先安装flutter_localizations

```shell
flutter pub add flutter_localizations
```

然后修改main.dart，指定MaterialApp的localizationsDelegates和supportedLocales。

```dart
import 'package:flutter_localizations/flutter_localizations.dart';

MaterialApp(
 localizationsDelegates: [
   // 本地化的代理类
   GlobalMaterialLocalizations.delegate,
   GlobalWidgetsLocalizations.delegate,
 ],
 supportedLocales: [
    const Locale('en', 'US'), // 美国英语
    const Locale('zh', 'CN'), // 中文简体
    // 其他Locales
  ],
)
```

# 3. 获取当前区域Locale

Locale类是用来标识用户的语言环境的，它包括语言和国家两个标志，如下：

```dart
const Locale('zh', 'CN')
```

可以通过以下方式来获取应用的当前区域Locale：

```dart
Locale myLocale = Localizations.localeOf(context);
```

# 4. 监听语言切换

当我们更改系统语言设置时，APP中的Localizations组件会重新构建，Localizations.localeOf(context) 获取的Locale就会更新，最终界面会重新build达到切换语言的效果。

```dart
class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      localizationsDelegates: [
        GlobalMaterialLocalizations.delegate,
        GlobalWidgetsLocalizations.delegate,
      ],
      // 语言切换回调
      localeResolutionCallback: (locales, supportedLocales) {
        // 在这里自定义语言环境
        if(true) {
          return const Locale('zh', 'CN');
        } else {
          return const Locale('en', 'US');
        }
      },
      home: MyHomePage(),
    );
  }
}
```

# 5. 实现国际化

这里我们使用intl来实现国际化。

## 5-1. 添加依赖

在pubspec.yaml里添加`intl`，`flutter_localizations`依赖。以及开启生成器。

```yaml
dependencies:
  flutter:
    sdk: flutter
  # 国际化
  intl: ^0.18.1 # add
  flutter_localizations: # add
    sdk: flutter # add
# The following section is specific to Flutter packages.
flutter:
  generate: true # add
```

## 5-2. 创建arb文件

在Android studio内直接搜索flutter Intl，添加插件，安装完成后，点击顶部工具栏，选择Tools =》 flutter intl =》 lnitalize for the project，初始化一下。插件会默认创建l10n文件夹，generated文件夹，里面包含messages_all.dart、messages_en.dart以及intl_en.arb文件。

如果要新建语言类型，点击顶部工具栏，选择Tools =》 flutter intl =》 add locale，输入语言类型，插件会自动创建对应语言的arb文件。

然后就是填写你的文案字段了。

- intl_zh.arb

```arb
{
  "appName": "例子",
  "appTitle": "你好世界！"
}
```

- intl_en.arb

```arb
{
  "appName": "例子",
  "appTitle": "你好世界！"
}
```

## 5-3. 添加provider状态管理

这里为什么要添加provider呢，因为我们切换语言后的当前语言环境，我们要保存住，然后通过provider通知到其他组件。这样才是一个完整的国际化。

在pubspec.yaml里添加`provider`依赖，然后pub get一下。

```yaml
# 状态管理：https://pub.dev/packages/provider
provider: ^6.1.1
```

在lib文件夹下创建一个provider文件夹，在provider文件夹下创建language.dart文件用来管理语言，添加如下代码：

```dart
import 'package:flutter/material.dart';

class LanguageProvider extends ChangeNotifier {
  /// 默认语言
  Locale currentLanguage = const Locale('zh', 'CN');
  /// 获取当前语言
  Locale get getCurrentLanguage => currentLanguage;
  /// 修改语言
  void changeLanguage(Locale newLanguage) {
    currentLanguage = newLanguage;
    notifyListeners();
  }
}
```

然后修改main.dart的代码，添加语言包和一些代理，以及provider的状态。如下：

```dart
import 'package:flutter/material.dart';
import 'views/home/view.dart';
import 'package:demo2/generated/l10n.dart';
import 'package:flutter_localizations/flutter_localizations.dart';
import 'package:demo2/provider/language.dart';
import 'package:provider/provider.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return MultiProvider(
      // 所有要使用的provider都放到这里
      providers: [
        ChangeNotifierProvider(create: (context) => LanguageProvider())
      ],
      builder: (context, child) {
        return MaterialApp(
          title: 'Flutter Demo',
          theme: ThemeData(
            colorScheme: ColorScheme.fromSeed(seedColor: Colors.deepPurple),
            useMaterial3: true,
          ),
          // 设置当前语言
          locale: Provider.of<LanguageProvider>(context).currentLanguage,
          // 设置语言类型有哪几种
          supportedLocales: S.delegate.supportedLocales,
          // 初始化指定语言
          localizationsDelegates: const [
            // 本地化的代理类
            S.delegate,
            // 指定本地化的字符串和一些其他的值
            GlobalMaterialLocalizations.delegate,
            // 对应的Cupertino风格
            GlobalCupertinoLocalizations.delegate,
            // 指定默认的文本排列方向, 由左到右或由右到左
            GlobalWidgetsLocalizations.delegate
          ],
          home: const HomePage(),
        );
      },
    );
  }
}
```

## 5-4. 完成切换

然后在页面中使用语言切换

```dart
import 'package:flutter/material.dart';
import 'package:demo2/provider/language.dart';
import 'package:demo2/config/config.dart';
import 'package:demo2/generated/l10n.dart';
import 'package:provider/provider.dart';

class HomePage extends StatefulWidget {
  const HomePage({super.key});

  @override
  State<HomePage> createState() => HomePageState();
}

class HomePageState extends State<HomePage> {
  int self = 1;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        // 这样使用
        title: Text(S.of(context).appTitle),
      ),
      body: Center(
        child: Text('${S.of(context).appName}：$self'),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () {
          setState(() {
            self++;
          });
          // 获取语言管理provider 并且不监听变化
          final languageProvider = Provider.of<LanguageProvider>(context, listen: false);
          late Locale newLanguage;
          if (self % 2 == 0) {
            newLanguage = const Locale('en', 'US');
          } else {
            newLanguage = const Locale('zh', 'CN');
          }
          languageProvider.changeLanguage(newLanguage);
        },
        child: const Icon(Icons.add),
      ),
    );
  }
}
```