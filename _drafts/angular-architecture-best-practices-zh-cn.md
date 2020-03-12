# 【Angular 架构鉴赏 01】【译文】：Angular 架构模式与最佳实践



## 前言

前言是作为译者我想说的话，并非原文中的内容。

我猜此时此刻你心里的第一个疑问一定是：为什么是 Angular，不是 React，不是 Vue，不是 Flux，不是 Redux ? 因为你已经对它们太熟悉了

**我个人作为开发者最希望是能够汲取到“圈外”的“营养”，这样才能给我的成长带来帮助**，我想对各位也是一样。你不用担心因为不会 Angular 而看不懂这一些列文章，它们基本上谈论的是应用架构——关于设计、组织、抽象，很少会落到具体的实现，即使有，连蒙带猜也能推测出一二。这也能从侧面说明我衷心想推荐这些佳作的原因：通过大段的代码阐述很容易，多少都不嫌多；难的是用很少的代码甚至几乎不用代码来跨编程语言的说明高层次的东西，比如 Martin Flowler, Uncle Bob Martin

我不评价框架的流行，好坏和孰是孰非，我只是把一切的一切都呈现在各位的眼前。它们并非和 Flux，Vuex 大相径庭，反而你们会看到它们的影子，但更多的是不一样的东西。

在文中我会以“引用”的格式和“译者注”开头穿插一些我的个人备注和带给我启发性的问题，你可以理解为文章的“评论音轨”。这些问题我会尝试在这个系列的最终的文章中进行回答。

我有一些问题留给你们，也是我曾经自己的问题，如果带着这些问题去阅读和思考，我想会更有帮助：

* Angular 架构某些方面会比 Redux 更好吗？或者反之？如果好，好在哪？如果不好，欠缺在哪？
* 无论哪一种架构（Angular / Flux / Redux / Vuex）一定是最佳解吗？如果不是，哪些常见会不适合它们？如果给你一个机会去改进它们？你会怎么做？
* 想象一个当前一个工作中你正在解决的问题，抛开这些架构，假设需要你来设计一个架构，你是否会比已知的这些架构做的更好？或者只针对当前这个 case 做的更好？
* 当你阅读到在实现同一功能上 Angular 架构中的做法和 React 中不同时，你觉得哪种更好？互换一下呢？

下面正文开始

## 正文

本文原文：[Angular Architecture Patterns and Best Practices (that help to scale)](https://angular-academy.com/angular-architecture-best-practices/)

搭建可拓展性（scalable）的软件是一项具有挑战性的任务。当我们在思考前端应用的可拓展性时，我们会想到递增的复杂度，越来越多的业务规则，应用需要加载越来越多的数据以及遍布全球的分布式团队。为了应对上述的各种因素从而保证高质量的交付和避免技术债的产生，健壮及牢固的架构必不可少。虽然 Angular 自身是具有强烈技术主见的框架，迫使开发者以*恰当*的方式进行开发，但是许多地方依然容易犯错。在这篇文章里，我会展示极力推荐的基于最佳实践和经过实战检验的具有良好设计的 Angular 应用架构。我们在本文的终极目标是学习如何设计 Angular 应用为了维持长时期内的**可持续的开发效率**和**增加新功能的简易程度**。为了达成这些目标我们会使用到

* 应用层次之间恰当的抽象
* 单向数据流
* 反应式（reactive）状态管理
* 模块设计
* 容器（smart）组件模式和纯展现（dumb）组件模式

> 译者注:
>
> 我听过一种说法说 React 是类库，Angular 是框架，而类库和框架区别在于类库是被你所写的代码调用的，而框架是调用你所写的代码。然而这个对 React 真的成立吗？难道 React 配合 Flux 和 Redux 才能组成某种框架吗？以及为什么我们需要 Redux ?
>
> smart component 和 dumb component 是不是很眼熟?

![](./images/angular-architecture-best-practices/bullets.png)

## 目录

* 前端的可拓展性问题
* 软件架构
* 高层次抽象层
  * 展现层
  * 抽象层
  * 核心层
* 单向数据流和反应式状态管理
* 模块设计
  * 模块目录结构
* 容器组件模式和纯展现组件模式
* 总结

## 前端的可扩展性问题

让我们回想一下在开发现代前端应用中面临扩展性问题。当下前端应用不再“仅仅展现”数据和接受用户的输入。单页面应用（Single Page Applications）为用户提供了丰富交互并使用后端作为数据持久层。这也意味着更多的职责被转移到了软件系统的前端。这导致了我们需要处理的前端逻辑变得越发复杂。不止需求一直在增长，连应用需要加载的数据量也在增加。基于这些我们需要照顾到极易受损的应用性能。最后我们的开发团队也在增长（或者至少在轮替，有人来有人走） ，让新加入的开发者尽可能快的融入非常重要

![](./images/angular-architecture-best-practices/meme.jpg)

解决上述问题的方案之一就是坚固的系统架构。但是这需要代价，代价是从第一天起就要拥抱架构。当系统非常小时，足够快的交付功能这对我们开发者来说非常有吸引力。在这个阶段，一起都容易理解，所以开发速度非常快。但是除非我们关心架构，否则经过几轮程序员的轮替，开发奇怪的功能，重构，引入一些新模块之后，开发速度会断崖式下跌。下面的图表展示了在我的开发生涯中通常的情况。这不是任何的科学研究，只代表我时这么看待它的

![](./images/angular-architecture-best-practices/chart.png)

## 软件架构

为了讨论架构的最佳实践和模式，我们首先需要回答一个问题，软件架构是什么。[Martin Fowler](https://martinfowler.com/) 把架构定义为：*“最高层次的将系统拆解为不同的部分（*highest-level breakdown of a system into its parts*）”* 。基于此我会将软件结构描述为关于软件如何将它部分组合在一起并且部分间通信的*规则*和*约束*。通常来说，我们在系统开发中的架构决策在系统演进的过程中很难发生更改。这也是为什么非常重要的是从项目一开始就要对这些决策多加留意，尤其当是我们搭建的软件需要在生产环境中运行多年的话。[Robert C. Martin](https://en.wikipedia.org/wiki/Robert_C._Martin) 曾经说过：软件真正的开销是维护。拥有牢固地基的架构帮助减少系统维护的成本

> **软件架构**是关于如何组织它部分的方式以及部分之间通信的*规则*和*约束*

> 译者注：在上面的翻译中我将 parts 翻译成了“部分”。你或许会认为或者翻译为“组件”听上去更合适，但“组件（components）”在不同的编程语言中有更特定的意思

## 高层次抽象层