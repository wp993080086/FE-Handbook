# 📚 目录

1. [通过HttpClient发起HTTP请求](#1-通过httpclient发起http请求)
2. [使用dio发起请求](#2-使用dio发起请求)
    2. [安装dio库](#2-1-安装dio库)
    2. [发起请求](#2-2-发起请求)
    2. [完整例子](#2-3-完整例子)
3. [JSON转DartModel类](#3-json转dartmodel类)
    3. [json转dart](#3-1-json转dart)
    3. [json转dartmodel](#3-2-json转dartmodel)
    3. [自动生成model类](#3-3-自动生成model类)
---

# 1. 通过HttpClient发起HTTP请求

Dart IO库中提供了用于发起Http请求的一些类，我们可以直接使用HttpClient来发起请求。

```dart
import 'dart:convert';
import 'dart:io';
import 'package:flutter/material.dart';

/// 定义
class HomePage extends StatefulWidget {
  const HomePage({super.key});

  @override
  State<HomePage> createState() => HomePageState();
}

/// 实现
class HomePageState extends State<HomePage> {
  // 请求地址
  final String serverPath = 'https://xxxxxxxxx';
  
  String str = '';

  @override
  Widget build(BuildContext context) {
    return Scaffold(
        appBar: AppBar(
          title: const Text('Flutter Home'),
        ),
        body: Container(
          alignment: Alignment.center,
          child: Padding(
            padding: const EdgeInsets.all(16.0),
            child: Column(
              children: [
                Text(str, style: const TextStyle(fontSize: 24.0),)
              ],
            ),
          ),
        ),
    floatingActionButton: FloatingActionButton(
      onPressed: () async {
        handleGetData();
      },
      child:  const Text('请求'),
    ),);
  }

  handleGetData() async {
    String path = '$serverPath/get/message';
    try {
      /// 创建一个HttpClient
      HttpClient httpClient = HttpClient();
      /// 打开http连接
      HttpClientRequest requestObject =
      await httpClient.getUrl(Uri.parse(path));
      /// 设置请求头
      requestObject.headers.add('Content-Type', 'application/json');
      /// 等待连接服务器（会将请求信息发送给服务器）
      HttpClientResponse response = await requestObject.close();
      /// 读取响应内容
      str = await response.transform(const Utf8Decoder()).join();
      /// 输出响应头
      debugPrint('响应头：${response.headers.toString()}');
    } catch (error) {
      debugPrint(error.toString());
    }
  }
}
```

# 2. 使用dio发起请求

直接使用HttpClient发起网络请求是比较麻烦的，很多事情得我们手动处理，如果再涉及到文件上传/下载、Cookie管理等就会非常繁琐。而第三方http请求库，用它们来发起http请求将会简单的多，比如目前人气较高的dio库。

## 2-1. 安装dio库

尽量使用pub.dev上的最新版本。

```yaml
dependencies:
  dio: ^x.x.x
```
## 2-2. 发起请求

一个dio实例可以发起多个http请求，一般来说，APP只有一个http数据源时，dio应该使用单例模式。

- 引入并创建实例

```dart
import 'package:dio/dio.dart';

Dio dio =  Dio();

Response response;
```

- 发起get请求

```dart
response = await dio.get("/test?id=12&name=test");
// 或者
response = await dio.get("/test", queryParameters:{"id":12,"name":"test"})

debugPrint(response.data.toString());
```

- 发起post请求

```dart
response = await dio.post("/test",data:{"id":12,"name":"test"})
```

- 同时发起多个请求

```dart
response = await Future.wait([dio.post("/info"), dio.get("/token")]);
```

- 下载文件

```dart
response = await dio.download("https://www.google.com/", savePath);
```

- 发送FormData

```dart
FormData formData = FormData.from({
   "name": "wendux",
   "age": 25,
});
response = await dio.post("/info", data: formData)
```

- 设置代理

```dart
(dio.httpClientAdapter as DefaultHttpClientAdapter).onHttpClientCreate = (client) {
  // 设置代理 
  client.findProxy = (uri) {
    return "PROXY 192.168.1.2:8888";
  };
  // 校验证书
  httpClient.badCertificateCallback = (X509Certificate cert, String host, int port){
    if(cert.pem == PEM){
      // 证书一致，则允许发送数据
      return true;
    }
    return false;
  };   
};
```

## 2-3. 完整例子

```dart
import 'package:flutter/material.dart';
import 'package:dio/dio.dart';

/// 定义
class HomePage extends StatefulWidget {
  const HomePage({super.key});

  @override
  State<HomePage> createState() => HomePageState();
}

/// 实现
class HomePageState extends State<HomePage> {
  Dio myDio = Dio();

  @override
  Widget build(BuildContext context) {
    return Scaffold(
        appBar: AppBar(
          title: const Text('Flutter Home'),
        ),
        body: Container(
            alignment: Alignment.center,
            child: FutureBuilder(
              future:
                  myDio.get("https://api.github.com/orgs/flutterchina/repos"),
              builder: (BuildContext context, AsyncSnapshot snapshot) {
                //请求完成
                if (snapshot.connectionState == ConnectionState.done) {
                  Response response = snapshot.data;
                  //发生错误
                  if (snapshot.hasError) {
                    return Text(snapshot.error.toString());
                  }
                  //请求成功，通过项目信息构建用于显示项目名称的ListView
                  return ListView(
                    children: response.data
                        .map<Widget>(
                            (e) => ListTile(title: Text(e["full_name"])))
                        .toList(),
                  );
                }
                //请求未完成时弹出loading
                return const CircularProgressIndicator();
              },
            )));
  }
}
```

# 3. JSON转DartModel类

后台接口往往会返回一些结构化数据，如 JSON、XML 等，如之前我们请求 Github API 的示例，它返回的数据就是 JSON 格式的字符串，为了方便我们在代码中操作 JSON，我们先将 JSON 格式的字符串转为 Dart 对象，这个可以通过 dart:convert 中内置的 JSON 解码器json.decode()来实现，该方法可以根据 JSON 字符串具体内容将其转为 List 或 Map，这样我们就可以通过他们来查找所需的值。

## 3-1. JSON转Dart

将 JSON 格式的字符串转为 Dart 对象，可以使用 dart:convert 中的 JSON 解码器 json.decode() 来实现，该方法可以根据 JSON 字符串具体内容将其转为 List 或 Map，这样我们就可以通过他们来查找所需的值。

```dart
// 一个JSON格式的用户列表字符串
String jsonStr='[{"name":"Jack"},{"name":"Rose"}]';`

// 将JSON字符串转为Dart对象(此处是List)
List items=json.decode(jsonStr);

// 输出第一个用户的姓名
print(items[0]["name"]);
```

## 3-2. JSON转DartModel

但是，由于json.decode()仅返回一个Map<String, dynamic>，这意味着直到运行时我们才知道值的类型。 通过这种方法，我们失去了大部分静态类型语言特性：类型安全、自动补全和最重要的编译时异常。解决方法即“Json Model化”，具体做法就是，通过预定义一些与 Json 结构对应的 Model 类，然后在请求到数据后再动态根据数据创建出 Model 类的实例。这样一来，在开发阶段我们使用的是 Model 类的实例，而不再是 Map/List，这样访问内部属性时就不会发生拼写错误。如下，是一个简单的模型类：

```dart
/// 定义
class User {
  final String name;
  final String email;

  User(this.name, this.email);

  User.fromJson(Map<String, dynamic> json)
      : name = json['name'],
        email = json['email'];

  Map<String, dynamic> toJson() =>
    <String, dynamic>{
      'name': name,
      'email': email,
    };
}

/// 使用
Map userMap = json.decode(json);
var user = User.fromJson(userMap);

print('名字：${user.name}');
print('邮箱：${user.email}');
```

## 3-3. 自动生成Model类

在 Android Studio 中安装 JsonToDart 插件，打开 Preferences（Mac）或者 Setting（Window），选择 Plugins，搜索 JsonToDart。点击 Install 安装，安装完成后重启。这个时候选定目录，点击右键，选择 New->Json to Dart，选中 Json To Dart 后，弹出页面输入要转换的json数据，点击完成，就会生成对应的模型文件了。