# 📚 目录

1. [介绍](#1-介绍)
2. [属性介绍](#2-属性介绍)
3. [使用](#3-使用)

---

# 1. 介绍

在 Flutter 的开发过程中，经常会遇到图片的显示和加载处理，通常显示一个图片，都有很多细节需要处理，比如图片的加载、缓存、错误处理、图片的压缩、图片的格式转换等，如果每个地方都手动处理，就太麻烦了，这时候就可以使用糖果大佬的插件 extended_image，它是官方 Image 的扩展三方库，不但支持图片加载以及失败显示，缓存网络图片，缩放拖拽图片，图片浏览等，还支持滑动退出页面，编辑图片(裁剪旋转翻转)，保存，绘制自定义效果等功能。

# 2. 属性介绍

| 属性                        | 描述                           | 值                     |
| --------------------------- | ------------------------------ | ---------------------- |
| url                         | 网络请求地址                   | required               |
| key                         | 唯一标识符                     | -                      |
| semanticLabel               | 语义标签                       | -                      |
| excludeFromSemantics        | 是否排除语义                   | false                  |
| width                       | 宽度                           | -                      |
| height                      | 高度                           | -                      |
| color                       | 颜色                           | -                      |
| opacity                     | 透明度动画                     | -                      |
| colorBlendMode              | 颜色混合模式                   | -                      |
| fit                         | 图片适应方式                   | -                      |
| alignment                   | 对齐方式                       | Alignment.center       |
| repeat                      | 图片重复方式                   | ImageRepeat.noRepeat   |
| centerSlice                 | 九宫格切片区域                 | -                      |
| matchTextDirection          | 是否匹配文本方向               | false                  |
| gaplessPlayback             | 是否无缝播放                   | false                  |
| filterQuality               | 滤镜质量                       | FilterQuality.low      |
| loadStateChanged            | 图片加载状态回调               | Function               |
| shape                       | 盒子形状                       | -                      |
| border                      | 盒子边框                       | -                      |
| borderRadius                | 圆角半径                       | -                      |
| clipBehavior                | 裁剪行为                       | Clip.antiAlias         |
| enableLoadState             | 是否启用加载状态               | true                   |
| beforePaintImage            | 图片绘制前回调                 | -                      |
| afterPaintImage             | 图片绘制后回调                 | -                      |
| mode                        | 扩展图片模式（默认/手势/编辑） | ExtendedImageMode.none |
| enableMemoryCache           | 是否启用内存缓存               | true                   |
| clearMemoryCacheIfFailed    | 加载失败时是否清除内存缓存     | true                   |
| onDoubleTap                 | 双击事件回调                   | -                      |
| initGestureConfigHandler    | 初始化手势配置回调             | -                      |
| enableSlideOutPage          | 是否启用滑动退出页面           | false                  |
| constraints                 | 约束条件                       | -                      |
| cancelToken                 | 用于取消请求的 Token           | CancellationToken()    |
| retries                     | 请求尝试次数                   | 3                      |
| timeLimit                   | 请求超时时间                   | -                      |
| headers                     | 请求头                         | -                      |
| cache                       | 是否缓存                       | true                   |
| scale                       | 图片缩放比例                   | 1.0                    |
| timeRetry                   | 请求重试间隔                   | milliseconds: 100      |
| extendedImageEditorKey      | 扩展图片编辑器键               | -                      |
| initEditorConfigHandler     | 初始化编辑器配置回调           | -                      |
| heroBuilderForSlidingPage   | 滑动退出页面时的英雄构建器     | -                      |
| clearMemoryCacheWhenDispose | 销毁时是否清除内存缓存         | false                  |
| handleLoadingProgress       | 是否处理加载进度               | false                  |
| extendedImageGestureKey     | 扩展图片手势键                 | -                      |
| cacheWidth                  | 缓存宽度                       | -                      |
| cacheHeight                 | 缓存高度                       | -                      |
| isAntiAlias                 | 是否开启抗锯齿                 | false                  |
| cacheKey                    | 缓存键                         | -                      |
| printError                  | 是否打印错误信息               | true                   |
| compressionRatio            | 压缩比例                       | -                      |
| maxBytes                    | 最大字节数                     | -                      |
| cacheRawData                | 是否缓存原始数据               | false                  |
| imageCacheName              | 图片缓存名称                   | -                      |
| cacheMaxAge                 | 缓存最大寿命                   | -                      |
| layoutInsets                | 布局插入区域                   | EdgeInsets.zero        |

# 3. 使用例子

本例子包含如下：

- 网络图片加载
- 图片加载中、加载成功、加载失败的处理
- 放大查看

更详细的使用方式，请参考[extended_image 文档](https://github.com/fluttercandies/extended_image)。

```dart
ExtendedImage.network(
  imagePath,
  width: double.infinity,
  fit: BoxFit.fitHeight,
  cache: true,
  mode: ExtendedImageMode.gesture,
  initGestureConfigHandler: (state) {
    return GestureConfig(
      // 缩放最小值
      minScale: 0.8,
      // 缩放动画最小值 当缩放结束时回到 minScale 值
      animationMinScale: 0.8,
      // 缩放最大值
      maxScale: 2.0,
      // 缩放动画最大值 当缩放结束时回到 maxScale 值
      animationMaxScale: 3.5,
      // 缩放拖拽速度
      speed: 1.0,
      // 拖拽惯性速度
      inertialSpeed: 100.0,
      initialScale: 1.0,
      // 是否使用 ExtendedImageGesturePageView 展示图片
      inPageView: false,
      // 当图片的初始化缩放大于 1.0 的时候，根据相对位置初始化图片
      initialAlignment: InitialAlignment.center
    );
  }，
  /// 加载状态回调
  loadStateChanged: (ExtendedImageState state) {
    switch (state.extendedImageLoadState) {
      /// 加载中
      case LoadState.loading:
        // 自己写的加载中的Loading组件
        return LoadWait();

      /// 加载成功
      case LoadState.completed:
        return state.completedWidget;

      /// 加载失败
      case LoadState.failed:
        // 自己写的加载失败的组件 并且把重试的回调传递过去
        return LoadFailed(callback: state.reLoadImage);
    }
  }
)
```

# 4. 完整封装

封装思路：关键参数必传，次要参数有默认值，统一加载中、加载失败、加载成功的状态，统一处理图片加载失败的情况，统一处理图片加载成功的情况

封装代码：

> 注意，这里使用了屏幕适配插件 flutter_screenutil，全局使用.w 做尺寸单位。

```dart
import 'package:flutter/material.dart';
import 'package:extended_image/extended_image.dart';
import 'package:flutter_screenutil/flutter_screenutil.dart';

/// 加载网络图片
class MyLoadNetworkImage extends StatelessWidget {
  /// 图片地址
  final String imageUrl;

  /// 背景色
  final Color bgColor;

  /// 失败回调
  final Function()? onFailed;

  /// 成功回调
  final Function()? onSucceed;

  /// 宽度
  final double? width;

  /// 高度
  final double? height;

  /// 是否缓存
  final bool cache;

  /// 图片适应方式
  final BoxFit fit;

  /// 图片模式
  final ExtendedImageMode mode;

  /// 图片手势设置 需配合 mode = ExtendedImageMode.gesture
  final GestureConfig Function(ExtendedImageState)? initGestureConfigHandler;

  /// 图片Load动画
  final bool loadAnime;

  /// 压缩率
  final double compressionRatio;

  /// 缓存宽度
  final int? cacheWidth;

  /// 缓存高度
  final int? cacheHeight;

  /// Load图宽高
  final double loadBoxWAndH;

  const MyLoadNetworkImage(
      {Key? key,
      required this.imageUrl,
      this.onFailed,
      this.onSucceed,
      this.bgColor = Colors.black,
      this.width,
      this.height,
      this.cache = true,
      this.fit = BoxFit.fitHeight,
      this.mode = ExtendedImageMode.none,
      this.initGestureConfigHandler,
      this.loadAnime = false,
      this.compressionRatio = 1.0,
      this.cacheWidth,
      this.cacheHeight,
      this.loadBoxWAndH = 64})
      : super(key: key);

  @override
  Widget build(BuildContext context) {
    return ExtendedImage.network(
      imageUrl,
      width: width,
      height: height,
      fit: fit,
      cache: cache,
      initGestureConfigHandler: initGestureConfigHandler,
      mode: mode,
      compressionRatio: compressionRatio,
      cacheMaxAge: Duration(hours: 24),
      cacheWidth: cacheWidth,
      cacheHeight: cacheHeight,

      /// 加载状态回调
      loadStateChanged: (ExtendedImageState state) {
        switch (state.extendedImageLoadState) {
          /// 加载中
          case LoadState.loading:
            return loadAnime
                ? Container(
                    constraints: BoxConstraints.expand(),
                    alignment: Alignment.center,
                    child: SvgPicture.asset(xxxxx.loadingImage),
                  )
                : WaitOrLoadWidget(radius: loadBoxWAndH);

          /// 加载成功
          case LoadState.completed:
            if (onSucceed != null) {
              onSucceed!();
            }
            return state.completedWidget;

          /// 加载失败
          case LoadState.failed:
            if (onFailed != null) {
              onFailed!();
            }
            return LoadFailedWidget(callback: state.reLoadImage);
        }
      },
    );
  }
}

/// 等待中或加载中的组件
class WaitOrLoadWidget extends StatelessWidget {
  final double radius;
  const WaitOrLoadWidget({Key? key, this.radius = 64}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    double myRadius = radius == 64 ? 64.w : radius;

    return Container(
      color: Colors.black,
      alignment: Alignment.center,
      constraints: BoxConstraints.expand(),
      child: SizedBox(
        width: myRadius,
        height: myRadius,
        child: SvgPicture.asset(xxxxx.loadImage),
      ),
    );
  }
}

/// 加载失败的组件
class LoadFailedWidget extends StatelessWidget {
  final double radius;

  /// 回调函数
  final Function() callback;

  const LoadFailedWidget({Key? key, required this.callback, this.radius = 64})
      : super(key: key);

  @override
  Widget build(BuildContext context) {
    double myRadius = radius == 64 ? 64.w : radius;

    return Container(
      color: Colors.black,
      alignment: Alignment.center,
      constraints: BoxConstraints.expand(),
      child: GestureDetector(
        onTap: callback,
        child: SizedBox(
          width: myRadius,
          height: myRadius,
          child: SvgPicture.asset(xxxxx.failedImage),
        ),
      ),
    );
  }
}
```