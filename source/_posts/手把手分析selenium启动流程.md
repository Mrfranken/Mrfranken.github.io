---
title: 手把手分析selenium启动流程
date: 2020-05-01
tags:
- selenium
- python
category: web测试
abstract: selenium启动流程详细分析
---

python selenium库的解读

IDE: Pycharm

先给出这次源码阅读的测试脚本：

    from selenium.webdriver import Chrome
    from selenium.webdriver.chrome.options import Options as ChromeOptions
    
    
    class BaseWebdriver(object):
    
        def _add_proxy(self, option):
            option.add_argument('--proxy-server=http://x.x.x.x')
            return option
    
        def _setup_headless(self, option):
            option.add_argument('--headless')
            return option
    
        def __enter__(self):
            return self
    
        def __exit__(self, exc_type, exc_val, exc_tb):
            self.close()
    
    
    class CustomChromeWebdriver(Chrome, BaseWebdriver):
        def __init__(self, download_dir, headless=False):
            chrome_options = ChromeOptions()
            chrome_options = self._self_define_download_dir(chrome_options, download_dir)
            chrome_options.add_argument('--ignore-certificate-errors')  # 启动参数1
            chrome_options.add_argument('--no-sandbox')  # 启动参数2
            chrome_options.add_argument('--disable-gpu')  # 启动参数3
    
            if headless:  # 用来控制是否已headless模式启动
                chrome_options = self._setup_headless(chrome_options)
            super(CustomChromeWebdriver, self).__init__(chrome_options=chrome_options)
    
        def _self_define_download_dir(self, option, download_dir):
            prefs = {"profile.default_content_settings.popups": 0,
                     "download.default_directory": download_dir}
            option.add_experimental_option('prefs', prefs)
            return option
            
    
    if __name__ == '__main__':
        driver = CustomChromeWebdriver(r'C:\\D', False)
        driver.get('https://www.baidu.com')
        elem = driver.find_element_by_xpath("//input[@type='submit']")
        elem.click()
        title = driver.title()

看下CustomChromeWebdriver这个类，使用ChromeOptions设置了一些启动的参数（具体见代码注释）。调用方法是：启动浏览器，使用浏览器访问百度，找到百度的搜索按钮并点击。

使用Pycharm找到 selenium -> webdriver -> chrome -> options.py，打开文件可以看到里面有个Options的类，专门服务于chrome，同时定义了一些方法去添加启动参数。这里不做深究，因为里面没什么东西。

selenium -> webdriver -> chrome -> webdriver.py 解读

启动测试脚本，在 selenium -> webdriver -> chrome -> webdriver.py 的int函数中打上断点。我们一步步来看：

    if chrome_options:
        warnings.warn('use options instead of chrome_options',
                      DeprecationWarning, stacklevel=2)
        options = chrome_options
    
    if options is None:
        # desired_capabilities stays as passed in
        if desired_capabilities is None:
            desired_capabilities = self.create_options().to_capabilities()
    else:
        if desired_capabilities is None:
            desired_capabilities = options.to_capabilities()
        else:
            desired_capabilities.update(options.to_capabilities())
    
    # 首先是这一段，主要就是讲传进来的 options、desired_capabilities 和 chrome_options 通通转化成 desired_capabilities，我们来看下此时的desired_capabilities是什么样：
    {'browserName': 'chrome',
     'version': '',
     'platform': 'ANY',
     'goog:chromeOptions': {'prefs': {'profile.default_content_settings.popups': 0,
       'download.default_directory': 'C:\\\\D'},
      'extensions': [],
      'args': ['--ignore-certificate-errors', '--no-sandbox', '--disable-gpu']}}
    
    # 前三项 browserName、version、platform都是在ChromeOptions中给添加进去
    # 后面的 goog:chromeOptions 是ChromeOptions的类属性
    # {'prefs': {'profile.default_content_settings.popups': 0,
       'download.default_directory': 'C:\\\\D'}是我们测试脚本中添加进来的
    # 最后的['--ignore-certificate-errors', '--no-sandbox', '--disable-gpu']也是来自脚本

    self.service = Service(
        executable_path,
        port=port,
        service_args=service_args,
        log_path=service_log_path)
    self.service.start()
    
    # 打开Service这个类（selenium -> webdriver -> chrome -> service.py），可以看到它初始化了selenium -> webdriver -> common -> service.py中的Service这个类。
    # 同时self.service.start()调用的也是Service中的start方法，我们看看这个方法里面都是什么：
    
    def start(self):
        """
        Starts the Service.
        """
        try:
            cmd = [self.path]
            cmd.extend(self.command_line_args())
            self.process = subprocess.Popen(cmd, env=self.env,
                                            close_fds=platform.system() != 'Windows',
                                            stdout=self.log_file,
                                            stderr=self.log_file,
                                            stdin=PIPE)
        except：
        ...
        ...
        ...
    
    # 首先第一行是一个path，这个path是chromedriver的执行path，当前我们没有指定，默认值是字符串 chromedriver
    # 第二行command_line_args是什么东西呢？在 selenium -> webdriver -> chrome -> service.py中实现了它，实际上就是给了一个 --port 的参数，然后放到父类Service的start方法中组合起来。
    # 最后这个cmd得到的是什么样子的：['chromedriver', '--port=8034']
    # 可以看到start方法最终使用subprocess这个库去启动了chromedriver，并让他监听在8034这个端口上（8034是随机的，每次启动可能都不一样，这一块下次再说。）

