# **通用认证授权**

您可以使用 ```Grafana``` 的通用 ```oauth2``` 功能配置多种不同的 ```oauth2``` 身份认证服务. 你可以在下面找到使用 ```Okta```, ```BitBucket```, ```OneLogin``` 和 ```Azure``` 的示例.

这个回调 ```URL``` 必须与你在浏览器中用于访问 ```Grafana``` 的完整 ```HTTP``` 地址相匹配. 但路径前缀为 ```/login/generic_oauth```.

你需要为回调 ```URL``` 设置 ```[server]``` 的 ```root_url``` 选项才行. 例如, 如果在代理服务器后面提供 ```Grafana``` 服务.

配置示例:

```python
[auth.generic_oauth]
enabled = true
client_id = YOUR_APP_CLIENT_ID
client_secret = YOUR_APP_CLIENT_SECRET
scopes =
auth_url =
token_url =
api_url =
allowed_domains = mycompany.com mycompany.org
allow_sign_up = true
```

设置返回 [OpenID UserInfo](https://connect2id.com/products/server/docs/api/userinfo) 兼容信息的资源的 ```api_url```.

```Grafana``` 将尝试通过按以下顺序查询 ```OAuth``` 提供程序来确定用户的电子邮件地址, 直到找到电子邮件地址为止:

1.通过 ```OAuth id_token``` 参数中编码的 ```email``` 字段检查是否存在电子邮件地址.

2.检查 ```OAuth id_token``` 参数中编码的 ```attributes``` 映射中是否存在电子邮件地址. 默认情况下, ```Grafana``` 将使用 ```email:primary``` 键, 对属性映射执行查找, 然而, 这是可配置的, 可以使用 ```email_attribute_name``` 配置选项进行调整.

3.查询 ```OAuth``` 提供程序 ```API``` 的 ```/emails``` 节点(使用 ```api_url``` 配置) 并检查是否存在标记为主地址的电子邮件地址.

4.如果在步骤(1-3)中未找到电子邮件地址, 则将用户的电子邮件地址设置为空字符串.

## **使用 Okta 设置 OAuth2**

首先在 ```Okta``` 中将 ```Grafana``` 设置为 ```OpenId``` 客户端 "```webapplication```". 然后将 ```Base URI``` 设置为 ```https://<grafana domain>/``` 以及将登录重定向 URIs 设置为 ```https://<grafana domain>/login/generic_oauth```. 

最后设置这样的通用 ```oauth``` 模块:

```python
[auth.generic_oauth]
name = Okta
enabled = true
scopes = openid profile email
client_id = <okta application Client ID>
client_secret = <okta application Client Secret>
auth_url = https://<okta domain>/oauth2/v1/authorize
token_url = https://<okta domain>/oauth2/v1/token
api_url = https://<okta domain>/oauth2/v1/userinfo
```

## **使用 Bitbucket 设置 OAuth2**

```python
[auth.generic_oauth]
name = BitBucket
enabled = true
allow_sign_up = true
client_id = <client id>
client_secret = <client secret>
scopes = account email
auth_url = https://bitbucket.org/site/oauth2/authorize
token_url = https://bitbucket.org/site/oauth2/access_token
api_url = https://api.bitbucket.org/2.0/user
team_ids =
allowed_organizations =
```

## **使用 OneLogin 设置 OAuth2**

1.使用以下设置创建新的自定义连接器:

- 名称: ```Grafana```
  
- 登录方法: ```OpenID Connect```

- 重定向 URI: ```https://<grafana domain>/login/generic_oauth```

- 签名算法: ```RS256```

- 登录 URL: ```https://<grafana domain>/login/generic_oauth```

然后:

2.将应用程序添加到 ```Grafana``` 连接器

- 显示名称: ```Grafana```

然后:

3.在 ```Grafana App``` 详细信息页面上的 ```SSO``` 选项卡下, 你将找到客户端ID和客户端密钥. 您的 ```OneLogin``` 域将与您用于访问 ```OneLogin``` 的 ```URL``` 匹配.

```Grafana``` 配置如下:

```python
[auth.generic_oauth]
name = OneLogin
enabled = true
allow_sign_up = true
client_id = <client id>
client_secret = <client secret>
scopes = openid email name
auth_url = https://<onelogin domain>.onelogin.com/oidc/auth
token_url = https://<onelogin domain>.onelogin.com/oidc/token
api_url = https://<onelogin domain>.onelogin.com/oidc/me
team_ids =
allowed_organizations =
```

## **使用 Auth0 设置 OAuth2**

1.在 ```Auth0``` 中创建一个新客户端

- 名称: ```Grafana```

- 类型: ```Regular Web Application```

2.转到 "```Settings```" 选项卡并进行设置

- 允许的回调 URLs: ```https://<grafana domain>/login/generic_oauth```

3.单击保存更改, 然后使用页面顶部的值配置 ```Grafana```:

```python
[auth.generic_oauth]
enabled = true
allow_sign_up = true
team_ids =
allowed_organizations =
name = Auth0
client_id = <client id>
client_secret = <client secret>
scopes = openid profile email
auth_url = https://<domain>/authorize
token_url = https://<domain>/oauth/token
api_url = https://<domain>/userinfo
```

## **使用 Azure Active Directory 设置 OAuth2**

1.登录 ```portal.azure.com```, 然后单击侧边菜单中的 "```Azure Active Directory```", 然后单击 "```Properties```" 子菜单项.

2.复制 "```Directory ID```", 这是稍后设置 URL 所必需的.

3.单击 "```App Registrations```" 并添加新的应用程序注册:

- 名称: ```Grafana```

- 应用类型: ```Web app/API```

- 登录 URL: ```https://<grafana domain>/login/generic_oauth```
  
4.单击新应用程序的名称以打开应用程序详细信息页面

5.记下 "```Application ID```", 这将是 ```OAuth``` 客户端 ID.

6.点击 "```Settings```", 然后点击 "```Keys```" 在密码下添加新条目

- Key 说明: ```Grafana OAuth```

- 期限: ```Never Expires```

7.单击保存, 然后复制 key 值, 这将是 ```OAuth``` 客户端密钥.

8.配置 ```Grafana``` 如下:

```python
[auth.generic_oauth]
name = Azure AD
enabled = true
allow_sign_up = true
client_id = <application id>
client_secret = <key value>
scopes = openid email name
auth_url = https://login.microsoftonline.com/<directory id>/oauth2/authorize
token_url = https://login.microsoftonline.com/<directory id>/oauth2/token
api_url =
team_ids =
allowed_organizations =
```

>注意: 确保 ```Grafana``` 中的 ```root_url``` 设置在 ```Azure``` 应用程序回复URL(应用程序 -> 设置 -> 回复 URL)中非常重要.

## **使用 Centrify 设置 OAuth2**

1.在 ```Centrify``` 仪表板中创建新的自定义 ```OpenID Connect``` 应用程序配置.

2.创建一个可记忆的唯一应用程序 ID, 例如 "```Grafana```", "```grafana_aws```" 等.

3.放入其他基本配置(名称, 描述, 徽标, 类别).

4.在 "信任" 选项卡上, 生成一个长密码并将其放入 ```OpenID Connect Client Secret``` 字段.

5.将 URL 放到 ```Grafana``` 实例的首页, 进入 "```Resource Application URL```" 字段.

6.添加授权的重定向 URI, 如 [https://your-grafana-server/login/generic_oauth](https://your-grafana-server/login/generic_oauth).

7.像任何其他 ```Centrify``` 应用程序一样设置权限, 策略等.

8.```Grafana``` 配置如下:

```python
[auth.generic_oauth]
name = Centrify
enabled = true
allow_sign_up = true
client_id = <OpenID Connect Client ID from Centrify>
client_secret = <your generated OpenID Connect Client Sercret"
scopes = openid email name
auth_url = https://<your domain>.my.centrify.com/OAuth2/Authorize/<Application ID>
token_url = https://<your domain>.my.centrify.com/OAuth2/Token/<Application ID>
```

## **使用非常规的提供商设置 OAuth2**

>仅适用于 ```Grafana v6.0``` 及更高版本

某些 ```OAuth2``` 提供程序可能不支持通过基本身份验证 HTTP 头传递的 ```client_id``` 和 ```client_secret```, 这会导致 ```invalid_client``` 错误. 允许 ```Grafana``` 通过这些类型的提供程序进行身份验证, 必须通过 POST body 发送客户端标识符, 可以通过以下设置启用:

```python
[auth.generic_oauth]
send_client_credentials_via_post = true
```
