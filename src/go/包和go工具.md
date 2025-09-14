# 包和Go工具
包: http://godoc.org

## 引言
每个包定义了一个不同的命名空间作为标识符。
所有导入都必须在每一个源文件的开头显式列出；包的依赖性形成有向无环图。

## 导入路径
**导入路径**在import声明中，唯一字符串进行标识。
准备共享的包或者开放的包，导入路径全局唯一，应以互联域名作为路径开始。

## 包的声明
源文件开头，都要包声明，主要目的是当该包被其他包引入时候作为默认标识符（包名）。通常，包名是导入路径的最后一段，导入路径不同，包名可以重复。

_test.go的文件，包名中会出现_test结尾，_test后缀告诉go test两个包都需要构建。

## 导入声明
import (
    "fmt"
    "os"
)
可以用*空行*进行分组：表示不同领域的包，导入顺序不重要，惯例是按照字母排序（gofmt和goimport工具都会自动进行分组并排序）。
如果遇到包名一样的，必须至少为其中一个*重命名*来避免冲突。
```go
import (
    "crypto/rand"
    mrand "math/rand" // 通过指定不同名称避免冲突
)
```
如果有依赖循环，go build工具会报错。
注意：前端编译不会报错，只有eslint配置提示。

## 空导入
如果导入包没有引用，会产生编译错误。
vs：ts不会，只是会提示eslint warning，打包摇掉这个包。
**空导入**：必须导入，为了变量初始化，执行init函数等，用_替代名字，表示导入的内容为空白标识符。

## 包及其命名
尽可能保持可读性和无歧义，避免使用有其他含义的包名。
包成员的命名：举个例子：
bytes.Equal   flag.Int    Http.Get

## go工具
go tool，用来下载、查询、格式化、构建、测试以及安装Go代码包。
go工具将不同种类的工具集合并为一个命名集，命令接口使用"瑞士军刀"风格。
build  compile packages and dependencies
clean  remove object files
get    download and install packages and dependencies

### 工作空间的组织
**GOPATH**环境变量，export GOPATH=$HOME/gobook...
GOPATH有3个子目录：
1. src目录，包含源文件，例如src/gopl.io/ch1/helloword/main.go
2. bin目录，放置像helloworld可执行程序；
3. pkg目录，构建工具存储编译后的包的位置

**GOROOT**环境变量，指定Go发行版的根目录，提供所有标准库的包，目录结构类似GOPATH，比如fmt包源代码放在$GOROOT/src/fmt，一般用户无须设置GOROOT.

### 包的下载
go get命令可以下载单一的包。go get创建的目录是远程仓库的真实客户端，而不仅仅是文件的副本。例如：
```sh
# net包目录实际是一个Git客户端
cd $GOPATH/src/golang.org/x/net
git remove -v
# 输出
# origin https://go.googlesource.com/net(fetch) 
# origin https://go.googlesource.com/net(push)
```
包导入路径中域名（golang.org）和Git服务器实际域名不同，这是go工具的特性，包可以在导入路径中自定义域名：

```html
<meta name="go-import" content="golang.org/x/net git https://go.googlesource.com/net">
```
go get -u xx, 会按照最新版本更新。

### 包的构建
*go build*命令编译包。以下三种方式都可以：

```sh
# 1
cd $GOPATH/src/gopl.io/ch1/helloworld
go build

# 2
cd anywhere
go build gopl.io/ch1/helloworld

# 3
cd $GOPATH
go build ./src/gopl.io/ch1/helloworld
```
默认情况下，go build命令构建所有需要的包以及它们所有的依赖性，然后丢弃除了最终可执行程序之外的所有编译后的代码。
*go run*: 即用即抛，构建之后尽快运行，go run xx.go 参数
*go install*和go build相似，区别是保存每个包的编译代码和命令，而不是丢弃，编译后的包保存在$GOPATH/pkg目录中。go install将保存的目录名和GOOS、GOARCH相关，因为有些包需要为特定的处理器编译不同版本的代码，比如net_linux.go.
**构建标签**：提供更细粒度的控制
// +build linux darwin
go build只会构建Linux or mac os系统应用时候才编译
// +build ignore 
都不要编译

### 包的文档化
如果想浏览自己的包，可以运行一个godoc实例，在浏览器中访问
http://localhost:8000/pkg:
godoc -http :8000

### 内部包
想在不进行导出的情况下，在项目的一些包中间共享一些工具函数，不是所有人都能访问。可以放在*internal*里，这些包叫做**内部包**。内部包只能被另一个包导入，是以internal的父目录为根目录的包。

### 包的查询
*go list*工具上报可用包的信息。go list xx，判断一个包是否存在于空间中，有会输入导入路径。
*go list ...*: 枚举一个GO工作间的所有包
*go list -json hash*：输出完整记录
