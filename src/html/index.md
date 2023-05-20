# HTML
## 1. 如何理解HTML语义化？
就是让标签和其所包裹的内容的意思相吻合
1. 方便机器理解代码，利于seo
2. 代码更简洁，复用性更高，使用合适的标签，可以少一些css或js
3. 访问性更好，针对读屏器或其他一些对css理解不好的浏览器，可以做到脱离css还能看

## 2. meta viewport是做什么的？怎么写？
手机浏览器是把页面放在一个虚拟的“窗口”中，通常这个虚拟窗口比屏幕宽，用户可通过平移和缩放来看网页的不同部分。
```html
<meta name="viewport" content="width=device-width,initial-scale=1,maximum-scale=1" user-scaleble="no" />
```
即viewport宽度为物理设备上的真实分辨率，不允许用户缩放，网页显得更高更细腻。  
完美适配：用户不需缩放和横向滚动条即可查看网站所有内容，比如14px的字在何种分辨率下，显示大小应差不多。




## 3. canvas和svg区别 ？
1. svg绘制的每一个图形元素都是独立的DOM节点，canvas输出一整副画布；
2. svg输出矢量图形，修改参数不会失真or锯齿，canvas就像图片一样，放大会失真。

## 4. 什么是FOUC?
如何避免？  
Flash of Unstyled Content: 用户定义样式表加载之前浏览器使用默认样式显示文档，用户样式加载渲染之后再重新显示文档，造成页面闪烁。  
解决：样式表放到head


## 4. 自适应
UE图像素单位 => rem ?  
假设UE尺寸640px，UE稿中一个100px，css中写成什么尺寸？  
例如：用sass处理
因为100 / 640 * 100 = 15.625rem;
```html
<script>
  var docEl = document.documentElement;
  var rem = docEl.clientWidth / 100;
  docEl.style.fontSize = rem + 'px';
</script>
```
```sass
@function px2rem($px) {
  @return #{$px / $ue-width * 100}rem;
}
p {
  width: px2rem(100);
}
```