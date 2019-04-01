## **使用 Docker 安装**

Grafana 可以很容易的使用 *```Docker```* 官方容器进行安装和运行.

```
$ docker run -d -p 3000:3000 grafana/grafana
```

### **配置**
所有的配置量都在 *```conf/grafana.ini```*, 可以使用 *```GF_<SectionName>_<KeyName>```* 这种语法然后通过环境变量的方式进行覆盖.例如:
```
$ docker run \
  -d \
  -p 3000:3000 \
  --name=grafana \
  -e "GF_SERVER_ROOT_URL=http://grafana.server.name" \
  -e "GF_SECURITY_ADMIN_PASSWORD=secret" \
  grafana/grafana
```

后端 *Web* 服务也有一些配置项.详见[配置](http://docs.grafana.org/installation/configuration/)页面.

> 如果要让 *```conf/grafana.ini```* 的变更 (或者相关的环境变量) 生效,你需要重启 *```Docker```* 容器来重启 *Grafana* 服务.

### **默认目录**
以下这些设置是硬编码的,你可以在 *Grafana* 的 *```Docker```* 容器启动的时候通过环境变量的方式来变更这些设置,而不是通过 *```conf/grafana.ini```*.

|设置|默认值|
| :----- | :------ |
| GF_PATHS_CONFIG | /etc/grafana/grafana.ini |
| GF_PATHS_DATA | /var/lib/grafana |
| GF_PATHS_HOME | /usr/share/grafana |
| GF_PATHS_LOGS | /var/log/grafana |
| GF_PATHS_PLUGINS | /var/lib/grafana/plugins |
| GF_PATHS_PROVISIONING | /etc/grafana/provisioning |

### **运行指定版本的 Grafana**
```
# specify right tag, e.g. 5.1.0 - see Docker Hub for available tags
$ docker run \
  -d \
  -p 3000:3000 \
  --name grafana \
  grafana/grafana:5.1.0
```

### **运行主分支版本**
我们为每个构建成功的主分支更新 *```grafana/grafana:master```* 标签,然后创建一个新的标签 *```grafana/grafana-dev:master-<commit hash>```* 并且包含 *```git```* 提交时的 *```hash```*,这样你总能获取到最新版的 *Grafana*.

如果在生产环境中使用 *Grafana* 的话,强烈推荐使用 *```grafana/grafana-dev:master-<commit hash>```* 标签,这样可以保证你使用的是指定版本的 *Grafana* 而不是最新提交的版本.

要获取可用标签的版本列表,可以检出[grafana/grafana](https://hub.docker.com/r/grafana/grafana/tags/))和[grafana/grafana-dev](https://hub.docker.com/r/grafana/grafana-dev/tags/).

### **安装 Grafana 插件**
通过 *```GF_INSTALL_PLUGINS```* 环境变量作为单独的命令列,你可以安装插件.这样将会传递每个插件名称到 *```grafana-cli plugins install ${plugin}```*,然后 *Grafana* 启动的时候会进行安装操作.
```
docker run \
  -d \
  -p 3000:3000 \
  --name=grafana \
  -e "GF_INSTALL_PLUGINS=grafana-clock-panel,grafana-simple-json-datasource" \
  grafana/grafana
```

### **构建预安装插件的 Grafana 定制镜像**
在[grafana-docker](https://github.com/grafana/grafana/tree/master/packaging/docker)中有个叫 *```custom/```* 的目录,里面有个 *```Dockerfile```* 可以用于构建定制的镜像.它接受 *```GRAFANA_VERSION```* 和 *```GF_INSTALL_PLUGINS```* 作为构建参数.

如下:
```
cd custom
docker build -t grafana:latest-with-plugins \
  --build-arg "GRAFANA_VERSION=latest" \
  --build-arg "GF_INSTALL_PLUGINS=grafana-clock-panel,grafana-simple-json-datasource" .

docker run \
  -d \
  -p 3000:3000 \
  --name=grafana \
  grafana:latest-with-plugins
```

### **从源码安装插件**
>*Grafana v5.3.1+* 才支持

可以通过指定如下的 *url:s* 进行安装:

*```GF_INSTALL_PLUGINS=<url to plugin zip>;<plugin name>```*
```
docker run \
  -d \
  -p 3000:3000 \
  --name=grafana \
  -e "GF_INSTALL_PLUGINS=http://plugin-domain.com/my-custom-plugin.zip;custom-plugin" \
  grafana/grafana
```

### **配置 AWS 证书支持 CloudWatch**
```
$ docker run \
  -d \
  -p 3000:3000 \
  --name=grafana \
  -e "GF_AWS_PROFILES=default" \
  -e "GF_AWS_default_ACCESS_KEY_ID=YOUR_ACCESS_KEY" \
  -e "GF_AWS_default_SECRET_ACCESS_KEY=YOUR_SECRET_KEY" \
  -e "GF_AWS_default_REGION=us-east-1" \
  grafana/grafana
```

你也许可以指定多个 *```profiles```* 给 *```GF_AWS_PROFILES```* (e.g. ```GF_AWS_PROFILES=default another```).

支持的变量:
- *```GF_AWS_${profile}_ACCESS_KEY_ID```*: AWS 访问秘钥 ID (需要).
- *```GF_AWS_${profile}_SECRET_ACCESS_KEY```*: AWS 私有访问秘钥 (需要).
- *```GF_AWS_${profile}_REGION```*: AWS region (可选).

### **持久化存储 Grafana 容器 (推荐)**
```
# create a persistent volume for your data in /var/lib/grafana (database and plugins)
docker volume create grafana-storage

# start grafana
docker run \
  -d \
  -p 3000:3000 \
  --name=grafana \
  -v grafana-storage:/var/lib/grafana \
  grafana/grafana
```

### **Grafana 容器使用绑定挂载**
你也许想在 *```Docker```* 中使用 *Grafana*,但是又想使用你主机上的数据库或者配置.
这样,通过指定的用户启动容器会变的很重要,因为这样你才有权限读写该用户映射到容器中的目录.
```
mkdir data # creates a folder for your data
ID=$(id -u) # saves your user id in the ID variable

# starts grafana with your user id and using the data folder
docker run -d --user $ID --volume "$PWD/data:/var/lib/grafana" -p 3000:3000 grafana/grafana:5.1.0
```

### **从文件读取秘钥 (支持 Docker 秘钥)**
>*Grafana v5.2+* 才支持

可以通过以配置文件的形式来提供 Grafana 服务,通过使用 [Docker 秘钥](https://docs.docker.com/engine/swarm/secrets/) 作为默认的秘钥, 通过映射到容器的 *```/run/secrets/<name of secret>```* 的这个位置进行获取, 这样可以很好的实现这个功能.

你可以在 *```conf/grafana.ini```* 中配置 *```GF_<SectionName>_<KeyName>__FILE```* 来指定秘钥位置.

也就是说你可以这样设置管理员密码.
- 管理员密码秘钥: *```/run/secrets/admin_password```*
- 环境变量: *```GF_SECURITY_ADMIN_PASSWORD__FILE=/run/secrets/admin_password```*

### **迁移之前版本的容器到 5.1 或者近期的版本**
Grafana 容器在 5.1 版本有较大改动.

主要变更
- 文件所有权不再在启动期间通过 *```chown```* 编辑
- 默认用户 id 从 ```104``` 变更为 ```472```
- 下面的卷不再隐式
   - ```/var/lib/grafana```
   - ```/etc/grafana```
   - ```/var/log/grafana``` 

### **移除隐式卷**
早前的 *```/var/lib/grafana```*, *```/etc/grafana```* 和 *```/var/log/grafana```* 作为卷定义在 *```Dockerfile```* 里面. 这会导致启动新的 Grafana 容器实例时同一时间创建这三个卷,不管你需不需要他们.

你应该谨慎定义你自己命名的卷,因为如果你基于这三个卷,那么在升级后将不再使用它们.

**警告:** 如果使用 *```Docker compose```* 以及隐式卷从较早版本迁移到5.1或者近期的版本,你需要使用 *```docker inspect```* 找出哪些卷你要映射到,这样你就可以把它们映射到已升级的容器.同时你还要像下面文档那样变更文件所有权(或者用户).

#### **用户ID变更**
在5.1版本中我们切换成 *grafana* 用户的ID.不幸的是,这样5.1版本之前创建的文件在后续版本中的权限就不对了.我们为此进行更改,以便 *grafana* 用户ID更有可能是 *Grafana* 独有的.例如,在 Ubuntu 16.04 *104* 还被 syslog 用户使用.

| 版本 | 用户 | 用户ID |
| :----- | :----- | :----- |
| < 5.1 | grafana | 104 |
| >= 5.1 | grafana | 472 |

这个问题有两种可能的方案.可以使用 root 用户启动新的容器来变更权限,从 ```104``` 到 ```472```,或者使用用户 ```104``` 启动已升级的容器.

#### **使用其他用户运行Docker**
```
docker run --user 104 --volume "<your volume mapping here>" grafana/grafana:5.1.0
```

在 ```docker-compose.yml``` 中指定用户
```
version: "2"

services:
  grafana:
    image: grafana/grafana:5.1.0
    ports:
      - 3000:3000
    user: "104"
```

#### **变更权限**
下面的命令将在包含你映射卷的容器中执行*bash*.这意味着可以编辑文件权限来适配新的容器.不过编辑权限要时刻小心.

```
$ docker run -ti --user root --volume "<your volume mapping here>" --entrypoint bash grafana/grafana:5.1.0

# in the container you just started:
chown -R root:root /etc/grafana && \
  chmod -R a+r /etc/grafana && \
  chown -R grafana:grafana /var/lib/grafana && \
  chown -R grafana:grafana /usr/share/grafana
```



### **首次登录**
使用浏览器打开 [http://localhost:3000/](http://localhost:3000/).如果你没有[配置不同端口](http://docs.grafana.org/installation/configuration/#http-port)的话,3000是 Grafana 的默认监听端口.接下来的一些介绍在[这里](http://docs.grafana.org/guides/getting_started/). 
