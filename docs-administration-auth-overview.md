## **用户认证概述**

*```Grafana```* 提供了多种认证用户的方式.有些集成认证还支持同步权限以及组织成员.

## **认证集成**
- [谷歌认证](https://grafana.com/docs/v6.0/auth/google/)
- [GitHub 认证](https://grafana.com/docs/v6.0/auth/github/)
- [GitLab 认证](https://grafana.com/docs/v6.0/auth/gitlab/)
- [通用认证](https://grafana.com/docs/v6.0/auth/generic-oauth/) (Okta2, BitBucket, Azure, OneLogin, Auth0)

## **LDAP集成**
- [LDAP 认证](https://grafana.com/docs/v6.0/auth/ldap/) (OpenLDAP, ActiveDirectory, etc)

## **认证代理**
- [认证代理](https://grafana.com/docs/v6.0/auth/auth-proxy/) 如果你想使用反向代理在 *```Grafana```* 外认证.

## **Grafana 认证**
*```Grafana```* 默认会开启内建的使用密码认证的用户认证系统.你可以通过启用匿名访问来禁用身份认证.你还可以隐藏登录表单, 只允许通过身份认证提供程序登录(如上所列).还有允许自行注册的选项.

### **登录和短期令牌**
> 以下内容适用于 *```Grafana```* 内建用户认证, LDAP(不使用认证代理)或者认证集成.

*```Grafana```* 使用短期令牌作为认证以经过校验的用户的机制. 对于活动的已认证的用户, 短期令牌每隔 *```token_rotation_interval_minutes```* 轮换一次.

对于活动的已认证的用户, 获取令牌后, 将延长 *```Grafana```* 记住用户登录的时间, 从"现在"到 *```login_maximum_inactive_lifetime_days ```*. 这意味着用户可以关闭浏览器并在 *```now + login_maximum_inactive_lifetime_days```* 之前返回时不用重新登录. 只要用户的登录时间少于 *```login_maximum_lifetime_days```*.

例如:
```
[auth]

# Login cookie name
login_cookie_name = grafana_session

# The lifetime (days) an authenticated user can be inactive before being required to login at next visit. Default is 7 days.
login_maximum_inactive_lifetime_days = 7

# The maximum lifetime (days) an authenticated user can be logged in since login time before being required to login. Default is 30 days.
login_maximum_lifetime_days = 30

# How often should auth tokens be rotated for authenticated users when being active. The default is each 10 minutes.
token_rotation_interval_minutes = 10
```

### **匿名认证**

你可以通过启用匿名访问来禁用身份认证, 详见下面的配置文件.

例如:
```
[auth.anonymous]
enabled = true

# Organization name that should be used for unauthenticated users
org_name = Main Org.

# Role for unauthenticated users, other valid values are `Editor` and `Admin`
org_role = Viewer
```

如果在 *```Grafana UI```* 中更改组织名称, 则需要更新此设置以匹配新的名称.

### **Basic 认证**
默认情况下启用基本身份认证, 并使用内建的 *```Grafana```* 用户密码身份认证系统和 *```LDAP```* 身份认证集成.

禁用 Basic 身份认证:
```
[auth.basic]
enabled = false
```

### **禁用登录表单**
你可以使用下面的配置来隐藏登陆表单.
```
[auth]
disable_login_form = true
```

### **自动 OAuth 认证登录**
设置为 *```true```* 以尝试自动使用 *```OAuth```* 登录, 跳过登录界面. 如果配置了多个 *```OAuth```* 提供程序, 则会忽略此设置. 默认为 *```false```*.
```
[auth]
oauth_auto_login = true
```

### **隐藏注销登录菜单**
设置为下面的选项为 *```true```* 以隐藏注销登录菜单链接. 使用认证代理时很有用.
```
[auth]
disable_signout_menu = true
```

### **退出后重定向的 URL**
从 *```Grafana```* 退出后将用户重定向到 URL. 例如, 可用于启用 *```oauth```* 提供程序的注销.
```
[auth]
signout_redirect_url =
```
