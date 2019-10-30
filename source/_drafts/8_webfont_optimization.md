---
title: 优化 Blog - Web Font
date: 2019-10-29 22:18:18
tags: [Web Font, Font-spider, Blog, Hexo]
categories: [Web Frontend]
---

前段时间发现本站点的首屏加载很慢，Chrome 上的 favicon 一直在转圈，十分影响体验。于是就有了优化这块的想法~

<!--more-->

## 找出慢的原因

让我们先打开浏览器的开发者模块，切换到 Network Tab 下，关闭缓存 (Disable cache) 和打开历史日志 (Preserve log) 便于之后的调试。然后刷新本站点，观察 Network 日志：

![本站点首屏加载信息](/images/posts/8_webfont_optimization/slowly_load_assets.png)

我们可以发现 langting.woff2 这个文件加载了近两分钟半钟，粗略计算本次渲染的白屏、首屏时间 (毫秒)：

![白屏、首屏时间信息](/images/posts/8_webfont_optimization/abovethefold.png)

白屏时间 (浏览器开始显示内容的时间) 本次请求约等于服务器返回的 Document 文件时间，可以忽略不计。

首屏时间 (用户打开网站开始，到浏览器首屏内容渲染完成的时间) 本次请求以网页整个 load 时间计算，约等于 langting.woff2 的加载时间。当然也可以根据业务需求使用其他指标计算，如最后图片的加载结束时间计算。

本站点在图片等资源加载完后，还加载 langting.woff2 一段很长的时间，十分影响体验和观感，后面让我们优化它。

## Web Font 介绍

## 优化 Web Font

## 参考

1. [web前端基础资料 - 性能](https://www.kancloud.cn/jonjo/jonjo/1025441)
2. [细说 window.performance](https://zhangxiang958.github.io/2017/05/20/%E7%BB%86%E8%AF%B4%20window.performance/)
3. [网页字体优化](https://developers.google.com/web/fundamentals/performance/optimizing-content-efficiency/webfont-optimization?hl=zh-CN)