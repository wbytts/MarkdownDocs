# 一些概念

## HTTP相关知识

自己查

了解以下概念：

- 发送网络请求（需要带一定的数据给服务器，不带也可以）
  - 请求头

- 响应
  - 响应状态码

- DNS域名解析
- 

## 爬虫基本概念

使用代码模拟用户，批量发送网络请求，批量的获取数据

爬虫的价值：数据的价值

数据分析

流量

合法性：灰色产业，没有规定爬虫合法，也没有规定爬虫不合法

爬虫只能爬取用户正常能访问到的数据



爬虫的分类：

- 通用爬虫
  - 如搜索引擎
  - 优势：开放性、速度快
  - 劣势：目标不明确
  - 返回内容：很多是用户不需要的
  - 不清楚用户的具体需求
- 聚焦爬虫
  - 目标明确
  - 对用户的需求非常精准
  - 返回的内容很固定
- 增量式爬虫
  - 例如翻页
- 深度爬虫
  - 静态数据、动态数据等



robots：是否允许其他爬虫爬取某些内容（网站指定的协议）

- 只是一份协议，尊不遵守看你自己

在网站后面接上 `robots.txt` 即可查看内容

# urllib 库

## get请求

```python
import urllib.request

url = "http://www.baidu.com"
# get的请求
# response：Http响应的对象（响应体）
response = urllib.request.urlopen(url)
print(response)
# 读取内容
data = response.read()
print(data)
# 默认读取的是byte，可以转码
str_data = data.decode('utf-8')
print(str_data)
```

## url拼接参数有中文

```python
import urllib.request
import urllib.parse
import string


def get_method_params(wd):
    url = "http://www.baidu.com/s?wd="
    # 拼接字符串
    final_url = url + wd
    # 将包含汉字的网址进行转义
    encode_new_url = urllib.parse.quote(final_url, safe=string.printable)
    # 发送网络请求
    response = urllib.request.urlopen(encode_new_url)
    print(response.read().decode("utf-8"))


get_method_params("美女")
```

demo2：

```python
import urllib.request
import urllib.parse
import string


def get_params():
    url = "http://www.baidu.com/s?w"

    # 使用字典对象加入请求参数
    params = {
        "wd": "美女",
        "key": "zhang",
        "value": "san"
    }

    str_params = urllib.parse.urlencode(params)
    print(str_params)

    final_url = url + str_params
    # 将带有中文的url转义
    encode_url = urllib.parse.quote(final_url, safe=string.printable)

    response = urllib.request.urlopen(encode_url)
    data = response.read().decode("utf-8")
    print(data)


get_params()
```

## 请求头 User-Agent

每次发请求都会带着的一个参数，代表着当前发起访问的身份

```python
import urllib.request


def load_baidu():
    url = "http://www.baidu.com"

    request_headers = {
        "User-Agent": "User-Agent,Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/78.0.3904.108 Safari/537.36"
    }

    # 创建请求对象
    request = urllib.request.Request(url, headers=request_headers)
    # 请求网络数据
    response = urllib.request.urlopen(request)

    # 请求头
    print(request.headers)

    # 响应头
    # print(response.headers)

    print(response.read().decode("utf-8"))


load_baidu()
```

设置随机请求头：

