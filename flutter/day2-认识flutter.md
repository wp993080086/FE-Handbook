# 📚 目录

1. [简介](#1-简介)
2. [特点](#2-特点)
3. [架构](#3-架构)
   1. [框架层](#3-1-框架层)
   2. [引擎层](#3-2-引擎层)
   3. [嵌入层](#3-3-嵌入层)
---

# 1. 简介

Flutter 是 Google 推出并开源的移动应用开发框架，主打跨平台、高保真、高性能。开发者可以通过 Dart 语言开发 App，一套代码同时运行在 iOS 和 Android平台。 Flutter 提供了丰富的组件、接口，开发者可以很快地为 Flutter 添加 Native扩展。

# 2. 特点

- 跨平台自绘引擎
  - Flutter 既不使用 WebView，也不使用操作系统的原生控件。 相反，Flutter 使用自己的高性能渲染引擎来绘制 Widget（组件）。这样不仅可以保证在 Android 和iOS 上 UI 的一致性，也可以避免对原生控件依赖而带来的限制及高昂的维护成本。
  - Flutter 底层使用 Skia 作为其 2D 渲染引擎，Skia 是 Google的一个 2D 图形处理函数库，包含字型、坐标转换，以及点阵图，它们都有高效能且简洁的表现。Skia 是跨平台的，并提供了非常友好的 API，目前 Google Chrome浏览器和 Android 均采用 Skia 作为其 2D 绘图引擎。

- 高性能
  - Flutter App 采用 Dart 语言开发。Dart 在 JIT（即时编译）模式下，执行速度与 JavaScript 基本持平。但是 Dart 支持 AOT，当以 AOT模式运行时，JavaScript 便远远追不上了。执行速度的提升对高帧率下的视图数据计算很有帮助。
  - Flutter 使用自己的渲染引擎来绘制 UI ，布局数据等由 Dart 语言直接控制，所以在布局过程中不需要像 RN 那样要在 JavaScript 和 Native 之间通信，这在一些滑动和拖动的场景下具有明显优势。

- 采用Dart语言开发
  - Flutter 在开发阶段采用 JIT 模式（动态解释），这样就避免了每次改动都要进行编译，极大地节省了开发时间。
  - Flutter 在发布时可以通过 AOT（提前编译） 生成高效的机器码以保证应用性能，而 Dart 支持 AOT。
  - Dart 语言的静态类型检查，可以有效避免运行时错误，减少崩溃率。
  - 类型安全和空安全+快速内存分配

# 3. 架构

Flutter 从上到下可以分为三层：框架层、引擎层和嵌入层。

## 3-1. 框架层

框架层Flutter Framework，这是一个纯 Dart实现的 SDK，它实现了一套基础库，自底向上。

- dart UI层（Foundation 和 Animation、Painting、Gestures）：
  - 对应的是Flutter中的dart:ui包，它是 Flutter Engine 暴露的底层UI库，提供动画、手势及绘制能力。

- 渲染层Rendering：
  - 这一层是一个抽象的布局层，它依赖于 Dart UI 层，渲染层会构建一棵由可渲染对象组成的渲染树，当动态更新这些对象时，渲染树会找出变化的部分，然后调用底层 dart:ui，进行坐标变换、绘制，更新渲染。

- 组件层Widgets：
  - Flutter 提供的一套基础组件库，在基础组件库之上，Flutter 还提供了 Material 和 Cupertino 两种视觉风格的组件库，它们分别实现了 Material 和 iOS 设计规范。

Flutter 框架相对较小，因为一些开发者可能会使用到的更高层级的功能已经被拆分到不同的软件包中，使用 Dart 和 Flutter 的核心库实现，其中包括平台插件，例如 camera (opens new window)和 webview (opens new window)，以及和平台无关的功能，例如 animations (opens new window)。

## 3-2. 引擎层

引擎层Flutter Engine，这是 Flutter 渲染和计算的核心。该层主要是 C++ 实现，其中包括了 Skia 引擎、Dart 运行时（Dart runtime）、文字排版引擎等。在代码调用 dart:ui库时，调用最终会走到引擎层，然后实现真正的绘制和显示。

## 3-3. 嵌入层

嵌入层Flutter Embedding，这是 Flutter 实现与平台无关的关键，它提供了 Flutter 与平台交互的接口，包括创建 Flutter 线程、线程间通信、渲染 Surface 等。嵌入层主要是将 Flutter 引擎 ”安装“ 到特定平台上。嵌入层采用了当前平台的语言编写，例如 Android 使用的是 Java 和 C++， iOS 和 macOS 使用的是 Objective-C 和 Objective-C++，Windows 和 Linux 使用的是 C++。Flutter 代码可以通过嵌入层，以模块方式集成到现有的应用中，也可以作为应用的主体。