接着往下看，你会看到下面这样的一段代码：

    try:
        RemoteWebDriver.__init__(
            self,
            command_executor=ChromeRemoteConnection(
                remote_server_addr=self.service.service_url,
                keep_alive=keep_alive),
            desired_capabilities=desired_capabilities)
    except Exception:
        self.quit()
        raise

这里我们慢慢拆开看，首先我们可以知道：

- RemoteWebDriver 类接受两个参数： command_executor 和 desired_capabilities
- command_executor 是ChromeRemoteConnection的实例，并接受两个参数 remote_server_addr 和 keep_alive
- remote_server_addr 这个参数对应的值是self.service.service_url，这个是个什么东西？打印之后发现是一个url：http://localhost:8034

selenium -> webdriver -> chrome -> remote_connection.py 和 webdriver.py 解读

接着上一节，我们说 ChromeRemoteConnection 要开始初始化了，进入到  remote_connection.py 查看这个类：

    class ChromeRemoteConnection(RemoteConnection):
    
        def __init__(self, remote_server_addr, keep_alive=True):
            RemoteConnection.__init__(self, remote_server_addr, keep_alive)
            self._commands["launchApp"] = ('POST', '/session/$sessionId/chromium/launch_app')
            self._commands["setNetworkConditions"] = ('POST', '/session/$sessionId/chromium/network_conditions')
            self._commands["getNetworkConditions"] = ('GET', '/session/$sessionId/chromium/network_conditions')
            self._commands['executeCdpCommand'] = ('POST', '/session/$sessionId/goog/cdp/execute')

可以发现只有几行代码，并且只是将 remote_server_addr, keep_alive 传递给了父类 RemoteConnection，然后在一个叫 _commands 的实例属性上添加了一些tuple。我们单步进入RemoteConnection 这个类（这里我截取了最重要的一部分）：

    # selenium.webdriver.remote.remote_connection
    class RemoteConnection(object):
        ...
        ...
        ...
        
        def __init__(self, remote_server_addr, keep_alive=False, resolve_ip=True):
            ...
            ...
            ...
            self._url = remote_server_addr  # init方法中第一件事是初始化了chromedriver监听端口的url：http://127.0.0.1:8034
            if keep_alive:
                self._conn = urllib3.PoolManager(timeout=self._timeout)  # 使用PoolManager初始化了一个连接池用来后续向的url发起请求
            self._commands = {  # 初始化commans的mapping
                Command.STATUS: ('GET', '/status'),
                Command.NEW_SESSION: ('POST', '/session'),
                Command.GET_ALL_SESSIONS: ('GET', '/sessions'),
                ...
                ...
                ...
            }
        

可以看到在 RemoteConnection 中使用 urllib3的PoolManager进行了初始化。然后下面就是非常长的self._commands的赋值。我们单步跳出init方法。

