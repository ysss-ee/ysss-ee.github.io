---
title: '如何设置一个动态背景'
description: '基于astro的博客动态背景实现。'
category: '前端'
pubDate: '2026-08-18'
heroImage: ''
tags:
  - Astro
  - CSS
  - 视频背景
---
# 视频组件

## 组件结构

核心结构其实很简单：

```astro
<div class="site-background" aria-hidden="true">
  <video
    class="site-background-media"
    autoplay
    muted
    loop
    playsinline
    preload="metadata"
    poster="/background.jpg"
  >
    <source src="/video.mp4" type="video/mp4" />
  </video>
  <div class="site-background-tint"></div>
</div>
```

这里有三个重点：

1. `video` 负责动态效果。
2. `poster` 提供视频未加载完成时的封面图。
3. `site-background-tint` 用来压一层色，保证前景文字始终清晰。

## 为什么要这样写

### 1. 让背景始终铺满页面

外层容器用了：

```css
.site-background {
  position: fixed;
  inset: 0;
  z-index: -2;
  overflow: hidden;
  background: #08101f url('/background.jpg') center / cover no-repeat;
  pointer-events: none;
}
```

`position: fixed` 和 `inset: 0` 能让背景固定在视口里，不随页面滚动而移动。
`background` 则是兜底方案，即使视频没加载成功，页面也不会空掉,会显示兜底的图片。

### 2. 视频按比例裁切

```css
.site-background-media {
  width: 100%;
  height: 100%;
  object-fit: cover;
  object-position: center;
  display: block;
}
```

`object-fit: cover` 会让视频像背景图一样铺满容器，必要时裁掉边缘，避免变形。

### 3. 加一层统一色调

```css
.site-background-tint {
  position: absolute;
  inset: 0;
  z-index: 1;
  background: linear-gradient(rgba(8, 12, 28, 0.18), rgba(8, 12, 28, 0.34));
}
```

纯视频背景通常会有明暗变化，会导致正文在上面容易看不清。
加一层半透明渐变后，页面层次会稳定很多，主体也更容易阅读。

## 无障碍处理

我在组件里加了：

```astro
<div class="site-background" aria-hidden="true">
```

因为背景只是装饰，不应该被屏幕阅读器当成内容读取。
另外还加了：

```css
@media (prefers-reduced-motion: reduce) {
  .site-background-media {
    display: none;
  }
}
```

这能照顾到偏好减少动效的用户。

## 实际使用建议

如果你也要做类似效果，我建议优先注意这几件事：

- 视频尽量短、轻、循环自然。(不需要音乐的记得去掉音轨)
- 准备静态封面图，别让页面在加载期间空白。
- 给减少动态效果的用户保留静态方案。

# 加入使用

只需要在Header模块中导入，在最开头加入就好啦

```astro
import BackgroundVideo from './BackgroundVideo.astro';
<BackgroundVideo />
```

# 优化背景显示

前言，一开始为了使页面切换更加柔和自然，我用了`astro:transitions`的clientrouter

再写完动态背景后，由于视频加载问题，在切换页面时即使主页面已经加载好了背景，切换后由于重新加载导致仍会出现短暂的兜底图片，为了解决这个问题，我对整个背景层加了持久化。

然后我发现由于加了持久化，视频背景虽然稳定了，但是原先clientrouter带来的页面切换时淡入淡出的过渡效果没有了，导致整个页面切换十分的突兀，会出现几秒空白只有视频背景的窗口期，一番问题查找下，最后我单独对上层的需要过渡的元素加了淡入淡出效果才解决了显示问题
