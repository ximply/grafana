## **配置**

*```Grafana```* 后端有许多配置选项可以在 *```.ini```* 配置文件中指定或使用环境变量指定.

>注意. 需要重新启动 *```Grafana```* 才能使配置更改生效.

### **.ini 配置文件注释**
分号(*```;```* 字符)是注释 *```.ini```* 文件中的行的标准方法.

常见的问题是忘记取消 *```custom.ini```*(或 *```grafana.ini```*)文件中的注释, 导致配置项被忽略.

### **配置文件位置**
- 默认配置文件 *```$WORKING_DIR/conf/defaults.ini```*
- 自定义配置文件 *```$WORKING_DIR/conf/custom.ini```*
- 自定义配置文件的位置可以使用 *```--config```* 参数指定
>注意. 如果你已经用 *```deb```* 或者 *```rpm```* 包安装了 *```Grafana```*, 配置文件是在 *```/etc/grafana/grafana.ini```*. 这个路径在 *```Grafana init.d```* 脚本中通过 *```--config```* 参数指定.

### **使用环境变量**
配置文件里的所有配置项(下面列出的), 均可以使用下面的语法通过环境变量的方式来覆盖指定:

```GF_<SectionName>_<KeyName>```

括号之间的文本是段名. 所有均要使用大写, *```.```* 将被替换成 *```_```*. 例如下面的配置:
```
# default section
instance_name = ${HOSTNAME}

[security]
admin_user = admin

[auth.google]
client_secret = 0ldS3cretKey
```
你可以覆盖为:
```
export GF_DEFAULT_INSTANCE_NAME=my-instance
export GF_SECURITY_ADMIN_USER=true
export GF_AUTH_GOOGLE_CLIENT_SECRET=newS3cretKey
```

---

### **instance_name**
设置 *```Grafana```* 服务的实例名称.集群信息, 日志以及内部 *```metrics```* 会使用到.默认为: *```${HOSTNAME}```*, 可以使用环境变量 *```HOSTNAME```* 进行覆盖, 空的或者不存在的话, *```Grafana```* 会使用系统调用来获取机器名.

---

