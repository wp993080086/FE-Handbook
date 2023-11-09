# 📚 目录

1. [指定库前缀](#1-指定库前缀)
2. [仅导入库的一部分](#2-仅导入库的一部分)
3. [延迟加载库](#3-延迟加载库)
---

# 1. 指定库前缀

如果导入两个具有冲突标识符的库，则 可以为一个或两个库指定前缀。例如，如果 library1 和 library2 都有一个 Element 类，如下：

```dart
import 'package:lib1/lib1.dart';
import 'package:lib2/lib2.dart' as lib2;

Element element1 = Element();

lib2.Element element2 = lib2.Element();
```

# 2. 仅导入库的一部分

如果只想使用库的一部分，则可以有选择地导入，如下：

- 只导入foo

```dart
import 'package:lib1/lib1.dart' show foo;
```

- 导入除foo以外的所有部分

```dart
import 'package:lib2/lib2.dart' hide foo
```

# 3. 延迟加载库

延迟加载允许 Web 应用按需加载库， 是否以及何时需要库。 以下是一些可能使用延迟加载的情况：

- 减少 Web 应用的初始启动时间。
- 加载很少使用的功能，例如可选屏幕和对话框。

若要延迟加载库，必须首先 使用 导入它：

```dart
import 'package:greetings/hello.dart' deferred as hello;
```

当您需要库时，请使用库的标识符进行调用：

```dart
Future<void> greet() async {
  await hello.loadLibrary();
  hello.printGreeting();
}
```

在上面的代码中， 关键字暂停执行，直到加载库。