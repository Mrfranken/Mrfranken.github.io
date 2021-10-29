---
title: 如何在python中手动启动chromederiver
date: 2020-02-09
tags: 
- 测试 
- selenium
category: 测试
---

## 如何在python中手动启动chromederiver

#### 启动chrmedriver
```
import subprocess
import os
import platform

cmd = ['chromedriver', '--port=54824']
subprocess.Popen(cmd, env=os.environ,
                 close_fds=platform.system() != 'Windows',
                 stdout=self.log_file,
                 stderr=self.log_file,
                 stdin=PIPE)
```

#### from selenium.webdriver.remote.remote_connection import RemoteConnection line 397
```
self._conn = urllib3.PoolManager(timeout=self._timeout)
method = 'POST'
url = 'http://127.0.0.1:54824/session'
body = '{"capabilities": {"firstMatch": [{}], "alwaysMatch": {"browserName": "chrome", "platformName": "any", "goog:chromeOptions": {"args": [], "extensions": []}}}, "desiredCapabilities": {"version": "", "browserName": "chrome", "platform": "ANY", "goog:chromeOptions": {"args": [], "extensions": []}}}'
headers = {
    'Accept': 'application/json',
    'Connection': 'keep-alive',
    'Content-Type': 'application/json;charset=UTF-8',
    'User-Agent': 'selenium/3.141.0 (python windows)'
}
resp = self._conn.request(method, url, body=body, headers=headers)

```