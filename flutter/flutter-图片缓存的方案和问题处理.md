# 📚 目录

1. [图片缓存](#1-图片缓存)
2. [问题](#2-问题)
3. [解决方法](#3-解决方法)
---

# 1. 图片缓存

我使用的图片库是[extended_image](https://github.com/fluttercandies/extended_image)，代码如下：

```dart
ExtendedImage.network(
      key: ValueKey(imageUrl),
      imageUrl,
      width: width,
      height: height,
      fit: BoxFit.fitHeight,
      cache: true,
      compressionRatio: 1.0,
      cacheMaxAge: Duration(hours: 2),
      cacheWidth: cacheWidth,
      cacheHeight: cacheHeight,

      /// 加载状态回调
      loadStateChanged: (ExtendedImageState state) {
        switch (state.extendedImageLoadState) {
          /// 加载中
          case LoadState.loading:
            // 加载中组件
            return LoadWait();

          /// 加载成功
          case LoadState.completed:
            return state.completedWidget;

          /// 加载失败
          case LoadState.failed:
            // 失败组件
            return LoadFailed(callback: state.reLoadImage);
        }
      },
    );
```

# 2. 问题

我写的 App 涉及大量的图片瀑布流列表和图片预览，在加载网络图片时候，我全都添加了缓存，避免再次加载。但是！图片多了之后，之前缓存好的图片,离屏后重新显示就会重新加载。我百思不得其解，经过排查，发现是因为 flutter 的内存缓存机制导致图片被内存回收了，特此记录。

机制规则如下：

- 最大缓存数量：Flutter 默认设置的图片缓存的最大数量是 1000 张。意味着一旦达到这个数量，最早未使用的（或最少使用的）图片将从缓存中移除，以为新的图片腾出空间
- 最大缓存空间：除了图片数量的限制，Flutter 还限制了缓存的总大小，默认情况下最大为 100MB。这意味着即使图片的数量没有达到 1000 张，如果缓存的总大小达到了 100MB，那么同样会根据缓存管理策略移除一些图片
- 缓存管理策略：Flutter 使用 LRU（最近最少使用）算法来管理缓存。这种策略会保留最近经常使用的图片在缓存中，而将不常用的图片移出缓存。
- 缓存的局限性：Flutter 仅提供内存中的图片缓存，并没有提供磁盘缓存功能。这意味着一旦应用程序重启，所有的图片缓存将会丢失，需要重新加载图片。

# 3. 解决方法

- 修改缓存配置（确保在应用启动时就进行了配置）

```dart
import 'package:flutter/services.dart';

void main() {
  /// 省略其余代码……

  /// 扩大imageCache的缓存数量为300
  PaintingBinding.instance.imageCache.maximumSize = 300;
  /// 扩大imageCache的缓存为512MB
  PaintingBinding.instance.imageCache.maximumSizeBytes = 512 << 20;
}
```

> 注意：增加缓存大小会增加应用占用的内存，可能导致性能问题，调整时要谨慎。

- 列表的图片尽量使用缩略图，尺寸控制在300kb以内

- 尽可能使用固定高度的元素，并且在参数中指定高度
