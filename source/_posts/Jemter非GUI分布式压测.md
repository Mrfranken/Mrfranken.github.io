---
title: Jemter非GUI分布式压测
date: 2019-05-10
tags:
- Jmeter
- 性能测试
category: 
- 性能测试
- Jmeter
abstract: Jmeter在非GUI模式下使用阿里云服务器进行压测
---

### 单机压测
#### 准备工作

- 在阿里云上部署Flask web工程（这边我自己简单写了几个接口以做测试）
    - [gunicorn + nginx 进行部署](https://zhuanlan.zhihu.com/p/92003835)
    - [flask项目部署到云服务器+域名绑定](https://www.javazhiyin.com/41606.html)
- 安装Java（参考[Ubuntu 安装 JDK8 的两种方式](https://yq.aliyun.com/articles/320088)）
    - [国内Java源](https://mirrors.huaweicloud.com/java/jdk/)
- [下载 jmeter，将 jmeter bin目录添加到环境变量](https://zhuanlan.zhihu.com/p/90509029)
- 本地压测远程接口保存测试计划
- 将测试计划中接口IP改为aliyun==内部IP==(因为我这边使用是阿里云的服务器)
- 上传测试计划文件进行压测

#### 非GUI模式压测命令
##### 压测命令
```
参数讲解:
-h 帮助
-n 非GUI模式
-t 指定要运行的 JMeter 测试脚本文件
-l 记录结果的文件 每次运行之前，(要确保之前没有运行过,即xxx.jtl不存在，不然报错)
-r Jmter.properties文件中指定的所有远程服务器
-e 在脚本运行结束后生成html报告
-o 用于存放html报告的目录（目录要为空，不然报错）

压测命令: jmter -n -t 测试计划.jmx -l result.jlt -e -o /xxx/xxx/xxx
```
- jtl文件为汇总文件，在添加 -> 监听器 -> 汇总报告，其中点击浏览按钮便可以加载查看jtl文件

### 压测接口的性能优化
```
1. 使用非GUI模式：jmeter -n -t test.jmx -l result.jtl
2. 少使用Listener， 如果使用-l参数，它们都可以被删除或禁用。
3. 在加载测试期间不要使用“查看结果树”或“查看结果”表监听器，只能在脚本阶段使用它们来调试脚本。
4. 包含控制器在这里没有帮助，因为它将文件中的所有测试元素添加到测试计划中。
5. 不要使用功能模式,使用CSV输出而不是XML
6. 只保存你需要的数据,尽可能少地使用断言
7. 如果测试需要大量数据，可以提前准备好测试数据放到数据文件中，以CSV Read方式读取。
8. 用内网压测，减少其他带宽影响压测结果
9. 如果压测大流量，尽量用多几个节点以非GUI模式向服务器施压

官方推荐 ：http://jakarta.apache.org/jmeter/usermanual/best-practices.html#lean_mean
```

### 分布式压测
![YEAP9H.jpg](https://s1.ax1x.com/2020/05/06/YEAP9H.jpg)
#### 准备工作
[分布式压测官网教程](http://jmeter.apache.org/usermanual/jmeter_distributed_testing_step_by_step.html)
```
1. 确认防火墙关闭或压测的端口已经添加例外
2. 客户端都在同一个子网上
3. 确保JMeter可以访问服务器
4. 确保在所有系统上使用相同版本的JMeter和Java，混合版本将无法正常工作
5. 已为RMI设置SSL或将其禁用
```

#### 配置Jmeter
- 在每一台从机上安装Java和Jmeter（参考上面单机安装）
- 修改slave上jmeter.properties中的字段
    ```
    remote_hosts=127.0.0.1
    server_port=1100
    server.rmi.ssl.disable=True
    ```
- 修改master上jmeter.properties中的字段
    ```
    remote_hosts=192.168.0.102:8899,192.168.0.101:8899  # 其中的IP是slave的IP，port是slave指定的server_port
    server.rmi.ssl.disable=true
    ```

#### 启动压测
- 在每一台slave上执行 ./jmeter-server
- 在master上执行测试计划
```
jmeter -n -t remote.jmx -r -l result.jtl -e -o /tmp/result
```