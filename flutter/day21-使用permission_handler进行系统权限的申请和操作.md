# 📚 目录

1. [介绍](#1-介绍)
2. [环境准备](#2-环境准备)
    1. [Android](#2-1-android)
    2. [iOS](#2-2-ios)
3. [使用](#3-使用)
---

# 1. 介绍

在大多数操作系统上，权限不是在安装时才授予应用程序的。相反，开发人员必须在应用程序运行时请求用户的许可。在 flutter 开发中，则需要一个跨平台(iOS, Android)的 API 来请求权限和检查他们的状态，这时候就需要使用 flutter 插件[permission_handler](https://pub.dev/packages/permission_handler)来帮忙了。它允许您请求和检查权限。你还可以打开设备的应用程序设置，以便用户授予权限。

# 2. 环境准备

项目更目录打开运行窗口，安装插件：

```shell
flutter pub add permission_handler
```

当在运行时请求权限时，你仍然需要告诉操作系统你的应用程序可能会使用哪些权限。这需要在 Android 和 ios 特定文件中添加权限配置。

## 2-1. Android

1. 检查你的 android 目录下的 gradle.properties 文件，是否有下列代码：

```yaml
android.useAndroidX=true
android.enableJetifier=true
```

2. 检查你的 android/app 目录下的 build.gradle 文件，是否有下列代码：

```yaml
android {
  compileSdkVersion 33
  # 省略代码……
}
```

3. 添加需要的权限到 android/app/src/main 目录下 AndroidManifest.xml 文件

```yaml
<manifest xmlns:android="http://schemas.android.com/apk/res/android">
    <!-- 网络权限 -->
    <uses-permission android:name="android.permission.INTERNET" />
    <!-- 写权限 -->
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
    <!-- 读权限 -->
    <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
    # 省略代码……
</manifest>
```

## 2-2. iOS

1. 检查你的 ios 目录下的 podfile 文件，添加下列代码：

```yaml
# 权限配置开始
  config.build_settings['GCC_PREPROCESSOR_DEFINITIONS'] ||= [
      '$(inherited)',

      ## 仅允许写入日历的权限（iOS 16 及以下）
      # 'PERMISSION_EVENTS=1',

      ## 允许完全访问日历的权限（iOS 17及以上）
      # 'PERMISSION_EVENTS_FULL_ACCESS=1',

      ## 提醒事项权限
      # 'PERMISSION_REMINDERS=1',

      ## 联系人权限
      # 'PERMISSION_CONTACTS=1',

      ## 相机权限
      # 'PERMISSION_CAMERA=1',

      ## 麦克风权限
      # 'PERMISSION_MICROPHONE=1',

      ## 语音识别权限
      # 'PERMISSION_SPEECH_RECOGNIZER=1',

      ## 照片权限
      # 'PERMISSION_PHOTOS=1',

      ## 位置权限组（包括始终、使用中）
      # 'PERMISSION_LOCATION=1',

      ## 通知权限
      # 'PERMISSION_NOTIFICATIONS=1',

      ## 媒体库权限
      # 'PERMISSION_MEDIA_LIBRARY=1',

      ## 传感器权限
      # 'PERMISSION_SENSORS=1',

      ## 蓝牙权限
      # 'PERMISSION_BLUETOOTH=1',

      ## 应用跟踪透明度权限
      # 'PERMISSION_APP_TRACKING_TRANSPARENCY=1',

      ## 关键警报权限
      # 'PERMISSION_CRITICAL_ALERTS=1'
    ]
    # 权限配置结束
  end
```

2. 打开 ios/Runner 目录中的 Info.plist 文件，添加需要的权限

| 权限名                      | Info.plist 的键                                                    | 指令 Macro                    |
| --------------------------- | ------------------------------------------------------------------ | ----------------------------- |
| 日历权限（< iOS 17）        | NSCalendarsUsageDescription                                        | PERMISSION_EVENTS             |
| 日历写权限（iOS 17+）       | NSCalendarsWriteOnlyAccessUsageDescription                         | PERMISSION_EVENTS             |
| 日历完全访问权限（iOS 17+） | NSCalendarsFullAccessUsageDescription                              | PERMISSION_EVENTS_FULL_ACCESS |
| 提醒事项权限                | NSRemindersUsageDescription                                        | PERMISSION_REMINDERS          |
| 联系人权限                  | NSContactsUsageDescription                                         | PERMISSION_CONTACTS           |
| 相机权限                    | NSCameraUsageDescription                                           | PERMISSION_CAMERA             |
| 相册权限                    | NSPhotoLibraryAddUsageDescription                                  | -                             |
| 麦克风权限                  | NSMicrophoneUsageDescription                                       | PERMISSION_MICROPHONE         |
| 语音识别权限                | NSSpeechRecognitionUsageDescription                                | PERMISSION_SPEECH_RECOGNIZER  |
| 照片权限                    | NSPhotoLibraryUsageDescription                                     | PERMISSION_PHOTOS             |
| 位置权限                    | NSLocationUsageDescription,                                        | PERMISSION_LOCATION           |
| 始终使用位置权限            | NSLocationAlwaysAndWhenInUseUsageDescription                       | PERMISSION_NOTIFICATIONS      |
| 仅在使用时使用位置权限      | NSLocationWhenInUseUsageDescription                                | PERMISSION_NOTIFICATIONS      |
| 通知权限                    | PermissionGroupNotification                                        | PERMISSION_NOTIFICATIONS      |
| 媒体库权限                  | NSAppleMusicUsageDescription（需要与一个字符串（string）配对使用） | -                             |
| 媒体库权限                  | kTCCServiceMedia                                                   | -                             |

```yaml
<!-- 保存图片到相册 -->
<key>NSPhotoLibraryAddUsageDescription</key>
<string>Please allow the APP to save photos to the album</string>
```

# 3. 使用

注意，获取存储权限的系统弹框只会出现一次，假如第一次不同意，下次再申请就需要自己写确认框引导用户打开 app 权限的页面：openAppSettings()。

```dart
import 'package:flutter/foundation.dart';
import 'package:permission_handler/permission_handler.dart';

/// 获取存储权限
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

void checkPermission() async {
  // 请求存储权限
  final permissionState = await getStoragePermission();
  if (permissionState) {
    // 权限被授予
  } else {
    // 权限被拒绝 打开手机上该App的权限设置页面
    openAppSettings();
  }
}
```
