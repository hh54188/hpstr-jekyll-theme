---
layout: post
title: 待定
modified: 2014-10-14
tags: [css, mobile, javascript]
image:
  feature: abstract-3.jpg
  credit: dargadgetz
  creditlink: http://www.dargadgetz.com/ios-7-abstract-wallpaper-pack-for-iphone-5-and-ipod-touch-retina/
comments: true
share: true
---

## CSS像素是相对的

假设我们有一台非常普通的显示器，最佳分辨率为1280x800。此时我们用浏览器打开一个测试页面，浏览器最大化，页面上有一个320x200(单位：px)的红色容器，那么我们只要复制这个容器四份，从左至右依次排列，便能在宽度上占满整个浏览器，也就占据了整个屏幕。

假设情况变的复杂一点，我们将浏览器页面放大至200%（“Ctrl键”加上“+号键”）。此时会发生什么？本来只占页面四分之一宽度的容器已经占据了屏幕的一半。需要注意的是，我们既没有增加容器的宽度，也没有改变屏幕的分辨率。放大页面实际上操作的是页面的像素，**也就是我们把像素拉伸到原来的两倍了**。 请注意这里的两倍是相对谁而言的，是相对于100%状态页面状态而言，也就是相对于屏幕像素而言。

CSS像素与屏幕像素1：1同样大小时：
![origin_pixel](./images/ppi/csspixels_100.gif)
CSS像素开始被拉伸，此时1个CSS像素大于1个屏幕像素
![zoom_in_pixel](./images/ppi/csspixels_in.gif)

举这个例子我想说明一件事：CSS像素从来只是一个**相对值**。 W3C从来就没有规定过一个单位的CSS像素值需要和一个屏幕像素值相等，虽然到今天为止大部分PC仍然是这么做的。

在这稍等一会，在这里我们要探讨一下上面提到的屏幕像素值，如果这里的所说的屏幕像素值仍然是一个相对值呢？

大部分显示器都是基于点阵的，也就是说通过一系列的小点排成一个大矩形，每个小点显示不同的颜色来形成图像。在上面的例子中，显示器分辨率宽度为1280px，也就意味着显示器上有1280个物理像素点，为了区分，我们在这里特意加上了*物理*这两个字。一般情况下，分辨率的像素点和物理像素点是一一对应的。

那么问题来了，如果分辨率像素和物理像素不再是一一对应的关系——也就是说，CSS像素相对于分辨率像素，而分辨率像素又相对于屏幕物理像素时，CSS像素应该如何计算？

为了说明这个问题，此时我们又必须要引入另外两个概念：PPI和DPI。


## PPI VS DPI

这两个概念很相似也非常让人困惑，在不同的行业和上下文中，两者的概念可能相似也可能不同。

PPI全称为Pixel Per Inch，译为每英寸像素取值，更确切的说法应该是像素密度，也就是衡量单位物理面积内的像素密度情况的值。对于显示器而言，像素密度与显示器尺寸与屏幕的当前分辨率有关，同一尺寸的显示器在800x600分辨与1024x768分辨率下的像素密度明显是不同的，明显后者单位面积内的像素更多，当然后者的像素密度更高。 所以针对可以调整分辨率的液晶显示器而言谈PPI是有些没有意义的。 注意这里我们仍然还没有牵扯进设备的物理像素。 原理如下图所示：

![original](./images/ppi/original.png)

我们似乎更关心移动设备的PPI情况。因为更高的PPI意味着在同一物理屏幕上能够展现更多的画面细节，也就意味着更平滑的画面。

![original](./images/ppi/original.jpg)

但同时也需要注意到，相同的图片素材，在越高的设备上会显示的越小，以下是一个像素(1 pixel)在不同PPI设备上的可见情况：

![pixel-density-1](./images/ppi/pixel-density-1.png)

也就是说如果你想让你的图片在同样尺寸高PPI和低PPI的设备上展现同样物理尺寸的话，在高PPI设备上你的图片必须具有更高的像素和更高清的细节
。因为像素Pixel的本意就是Picture Element，是图片的信息单位，像素的缩小也就意味着图片的缩小，这非常影响用户体验。如果你使用过Windows操作系统的2k笔记本电脑，就会遇到这样的问题，假设PPI提高了一倍，那么就意味着程序界面缩小了4倍（长乘以宽），也有可能为了和原体验保持一致图片素材被拉伸到原来的四倍，如果软件使用的是点阵字体（.otf字体，你可以理解为使用的是位图），字体也会变得模糊。

以三星Galaxy s4为例，当它告诉我们它的分辨率有1920x1080的时候，说明它的设备上真的只有1920x1080个物理像素点。但是如果你使用的是Macbook Retina，你会发现它的分辨率只有1440x800，但实际上它的显示屏上有2880x1600个物理像素点。 从上面的例子可以看出







但我们不得不面临设备差异化的问题，面对不同设备的物理像素和像素密度，当CSS像素不再与物理像素一比一相匹配是，那么应该如何对CSS像素取值？

W3C因此定义了一个*相对像素(reference pixel)*的[概念](http://www.w3.org/TR/CSS21/syndata.html#length-units)：

>The reference pixel is the visual angle of one pixel on a device with a pixel density of 96dpi and a distance from the reader of an arm's length. For a nominal arm's length of 28 inches, the visual angle is therefore about 0.0213 degrees. For reading at arm's length, 1px thus corresponds to about 0.26 mm (1/96 inch).

>把人以手臂长的距离看到的像素密度为96dpi的设备一个像素点定义为*相对像素*，也就是说，1px的像素对应的物理实际单位是0.26mm(1/96英寸)。同时假设人的手臂长度为28英寸，通过正切函数，我们能算出人看到一像素的夹角（下图中的α）为0.0213度

![pxangles](./images/ppi/pxangles.png)

通过参考这个相对像素，再结合设备的物理像素和手持距离，可以计算出该设备的理想像素，甚至可以推算出设备的设备像素比(devicePixelRatio)。

比如当我们手持一台像素密度为180dpi的手机时，我们假设因为屏幕较小的缘故，人会把手机拉近操作，因此我们可以假设此时人眼离手机的距离为18英寸，对比相对像素，利用三角函数公式，我们可以计算出理想的像素密度为150dpi ((28/18) * 96 = 150)

![calculate-dpr](./images/ppi/calculate-dpr.png)

因此设备的像素比为（实际像素/理想像素） devicePixelRatio = 180 / 150 = 1.2。

使用上面这样的方式计算像素比或许比较陌生，让我们看看传统的方法

>window.devicePixelRatio = physical pixels / dips





