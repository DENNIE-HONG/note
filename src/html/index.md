# HTML
## 如何理解HTML语义化？
就是让标签和其所包裹的内容的意思相吻合
1. 方便机器理解代码，利于seo
2. 代码更简洁，复用性更高，使用合适的标签，可以少一些css或js
3. 访问性更好，针对读屏器或其他一些对css理解不好的浏览器，可以做到脱离css还能看

## meta viewport是做什么的？怎么写？
手机浏览器是把页面放在一个虚拟的“窗口”中，通常这个虚拟窗口比屏幕宽，用户可通过平移和缩放来看网页的不同部分。

```html
<meta name="viewport" content="width=device-width,initial-scale=1,maximum-scale=1" user-scaleble="no" />
```
即viewport宽度为物理设备上的真实分辨率，不允许用户缩放，网页显得更高更细腻。