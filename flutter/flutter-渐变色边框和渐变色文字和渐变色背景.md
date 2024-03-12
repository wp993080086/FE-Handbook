# 📚 目录

1. [介绍](#1-介绍)
2. [实现](#2-实现)
   1. [渐变色背景](#2-1-渐变色背景)
   2. [渐变色边框](#2-2-渐变色边框)
   3. [宽高由内容撑起的渐变色边框](#2-3-宽高由内容撑起的渐变色边框)
   4. [渐变色文本](#渐变色文本)
3. [完整例子](#2-完整例子)

---

# 1. 介绍

在 flutter 中，渐变有三种，线性渐变 LinearGradient、放射状渐变 RadialGradient、扇形渐变 SweepGradient。一般都是使用线性渐变 LinearGradient，下面是本文带你实现的几个渐变色效果：

# 2. 代码实现

LinearGradient 类接受三个参数：

- begin：渐变的起始点，例如 Alignment.centerLeft 表示从左侧开始渐变
- end：渐变的结束点，例如 Alignment.centerRight 表示渐变到右侧结束
- colors：渐变过程中的颜色数组

## 2-1. 渐变色背景

```dart
Container(
  height: 200,
  alignment: Alignment.center,
  margin: const EdgeInsets.symmetric(vertical: 30),
  decoration: BoxDecoration(
      borderRadius: BorderRadius.circular(12),
      gradient: const LinearGradient(
          begin: Alignment.topLeft,
          end: Alignment.bottomRight,
          colors: colorList)),
  child: const Text('渐变色背景',
      style:
          TextStyle(color: Colors.white, fontSize: 16)))
```

## 2-2. 渐变色边框

需要先安装[gradient_borders](https://pub.dev/packages/gradient_borders/install)库。

```shell
flutter pub add gradient_borders
```

```dart
import 'package:gradient_borders/box_borders/gradient_box_border.dart';

Container(
  height: 200,
  alignment: Alignment.center,
  margin: const EdgeInsets.only(bottom: 30),
  decoration: BoxDecoration(
      borderRadius: BorderRadius.circular(12),
      border: const GradientBoxBorder(
          gradient: LinearGradient(colors: colorList),
          width: 2)),
  child: const Text('渐变色边框',
      style:
          TextStyle(color: Colors.white, fontSize: 16)))
```

## 2-3. 宽高由内容撑起的渐变色边框

```dart
Container(
  padding: const EdgeInsets.all(10),
  margin: const EdgeInsets.only(bottom: 30),
  decoration: BoxDecoration(
      borderRadius: BorderRadius.circular(12),
      border: const GradientBoxBorder(
          gradient: LinearGradient(colors: colorList),
          width: 2)),
  child: const Text('宽高不固定由内容撑起来的渐变色边框',
      style:
          TextStyle(color: Colors.white, fontSize: 16)))
```

## 2-4. 渐变色文本

```dart
Container(
  height: 50,
  alignment: Alignment.center,
  child: ShaderMask(
      shaderCallback: (rect) {
        return const LinearGradient(
          begin: Alignment.topLeft,
          end: Alignment.bottomRight,
          colors: colorList,
        ).createShader(rect);
      },
      child: const Text('渐变色文字',
          style: TextStyle(
              color: Colors.white, fontSize: 16))))
```

# 3. 完整例子

```dart
import 'package:flutter/material.dart';
import 'package:gradient_borders/box_borders/gradient_box_border.dart';

/// 渐变色背景和边框和文字
class GradientColorPage extends StatelessWidget {
  const GradientColorPage({super.key});

  @override
  Widget build(BuildContext context) {
    const List<Color> colorList = [Colors.red, Colors.yellow, Colors.blue];

    return Scaffold(
        backgroundColor: Colors.black,
        body: SafeArea(
            child: Container(
                alignment: Alignment.center,
                width: MediaQuery.of(context).size.width,
                height: MediaQuery.of(context).size.height,
                padding: const EdgeInsets.all(20),
                child: Column(
                  mainAxisSize: MainAxisSize.max,
                  children: [
                    /// 渐变色背景
                    Container(
                        height: 200,
                        alignment: Alignment.center,
                        margin: const EdgeInsets.symmetric(vertical: 30),
                        decoration: BoxDecoration(
                            borderRadius: BorderRadius.circular(12),
                            gradient: const LinearGradient(
                                begin: Alignment.topLeft,
                                end: Alignment.bottomRight,
                                colors: colorList)),
                        child: const Text('渐变色背景',
                            style:
                                TextStyle(color: Colors.white, fontSize: 16))),

                    /// 渐变色边框
                    Container(
                        height: 200,
                        alignment: Alignment.center,
                        margin: const EdgeInsets.only(bottom: 30),
                        decoration: BoxDecoration(
                            borderRadius: BorderRadius.circular(12),
                            border: const GradientBoxBorder(
                                gradient: LinearGradient(colors: colorList),
                                width: 2)),
                        child: const Text('渐变色边框',
                            style:
                                TextStyle(color: Colors.white, fontSize: 16))),

                    /// 宽高不固定由内容撑起来的渐变色边框
                    Container(
                        padding: const EdgeInsets.all(10),
                        margin: const EdgeInsets.only(bottom: 30),
                        decoration: BoxDecoration(
                            borderRadius: BorderRadius.circular(12),
                            border: const GradientBoxBorder(
                                gradient: LinearGradient(colors: colorList),
                                width: 2)),
                        child: const Text('宽高不固定由内容撑起来的渐变色边框',
                            style:
                                TextStyle(color: Colors.white, fontSize: 16))),

                    /// 渐变色文字
                    Container(
                        height: 50,
                        alignment: Alignment.center,
                        child: ShaderMask(
                            shaderCallback: (rect) {
                              return const LinearGradient(
                                begin: Alignment.topLeft,
                                end: Alignment.bottomRight,
                                colors: colorList,
                              ).createShader(rect);
                            },
                            child: const Text('渐变色文字',
                                style: TextStyle(
                                    color: Colors.white, fontSize: 16))))
                  ],
                ))));
  }
}
```
