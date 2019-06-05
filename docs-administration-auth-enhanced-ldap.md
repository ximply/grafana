# **增强型 LDAP 集成**

>增强型 ```LDAP``` 集成只适用于企业版的 ```Grafana```. 了解更多[```Grafana```企业版](https://grafana.com/docs/enterprise/).

增强型 ```LDAP``` 集成在[现有的 ```LDAP``` 集成](https://grafana.com/docs/auth/ldap/)之上增加了额外的功能.

## **团队 LDAP 组同步**

<img src="https://grafana.com/docs/img/docs/enterprise/team_members_ldap.png">

通过增强的 ```LDAP``` 集成, 可以在 ```LDAP``` 组和团队之间设置同步. 这使得作为某些 ```LDAP``` 组成员的 ```LDAP``` 用户能够自动添加/删除 ```Grafana``` 中某些团队的成员. 目前, 只有用户每次登录时才会进行同步, 但目前正在开发一种活跃的后台同步.

```Grafana``` 会跟踪团队中所有同步的用户, 可以在团队成员列表中查看已从 ```LDAP``` 同步的用户, 查看上面屏幕截图中的 ```LDAP``` 标签. 此机制允许 ```Grafana``` 在其 ```LDAP``` 组成员身份更改时从团队中删除现有的同步用户. 此机制还允许手动将用户添加为团队成员, 并且在用户登录时不会将其删除. 这样可以灵活地组合 ```LDAP``` 组成员身份和 ```Grafana``` 团队成员身份.

## **启用团队 LDAP 组同步**

<img src="https://grafana.com/docs/img/docs/enterprise/team_add_external_group.png">

1.导航到 "```Configuration/Teams```"

2.选择一个 团队

3.选择 "```External group sync```" 选项卡, 然后单击 "```Add group```" 按钮

4.插入要与团队同步的 ```LDAP``` 组的 ```LDAP``` 专有名称(DN)

5.单击 "```Add group```" 按钮进行保存