ChromeRemoteConnection 初始化完成之后被赋值给了command_executor，接下来单步进入到 RemoteWebDriver 的init方法中：

    # selenium.webdriver.remote.webdriver
    def __init__(self, command_executor='http://127.0.0.1:4444/wd/hub',
                     desired_capabilities=None, browser_profile=None, proxy=None,
                     keep_alive=False, file_detector=None, options=None):
        capabilities = {}
        if options is not None:  # 如果options存在则将options转换成capabilities
            capabilities = options.to_capabilities()
        if desired_capabilities is not None:  # capabilities必须是一个dict类型
            if not isinstance(desired_capabilities, dict):
                raise WebDriverException("Desired Capabilities must be a dictionary")
            else:
                capabilities.update(desired_capabilities)
        if proxy is not None:  # 判断proxy是否存在
            warnings.warn("Please use FirefoxOptions to set proxy",
                          DeprecationWarning, stacklevel=2)
            proxy.add_to_capabilities(capabilities)
        self.command_executor = command_executor  # 将 RemoteConnection 的实例赋值给command_executor
        if type(self.command_executor) is bytes or isinstance(self.command_executor, str):  # 判断command_executor的类型，如果是字符串或bytes，则使用RemoteConnection加载
            self.command_executor = RemoteConnection(command_executor, keep_alive=keep_alive)
        self._is_remote = True
        self.session_id = None
        self.capabilities = {}
        self.error_handler = ErrorHandler()
        self.start_client()
        if browser_profile is not None:
            warnings.warn("Please use FirefoxOptions to set browser profile",
                          DeprecationWarning, stacklevel=2)
        self.start_session(capabilities, browser_profile)
        self._switch_to = SwitchTo(self)
        self._mobile = Mobile(self)
        self.file_detector = file_detector or LocalFileDetector()

这一段我捡最重要的说，在这个类中：

- 将RemoteConnection 的实例赋值给command_executor
- 调用start_session这个方法，那我们看看start_session这个方法里面，单步进入（见下方代码）
```
# selenium.webdriver.remote.webdriver
def start_session(self, capabilities, browser_profile=None):
    """
    Creates a new session with the desired capabilities.
    """
    ...
    ...
    ...
    w3c_caps = _make_w3c_caps(capabilities)  # 使用w3c规范capabilities
    parameters = {"capabilities": w3c_caps,
                  "desiredCapabilities": capabilities}  # 不是很明白为什么使用w3c封装后，再加入desiredCapabilities，待研究
    response = self.execute(Command.NEW_SESSION, parameters)  # 发出打开浏览器的请求
    if 'sessionId' not in response:
        response = response['value']
    self.session_id = response['sessionId']  # 启动浏览器后获得chromedriver返回的sessionId
    ...
    ...
    ...
```
在start_session中，做了这么几件事：

- 用_make_w3c_caps这个方法将我们我们初始化传递进来的参数进行了封装，使用w3c的标准剔除了capabilities当中不符合规范的字段（这个还需要再研究，为什么要用w3c的这个标准）
- 调用execute方法发出打开浏览器的请求
- 从返回的reponse中取出sessionId并赋值给属性session_id



接下来我们单步进入exexute方法：
```
# selenium.webdriver.remote.webdriver
def execute(self, driver_command, params=None):
    """
    Sends a command to be executed by a command.CommandExecutor.
    """
    if self.session_id is not None:  # 初始化时session_id为None，浏览器打开后接下来session_id不为None
        if not params:
            params = {'sessionId': self.session_id}
        elif 'sessionId' not in params:
            params['sessionId'] = self.session_id   # 最终将session_id加入到params中

    params = self._wrap_value(params)  # 一个递归函数（具体作用待看）
    response = self.command_executor.execute(driver_command, params)  # 调用ChromeRemoteConnection的execute函数
    if response:
        self.error_handler.check_response(response)
        # 可以看到在_unwrap_value方法中会根据返回的响应去判断是open一个broswer
        # 还是类似find element，如果是find element这种操作对应的响应会返回WebElement对象
        response['value'] = self._unwrap_value(response.get('value', None))  
        return response
    # If the server doesn't send a response, assume the command was
    # a success
    return {'success': 0, 'value': None, 'sessionId': self.session_id}
```
在execute方法中：

- 首先判断session_id是不是None，因为我们还没有启动浏览器，所以这个session_id目前没有值
- 然后使用_wrap_value方法对params进行包装（这是个递归函数），目前还不知道为什么要这样做
- 接下来就是调用command_executor的execute方法进行浏览器打开的请求发送



接下来就是直接与webdriver打交道的这个类了。接着上面，我们单步进入execute方法：

    # selenium.webdriver.chrome.remote_connection.ChromeRemoteConnection
    def execute(self, command, params):
        """
        Send a command to the remote server.
        """
        command_info = self._commands[command]  # 通过命令名称找到http请求实体
        assert command_info is not None, 'Unrecognised command %s' % command
        path = string.Template(command_info[1]).substitute(params)
        if hasattr(self, 'w3c') and self.w3c and isinstance(params, dict) and 'sessionId' in params:
            del params['sessionId']
        data = utils.dump_json(params)  # 将数据转换成json格式字符串
        url = '%s%s' % (self._url, path)  # http://127.0.0.1:54817/session
        return self._request(command_info[0], url, body=data)  # 发送请求

继续分析：

