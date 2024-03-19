# 📚 目录

1. [前言](#1-前言)
1. [导包](#2-导包)
1. [配置](#3-配置)
1. [完整代码](#4-完整代码)
---

# 1. 前言

在我们开发 flutter 项目过程中，难免会遇到将图片和媒体保存到手机上的需求。这时候，就涉及应用权限配置和下载等操作，本文将详细介绍如何使用(image_gallery_saver)[https://pub.dev/packages/image_gallery_saver]插件，实现下载图片和媒体文件。

# 2. 配置

下载之前，需要分别在 iOS 和 Android 的配置文件中写入权限配置。

- iOS：

在项目根目录下的 ios\Runner\Info.plist 文件中，找到 dict 标签，在标签里添加如下配置：

```shell
<key>NSPhotoLibraryAddUsageDescription</key>
<string>Please allow the APP to save photos to the album</string>
```

- Android：

在项目根目录下的 android\app\src\main\AndroidManifest.xml 文件中，找到 manifest 标签，在标签里添加如下配置：

```xml
<!-- 读权限 -->
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
<!-- 写权限 -->
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
```

# 3. 导包

一共需要下面这些包：

- 图片保存到本地：(mage_gallery_saver ^2.0.3)[https://pub.dev/packages/permission_handler]
- 申请权限：(permission_handler ^10.2.0)[https://pub.dev/packages/permission_handler]
- 网络请求：(dio ^5.3.3)[https://pub.dev/packages/dio]

# 4. 完整代码

```dart
import 'package:flutter/foundation.dart';
import 'package:permission_handler/permission_handler.dart';
import 'dart:typed_data';
import 'package:dio/dio.dart';
import 'package:image_gallery_saver/image_gallery_saver.dart';

// 图片加载和缓存的插件
import 'package:extended_image/extended_image.dart';

/// 获取存储权限 系统权限的弹框只会出现一次
/// 假如第一次不同意，下次再申请就需要自己写确认框
/// 引导用户打开app权限的页面：openAppSettings()
Future<bool> getStoragePermission() async {
  late PermissionStatus myPermission;

  /// 读取系统权限
  if (defaultTargetPlatform == TargetPlatform.iOS) {
    myPermission = await Permission.photosAddOnly.request();
  } else {
    myPermission = await Permission.storage.request();
  }
  if (myPermission != PermissionStatus.granted) {
    return false;
  } else {
    return true;
  }
}

/// 下载网络图片（先读缓存资源，缓存没有再重新获取资源）
Future<bool> downloadNetworkImage(String imagePath) async {
  // 前缀
  const String prefix = 'Save';
  // 获取缓存图片
  var cacheData = await getNetworkImageData(imagePath, useCache: true);
  // 获取当前时间戳
  int timestamp = DateTime.now().millisecondsSinceEpoch;
  // 将时间戳转换为可读的日期格式
  String dateTime = DateTime.fromMillisecondsSinceEpoch(timestamp).toString();
  // 拼接名字
  String saveName = '$prefix-$dateTime';
  // 下载逻辑
  late dynamic result;
  /// 如果缓存图片不为空
  if (cacheData != null) {
    result = await ImageGallerySaver.saveImage(cacheData,
        quality: GetPlatform.isIOS ? 100 : 80, name: saveName);
  }
  /// 如果缓存图片为空 则从网络下载
  else {
    var response = await Dio()
        .get(imagePath, options: Options(responseType: ResponseType.bytes));
    result = await ImageGallerySaver.saveImage(
        Uint8List.fromList(response.data),
        quality: GetPlatform.isIOS ? 100 : 80,
        name: saveName);
  }
  return result['isSuccess'];
}

/// 下载图片
void handleImageDownload() async {
  // 获取权限
  bool myPermission = await getStoragePermission();
  if (myPermission) {
    final isOk = await downloadNetworkImage(imagePath);
    if (isOk) {
      // 成功了
    } else {
      // 失败了
    }
  } else {
    // 无权限 打开设置
    openAppSettings();
  }
}
```
