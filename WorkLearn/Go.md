# Go语言知识点

## 环境配置
### 查看Go的环境变量
```
go env
```
### 设置GOPROXY代理
```
go env -w GOPROXY=https://goproxy.cn,direct
```

## Go mod
### go.mod说明
用来管理包的说明文件,现阶段非常简单,文件内一共四种指令
- module: 模块名称
- require: 依赖包列表以及版本
- exclude: 禁止依赖包列表(仅在当前模块为主模块时生效)
- replace: 替换依赖包列表(仅在当前模块为主模块时生效)

### go mod常用命令
- go help mode: 查看帮助信息,可以查询下面的命令功能
- go mod tidy: 添加缺失的模块,移除未使用的模块
- go mod verify: 验证依赖项是否达到预期目的,没问题的返回值 all modules verified
- go mod graph: 打印模块要求图
- go mod download: 下载模块到本地缓存,也可以通过命令 go clean -cache 清除缓存
- go mod edit -editing flags: 从工具或脚本中编辑go.mod,根据flag不同，作用不同
- go mod why: 解释为什么需要包或模块
- go mod vendor: 制作依赖项的副本，实际测试就是把包copy一份到目录下保存

## Go 交叉编译 (跨平台编译)
### Mac 下编译 Linux 和 Windows 64位可执行程序
```
CGO_ENABLED=0 
GOOS=linux 
GOARCH=amd64 
go build main.go
​
CGO_ENABLED=0 
GOOS=windows 
GOARCH=amd64 
go build main.go
```

###  Linux 下编译 Mac 和 Windows 64位可执行程序
```
CGO_ENABLED=0 
GOOS=darwin 
GOARCH=amd64 
go build main.go
​
CGO_ENABLED=0 
GOOS=windows 
GOARCH=amd64 
go build main.go
```
###  Windows 下编译 Mac 和 Linux 64位可执行程序
```
SET CGO_ENABLED=0
SET GOOS=darwin
SET GOARCH=amd64
go build main.go
​
SET CGO_ENABLED=0
SET GOOS=linux
SET GOARCH=amd64
go build main.go
```
