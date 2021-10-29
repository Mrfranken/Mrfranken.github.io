---
title: 如何在Gitlab上开启邮件功能
date: 2019-11-08
tags:
- DevOps
- Gitlab
category: DevOps
abstract: 如何在Gitlab上开启邮件功能
---

### 配置邮件服务 
上一篇我们提到了gitlab的安装和优化。但是有个问题，gitlab自己一个人用也太无趣了，我们需要开启
邮件服务然后让用户能注册使用，接下来来看看怎么开启gitlab的邮件服务。
因为我购买的是阿里云的服务器，关于邮件服务的25端口阿里云是禁止的，所以这边我们使用465端口。

- 首先打开自己的邮箱（我这边使用的是网易的163邮箱），然后找到打开SMTP服务的选项，
163在打开后会提供一个类似**授权码**的东西给我们，记下来后面会用到。

- 编辑gitlab的配置文件gitlab.rm
```
vi /etc/gitlab/gitlab.rb
```

- 找到smtp的选项
```
gitlab_rails['smtp_enable'] = true
gitlab_rails['smtp_address'] = "smtp.163.com"
gitlab_rails['smtp_port'] = 465
gitlab_rails['smtp_user_name'] = "xxxxx@163.com"  # 这里填入自己的邮箱
gitlab_rails['smtp_password'] = "xxxxx"  # 这里填入上面提到的授权码，注意不是邮箱的登陆密码
gitlab_rails['smtp_domain'] = "163.com"
gitlab_rails['smtp_authentication'] = "login"
gitlab_rails['smtp_enable_starttls_auto'] = true
gitlab_rails['smtp_tls'] = true

```

- 继续修改配置
```
gitlab_rails['gitlab_email_enabled'] = true
gitlab_rails['gitlab_email_from'] = 'xxxxx@163.com'
user['git_user_email'] = "xxxxx@163.com"
```

### 重启服务测试邮箱

- 配置好之后，重启gitlab服务
```
gitlab-ctl reconfigure
```

- 测试邮箱功能是否可用
```
gitlab-rails console

Notify.test_email('target mail address', 'Message Subject', 'Message Body').deliver_now  
# 这条命令执行完目标邮箱中会收到我们配置的邮箱发送的测试邮件
```