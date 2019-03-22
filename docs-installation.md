# 安装

## 在 Debian/Ubuntu 上安装
| 描述 | 安装包下载地址 |
| :----- | :----- |
| 基于 Debian Linux 的稳定版 | [x86-64](https://grafana.com/grafana/download?platform=linux) |
| 基于 Debian Linux 的稳定版 | [ARM64](https://grafana.com/grafana/download?platform=arm) |
| 基于 Debian Linux 的稳定版 |[ARMv7](https://grafana.com/grafana/download?platform=arm) |

升级现有版本的话可以查看[升级Grafana](http://docs.grafana.org/installation/upgrading/)文档中的注意事项和指南.

### 安装稳定版
```
wget <debian package url>
sudo apt-get install -y adduser libfontconfig
sudo dpkg -i grafana_<version>_amd64.deb
```
例如:
```
wget https://dl.grafana.com/oss/release/grafana_5.4.2_amd64.deb
sudo apt-get install -y adduser libfontconfig
sudo dpkg -i grafana_5.4.2_amd64.deb
```

### APT 仓库
创建 `/etc/apt/sources.list.d/grafana.list` 这个文件,然后把下面这一行加进去.
```
deb https://packages.grafana.com/oss/deb stable main
```
下面是Beta版的仓库.
```
deb https://packages.grafana.com/oss/deb beta main
```
如果你用的是 Ubuntu 或者其的 Debian 版本的请使用上面那行命令.然后添加我们的GPG公钥,这样你才能安装有签名的安装包.
```
curl https://packages.grafana.com/gpg.key | sudo apt-key add -
```
更新你的 APT仓库 后就可以安装 Grafana 了
```
sudo apt-get update
sudo apt-get install grafana
```
在一些版本较老的Ubuntu和Debian上你可能需要安装 `apt-transport-https` 包来获取通过HTTPS方式获取的包.
```
sudo apt-get install -y apt-transport-https
```

### 安装明细
* 二进制文件安装到 `/usr/sbin/grafana-server`
* Init.d 启动脚本安装到 `/etc/init.d/grafana-server`
* 创建default文件(环境变量)到 `/etc/default/grafana-server`
* 安装配置文件到 `/etc/grafana/grafana.ini`
* 安装 systemd 服务(如果systemd可用的话)名到 `grafana-server.service`
* 默认的日志文件路径 `/var/log/grafana/grafana.log`
* 默认指定 sqlite3 数据库 `/var/lib/grafana/grafana.db`
* 安装 HTML/JS/CSS 以及其他 Grafana 文件到 `/usr/share/grafana`

### 启动服务(init.d 服务)
通过下面命令运行 Grafana:
```
sudo service grafana-server start
```
这样将会以安装时候创建的 `grafana` 用户的权限启动 `grafana-server` 进程.默认的 HTTP 端口是 `3000`,然后默认的用户组是 `admin`.

默认的登录账号信息为: `admin/admin`

配置 `Grafana server` 开机启动:
```
sudo update-rc.d grafana-server defaults
```

### 启动服务(通过 systemd)
使用 systemd 启动服务:
```
systemctl daemon-reload
systemctl start grafana-server
systemctl status grafana-server
```
配置 `Grafana server` 开机启动:
```
sudo systemctl enable grafana-server.service
```

### 环境变量文件
systemd 服务文件和 init.d 脚本都使用 `/etc/default/grafana-server` 这个文件中的环境变量进行后台启动.你可以自行修改日志目录,数据目录以及其他变量.

### 日志
默认写到 `/var/log/grafana`

### 数据库
默认指定sqlite3数据库到 `/var/lib/grafana/grafana.db`.升级之前请务必备份这个数据库.你也可以使用 MySQL 或者 Postgres 作为 Grafana 的数据库,详见[配置页面](http://docs.grafana.org/installation/configuration/#database).

### 配置
配置文件是 `/etc/grafana/grafana.ini`.所有详细的配置可以看[配置页面](http://docs.grafana.org/installation/configuration/).

### 添加数据源
* [Graphite](http://docs.grafana.org/features/datasources/graphite/)
* [InFluxDB](http://docs.grafana.org/features/datasources/influxdb/)
* [OpenTSDB](http://docs.grafana.org/features/datasources/opentsdb/)
* [Prometheus](http://docs.grafana.org/features/datasources/prometheus/)

### 使用二进制包安装
下载[最新的`.tar.gz`文件](https://grafana.com/get)然后解压.将会解压到以版本号为后缀的目录.这个目录包含 Grafana 运行所需的所有文件.这个包不包含 init 脚本以及安装脚本.

添加 `custom.ini` 配置文件到 `conf` 目录,然后重新配置 `conf/defaults.ini` 里面的所有配置项.

执行 `./bin/grafana-server web` 启动 Grafana.`grafana-server` 这个二进制文件的工作目录要在安装的根目录下(二进制文件和 `public` 目录所在的位置).

### 首次登录
使用浏览器打开 [http://localhost:3000/](http://localhost:3000/).如果你没有[配置不同端口](http://docs.grafana.org/installation/configuration/#http-port)的话,3000是 Grafana 的默认监听端口.接下来的一些介绍在[这里](http://docs.grafana.org/guides/getting_started/). 


## 在基于RPM的Linux上安装(CentOS,Fedora,OpenSuse,RedHat)
