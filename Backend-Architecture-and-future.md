# 后端:架构和后续规划 [原文](https://github.com/grafana/grafana/wiki/Backend:-Architecture-and-future)
Torkel Ödegaard 于2018.6.4编辑本页面

Grafana 的后端采用GO语言编写, 使用 sqlite3/mysql 或者 postgres 作为仪表板,用户数据等的存储. Grafana 诞生之初并没有太多关于如何编写中型应用程序的指南或方向. 因此 Grafana 部分基础代码没有我们想要的那么通用. 那么下文就说下更多关于当前重写的内容! :)

## Grafana 的后端核心组件
| 包 | 描述 |
| :------| :------ |
| /pkg/api | Http处理程序和路由. 大部分的处理程序函数是全局的,而这是我们后续要改进的. 处理程序应该与引用所有依赖的结构体相关联. 额外的总线 |
| /pkg/bus | 总线! 下面会有更多的介绍 |
| /pkg/cmd | 我们要构建的二进制文件. Grafana-server 和 grafana-cli |
| /pkg/metrics | 仪器代码用于检测 Grafana 本身. 目前使用 Prometheus 客户端库抓取数据,不过也可以把数据导入 graphite. |
| /pkg/models | 这是我们保留 domain model 的地方. 这个包不依赖于标准库外的任何程序包. (确实有包含了一些 Grafana 中的引用,不过这正是我们后续要避免的) |
| /pkg/registry | service 管理包. |
| /pkg/services/alerting | Grafana 的报警服务. 该报警引擎运行在独立的 go routine 而且不依赖于包括 Grafana 在内的其他东西. |
| /pkg/services/sqlstore | 目前所有的数据库调用都在这里. |
| /pkg/setting | Grafana 的配置包. 任何与 Grafana 相关的全局配置都可以使用这个包来处理. |
| /pkg/tsdb | Grafana datasources 的所有后端实现. 包括 Grafana 前端以及报警服务也在使用. |

## 测试
我们重视清晰可读的代码,松耦合并且覆盖单元测试.这样可以更轻松地协作和维护代码.在 sqlstore 包中，我们在测试中进行数据库操作，而有些人可能会说这不适合单元测试.我们认为这样做相当快并且有价值.

我们的大多数测试都使用 go 输出，但标准库测试也做的很好。

## 总线
总线是我们在http处理程序和 sqlstore 之间引入隔离的方法(负责从数据库中获取数据).Http处理程序和 sqlstore 相不依赖.它们只依赖于总线和 domain model（pkg / models）.这样可以更轻松地测试代码,避免耦合.更多信息详见*目前的重写/重构*

## 服务/知识图谱
Grafana 中的服务是要自包含的，并且只能使用通过 Grafana 服务注册表提供的总线或知识图谱与 Grafana 的其他部分通信.所有服务都应该在init函数中自注册.只能在init函数中进行注册.应尽可能避免使用Init函数.

当 Grafana 启动时，将调用服务中的所有init函数并自行注册.然后,Grafana 将创建所有依赖项的图并注入其他服务所依赖的服务中.这在 https://github.com/grafana/grafana/blob/master/pkg/cmd/grafana-server/server.go#L75 通过[注入库](https://github.com/facebookgo/inject)解决了.

## 扩展性
应该可以使用配置文件配置所有需要的新功能.额外的数据源,警报通知程序,仪表板,团队管理等.目前,它只能配置数据源和仪表板,但我们想要支持所有 Grafana 的东西.

## 当前的重写/重构
- 删除全局变量
- 尽可能避免使用内部变量
- 减少使用init（）函数,只注册服务/实现
  
### 配置重构
计划是将所有配置项从配置包中的包级别变量移动到setting.Cfg对象.可以通过注入setting.Cfg对象来访问配置 services/components.

Cfg 对象: https://github.com/grafana/grafana/blob/master/pkg/setting/setting.go#L184

注入实例: https://github.com/grafana/grafana/blob/master/pkg/services/cleanup/cleanup.go#L19

### Sqlstore 重构
sqlstore 处理程序使用全局的 xorm 引擎变量,这要重构为 Sqlstore 实例.

### Http 处理程序重构
Http 处理程序要重构为 HttpServer 实例的方法.这样他们就可以访问 HttpServer 的服务依赖项（以及 Cfg 对象）.

## 编程风格
我们的目标是遵循[golang的风格](https://github.com/golang/go/wiki/CodeReviewComments),但我们也不会全盘接受.

当我们进行持续集成构建时,运行 *gofmt -w -s* 和 *gometalinter*,以验证是否最重要的部分获益.
