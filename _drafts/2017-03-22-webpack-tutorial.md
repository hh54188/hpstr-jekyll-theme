# Webpack 从入门到精通

## 前言

“从入门到精通”这个标题有没有很吸引你？哈哈哈。但这就是一篇这么有吸引力的文章！包你入门，包教包会，还能进阶，童叟无欺。

这篇文章本意是写给我自己看的。原因是虽然工作中会涉及到React开发，但并不是持续性的。可能两个功能的迭代相隔几周甚至一个月。期间则是使用其他的工具或者框架进行开发。而每次捡起来重新开发时或者立新项时，发现已经不太会写webpack配置了，又需要重新查询各种教程。后来反思其实是因为从来就没有真的学懂过webpack。这篇文章就是我在重新彻底学习完webpack之后的总结文章。也为了方便自己今后查询用。

## 什么是 Webpack

webpack 是一个打包工具，为什么需要打包？因为有的人的脚本开发语言可能是 CoffeeScript 或者是 TypeScript，样式开发工具可能是 Less 或者 Sass，这都需要工具把它们“编译”成浏览器能识别 Javascript 和 CSS。webpack就是干这个的。借用它们官网的一张图很好的诠释了以上的描述：

![what is webpack](./images/webpack-tutorial/what-is-webpack.png)

现在你可能会问为什么我要用它？Grunt和Gulp不是也能做相同的事情吗？我也是这么认为的。Grunt和Gulp定位为任务/流程工具（Grunt的副标题为The JavaScript Task Runner），除了打包工作外，它们还能执行图片压缩，文档生成（虽然这其中的很多webpack也已经能做了），代码检查等等，你可以自己自由选择要执行的任务然后把它们一环连一环的拼接在一起。理论上来说，webpack是Grunt的功能子集。

然后为什么我要用webpack？好吧，这个问题你也可以用在为什么已经有Grunt了还要造一个Gulp？以及为什么我要用Gulp替代Grunt，它们俩功能不也类似吗？客套点的答案是，存在的即是合理的，它们的出现必然有可取之处；残酷一点的答案是webpack是当下最流行最前沿的，是作为前端工程师先进性的表现，所以你必须要学。就和使用gmail比使用qq邮箱求职更让人看得起一样，其实没什么道理。什么？你说你不想学，不会也就不会了？这话别对我说，对你将来的面试官说

## Webpack 误区

我接触 Webpack 是从学习React开始，所以一直有个误区：Webpack，React，Babel是深度绑定的。其实不是。如果你不是在进行React开发，你仍然可以是用Webpack做CoffeScript或者是Sass的打包工作，自然也就不需要Babel。即使你在进行React开发，但是不使用jsx，你仍然可以选择不使用Babel。

Webpack是一个很强悍的工具，提供非常的多的参数供配置，你自己也可以编写webpack插件。本文只覆盖我目前接触到的、常用的或者是比较好用的一些参数，解释应该在什么情况下如何使用它们，相信已经可以覆盖大部分的开发情况了。

在自学Webpack的时候发现webpack存在碎片化的问题，就是在不同版本中编写参数的规则可能不同。本文都统一以 webpack 2 为标准

#

## 基础



- https://github.com/AriaFallah/WebpackTutorial
- https://scotch.io/tutorials/getting-started-with-webpack-module-bundling-magic
- https://scotch.io/tutorials/setup-a-react-environment-using-webpack-and-babel
- https://www.codementor.io/tamizhvendan/beginner-guide-setup-reactjs-environment-npm-babel-6-webpack-du107r9zr
- https://www.twilio.com/blog/2015/08/setting-up-react-for-es6-with-webpack-and-babel-2.html
- https://webpack.github.io/docs/tutorials/getting-started/

