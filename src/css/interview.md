# 常问面试题


## 一个不定宽高的DIV,垂直水平居中？

公司：阿里、滴滴、易车、新东方、虎扑、饿了么、爱范儿、心娱、58

1. flex
```css
.parent {
  display: flex;
  justify-content: center;
  align-items: center;
}
```

2. 绝对定位
```css
.parent {
  display: relative;
}
div {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}
```

3. 利用定位 + margin: auto
```css
.parent {
    position: relative;
}

div {
    position: absolute;
    top: 0;
    bottom: 0;
    left: 0;
    right: 0;
    margin: auto;
}
    
```

4. 利用table  
利用vertical和text-align可以让所有的行内块级元素水平垂直居中
```css
.parent {
  display: table-cell;
  text-align: center;
  vertical-align: middle;
}
div {
  display: inline-block;
  vertical-align: middle;
}
```

5. grid网格布局

```css
.parent {
    display: grid;
    align-items: center;
    justify-content: center;
}
div {
    width: 10px;
    height: 10px;
}
```


## display: none;与visibity:hidden；的区别？
1. display:none会让元素完全从渲染树中消失，渲染时候不占据任何空间。visibility: hidden不会让渲染树消失，占据空间，内容不可见。

2. display:none;非继承属性，修改子孙节点属性无法显示。visibility:hidden继承属性，通过设置可让子孙节点显示。

3. 修改常规流中元素display造成重排，修改visibility造成重绘；

4. 读屏器不读display:none内容，会读visibility内容。




## 伪元素与伪类的区别？
**伪元素**是对元素中特定内容进行操作。选取某些内容前面or后面这种普通选择器无法完成的工作。本身只是基于元素的抽象，并不存在于文档中。
例如：
* ::before(元素内容之前插入额外生成的内容)
* ::after
* ::first-letter(首个字符)
* ::first-line(元素的第一行)
* ::selection
* ::grammar-error

**伪类**：一个元素特定状态改变时可能得到or失去某个样式功能，和class有些类似。
* :target
* :link
* :hover
* :active
* :visited
* :focus
* :not
* :lang
* :enabled
* :checked
* :empty






##  BFC 是什么？触发 BFC 的条件是什么？有哪些应用场景？
Block Formatting Context(块级格式化上下文)，是页面上独立的一块渲染区域。
### BFC特性？
1. 内部box会在垂直方向，一个接一个的放置；
2. 每个元素的margin box的左边，与包含块border box的左边相接触；
3. box垂直方向的距离由margin决定，属于同一个bfc的2个相邻box的margin会重叠；
4. bfc区域不会与浮动区域的box重叠；
5. bfc是页面上独立的容器，外面元素不会影响bfc里的元素；
6. 计算bfc的高度时，浮动元素也参与计算；

### 触发BFC条件有：
* float值不为none
* overflow的值不为visible
* display的值为table-cell、table-caption、inline-block、flex、inline-flex
* position不为relative、static
* 根元素

BFC 主要的作用是：

清除浮动
防止同一 BFC 容器中的相邻元素间的外边距重叠问题




##  Css 选择器都有什么，权重是怎么计算的?
index.md有答案。





## 介绍下 Flex 布局，属性都有哪些，都是干啥的？

公司：阿里、虎扑、快手








## 清除图片底部间隙？
1. 图片块状化，无基线对齐
```css
img {
  display: block;
}
```
2. 图片底线对齐
```css
img {
  vertival-align: botom;
}
```
3. 行高足够小





## 清除浮动方法？
1. 父级div设高, 缺点是并不是所有元素的高度都是固定；
2. 使用空元素
在元素最后下加一个元素设置clear：both属性，缺点是会增加多余无用的html元素
```html
<div class="container"> 
  <div class="left">left浮动</div> 
  <div class="right">right浮动</div>
  <div style="clear:both;"></div>
</div>
```

3. 父元素div一起浮动
4. 父级div定义display: table;
5. 父元素触发了BFC
```css
.fu {
  overflow: hidden;
}
```
6. 父级div定义伪元素after
```css
.clearfix: after {
  content: '.';
  display: block;
  height: 0;
  clear: both;
  visibility: hidden;
}
```



## 说说你了解的圣杯布局和双飞翼布局？

```js
______________________
|  header             |
|—————————————————————
|left|   main   |right|
|    |          |     |
|    |          |     |
|____|__________|_____|
|       footer        |
|_____________________|
```

```html
<header>header</header>
<section class="wrapper">
  <aside>lef</aside>
  <section>main</section>
   <aside>right</aside>
</section>
<footer>footer</footer>
```

