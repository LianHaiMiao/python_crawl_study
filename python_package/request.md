
# python爬虫学习课程（一）requests库

## 0.说明

该笔记本主要根据 北京理工大学嵩天老师的课程 [Python网络爬虫与信息提取](http://www.icourse163.org/course/BIT-1001870001) 记录而成

request库的官方文档请参考 [request](http://www.python-requests.org/en/master/)

## 1.学习使用urllib库


```python
from urllib import request

url = 'http://www.csdn.net'
# 使用headers可以设置请求头，包装头部信息，让爬虫更加逼真2333
headers = ''
req = request.Request(url)
page = request.urlopen(req)
content = page.read()
# 这时候输出的是一大串乱七八糟毫无格式可言的bytes类型数据。我们需要美化一下，所以我们使用decode()方法来转换成str类型
# print(html)


# 这时候输出的就是稍微好看一丢丢的数据类型格式了。
html = content.decode('utf-8')
# print(html)
```

目前，我们已经能够简单的获取Html页面了，现实中，我们需要设置http请求的，使得我们发起的请求更像“人”


```python
import urllib

# 设置headers

#用我本地的api接口进行测试
url = 'http://127.0.0.1/api/admin/login'
# 登录信息
login_data = [('account', 'admin'), ('password', 'admin')]
# 将登入数据设置为str类型
pre_login_data = urllib.parse.urlencode(login_data)
# 将登入数据设置为二进制形式
bi_login_data = pre_login_data.encode('utf-8')

# 配置爬虫访问url
request = urllib.request.Request(url)
# 配置爬虫访问头部headers
request.add_header('User-Agent', 'Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:53.0) Gecko/20100101 Firefox/53.0')
request.add_header('Referer', 'http://127.0.0.1/login/')

response = urllib.request.urlopen(request, data=bi_login_data)
page = response.read()

# print(page)
```

## 2.学习使用Requests库

**requests库的7个方法:**

requests.request()    构造一个请求，支撑以下各方法的基础方法

requests.get()    获取HTML网页的主要方法，对应于HTTP的GET

requests.head()    获取HTML网页头信息的方法， 对应于HTTP的HEAD

requests.post()    向HTML网页提交POST请求， 对应于HTTP的POST

requests.put()    向HTML网页提交PUT请求， 对应于HTTP的PUT

requests.patch()    向HTML网页提交局部修改请求， 对应于HTTP的PATCH

requests.delete()    向HTML网页提交删除请求， 对应于HTTP的DELETE


```python
import requests
# requests.get方法示范
r = requests.get('http://www.baidu.com')
print(r.status_code)
# 200
r.encoding = 'utf-8'
# 返回网页内容 无css和js
r.text
```

** Response 对象的属性 **

r.status_code HTTP请求的返回状态， 200表示连接成功，404表示失败

r.text HTTP响应内容的字符串形式，即，url对应的页面内容

r.encoding 从HTTP header中猜测的响应内容编码方式

r.apparent_encoding 从内容中分析出的响应内容编码方式（备选编码方式）

*附注： 当encoding无法解析的时候，我们使用apparent_encoding*

r.content HTTP响应内容的二进制形式


**理解requests库的异常**

requests.ConnectionError 网络连接错误异常，如DNS查询失败、拒绝连接等

requests.HTTPError HTTP错误异常

requests.URLRequired URL缺失异常

requests.TooManyRedirects 超过最大重定向次数、产生重定向异常

requests.ConnectTimeout 连接远程服务器超时异常

requests.Timeout 请求URL超时，产生超时异常

*这里需要注意啦～response对象也会返回异常哦～*

r.raise_for_status() 如果不是200，产生异常requests.HTTPError


```python
import requests

# 爬虫的通用代码框架
def getHTMLText(url):
    try:
        r = requests.get(url, timeout=30)
        r.raise_for_status() #如果状态不是200，引发HTTPError异常
        r.encoding = r.apparent_encoding
        return r.text
    except:
        return "产生异常\n"
    
if __name__ == "__main__":
    url = "http://www.baidu.com"
    print(getHTMLText(url))
```

### requests.request(method, url, **kwargs) 函数详解

method: 七种方法(GET, POST, HEAD, PUT, PATCH, DELETE, OPTIONS)

url: url链接

**kwargs: 控制访问的参数，均为可选项

1. params: 字典或字节序列， 作为参数增加到url中

例子：

kv = {'key1': 'value1', 'key2': 'value2'}

r = requests.request('GET', 'http://python123.io/ws', params=kv)

print(r.url)

> 结果：http://python123.io/ws?key1=value1&key2=value2

2. data: 字典、字节序列或文件对象， 作为Request的内容， 其实就是POST请求中的添加的数据


3. json: JSON格式的数据， 作为Requset的内容

例子：

kv = {'key': 'value'}

r = requests.request('POST', 'http://python123.io/ws', json=kv)

4. headers: 字典， 使用这个字段来定制访问url的协议头

5. cookies: 字典或CookieJar, Request中的cookie

6. auth: 元组， 支持HTTP认证功能

7. files: 字典类型， 传输文件

例子：

fs = {'file': open('data.xls', 'rb')}

r = requests.request('POST', 'http://python123.io/ws', files=fs)

8. timeout: 设定超时时间， 秒为单位

9. proxies: 字典类型， 设定访问代理服务器， 可以增加登录认证

例子：

使用了这个字段，可以有效的防止爬虫的逆追踪，还有禁止ip的话，也不会被禁止了。

pxs = {'http': 'http://user:pass@10.10.10.1:1234', 'https': 'https://10.10.10.1:4321'}

r = requests.request('GET', 'http://python123.io/ws', proxies=pxs)

10. allow_redirects: True/False, 默认为True, 重定向开关

11. stream: True/False, 默认为True, 获取内容立即下载开关

12. verify: True/False, 默认为True, 认证SSL证书开关

13. cert: 本地SSL证书路径

### requests.get(url, params=None , **kwargs) 函数详解

url: url链接

params: url中的额外参数，字典或字节流格式，可选

**kwargs: 12个控制访问参数

### requests.post(url, data=None , json=None, **kwargs) 函数详解

url: url链接

data: 字典、字节序列或文件对象， 作为Request的内容

json: JSON格式的数据， Request的内容

**kwargs: 11个控制访问参数

### requests.put(url, data=None, **kwargs) 函数详解

url: url链接

data: 字典、字节序列或文件对象， 作为Request的内容

**kwargs: 12个控制访问参数


### requests.delete(url, **kwargs) 函数详解

url: url链接

**kwargs: 13个控制访问参数



## 3.Robots协议

Robots Exclusion Standard 网络爬虫排除标准

作用： 网站告知网络爬虫哪些页面可以抓取，哪些不行

形式： 在网站的根目录下放置一个robots.txt文件，写明哪些目录可以爬去，哪些不行。

**看一看JD的robots.txt文件**

```
User-agent: * 
    # 任何爬虫都不能访问以/?开头的路径
    Disallow: /?* 
    
    # 任何爬虫都不允许访问/pop/*.html
    Disallow: /pop/*.html 
    
    # 任何爬虫都不允访问/pinpai/*.html?*  这个形式的通配符
    Disallow: /pinpai/*.html?* 
    
    # 以下四种爬虫不允许访问JD的任何资源
    User-agent: EtaoSpider 
    Disallow: / 
    User-agent: HuihuiSpider 
    Disallow: / 
    User-agent: GwdangSpider 
    Disallow: / 
    User-agent: WochachaSpider 
    Disallow: /    
```

**Robots协议基本语法**

```
    User-agent: *
    Disallow: /
```

## 4.实例部分

### 4.1京东商品页面的爬取


```python
import requests

url = "https://item.jd.com/5182580.html"
try:
    r = requests.get(url)
    r.raise_for_status()
    r.encoding = r.apparent_encoding
    print(r.text[:1000])
except:
    print('爬取失败！')
```

### 4.2亚马逊商品页面的爬取


```python
import requests

url = "https://www.amazon.cn/gp/product/B01DVYESFY"

try:
    # 设置一下User-Agent
    kv = {'user-agent': 'Mozilla/5.0'}
    r = requests.get(url, headers = kv)
    r.raise_for_status()
    r.encoding = r.apparent_encoding
    print(r.text[20000:21000])
except:
    print('爬取失败！')
```

### 4.3百度/360搜索关键词提交


```python
import requests

kv = {'wd': 'Python'}
url = 'http://www.baidu.com/s'
try:
    r = requests.get(url, params=kv)
    print(r.request.url)
    r.raise_for_status
    print(r.text)
except:
    print('爬取失败！')
# 查看长度
#len(r.text)
```

### 4.4网络图片的爬取和存储


```python
import requests
import os

url = 'http://image.nationalgeographic.com.cn/2017/0605/20170605040630158.jpg'
root = './pics/'
path = root + url.split('/')[-1]
try:
    # 如果文件目录不存在就创建
    if not os.path.exists(root):
        os.mkdir(root)
    # 如果文件不存在就执行
    if not os.path.exists(path):
        r = requests.get(url)
        with open(path, 'wb') as f:
            f.write(r.content)
            f.close()
            print('文件保存成功')
    else:
        print('文件已存在')
except:
    print('爬取失败！')
```

### 4.5IP地址归属地的自动查询


```python
import requests

url = 'http://m.ip138.com/ip.asp'
kv = {'ip': '59.52.101.77'}
try:
    r = requests.get(url, params=kv)
    r.raise_for_status
    print(r.text[-500:])
except:
    print('爬取失败！')
```