- 首先通过command找到要发送的命令的结构
    ```
    command
    Out[8]: 'newSession'
    command_info
    Out[9]: ('POST', '/session')
    ```
  - 这里我将变量的内容打印了出来。可以看到command是字符串newSession，从self._command中取出来的是一个tuple，内容是('POST', '/session')。表明此次请求是一个POST请求，请求url为/session
- 接着是
      string.Template(command_info[1]).substitute(params)
      # 这句话是什么意思呢？看看下面这个例子。
    ```
    >>> from string import Template
    >>> template_string = '$who likes $what'
    >>> s = Template(template_string)
    >>> d = {'who': 'Tim', 'what': 'kung pao'}
    >>> s.substitute(d)
    'Tim likes kung pao'
    ```

- 然后将数据转换成json字符串
- 最后调用request方法发送请求(看下请求参数)
    ```
      command_info[0]
      Out[16]: 'POST'
      url
      Out[17]: 'http://127.0.0.1:54817/session'
      data
      Out[18]: '{"capabilities": {"firstMatch": [{}], "alwaysMatch": {"browserName": "chrome", "platformName": "any", "goog:chromeOptions": {"prefs": {"profile.default_content_settings.popups": 0, "download.default_directory": "/Users/vince/Desktop/ta"}, "extensions": [], "args": ["--ignore-certificate-errors", "--no-sandbox", "--disable-gpu"]}}}, "desiredCapabilities": {"browserName": "chrome", "version": "", "platform": "ANY", "goog:chromeOptions": {"prefs": {"profile.default_content_settings.popups": 0, "download.default_directory": "/Users/vince/Desktop/ta"}, "extensions": [], "args": ["--ignore-certificate-errors", "--no-sandbox", "--disable-gpu"]}}}'
        
      # 到这里为止仍然还没有将这个请求发送出去。我们先不看_request方法中写了什么，分析一下当前的情况。
      chromedriver已经监听在了54817这个port上，然后我们也知道了请求方法，请求地址和请求参数。那我们能不能自己构造一个请求呢？
      
      url = 'http://127.0.0.1:54817/session'
      data = '{"capabilities": {"firstMatch": [{}], "alwaysMatch": {"browserName": "chrome", "platformName": "any", "goog:chromeOptions": {"prefs": {"profile.default_content_settings.popups": 0, "download.default_directory": "/Users/vince/Desktop/ta"}, "extensions": [], "args": ["--ignore-certificate-errors", "--no-sandbox", "--disable-gpu"]}}}, "desiredCapabilities": {"browserName": "chrome", "version": "", "platform": "ANY", "goog:chromeOptions": {"prefs": {"profile.default_content_settings.popups": 0, "download.default_directory": "/Users/vince/Desktop/ta"}, "extensions": [], "args": ["--ignore-certificate-errors", "--no-sandbox", "--disable-gpu"]}}}'
      resp = requests.post(url, data=data).content
      
      # 在Pychram的调试模式下输入上面的内容（当然你们的情况可能跟我不一样，端口不一样，请求参数也不同），就能看到chrome被打开了。
    ```
    
    
接下来我们看看_request中的内容
```
def _request(self, method, url, body=None):
    """
    Send an HTTP request to the remote server.
    """
    ...
    parsed_url = parse.urlparse(url)
    headers = self.get_remote_connection_headers(parsed_url, self.keep_alive)
		...
    if self.keep_alive:
        resp = self._conn.request(method, url, body=body, headers=headers)
        statuscode = resp.status
    else:
				...
    data = resp.data.decode('UTF-8')
    try:
    	  ...
      	return data
    finally:
        LOGGER.debug("Finished Request")
        resp.close()
```

在_request方法捡最重要的说一下：

- parse.urlparse(url) 解析url
    ```
    url
    Out[30]: 'http://127.0.0.1:54817/session'
    ```

- get_remote_connection_headers(parsed_url, self.keep_alive) 构造请求头
    ```
    headers
    Out[31]: 
    {'Accept': 'application/json',
    'Content-Type': 'application/json;charset=UTF-8',
    'User-Agent': 'selenium/3.141.0 (python mac)',
    'Connection': 'keep-alive'}
    ```

- self._conn.request(method, url, body=body, headers=headers) 发送请求请返回response

到这里为止chrome的启动流程就分析完了，中间省略了一下地方，response返回之后也省略了一些。但是并不影响整个启动流程的讲解。接下来可能会分析下selenium中其他模块，比如很重要的common模块里面就包含了很多在UI自动化时需要用到的东西。

最后放一张自己画的selenium的启动架构简图：
![tvPGmn.jpg](https://s1.ax1x.com/2020/06/13/tvPGmn.jpg)