### 1.圣杯
```css
.wrapper {
  padding: 0 100px 0 100px;
  overflow: hidden;
}
.main {
  position: relative;
  float: left;
  width: 100%;
  height: 200px;
}
.left {
  position: relative;
  float: left;
  width: 100px;
  height: 200px;
  margin-left: -100%;
  left: -100px;
}
.right {
  position: relative;
  float: left;
  width: 100px;
  height: 200px;
  right: -100px;
  margin-left: -100px;
}
```
问题：当main部分宽度 < left部分，发生布局混乱。

### 2. 双飞翼布局
为了修复这个问题，中间变了，多层dom
```html
<section class="wrapper">
  <section class="pull-left main">
    <section class="main-wrap">main
    </section>
  </section>
  <aside class="pull-left left">left</aside>
  <aside class="pull-left right">
      right
  </aside>
</section>
```
```css
.wrapper {
  padding: 0;
  overflow: hidden;
}
.pull-left {
  float: left;
}
.main {
  width: 100%;
}
.main-wrap {
  margin: 0 100px 0 100px;
  height: 200px;
}
.left {
  width: 100px;
  height: 200px;
  margin-left: -100%;
}
.right {
  width: 100px;
  height: 200px;
  margin-left: -100px;
}
```






## 介绍 css3 中 position:sticky

公司：网易  


position属性中最有意思的就是sticky了，设置了sticky的元素，在屏幕范围（viewport）时该元素的位置并不受到定位影响（设置是top、left等属性无效），当该元素的位置将要移出偏移范围时，定位又会变成fixed，根据设置的left、top等属性成固定位置的效果。

可以知道sticky属性有以下几个特点：

- 该元素并不脱离文档流，仍然保留元素原本在文档流中的位置。

- 当元素在容器中被滚动超过指定的偏移值时，元素在容器内固定在指定位置。亦即如果你设置了top: 50px，那么在sticky元素到达距离相对定位的元素顶部50px的位置时固定，不再向上移动。

- 元素固定的相对偏移是相对于离它最近的具有滚动框的祖先元素，如果祖先元素都不可以滚动，那么是相对于viewport来计算元素的偏移量



## 什么情况会出现浏览器分层?

1. opacity !== 1;
2. transform !== none;
3. filter !== none;
4. isolution: isolate;
5. mix-blend-mode !== normal;
6. -webkit-overflow-scrolling: touch;
7. will-change: opacity/transform...;
8. z-index不为auto的弹性盒子的子元素




## 说一下 Css 预处理器，Less 带来的好处？
1. Less 是一个 CSS 预处理器。编译后，它会生成简单的CSS，适用于浏览器。

2. Less 支持跨浏览器兼容性。

3. Less 使用嵌套，使得代码更短、更干净，并以特定的方式组织

4. Less 使用变量，可以更快地实现维护。

5. Less 提供了一系列运算符，使编码更快，更省时。

6. Less 提供 @mport 规则，这样我们就可以轻松地处理外部文件。
注：导入是必需的，因为许多人将样式表分割为多个文件，而不是将其放入一个文件中。

7. Less 提供了合并属性。
Less 最令人兴奋的特征是接受多个值，如 transform，transition和 box-shadow。

8. Less 是用 JavaScript 编写的，它可以比 CSS 的其他预处理器更快地编译。



## inherit、initial、unset 三者的区别
* inherit: 指定一个属性应从父元素继承它的值。
* initial: 用于设置 CSS 属性为它的默认值。
* unset: 不做设置。但是与css是否继承属性相关，如果一个样式是可继承属性，该设置等于inherit;反之如果是非继承属性，则设置等于initial。






## 如何解决移动端 Retina 屏 1px 像素问题
1 伪元素 + transform scaleY(.5)
2 border-image
3 background-image
4 box-shadow

这个问题，现在移动端兼容性越来越好了。





## Css 实现 div 宽度自适应，宽高保持等比缩放

公司：快手

如何通过 CSS 实现高度 height 随着宽度 width 变化而变化, 保持长宽比例不变, 且宽度是根据父元素宽度变化的
使用 :before 伪元素, 能获取到实际高度 (推荐)


```css
.content{
  background: #ff0000;
  width: 100%;
}

.content:before {
  content: '';
  display: inline-block;
  padding-bottom: 100%;
  width: 0.1px;
  vertical-align: middle;
}
```






## 参考
>https://zhuanlan.zhihu.com/p/366941015
