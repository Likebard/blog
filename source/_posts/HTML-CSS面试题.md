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
BFC 是 Block Formatting Context 的简写，我们可以直接翻译成“块级格式化上下文”。BFC是一个独立的布局环境，其中的元素布局是不受外界的影响，并且在一个BFC中，块盒与行盒（行盒由一行中所有的内联元素所组成）都会垂直的沿着其父元素的边框排列。
一个BFC区域包含创建该上下文元素的所有子元素，但是不包括创建了新的BFC的子元素的内部元素。如下
``` html
<div class="box1" id="wrap1">
    <div class="box2"></div>
    <div class="box3"></div>
    <div class="box4"></div>
    <div class="box5" id="wrap2">
        <div class="box6"></div>
        <div class="box7"></div>
        <div class="box8"></div>
    </div>
</div>
```
wrap1会创建一个BFC区域，区域包含box1,box2,box3,box4,box5,也就是所有wrap1的子元素。同样wrap2也创建了一块BFC区域，包含了box6,box7,box8。
#### 如何形成BFC
并不是任意一个元素都可以被当做BFC，只有当这个元素满足以下任意一个条件的时候，这个元素才会被当做一个BFC。
1. body根元素
1. 设置浮动，不包括none
2. 设置定位，absoulte或者fixed
3. 行内块显示模式，inline-block
4. 设置overflow，即hidden，auto，scroll
5. 表格单元格，table-cell
6. 弹性布局，flex

#### 利用BFC解决问题
重点：__每一个BFC区域都是相互独立，互不影响的。__
1. 利用BFC避免margin重叠
考虑如下结构：
``` html
<body>
    <p>看看我的 margin是多少</p>
    <p>看看我的 margin是多少</p>
</body>
```
假如我们给两个p都设置上margin，那么属于同一个BFC的两个相邻的Box会发生margin重叠。所以我们可以设置，两个不同的BFC。也就是我们可以把第二个p用div包起来，然后激活它使其成为一个BFC。__上文说过，BFC，就是一个与世隔绝的独立区域，不会互相影响。__ 因此两个盒子不再margin重叠。
``` html
<p>看看我的 margin是多少</p>
<div>
    <p>看看我的 margin是多少</p>
</div>
```

2. 利用BFC解决包含塌陷
给子组件添加margin-top,会把父组件一起带跑
``` html
<body>
  <div class="parent">
    <div class="child"></div>
  </div>
</body>
<style>
.parent{
    width: 300px;
    height: 300px;
    background-color: red;
}
.child{
    background-color: pink;
    height: 100px;
    width: 100px;
    margin-top:50px;
}
</style>
```
上面的例子中，想要的效果应该是粉色盒子与红色盒子的顶部距离为50px。但是子元素影响父元素，导致父元素距离上方偏移了50px,这个时候就应该触发BFC，比如使用overflow:hidden将父元素变成一个独立区域。
3. 清除元素内部浮动
比如给子元素设置浮动后，父元素塌陷消失了，也可以给父元素加上overflow:hidden.
4. 自适应布局
利用BFC的区域不会与float box重叠这一特性，我们可以实现多栏自适应的效果：这里演示一下自适应两栏布局，左边定宽加上浮动，右边设置overflow:hidden激活成为一个BFC：
``` html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
</head>
<style>
    *{
        margin: 0;
        padding: 0;
    }
    body {
        width: 100%;
        position: relative;
    }
 
    .left {
        width: 100px;
        height: 150px;
        float: left;
        background: rgb(139, 214, 78);
        text-align: center;
        line-height: 150px;
        font-size: 20px;
    }
 
    .right {
        /* BFC */
        overflow: hidden; 
        height: 300px;
        background: rgb(170, 54, 236);
        text-align: center;
        line-height: 300px;
        font-size: 40px;
    }
</style>
<body>
    <div class="left">LEFT</div>
    <div class="right">RIGHT</div>
</body>
</html>
```

