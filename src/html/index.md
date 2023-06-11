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


## 5. 自适应
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

## 6. attr和Property
**HTML attribute 和DOM Property区别？**
| HTML attribute     | DOM Property     |
| ---------------    | -----------------|
|值永远是字符串or null  |值可以是任意合法js类型|
| 大小写不敏感          | 大小写敏感|
|不存在返回null|不存在返回undefined|
|对于href,返回html|对于href返回解析后的完整url|
|设置的值||
|更新value，属性也更新|更新value，特性不更新|

```html
<input id="name" value="justjavac" />
html -> DOM对象，HTML特性 -映射-> DOM属性
```
可添加非标属性
```html
<input foo="bar" id="name" />
```
只映射标准：
```js
const el = document.getElementById('name');
el.foo === undefined;
```

```js
el.getAttribute('checked') === ''; // 特性是字符串
el.checked === false; // 属性是boolean类型
el.getAttribute('href') === '#tag'; // 特性原样返回
el.href === 'http://xxx#tag'; // 属性返回解析后完整url
```

## 7. DOM事件流
***

事件流包括：事件捕获阶段、处于目标阶段、事件冒泡阶段
事件从文档的根节点出发，随着DOM树结构向事件目标点流去。经过各个层级DOM节点，触发捕获事件，直到到达目标节点。然后进入目标阶段，事件在目标节点上被触发，然后随着DOM树向上冒泡，直到根节点。

```js
            ________
           | window |
            ————————
              ↓↑
           __________
          | Document |
           ——————————
              ↓↑
           ________
          | <html> |
           ————————
              ↓↑
           ________
          | <body> |
           ————————
              ↓↑
           ________
          |<table> |
           ————————
              ↓↑
           ________
          |<tbody> |
           ————————
         ___/_     ↓↑______
        |<tr> |     | <tr> |
         —————       ——————
                ______↓↑   _\____
               | <td> |   | <td> |
                ——————     ——————
```




## 8. Forms API

***
### spellcheck
带有文本内容输入控件和textarea控件设置spellcheck，即拼写检查结果反馈，拼写不匹配文本有红色虚线，大部分浏览器默认启用。
```html
<textarea spellcheck="true">
```

### <font color="red">list和datalist元素</font>

输入型控件构造一张选值列表。
举个栗子：
```html
<input type="email" id="contacts" list="contactList" />
<dataList id="contactList">
   <option value="x@example.com" label="RacerX">
   <option value="peter@example.com" label="Peter">
</dataList>
```
 
### validityState对象
表单验证：ValidityState对象
```js
// 获取名为myInput表单元素的validityState对象
// 有8种约束条件，均通过则valCheck.valid 返回true
var valCheck = document.myForm.myInput.validity;

```
* valueMissing: 有required，输入为空返回true；
* typeMissing：输入不符合对应类型规则返回true；
* patternMismatch：设定正则表达式验证机制，不符合true；
* tooLong：超过设置maxLength, true;
* rangeUnderflow: 数值 < 下限，true;
* rangeOverflow: 数值 > 上限，true；
* stepMismatch：符合最小值与step倍数之和；
* customError: 自定义验证消息，消除错误：setCustomValidity("");

#### 验证反馈
所有未通过验证的表单都会接收到一个invalid事件，例如：
```js
function invalidHandler(evt) {
   var validity = evt.srcElement.validity;
   if (validity.valueMissing) {
      // 提示用户必填项
      // 可取消浏览器默认的验证反馈evt.preventDefault();
   }
}
myField.addEventListener('invalid', invalidHandler, false);
```
关闭验证：
```html
<input type="submit" formnovalidate name="save" value="保存">
```

例子：密码确认
利用customerError可用来处理表单控件的错误

```html
<form name="passwordChange">
   <p>
      <label for="password1">New Password:</label>
      <input type="password" id="password1" onchange="checkPasswords()" />
   </p>
   <p>
      <label for="password2">Confirm Password:</label>
      <input type="password" id="password2" onchange="checkPasswords()" />
   </p>
</form>
```
```js
function checkPasswords() {
   var pass1 = document.getElementById("password1");
   var pass2 = document.getElementById("password2");
   if (pass1.value !== pass2.value) {
      pass1.setCustomValidity("Your passwords do not match");
   } else {
      pass1.setCustomValidity("");
   }
}
```