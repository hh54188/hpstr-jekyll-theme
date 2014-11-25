# 移动开发的那些事（下）：响应式图片设计

如果你还没有了解过有关于响应式设计的一些概念，可以再开始阅读这篇文章之前阅读这一系列的上篇开始：
[浅谈移动Web开发（上）：深入概念](http://www.infoq.com/cn/articles/development-of-the-mobile-web-deep-concept)。上篇介绍了一些你必须掌握的移动端开发中要用到的概念，并且同其他概念做了一些区分。可以说上一篇是为这一篇服务的。在这一篇中，我们就需要运用到上一篇中了解的那些概念。

你可能会有疑问，为什么下篇说的是响应式图片的设计，为什么不是响应式设计。一方面因为响应式设计已经被说的太多了，有太多的文章在谈；另一方面，如果你对响应式这一块有关注的话，响应式图片与响应式设计并非包含关系，从某种意义上来说，响应式图片已经可以成为与响应式设计平行的一个技术分支。所以我们可以拿出来着重的说。

在构思这篇文章的时候，我的想法是按部就班的把响应式图片的技术一一做一个介绍。介绍技术用法非常简单，贴上语法，附上例子，相信大部分人就能看懂就能照葫芦画瓢了。但我一直觉得了解这门技术诞生的原因，了解原技术的不足，了解这个技术的缺陷同样也非常重要。更有助于我们理解这项技术。

但是在学习中逐渐发现，响应式图片技术产生的原因，或者它所面临的困境更加的有趣或者吊诡。或者说，如果我能把所有的这些前因后果做一个介绍，或许能帮助大家更好的理解掌握技术，更好的集思广义。

当然，正如文章的标题所说，文字的容量总是有限的，所以做到的只能是浅谈。所以这篇文章里更多描述的是想法：问题是如何产生的，现有解决方案的不足，新的解决方案的思路，新的解决办法的不足。

## User Cases

首先我们要了解清楚在什么情况下需要用到响应式图片。W3C的RICG(Responsive Images Community Group)小组给出了如下的[四种用户用例](http://usecases.responsiveimages.org#h2_use-cases)，

### 基于分辨率选择：

![resolution_selection](./images/resolution_selection.jpg)

这个是最基本的需求，也就根据网页所处设备分辨率的不同选择图片。你可能觉得说分辨率变了，图片使用同一张用css控制宽高等比压缩就好了。其实不仅仅是根据分辨率不同来选择图片，比如说较小分辨率可能就意味着加载网页的这台设备性能有限，很大概率下是手机的话就意味着带宽有限。如果能根据分辨率加载图片，我们就能提供给适合该设备的图片。节省带宽与时间。

### 基于DevicePixelRatio

![dpr_selection](./images/dpr_selection.jpg)

在上一篇中我们从PPI引出了DevicePixelRatio（以下简称DPR）。 我们知道高像素密度下设备的物理像素很小，而大部分的优化方案会造成图片素材的模糊。因此我们需要为高DPR的设备特意准备更高清的素材。

### 基于viewport

![viewport_selection](./images/viewport_selection.jpg)

或许你的第一反应是基于viewport貌似和基于分辨率非常的像？某种意义上是的。但就像在上一篇所说的那样，移动设备的上的viewport通常与分辨率并非一一对应的关系。比如在iPhone4上，默认的viewport是980px，但是你也可以指定为与visual viewport大小相等。可以这么理解，基于viewport比基于屏幕分辨率更加精确。

### Art direction

我不知道应该如何翻译这个词组比较好，我们姑且采用百度百科里的翻译，译为“美术设计”。

它描述的是这样一种情况，假设我们基于分辨率设计响应式图片。那么在高分辨率和低分辨率下图片素材对比如图：

高分辨率：

![obama-500](./images/obama-500.jpg)

低分辨率：

![obama-100](./images/obama-100.jpg)

我们的任务就完成了，虽然做到了响应式，但是第二张图中的任务几乎是不可见的，我们压根无法分辨出其中的奥巴马。
所以更适合的图片应该是这样：

![obama-100-art](./images/obama-100-art.jpg)

虽然图片大小仍然和低分辨下保持一致，但我们对图片进行了剪裁，略去了可以被忽视的背景图片。

在这之前的几种场景只需要用户上传一张尺寸足够大的图片，然后仅仅缩放图片去适应就好了。而这一种情况下，图片不仅仅是被缩放，还需要被重新设计和剪裁。所以在后面的部分你会看到它使用的技术是和前三者不一样的。

![art_direction](./images/art_direction.jpg)

## `srcset`

【介绍srcset语法的由来】

响应式图片的解决方案简单的可以划分为两类：1.用一张图片解决；2.用多张图片解决

在这里我们先看更实用更复杂更多争议的后者。我们需要用到`picture`元素和`srcset`属性。

为了展示srcset的用法，我们考虑一种最简单的情况。 我们要同一张图为iPhone 3gs与iPhone 4做适配。我们只需要为他们不同对策DPR做适配。
在这种情况下，我们不用考虑屏幕的分辨率，不需要考虑viewport大小。只是两台尺寸同样大小的DPR不同罢了。那么我们需要用一张large.jpg的图片应付DPR为2.0的情况，用一张small.jpg的图片应付DPR为1.0的情况。 使用srcset的语法应该如下：

```
<img src="small.jpg" srcset="large.jpg 2x, small.jpg 1x">
```
非常简单，一目了然。 只需要把图片和对应的设备参数匹配起来（比如上面用x为单位描述屏幕的DPI，也可以用w或者h描述viewport的宽度），再串成字符串用逗号间隔开。
那么一旦浏览器发现了匹配的DPR参数，就加载响应的图片。`img`自带的src标签用于处理默认情况

OK，那让我们把情况变得复杂一些。这次我们响应的不再是屏幕的DPR，而是viewport的宽度。并且我们需要响应三种宽度，小于600px，600px至800px之间，大于800px。我们也用三张图片对应这三种宽度：small.jpg, medium.jpg, large.jpg。参照上面的DPR写法，你可能会想当然的参照上面DPR的写法：

```
<img src="large.jpg" srcset="medium.jpg 800w, small.jpg 600w">
```
ちょっと待（ま）って（等等），你真的确定吗？

宽度与DPR不同的地方在于，如果你熟悉media query的话，DPR（x单位）描述的是精确值；而宽度(width)(w单位)描述的是边界值，但问题是最大边界(max-width)还是最小边界(min-width)呢。为了描述的更加清楚，我们可以以media query中的情况为例：

A写法：

```
body {}

@media screen and (min-width:480px) {}

@media screen and (min-width:800px) {}
```

B写法

```
body {}

@media screen and (max-width:800px) {}

@media screen and (max-width:480px) {}
```
其实两种写法最终达成的目的并无不同，都将布局划分为三段式响应。

A写法中总是取min-width为分界点，并且分界点逐渐递增。越往后所对应的样式为更宽的样式。这样的划分方法我们称之为移动布局优先（mobile first）
B写法中总是取max-width为分界点，并且分界点逐渐递减。越往后所对应的样式为更窄的样式，这样的划分方法我们称之为桌面优先(desktop first)

回到最上面的srcset关于宽度的写法，我们无法用语法准确的表达我们到底是希望mobile first或者desktop first。或者我们也无从知晓浏览器以为的是采用哪一种趋势。 你可能会建议我现在不如写一段代码来测试一下，但即使通过测试结果浏览器能够明确的表示采用何种趋势，但是对于另一种趋势来说是公平的吗？

再退一步说，即使我们默认采用其中的某一种趋势，比如mobile first好了。并且为了兼容高DPR的情况。 我们的最终语法可能会是这个样子的：

http://jsfiddle.net/rgefayoL/

```
【待续整理】
```

如何解决这个问题。我们来审视一下这个解决方案的症结所在。

浏览器在加载的时候只知道关于“自己所处的环境”：屏幕的分辨率，DPR，viewport大小。mediaquery处理的机制就是在特定的环境下让浏览器做特定的事情。就类似于分支非常多的条件语句，比如：“当viewport宽度大于xxx时，显示侧边栏；否则隐藏侧边栏”。“当用户的屏幕是Retina时，选择这张高清图片；使用一张普通的图片”。

这看似没有什么问题，但是一旦我们的分支情况比如多了之后，比如需要同时考虑像素密度，viewport宽度，假设分别都只有两种情况，那么我们也要写2x2=4条语句。如果这么说还不够感性的话，我们可以看一看下面这个例子：




【
	怎么来说接下去的问题？
	从DPR引出srcset？
		引出srcset语法？

	引出viewport情况？
	虽然和DPR情况一致，但是在这里存在min-width或者max-width的区别
	引出srcset在w下的矛盾 **<----思想**

	如何决定images breakpoints***<----思想*

	姑且以mobile-first写出查询代码
	发现非常的繁琐
	于是引入sizes

	解决了刚刚的那个问题，所有都交给了智能的浏览器来解决
	但是要注意算法的问题，很多情况并非如我们所期望的 **<---思想**

	direction的解决方案有所不同，采用的是picture元素和source元素。虽然上面的sizes也是picture的语法

	为什么srcset和<picture>没法互相替换？http://html5hub.com/the-src-n-responsive-image-solution/
	srcset的局限在哪里？

	1. breakpoints变的不是那么清晰可见
	2. 有大量的计算算出
	3. 图片在不同viewport宽度下占用的宽度也不是清晰可见了
	4. 缺少远见，只考虑到了1x, 2x的情况
	5. 缺少可维护性，每次修改可能引发修改其他的片段


	srcset与preloader的矛盾 <---思想：		
	picture的fallback问题 <---思想
	
		http://www.smashingmagazine.com/2013/05/10/how-to-avoid-duplicate-downloads-in-responsive-images/
		http://blog.cloudfour.com/the-real-conflict-behind-picture-and-srcset/


	！！！以上引入的的思想不多啊
】


