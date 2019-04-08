## **升级 Grafana**

我们建议大家经常升级 *```Grafana```*, 以便及时了解最新的修复和增强功能.为此, *```Grafana```* 升级是向后兼容的,升级过程简单快捷.

升级通常是安全的(在主版本下的次版本之间),仪表盘和图表看起来也是一样的.当然,也有些边缘次版本会有不兼容的变更,在[发布说明](https://community.grafana.com/c/releases)和[变更日志](https://github.com/grafana/grafana/blob/master/CHANGELOG.md)中都会列出.

### **数据库备份**
在升级之前最好备份一下 *```Grafana```* 数据库.这样可以确保你能够回滚到先前的版本.在启动过程中, *```Grafana ```* 会自动迁移数据库结构(如果欧变更或者新的表).如果你之后想要降级,这有时可能会有问题.

#### **sqlite**

如果是使用 *```sqlite```*, 你只需要备份 *```grafana.db```* 文件.在类 *```unix```* 系统上, 通常在 *```/var/lib/grafana/grafana.db```*.如果不确定使用的数据库以及存储位置, 请检查下 *```grafana```* 配置文件.如果使用定制的二进制安装包 ```tar/zip```, 则在 *```<grafana_install_dir>/data```*.

#### **mysql**
```
backup:
> mysqldump -u root -p[root_password] [grafana] > grafana_backup.sql

restore:
> mysql -u root -p grafana < grafana_backup.sql
```

#### **postgres**
```
backup:
> pg_dump grafana > grafana_backup

restore:
> psql grafana < grafana_backup
```

### **Ubuntu / Debian**
如果你通过下载 *```Debian```* 的安装包(*```.deb```*), 你只要根据相同的安装说明然后执行相同的 *```dpkg -i```* 命令, 而只是包是新的而已.这样就可以升级你的 *```Grafana```* 了.

如果你使用我们的 *```APT```* 仓库:
```
sudo apt-get update
sudo apt-get install grafana
```

#### **从二进制 tar 包升级**
如果你已经下载二进制 *```tar ```* 包, 你只要下载解压新的包覆盖原来的即可.但这可能会覆盖你的配置.我们推荐把自定义配置放在 *```<grafana_install_dir>/conf/custom.ini```*, 因为这样更容易升级, 避免配置被修改的风险.

### **Centos / RHEL**
如果你通过下载 *```rpm```* 包进行安装, 你只要根据相同的安装说明然后执行相同的 *```yum install```* 或者 *```rpm -i```* 命令即可, 而只是包是新的而已.这样就可以升级你的 *```Grafana```* 了.

如果你使用我们的 *```YUM```* 仓库:
```
sudo yum update grafana
```

### **Docker**
下面只是一个例子, 具体根据你如何配置你的 *```Grafana```* 容器.
```
docker pull grafana
docker stop my-grafana-container
docker rm my-grafana-container
docker run --name=my-grafana-container --restart=always -v /var/lib/grafana:/var/lib/grafana
```

### **Windows**
如果你下载 *```Windows```* 二进制包进行安装, 你只需要下载新的包然后解压覆盖原先的目录即可(覆盖原先存在的文件).但这可能会覆盖你的配置.我们推荐把自定义配置放在 *```<grafana_install_dir>/conf/custom.ini```*, 因为这样更容易升级, 避免配置被修改的风险.

### **从1.x版本升级**
[从1.x迁移到2.x](http://docs.grafana.org/installation/migrating_to2/)

### **从2.x版本升级**
我们不知道直接从2.x升级到4.x会有什么问题, 但为了安全起见, 请通过3.x => 4.x.

### **升级到v5.0**
仪表板的网格布局引擎有进行更改.你在v5版本中加载时, 所有仪表板将自动升级到新的定位系统.在v5版本中保存的仪表板在旧版 *```Grafana```* 中不能使用.有些第三方面板插件可能需要更新才能正常工作.

更多细节,请[点击](http://docs.grafana.org/reference/dashboard/#panel-size-position).

### **升级到v5.2**
其中一个版本的数据库迁移, 包括此版本, 所有注释的时间戳将从秒更新到毫秒的精度.如果你有大量的注释需要进行数据库迁移可能会花很长时间, 如果使用 *```systemd```* 命令的话可能会有问题.

我们有收到一个使用 *```systemd```* 命令的反馈, *```PostgreSQL```* 以及大量注释(表大小1645mb)花了8-20分钟完成数据库迁移.然而, *```grafana-server```* 进程在90秒后被 *```systemd```* 杀死.正在进行的任何数据库迁移查询在 *```systemd```* 杀死 *```grafana-server```* 进程后会继续在数据库中执行直到完成.

如果你使用 *```systemd```* 进行迁移且包含大量注释时, 升级之前考虑临时调整一下 *```systemd```* 的 *```TimeoutStartSec```* 设置, 例如30分钟.

### **升级到v6.0**
如果你有带脚本标签的 *```text Panel```* 将无法继续工作, 因为新的配置导致默认情况下不允许未经审查的 *```HTML```*.新配置的更多信息在[这里](http://docs.grafana.org/installation/configuration/#disable-sanitize-html).

#### **认证和安全**
如果使用 *```Grafana```* 内置的认证, *```LDAP```* (不使用认证代理) 或者 *```OAuth```* 认证, 在升级之后, 所有用户都需要重新登录后才能访问.

如果 *```session```* 段中的 *```cookie_secure```* 设置为 *```true```*, 那么相应的可能也要在 *```security```* 段中设置 *```cookie_secure```* 为 *```true```*.以类似下面的配置结尾:
```
[session]
cookie_secure = true

[security]
cookie_secure = true
```

*```login_remember_days```*, *```cookie_username```*, 和 *```cookie_remember_name```* 这几个配置项将不再使用, 因此可以安全删除.

如果你的 *```login_remember_days```* 设置为0(zero), 你要改变你的配置以实现类似的效果, i.e. 登录用户1天后将需要重新登录:
```
[auth]
login_maximum_inactive_lifetime_days = 1
login_maximum_lifetime_days = 1
```
默认用于存储的认证令牌的 *```cookie```* 名称是 *```grafana_session```*, 你可以自己在 *```[auth]```* 段中配置自己的 *```login_cookie_name```*.
