## **在Mac上安装**

### **使用 homebrew 安装**
安装可以使用 [homebrew](http://brew.sh/)

安装最新稳定版:
```
brew update
brew install grafana
```

注意命令行打印输出,看到 homebrew 安装完成的信息后即可启动 Grafana.

升级的话使用 reinstall 命令
```
brew update
brew reinstall grafana
```

你也可以通过 git 安装最新的的非稳定版:
```
brew install --HEAD grafana/grafana/grafana
```

如果你是从 HEAD 版本安装的话可以通过以下方式升级:
```
brew reinstall --HEAD grafana/grafana/grafana
```

### **启动 Grafana**
首次通过 homebrew 服务启动 Grafana 之前要确保 homebrew/services 是否已安装.
```
brew tap homebrew/services
```

使用下面的命令启动 Grafana:
```
brew services start grafana
```

默认的登录账号信息为: `admin/admin`

### **配置**
配置文件是 `/usr/local/etc/grafana/grafana.ini`

### **日志**
日志文件是 `/usr/local/var/log/grafana/grafana.log`

### **插件**
手动安装插件的目录是 `/usr/local/var/lib/grafana/plugins`

### **数据库**
sqlite数据库在 `/usr/local/var/lib/grafana`

### **使用二进制包安装**
下载[最新的`.tar.gz`文件](https://grafana.com/get)然后解压.将会解压到以版本号为后缀的目录.这个目录包含 Grafana 运行所需的所有文件.这个包不包含 init 脚本以及安装脚本.

添加 `custom.ini` 配置文件到 `conf` 目录,然后重新配置 `conf/defaults.ini` 里面的所有配置项.

执行 `./bin/grafana-server web` 启动 Grafana.`grafana-server` 这个二进制文件的工作目录要在安装的根目录下(二进制文件和 `public` 目录所在的位置).

### **首次登录**
使用浏览器打开 [http://localhost:3000/](http://localhost:3000/).如果你没有[配置不同端口](http://docs.grafana.org/installation/configuration/#http-port)的话,3000是 Grafana 的默认监听端口.接下来的一些介绍在[这里](http://docs.grafana.org/guides/getting_started/). 
