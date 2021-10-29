---
title: Gitlab的安装与优化
date: 2019-10-8
tags:
- DevOps
- Gitlab
category: DevOps
abstract: Ubuntu上Gitlab的安装与优化
---

### 如何在Ubuntu上安装gitlab

- 首先我们需要将清华的gitlab源配置到我们的系统中，因为gitlab海外下载速度非常慢。

    - 默认我们使用root账户登录系统，首先信任 GitLab 的 GPG 公钥
    ```
    curl https://packages.gitlab.com/gpg.key 2> /dev/null | sudo apt-key add - &>/dev/null
    ```
    
    - 添加清华源
    ```
    touch /etc/apt/sources.list.d/gitlab-ce.list
    
    echo "deb https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/ubuntu xenial main" >> /etc/apt/sources.list.d/gitlab-ce.list
    ```

    - 更新源并安装gitlab
    ```
    apt-get update
    apt-get install gitlab-ce
    ```

- 安装完成之后，我们需要对gitlab进行配置。配置文件路径：/etc/gitlab/gitlab.rb

    - 修改gitlab对外url。我这里使用的是阿里云的服务器，在购买完成后会分配给你一个公网IP，
    假设这里是 48.103.200.64。还要指定对外的端口，假设是8000。去服务器管理页面进行防火墙端口例外添加。找到gitlab.rb然后修改下面字段：
    ```
    因为没有申请SSL证书，所以这里先用http协议（阿里云提供免费的SSL证书申请）
    external_url 'http:// 48.103.200.64:8000'
    ```

- 配置完成之后就可以访问了
```
使用 gitlab-ctl reconfigure 使配置生效
使用 gitlab-ctl start 启动gitlab
```

### 优化gitlab内存使用

当我们启动gitlab并在浏览器中输入url进行访问时，性能不足的服务器会返回502的情况，比如我现在使用的服务器是阿里云1核2G服务器。那怎么进行优化呢？

#### swap的创建
我们需要创建swap分区。因为gitlab默认要求是4G内存，所以我需要再创建2G内存。

- 创建swap的命令格式为：
```
dd if=/dev/zero of=/mnt/swap bs=block_size count=number_of_block
block_size、number_of_block大小可以自定义，比如bs=1M count=1024代表设置1G大小SWAP分区
```

- 创建用于交换分区的文件，我这里创建2G
```
dd if=/dev/zero of=/var/swap bs=1M count=1024
```

- 设置交换分区文件
```
 mkswap /var/swap
```

- 如需要设置开机时自启动SWAP分区，需要修改文件/etc/fstab中的SWAP行，添加以下代码
```
echo /mnt/swap swap swap defaults 0 0 >> /etc/fstab
```

- 同时我们需要修改swappiness的值
    > 在Linux系统中，可以通过查看/proc/sys/vm/swappiness内容的值来确定系统对SWAP分区的使用原则。当 swappiness内容的值为0时，表示最大限度地使用物理内存，物理内存使用完毕后，才会使用SWAP分区。当swappiness内容的值为100时，表示积极地使用SWAP分区，并且把内存中的数据及时地置换到SWAP分区。查看修改前为0，需要在物理内存使用完毕后才会使用SWAP分区。

    ```
    修改文件swappiness中的值为100
    echo 100 > /proc/sys/vm/swappiness
    ```

    ```
    若需要永久修改此配置，在系统重启之后也生效的话，通过vim命令编辑/etc/sysctl.conf文件，并增加以下内容
    vm.swappiness = 100
    
    执行以下命令，验证添加成功
    sysctl -p
    ```
    
    > [Linux实例SWAP分区的配置和常见问题处理](https://help.aliyun.com/knowledge_detail/42534.html)

#### gitlab配置优化
在上一节，我们通过配置SWAP分区对gitlab内存使用的问题做了优化，但是使用 free -h 命令会发现gitlab的内存使用率特别高，那接下来对gitlab本身的配置进行修改。找到/etc/gitlab/gitlab.rb

- 减少进程数
```
修改 worker_processes 的值，并取消注释
unicorn['worker_processes'] = 3
```

- 修改超时时间
```
修改 worker_timeout 的值，并取消注释
unicorn['worker_timeout'] = 60    
```

- 减少数据库缓存大小 默认256，可适当改小
```
postgresql['shared_buffers'] = "128MB"
```

- 减少数据库并发数
```
postgresql['max_worker_processes'] = 3
```

- 减少sidekiq并发数
```
sidekiq['concurrency'] = 6
```

- 减少unicorn内存使用
```
unicorn['worker_memory_limit_min'] = "200 * 1 << 20"
unicorn['worker_memory_limit_max'] = "297 * 1 << 20"
```

- 使配置生效并启动gitlab
```
使用 gitlab-ctl reconfigure 是配置生效
使用 gitlab-ctl start 启动gitlab
```
> [解决GitLab内存消耗大的问题](https://blog.csdn.net/ouyang_peng/article/details/84066417)
> 
> [阿里云安装GitLab内存不足解决方法](https://blog.csdn.net/SirLZF/article/details/88954488)

