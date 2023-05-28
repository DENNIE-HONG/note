# Babel
js编译器，涉及创建操作抽象语法树（AST）
步骤：解析 ——> 转换 ——> 生成

#### 解析
输出AST:
* 词法分析；代码——> 令牌流
* 语法分析：令牌流——> AST

#### 转换
接AST进行遍历，对节点添加、更新、移除

#### 生成
变换后AST ——> 字符串形式代码

## AST
AST每一层都有：
* type: "FunctionDeclaration"
* id: {...}
* params: [...]
* body: {...}

额外属性start/end/loc等，描述该节点在原始代码中的位置。

## 配置
preset: 插件集合。例如es2015包含十几二十转义插件，可单个安装也可使用preset套餐。
执行顺序：
* plugin: 运行在preset之前，从前到后顺序执行；
* preset：从后向前（保证向后兼容）,例如preset: ["es2015", "stage-0"]先执行stage-0才不报错。

env：通过配置得知目标环境的特点，只做必要转换。  
babel-polyfill:转换新的API(很大)，babel会转换js语法，但每个转化的文件都会插入一个段转换后定义的代码导致重复。  
使用babel-plugin-transform-runtime之后，从定义方法改成引用，重复定义即重复引用，不存在代码重复了。  
babel-runtime是这些方法集合处:
1. core-js:转换一些内置类（Promise、Symbol等）、静态方法等；
2. regenerator:generator/yeild、async/await、
3. helpers
