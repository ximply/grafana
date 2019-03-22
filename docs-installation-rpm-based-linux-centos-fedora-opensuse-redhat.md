## 在基于RPM的Linux上安装(CentOS,Fedora,OpenSuse,RedHat)
| 描述 | 安装包下载地址 |
| :----- | :----- |
| 基于 CentOS / Fedora / OpenSuse / Redhat Linux 的稳定版 | [x86-64](https://grafana.com/grafana/download?platform=linux) |
| 基于 CentOS / Fedora / OpenSuse / Redhat Linux 的稳定版 | [ARM64](https://grafana.com/grafana/download?platform=arm) |
| 基于 CentOS / Fedora / OpenSuse / Redhat Linux 的稳定版 |[ARMv7](https://grafana.com/grafana/download?platform=arm) |

升级现有版本的话可以查看[升级Grafana](http://docs.grafana.org/installation/upgrading/)文档中的注意事项和指南.

### 安装稳定版
可以直接使用 Yum 安装.
```
$ sudo yum install <rpm package url>
```
例如:
```
$ sudo yum install https://dl.grafana.com/oss/release/grafana-5.4.2-1.x86_64.rpm
```
或者使用 `rpm` 包手动安装.首先执行
```
$ wget <rpm package url>
```
例如:
```
$ wget https://dl.grafana.com/oss/release/grafana-5.4.2-1.x86_64.rpm
```

### CentOS / Fedora / Redhat 上:
```
$ sudo yum install initscripts fontconfig
$ sudo rpm -Uvh <local rpm package>
```

### OpenSuse 上:
```
$ sudo rpm -i --nodeps <local rpm package>
```

### 通过 YUM 仓库安装
添加以下内容到 `/etc/yum.repos.d/grafana.repo` 这个文件
```
[grafana]
name=grafana
baseurl=https://packages.grafana.com/oss/rpm
repo_gpgcheck=1
enabled=1
gpgcheck=1
gpgkey=https://packages.grafana.com/gpg.key
sslverify=1
sslcacert=/etc/pki/tls/certs/ca-bundle.crt
```
下面这个是单独的 Beta 版的仓库.
```
[grafana]
name=grafana
baseurl=https://packages.grafana.com/oss/rpm-beta
repo_gpgcheck=1
enabled=1
gpgcheck=1
gpgkey=https://packages.grafana.com/gpg.key
sslverify=1
sslcacert=/etc/pki/tls/certs/ca-bundle.crt
```
然后使用 `yum` 名利安装 Grafana.
```
$ sudo yum install grafana
```
### RPM GPG 公钥
RPMs 是有带签名的,你可以通过这个[GPG公钥](https://packages.grafana.com/gpg.key)进行校验.

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

### 服务端图形渲染
服务器端图形(png)渲染是可选功能,但是进行可视化分享的时候就很有用了,例如报警通知可以附图.

如果图形出现文本缺失,那么你得保证字体包已经安装.
```
yum install fontconfig
yum install freetype*
yum install urw-fonts
```

### 使用二进制包安装
下载[最新的`.tar.gz`文件](https://grafana.com/get)然后解压.将会解压到以版本号为后缀的目录.这个目录包含 Grafana 运行所需的所有文件.这个包不包含 init 脚本以及安装脚本.

添加 `custom.ini` 配置文件到 `conf` 目录,然后重新配置 `conf/defaults.ini` 里面的所有配置项.

执行 `./bin/grafana-server web` 启动 Grafana.`grafana-server` 这个二进制文件的工作目录要在安装的根目录下(二进制文件和 `public` 目录所在的位置).

### 首次登录
使用浏览器打开 [http://localhost:3000/](http://localhost:3000/).如果你没有[配置不同端口](http://docs.grafana.org/installation/configuration/#http-port)的话,3000是 Grafana 的默认监听端口.接下来的一些介绍在[这里](http://docs.grafana.org/guides/getting_started/). 
