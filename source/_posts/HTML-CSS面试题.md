---
title: HTML&&CSS面试题
date: 2021-08-13 11:22:21
categories: 面试
tags: 八股
---
### 语义化是什么、为什么、怎么做：
- HTML 语义化就是：根据结构化的内容，选择合适的标签。
- 什么要做到语义化：
    - 有利于 SEO
    - 开发维护体验更好
    - 用户体验更好（如使用 alt 标签用于解释图片信息）
    - 更好的 accessibility，方便任何设备解析（如盲人阅读器）
- 如何做到语义化：实时跟进、学习并使用语义化标签。
    ``` javascript
    if (导航) {
    return <nav />
    }
    else if (文稿内容、博客内容、评论内容...包含标题元素的内容) {
    return <article />
    }
    else if (目录抽象、边栏、广告、批注) {
    return <aside />
    }
    else if (含有附录、图片、代码、图形) {
    return <figure />
    }
    else if (含有多个标题或内容的区块) {
    return <section />
    }
    else if (含有段落、语法意义) {
    return <p /> || <address /> || <blockquote /> || <pre /> || ...
    }
    else {
    return <div />
    }
    ```
### BFC：
#### Box: css布局的基本单位
Box 是 CSS 布局的对象和基本单位， 直观点来说，就是一个页面是由很多个 Box 组成的。元素的类型和 display 属性，决定了这个 Box 的类型。 不同类型的 Box， 会参与不同的 Formatting Context（一个决定如何渲染文档的容器），因此Box内的元素会以不同的方式渲染。让我们看看有哪些盒子：
-   block-level box:display 属性为 block, list-item, table 的元素，会生成 block-level box。并且参与 block fomatting context
- inline-level box:display 属性为 inline, inline-block, inline-table 的元素，会生成 inline-level box。并且参与 inline formatting context；
- CSS3 规范中又加入了 run in box, 以后再研究。

#### Formatting Context
Formatting context 是 W3C CSS2.1 规范中的一个概念。它是页面中的一块渲染区域，并且有一套渲染规则，它决定了其子元素将如何定位，以及和其他元素的关系和相互作用。最常见的 Formatting context 有 Block fomatting context (简称BFC)和 Inline formatting context (简称IFC)。

#### BFC 是什么
    - BFC 是 Block Formatting Context 的简写，我们可以直接翻译成“块级格式化上下文”。BFC是一个独立的布局环境，其中的元素布局是不受外界的影响，并且在一个BFC中，块盒与行盒（行盒由一行中所有的内联元素所组成）都会垂直的沿着其父元素的边框排列。

#### 如何形成BFC
1. float的值不是none。
2. position的值不是static或者relative。
3. display的值是inline-block、table-cell、flex、table-caption或者inline-flex
4. overflow的值不是visible

#### BFC的作用
1. 利用BFC避免margin重叠
考虑如下结构：
```
<body>
    <p>看看我的 margin是多少</p>
    <p>看看我的 margin是多少</p>
</body>
```
假如我们给两个p都设置上margin，那么属于同一个BFC的两个相邻的Box会发生margin重叠。所以我们可以设置，两个不同的BFC。也就是我们可以把第二个p用div包起来，然后激活它使其成为一个BFC
```
<p>看看我的 margin是多少</p>
<div>
    <p>看看我的 margin是多少</p>
</div>
```