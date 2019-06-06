# **谷歌**

要启用 ```Google OAuth2```, 你必须注册 ```Google``` 应用程序. ```Google``` 将生成客户端 ID 和密钥供你使用.

## **创建 Google OAuth 密钥**

首先, 你需要创建 ```Google OAuth``` 客户端:

1.到[https://console.developers.google.com/apis/credentials](https://console.developers.google.com/apis/credentials).

2.点击 "```Create Credentials```" 按钮, 然后单击下拉菜单中的 "```OAuth Client ID```".

3.输入以下内容:

- 应用类型: ```Web Application```
  
- 名称: ```Grafana```

- 授权的 ```Javascript``` 源: ```https://grafana.mycompany.com/```

- 授权的重定向 URLs: ```https://grafana.mycompany.com/login/google```

- 将 ```https://grafana.mycompany.com``` 替换为 ```Grafana``` 实例的 URL.

4.点击 "Create"

5.从 "OAuth Client" 框中复制客户端 ID 和客户端密钥

## **在 Grafana 中启用 Google OAuth**

在 [Grafana 配置文件中](https://grafana.com/docs/v6.0/installation/configuration/#config-file-locations) 指定客户端 ID 和密钥. 例如:

```python
[auth.google]
enabled = true
client_id = CLIENT_ID
client_secret = CLIENT_SECRET
scopes = https://www.googleapis.com/auth/userinfo.profile https://www.googleapis.com/auth/userinfo.email
auth_url = https://accounts.google.com/o/oauth2/auth
token_url = https://accounts.google.com/o/oauth2/token
allowed_domains = mycompany.com mycompany.org
allow_sign_up = true
```

你需要为回调 URL 设置 ```[server]``` 正确的 ```root_url``` 选项才行. 例如, 如果在代理服务器后面提供 ```Grafana``` 服务.

重启 ```Grafana``` 后端. 现在应该可以在登录页面上看到 ```Google``` 登录按钮. 您现在可以登录或注册 ```Google``` 帐户. ```allowed_domains``` 是可选项, 域由空格分隔.

可以通过将 ```allow_sign_up``` 选项设置为 ```true``` 来允许用户通过 ```Google``` 身份验证进行注册. 当此选项设置为 ```true``` 时, 任何通过 ```Google``` 身份验证成功验证的用户都将自动注册.

