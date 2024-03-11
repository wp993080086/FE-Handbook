# 📚 目录

1. [介绍](#1-介绍)
2. [用法](#2-用法)
3. [例子](#3-例子)
---

# 1. 介绍

在APP中，常常需要一个广播机制，用以跨页面的事件通知。比如父子组件通信，跨页面通信，全局事件监听等。事件总线一般实现了订阅者模式，包含发布者和订阅者两种角色，能够经过事件总线来触发事件和监听事件。在Flutter中，事件总线一般使用[event_bus](https://pub.dev/packages/event_bus)来实现。

# 2. 用法

- 安装插件

```shell
flutter pub add event_bus
```

- 创建事件总线

```dart
import 'package:event_bus/event_bus.dart';

EventBus eventBus = EventBus();
```

- 定义事件

```dart
class UserLoggedInEvent {
  User user;

  UserLoggedInEvent(this.user);
}

class NewOrderEvent {
  Order order;

  NewOrderEvent(this.order);
}
```

- 注册单个监听器

```dart
eventBus.on<UserLoggedInEvent>().listen((event) {
  // 所有事件的类型都是UserLoggedInEvent（或它的子类型）
  print(event.user);
});
```

- 为所有事件注册监听器

```dart
eventBus.on().listen((event) {
  // 打印运行时类型（这样的设置可以用于日志记录）
  print(event.runtimeType);
});
```

- 发布事件

```dart
User myUser = User('Mickey');
eventBus.fire(UserLoggedInEvent(myUser));
```

- 取消监听

```dart
eventBusExample.cancel();
```

# 3. 例子

下面是一个例子，父组件修改索引，子组件监听索引变化而修改背景颜色：

1. 在lib文件夹下创建utlis目录，新建一个`eventBus.dart`文件，内容如下：

```dart
import 'package:event_bus/event_bus.dart';

EventBus eventBus = EventBus();

/// 通知测试
class BusNotificationTest {
  int index;
  BusNotificationTest(this.index);
}
```

2. 父组件引入

```dart
import 'package:flutter/material.dart';
import 'package:demo3/widget/appBar.dart';
import 'package:demo3/utils/eventBus.dart';
import 'eventBusChild.dart';

/// 事件Bus通知父类
class EventBusParentPage extends StatefulWidget {
  const EventBusParentPage({super.key});

  @override
  EventBusParentPageState createState() => EventBusParentPageState();
}

class EventBusParentPageState extends State<EventBusParentPage> {
  /// 颜色索引
  int index = 0;

  /// 通知变色
  void handleColorChange() {
    index++;
    if (index > 2) {
      index = 0;
    }
    eventBus.fire(BusNotificationTest(index));
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: wangAppBar(title: 'Event Bus'),
      body: Column(
        children: [
          Expanded(
              child: Container(
                  alignment: Alignment.center,
                  child: TextButton(
                      onPressed: handleColorChange,
                      child: const Text('点击变色',
                          style:
                              TextStyle(color: Colors.black, fontSize: 20))))),
          const EventBusChildPage()
        ],
      ),
    );
  }
}
```

3. 子组件监听

```dart
import 'dart:async';
import 'package:flutter/material.dart';
import 'package:demo3/utils/eventBus.dart';

/// 事件Bus通知子类
class EventBusChildPage extends StatefulWidget {
  const EventBusChildPage({super.key});

  @override
  EventBusChildPageState createState() => EventBusChildPageState();
}

class EventBusChildPageState extends State<EventBusChildPage> {
  /// 颜色列表
  List<Color> colorList = [Colors.red, Colors.yellow, Colors.blue];

  /// 刷新监听
  late StreamSubscription<BusNotificationTest> eventBusExample;

  /// 背景颜色
  late Color bgColor;

  @override
  void initState() {
    super.initState();
    bgColor = colorList[0];
    // 监听颜色索引变化
    eventBusExample = eventBus.on<BusNotificationTest>().listen((event) {
      setState(() {
        bgColor = colorList[event.index];
      });
    });
  }

  @override
  void dispose() {
    eventBusExample.cancel();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return SizedBox(
        width: MediaQuery.of(context).size.width,
        height: MediaQuery.of(context).size.height / 2,
        child: Container(color: bgColor, child: null));
  }
}
```