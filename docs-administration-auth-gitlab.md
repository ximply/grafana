# **GitLab OAuth2 认证**

要启用 ```GitLab OAuth2```, 必须使用 ```GitLab``` 注册应用程序. ```GitLab``` 将生成一个客户端ID和密钥供你使用.

## **创建 GitLab OAuth 密钥**

你需要[创建一个 GitLab OAuth 应用](https://docs.gitlab.com/ce/integration/oauth_provider.html). 选择描述性 ***Name***, 并使用以下 ***Redirect URI***:

```python
https://grafana.example.com/login/gitlab
```

其中 ```https://grafana.example.com``` 是你用来连接 ```Grafana``` 的 URL. 如果您不使用 HTTPS 或使用其他端口, 请根据需要进行调整; 例如, 如果你访问 ```Grafana``` 的地址是 ```http://203.0.113.31:3000```, 你应该使用:

```python
http://203.0.113.31:3000/login/gitlab
```

最后, 选择 ***api*** 作为 ***Scope*** 然后提交表单. 请注意，如果你不打算使用 ```GitLab``` 组进行授权(即, 不设置 ```allowed_groups```, 如下), 你可以选择 ***read_user*** 代替 ***api*** 作为 ***Scope***, 从而提供对```GitLab API``` 的更严格的访问权限.

你将返回信息中获取到 ***Application Id*** 和 ***Secret***; 我们称为 ```GITLAB_APPLICATION_ID``` 和 ```GITLAB_SECRET``` 分别用于本节的其余部分.

## **在 Grafana 中启用 GitLab**

将以下内容添加到 ```Grafana``` 配置文件中以启用 ```GitLab``` 身份验证:

```python
[auth.gitlab]
enabled = true
allow_sign_up = false
client_id = GITLAB_APPLICATION_ID
client_secret = GITLAB_SECRET
scopes = api
auth_url = https://gitlab.com/oauth/authorize
token_url = https://gitlab.com/oauth/token
api_url = https://gitlab.com/api/v4
allowed_groups =
```

你需要为回调 URL 设置 ```[server]``` 正确的 ```root_url``` 选项才行. 例如, 如果在代理服务器后面提供 ```Grafana``` 服务.

重新启动 ```Grafana``` 后端使更改生效.

如果您使用自己的 ```GitLab``` 实例而不是 ```gitlab.com```, 需要相应地调整 ```auth_url```, ```token_url``` 和 ```api_url``` 中的主机名 ```gitlab.com``` 为你自己的主机名.

将 ```allow_sign_up``` 设置为 ```false```, 只有现有用户才能使用他们的 ```GitLab``` 帐户登录, 但是将 ```allow_sign_up``` 设置为 ```true```, 任何可以在 ```GitLab``` 上进行身份验证的用户都可以登录 ```Grafana``` 实例; 如果你使用公共的 ```gitlab.com```, 这意味着世界上任何人都可以登录你的 ```Grafana``` 实例.

但是, 你可以通过设置 ```allowed_groups``` 选项来限制仅访问给定组或组列表的成员.

### **allowed_groups**

限制对作为一个或多个 [GitLab组](https://docs.gitlab.com/ce/user/group/index.html) 成员的经过身份验证的用户的访问. 将 ```allowed_groups``` 设置为以逗号或空格分隔的组列表. 例如, 如果只想授予对 ```example``` 组成员的访问权限, 设置:

```python
allowed_groups = example
```

如果您还想要访问子分组 ```bar``` 的成员, 在 ```foo``` 组中, 设置:

```python
allowed_groups = example, foo/bar
```

请注意, 在 ```GitLab``` 中, 组或子分组名称并不总是与其显示名称匹配, 特别是如果显示名称包含空格或特殊字符. 确保始终使用组或子分组的 URL 中显示的组或子分组名称.

这是启用 ```allow_sign_up``` 的完整示例, 访问仅限于 ```example``` 和 ```foo/bar``` 组:

```python
[auth.gitlab]
enabled = true
allow_sign_up = true
client_id = GITLAB_APPLICATION_ID
client_secret = GITLAB_SECRET
scopes = api
auth_url = https://gitlab.com/oauth/authorize
token_url = https://gitlab.com/oauth/token
api_url = https://gitlab.com/api/v4
allowed_groups = example, foo/bar
```
