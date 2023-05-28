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
