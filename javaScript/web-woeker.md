[toc]

# 1、前言

最近做的项目出现了界面卡顿的问题，经过一番排查，发现是因为有个数据做了一些格式化和生成转换，本来只有1000条数据，处理完之后变成了N万条数据（业务需求），导致页面渲染很慢，甚至会崩溃。于是就想着优化一下。初始化的时候不加载，等需要的时候，再使用Web Worker来处理数据，避免主线程卡顿。

# 2、介绍Web Worker

在介绍之前，先说一下Web Worker是为什么而诞生的。

因为JavaScript语言采用的是单线程模型，也就是说，所有任务只能在一个线程上完成，一次只能做一件事。前面的任务没做完，后面的任务只能等着。随着电脑计算能力的增强，尤其是多核CPU的出现，单线程带来很大的不便，无法充分发挥计算机的计算能力。

Web Worker的作用，就是为JavaScript创造多线程环境。它是HTML5标准的一部分，它赋予了开发者利用JavaScript操作多线程的能力。允许主线程创建Worker线程，将一些任务分配给Worker线程运行。在主线程运行的同时，Worker线程在后台运行，两者互不干扰。等到Worker线程完成计算任务，再把结果返回给主线程。这样的好处是，一些计算密集型或高延迟的任务，被Worker线程负担了，主线程（通常负责 UI 交互）就会很流畅，不会被阻塞或拖慢。

# 3、使用须知及兼容性

在使用Worker前，需要先了解一些规则和浏览器的兼容性，避免出现一些问题。

## 3.1、使用须知

1. 资源耗费：Worker线程一旦新建成功就会始终运行，不会被主线程上的活动（比如用户点击按钮、提交表单）打断。这样有利于随时响应主线程的通信。但是也造成了Worker比较耗费资源，建议使用完毕就关闭。

2. 同源限制：分配给 Worker 线程运行的脚本文件，必须与主线程的脚本文件同源。

3. DOM限制：Worker线程所在的全局对象是self，它与主线程不一样，无法读取主线程所在网页的window，DOM，document，parent等全局对象，但可以读取主线程的：navigator和location对象。

4. 脚本限制：Web Worker中可以使用XMLHttpRequest和Axios发送请求。

5. 通信联系：Worker 线程和主线程不在同一个上下文环境，它们不能直接通信，必须通过消息完成。

6. 文件限制：Worker线程中无法读取本地文件，即不能打开本机的文件系统（file://），它所加载的脚本，必须来自网络。

## 3.2、兼容性


浏览器 | 兼容性 |	最低兼容版本
---|---|---|
Chrome |	完全兼容 | 4.0 (2008年)
Firefox |	完全兼容 |	3.5 (2009年)
Safari |	完全兼容 |	3.1 (2007年)
Edge |	完全兼容 |	79 (2020年)
IE | 部分兼容	|	10 (2012年)
Opera |	完全兼容 |	10.5 (2010年)

# 4、使用Web Worker

直接使用JavaScript原生的Worker()构造函数，它的参数如下：

参数 | 说明
---|---
path | 有效的js脚本的地址，必须遵守同源策略
options.type | 可选。用以指定 worker 类型。该值可以是 classic 或 module，默认classic
options.credentials |	可选。指定 worker 凭证。该值可以是 omit, same-origin，或 include。如果未指定，或者 type 是 classic，将使用默认值 omit (不要求凭证)
options.name | 可选。在 DedicatedWorkerGlobalScope 的情况下，用来表示 worker 的 scope 的一个 DOMString 值，主要用于调试目的。

## 4.1、创建Web Worker

主线程：

```javascript
const myWorker = new Worker('/worker.js')

// 接收消息
myWorker.addEventListener('message', e => {
  console.log(e.data)
})

// 向 worker 线程发送消息
myWorker.postMessage('Greeting from Main.js')
```

## 4.2、与主线程通信

worker线程：

```javascript
// 接收到消息
self.addEventListener('message', e => {
    console.log(e.data)
})

// 一顿计算后 发送消息
const calculateDataFn = () => {
  self.postMessage('ok')
}
```

## 4.3、终止Web Worker

两个线程里都可以操作，自由选择。

- 在主线程中操作：

```javascript
// 创建worker
const myWorker = new Worker('/worker.js')
// 关闭worker
myWorker.terminate()
```

- 在worker线程中操作：

```javascript
self.close()
```

## 4.4、使用Shared Worker

SharedWorker允许多个页面共享同一个后台线程，从而实现更高效的资源利用和协同计算。


## 4.5、调试Shared Worker

## 4.6、使用Service Worker

# 5、使用的一些坑

## 5.1、Web Woeker中引入了其余文件

## 5.2、在WebPack或Vite中使用

## 5.3、Hook封装

# 6、总结