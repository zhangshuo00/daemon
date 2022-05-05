+++
author = "zs"
title = "使用加速合成的前端性能优化 -- 第一部分"
date = "2022-04-20"
lastmod = "2022-04-22"
description = ""
tags = [
    "optimise",
]
+++

>* 原文地址：[Front-End Performance Optimization with Accelerated Compositing Part 1](https://www.alibabacloud.com/blog/front-end-performance-optimization-with-accelerated-compositing-part-1_594194)
> * 原文作者：Alibaba Clouder

呈现一个网页通常包括以下几步：

>**JavaScript -> Style -> Layout -> Paint -> Composite**

1. JavaScript：使用JavaScript是为了实现视觉上的变化。比如，添加一个动画或者一些 DOM 元素。

2. Style：通过 css 选择器，把每个 DOM 元素和相应的 CSS 样式匹配在一起。这样每个 DOM 元素的样式也就确定了。

3. Layout：计算要在屏幕上显示的每个 DOM 元素的大小和位置。页面上元素之间的布局是相对的（relative），这意味着一个元素可以影响其他元素。比如，如果元素的宽度发生改变，它的子元素和孙元素也会收到影响。因此，布局的进程也会涉及到浏览器。

4. Paint：本质上来说，绘制是一个填充像素的过程。它需要对 DOM 元素的文本、颜色、图片、边框、阴影和其他可视的效果进行绘制。一般来说，绘制会分为多层来完成。

5. Composite：正如上面说的，DOM 元素的绘制是在页面的许多层上完成的。一旦完成，浏览器会将所有的图层按照正确的顺序合并为一个图层，并在屏幕上显示出来。这个过程对于有重叠元素的页面尤其重要，因为不正确的图层组成顺序可能导致元素的显示异常。

这篇文章将只关注页面开发的“合成 composite”阶段。


# 浏览器的渲染

在介绍页面合成之前，这个部分先简要地介绍各浏览器的渲染原理（本文以 Chrome 浏览器为例），以方便大家理解概念。更多信息请参见[《Chrome 浏览器的 GPU 加速合成》](http://www.chromium.org/developers/design-documents/gpu-accelerated-compositing-in-chrome?spm=a2c65.11461447.0.0.5c88790bw6zV4C)。

>注意：由于 Chrome 浏览器修改了 Blank 引擎的一些实现，许多已知的类名都发生了变化。例如，用 LayoutObject 替换了 RenderObject，用 PaintLayer 替换了 RenderLayer。更多信息请阅读[Slimming Paint](https://www.chromium.org/blink/slimming-paint?spm=a2c65.11461447.0.0.5c88790bw6zV4C)。

浏览器把页面的内容存储为一棵由 Node 对象组成的树，称为 DOM 树。每个 HTML 元素都有一个相关联的 Node 对象。同样的，**DOM 树的根节点总是 Document 节点**。所有人都知道这一点。不过，从 DOM 树到最终渲染的转换映射有一个要求。

# 从节点到图层对象

DOM 树中的每个节点都有一个与之对应的图层对象（LayoutObjects），它知道如何在屏幕上绘制节点的内容。

# 从图层对象到绘制图层

一般来说，具有相同空间坐标的 LayoutObjects 属于同一 PaintLayer。PaintLayer 最初的用途是层叠上下文，以保证页面元素以正确的顺序合成，从而使重叠和半透明的元素能够正确现实。
因此，PaintLayer 的创建必须是针对满足层叠上下文条件的 LayoutObjects，以及特殊情况下的特殊 LayoutObjects，比如 overflow != visible 的元素。
以下是根据创建原因将 PaintLayers 分为三种类型：

1. **NormalPaintLayer**：

    1). 根元素(HTML).

    2). 具有明确的定位属性(relative, fixed, sticky, absolute).

    3). 有透明属性的(opacity 小于1).

    4). 有 CSS 过滤器的.

    5). 有 CSS mask 属性.

    6). 有 mix-blend-mode 属性(非 normal).

    7). 有 transform 属性(非 none).

    8). backface-visibility 属性为 hidden.

    9). 有 reflection 属性.

    10). 有 column-count 属性(非 auto)或 column-width 属性(非 auto).

    11). 动画的应用包括 opacity, transform, filter, backdrop-filter.

2. **OverflowClipPaintLayer**: overflow 不是 visible.

3. **NoPaintLayer**: 它是没有绘制的 PaintLayer，比如，没有 visual 属性(background, color, shadow)的空 div.

满足上述条件的 LayoutObject 有一个独立的 PaintLayer，而其他 LayoutObjects 与拥有该 PaintLayer 的第一个父元素共享 PaintLayer。