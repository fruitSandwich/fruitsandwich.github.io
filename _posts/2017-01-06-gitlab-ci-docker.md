---
layout: post
title:  "使用gitlab-ci持续集成"
categories: [git,工具]
---

之前尝试了使用docker部署gitlab管理私有git代码库。可以看<a href='/gitlab-docker-build/'>这里</a>。今天尝试了一下gitlab ci进行持续集成

持续集成相关概念：
- <a href='http://www.ruanyifeng.com/blog/2015/09/continuous-integration.html'> http://www.ruanyifeng.com/blog/2015/09/continuous-integration.html</a>

持续集成是一种软件开发实践，即团队开发成员经常集成他们的工作，通常每个成员每天至少集成一次，也就意味着每天可能会发生多次集成。每次集成都通过自动化的构建（包括编译，发布，自动化测试)来验证，从而尽快地发现集成错误。许多团队发现这个过程可以大大减少集成的问题，让团队能够更快的开发内聚的软件。


目前持续集成工具有很多，比如Jenkins、Travis CI还有接下来要说的GitLab CI。

持续集成GitLab相关资料：

- <a href='https://segmentfault.com/a/1190000006120164'>https://segmentfault.com/a/1190000006120164</a>
- <a href='http://www.jianshu.com/p/2b43151fb92e'>http://www.jianshu.com/p/2b43151fb92e</a>
- <a href='https://my.oschina.net/donhui/blog/717930'>https://my.oschina.net/donhui/blog/717930</a>

GitLab-CI就是一套配合GitLab使用的持续集成系统（当然，还有其它的持续集成系统，同样可以配合GitLab使用，比如Jenkins）。GitLab8.0以后的版本默认集成了GitLab-CI并且默认启用的，所以这里直接就可以在gitlab中使用gitlab-ci了。

然后就是gitlab runner。当git工程的仓库代码发生变动时，比如有人push了代码，GitLab就会将这个变动通知GitLab-CI。这时GitLab-CI会找出与这个工程相关联的Runner，并通知这些Runner把代码更新到本地并执行预定义好的执行脚本。下面这张图说明了gitlab、gitlab-ci、gitlab runner三者的关系：

![image](/asserts/201701/gitlab-ci-runner.png)


上次用docker安装的gitlab自带gitlab-ci所以，只需要安装gitlab runner就可以了(有意思的是，gitlab runner不一定要和gitlab安装在一块，可以安装在其他地方，也可以安装多个)。

我仍然选择了docker的方式来<a href='https://docs.gitlab.com/runner/install/docker.html'>安装gitlab runner</a>。安装脚本：


1.首先拉取docker镜像

```
docker pull gitlab/gitlab-runner     
```

2.然后跑起来(注意下面的srv需要替换成本地目录)

```
docker run -d --name gitlab-runner --restart always \
  -v /srv/gitlab-runner/config:/etc/gitlab-runner \
  gitlab/gitlab-runner:latest
```

3.注册gitlab-runner

```
docker exec -it gitlab-runner gitlab-runner register

Please enter the gitlab-ci coordinator URL (e.g. https://gitlab.com )
https://gitlab.com
Please enter the gitlab-ci token for this runner
xxx
Please enter the gitlab-ci description for this runner
my-runner
INFO[0034] fcf5c619 Registering runner... succeeded
Please enter the executor: shell, docker, docker-ssh, ssh?
docker
Please enter the Docker image (eg. ruby:2.1):
ruby:2.1
INFO[0037] Runner registered successfully. Feel free to start it, but if it's
running already the config should be automatically reloaded!
```

上面这段脚本，第一句是输入后执行的，下面的脚本是一个个提示输入的。需要填几个地方,其中第一条URL和第二条gitlab-ci token来自项目的runners页面；提示registering runner成功后还有executor要填，选择shell一开始漏填导致runner注册上但不可用（可用的runner如下图显示为绿色圆点）。
![image](/asserts/201701/gitlab-ci-page.png)


注册成功，runners页面显示有可用的runners后就可以试用一下了，在项目根路径添加.gitlab-ci.yml配置，内容如下：

```
# 定义 stages
stages:
  - build
  - test
# 定义 job
job1:
  stage: test
  script:
    - echo "I am job1"
    - echo "I am in test stage"
# 定义 job
job2:
  stage: build
  script:
    - echo "I am job2"
    - echo "I am in build stage"
```

git提交并push到gitlab，可以打开commits或pipelines查看runner执行结果：
![image](/asserts/201701/gitlab-ci-result.png)
发现出错了，查看一下出错信息:

```
Running with gitlab-ci-multi-runner 1.9.2 (ade6572)
Using Shell executor...
Running on 1cb7361fc01b...
Cloning repository...
Cloning into '/home/gitlab-runner/builds/d995a5a8/0/fruitsandwich/book'...
fatal: unable to access 'http://gitlab-ci-token:xxxxxxxxxxxxxxxxxxxx@localhost:10080/fruitsandwich/book.git/': Failed to connect to localhost port 10080: Connection refused
ERROR: Build failed: exit status 1
```

不能连接到localhost！！！！

想了半天不知道localhost哪来的，明明注册的时候填的是具体ip，为何就变成了localhost。找了半天原因，在gitlab.com上试验了一下发现没有这种问题，应该是本地gitlab的问题。找了一下安装gitlab的docker-commpose.yml，发现其中有这么一行：

```
  - GITLAB_HOST=localhost
```

gitlab指定的host为localhost，然后改成具体IP重启gitlab再提交就成功了。。。
