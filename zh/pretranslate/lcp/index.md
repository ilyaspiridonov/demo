---
layout: post
title: 最大的内容绘制 (LCP)
authors:
  - philipwalton
date: '2019-08-08'
updated: '2020-06-17'
description: |2

  这篇文章介绍了最大内容绘制 (LCP) 指标并解释了

  如何测量
tags:
  - performance
  - metrics
---

{% Aside %}Largest Contentful Paint (LCP) 是衡量[感知加载速度的](/user-centric-performance-metrics/#types-of-metrics)一个重要的、以用户为中心的指标，因为它在页面的主要内容可能已加载时标记了页面加载时间线中的点——快速的 LCP 有助于让用户确信页面[很有用](/user-centric-performance-metrics/#questions)。 {% endAside %}

从历史上看，对于 Web 开发人员来说，衡量网页主要内容的加载速度和对用户可见的速度一直是一个挑战。

[像 load](https://developer.mozilla.org/en-US/docs/Web/Events/load)或[DOMContentLoaded](https://developer.mozilla.org/en-US/docs/Web/Events/DOMContentLoaded)这样的旧指标并不好，因为它们不一定对应于用户在屏幕上看到的内容。而更新的、以用户为中心的性能指标，如[First Contentful Paint (FCP)](/fcp/)只捕捉加载体验的最开始。如果页面显示启动画面或显示加载指示器，则这一时刻与用户不太相关。

过去，我们推荐了性能指标，例如[首次有意义的绘制 (FMP)](/first-meaningful-paint/)和[速度指数 (SI)](/speed-index/) （两者都在 Lighthouse 中可用）来帮助捕获初始绘制后的更多加载体验，但这些指标很复杂，难以解释，而且通常是错误的——这意味着他们仍然无法识别页面的主要内容何时加载。

有时越简单越好。根据[W3C Web 性能工作组的](https://www.w3.org/webperf/)讨论和 Google 的研究，我们发现衡量页面主要内容何时加载的更准确方法是查看最大元素何时呈现。

## 什么是LCP？

最大内容绘制 (LCP) 指标报告视口内可见[的最大图像或文本块](#what-elements-are-considered)[的渲染时间，相对于页面首次开始加载的时间](https://w3c.github.io/hr-time/#timeorigin-attribute)。

<picture>
  <source srcset="{{ " image imgix media="(min-width: 640px)" width="400" height="100">{% Img src="image/eqprBhZUGfb8WYnumQ9ljAxRrA72/8ZW8LQsagLih1ZZoOmMR.svg", alt="Good LCP values are 2.5 seconds, poor values are greater than 4.0 seconds and anything in between needs improvement", width="400", height="300", class="w-screenshot w-screenshot--filled width-full" %}</source></picture>

### 什么是好的 LCP 分数？

为了提供良好的用户体验，网站应努力将最大内容绘制**时间设为 2.5 秒**或更短。为确保您的大多数用户都能达到此目标，一个很好的衡量阈值是**页面加载的第 75 个百分点**，在移动设备和桌面设备之间进行细分。

{% Aside %}要详细了解此建议背后的研究和方法，请参阅：[定义核心 Web Vitals 指标阈值](/defining-core-web-vitals-thresholds/){% endAside %}

### 考虑哪些要素？

目前在[Largest Contentful Paint API 中](https://wicg.github.io/largest-contentful-paint/)指定，Largest Contentful Paint 考虑的元素类型是：

- `<img>`元素
- `<svg>`元素内的`<image>`
- `<video>`元素（使用海报图片）
- [`url()`](https://developer.mozilla.org/en-US/docs/Web/CSS/url())函数加载的背景图像的元素（与[CSS 渐变](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Images/Using_CSS_gradients)相反）
- 包含文本节点或其他内联级文本元素子级的[块级元素。](https://developer.mozilla.org/en-US/docs/Web/HTML/Block-level_elements)

请注意，将元素限制在这个有限的集合中是有意的，以便在开始时保持简单。随着更多研究的进行，未来可能会添加其他元素（例如`<svg>` 、 `<video>`

### 元素的大小是如何确定的？

为最大内容绘制报告的元素大小通常是用户在视口内可见的大小。如果元素延伸到视口之外，或者任何元素被剪裁或具有不可见的[溢出](https://developer.mozilla.org/en-US/docs/Web/CSS/overflow)，则这些部分不计入元素的大小。

[对于从其固有大小](https://developer.mozilla.org/en-US/docs/Glossary/Intrinsic_Size)调整大小的图像元素，报告的大小是可见大小或固有大小，以较小者为准。例如，缩小到远小于其内在尺寸的图像将仅报告其显示时的尺寸，而拉伸或扩展至更大尺寸的图像将仅报告其内在尺寸。

对于文本元素，仅考虑其文本节点的大小（包含所有文本节点的最小矩形）。

对于所有元素，不考虑通过 CSS 应用的任何边距、填充或边框。

{% Aside %} 确定哪些文本节点属于哪些元素有时会很棘手，尤其是对于子元素包括行内元素和纯文本节点以及块级元素的元素。关键是每个文本节点都属于（并且仅属于）其最近的块级祖先元素。在[规范方面](https://wicg.github.io/element-timing/#set-of-owned-text-nodes)：每个文本节点都属于生成其[包含块](https://developer.mozilla.org/en-US/docs/Web/CSS/Containing_block)的元素。 {% endAside %}

### 什么时候报告最大的内容？

网页通常分阶段加载，因此，页面上最大的元素可能会发生变化。

为了处理这种潜在的变化，浏览器在浏览器绘制第一帧后立即`largest-contentful-paint`类型[`PerformanceEntry`](https://developer.mozilla.org/en-US/docs/Web/API/PerformanceEntry)但是，在渲染后续帧之后，它会在最大内容元素发生变化时[`PerformanceEntry`](https://developer.mozilla.org/en-US/docs/Web/API/PerformanceEntry)

例如，在带有文本和英雄图像的页面上，浏览器最初可能只呈现文本——此时浏览器将调度一个`largest-contentful-paint`条目，其`element`属性可能引用一个`<p>`或`<h1>` 。稍后，一旦主图像完成加载，第二`largest-contentful-paint`条目将被调度，其`element`属性将引用`<img>` 。

需要注意的是，一个元素只有在呈现并且对用户可见后才能被视为最大的内容元素。尚未加载的图像不被视为“渲染”。 [在字体块期间](https://developer.mozilla.org/en-US/docs/Web/CSS/@font-face/font-display#The_font_display_timeline)也不使用网络字体的文本节点。在这种情况下，较小的元素可能会被报告为最大的内容元素，但一旦较大的元素完成渲染，它将通过另一个`PerformanceEntry`对象进行报告。

除了延迟加载图像和字体之外，当新内容可用时，页面可能会向 DOM 添加新元素。如果这些新元素中的任何一个大于先前最大的内容元素，则还将报告`PerformanceEntry`

如果当前是最大内容元素的元素从视口中删除（甚至从 DOM 中删除），除非呈现更大的元素，否则它将保持最大内容元素。

{% Aside %} 在 Chrome 88 之前，移除的元素不被视为最大的内容元素，移除当前候选元素会触发一个新的`largest-contentful-paint`条目被调度。但是，由于流行的 UI 模式（例如经常删除 DOM 元素的图像轮播），该指标已更新以更准确地反映用户体验。有关更多详细信息，请参阅[变更日志。](https://chromium.googlesource.com/chromium/src/+/master/docs/speed/metrics_changelog/2020_11_lcp.md) {% endAside %}

一旦用户与页面交互（通过点击、滚动或按键），浏览器将停止报告新条目，因为用户交互通常会改变用户可见的内容（滚动时尤其如此）。

出于分析目的，您应该只向您的分析服务`PerformanceEntry`

{% Aside 'caution' %}由于用户可以在背景标签中打开页面，因此在用户聚焦该标签之前可能不会发生最大的内容绘制，这可能比他们第一次加载它的时间要晚得多。 {% endAside %}

#### 加载时间与渲染时间

[`Timing-Allow-Origin`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Timing-Allow-Origin)标头的跨源图像，不会公开图像的渲染时间戳。相反，只公开它们的加载时间（因为这已经通过许多其他 Web API 公开了）。

下面的[使用示例](#measure-lcp-in-javascript)展示了如何处理渲染时间不可用的元素。但是，在可能的情况下，始终建议设置[`Timing-Allow-Origin`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Timing-Allow-Origin)标头，以便您的指标更加准确。

### 如何处理元素布局和大小更改？

为了保持计算和分派新性能条目的性能开销较低，对元素大小或位置的更改不会生成新的 LCP 候选。仅考虑元素在视口中的初始大小和位置。

这意味着最初在屏幕外渲染然后在屏幕上过渡的图像可能不会被报告。这也意味着最初在视口中呈现的元素然后被推下，视口外仍将报告其初始的视口内大小。

### 例子

以下是一些流行网站上发生最大内容绘制时的一些示例：

{% Img src="image/admin/bsBm8poY1uQbq7mNvVJm.png", alt="Largest Contentful Paint timeline from cnn.com", width="800", height="311" %}

{% Img src="image/admin/xAvLL1u2KFRaqoZZiI71.png", alt="Largest Contentful Paint timeline from techcrunch.com", width="800", height="311" %}

在上面的两个时间线中，最大的元素随着内容加载而变化。在第一个示例中，新内容被添加到 DOM 并且改变了最大的元素。在第二个示例中，布局更改并且以前最大的内容从视口中删除。

虽然延迟加载的内容通常比页面上已有的内容大，但情况并非一定如此。接下来的两个示例显示了在页面完全加载之前发生的最大内容绘制。

{% Img src="image/admin/uJAGswhXK3bE6Vs4I5bP.png", alt="Largest Contentful Paint timeline from instagram.com", width="800", height="311" %}

{% Img src="image/admin/e0O2woQjZJ92aYlPOJzT.png", alt="Largest Contentful Paint timeline from google.com", width="800", height="311" %}

在第一个示例中，Instagram 徽标加载得相对较早，即使其他内容逐渐显示，它仍然是最大的元素。在 Google 搜索结果页面示例中，最大的元素是在任何图像或徽标完成加载之前显示的一段文本。由于所有单个图像都小于此段落，因此在整个加载过程中它仍然是最大的元素。

{% Aside %} 在 Instagram 时间线的第一帧中，您可能会注意到相机徽标周围没有绿框。这是因为它是一个`<svg>`元素，而`<svg>`元素目前不被视为 LCP 候选元素。第一个 LCP 候选是第二帧中的文本。 {% endAside %}

## 如何测量 LCP

LCP 可以[在实验室](/user-centric-performance-metrics/#in-the-lab)或[现场测量](/user-centric-performance-metrics/#in-the-field)，并且可以在以下工具中使用：

### 现场工具

- [Chrome 用户体验报告](https://developers.google.com/web/tools/chrome-user-experience-report)
- [PageSpeed 洞察力](https://developers.google.com/speed/pagespeed/insights/)
- [Search Console（核心网络生命力报告）](https://support.google.com/webmasters/answer/9205520)
- [`web-vitals` JavaScript 库](https://github.com/GoogleChrome/web-vitals)

### 实验室工具

- [Chrome 开发者工具](https://developers.google.com/web/tools/chrome-devtools/)
- [灯塔](https://developers.google.com/web/tools/lighthouse/)
- [网页测试](https://webpagetest.org/)

### 在 JavaScript 中测量 LCP

要在 JavaScript 中测量 LCP，您可以使用[Largest Contentful Paint API](https://wicg.github.io/largest-contentful-paint/) 。以下示例展示了如何创建[`PerformanceObserver`](https://developer.mozilla.org/en-US/docs/Web/API/PerformanceObserver)来侦听`largest-contentful-paint`条目并将它们记录到控制台。

```js
new PerformanceObserver((entryList) => {
  for (const entry of entryList.getEntries()) {
    console.log('LCP candidate:', entry.startTime, entry);
  }
}).observe({type: 'largest-contentful-paint', buffered: true});
```

{% Aside 'warning' %}

此代码显示了如何将`largest-contentful-paint`条目记录到控制台，但在 JavaScript 中测量 LCP 更为复杂。详情请见下文：

{% endAside %}

在上面的例子中，每个记录的`largest-contentful-paint`条目代表当前的 LCP 候选。通常， `startTime`值是 LCP 值——但是，情况并非总是如此。并非所有`largest-contentful-paint`条目都适用于测量 LCP。

以下部分列出了 API 报告的内容与指标计算方式之间的差异。

#### 指标和 API 之间的差异

- API 将为`largest-contentful-paint`条目，但在计算 LCP 时应忽略这些页面。
- API 将`largest-contentful-paint`条目，但在计算 LCP 时应忽略这些条目（只有当页面始终处于前台时才考虑元素）。
- 当页面从[back/forward 缓存](/bfcache/#impact-on-core-web-vitals)恢复时，API 不会报告`largest-contentful-paint`条目，但在这些情况下应该测量 LCP，因为用户将它们体验为不同的页面访问。
- API 不考虑 iframe 中的元素，但要正确测量 LCP，您应该考虑它们。子框架可以使用 API 将它们`largest-contentful-paint`条目报告给父框架以进行聚合。

[`web-vitals` JavaScript 库](https://github.com/GoogleChrome/web-vitals)来衡量 LCP，而不是记住所有这些细微的差异，它会为您处理这些差异（在可能的情况下）：

```js
import {getLCP} from 'web-vitals';

// Measure and log LCP as soon as it's available.
getLCP(console.log);
```

您可以参考[`getLCP()`的源代码](https://github.com/GoogleChrome/web-vitals/blob/master/src/getLCP.ts)，了解如何在 JavaScript 中测量 LCP 的完整示例。

{% Aside %}在某些情况下（例如跨域 iframe），无法在 JavaScript 中测量 LCP。有关详细信息，请参阅`web-vitals`[库的限制](https://github.com/GoogleChrome/web-vitals#limitations)部分。 {% endAside %}

### 如果最大的元素不是最重要的怎么办？

在某些情况下，页面上最重要的元素（或多个元素）与最大的元素不同，开发人员可能更感兴趣的是测量这些其他元素的渲染时间。这可以使用[Element Timing API 实现](https://wicg.github.io/element-timing/)[，如关于自定义指标](/custom-metrics/#element-timing-api)的文章中所述。

## 如何改进 LCP

LCP主要受四个因素影响：

- 服务器响应速度慢
- 阻止渲染的 JavaScript 和 CSS
- 资源加载时间
- 客户端渲染

要深入了解如何改进 LCP，请参阅[优化 LCP](/optimize-lcp/) 。有关也可以改进 LCP 的单个性能技术的其他指导，请参阅：

- [使用 PRPL 模式应用即时加载](/apply-instant-loading-with-prpl)
- [优化关键渲染路径](https://developers.google.com/web/fundamentals/performance/critical-rendering-path/)
- [优化你的 CSS](/fast#optimize-your-css)
- [优化您的图像](/fast#optimize-your-images)
- [优化网页字体](/fast#optimize-web-fonts)
- [优化您的 JavaScript](/fast#optimize-your-javascript) （针对客户端呈现的网站）

## 其他资源

- [安妮·沙利文](https://anniesullie.com/)(Annie Sullivan) 在[performance.now()](https://perfnow.nl/) (2019) 上从 Chrome 中的[性能监控中吸取的教训](https://youtu.be/ctavZT87syI)

{% include 'content/metrics/metrics-changelog.njk' %}