```python
import urllib.request
import urllib.parse
import string
import random


def get_random_user_agent():
    import random
    user_agent_list = [
        "User-Agent,Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/78.0.3904.108 Safari/537.36",
        "User-Agent,Mozilla/5.0 (Macintosh; U; Intel Mac OS X 10_6_8; en-us) AppleWebKit/534.50 (KHTML, like Gecko) Version/5.1 Safari/534.50",
        "User-Agent,Mozilla/5.0 (Windows; U; Windows NT 6.1; en-us) AppleWebKit/534.50 (KHTML, like Gecko) Version/5.1 Safari/534.50",
        "User-Agent,Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Trident/5.0;",
        "User-Agent,Mozilla/4.0 (compatible; MSIE 8.0; Windows NT 6.0; Trident/4.0)",
        "User-Agent,Mozilla/4.0 (compatible; MSIE 7.0; Windows NT 6.0)",
        "User-Agent, Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1)",
        "User-Agent, Mozilla/5.0 (Macintosh; Intel Mac OS X 10.6; rv,2.0.1) Gecko/20100101 Firefox/4.0.1",
        "User-Agent,Mozilla/5.0 (Windows NT 6.1; rv,2.0.1) Gecko/20100101 Firefox/4.0.1",
        "User-Agent,Opera/9.80 (Macintosh; Intel Mac OS X 10.6.8; U; en) Presto/2.8.131 Version/11.11",
        "User-Agent,Opera/9.80 (Windows NT 6.1; U; en) Presto/2.8.131 Version/11.11",
        "User-Agent, Mozilla/5.0 (Macintosh; Intel Mac OS X 10_7_0) AppleWebKit/535.11 (KHTML, like Gecko) Chrome/17.0.963.56 Safari/535.11",
        "User-Agent, Mozilla/4.0 (compatible; MSIE 7.0; Windows NT 5.1; Maxthon 2.0)",
        "User-Agent, Mozilla/4.0 (compatible; MSIE 7.0; Windows NT 5.1; TencentTraveler 4.0)",
        "User-Agent, Mozilla/4.0 (compatible; MSIE 7.0; Windows NT 5.1)",
        "User-Agent, Mozilla/4.0 (compatible; MSIE 7.0; Windows NT 5.1; The World)",
        "User-Agent, Mozilla/4.0 (compatible; MSIE 7.0; Windows NT 5.1; Trident/4.0; SE 2.X MetaSr 1.0; SE 2.X MetaSr 1.0; .NET CLR 2.0.50727; SE 2.X MetaSr 1.0)",
        "User-Agent, Mozilla/4.0 (compatible; MSIE 7.0; Windows NT 5.1; 360SE)",
        "User-Agent, Mozilla/4.0 (compatible; MSIE 7.0; Windows NT 5.1; Avant Browser)",
        "User-Agent, Mozilla/4.0 (compatible; MSIE 7.0; Windows NT 5.1)",
        "User-Agent,Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/55.0.2883.87 UBrowser/6.2.4094.1 Safari/537.36",
        "User-Agent,Mozilla/5.0 (iPhone; U; CPU iPhone OS 4_3_3 like Mac OS X; en-us) AppleWebKit/533.17.9 (KHTML, like Gecko) Version/5.0.2 Mobile/8J2 Safari/6533.18.5",
        "User-Agent,Mozilla/5.0 (iPod; U; CPU iPhone OS 4_3_3 like Mac OS X; en-us) AppleWebKit/533.17.9 (KHTML, like Gecko) Version/5.0.2 Mobile/8J2 Safari/6533.18.5",
        "User-Agent,Mozilla/5.0 (iPad; U; CPU OS 4_3_3 like Mac OS X; en-us) AppleWebKit/533.17.9 (KHTML, like Gecko) Version/5.0.2 Mobile/8J2 Safari/6533.18.5",
        "User-Agent, Mozilla/5.0 (Linux; U; Android 2.3.7; en-us; Nexus One Build/FRF91) AppleWebKit/533.1 (KHTML, like Gecko) Version/4.0 Mobile Safari/533.1",
        "User-Agent, MQQBrowser/26 Mozilla/5.0 (Linux; U; Android 2.3.7; zh-cn; MB200 Build/GRJ22; CyanogenMod-7) AppleWebKit/533.1 (KHTML, like Gecko) Version/4.0 Mobile Safari/533.1",
        "User-Agent, Opera/9.80 (Android 2.3.4; Linux; Opera Mobi/build-1107180945; U; en-GB) Presto/2.8.149 Version/11.10",
        "User-Agent, Mozilla/5.0 (Linux; U; Android 3.0; en-us; Xoom Build/HRI39) AppleWebKit/534.13 (KHTML, like Gecko) Version/4.0 Safari/534.13",
        "User-Agent, Mozilla/5.0 (BlackBerry; U; BlackBerry 9800; en) AppleWebKit/534.1+ (KHTML, like Gecko) Version/6.0.0.337 Mobile Safari/534.1+",
        "User-Agent, Mozilla/5.0 (hp-tablet; Linux; hpwOS/3.0.0; U; en-US) AppleWebKit/534.6 (KHTML, like Gecko) wOSBrowser/233.70 Safari/534.6 TouchPad/1.0",
        "User-Agent, Mozilla/5.0 (SymbianOS/9.4; Series60/5.0 NokiaN97-1/20.0.019; Profile/MIDP-2.1 Configuration/CLDC-1.1) AppleWebKit/525 (KHTML, like Gecko) BrowserNG/7.1.18124",
        "User-Agent, Mozilla/5.0 (compatible; MSIE 9.0; Windows Phone OS 7.5; Trident/5.0; IEMobile/9.0; HTC; Titan)",
        "User-Agent, UCWEB7.0.2.37/28/999",
        "User-Agent, NOKIA5700/ UCWEB7.0.2.37/28/999",
        "User-Agent, Openwave/ UCWEB7.0.2.37/28/999",
        "User-Agent, Mozilla/4.0 (compatible; MSIE 6.0; ) Opera/UCWEB7.0.2.37/28/999",
    ]
    random_user_agent = random.choice(user_agent_list)
    return random_user_agent


url = "http://www.baidu.com"
request = urllib.request.Request(url)
# 添加请求头信息
request.add_header("User-Agent", get_random_user_agent())
# 请求数据
response = urllib.request.urlopen(request)
# print(response.read().decode("utf-8"))
# 获取请求头信息，注意这里的agent小写
print(request.get_header("User-agent"))
```

