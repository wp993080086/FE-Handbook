# 📚 目录

1. [变量声明](#1-变量声明)
2. [操作符](#2-操作符)
3. [数据类型](#3-数据类型)
4. [函数](#4-函数)
---

# 1. 变量声明

- var

类似于 JavaScript 中的var，它可以接收任何类型的变量，但最大的不同是 Dart 中 var 变量一旦赋值，类型便会确定，则不能再改变其类型：

```dart
var name = "李四"

name = "张三" // 对

name = 1000 // 错
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

# 4. 函数