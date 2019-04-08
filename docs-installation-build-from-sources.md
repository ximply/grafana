## **从源码构建**

以下指南将教你如何在开发环境中打包然后跑起来.*```Grafana```* 自带后端服务;而且完全开源.它采用 *```Go```* 语言编写并且有丰富的 [HTTP API](http://docs.grafana.org/v2.1/reference/http_api/).

### **依赖**
- [Go(最新稳定版)](https://golang.org/dl/)
- [Git](https://git-scm.com/downloads)
- [NodeJS TLS](https://nodejs.org/download/)
- *```node-gyp```* 是 *```Node.js```* 原生自带的构建工具,它需要额外的依赖: *```python 2.7```*,*```make```* 和 *```GCC```*.

后端 *Web* 服务也有一些配置项.详见[配置](http://docs.grafana.org/installation/configuration/)页面.大部分的 *```Linux```* 发行版和 *```MacOS```* 上都有安装这些.更多关于 *```Windows```* 上构建的信息可以参阅 [node-gyp 安装说明](https://github.com/nodejs/node-gyp#installation).


### **获取源码**
创建项目目录并设置相应路径(或者使用 [默认的 Go 默认工作目录](https://golang.org/doc/code.html#GOPATH)).然后下载和安装 *```Grafana```* 到你的 *```$GOPATH```* 目录:
```
export GOPATH=`pwd`
go get github.com/grafana/grafana
```
在 *```Windows```* 上使用 *```setx```* 代替 *```export```* 然后重启命令提示符.
```
setx GOPATH %cd%
```
你可能回看到下面的错误:

```package github.com/grafana/grafana: no buildable Go source files```.这只是一个警告,你可以继续执行后续指令.

### **构建后端**

```
cd $GOPATH/src/github.com/grafana/grafana
go run build.go setup
go run build.go build              # (or 'go build ./pkg/cmd/grafana-server')
```

#### **在Windows上构建**
*```Grafana```* 后端包含的 *```Sqlite3```* 需要使用 *```GCC```* 编译.因此,为了在 *```Windows```* 上编译 *```Grafana```*,你需要安装 *```GCC```*.我们推荐 [TDM-GCC](http://tdm-gcc.tdragon.net/download). 

*```node-gyp```* 是 *```Node.js```* 原生自带的构建工具,它需要在 *```Windows```* 额外的依赖.以管理员身份运行命令提示符,然后执行:
```
npm --add-python-to-path='true' --debug install --global windows-build-tools
```

### **构建前端资源**
你需要 *```nodejs```* (v.6+)
```
npm install -g yarn
yarn install --pure-lockfile
yarn start
```

### **本地运行 Grafana**
可以通过下面命令运行本地 *```Grafana```* 服务:
```
./bin/grafana-server
```
或者,你想通过 *```go run build.go build```* 构建二进制文件,可以执行 *```./bin/<os>-<architecture>/grafana-server```*

如果你使用 *```go build .```* 构建的话,可以执行 *```./grafana```*

打开你的浏览器 (默认 [http://localhost:3000/](http://localhost:3000/)) 然后使用 ```admin``` 用户登录(默认 ```user/pass = admin/admin```).


### **Grafana 开发**
要增加功能,自定义配置,等等,如果有变更代码的话你需要重新构建后端.我们使用 *```bra```* 工具进行如下操作.
```
go get github.com/Unknwon/bra

bra run
```
你可能还需要运行 *```yarn start```* 来监视前端(*```typescript, html, sass```*)这类文件是否有变更.

#### **运行测试**
- 运行 *```go test ./pkg/...```* 进行后端测试.
- 运行 *```yarn test```* 可以进行所有前端测试.

书写&观察前端测试.
- 启动监视器: *```yarn jest```*
- *```Jest```* 将执行所有以 *```".test.ts"```* 结尾的测试文件.

#### **提供数据源和仪表盘**
[这里](https://github.com/grafana/grafana/tree/master/devenv)你可以找到一些有用的脚本和 *```docker-compose```* 构建,这样会完善你的开发环境,以便更快地测试和体验.

### **创建优化版的发布包**
该步骤将构建 *```Linux```* 包,需要安装 *```fpm```* 打包工具.可以通过 *```gem install fpm```* 安装.
```
go run build.go build package
```

### **开发环境相关配置**
在 *```conf```* 目录创建一个 *```custom.ini```* 配置文件以覆盖默认配置.你只需要添加要覆盖的项,配置文件将按下面的顺序加载并设置:
```
1.grafana.ini
2.custom.ini
``` 

#### **设置 app_mode 为开发环境模式**
删除开头的 *```;```* 号,可以取消 *```custom.ini```* 的注释.然后设置 *```app_mode = development```*.

可以在[配置部分](http://docs.grafana.org/installation/configuration/)了解更多的 *```Grafana```* 的配置项.

### **创建拉取请求**
请为 *```Grafana```* 项目做贡献,提交你的 *```PR```*！构建新功能,编写或者更新文档,修复 *```Bugs```* 都会使 *```Grafana```* 变得更棒.

### **故障排查**
**问题:** 运行 *```grunt```* 时出现 *```PhantomJS```* 或者 *```node-sass```* 错误.

**解决:** 删除 *```node_modules```* 目录.安装适用于你的平台的 [node-gyp](https://github.com/nodejs/node-gyp#installation). 然后再次运行 *```yarn install --pure-lockfile```*.

**问题:** 第一次执行 *```bra run```* 报无法识别的命令的错.

**解决:** 将 *```Go```* 工作目录中的 *```bin```* 目录添加到 *```path```* 中.如果你已经设置了自己的工作目录,*```Linux```* 默认的是 *```$HOME/go/bin```*,*```Windows```* 是 *```%USERPROFILE%\go\bin```* 或者 *```$GOPATH/bin```*.

**问题:** *```Windows```* 上执行 *```go get```* 命令报 *```git```* 创库不存在的错.

**解决:** *```go get```* 依赖 *```Git```*.如果没有 *```Git```*,执行 *```go get```* 会在 *```Go```* 工作区中创建一个你想要获取的仓库的空目录.即使安装了 *```Git```* 之后,还是报同样的错.可以这么做,删除上面提到的那个空目录(例如: 如果你执行 *```go get github.com/Unknwon/bra```* 那么需要删除 *```%USERPROFILE%\go\src\github.com\Unknwon\bra```*)然后再次执行 *```go get```*.

**问题:** 在 *```Windows```* 上,报已安装的工具找不到的错.

**解决:** 这个一般是你没有配置 *```path```* 环境变量导致的,配置后重新打开命令提示符即可正常使用.

### **首次登录**
使用浏览器打开 [http://localhost:3000/](http://localhost:3000/).如果你没有[配置不同端口](http://docs.grafana.org/installation/configuration/#http-port)的话,3000是 Grafana 的默认监听端口.接下来的一些介绍在[这里](http://docs.grafana.org/guides/getting_started/). 