### 多种方式实现水平垂直居中
1. 仅适用于居中元素定宽高：
- absolute + 负 margin
``` css
.wp {
    position: relative;
}
.box {
    position: absolute;;
    top: 50%;
    left: 50%;
    margin-left: -50px;
    margin-top: -50px;
}
```
- absolute + margin auto
``` css
.wp {
    position: relative;
}
.box {
    position: absolute;;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    margin: auto;
}
```
- absolute + calc
绝对定位+设置top,left
``` css
.root {
    position: relative;
}
.textBox {
    position: absolute;;
    top: calc(50% - 50px);
    left: calc(50% - 50px);
}
```
2. 居中元素不定宽高
- absolute + transform:
transform 的 translate 属性也可以设置百分比，这个百分比是相对于自身的宽和高，因此可以将 translate 设置为 ﹣50%
``` css
.wp {
    position: relative;
}
.box {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
}
``` css
- lineheight
把 box 设置为行内元素，通过 text-align 也可以做到水平居中，同时通过 vertical-align 做到垂直方向上的居中，代码如下：
``` css
.wp {
    line-height: 300px;
    text-align: center;
    font-size: 0px;
}
.box {
    font-size: 16px;
    display: inline-block;
    vertical-align: middle;
    line-height: initial;
    text-align: left; /* 修正文字 */
}
```
- css-table:
```
.wp {
    display: table-cell;
    text-align: center;
    vertical-align: middle;
}
.box {
    display: inline-block;
}
```

- flex:
``` css
.wp {
    display: flex;
    justify-content: center;
    align-items: center;
}
```
- grid: 
``` css
.wp {
    display: grid;
}
.box {
    align-self: center;
    justify-self: center;
}
```
3. 总结
- PC 端有兼容性要求，宽高固定，推荐 absolute + 负 margin
- PC 端有兼容要求，宽高不固定，推荐 css-table
- PC 端无兼容性要求，推荐 flex
- 移动端推荐使用 flex

### 清除浮动的三种方法
1. 额外标签法（在最后一个浮动标签后，新加一个标签，给其设置clear：both；）（缺点：添加无意义标签，语义化差,不推荐）

2. 父级添加overflow属性（父元素添加overflow:hidden(内容增多的时候容易造成不会自动换行导致内容被隐藏掉，无法显示要溢出的元素,不推荐使用)

3. 使用after伪元素清除浮动（推荐使用）
``` css
.outer {zoom:1;}    /*==for IE6/7 Maxthon2==*/
.outer :after {clear:both;content:'.';display:block;width: 0;height: 0;visibility:hidden;}   /*==for FF/chrome/opera/IE8==*/
```
其中clear:both;指清除所有浮动；content: '.'; display:block;对于FF/chrome/opera/IE8不能缺少，其中content（）可以取值也可以为空。visibility:hidden;的作用是允许浏览器渲染它，但是不显示出来，这样才能实现清楚浮动。

### link 和 @import 的区别
1. link属于XHTML标签，而@import完全是css提供的一种方式。
2. 加载顺序的差别：当一个页面被夹在的时候（就是被浏览者浏览的时候），link引用的CSS会同时被加载，而@import引用的CSS会等到页面全部被下载完再加载。
3. 兼容性的差别。由于@import是CSS2.1提出的所以老的浏览器不支持，@import只有在IE5以上的才能识别，而link标签无此问题，完全兼容

### CSS 如何定义权重规则
!important>行间样式>id>class|属性>标签选择器>通配符
![css权重](HTML-CSS面试题/css-weight.png)
1. 注意这里不是10进制，是256进制
2. 假如两份样式权重一致，后面的会优先生效

### CSS Modules
__什么是CSS Modules: 项目中所有 class 名称默认都是局部起作用的。__
CSS Modules 并不是一个官方规范，更不是浏览器的机制。它依赖我们的项目构建过程，因此实现往往需要借助 Webpack。借助 Webpack 或者其他构建工具的帮助，可以将 class 的名字加上前缀唯一化，从而实现局部作用。
例如常用的css-loader就支持CSS Modules
``` css
{
    test: /\.css$/,
    loader: "style-loader!css-loader?modules"
}
```