### **[paths]**
#### **data**
这是 *```Grafana```* 存储 *```sqlite3```* 数据库(如果有使用的话), 基于文件的 *```sessions```*(如果有使用的话), 以及其他数据的路径.这个路径通常会通过命令行在 *```init.d```* 脚本或者 *```systemd```* 服务文件中指定.
#### **temp_data_lifetime**
临时图片将在 *```data```* 目录下保留多久.默认为: *```24h```*.支持的修饰符:*```h```*(小时), *```m```*(分钟), 例如: *```168h```*, *```30m```*, *```10h30m```*. 使用 *```0```* 则永远不会清理临时文件.
#### **logs**
*```Grafana```* 日志目录.这个路径通常会通过命令行在 *```init.d```* 脚本或者 *```systemd```* 服务文件中指定. 可以通过配置文件或者默认的环境变量文件进行覆盖.
#### **plugins**
*```Grafana```* 自动扫描和搜索插件的目录.
#### **provisioning**
包含 [provisioning](https://grafana.com/docs/administration/provisioning) 配置文件的目录, *```Grafana```* 在启动时会应用. 当 *```json```* 文件发生变化时会重新加载 *```dashboards```*.

---

### **[server]**
#### **http_addr**
要绑定的 *```IP```* 地址. 为空的话将绑定所有网络接口.
#### **http_port**
要绑定的端口, 默认是: *```3000```*. 要使用 *```80```* 端口的话同时还需要在二进制文件中进行修改:
```
$ sudo setcap 'cap_net_bind_service=+ep' /usr/sbin/grafana-server
```
或使用以下方法将 *```80```* 端口重定向到 *```Grafana```* 端口:
```
$ sudo iptables -t nat -A PREROUTING -p tcp --dport 80 -j REDIRECT --to-port 3000
```
另外一种方式是使用 *```Nginx```* 或者 *```Apache```* 反向代理 *```Grafana```* 的请求.
#### **protocol**
*```http```*, *```https```* 或者 *```socket```*
>注意 *```Grafana 3.0```* 之前的版本对 [POODLE.](https://en.wikipedia.org/wiki/POODLE) 支持不好. 所以我们强烈建议升级到 *```3.x```* 或使用反向代理进行 *```ssl termination```*.
#### **socket**
当 *```protocol=socket```* 时的 *```socket```* 路径.请确保 *```Grafana```* 有相应的权限.
#### **domain**
改配置只是作为 *```root_url```* 的一部分配置(查看下面)使用. 这很重要, 如果你使用 *```GitHub```* 或者 *```Google OAuth```* 的话.
#### **enforce_domain**
如果主机头与域不匹配, 则重定向到正确的域. 防止 *```DNS Rebinding```* 攻击. 默认是 *```false```*.
#### **root_url**
这是用于从Web浏览器访问 *```Grafana```* 的完整URL. 这很重要, 如果你使用 *```GitHub```* 或者 *```Google OAuth```* 的话(使正确的回调URL).
#### **static_root_path**
前端文件目录(HTML, JS, 以及 CSS 文件). 默认为 *```public```*, 这就是为什么 *```Grafana```* 二进制文件需要在工作目录设置为安装路径的情况下执行的原因.
#### **enable_gzip**
设置为 *```true```* 开启 HTTP 压缩功能, 这可以提高传输速度和带宽利用率. 推荐开启. 由于兼容性, 默认为 *```false```*.
#### **cert_file**
证书目录(如果 *```protocol```* 设置为 *```https```*).
#### **cert_key**
证书秘钥目录(如果 *```protocol```* 设置为 *```https```*).
#### **router_logging**
设置为 *```true```* 可以记录 *```Grafana```* 的所有 HTTP 请求(不仅仅是错误). 这些作为 Info 级别的日志记录到 *```Grafana```* 日志文件.

---

### **[database]**
*```Grafana```* 需要一个数据库来存储用户和仪表板(以及其他东西). 默认情况下, 它配置为使用 *```sqlite3```*, 这是一个嵌入式数据库(包含在主 *```Grafana```* 二进制文件中).
#### **url**
使用以下URL或其他字段配置数据库如下:
```
mysql://user:secret@host:port/database
```
#### **type**
无论是 *```mysql```*, *```postgres```* 还是 *```sqlite3```*, 都可以.
#### **path**
仅适用于 *```sqlite3```* 数据库. 数据库存储在该目录下.
#### **host**
仅适用于 *```MySQL```* 或 *```Postgres```*. 包括 *```IP```* 或主机名和端口, 或者在 *```unix sockets```* 的情况下的路径. 例如, 与 *```Grafana```* 运行在同一台主机上的 *```MySQL```*: *```host = 127.0.0.1:3306```* 或者 *```unix sockets: host = /var/run/mysqld/mysqld.sock```*

---

#### **name**
*```Grafana```* 的数据库名称. 将它设置为 *```Grafana```* 或其他名称.
#### **password**
数据库用户的密码(不适用于 *```sqlite3```*). 如果密码包含 *```#```* 或者 *```;```*, 必须用三重引号包含它. 例如 *```"""#password;"""```*.
#### **ssl_mode**
对于 *```Postgres```*, 可以使用 *```disable```*, *```require```* 或者 *```verify-full```*. 对于 *```MySQL```*, 可以使用 *```true```*, *```false```*, 或者 *```skip-verify```*.
#### **ca_cert_path**
CA 证书目录. 在大部分 Linux 系统上, 证书在 *```/etc/ssl/certs```*.
#### **client_key_path**
客户端 key 目录.仅用于服务端需要客户端认证的情况.
#### **client_cert_path**
客户端 cert 目录.仅用于服务端需要客户端认证的情况.
#### **server_cert_name**
证书的通用名称字段, 用于 *```MySQL```* 或 *```Postgres```* 服务. 如果*```ssl_mode```* 设置为 *```skip-verify```*, 则不需要.
#### **max_idle_conn**
连接池中的最大空闲连接数.
#### **conn_max_lifetime**
设置可重用连接的最长时间, 默认值为14400(表示14400秒或4小时). 例如 *```MySQL```*, 应短于 *```wait_timeout```* 变量的设置值.
#### **log_queries**
设置为 *```true```* 以记录 *```sql```* 调用和执行时间.
#### **cache_mode**
仅适用于 *```sqlite3```*. [共享缓存](https://www.sqlite.org/sharedcache.html)设置用于连接数据库. (private, shared) 默认为 private.

---

### **[remote_cache]**
#### **type**
可以是 *```redis```*, *```memcached```* 或者 *```database```*, 默认是 *```database```*.
#### **connstr**
远程缓存连接字符串. 使用数据库时放空, 因为将使用主数据库. *```Redis```* 示例配置: *```addr=127.0.0.1:6379,pool_size=100,db=grafana```* *```Memcache```* 示例配置: *```127.0.0.1:11211```*.

---

### **[security]**
#### **admin_user**
*```Grafana```* 默认的管理员名称(拥有所有权限). 默认为 ```admin```.
#### **admin_password**
*```Grafana```* 默认的管理员密码. 第一次运行的时候设置一次. 默认为 ```admin```.
#### **login_remember_days**
提醒我登录/记住我 *```cookie```* 的持续天数.
#### **secret_key**
用于签署一些数据源设置, 如秘钥和密码. 如果不更新数据源设置, 然后重新编码, 则无法更改.
#### **disable_gravatar**
设置为 *```true```* 可禁用用户 *```Gravatar```* 个人资料图像修改功能. 默认为 *```false```*.
#### **data_source_proxy_whitelist**
定义要在数据源中使用的 *```IP```* /域名的白名单. 格式: *```ip_or_domain:port```*, 多个的话使用空格进行分隔.
#### **cookie_secure**
如果在 *```HTTPS```* 后面托管 *```Grafana```* 的话需要设置为 *```true```*. 默认为 *```false```*.
#### **cookie_samesite**
设置 *```SameSite```* cookie 属性并阻止浏览器发送此 cookie 以及跨站点请求. 主要目标是降低跨站信息泄露的风险. 还提供一些防止跨站请求伪造攻击 (CSRF) 的保护, [了解更多信息](https://www.owasp.org/index.php/SameSite). 有效的配置值是 *```lax```*, *```strict```* 和 *```none```*. 默认为 *```lax```*.

---

### **[users]**
#### **allow_sign_up**
设置为 *```false```* 以禁止用户注册/创建用户帐户. 默认为 *```false```*. 管理员用户仍然可以从[Grafana 管理员页面](https://grafana.com/docs/reference/admin)创建用户.
#### **allow_org_create**
设置为 *```false```* 以禁止用户创建新的组织. 默认为 *```false```*.
#### **auto_assign_org**
设置为 *```true```* 以自动将新用户添加到主组织(id 为1). 设置为 *```false```* 时, 将自动为新用户创建新的组织.
#### **auto_assign_org_id**
设置此值可自动将新用户添加到提供的组织. 这需要将 *```auto_assign_org```* 设置为 *```true```*. 请确保该组织事先存在.
#### **auto_assign_org_role**
为主组织分配新用户的角色(如果上述设置设置为 *```true```*). 默认为 *```Viewer```*, 其他可用选项是 *```Admin```* 和 *```Editor```*. 例如: *```auto_assign_org_role = Viewer```*
#### **viewers_can_edit**
*```Viewer```* 可以在浏览器中编辑/检查仪表板设置. 但不能保存仪表板. 默认为 *```false```*.
#### **editors_can_admin**
*```Editor```* 可以管理他们创建的仪表板, 文件夹和团队. 默认为 *```false```*.
#### **login_hint**
在登录页面上用于登录/用户名输入的占位符文本的内容.

---

### **[auth]**
*```Grafana```* 提供了许多方法来验证用户. 身份验证的文档已分成以下不同的页面.
- [验证概述](https://grafana.com/docs/auth/overview/)(匿名访问, 隐藏登录等)
- [谷歌认证](https://grafana.com/docs/auth/google/)(auth.google)
- [Github 认证](https://grafana.com/docs/auth/github/)(auth.github)
- [Gitlab 认证](https://grafana.com/docs/auth/gitlab/)(auth.gitlab)
- [通用认证](https://grafana.com/docs/auth/generic-oauth/)(auth.generic_oauth, okta2, auth0, bitbucket, azure)
- [Basic 认证](https://grafana.com/docs/auth/overview/)(auth.basic)
- [LDAP 认证](https://grafana.com/docs/auth/ldap/)(auth.ldap)
- [认证代理](https://grafana.com/docs/auth/auth-proxy/)(auth.proxy)

---

### **[session]**
#### **provider**
可用选项是 *```memory, file, mysql, postgres, memcache```* 或者  *```redis```*. 默认是 *```file```*.
#### **provider_config**
应根据您配置的 *```session```* 类型，以不同方式配置.
- file: *```session```* 文件路径, e.g. *```data/sessions```*
- mysql: *```go-sql-driver/mysql```* dsn 配置字串, e.g. *```user:password@tcp(127.0.0.1:3306)/database_name```*
- postgres: ex: *```user=a password=b host=localhost port=5432 dbname=c sslmode=verify-full```*
- memcache: ex: *```127.0.0.1:11211```*
- redis: ex: *```addr=127.0.0.1:6379,pool_size=100,prefix=grafana```*. 对于 unix socket, 例如: *```network=unix,addr=/var/run/redis/redis.sock,pool_size=100,db=grafana```*

Postgres 可用的 *```sslmode```* 配置有 *```disable, require, verify-ca ```* 和 *```verify-full```* (默认值).
#### **cookie_name**
*```Grafana session cookie```* 的名称.
#### **cookie_secure**
如果用 *```HTTPS```* 托管 *```Grafana```*, 则设置为 *```true```*. 默认为 *```false```*.
#### **session_life_time**
*```session```* 保留时长. 默认为 86400 (24小时).

---

### **[dataproxy]**
#### **logging**
启用数据代理日志记录, 默认为 *```false```*.
#### **timeout**
数据代理在超时之前需要等待的时间, 默认为30(秒).
#### **send_user_header**
如果启用且用户不是匿名用户, 数据代理会将带有用户名的 *```X-Grafana-User```* 作为 *```header```* 添加到请求中, 默认为 *```false```*.

---

### **[analytics]**
#### **reporting_enabled**
启用后, *```Grafana```* 将向 *```stats.grafana.org```* 发送匿名使用情况统计信息. 不会跟踪 IP 地址, 只是简单统计运行的实例, 版本, 仪表板和错误数. 这对我们非常有用, 所以请保持启用状态. 计数器每24小时发送一次. 默认为 *```true```*.
### **google_analytics_ua_id**
如果想通过 *```Google Analytics```* 分析跟踪 *```Grafana```* 的使用情况, 请在此处指定 *```Universal Analytics ID```. 该功能默认禁用.
### **check_for_updates**
设置为 *```false```* 以禁用通过 [https://grafana.com](https://grafana.com) 检查安装的插件的新版本以及到 *```Grafana GitHub```* 仓库检查 *```Grafana```* 的新版本. 版本信息会通过某些 UI 来通知. 不会做任何自动更新, 也不发送任何敏感信息. 每10分钟检查一次.

---

### **[dashboards]**
#### **versions_to_keep**
要保留的仪表板版本数(每个仪表板). 默认 20, 最小 1.


### **[dashboards.json]**
>这已被5.0+中的仪表板[配置](https://grafana.com/docs/administration/provisioning)所取代.
#### **enabled**
*```true```* 或 *```false```*. 默认禁用.
#### **path**
包含 json 仪表板的完整路径.


### **[smtp]**
邮件服务器设置.
#### **enabled**
默认为 *```false```*.
#### **host**
默认为 *```localhost:25```*
#### **user**
因为是 SMTP 身份验证, 所以默认为 *```empty```*.
#### **password**
因为是 SMTP 身份验证, 所以默认为 *```empty```*.
#### **cert_file**
证书文件路径, 所以默认为 *```empty```*.
#### **key_file**
秘钥文件路径, 所以默认为 *```empty```*.
#### **skip_verify**
验证 smtp 服务器的SSL? 默认为 *```false```*.
#### **from_address**
发送电子邮件时使用的地址, 默认为 *```admin@grafana.localhost```*
#### **from_name**
发送电子邮件时使用的名称, 默认为 *```Grafana```*
#### **ehlo_identity**
在 SMTP 对话框中用作 EHLO 的客户端标识的名称, 默认为 *```instance_name```*


### **[log]**
#### **mode**
"console", "file", "syslog" 之一. 默认为 "console" 和 "file" 使用空间分隔多个模式. e.g. "console file"
#### **level**
"debug", "info", "warn", "error", "critical" 之一. 默认为 "info".
#### **filters**
可选设置, 为指定的记录器设置不同的级别, Ex *```filters = sqlstore:debug```*


### **[metrics]**
#### **enabled**
启用 *```metrics```* 报告. 默认为 *```true```*, 可通过 HTTP API *```/metrics```* 获取.
#### **basic_auth_username**
可以为 *```metrics```* 节点设置 Basic 认证用户名.
#### **basic_auth_password**
可以为 *```metrics```* 节点设置 Basic 认证密码.
#### **interval_seconds**
将 *```metrics```* 发送到外部 TSDB 时的刷新/写入间隔. 默认为10秒.


### **[metrics.graphite]**
如果要将内部 *```Grafana metrics```* 发送到 *```Graphite```*, 请配置此部分.
#### **address**
格式 *```<Hostname 或 ip>:port```*
#### **prefix**
*```Graphite metrics```* 前缀. 默认是 *```prod.grafana.%(instance_name)s```*.


### **[snapshots]**
#### **external_enabled**
设置为 *```false```* 以禁用外部快照发布节点(默认为 *```true```*)
#### **external_snapshot_url**
设置要发布外部快照的 *```Grafana```* 实例(默认是[https://snapshots-origin.raintank.io/](https://snapshots-origin.raintank.io/)).
#### **external_snapshot_name**
设置外部快照按钮的名称. 默认是 *```Publish to snapshot.raintank.io```*
#### **snapshot_remove_expired**
启用以自动删除过期的快照.


### **[external_image_storage]**
这些选项控制图像应该如何公开, 以便它们可以在像 slack 这样的服务上共享.
#### **provider**
你可以选择(s3, webdav, gcs, azure_blob, local). 如果留空, *```Grafana```* 将忽略上传操作.

### **[external_image_storage.s3]**
#### **bucket**
S3存储 bucket 名称. 例如 *```grafana.snapshot```*
#### **region**
S3的 region 名称. 例如 'us-east-1', 'cn-north-1'等.
#### **path**
可选的额外的 bucket 内部路径, 适用于过期策略应用.
#### **bucket_url**
(为了向后兼容, 仅在未配置存储 bucket 或 region 时有效) S3 的 Bucket URL. AWS region 可以在 URL 指定, 也可以默认为 "us-east-1", 例如: *```- http://grafana.s3.amazonaws.com/ - https://grafana.s3-ap-southeast-2.amazonaws.com/```*
#### **access_key**
访问秘钥. 例如: AAAAAAAAAAAAAAAAAAAA

访问秘钥需要S3存储 bucket 的权限来操作 's3:PutObject' 和 's3:PutObjectAcl'.
#### **secret_key**
秘钥. 例如: AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA

### **[external_image_storage.webdav]**
#### **url**
*```Grafana```* 发送图片 PUT 请求的 URL.
#### **public_url**
可选参数. 用户通知中需要发送的 URL. 如果字符串包含 ${file} 序列, 它将被替换成上传的文件名. 不然, 文件名将附加到网址的路径部分, 保持查询字符串不变.
#### **username**
Basic 认证用户名.
#### **password**
Basic 认证密码.

### **[external_image_storage.gcs]**
#### **key_file**
与 Google 服务帐户关联的 JSON 密钥文件的路径, 用于进行身份验证和授权. 创建和下载服务帐户密钥在[https://console.developers.google.com/permissions/serviceaccounts](https://console.developers.google.com/permissions/serviceaccounts).

服务帐户应具有 "Storage Object Writer" 角色.

#### **bucket name**
Google 云端存储上的存储 bucket 名称.
#### **path**
可选的额外的 bucket 内部路径.

### **[external_image_storage.azure_blob]**
#### **account_name**
存储帐户名称
#### **account_key**
存储帐户密钥
#### **container_name**
以随机名称存储 "Blob" 图像的位置的容器名称. 需要预先创建 blob 容器. 仅支持公共容器.

### **[alerting]**
#### **enabled**
默认为 *```true```*. 设置为 *```false```* 可以禁用报警引擎并隐藏来自 UI 的报警信息.
#### **execute_alerts**
可以关闭报警执行规则.
#### **error_or_timeout**
>5.3及以上版本可用

新报警规则的默认设置. 默认为将错误和超时归为报警.(alerting, keep_state)
#### **nodata_or_nullvalues**
>5.3及以上版本可用

*```Grafana```* 如何处理报警中的 *```nodata```* 或 *```null```* 值的默认设置(alerting, no_data, keep_state, ok).
#### **concurrent_render_limit**
>5.3及以上版本可用

报警通知可以包含图片, 但是同时渲染太多图像会使服务器过载. 此限制将保护服务器免于渲染重载, 并确保快速发送通知. 默认值为 *```5```*
#### **evaluation_timeout_seconds**
默认的报警计算超时设置. 默认值为 *```30```*
#### **notification_timeout_seconds**
默认的报警通知超时设置. 默认值为 *```30```*
#### **max_attempts**
默认的发送报警通知的最大尝试次数设置. 默认值为 *```3```*
### **[panels]**
#### **disable_sanitize_html**
如果设置为 *```true```*, *```Grafana```* 将允许在文本面板中使用脚本标记. 不推荐使用, 因为存在 XSS 漏洞. 默认为 *```false```*. 此设置是在 *```Grafana v6.0```* 中引入的.
### **[plugins]**
#### **enable_alpha**
如果要测试尚未准备好进行常规使用的 alpha 插件, 请设置为 *```true```*.
