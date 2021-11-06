# html2canvas 用法及踩坑实录

## 什么是 html2canvs?

`html2canvas` 的作用就是允许让我们直接在用户浏览器上拍摄网页或其部分的“截图”。

它的屏幕截图是基于 `DOM` 的，因此可能不会 100% 精确到真实的表示，因为它不会生成实际的屏幕截图，而是基于**页面上可用的信息**构建屏幕截图。

## html2canvas 可以用来做什么

从上的面的介绍可以知道， `html2canvas` 的作用就是根据 `DOM` 生成对应的图片，所以一个比较常见的应用场景就是我们可以使用它在 `H5` 端生成分享图。

## 如何使用 html2canvas

```js
let html2canvas = null;

export default {
  beforeMount() {
    import('html2canvas').then((plugin) => {
      html2canvas = plugin.default;
    });
  },
  methods: {
    // 获取分享图片 base64
    getShareImgBase64() {
      return new Promise((resolve) => {
        setTimeout(() => {
          // #capture 就是我们要获取截图对应的 DOM 元素选择器
          html2canvas(document.querySelector('#capture'), {
            useCORS: true, // 【重要】开启跨域配置
            scale: window.devicePixelRatio < 3 ? window.devicePixelRatio : 2,
            allowTaint: true, // 允许跨域图片
          }).then((canvas) => {
            const imgData = canvas.toDataURL('image/jpeg', 1.0);
            resolve(imgData);
          });
        }, 300); // 这里加上 300ms 的延迟是为了让 DOM 元素完全渲染完成后再进行图片的生成
      });
    },
  },
};
```
## 常见问题总结

### 图片跨域

我们在进行图片保存的时候经常会发现图片跨域了

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b321ad572bb7406f9bb7be9e85fd9da5~tplv-k3u1fbpfcp-watermark.image?)

这个时候我们去看它的请求，可以看到它本身就没有做跨域的相关配置。

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2b41993340264f13832f11e35ad5f80c~tplv-k3u1fbpfcp-watermark.image?)

对于允许跨域的图片我们可以在 `Headers` 里面看到
```js
Access-Control-Allow-Origin:*
```

对于这个问题，最简单的解决方案就是直接在所在图片的 `img` 标签里面加上 `crossOrigin = "anonymous"`，即：
```js
<img crossorigin="anonymous" src="xxx" >
```

在某些情况下如果你发现加上 `crossOrigin = "anonymous"` 之后，图片显示不出来了，此时给图片的 `url` 中拼上一个随机字符串即可。
```js
<img crossorigin="anonymous" :src="`xxx?_=${Date.now()}`" >
```

当然，想要永久的解决这个问题需要后端同学配合在图片服务器上设置 图片服务器配置 `Access-Control-Allow-Origin: *`。

### 本地图片统跳调试正常，上线之后APP中无法显示

这个问题比较蛋疼，目前也没找到导致该问题的具体原因，经过尝试之后一个可行的解决方案是将本地图片上传至 `OSS`，然后在 `img` 的 `src` 属性中加上时间戳，即：
```js
<img crossorigin="anonymous" :src="`xxx?_=${Date.now()}`">
```

### 截图不全

要解决这个问题，我们只需要在截图之前将页面滚动到顶部即可

```js
document.body.scrollTop = document.documentElement.scrollTop = 0;
```

