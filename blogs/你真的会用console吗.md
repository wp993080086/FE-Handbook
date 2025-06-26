# 前言
在前端开发中，`console.log()` 是最常用的调试工具，但 `console` 对象实际上蕴含着丰富的功能。除了基础的日志打印，它还能以表格形式展示数据、为输出添加自定义样式，甚至能在控制台中呈现图片和动图。本文将系统介绍 `console` 对象的各类实用方法，涵盖调试信息打印、数据可视化、性能监测等多个维度，帮助开发者更高效地利用控制台进行开发调试。


# 1. 基础信息打印

主要是一些文字信息：

**1. 调试信息打印**
```javascript
console.debug('console.debug：调试信息')
```
- **应用场景**：用于记录开发过程中的调试细节。
- **显示位置**：在浏览器控制台的 `levels` 选项中选择 `Verbose` 级别可见。

**2. 普通信息与提示信息**
```javascript
console.log('console.log：普通信息')
console.info('console.info：提示信息')
```
- **区别**：在谷歌浏览器中，`console.log` 与 `console.info` 的显示效果几乎一致，但在其他浏览器中可能存在样式差异（如图标或颜色不同）。


# 2. 数据可视化展示

可以平铺打印数组和对象等等
## 2.1. 表格形式打印
```javascript
console.table([
  { id: 1, name: '张三', age: 12 },
  { id: 2, name: '李四', age: 13 },
  { id: 3, name: '王五', age: 11 },
  { id: 4, name: '赵六', age: 14 },
]);
```
- **优势**：将对象数组或类数组数据以表格形式直观展示，支持点击表头排序，便于调试后端接口返回的数据。

## 2.2. 分组打印
```javascript
console.group('学生信息')
console.table([
  { id: 1, name: '张三', age: 12 },
  { id: 2, name: '李四', age: 13 }
])
console.groupEnd()

console.groupCollapsed('老师信息') // 折叠分组
console.table([
  { id: 3, name: '王五', age: 11 },
  { id: 4, name: '赵六', age: 14 }
])
console.groupEnd()
```
- **功能**：通过分组组织相关日志，避免控制台信息混乱。`console.groupCollapsed` 可默认折叠分组，节省界面空间。


# 3. 对象与性能调试
下面方法可用于代码调试和测试等等
## 3.1. 打印对象结构
```javascript
console.dir(document.body);
```
- **对比**：`console.log(document.body)` 仅显示 DOM 元素结构，而 `console.dir` 可展示对象的所有属性和方法，便于深入分析 DOM 对象或自定义对象的结构。

## 3.2. 计时功能
```javascript
console.time("循环");
const startTime = Date.now();
while (Date.now() - startTime < 1000) {}
console.timeEnd("循环"); // 输出执行时间
```
- **用途**：精确测量代码块的执行耗时，用于性能测试和优化。

## 3.3. 计数功能
```javascript
const startTime = Date.now();
while (Date.now() - startTime < 1000) {
  console.count("计数"); // 每次执行时计数加1
}
// 输出：计数: 执行次数
```
- **场景**：统计函数或循环的执行次数，辅助分析代码运行频率。


# 4. 调试与错误处理
如下：
## 4.1. 堆栈跟踪
```javascript
function funA() {
  console.trace();
}
function funB() {
  funA();
}
funB();
```
- **输出示例**：
```
console.trace
funA                  @index.tsx:187
funB                  @index.tsx:190
handlechangeTabsType  @index.tsx:192
handleclickTabs       @index.tsx:19
onclick               @index.tsx:30
```
- **作用**：显示函数调用堆栈，帮助定位代码执行路径，尤其适用于第三方库报错时的问题排查。

## 4.2. 断言机制
```javascript
// 断言为false时，会打印错误
console.assert(4 < 2);
```
- **注意**：在 Node 环境下，断言失败会中断程序；在浏览器环境中仅打印警告，不中断执行。

## 4.3. 警告与错误信息
```javascript
console.warn("这是一个警告信息");
console.error("这是一个错误信息");
```
- **显示效果**：警告和错误信息会以特定样式突出显示（如黄色背景或红色图标），便于快速识别异常。


# 5. 控制台高级操作
如下：
## 5.1. 清空消息
```javascript
console.log("a");
console.clear(); // 清空控制台所有历史消息
```

## 5.2. 自定义样式输出
```javascript
const message = '%c带样式的文本';
const style = 'color: red; font-size: 12px; font-weight: bold; border: 1px green solid';
console.log(message, style);
```
- **语法**：使用 `%c` 标记样式起始，后续字符串应用对应样式，支持 CSS 样式属性（如颜色、字体、边框等）。

## 5.3. 控制台打印图片（含动图）
```javascript
const image = 'data:image/png;base64,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'
console.log(
			`%c %c \nCSDN\n%c LOGO %c png `,
			`padding: 20px 40px; background-image: url(${image}); background-size: contain; background-repeat: no-repeat; color: transparent;`,
			'color: #FC5531; font-size: 12px;line-height:24px;',
			'background: #240F74; padding: 2px; border-radius: 4px 0 0 4px; color: #fff',
			'background: #FC5531; padding: 2px; border-radius: 0 4px 4px 0; color: #fff'
		)
```
- **注意事项**：
  - 有些浏览器不支持直接显示网络地址的图片，需将图片转换为 Base64 格式后放入 `background-image`。
  - 通过 CSS 背景图实现动图显示，需确保图片格式为 GIF 并转换为 Base64。


# 6. 总结
`console` 对象远不止 `log` 功能，合理运用这些高级方法能大幅提升调试效率：
- **数据展示**：`console.table` 和分组打印让复杂数据结构一目了然；
- **性能监测**：`time` 和 `count` 帮助量化代码执行效率；
- **调试技巧**：`trace` 和 `assert` 辅助定位逻辑错误；
- **创意应用**：自定义样式和图片打印为控制台增添趣味性（如公司 Logo、调试彩蛋）。

# 7. console插件
这里分享一些console打印的插件，祝你打印出帅气的信息：

- [loglevel](https://github.com/pimterry/loglevel)：最小轻量级日志记录，添加可靠的日志级别方法来包装任何可用的console.log方法。
- Turbo Console Log：VsCode插件，直接在VsCode扩展中搜索
- [eruda](https://github.com/liriliri/eruda)：移动端浏览器控制台。
- [unplugin-turbo-console](https://github.com/unplugin/unplugin-turbo-console)： 一个Vite插件，用于在开发时打印更漂亮的控制台日志。