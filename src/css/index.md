# CSS

## 父元素高度坍塌
一个块级元素没有设置height, 其height由子元素撑开。子元素浮动后脱离标准文档流，父元素没有内容可撑开，其height被忽略。

## 清除浮动方法
1. 父级div设高；
2. 使用空元素
```html
<div class="clear"></div>
```
```css
.clear{
  clear: both;
}
```
3. 父元素div一起浮动
4. 父级div定义display: table;
5. 父元素触发了BFC
```css
.fu {
  overflow: hidden;
}
```
6. 父级div定义伪类after和zoom
```css
.clearfix: after {
  content: '.';
  display: block;
  height: 0;
  clear: both;
  visibility: hidden;
}
// 兼容IE6/IE7
.clear {
  zoom: 1;
}
```

## 清除图片底部间隙
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

## 格式化上下文
* BFC,block formatting context
* IFC, inline formatting context(一个区域仅包含水平排列元素)
* FFC, flex formatting context
* GFC, grid formatting context

### inline box(行框)
包含IFC内部的所有子元素的虚拟矩形区域，形成的每一行。  
行框的模型结构：（6条线）
* top线
* text-top线
* middle线
* baseline线
* text-bottom线
* bottom线
```js
 _ _ _ _ _ _ _  ————> top线
|___________  |————> text-top             
|_ _ _xaj_ _ _|----> middle
|_____________|____ bottom线


```
#### 图片留白问题？
因为：可置换行内元素如img和inline-block的元素，在被浏览器解析时，会和“文本”一样，默认在baseline线上显示，而不是在行框的bottom线上。


### BFC是什么？
Block Formatting Context(块级格式化上下文)
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



## 一个不定宽高的DIV,垂直水平居中？
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

3. 利用table
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


## 伪元素
::before 和 :after中双冒号和单冒号有什么区别？
单冒号用于css伪类；
双冒号用于css伪元素；


## 流体布局
块级元素：
1. width固定，margin是auto，即margin撑满剩下的空间；
2. margin固定，width是auto，width撑满剩下的空间；

### css的外在盒子和内在盒子
例：display：inline-table;
外在盒子：inline,决定像内联元素一样并排显示；
内在盒子：table，决定了元素可以设置宽高、垂直方向的margin等。

### 块级元素
垂直方向发生margin合并：
1. 相邻兄弟元素之间margin合并；
2. 父的margin-top和子的margin-top；
  父的margin-bottom和子的margin-bottom；
3. 空块级元素自身的margin-top和margin-bottom;

阻止margin合并方法：
1. bfc;
2. border、padding阻隔margin；
3. 内联元素阻隔；
4. 父元素设高；

### 内联元素
vertical-align:
* sub super: 使元素的基线与父元素下标基线对齐；
* 20px 2em: baseline相当于0，使元素的基线对齐到父元素的基线之上给定长度；
* 20%：元素的基线对齐到父元素的基线之上的百分比，是line-height的百分比；

## 层叠上下文
css3新增层叠上下文：
1. opacity !== 1;
2. transform !== none;
3. filter !== none;
4. isolution: isolate;
5. mix-blend-mode !== normal;
6. -webkit-overflow-scrolling: touch;
7. will-change: opacity/transform...;
8. z-index不为auto的弹性盒子的子元素

层叠上下文内元素的层叠水平：
```js
 ___________________
| border/background |
|    __________________
 ———|负值z-index       |
    |   ________________
     ——|块级盒子         |
       |   ________________
        ——|浮动盒子         |
          |   ________________ 
           ——|内联盒子         |
             |   ________________
              ——|z-index:auto/0  |
                |   ________________
                 ——|正z-index       |
                   |                |
                    ————————————————
```    



## 小技巧
### 有切角的阴影
```css
filter: drop-shadow(2px 2px 10px rgba(0, 0, 0, 0.5));
```
### 两端对齐
```css
hyphens: auto;
```

### 小箭头自动继承背景和边框样式
```css
.callout {
  postion: relative;
}
.callout::before {
  content: '';
  position: absolute;
  background: inherit;
  boder: inherit;
  border-right: 0;
  border-bottom: 0;
  padding: 0.35em;
  transform: rotate(45deg);
}

```

### 禁止文本缩放
```css
-webkit-text-size-adjust: 100%;
```

### 字体在移动端比例缩小后出现锯齿
```css
-webkit-font-smoothing: antialiased;
```

### Retina下1px的解决方案
在devicePixelRatio = 2时，借助媒体查询来处理
```css
.border {
  border: 1px solid black;
}
@media (-webkit-min-device-pixel-ratio: 2) {
  .border {
    border-width: 0.5px;
  }
}
```

### css数据上报
比如根据某个按钮的点击事件
```css
.button:active::after {
  content: url(./pixel.gif?action=click&id=button)
  display: none;
}
```
比如统计用户表单操作成功率
```css
.track:invalid {
  background: url(./pixel.gif?action=rogister&status=invalid);
}
```