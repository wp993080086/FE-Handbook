# 📚 目录

1. [介绍](#1-介绍)
2. [例子](#2-例子)

---

# 1. 介绍

本节主要介绍如何制作一个淡入淡出的 Fade 过渡 Banner 切换。Fade 过渡通常指的是当元素（如图片、文本框等）显示或隐藏时，元素的透明度会逐渐变化，从而实现平滑的视觉过渡效果。这种效果可以提升用户的体验，使得页面的交互更加自然和流畅。

# 2. 例子

- main.dart

```dart
import 'package:xxx/fadeBanner.dart';
import 'package:flutter/material.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      home: FadeBannerWidget()
    );
  }
}
```

- fadeBanner.dart

```dart
import 'package:flutter/material.dart';
import 'dart:async';

class FadeBannerWidget extends StatefulWidget {
  const FadeBannerWidget({super.key});

  @override
  FadeBannerWidgetState createState() => FadeBannerWidgetState();
}

class FadeBannerWidgetState extends State<FadeBannerWidget>
    with SingleTickerProviderStateMixin {
  /// 图片列表
  final List<String> imageList = [
    'https://xxxxx.jpg',
    'https://xxxxx.jpg',
    'https://xxxxx.jpg',
    'https://xxxxx.jpg',
    'https://xxxxx.jpg'
  ];

  /// Fade持续时长
  static const fadeDuration = Duration(milliseconds: 300);

  /// Banner切换时长
  static const bannerDuration = Duration(milliseconds: 3000);

  /// 当前图片索引
  int currentImageIndex = 0;

  /// 动画控制器
  late AnimationController animeController;

  /// 图片定时器
  late Timer imageTimer;

  @override
  void initState() {
    super.initState();
    // 初始化动画控制器
    animeController = AnimationController(vsync: this, duration: fadeDuration);
    // 动画发射
    animeController.forward();
    // 初始化图片定时器
    imageTimer = Timer.periodic(bannerDuration, (_) {
      currentImageIndex = (currentImageIndex + 1) % imageList.length;
      setState(() {});
    });
  }

  @override
  void dispose() {
    animeController.dispose();
    imageTimer.cancel();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
        body: Container(
            alignment: Alignment.center,
            color: const Color(0xFF3F3F3F),
            child: AnimatedSwitcher(
                duration: bannerDuration,
                child: FadeTransition(
                    opacity: animeController,
                    key: ValueKey<int>(currentImageIndex),
                    child: Image.network(imageList[currentImageIndex])))));
  }
}
```
