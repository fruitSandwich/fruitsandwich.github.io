---
layout: post
title:  "使用docker快速搭建gitlab环境"
categories: [git,工具]
---


## 1.开放的git托管平台
版本控制工具选择git的话，github是首选的远程托管平台。但它只免费提供了共有仓库，如果想用私有仓库是需要付费的。

对于占时还不想开源又有远端版本管理的需求，国内有几个github的替代平台：
1. oschina:[https://git.oschina.net/](https://git.oschina.net/)
2. coding:[https://coding.net](https://coding.net)

## 2.gitlab
如果对提供了私有仓库的平台还不放心，比如私密性要求比较高那么就需要一个私有的远程git托管平台。目前最常用的是gitlab，oschina就是使用gitlab扩展的。

[gitlab](https://about.gitlab.com/)是一个使用 Ruby on Rails 开发的开源应用程序，与Github类似，能够浏览源代码，管理缺陷和注释，非常适合在团队内部使用。gitlab是基于Ruby on Rails的，安装和配置非常麻烦：[https://about.gitlab.com/downloads/](https://about.gitlab.com/downloads/)

## 3.使用docker-gitlab安装gitlab

安装部署这种脏累活，可以借助docker来简化。好在已经有人对提供了gitlab的容器镜像：[https://github.com/sameersbn/docker-gitlab](https://github.com/sameersbn/docker-gitlab)

使用docker安装gitlab十分便捷：
首选安装[docker](https://www.docker.com/)，各操作系统安装都非常简单。


依照docker gitlab的安装教程：[http://www.damagehead.com/docker-gitlab/](http://www.damagehead.com/docker-gitlab/)

### 3.1获取docker-compose.yml
```
wget https://raw.githubusercontent.com/sameersbn/docker-gitlab/master/docker-compose.yml
```
得到的文件大致是这个样子的


```
version: '2'

services:
  redis:
    restart: always
    image: sameersbn/redis:latest
    command:
    - --loglevel warning
    volumes:
    - /srv/docker/gitlab/redis:/var/lib/redis:Z

  postgresql:
    restart: always
    image: sameersbn/postgresql:9.5-4
    volumes:
    - /srv/docker/gitlab/postgresql:/var/lib/postgresql:Z
    environment:
    - DB_USER=gitlab
    - DB_PASS=password
    - DB_NAME=gitlabhq_production
    - DB_EXTENSION=pg_trgm

```
注意有两个文件地址"/srv/docker",这里要改成本机某个地址，否则会找不到位置。

### 3.2开始安装
```
docker-compose up
```
可能会出现docker-compose命令不存在的情况，需要先安装一下。

命令执行完之后，gitlab就启动了。。
![image](/asserts/201701/gitlab.png)
