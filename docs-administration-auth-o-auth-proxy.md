## **认证代理认证**

*```Grafana```* 可以配置成通过反向代理的方式来认证.流行的 Web 服务器有很多的插件式身份验证模块, 任何一个都可以和 AuthProxy 功能配合使用. 下面我们详细介绍一下具体配置.
```
[auth.proxy]
# Defaults to false, but set to true to enable this feature
enabled = true
# HTTP Header name that will contain the username or email
header_name = X-WEBAUTH-USER
# HTTP Header property, defaults to `username` but can also be `email`
header_property = username
# Set to `true` to enable auto sign up of users who do not exist in Grafana DB. Defaults to `true`.
auto_sign_up = true
# If combined with Grafana LDAP integration define sync interval
ldap_sync_ttl = 60
# Limit where auth proxy requests come from by configuring a list of IP addresses.
# This can be used to prevent users spoofing the X-WEBAUTH-USER header.
# Example `whitelist = 192.168.1.1, 192.168.1.0/24, 2001::23, 2001::0/120`
whitelist =
# Optionally define more headers to sync other user attributes
# Example `headers = Name:X-WEBAUTH-NAME Email:X-WEBAUTH-EMAIL`
headers =
```

## **通过 curl 与 Grafana 的 AuthProxy 交互**
```
curl -H "X-WEBAUTH-USER: admin"  http://localhost:3000/api/users
[
    {
        "id":1,
        "name":"",
        "login":"admin",
        "email":"admin@localhost",
        "isAdmin":true
    }
]
```

然后我们可以发送第二个请求到 ```/api/user``` 获取登录用户的详细信息.我们将使用该请求来说明 ```Grafana``` 如何自动将我们指定的新用户添加到系统中. 这里, 我们创建了一个名为 "anthony" 的新用户.
```
curl -H "X-WEBAUTH-USER: anthony" http://localhost:3000/api/user
{
    "email":"anthony",
    "name":"",
    "login":"anthony",
    "theme":"",
    "orgId":1,
    "isGrafanaAdmin":false
}
```

## **让 Apache 的 auth 与 Grafana 的 AuthProxy 一起工作**
我将演示如何使用 ```Apache``` 来验证用户身份. 在这个例子中, 我们使用 ```BasicAuth``` 和 ```Apache``` 的基于文本文件的身份验证处理程序, i.e. ```htpasswd``` 文件. 但是, 可以使用任何可用的 ```Apache``` 身份验证功能.

### **Apache BasicAuth**
在这个例子中, 我们使用 ```Apache``` 作为 ```Grafana``` 的反向代理. ```Apache``` 在将请求转发给 ```Grafana``` 后端服务之前处理用户身份验证.

```Apache``` 配置
```
    <VirtualHost *:80>
        ServerAdmin webmaster@authproxy
        ServerName authproxy
        ErrorLog "logs/authproxy-error_log"
        CustomLog "logs/authproxy-access_log" common

        <Proxy *>
            AuthType Basic
            AuthName GrafanaAuthProxy
            AuthBasicProvider file
            AuthUserFile /etc/apache2/grafana_htpasswd
            Require valid-user

            RewriteEngine On
            RewriteRule .* - [E=PROXY_USER:%{LA-U:REMOTE_USER},NS]
            RequestHeader set X-WEBAUTH-USER "%{PROXY_USER}e"
        </Proxy>

        RequestHeader unset Authorization

        ProxyRequests Off
        ProxyPass / http://localhost:3000/
        ProxyPassReverse / http://localhost:3000/
    </VirtualHost>
```

* 前4行是标准的虚拟主机配置, 因此我们不做具体分析.
* 我们使用 ```<proxy>``` 配置块将我们的身份验证规则应用于每个代理请求.这些规则包括所需的基本身份验证, 其中 ```user:password``` 凭据存储在 ```/etc/ apache2/grafana_htpasswd``` 文件中. 可以使用 ```htpasswd``` 命令创建此文件.
  * 接下来的配置比较棘手. 我们使用 ```Apache``` 重写引擎来创建 ```X-WEBAUTH-USER``` 头, 填充经过身份验证的用户.
    * RewriteRule .* - [E=PROXY_USER:%{LA-U:REMOTE_USER}, NS]:这行配置有点神奇. 它能做什么呢, 是为每个请求使用 ```rewriteEngines``` 预见(LA-U)功能来确定处理请求后 ```REMOTE_USER``` 变量将被设置为什么. 然后将结果分配给变量 ```PROXY_USER```. 这是必要的, 因为 ```REMOTE_USER``` 变量无法用于 ```RequestHeader``` 函数.
    * RequestHeader set X-WEBAUTH-USER "%{PROXY_USER}e": 使用经过身份验证的用户名现在存储在 ```PROXY_USER``` 变量中, 我们创建一个新的 HTTP 请求头, 该头将发送到包含用户名的后端 ```Grafana```.
