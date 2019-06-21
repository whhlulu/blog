# 性能优化-探究WebP

![0db963b36ed059dd4f1623198f67011b.jpeg](evernotecid://D0AE4054-ADFA-401E-B09B-0C01AF6E4F1D/appyinxiangcom/13527544/ENResource/p1994)

## 前言
> 不管是PC还是移动端，图片一直是流量大头。
> 
> 评价网站性能好坏的一个主要指标就是页面响应时间，也就是说用户打开完整页面的时间。基于JPEG还有PNG图片格式的网页，其图片资源加载往往都占据了页面耗时的主要部分，那么如何保证图片质量的前提下缩小图片体积，成为了一件有价值的事情。
> 
> WebP是一种支持有损压缩和无损压缩的图片文件格式，根据Google的测试，无损压缩后的WebP比PNG文件少了`26％`的体积，有损压缩后的WebP图片相比于等效质量指标的JPEG图片减少了`25％~34%`的体积。
> 
> 通过研究WebP图片格式，尽可能全面地了解WebP图片的优劣势以及应用WebP图片给我们带来的收益以及风险，最终提升用户体验。
> 
> 出自 [凹凸实验室-探究WebP一些事儿](https://aotu.io/notes/2016/06/23/explore-something-of-webp/index.html)

## 一、WebP探究
### 1.WebP 编码如何运作？
> WebP 的有损编码旨在于静止图像方面与 JPEG 一较高下。 WebP 的有损编码包括三个关键阶段：`宏块` `预测` ... [能力之所不及具体参见谷歌-自动化图像](https://developers.google.com/web/fundamentals/performance/optimizing-content-efficiency/automating-image-optimization/?hl=zh-cn#how-does-webp-encoding-work)


### 2.WebP兼容性
WebP目前支持桌面上的Chrome、Opera和Fixfox浏览器，
手机支持原生的Android浏览器、Android系统上的所有浏览器、及iOS系统的QQ、Baidu浏览器（目前iOS系统兼容性较差）。

![2b9cf4e31483aac95d28eab55b3a1e4d.png](evernotecid://D0AE4054-ADFA-401E-B09B-0C01AF6E4F1D/appyinxiangcom/13527544/ENResource/p1993)

根据对目前`2019年06月20日`浏览器占比与WebP的兼容性分析，如果采用WebP图片，大约有`78%`的用户可以直接体验到。

### 3.如何将图像转换为 WebP？
`注意` 为转换应用馈送最佳质量的源文件，最好是原件。

[Imagemin](https://github.com/imagemin/imagemin) 是流行的图像缩小模块，还具有用于将图像转换为 WebP (imagemin-webp) 的插件， 而且支持有损模式和无损模式。

此外，也提供 Sindre Sorhus 基于 imagemin-webp 构建的[适用于 Gulp 的 WebP 插件](https://github.com/sindresorhus/gulp-webp)，以及[适用于 WebPack 的 WebP 加载器](https://www.npmjs.com/package/webp-loader)。

通常情况下，您不想将所有图像转换为WebP格式，您只想制作备用版本。您可以使用多加载器来实现它：[multi-loader](https://github.com/webpack-contrib/multi-loader)

### 4.如何提供 WebP 图像？
> 不支持 WebP 的浏览器最终可能根本无法显示图像，这并不是理想的情况。 若要避免此问题，我们可以使用几个策略，在浏览器支持的情况下，有条件地提供 WebP 图像。

#### [服务器配置](https://developers.google.com/web/fundamentals/performance/optimizing-content-efficiency/automating-image-optimization/?hl=zh-cn#how-do-i-serve-webp)
使用 .htaccess 提供 WebP 副本,配合浏览器可以通过 Accept 标头显式发出 WebP 支持信号。

1. Apache：将以下代码添加到 .htaccess 文件
2. Nginx：将以下代码添加到 mime.types 文件
- 依赖 lua + nginx + ImageMagic 实现，对支持webp的请求设置cookies。利用nginx检测图片请求是否存在，如果不存在通过lua调用imageMagic创建webp图片并返回。

#### 客户端配置
1. 使用 <picture> 标记
2. 客户端判断是否支持 发送动态请求

```js
var isSupportWebp = function () {
  try {
    return document.createElement('canvas').toDataURL('image/webp', 0.5).indexOf('data:image/webp') === 0;
  } catch(err) {
    return false;
  }
}

isSupportWebp()
```

```js
function check_webp_feature(feature, callback) {
    var kTestImages = {
        lossy: "UklGRiIAAABXRUJQVlA4IBYAAAAwAQCdASoBAAEADsD+JaQAA3AAAAAA",
        lossless: "UklGRhoAAABXRUJQVlA4TA0AAAAvAAAAEAcQERGIiP4HAA==",
        alpha: "UklGRkoAAABXRUJQVlA4WAoAAAAQAAAAAAAAAAAAQUxQSAwAAAARBxAR/Q9ERP8DAABWUDggGAAAABQBAJ0BKgEAAQAAAP4AAA3AAP7mtQAAAA==",
        animation: "UklGRlIAAABXRUJQVlA4WAoAAAASAAAAAAAAAAAAQU5JTQYAAAD/////AABBTk1GJgAAAAAAAAAAAAAAAAAAAGQAAABWUDhMDQAAAC8AAAAQBxAREYiI/gcA"
    };
    var img = new Image();
    img.onload = function () {
        var result = (img.width > 0) && (img.height > 0);
        callback(feature, result);
    };
    img.onerror = function () {
        callback(feature, false);
    };
    img.src = "data:image/webp;base64," + kTestImages[feature];
}
```

## 二、WebP前端应用方案
### 1.项目使用参考步骤
- 生成webp？
- 判断浏览器动态渲染图片？

问题？  js？css？
问题？使用image-webpack-loader或者使用webp-loader有没有做动态判定？还是multi-loader的作用？
这部分未完结。。。

### 2.阿里云使用优化
https://help.aliyun.com/document_detail/44703.html?spm=a2c4g.11186623.6.1381.110b57f92aqECO

### 3.React-Native改造
在 Android 上支持 WebP 格式图片（rn-ios情况如何？？）
- 需要在android/app/build.gradle文件中根据需要手动添加以下模块
``` java
dependencies {
// 如果你需要支持Android4.0(API level 14)之前的版本
implementation 'com.facebook.fresco:animated-base-support:1.10.0'
// 如果你需要支持WebP格式，包括WebP动图
implementation 'com.facebook.fresco:animated-webp:1.10.0'
implementation 'com.facebook.fresco:webpsupport:1.10.0'
// 如果只需要支持WebP格式而不需要动图
implementation 'com.facebook.fresco:webpsupport:1.10.0'
}
```

#### 参考资料
- [谷歌webp](https://developers.google.com/web/fundamentals/performance/optimizing-content-efficiency/automating-image-optimization/?hl=zh-cn)
- [京东凹凸实验室](https://aotu.io/notes/2016/06/23/explore-something-of-webp/index.html)


