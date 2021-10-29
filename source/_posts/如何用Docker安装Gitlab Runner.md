---
title: 如何用Docker安装Gitlab Runner
date: 2019-12-08
tags:
- DevOps
- Gitlab
category: DevOps
abstract: 如何用Docker安装Gitlab Runner
---

上次我们在阿里云的服务器上安装和配置了Gitlab，并开启邮件邀请注册功能。这种情况下Gitlab其实已经可以正常开发使用了，但是有个问题，就是现在的Gitlab还无法完成工程的持续集成（CI）。因为我们没有对Gitlab-CI进行配置，同时也没有配置任何服务来验证代码的单元测试（UT）、静态代码检查（static code analysis）等等。这些过程Gitlab其实是支持的，但是需要我们自己手动的去配置。因为一台服务器安装一个runner实在是过于浪费，我们采取使用docker的方式来安装runner，这样可以在一台服务器上虚拟出多个runner。下面我们就来看一下如何在Docker中安装runner并将它注册到Gitlab中。

### 前置阅读

在安装之前大家不妨看看一些关于Gitlab-CI的文章：

- [用 GitLab CI 进行持续集成](https://scarletsky.github.io/2016/07/29/use-gitlab-ci-for-continuous-integration/)
- [GitLab CI/CD基础概念](http://www.idevops.site/gitlabci/chapter01/)
- [持续集成的基本概念](https://www.funtl.com/zh/apache-dubbo-ci/)

### 使用Docker安装Gitlab Runner

#### 添加Docker国内镜像

这边我购买了一台腾讯的云主机（Unbuntu 16.04）作为安装Docker的主机。

因为Docker有些镜像在国外，我们使用Docker进行image的安装时速度比较慢，所以在安装好Docker以后，我们需要为Docker添加国内镜像源。在**/etc/docker/**下找到daemon.json这个文件（如果没有就新建一个），然后添加如下内容：

```python
{
  "registry-mirrors": [
    "http://f1361db2.m.daocloud.io", 
    "https://mirror.ccs.tencentyun.com", 
    "https://registry.docker-cn.com"
  ]
}
```

添加成功之后重启Docker

```
sudo systemctl restart docker

docker info  # 使用docker info命令即可看到上面的源已经配置成功
```



#### 使用Docker安装Runner

安装命令：

```
docker run -d --name gitlab-runner --restart always \
-v /srv/gitlab-runner/config:/etc/gitlab-runner \
-v /var/run/docker.sock:/var/run/docker.sock \
gitlab/gitlab-runner:latest
```

参数说明：

```
-d  # 设置容器后台运行
--name  # 容器名称
--restart always  # 每次启动容器就重启 gitlab-runner
--v  # 共享目录挂载
```

安装完成之后，我们可以使用**docker ps -as**查看gitlab-runner是否正在运行。接着我们进入gitlab-runner：

```
docker exec -it gitlab-runner bash
```

使用命令进行runner的注册：

```
gitlab-ruuner register
```

#### 注册Gitlab Runner

- [GitLabRunner注册]([http://www.idevops.site/gitlabci/chapter02/01/1-3-gitlabrunner%E6%B3%A8%E5%86%8C/](http://www.idevops.site/gitlabci/chapter02/01/1-3-gitlabrunner注册/))
- [基于Docker注册Gitlab Runner]([https://www.funtl.com/zh/apache-dubbo-ci/%E5%9F%BA%E4%BA%8E-Docker-%E5%AE%89%E8%A3%85-GitLab-Runner.html#%E6%B3%A8%E5%86%8C-runner](https://www.funtl.com/zh/apache-dubbo-ci/基于-Docker-安装-GitLab-Runner.html#注册-runner))





