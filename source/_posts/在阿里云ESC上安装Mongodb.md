---
title: 在阿里云ESC上安装Mongodb
date: 2017-09-25 13:04:32
tags:
- 阿里云
- Mongodb
categories:
- 技术
---
# 在阿里云ESC上安装Mongodb
## 准备
### 本地环境及使用工具
- win10
- putty
- FileZilla
- Mongodb官网下载的安装包
- ROBO 3T

### 线上环境
- 阿里云ECS
- Ubuntu 16.04.2 LTS


## 安装Mongodb

首先我们需要现在本地下载Mongodb的压缩包，我这边使用的是mongodb-linux-x86_64-ubuntu1604-3.4.9版本。

之后连接上ESC，通过FileZilla将压缩包上传到指定文件夹。

上传成功后在压缩包所在的文件夹内对压缩包进行解压，我这边下载的压缩包格式为tgz，所以使用以下命令来解压

`tar -zxvf mongodb-linux-x86_64-ubuntu1604-3.4.9.tgz`

解压成功后当前目录下会出现一个以mongodb-linux-x86_64-ubuntu1604-3.4.9命名的文件夹，里面是一些可执行的脚本。

现在我们需要在创建一个文件夹来存放Mongodb的一些配置文件。我选择在服务器根目录下创建

`mkdir mongodb && cd mongodb`

创建mongodb文件夹后，还需要在该文件夹下创建几个文件夹

`mkdir data`

data文件夹用来存放数据

```
mkdir logs && cd logs
touch mongo.log
```

logs文件夹用来存放日志

```
mkdir etc && cd etc
vi mongo.conf
```

创建etc文件夹，在文件夹下创建conf文件来进行相关配置，以下是我的配置。

```
dbpath=/mongodb/data
logpath=/mongodb/logs/mongo.log
logappend=true
journal=true
quiet=true
port=777
```


特别需要说明的是，之前安装后一直无法启动mongodb，即使在阿里云控制台里允许了默认端口27017也不行，个人猜测可能因为安全问题阿里云关闭了数据库的默认端口，这里我把配置文件里的端口改为非27017之后，再次启动数据后就成功了。

写好配置文件后，需要将之前解压后的文件夹移动到mongodb文件夹下。

`mv mongodb-linux-x86_64-ubuntu1604-3.4.9 mongodb`

之后进入mongodb-linux-x86_64-ubuntu1604-3.4.9/bin文件夹下，使用配置文件启动mongodb

`mongod -f /mongodb/etc/mongo.conf`

![](http://okluou8tc.bkt.clouddn.com/%E5%90%AF%E5%8A%A8mongodb.png)

此时我们无法确定是否启动成功，可以使用putty新建一个窗口进入到服务器，输入`mongo`,如果出现以下信息，那么mongodb基本就启动成功了。

![](http://okluou8tc.bkt.clouddn.com/%E5%90%AF%E5%8A%A8%E6%88%90%E5%8A%9F.png)

启动成功后我们可以在本地使用ROBO 3T来连接服务器。

至此，Mongodb就已经成功安装到ECS服务器上了。







