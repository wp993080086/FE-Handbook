# 📚 目录

1. [介绍](#1-介绍)
2. [组合多个组件](#2-组合多个组件)
---

# 1. 介绍

默认情况下，Flutter SDK中的组件仅提供美国英语本地化资源（主要是文本）。要添加对其他语言的支持，应用程序须添加一个名为“flutter_localizations”的包依赖，然后还需要在MaterialApp中进行一些配置。

# 2. 使用

首先安装flutter_localizations

···shell
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
      // 回调
      localeResolutionCallback: (localeResolutionCallback) async {
        // 在这里自定义语言环境
        return const Locale('zh', 'CN');
      },
      home: MyHomePage(),
    );
  }
}
```

# 5. 实现国际化

在Android studio内直接搜索flutter Intl，添加插件，安装完成后，点击顶部工具栏，选择Tools =》 flutter intl =》 lnitalize for the project，初始化一下。插件会默认创建messages_all.dart、messages_en.dart以及intl_en.arb文件。如果要新建语言类型，点击顶部工具栏，选择Tools =》 flutter intl =》 add locale，输入语言类型，插件会自动创建对应语言的arb文件。

```arb
{
  "appName": "例子",
  "appTitle": "你好世界！"
}
```

然后，在pubspec.yaml里添加依赖

```yaml
dependencies:
  flutter:
    sdk: flutter
  # 国际化
  intl: ^0.18.1
  flutter_localizations:
    sdk: flutter
```

使用如下：

```dart
import 'package:demo1/generated/l10n.dart';

appBar: AppBar(
  backgroundColor: Theme.of(context).colorScheme.inversePrimary,
  // 使用
  title: Text(S.of(context).appName),
)
```