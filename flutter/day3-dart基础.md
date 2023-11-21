# 📚 目录

1. [变量声明](#1-变量声明)
2. [操作符](#2-操作符)
3. [数据类型](#3-数据类型)
4. [控制流](#4-控制流)
5. [错误处理和捕获](#5-错误处理和捕获)
6. [函数](#6-函数)
7. [mixin](#7-mixin)
8. [异步](#8-异步)
    1. [Future](#8-1-future)
    1. [Stream](#8-2-stream)
---

# 1. 变量声明

- var

类似于 JavaScript 中的var，它可以接收任何类型的变量，但最大的不同是 Dart 中 var 变量一旦赋值，类型便会确定，则不能再改变其类型：

```dart
var name = "李四"
/// 对
name = "张三"
/// 错
name = 1000
```

- Object

Object 是 Dart 所有对象的根基类，也就是说在 Dart 中所有类型都是Object的子类（包括Function和Null），所以任何类型的数据都可以赋值给Object声明的对象，且后期可以改变赋值的类型。但是声明的对象只能使用 Object 的属性与方法, 否则编译器会报错。

```dart
Object t = 1000

t = "张三"

print(t.length) // 错
```

- dynamic

任何类型的数据都可以赋值给dynamic声明的对象，且声明的变量都可以赋值任意对象，且后期可以改变赋值的类型。而编译器会提供所有可能的组合给声明的对象，这个特点使得我们在使用它时需要格外注意，比如下面代码在编译时不会报错，而在运行时会报错：

```dart
dynamic t

t = "王五"

print(t.xxx) // 错
```

- final

如果您从未打算更改一个变量，那么可以使用 final。一个 final 变量只能被设置一次。它是一个运行时常量，final变量在运行时才知道其值，用于防止变量在程序运行期间被改变。

- const

如果您从未打算更改一个变量，那么可以使用 const。与final的区别是，const 变量是一个编译时常量，其值必须在声明时就确定。（编译时直接替换为常量值）用于创建编译时常量，以便进行性能优化。

- 空安全

Dart 中一切都是对象，这意味着如果我们定义一个数字，在初始化它之前如果我们使用了它，假如没有某种检查机制，则不会报错。如下：

```dart
test() {
  int i; 
  print(i*8); // 错
}
```

在 Dart 引入空安全之后，如果一个变量没有初始化，则不能使用它，否则编译器会报错：

```dart
int a = 10
```

声明时候可以加上?，指定变量是可空：

```dart
String? name;
```

如果我们预期变量不能为空，但在定义时不能确定其初始值，则可以加上late关键字，表示会稍后初始化，但是在正式使用它之前必须得保证初始化过了，否则会报错：

```dart
late int c;

c = 100
```

如果一个变量我们定义为可空类型，且判空了，但是预处理器仍然有可能识别不出，这时我们就要在变量后面加一个`!`符号，告诉预处理器它已经不是null了。

```dart
class Test() {
  int? d;
  log() {
    if (d! = null) {
      print(d! * 100)
    }
  }
}
```

如果函数变量可空时，调用的时候可以用语法糖：

```dart
fun?.call()
```

- 特殊声明模式

```dart
// 同时声明
var (a, [b, c]) = ('str', [1, 2])

// 同时声明
var (c, d) = ('left', 'right');

// 交换2个值
(c, d)  = (d, c)

// 解构
var (name, age) = userInfo(json)

// 省略符 
var (a, b, ...rest) = [1, 2, 3, 4, 5, 6] // 1 2 [3,4,5,6]

var (a, b, ...rest, c, d) = [1, 2, 3, 4, 5, 6] // 1 2 [3,4] 5 6

var (a, b, ..., c, d) = [1, 2, 3, 4, 5, 6] // 1 2 5 6
```

# 2. 操作符

- 算数运算符

|符号|意义|描述|
|--|--|--|
|+|加|-|
|-|减|-|
|*|乘|-|
|/|除|-|
|~/|除|返回整数结果|
|%|取余|获取整数除法的余数|

- 等于运算符

|符号|意义|描述|
|--|--|--|
|==|平等|-|
|!=|不相等|-|
|>|大于|-|
|<|小于|-|
|>=|大于或等于|-|
|<=|小于或等于|-|

- 逻辑运算符

|符号|意义|描述|
|--|--|--|
|!|反转|-|
|\|\||或者|-|
|&&|并且|-|

- 赋值运算符

|符号|意义|描述|
|--|--|--|
|=|赋值|-|
|*=|乘法赋值|-|
|+=|加法赋值|-|
|-=|减法赋值|-|
|/=|除法赋值|-|
|%=|取模赋值|-|
|~/=|返回除法整数结果赋值|-|
||=|按位或赋值|-|
|&=|按位与赋值|-|
|^=|按位异或赋值|-|
|<<=|左移赋值|-|
|>>=|右移赋值（保留符号位）|-|
|>>>=|右移赋值（无符号右移）|-|

- 类型运算符

|符号|意义|描述|
|--|--|--|
|as|类型转换断言|-|
|is|如果对象具有指定的类型，则为 True|-|
|is!|如果对象没有指定的类型，则为 True|-|

```dart
// 如果您不确定该对象的类型是否为Person，则在使用对象之前用is检查
if (employee is Person) {
  employee.firstName = 'Bob';
}
```

- 按位运算符

|符号|意义|描述|
|--|--|--|
|&|与|-|
|\||或|-|
|^|异或|-|
|<<|左移|-|
|>>|右移|-|
|>>>|无符号右移|-|

- 条件表达式

|语法|意义|描述|
|--|--|--|
|condition ? expr1 : expr2|赋值|三元表达式|
|expr1 ?? expr2|赋值|如果 expr1 为非 null，则返回其值; 否则，计算并返回 expr2 的值|
|?.xxx|短路|条件成员访问|
|!|断言|将表达式强制转换为其基础不可为 null 的类型|

- 级联表示法

级联允许您在同一对象上，进行一系列操作。

```dart
// 级联语法
var paint = Paint()
  ..color = Colors.black
  ..strokeCap = StrokeCap.round
  ..strokeWidth = 5.0;

// 等价于如下语法
var paint = Paint();
paint.color = Colors.black;
paint.strokeCap = StrokeCap.round;
paint.strokeWidth = 5.0;
```

```dart
// 级联语法
querySelector('#confirm')
  ?..text = 'Confirm'
  ..classes.add('important')
  ..onClick.listen((e) => window.alert('Confirmed!'))
  ..scrollIntoView();

// 等价于如下语法
var button = querySelector('#confirm');
button?.text = 'Confirm';
button?.classes.add('important');
button?.onClick.listen((e) => window.alert('Confirmed!'));
button?.scrollIntoView();
```

# 3. 数据类型

|关键字|描述|
|--|--|
|int|不大于 64 位的整数值|
|double|64 位（双精度）浮点数|
|String|字符串，多行字符串可以用'''xxx'''|
|bool|字符串|
|Set|集|
|Map|地图|
|List|列表|
|Symbol|符号|
|Null|null|
|record|记录|
|typedef|类型别名|

# 4. 控制流

- switch

```dart
int size = 2;

// 条件判断
switch (size) {
  case 1:
    print('一');
  case [2, 3]:
    print('二 and 三')
  case [4 || 5]:
    print('四 or 五')
  case >= 6 && <= 10
    print('六 to 十')
}

// 条件赋值
var bg = Color.green

var isPrimary = switch (bg) {
  Color.red || Color.yellow || Color.blue => true,
  _ => false
}
```

- for

```dart
var message = StringBuffer('Dart is fun');

for (var i = 0; i < 5; i++) {
  message.write('!');
}
```

- fon-in

```dart
Map<String, int> listMap = {
  'a': 23,
  'b': 100,
};

for (final MapEntry(key: key, value: value) in listMap.entries) {
  print('键：$key 值：$value');
}
```

- while

```dart
while (!isDone()) {
  doSomething();
}
```

- do-while

```dart
do {
  printLine();
} while (!atEndOfPage());
```

- 跳过和停止

```dart
while (true) {
  if (shutDownRequested()) break;
  processIncomingRequests();
}

for (int i = 0; i < candidates.length; i++) {
  var candidate = candidates[i];
  if (candidate.yearsExperience < 5) {
    continue;
  }
  candidate.interview();
}
```

- if

```dart
if (isRaining()) {
  you.bringRainCoat();
} else if (isSnowing()) {
  you.wearJacket();
} else {
  car.putTopDown();
}
```

- if-case

```dart
// 接受一个名为pair的参数。如果pair的类型是[int x, int y]，则返回一个Point(x, y)对象
if (pair case [int x, int y]) return Point(x, y);
```

# 5. 错误处理和捕获

异常捕获和JavaScript差不多

```dart
try {
  breedMoreLlamas();
} on OutOfLlamasException {
  // 特定的异常
  buyMoreLlamas();
} on Exception catch (e) {
  // 任何其他的例外
  print('Unknown exception: $e');
} catch (e) {
  // 没有指定类型，处理所有类型
  print('Something really unknown: $e');
} finally {
  cleanLlamaStalls();
}
```

- 若要部分处理异常， 同时允许它传播， 使用关键字：rethrow

```dart
void misbehave() {
  try {
    dynamic foo = true;
    print(foo++); // 运行时错误
  } catch (e) {
    print('misbehave() partially handled ${e.runtimeType}.');
    rethrow; // 允许调用者看到异常
  }
}

void main() {
  try {
    misbehave();
  } catch (e) {
    print('main() finished handling ${e.runtimeType}.');
  }
}
```

- 断言

在开发过程中，使用断言语句，如果布尔条件为false，则触发。如果条件为true，则不触发。在生产代码中，断言将被忽略。

```dart
// assert(条件, 可选消息)
var sizi = 101;
assert(size < 100, "Error：size 小于 100！");
```

# 6. 函数

Dart是一种真正的面向对象的语言，所以即使是函数也是对象，并且有一个类型Function。这意味着函数可以赋值给变量或作为参数传递给其他函数，这是函数式编程的典型特征。

- 函数声明

```dart
bool isNoble(int ) {
  return atomicNumber > 40;
}

bool isNoble (int atomicNumber) => atomicNumber > 40;
```

- 函数作为参数传递

```dart
//定义函数execute，它的参数类型为函数
void execute(var callback) {
  // 执行传入的函数
  callback();
}

// 调用execute，将箭头函数作为参数传递
execute(() => print("xxx"))
```

- 可选的位置参数

```dart
String say(String a, String b, [String? c]) {
  var result = '$a and $b';
  if (c != null) {
    result = '$result and $c';
  }
  return result;
}
```

- 可选的命名参数

```dart
//设置[a]和[b]标志
void enableFlags({bool a, bool b}) {
  // ... 
}

// 调用函数时，可以使用指定命名参数
enableFlags(bold: true, hidden: false)
```

# 7. mixin

Dart 是不支持多继承的，但是它支持 mixin。可以定义几个 mixin，然后通过 with 关键字将它们组合成不同的类（同名会被最后的覆盖）。如下例子：

```dart
class Person {
  say() {
    print('say');
  }
}

mixin Eat {
  eat() {
    print('eat');
  }
}

mixin Walk {
  walk() {
    print('walk');
  }
}

mixin Code {
  code() {
    print('key');
  }
}

class Dog with Eat, Walk{}
class Man extends Person with Eat, Walk, Code{}
```

# 8. 异步

Dart类库有非常多的返回Future或者Stream对象的函数，这些函数被称为异步函数。

## 8-1. Future

Future与JavaScript中的Promise非常相似。

```dart
// 使用Future.delayed 创建了一个延时任务，2秒后返回结果字符串，然后在then中接收异步结果并打印
Future.delayed(Duration(seconds: 2),(){
  return "hi world!";
}).then((data){
  // 执行成功
  print(data);
}).catchError((e){
   //执行失败会走到这里  
   print(e);
}).whenComplete((){
   //无论成功或失败都会走到这里
});
```

- 多个异步：Future.wait

```dart
Future.wait([
  // 2秒后返回结果  
  Future.delayed(Duration(seconds: 2), () {
    return "hello";
  }),
  // 4秒后返回结果  
  Future.delayed(Duration(seconds: 4), () {
    return " world";
  })
]).then((results){
  print(results[0]+results[1]);
}).catchError((e){
  print(e);
});
```

- 解决回调地狱：async/await

```dart
task() async {
  try{
    String id = await login("alice","******");
    String userInfo = await getUserInfo(id);
    await saveUserInfo(userInfo);
    // 执行接下来的操作
  } catch(e){
    // 错误处理
    print(e);
  }
}
```

## 8-2. Stream

Stream 也是用于接收异步事件数据，和 Future 不同的是，它可以接收多个异步操作的结果（成功或失败）。 也就是说，在执行异步任务时，可以通过多次触发成功或失败事件来传递结果数据或错误异常。 Stream 常用于会多次读取数据的异步任务场景，如网络内容下载、文件读写等。

```dart
Stream.fromFutures([
  // 1秒后返回结果
  Future.delayed(Duration(seconds: 1), () {
    return "hello 1";
  }),
  // 抛出一个异常
  Future.delayed(Duration(seconds: 2),(){
    throw AssertionError("Error");
  }),
  // 3秒后返回结果
  Future.delayed(Duration(seconds: 3), () {
    return "hello 3";
  })
]).listen((data){
  print(data);
}, onError: (e){
  print(e.message);
},onDone: (){});

// 上述代码依次输出
// hello 1
// Error
// hello 3
```