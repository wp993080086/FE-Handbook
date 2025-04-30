```tsx
npm install --save stackblur-canvas
```

```tsx
import * as StackBlur from 'stackblur-canvas'
```

```tsx
// #region 数据
const imageUrl = 'https://img-test.crush.love/static/picture/20250120/anime-gen-out/1330829453185515520_2.jpg'
const [displayImage, setDisplayImage] = useState(imageUrl)

/**
 * @description 获取模糊图片Src
 */
const handleGetImageBlurSrc = (originalImage: string, scale: 0.5): Promise<string> => {
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
 * @description 设置模糊img标签
 */
const handleImageBlur = () => {
  const image = new Image()
  image.crossOrigin = 'anonymous'
  image.src = imageUrl
  image.onload = function () {
    const canvas = document.createElement('canvas')
    const ctx = canvas.getContext('2d')!
    const finalW = image.width / 2
    const finalH = image.height / 2
    canvas.width = finalW
    canvas.height = finalH
    ctx.drawImage(image, 0, 0, finalW, finalH)
    // 使用 StackBlur 让图片变模糊
    const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height)
    StackBlur.imageDataRGB(imageData, 0, 0, canvas.width, canvas.height, 50)
    ctx.putImageData(imageData, 0, 0)
    const blurredImage = new Image()
    const finalSrc = canvas.toDataURL('image/jpeg')
    blurredImage.src = finalSrc
    document.getElementById('test')!.appendChild(blurredImage)
  }
}

/**
 * @description 修改图片
 */
const handleChangeImage = async () => {
  const res = await handleImageBlur(imageUrl, 0.5)
  setDisplayImage(res)
}
// #endregion

// #region 视图
return (
  <div className={moduleStyle['wrapper']}>
    <div className={moduleStyle['button']}>
      <span onClick={handleChangeImage}>Button</span>
    </div>
    <div className={moduleStyle['page']}>
      <img src={displayImage} alt="image" />
    </div>
  </div>
)
// #endregion
```