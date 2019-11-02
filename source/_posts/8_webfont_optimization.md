---
title: 前端优化 - Web Font
date: 2019-10-29 22:18:18
tags: [Web Font, font-spider, Blog, Hexo]
categories: [Web Frontend]
---

前段时间发现本站点的首屏加载很慢，Chrome 上的 favicon 一直在转圈，十分影响体验。于是有了优化这块的想法~

<!--more-->

## 找出慢的原因

让我们先打开浏览器的开发者模块，切换到 Network Tab 下，关闭缓存 (Disable cache) 和打开历史日志 (Preserve log) 便于之后的调试。然后刷新本站点，观察 Network 日志：

![本站点首屏加载信息](/images/posts/8_webfont_optimization/slowly_load_assets.png)

我们可以发现 langting.woff2 这个文件加载了近两分钟半钟，粗略计算本次渲染的白屏、首屏时间 (毫秒)：

![白屏、首屏时间信息](/images/posts/8_webfont_optimization/abovethefold.png)

白屏时间 (浏览器开始显示内容的时间) 本次请求约等于服务器返回的 Document 文件时间，可以忽略不计。

首屏时间 (用户打开网站开始，到浏览器首屏内容渲染完成的时间) 本次请求以网页整个 load 时间计算，约等于 langting.woff2 的加载时间。当然也可以根据业务需求使用其他指标计算，如最后图片的加载结束时间计算。

本站点在图片等资源加载完后，还加载 langting.woff2 一段很长的时间，十分影响体验和观感，后面让我们优化它。

## 网页字体介绍

在介绍 WOFF2 文件是什么前，让我们先了解网页字体(Web Font)。网页字体是一个字形集合，每个字形是描述字母或符号的矢量形状。这意味着网页字体大小由集合中的字形数量及每个字形矢量路径的复杂程度决定。所以在选取字体时，重要的是考虑它支持的字符集。

### 字体文件格式

目前网络上使用的字体有 4 种格式文件: EOT、 TTF、 WOFF 和 WOFF2。遗憾的是，现在仍没有一种通用的字体文件格式能被所有浏览器支持： EOT 只有 IE 支持，TTF 获得部分 IE 支持，WOFF 得到广泛支持，但一些较旧的浏览器不支持，而 WOFF2 也尚未被大多浏览器支持。

没有一种通用的格式文件，意味着我们需要提供多种格式文件给浏览器：

- 将 WOFF2 格式文件提供给支持的浏览器
- 将 WOFF 格式文件提供给大部分支持的浏览器
- 将 TTF 格式文件提供给旧版浏览器 (Android 4.4 以下)
- 将 EOT 格式文件提供给旧版 IE 浏览器 (IE 9 以下)

### 定义字体

我们可以使用 `@font-face` 去定义一种字体，使用如下:

```css
@font-face {
  font-family: 'lanting';
  font-weight: 300;
  src: local('Lantinghei SC'),
       url('/fonts/lanting/lanting.woff2') format('woff2'),
       url('/fonts/lanting/lanting.woff') format('woff'),
       url('/fonts/lanting/lanting.ttf') format('truetype'), /* chrome, firefox, opera, Safari, Android, iOS 4.2+ */
       url('/fonts/lanting/lanting.eot?#iefix') format('embedded-opentype'), /* IE6-IE8 */
       url('/fonts/lanting/lanting.eot'); /* IE9 */
}

body {
  font-family: 'Microsoft Jhenghei', 'lanting', PingFang SC, sans-serif;
}
```

上面一开始我们声明了字体名称为 `lanting`, 并为它设置 300 的粗细，然后根据浏览器设置它的字体资源文件路径，其中 `local` 映射到本地系统上的 `Lantinghei SC` 字体。

当浏览器确定需要字体时，其就会遍历 `body` 的 `font-family`, 检查本地系统是否存在相关字体。如果不存在，浏览器会查看该字体是否有声明的格式，若浏览器支持该字体格式且字体定义了资源路径，此时浏览器将会下载字体资源。

## 优化网页字体加载

前面我们提到浏览器下载字体资源的策略，那浏览器是怎样加载这些资源的呢？

### 默认行为

默认情况下，浏览器在构建渲染树(它依赖 DOM 和 CSSOM 树)之前会延迟字体请求，因为只有构建出渲染树，浏览器才知道使用哪些字体资源来渲染文本。这就可能会导致文本渲染延迟。

![浏览器渲染字体过程](/images/posts/8_webfont_optimization/webfont_crp.png)

上图展示了浏览器渲染字体经历的过程:

1. 浏览器请求 HTML 文档。
2. 浏览器开始解析 HTML 响应和构建 DOM。
3. 浏览器发现 CSS、JS 以及其他资源并分派请求。
4. 浏览器在收到所有 CSS 内容后构建 CSSOM, 然后将其与 DOM 树合并以构建渲染树。
  - 在渲染树指示需要哪些字体在网页上渲染指定文本后，将分派字体请求。
5. 浏览器执行布局并将绘制内容。
  - 如果字体尚不可用，浏览器可能不会渲染任何文本。
  - 字体可用之后，浏览器将绘制文本像素。

由此，我们可以了解到网页首次绘制与字体资源请求之间的“竞赛”产生了“空白文本的问题”。

### 预加载网页字体资源

如果你事先知道网页要加载的特定字体资源，我们可以提前让浏览器去加载这些资源，而不必等到 DOM 树构建完之后。此时 `<link rel="preload">` 标签就能用上场了，该标签允许你在 `<head>` 上声明一个资源，提早触发对该资源的请求。

```html
<head>
    <link rel="preload" href="/fonts/awesome-l.woff2" as="font">
</head>
```