## Handler 处理器



```python
import urllib.request


def handler_openner():
    # urllib.request.urlopen()
    # 点进去看urlopen的源码可以知道，系统的urlopen并没有添加代理的功能，所以需要我们自定义这个功能

    # 安全套接层：SSL，第三方的CA数字证书
    # http与https的一个区别，http是80端口，https是443端口

    # urlopen为什么可以请求数据
    # 根据源码可以看出，是由于 openner，而openner又由handler而来

    url = "http://www.baidu.com"
    # 创建自己的处理器
    handler = urllib.request.HTTPHandler()
    # 创建自己的openner
    openner = urllib.request.build_opener(handler)
    # 用openner去请求
    response = openner.open(url)
    print(response.read().decode("utf-8"))


handler_openner()
```

## proxy Handler

```python
import urllib.request


def create_proxy_handler():
    url = "http://www.baidu.com"

    proxy = {
        # 免费的写法
        "http": "http://36.27.28.215:9999"
    }

    # 代理的处理器
    proxy_handler = urllib.request.ProxyHandler(proxy)

    # 创建自己的openner
    openner = urllib.request.build_opener(proxy_handler)
    # 拿着代理ip去发送请求
    response = openner.open(url)

    print(response.read().decode("utf-8"))





create_proxy_handler()
```

## Cookie

```python
import urllib.request

url = "需要cookie验证才能访问的链接"

headers = {
    'User-Agent': 'User-Agent,Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/78.0.3904.108 Safari/537.36',
    "Cookie": 'xxx手动粘贴cookie到这里'
}

request = urllib.request.Request(url, headers=headers)

response = urllib.request.urlopen(request)

data = response.read()

with open("data.html", "wb") as f:
    f.write(data)
```

自动登录拿到Cookie：

```python
import urllib.request
import urllib.parse
from http import cookiejar

headers = {
    'User-Agent': 'User-Agent,Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/78.0.3904.108 Safari/537.36',
}

login_url = "..."
target_url = "https://i-beta.cnblogs.com/posts"

login_form_data = {
    "username": "xxx",
    "password": "xxx",
    # ......其他参数
}

# 转码
login_form_data = urllib.parse(login_form_data).encode("utf-8")

# 发送请求，拿到response里的cookie

cookie_jar = cookiejar.CookieJar()
# 定义有添加cookie功能的处理器
cookie_handler = urllib.request.HTTPCookieProcessor(cookie_jar)
# 根据处理器生成openner
openner = urllib.request.build_opener(cookie_handler)

# 带着参数，发送post请求
login_request = urllib.request.Request(login_url, headers=headers, data=login_form_data)
# 如果登录成功，cookiejar会自动保存cookie
openner.open(login_request)

target_request = urllib.request.Request(target_url, headers=headers)
response = openner.open(target_request)
data = response.read()
print(data)
```



# requests 库

# 网页文本解析

## 正则表达式

## xpath

## beautifulsoup4



# 数据保存

## JSON

字符串：loads、dumps

文件对象：load、dump

### 字符串和dic、list转换

```python
import json

data = '里面是标准的json字符串'
list_data = json.loads(data)
```

## 文件对象和dic、list转换



## CSV

## MongoDB

Python操作MongoDB   pymongo

## Redis

Python操作Redis

## MySQL

Python操作MySQL