* **RequestHeader unset Authorization** 在将 HTTP 请求转发到 ```Grafana``` 之前, 从 HTTP 请求中移除 ```Authorization``` 头. 这可确保 ```Grafana``` 不会尝试使用这些凭据对用户进行身份验证(```BasicAuth``` 是 ```Grafana``` 中受支持的身份验证处理程序).


## **完全使用 Docker**
对于这个例子, 我们使用官方的 ```Grafana``` 镜像[Docker Hub](https://hub.docker.com/r/grafana/grafana/).
- 使用以下内容创建文件 ```grafana.ini```
```
[users]
allow_sign_up = false
auto_assign_org = true
auto_assign_org_role = Editor

[auth.proxy]
enabled = true
header_name = X-WEBAUTH-USER
header_property = username
auto_sign_up = true
```

启动 ```Grafana``` 容器, 使用我们的自定义 ```grafana.ini``` 来替换 ```/etc/grafana/grafana.ini```. 我们开放这个容器的任何端口, 因为它只能通过我们的 ```Apache``` 容器连接.
```
docker run -i -v $(pwd)/grafana.ini:/etc/grafana/grafana.ini --name grafana grafana/grafana
```

### **Apache 容器**
对于这个例子, 我们使用官方的 ```Apache``` 镜像[Docker Hub](https://hub.docker.com/_/httpd/).

- 使用以下内容创建文件 ```httpd.conf```
```
ServerRoot "/usr/local/apache2"
Listen 80
LoadModule authn_file_module modules/mod_authn_file.so
LoadModule authn_core_module modules/mod_authn_core.so
LoadModule authz_host_module modules/mod_authz_host.so
LoadModule authz_user_module modules/mod_authz_user.so
LoadModule authz_core_module modules/mod_authz_core.so
LoadModule auth_basic_module modules/mod_auth_basic.so
LoadModule log_config_module modules/mod_log_config.so
LoadModule env_module modules/mod_env.so
LoadModule headers_module modules/mod_headers.so
LoadModule unixd_module modules/mod_unixd.so
LoadModule rewrite_module modules/mod_rewrite.so
LoadModule proxy_module modules/mod_proxy.so
LoadModule proxy_http_module modules/mod_proxy_http.so
<IfModule unixd_module>
User daemon
Group daemon
</IfModule>
ServerAdmin you@example.com
<Directory />
    AllowOverride none
    Require all denied
</Directory>
DocumentRoot "/usr/local/apache2/htdocs"
ErrorLog /proc/self/fd/2
LogLevel error
<IfModule log_config_module>
    LogFormat "%h %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\"" combined
    LogFormat "%h %l %u %t \"%r\" %>s %b" common
    <IfModule logio_module>
    LogFormat "%h %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\" %I %O" combinedio
    </IfModule>
    CustomLog /proc/self/fd/1 common
</IfModule>
<Proxy *>
    AuthType Basic
    AuthName GrafanaAuthProxy
    AuthBasicProvider file
    AuthUserFile /tmp/htpasswd
    Require valid-user
    RewriteEngine On
    RewriteRule .* - [E=PROXY_USER:%{LA-U:REMOTE_USER},NS]
    RequestHeader set X-WEBAUTH-USER "%{PROXY_USER}e"
</Proxy>
RequestHeader unset Authorization
ProxyRequests Off
ProxyPass / http://grafana:3000/
ProxyPassReverse / http://grafana:3000/

```
- 创建一个 ```htpasswd``` 文件. 我们使用密码 **password** 创建一个新的用户 **anthony**
```
htpasswd -bc htpasswd anthony password
```

- 使用我们的自定义 ```httpd.conf``` 和 ```htpasswd``` 文件启动 ```httpd``` 容器. 容器监听80端口, 我们创建了一个指向 ```grafana``` 容器的链接, 以便这个容器可以将主机名 ```grafana``` 解析为 ```grafana``` 容器的 ```ip``` 地址.
```
docker run -i -p 80:80 --link grafana:grafana -v $(pwd)/httpd.conf:/usr/local/apache2/conf/httpd.conf -v $(pwd)/htpasswd:/tmp/htpasswd httpd:2.4
```  


### **使用 Grafana**
我们的 ```Grafana``` 和 ```Apache``` 容器运行起来了, 你现在可以连接到[http://localhost/](http://localhost/)并使用我们在 ```htpasswd``` 文件中创建的用户名/密码登录.
