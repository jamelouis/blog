---
layout: post
title: 笔记：编写高质量代码 - Web前端开发修炼之道
date: 2018-10-05 12:00:00 +0800
categories: web
---

## 从网站重构说起

* Web标准 -- 结构、样式和行为的分离
* Web标准：结构标准、样式标准和行为标准
* 结构标准：XML标准，XHTML标准，HTML标准
* 样式标准：CSS标准
* 行为标准：DOM标准，ECMAScript标准
* 高质量的代码：在Web标准的思想指导下，在实现结构、样式和行为分离的基础上，还要做到三点：精简、重用、有序。

## 团队合作

* 团队合作的最大难度不是技术，是人
* 制定规范、公共组件的设计和复杂功能的技术方案

## 高质量的HTML

* 搜索引擎看不到视觉效果，看到的只是代码，只能通过标签来判断内容的语义
* 正确的做法是，先确定HTML，确定语义的标签，再来选用合适的CSS
* 判断标签语义是否良好的方法：去掉样式，看网页结构是否组织良好有序，是否仍然有很好的可读性。
* 尽可能少地使用无语义标签div和span

### [Semantic HTML][1]

* Document Outline: Every HTML document has an "outline", which is how search engines and screen readers view the hierarchy of the content on the page.
* The &lt;h1&gt; through &lt;h6&gt; heading elements all contribute to a page's document outline.
* [HTML5 OUtliner][2] : inspecting the document outline of a page
* The &lt;article&gt; element represents an independent article in a web page.  
* The &lt;section&gt; element is sort of like an &lt;article&gt;, except it doesn't need to make sense outside the context of the document.
* The &lt;nav&gt; element lets you mark up the various navigation sections of your website.

## 高质量的CSS

* 怪异模式模拟老式浏览器的行为以防止老站点无法工作
* DTD(Document Type Definition): 文档类型定义
* 通过设定特定的DTD来触发浏览器的怪异模式
* CSS组织
    * 控制字体的CSS集中在font.css
    * 控制颜色的cSS集中在color.css
    * 控制布局的CSS集中在layout.css
* 作者推荐：base.css + common.css + page.css
    * base： 提供CSS reset功能和粒度最小的通用类 -- 原子类
    * common：提供组件级类
    * page：提供页面级的样式
* 模块与模块之间尽量不要包含相同的部分，如果有相同部分，应将它们提取出来，拆分成一个独立的模块
* 模块应在保证数量尽可能少的原则下，做到尽可能简单，以提高重用性
* 骆驼命名法用于区别不同单词，划线用于表明从属关系
* 低权重原则 -- 避免滥用子选择器
* 当不同选择符的样式设置有冲突时，会采用权重高的选择符设置样式
* 如果CSS选择符权重相同，那么样式会遵循就近原则，哪个选择符最后定义，就采用哪个选择符的样式

###  [The Elements of Typographic Style Applied to the Web][3]



[1]:https://internetingishard.com/html-and-css/semantic-html/
[2]:https://gsnedders.html5.org/outliner/
[3]:http://webtypography.net/



