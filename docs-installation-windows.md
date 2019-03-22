## 在Windows上安装
| 描述 | 安装包下载地址 |
| :----- | :----- |
| 最新的Windows稳定版安装包 | [x64](https://grafana.com/grafana/download?platform=windows) |

升级现有版本的话可以查看[升级Grafana](http://docs.grafana.org/installation/upgrading/)文档中的注意事项和指南.

### 配置
要点:下载完压缩包后还未解压之前,要用右键打开文件属性菜单然后点击确认 `解压`.

压缩包包含当前版本的 Grafana 的文件.可以解压到你想要的地方.进到 `conf` 目录然后复制 `sample.ini` 为 `custom.ini`,你要编辑 `custom.ini`,而不是 `defaults.ini`.

Grafana 的默认端口是3000,这个端口在Windows上需要有额外的权限.编辑 `custom.ini` 然后注释掉 `http_port` 这个配置项(`;` 是该文件的注释字符)然后改成  `8080` 或者其他端口.这个端口不需要额外的权限.

默认的登录账号信息为: `admin/admin`

通过执行 `bin` 目录下的 `grafana-server.exe` 来启动 Grafana,而且最好是在命令行下执行.如果你想要以Windows服务的形式运行 Grafana,可以下载[NSSM](https://nssm.cc/)这个工具.它可以简单的把 Grafana 添加为Windows服务.

更多配置详见[配置项](http://docs.grafana.org/installation/configuration/).

### 首次登录
使用浏览器打开你上面配置的例如[http://localhost:8080/](http://localhost:8080/)这个地址,那么接下来的一些介绍在[这里](http://docs.grafana.org/guides/getting_started/). 

### 在Windows上进行构建
Grafana 后端所使用的sqlite3需要GCC编译器来编译.所以为了在Windows上编译 Grafana 你需要安装GCC.我们推荐[TDM-GCC](http://tdm-gcc.tdragon.net/download).

