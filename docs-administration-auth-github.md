# **GitHub OAuth2 认证**

要启用 ```GitHub OAuth2```, 必须使用 ```GitHub``` 注册应用程序. ```GitHub``` 将生成一个客户端ID和密钥供你使用.

## **配置 GitHub OAuth 应用**

你需要创建一个 ```GitHub OAuth``` 应用程序(你可以在 ```GitHub``` 设置页面下找到它). 创建应用程序时, 您需要指定回调 URL. 如下指定回调:

```python
http://<my_grafana_server_name_or_ip>:<grafana_server_port>/login/github
```

此回调网址必须与你在浏览器中用于访问 ```Grafana``` 的完整 HTTP 地址相匹配, 但是使用 ```/login/github``` 为路径前缀. 创建 ```GitHub OAuth``` 应用程序后, 您将获得客户端 ID 和客户端密钥. 在 ```Grafana``` 配置文件中指定. 例如:

### **在 Grafana 中启用 GitHub**

```python
[auth.github]
enabled = true
allow_sign_up = true
client_id = YOUR_GITHUB_APP_CLIENT_ID
client_secret = YOUR_GITHUB_APP_CLIENT_SECRET
scopes = user:email,read:org
auth_url = https://github.com/login/oauth/authorize
token_url = https://github.com/login/oauth/access_token
api_url = https://api.github.com/user
team_ids =
allowed_organizations =
```

你需要为回调 URL 设置 ```[server]``` 正确的 ```root_url``` 选项才行. 例如, 如果在代理服务器后面提供 ```Grafana``` 服务.

重启 ```Grafana``` 后端. 现在应该在登录页面上看到 ```GitHub``` 登录按钮. 你现在可以登录或注册你的 ```GitHub``` 帐户.

可以通过将 ```allow_sign_up``` 选项设置为 ```true``` 来允许用户通过 ```GitHub``` 身份验证进行注册. 当此选项设置为 ```true``` 时, 任何通过 ```GitHub``` 身份验证成功验证的用户都将自动注册.

### **team_ids**

需要 ```GitHub``` 上至少一个给定的激活的团队的成员关系. 如果经过身份验证的用户不是至少一个团队的成员, 他们将无法注册或验证你的 ```Grafana``` 实例. 例如:

```python
[auth.github]
enabled = true
client_id = YOUR_GITHUB_APP_CLIENT_ID
client_secret = YOUR_GITHUB_APP_CLIENT_SECRET
scopes = user:email,read:org
team_ids = 150,300
auth_url = https://github.com/login/oauth/authorize
token_url = https://github.com/login/oauth/access_token
api_url = https://api.github.com/user
allow_sign_up = true
```

### **allowed_organizations**

需要 ```GitHub``` 上至少一个给定的激活的组织的成员关系. 如果经过身份验证的用户不是至少一个组织的成员, 他们将无法注册或验证你的 ```Grafana``` 实例. 例如:

```python
[auth.github]
enabled = true
client_id = YOUR_GITHUB_APP_CLIENT_ID
client_secret = YOUR_GITHUB_APP_CLIENT_SECRET
scopes = user:email,read:org
auth_url = https://github.com/login/oauth/authorize
token_url = https://github.com/login/oauth/access_token
api_url = https://api.github.com/user
allow_sign_up = true
# space-delimited organization names
allowed_organizations = github google
```

