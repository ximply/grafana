## **LDAP 认证**
```Grafana``` 中的 ```LDAP``` 集成允许 ```Grafana``` 用户使用 ```LDAP``` 凭据登录. 您还可以指定 ```LDAP``` 组成员身份与 ```Grafana``` 组织用户角色之间的映射.

### **支持的 LDAP 服务器**
```Grafana``` 使用第三方[LDAP库](https://github.com/go-ldap/ldap), 支持基本的 ```LDAP v3``` 功能. 这意味着能够使用任何兼容的 ```LDAP v3```服务器配置 ```LDAP``` 集成, 例如 [OpenLDAP](https://grafana.com/docs/auth/ldap/#openldap) 或 [Active Directory](https://grafana.com/docs/auth/ldap/#active-directory) 等 [其他的](https://en.wikipedia.org/wiki/Directory_service#LDAP_implementations).

### **启用 LDAP**
要集成 ```LDAP```, 首先需要在[主配置文件中](https://grafana.com/docs/installation/configuration/)启用 ```LDAP```, 以及指定 ```LDAP``` 特定配置文件的路径 ```(默认是: /etc/grafana/ldap.toml)```.
```
[auth.ldap]
# Set to `true` to enable LDAP integration (default: `false`)
enabled = true

# Path to the LDAP specific configuration file (default: `/etc/grafana/ldap.toml`)
config_file = /etc/grafana/ldap.toml

# Allow sign up should almost always be true (default) to allow new Grafana users to be created (if ldap authentication is ok). If set to
# false only pre-existing Grafana users will be able to login (if ldap authentication is ok).
allow_sign_up = true
```

### **Grafana LDAP 配置**
根据您使用的 ```LDAP``` 服务器以及配置方式, ```Grafana LDAP``` 配置可能会有所不同. 详见[配置例子](https://grafana.com/docs/auth/ldap/#configuration-examples).

**LDAP 特定配置文件(ldap.toml)示例:**
```
[[servers]]
# Ldap server host (specify multiple hosts space separated)
host = "127.0.0.1"
# Default port is 389 or 636 if use_ssl = true
port = 389
# Set to true if ldap server supports TLS
use_ssl = false
# Set to true if connect ldap server with STARTTLS pattern (create connection in insecure, then upgrade to secure connection with TLS)
start_tls = false
# set to true if you want to skip ssl cert validation
ssl_skip_verify = false
# set to the path to your root CA certificate or leave unset to use system defaults
# root_ca_cert = "/path/to/certificate.crt"
# Authentication against LDAP servers requiring client certificates
# client_cert = "/path/to/client.crt"
# client_key = "/path/to/client.key"

# Search user bind dn
bind_dn = "cn=admin,dc=grafana,dc=org"
# Search user bind password
# If the password contains # or ; you have to wrap it with triple quotes. Ex """#password;"""
bind_password = 'grafana'

# User search filter, for example "(cn=%s)" or "(sAMAccountName=%s)" or "(uid=%s)"
# Allow login from email or username, example "(|(sAMAccountName=%s)(userPrincipalName=%s))"
search_filter = "(cn=%s)"

# An array of base dns to search through
search_base_dns = ["dc=grafana,dc=org"]

# group_search_filter = "(&(objectClass=posixGroup)(memberUid=%s))"
# group_search_filter_user_attribute = "distinguishedName"
# group_search_base_dns = ["ou=groups,dc=grafana,dc=org"]

# Specify names of the ldap attributes your ldap uses
[servers.attributes]
name = "givenName"
surname = "sn"
username = "cn"
member_of = "memberOf"
email =  "email"
```

### **绑定**
**绑定 & 绑定密码**

默认情况下, 要求绑定DN和绑定密码. 这应该是可以执行 ```LDAP``` 搜索的只读用户. 找到用户DN后, 将使用用户提供的用户名和密码执行第二次绑定(在正常的```Grafana```登录页面中).
```
bind_dn = "cn=admin,dc=grafana,dc=org"
bind_password = "grafana"
```

**单绑定示例**

如果你可以提供与所有可能的用户匹配的单个绑定表达式, 您可以跳过第二个绑定并直接绑定用户DN. 这允许你不必在配置文件中指定 ```bind_password```.
```
bind_dn = "cn=%s,o=users,dc=grafana,dc=org"
```
这种情况下你不必设置 ```bind_password```, 取而代之的是在某个地方包含有 ```%``` 的 ```bind_dn``` 值. 这将替换为 ```Grafana``` 登录页面中输入的用户名. 仍需要搜索过滤器和搜索库设置来执行 ```LDAP``` 搜索以检索其他 ```LDAP``` 信息(例如 ```LDAP``` 组和邮箱).

### **POSIX 结构**
如果你的 ```LDAP``` 服务器不支持 ```memberOf``` 属性，请添加以下选项:
```
## Group search filter, to retrieve the groups of which the user is a member (only set if memberOf attribute is not available)
group_search_filter = "(&(objectClass=posixGroup)(memberUid=%s))"
## An array of the base DNs to search through for groups. Typically uses ou=groups
group_search_base_dns = ["ou=groups,dc=grafana,dc=org"]
## the %s in the search filter will be replaced with the attribute defined below
group_search_filter_user_attribute = "uid"
```
还要在 ```[servers.attributes]``` 段中设置 ```member_of = "dn"```.


### **组映射**
在 ```[[servers.group_mappings]]``` 中你可以将 ```LDAP``` 组映射到 ```Grafana``` 组织和角色. 每次用户登录时都会同步这些内容, ```LDAP``` 是权威来源. 因此, 如果更改用户在 ```Grafana``` 组织中的角色, 用户页面, 此更改将在用户下次登录时重置. 如果更改用户的 ```LDAP``` 组, 更改将在用户下次登录时生效.

```LDAP``` 用户匹配的第一个组映射将用于同步. 如果您具有适合多个映射的 ```LDAP``` 用户, 将使用 ```TOML``` 配置中最顶层的映射.

**LDAP 特定配置文件(ldap.toml)示例:**
```
[[servers]]
# other settings omitted for clarity

[[servers.group_mappings]]
group_dn = "cn=superadmins,dc=grafana,dc=org"
org_role = "Admin"
grafana_admin = true # Available in Grafana v5.3 and above

[[servers.group_mappings]]
group_dn = "cn=admins,dc=grafana,dc=org"
org_role = "Admin"

[[servers.group_mappings]]
group_dn = "cn=users,dc=grafana,dc=org"
org_role = "Editor"

[[servers.group_mappings]]
group_dn = "*"
org_role = "Viewer"
```

设置 | 必要 | 说明 | 默认 |
:- | :-: | :- | :- |
```group_dn``` | 是 | ```LDAP``` 组的 ```LDAP``` 专有名称(DN). <br> 如果要匹配所有(或没有```LDAP```组)则可以使用通配符("```*```") |  |
```org_role``` | 是 | 为 ```group_dn``` 的用户分配组织角色"```Admin```", "```Editor```" 或 "```Viewer```" |  |
```group_dn``` | 否 | ```Grafana``` 组织数据库ID. <br>允许设置多个 ```group_dn``` 到不同 ```org_id ``` 的同一个 ```org_role ``` | ```1``` (默认的 org id) |
```org_role``` | 否 | 为 ```true``` 时, 设置 ```group_dn``` 为 ```Grafana admin``` 用户. <br>```Grafana``` 服务器管理员具有对所有组织和用户的管理员访问权限.<br>可在 ```Grafana v5.3``` 及更高版本中使用 | ```false``` |

### **嵌套/递归组成员关系**
具有嵌套/递归组成员身份的用户必须具有支持的 ```LDAP``` 服务器 ```LDAP_MATCHING_RULE_IN_CHAIN``` 以及配置 ```group_search_filter``` 以一种方式返回提交的用户名是其成员的组.

**Active Directory示例:**

```Active Directory``` 组存储成员的专有名称(DN), 因此您的过滤器只需要根据提交的用户名知道用户的DN. 通过将过滤器与 ```LDAP OR``` 运算符组合, 可以搜索多个DN模板. 例如:
```
group_search_filter = "(member:1.2.840.113556.1.4.1941:=CN=%s,[user container/OU])"
group_search_filter = "(|(member:1.2.840.113556.1.4.1941:=CN=%s,[user container/OU])(member:1.2.840.113556.1.4.1941:=CN=%s,[another user container/OU]))"
group_search_filter_user_attribute = "cn"
```

有关AD搜索的详细信息, 请参阅[Microsoft搜索过滤语法](https://docs.microsoft.com/en-us/windows/desktop/adsi/search-filter-syntax)文档.

用于故障排除, 通过改变 ```[servers.attributes]``` 中的 ```member_of``` 为 "```dn```", [开启调试](https://grafana.com/docs/auth/ldap/#troubleshooting)会显示更准确的组成员关系.

### **配置示例**
#### **OpenLDAP**
[OpenLDAP](http://www.openldap.org/)是开源的目录服务.

**LDAP 特定配置文件(ldap.toml)示例:**
```
[[servers]]
host = "127.0.0.1"
port = 389
use_ssl = false
start_tls = false
ssl_skip_verify = false
bind_dn = "cn=admin,dc=grafana,dc=org"
bind_password = 'grafana'
search_filter = "(cn=%s)"
search_base_dns = ["dc=grafana,dc=org"]

[servers.attributes]
name = "givenName"
surname = "sn"
username = "cn"
member_of = "memberOf"
email =  "email"

# [[servers.group_mappings]] omitted for clarity
```

#### **多 LDAP 服务器**
```Grafana``` 确实支持从多个 ```LDAP``` 服务器接收信息.
**LDAP 特定配置文件(ldap.toml)示例:**
```
# --- First LDAP Server ---

[[servers]]
host = "10.0.0.1"
port = 389
use_ssl = false
start_tls = false
ssl_skip_verify = false
bind_dn = "cn=admin,dc=grafana,dc=org"
bind_password = 'grafana'
search_filter = "(cn=%s)"
search_base_dns = ["ou=users,dc=grafana,dc=org"]

[servers.attributes]
name = "givenName"
surname = "sn"
username = "cn"
member_of = "memberOf"
email =  "email"

[[servers.group_mappings]]
group_dn = "cn=admins,ou=groups,dc=grafana,dc=org"
org_role = "Admin"
grafana_admin = true

# --- Second LDAP Server ---

[[servers]]
host = "10.0.0.2"
port = 389
use_ssl = false
start_tls = false
ssl_skip_verify = false

bind_dn = "cn=admin,dc=grafana,dc=org"
bind_password = 'grafana'
search_filter = "(cn=%s)"
search_base_dns = ["ou=users,dc=grafana,dc=org"]

[servers.attributes]
name = "givenName"
surname = "sn"
username = "cn"
member_of = "memberOf"
email =  "email"

[[servers.group_mappings]]
group_dn = "cn=editors,ou=groups,dc=grafana,dc=org"
org_role = "Editor"

[[servers.group_mappings]]
group_dn = "*"
org_role = "Viewer"
```

### **活动目录**
[Active Directory](https://technet.microsoft.com/en-us/library/hh831484(v=ws.11).aspx)是 ```Windows``` 环境中常用的目录服务.

假设以下 ```Active Directory``` 服务器设置:
- IP地址: ```10.0.0.1```
- 域名: ```CORP```
- DNS名称: ```corp.local```
  
**LDAP 特定配置文件(ldap.toml)示例:**
```
[[servers]]
host = "10.0.0.1"
port = 3269
use_ssl = true
start_tls = false
ssl_skip_verify = true
bind_dn = "CORP\\%s"
search_filter = "(sAMAccountName=%s)"
search_base_dns = ["dc=corp,dc=local"]

[servers.attributes]
name = "givenName"
surname = "sn"
username = "sAMAccountName"
member_of = "memberOf"
email =  "mail"

# [[servers.group_mappings]] omitted for clarity
```

**端口要求**

在上面的示例中, 启用了 ```SSL``` 并且已配置加密端口. 如果您的 ```Active Directory``` 不支持 ```SSL```, 请更改 ```enable_ssl = false``` 和 ```port = 389```. 请检查 ```Active Directory``` 配置和文档以查找正确的设置. 有关```Active Directory``` 和端口要求的详细信息, 请[参阅](https://technet.microsoft.com/en-us/library/dd772723(v=ws.10)).

### **故障排除**
要进行故障排除并获取更多日志信息, 要在[主配置文件中](https://grafana.com/docs/installation/configuration/)启用 ```ldap debug logging``` 配置.
```
[log]
filters = ldap:debug
```
