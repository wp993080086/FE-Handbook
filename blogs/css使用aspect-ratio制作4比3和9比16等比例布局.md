# 1. 前言

在网页制作过程中，有时候我们只知道宽度，或者只知道高度，这时候需要制作一个4:3和9:16这种类似的盒子，不使用js而使用纯css如何实现呢：

aspect-ratio 是 CSS 中用于定义元素宽高比的属性。它允许开发者指定一个元素的宽度和高度之间的固定比例关系，无论元素的实际尺寸如何变化，都会保持这个比例。这种特性在响应式设计、图片和视频布局等场景中非常实用，能够帮助开发者更轻松地控制元素的外观和布局，避免因内容尺寸变化导致的布局错乱问题。

# 2. 用法

下面列举了一些常用的用法：

 ## 2.1 基本语法

aspect-ratio属性接受一个由斜杠（/）分隔的两个数字，表示宽度与高度的比例，其语法如下：

```css
element {
  aspect-ratio: <width> / <height>;
}
```

其中，`width`和`height`为非负数字，例如：

```css
.box {
  aspect-ratio: 16 / 9; /* 常见的视频宽高比 */
  width: 500px;
  background-color: lightblue;
}
```

在上述代码中，.box元素的宽度设置为500px，由于aspect-ratio设置为16/9，因此该元素的高度会自动计算为500px * 9 / 16 = 281.25px，从而保持16:9的宽高比。

## 2.2. 与max-width、max-height等属性结合使用

aspect-ratio属性可以与max-width、max-height等属性配合，实现更灵活的响应式布局。例如：

```css
.image-container {
  max-width: 800px;
  aspect-ratio: 4 / 3;
  overflow: hidden;
}

.image-container img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}
```

在这个例子中，.image-container的最大宽度为800px，并且保持4:3的宽高比。内部的图片会自动填满容器，并通过object-fit: cover属性确保图片在保持比例的同时，完整显示内容。

## 2.3. 动态计算比例

aspect-ratio的值也可以通过 CSS 变量（var()）动态计算，以适应不同的布局需求：

```css
:root {
  --ratio-width: 2;
  --ratio-height: 1;
}

.dynamic-box {
  aspect-ratio: var(--ratio-width) / var(--ratio-height);
  background-color: lightcoral;
}
```

通过修改 CSS 变量的值，可以轻松改变元素的宽高比，而无需修改大量的 CSS 代码。

# 3. 应用场景

下面列举了一些常见的应用场景：

- 响应式图片和视频布局

在响应式设计中，aspect-ratio属性常用于保持图片和视频的比例，防止其在不同设备上拉伸变形。例如，对于视频播放器：

```css
.video-player {
  width: 100%;
  aspect-ratio: 16 / 9;
}
```

无论设备屏幕宽度如何变化，视频播放器都会始终保持16:9的比例，确保视频的正确显示。

- 卡片式布局

在卡片式布局中，使用aspect-ratio可以让卡片在不同屏幕尺寸下保持一致的外观。例如：

```css
.card {
  width: 300px;
  aspect-ratio: 3 / 4;
  border: 1px solid #ccc;
  border-radius: 10px;
  overflow: hidden;
}
```

这样，每张卡片都会保持固定的宽高比，即使卡片内容有所不同，整体布局也会显得整齐有序。

- 图表和图形绘制

在绘制图表、流程图等图形时，aspect-ratio可以帮助开发者精确控制图形的比例，使其在不同屏幕上都能正确显示。例如：

```css
.chart {
  width: 600px;
  aspect-ratio: 5 / 3;
  background-color: #f9f9f9;
}
```

通过设置合适的宽高比，图表可以更好地展示数据，提升视觉效果。

# 4. 兼容性和替代方案

虽然aspect-ratio属性功能强大，但在使用时需要注意其浏览器兼容性：

|环境|版本|支持情况|
|--|--|--|
|Chrome|89+|支持|
|Firefox|87+|支持|
|Edge|89+（新版本基于Chromium）|支持|
|Safari|15.4+|支持|

对于不支持aspect-ratio属性的旧版浏览器，可以使用 JavaScript 来实现类似的效果。或者利用css里`百分比 padding 是相对于父容器宽度计算`的这个特性来实现：

- 通过 JavaScript 监听窗口大小变化，动态计算并设置元素的高度：

```html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Aspect Ratio Fallback</title>
  <style>
    .fallback-box {
      width: 100%;
      background-color: lightgreen;
    }
  </style>
</head>

<body>
  <div class="fallback-box" id="fallbackBox"></div>
  <script>
    const fallbackBox = document.getElementById('fallbackBox');
    const ratio = 16 / 9;

    function setBoxHeight() {
      const width = fallbackBox.offsetWidth;
      fallbackBox.style.height = (width / ratio) + 'px';
    }

    window.addEventListener('load', setBoxHeight);
    window.addEventListener('resize', setBoxHeight);
  </script>
</body>
</html>
```

- css里`百分比 padding 是相对于父容器宽度计算`的特性

```html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Aspect Ratio Fallback</title>
  <style>

    .container {
		  width: 400px; /* 可以根据需要调整宽度 */
		  position: relative;
		}
		/* 设置 .box 的 height: 0，然后通过 padding-top: 75% 来创建高度。 因为 padding-top 的百分比是基于父容器的宽度计算 */
		/* 所以 75% 表示高度是宽度的 75%，即实现了 4:3 的比例（3 ÷ 4 = 0.75）。 */
		.box {
		  height: 0;
		  padding-top: 75%; /* 3/4 = 0.75 = 75% */
		  background-color: lightblue;
		  position: relative;
		}
		
		/* 如果你想在盒子里放内容，可以用一个绝对定位的子元素 */
		.box-content {
		  position: absolute;
		  top: 0;
		  left: 0;
		  width: 100%;
		  height: 100%;
		  display: flex;
		  align-items: center;
		  justify-content: center;
		}
  </style>
</head>

<body>
  <div class="container">
	  <div class="box"></div>
	</div>
</body>
</html>
```

这2种方式可以在不支持aspect-ratio属性的浏览器中，模拟出类似的宽高比效果。

# 5. 总结

aspect-ratio属性为 CSS 开发者提供了一种简单而有效的方式来控制元素的宽高比，极大地简化了响应式设计和布局的实现过程。尽管存在一定的浏览器兼容性问题，但随着主流浏览器的不断更新，该属性的应用将会越来越广泛。在实际项目中，合理运用aspect-ratio属性，可以提升页面的视觉效果和用户体验，使布局更加灵活和美观。