并非所有浏览器都支持 `<link rel="preload">`, 不支持的浏览器将忽略此标签。

**Note**: `<link rel="preload">` 将无条件优化发送字体资源请求，而不考虑页面实际是否使用到此字体。如果此时浏览器本地安装了此字体，那么这个请求就是多余的。预加载但不实际使用资源时，某些浏览器会在其开发者工具控制台中显示警告。

### 自定义文本渲染延迟

虽然预加载增加网络字体被及时使用的可能性，但并不保证如此。你仍需要考虑所渲染的文本使用了尚未可用的 `font-family` 时的行为。

网页内容首次绘制与字体资源请求之前产生的“竞争”导致“空白文本问题”，出现该问题时，浏览器会在渲染内容时遗漏所有文本。大部分浏览器在等待字体下载时都执行最大超时策略，超时之后将使用回退字体渲染文本。不过，各个浏览器的执行方式并不相同:

- Chrome 和 Firefox 的超时时间是 3 秒，在此之后，将使用回退字体显示文本。等字体下载成功后，将发生交换，以期望字体重新渲染文本。
- IE 超时时间为零秒，这表示文本得以立即渲染。如果字体尚不可用，则使用回退字体，之后可用后再重新渲染文本。
- Safari 没有超时行为

为确保之后的一致性，我们可以在 CSS 中使用 `font-display` 描述 `@font-face`。`font-display` 支持以下取值:

- `auto` 使用浏览器默认的字体显示策略。现大多浏览器使用类似 `block` 的策略。
- `block`　为字体指定较短的阻止期 (大部分情况，建议值为 3 秒) 以及无限的交换期，类似 Chrome 的默认行为。
- `swap` 为字体指定零秒的阻止期以及无限的交换期，类似 IE 的默认行为。
- `fallback` 为字体指定极短的阻止期 (大部分情况建议值为 100 毫秒或更短) 以及较短交换期 (大部分情况建议值为 3 秒)。fallback 非常适合于正文等内容，对于这些内容，您希望用户尽快开始阅读，不想让新字体载入时发生的文本移动干扰其体验。
- `optional` 为字体指定极短的阻止期（在大部分情况下，建议值为 100 毫秒或更短）以及零秒的交换期。此值非常适合在下载的字体可以“锦上添花”，但对于用户体验并非至关重要时使用。

### Font Loading API

我们配合使用 `<link rel="preload">` 与 `font-display` 可以较好的控制字体加载与渲染，而不必增加额外的开销。如果需要进一步定义，则需要承担 JavaScript 所引入的开销。 Font Loading API 提供了定义和操纵 CSS 字体，跟踪字体下载进度以及替换默认渲染行为的能力。例如你需要特定字体，并在其加载成功后显示内容：

```javascript
var font = new FontFace("Awesome Font", "url(/fonts/awesome.woff2)", { style: 'normal' })

font.load().then(function () {
  document.fonts.add(font);
  document.body.style.fontFamily = "Awesome Font, serif";

  var content = document.getElementById("content");
  content.style.visibility = "visible";
})
```

## 压缩网页字体

字体是字形的集合，其中每个字形都是一组描述字符形状的路径。我们可以通过压缩字形中相似信息或者缩减字形范围，从而达到压缩字体的效果。

### 压缩字形

各个字形虽然不同，但它们包含大量相似的信息。这些信息可通过 GZIP 或兼容的压缩工具进行压缩:

- EOT 和 TTF 格式默认情况下不进行压缩。提供这些格式时，确保你服务器配置为应用 GZIP 压缩。
- WOFF 具有内建压缩。 确保你的 WOFF 压缩工具使用了最佳压缩设置。
- WOFF2 采用自定义预处理和压缩算法，提供的文件大小压缩率比其他格式高大约 30%。

### 提取字形

我们可以使用一些工具自动分析出页面中使用的 Web Font 并按需进行压缩，如 [font-spider](https://github.com/aui/font-spider)。font-spider 只支持静态网页，现在我们尝试使用 font-spider 为 hexo 静态网页压缩字体:

```sh
$ npm install -g font-spider # 安装 font-spider
$ hexo generate # hexo 生成静态网页
$ sed -i "s~/css/style.css~`pwd`/public/css/style.css~g" ./public/**/*.html # 将网页中的 css 资源资源链接替换成文件系统的绝对地址(资源链接根据你的实际情况进行替换)
$ font-spider ./public/**/*.html --debug # 使用 font-spider 压缩字体
$ sed -i "s~`pwd`/public/css/style.css~/css/style.css~g" ./public/**/*.html # 替换回 css 资源链接
$ hexo deploy  # 发布
```

通过执行以上命令我们就能为使用到的字体进行 “瘦身”。结合 CI，我们还可以做到发布时自动压缩字体～

## 总结

当我们决定要在网页中使用一种字体时，需要对该字体的系统普及度做调查。如果是一种不常用字体，此时我们需要下载资源并对其进行优化。在优化字体时，首先考虑先压缩字体或者使用 CDN 加载字体下载，再考虑在客户端上进行处理。一切从用户体验上出发，如果因为字体影响了用户体验，此时我们就要考虑此字体是否有必要引用了。

## 参考

1. [web前端基础资料 - 性能](https://www.kancloud.cn/jonjo/jonjo/1025441)
2. [细说 window.performance](https://zhangxiang958.github.io/2017/05/20/%E7%BB%86%E8%AF%B4%20window.performance/)
3. [网页字体优化](https://developers.google.com/web/fundamentals/performance/optimizing-content-efficiency/webfont-optimization?hl=zh-CN)