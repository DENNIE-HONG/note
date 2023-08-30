# 常问面试题
## display: none;与visibity:hidden；的区别？
1. display:none会让元素完全从渲染树中消失，渲染时候不占据任何空间。visibility: hidden不会让渲染树消失，占据空间，内容不可见。

2. display:none;非继承属性，修改子孙节点属性无法显示。visibility:hidden继承属性，通过设置可让子孙节点显示。

3. 修改常规流中元素display造成重排，修改visibility造成重绘；

4. 读屏器不读display:none内容，会读visibility内容。




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


## 清除浮动方法？
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
6. 父级div定义伪元素after和zoom
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
