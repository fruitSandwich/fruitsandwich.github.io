---
layout: post
title:  "npm私有仓库"
categories: [工具]
---

## 一、npm

NPM是随同NodeJS一起安装的包管理工具，能解决NodeJS代码部署上的很多问题，常见的使用场景有以下几种：
- 允许用户从NPM服务器下载别人编写的第三方包到本地使用。
- 允许用户从NPM服务器下载并安装别人编写的命令行程序到本地使用。
- 允许用户将自己编写的包或命令行程序上传到NPM服务器供别人使用。

和<a href='/gitlab-docker-build'>版本管理工具</a>类似,公司出于自身隐私保护需要,不想把自己的代码开源到包管理区，但是又急需一套完整包管工具，来管理越来越多的组件、模块和项目。那么就需要搭建一套私有npm仓库。

## 二、sinopia

私有npm包管理工具最省事方便的是<a href='https://github.com/rlidwka'>rlidwka</a>大神的<a href='https://github.com/rlidwka/sinopia'>sinopia</a>。安装和使用都非常方便。

sinopia还有配套的docker image，使用docker安装更加方便。

## 三、docker工具补充

最近发现一个docker工具，提供可视化的docker镜像、容器的管理和操作。

![image](/asserts/201701/docker-kitematic.png)

docker-toolbox:<a href='https://github.com/docker/toolbox'>https://github.com/docker/toolbox</a>

文档：<a href='https://docs.docker.com/toolbox/toolbox_install_windows/'>https://docs.docker.com/toolbox/toolbox_install_windows/</a>

## 四、sinopia docker镜像安装

docker-sinopia:<a href='https://github.com/kfatehi/docker-sinopia'>https://github.com/kfatehi/docker-sinopia</a>

安装镜像：

```
docker pull keyvanfatehi/sinopia:latest
```

创建容器：

```
docker run --name sinopia -d -p 4873:4873 keyvanfatehi/sinopia:latest
```

容器启动后，可以在docker toolbaox Kitematic中看到容器的状态：

![image](/asserts/201701/kitematic-sinopia.png)

打开对应的web：

![image](/asserts/201701/sinopia.png)

使用npm私有库：

```
npm set registry http://192.168.205.88:4873
npm adduser --registry http://192.168.205.88:4873
```
