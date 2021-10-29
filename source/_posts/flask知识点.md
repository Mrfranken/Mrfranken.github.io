---
title: flask知识点
date: 2020-05-15
tags:
- web
- python
- flask
category: 
- web
- python
abstract: 一些flask知识点
---

- 路由的注册方法
    - 使用flask核心对象注册
    - 使用blueprint注册视图函数，app核心对象注册blueprint
- 响应对象
    - 响应内容的不同类型导致浏览器显示不一样
    - return content, status_code, header更简洁的返回
        - 如果要返回json字符串，直接使用用jsonify

- if \_\_name\_\_ == "\_\_main\_\_" 的问题


- 循环引用

    [![YuGcWD.jpg](https://s1.ax1x.com/2020/05/08/YuGcWD.jpg)](https://imgchr.com/i/YuGcWD)

    - 循环引用情况下找不到视图函数的原因？图中红色主流程创建的app对象被蓝色流程的app覆盖，app.route注册时使用的是蓝色流程的app，但运行时使用的是红色主流程的app核心对象。
    - 将试图函数拆分到单独的文件，使用blueprint进行注册
- 应用、蓝图与视图函数

[![Yusj8s.png](https://s1.ax1x.com/2020/05/08/Yusj8s.png)](https://imgchr.com/i/Yusj8s)

- 最上层是app核心对象；
- 核心对象上可以插入很多蓝图，这个蓝图是不能单独存在的，必须使用app核心对象将其插入；
- 在每一个蓝图上，可以注册很多静态文件，视图函数，模板；
- 一个业务模块可以做为一个蓝图，比如book，user。可以把视图函数注册到蓝图上再插入app。以此来达到之前分文件的目的
- **并不是用来拆文件用的（小题大做），而是用来在工程当中拆分模块的。**
- **新建一个文件初始化蓝图类，然后在业务类类中导入这个蓝图添加route即可（单蓝图多视图拆分文件）**

- request对象
- 使用WTForms进行参数校验
    - from wtforms import Form
    - StringField, IntegerField from wtforms.validators import Length, NumberRange, DataRequired

- 拆分配置文件（不同的配置信息放在不同的配置文件里）
    - 在需要使用配置内容的情况下使用current_app进行配置信息的加载
        - from flask import current_app
current_app.config['PER_PAGE']

- Model Frist、Database First、Code First
    - 自定义数据表类，并使用sqlalchemy规定的字段
    - 使用flask_sqlalchemy进行代码到数据表的映射
    - Code First：解决的是创建数据的问题
    - ORM（对象关系映射）: 不仅包含数据的创建，也包含数据的增删改查

- 应用上下文和请求上下文
    - AppContext：封装Flask核心对象并添加额外方法
    - RequestContext：封装Request对象并添加额外方法
    - current_app 和 request 作为localproxy对象提供了操作上下文的能力

- **核心知识点**
    - ==读懂 Local 和 LocalStack 类的作用==
    - ==一个请求过来，RequestContext得到初始化。并调用RequestContext的push方法。在push方法中先\_request_ctx_stack先进行判断，然后在对\_app_ctx_stack进行判断。判断完成之后先对app context进行入栈，然后再对request context进行入栈。==

    - 通过Flask的核心对象app，拿到app context再push入栈是可以的
        ```
        from flask import Flask, current_app
        
        app = Flask(__name__)
        ctx = app.app_context()
        ctx.push()
        # use current_app do sth
        ctx.pop()
        ```
        
    - 但还有一种更为简洁的方式，可以发现app_context()返回一个AppContext对象，并且AppContext中重写了enter和exit方法，并且在enter方法中已经自动将app context入栈,在exit自动将app出栈。所以可以使用更为简洁的上下文管理器，无需再手动入栈和出栈：
        ```
        from flask import Flask, current_app
        
        app = Flask(__name__)
        with app.app_context() as c:
            current = c.app
            
        或
        
        with app.app_context():
            cp = current_app
        ```
    - 进程和线程
        - 线程是进程的一部分
        - 进程用来分配和调度资源
        - 线程利用CPU执行代码
        - pyhton因为GIL的原因不能充分利用多线程的优势
    - 默认情况下flask的web server使用单进程、单线程的方式响应client的请求，如果想要开启多线程，在run方法中传入 ==threaded=True==
    - 线程隔离对象Local
        - 通过线程ID进行隔离（self.\_\_storage\_\_[self.\_\_ident\_func\_\_()][name]）
    
    - **LocalStack、Local和dict之间的关系（理解LocalStack、Local源码）**
        - LocalStack线程隔离的栈结构（既是栈结构也是线程隔离的）
        - Local实现了线程隔离
    - LocalStack的基本用法
        - 栈特性：先进后出
        - 线程隔离特性：LocalStack的同一个对象在不同线程中是线程隔离的
        - push
        - pop
        - top
    - **flask中被线程隔离的对象**
        - flask需要将两个上下文推入到栈中
        - flask需要上下文之间是线程隔离的
        - 基于上面两个特性 栈结构 + 线程隔离， LocalStack可以满足
        - flask使用LocalStack为了两个上下文对象（除了两个类型的上下文放在不同的栈中，上下文对象本身也是隔离的），那为什么要隔离两个对象？想象一个多个请求指向同一个视图函数，那意味着一个上下文需要用一个变量名（例如request）指向多个Request实例化对象，那怎样做到不混淆呢，这个时候Local的线程隔离就发挥了作用。
        - ==使用线程隔离的意思在于使当前线程能够正确引用到它自己所创建的对象，而不是引用到其他线程所创建的对象。==
        - 除了Request上下文对象是线程隔离的，Flask中==session、g都是线程隔离的==，但是==current_app也就是flask的app核心对象并不是线程隔离的，它是全局唯一的 ==。
        
    - flask中的一些名词
        - 线程隔离对象和被线程隔离的对象
            - Local 和 LocalStack 是线程隔离对象
            - AppContext 和 RequestContext 是被线程隔离的对象
        - AppContext将app核心对象作为一个属性保存起来
        - RequestContext将请求对象Request保存起来
        - current_app指向栈顶元素AppContext
        - AppContext的top属性指向app核心对象
        - request指向栈顶元素RequestContext
        - RequestContextt的top属性指向Rrequest对象
