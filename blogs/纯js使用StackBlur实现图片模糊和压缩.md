# 1. 前言

在现代 Web 项目开发的过程中，图片处理是一个常见且重要的需求。其中，图片模糊效果的实现往往能为界面增添独特的视觉风格。近期笔者在项目中遇到了这样的需求，经过多方面的技术调研与方案对比，最终采用StackBlur库成功实现了纯前端图片模糊与压缩功能。本文将详细阐述整个实现过程，希望能为同样有此类需求的开发者提供有益的参考与借鉴。
![模糊对比](https://i-blog.csdnimg.cn/direct/fdcea34a4f0f48b7b29e951b73114559.png#pic_center)
# 2. 安装依赖和引入插件

对于基于npm管理的项目，执行以下命令即可完成安装：

```shell
npm install --save stackblur-canvas
```

安装完成后，在代码中引入该库，以便后续使用其功能：

```shell
import * as StackBlur from'stackblur-canvas';
```

如果需要进一步了解StackBlur库的详细用法和特性，可以查阅其官方文档，文档地址为：[StackBlur](https://github.com/flozz/StackBlur)

# 3. 使用

为了更直观地展示StackBlur库实现图片模糊与压缩的过程，下面通过一个具体的 React 示例进行说明。在这个示例中，界面包含一个复原按钮和一个模糊按钮，点击模糊按钮将触发图片模糊化与压缩操作，点击复原按钮则能使图片恢复至原始状态。

```tsx
// #region 数据
const imageUrl = 'https://xxxxx.jpeg'
const [displayImage, setDisplayImage] = useState(imageUrl)
// #endregion

// #region ------------------------------------------------------逻辑
	/**
	 * @description 获取模糊图片
	 * @param {string} originalImage 原图片
	 * @param {number} scale 缩放
	 */
	const handleGetBlurImage = ({ originalImage, scale = 0.5 }: { originalImage: string; scale?: number }): Promise<string> => {
		return new Promise((resolve, reject) => {
			try {
				const image = new Image()
				image.crossOrigin = 'anonymous'
				image.src = originalImage
				image.onload = function () {
					const canvas = document.createElement('canvas')
					const ctx = canvas.getContext('2d')!
					const finalW = image.width * scale
					const finalH = image.height * scale
					canvas.width = finalW
					canvas.height = finalH
					ctx.drawImage(image, 0, 0, finalW, finalH)
					// 使用 StackBlur 让图片变模糊
					const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height)
					StackBlur.imageDataRGB(imageData, 0, 0, canvas.width, canvas.height, 50)
					ctx.putImageData(imageData, 0, 0)
					const finalSrc = canvas.toDataURL('image/jpeg')
					resolve(finalSrc)
				}
				image.onerror = function () {
					throw new Error()
				}
			} catch (_) {
				return reject(originalImage)
			}
		})
	}

	/**
	 * @description 还原图片
	 */
	const handleResetImage = async () => {
		setDisplayImage(networkImage)
	}

	/**
	 * @description 设置图片模糊
	 */
	const handleSetImageBlur = async () => {
		const res = await handleGetBlurImage({
			originalImage: networkImage
		})
		setDisplayImage(res)
	}
	// #endregion

	// #region 视图
	return (
		<div className={moduleStyle['test-wrapper']}>
			<div className={moduleStyle['operate-wrapper']}>
				<div className={moduleStyle['reset-button']} onClick={handleResetImage}>
					复原
				</div>
				<div className={moduleStyle['blur-button']} onClick={handleSetImageBlur}>
					模糊
				</div>
			</div>
			<div className={moduleStyle['image-wrapper']}>
				<img src={displayImage} alt="image" crossOrigin="anonymous" />
			</div>
		</div>
	)
	// #endregion
```

上述`tsx`代码定义了界面的布局结构，包含操作按钮区域和图片展示区域，通过为按钮绑定相应的点击事件处理函数，实现交互功能。

# 4. 总结

通过上述实践，我们成功实现了纯前端图片模糊与压缩功能。其核心原理是利用canvas强大的绘图能力，先将图片缩放到指定大小，再借助StackBlur库对图片数据进行模糊处理，最后将处理后的内容重新绘制到canvas上，并转换为base64格式的图片 URL。这种方式无需依赖后端服务，在前端即可快速高效地完成图片处理，为项目开发提供了灵活的解决方案。同时，开发者可以根据实际需求，调整缩放比例、模糊半径等参数，以达到理想的图片处理效果 。