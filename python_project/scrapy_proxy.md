# scrapy　使用代理

## 0.说明

因为项目需求，最近在爬取豆瓣的用户信息，爬着爬着就变成了 IP 异常.为此只能尝试使用代理池,然后免费的代理,真的打扰了...

踩了几个坑,最后决定使用 收费代理,调取API,获得代理信息.

效果还不错,于是准备把自己构建代理中间件的过程记录下来.

## 项目简介

**项目目的：**

在scrapy框架中如何根据 **付费代理** 提供的API接口去构建代理池.


**项目说明：**

很多网站,你爬取地时候,会对你的 IP 进行检测,一段时间内频繁访问,会要求你登录.或者证明自己不是机器人...

所以,为了避免陷入这样的情况,推荐几个做法

1. 使用user agent池，轮流选择之一来作为user agent。池中包含常见的浏览器的user agent(google一下一大堆) (这个效果不错,但是碰上那种,不讲道理直接封你IP的,也很倒霉)
2. 设置下载延迟(2或更高)。DOWNLOAD_DELAY = 2, 这个也真心不错.
3. 使用高度分布式的下载器(downloader)来绕过禁止(ban)，您就只需要专注分析处理页面。这样的例子有: Crawlera (这个付费的,没用过,不过看了API接口,还是很方便的,但是对于国内用户来说有点小贵)
4. 使用IP池 (免费的尽量别用了吧,因为大家都在用,所以花点小钱吧).

以上四点说明采集自 [Scrapy 官方文档之实践经验(Common Practices)](http://scrapy-chs.readthedocs.io/zh_CN/latest/topics/practices.html)


**本人购买的代理反馈结果：**

为了能够好好滴将爬虫进行下去,特意购买了代理.

调用代理API,反馈的结果如下所示:

```
{"ERRORCODE":"0","RESULT":[{"port":"33530","ip":"117.31.154.23"},{"port":"41762","ip":"171.211.53.36"},{"port":"49809","ip":"125.78.12.145"},{"port":"32762","ip":"182.42.38.170"},{"port":"24162","ip":"60.184.198.147"},{"port":"40991","ip":"171.12.176.99"},{"port":"33085","ip":"180.109.241.150"},{"port":"31207","ip":"115.237.235.238"},{"port":"44077","ip":"218.66.147.157"},{"port":"24174","ip":"113.121.246.41"}]}

```

可以看到,该 API 返回给我们的是一个 Json 串,解析之后,就是键值对的形式.



**构建代理池的思路:**

1. 调用代理API获取可以使用的IP,构建IP池

2. 验证该IP是否有效

3. 验证完IP有效,就使用该IP去完成接下来的代理任务.

4. 附加: 可以人为地增加代理IP的有效时长或者使用次数.(这里我们并没有增加这个功能,因为我碰到的爬取任务,并不需要这样...)

**注意事项：**

如果对scrapy库有疑惑可以查阅官方文档,另外对于新手,推荐使用 scrapy-cookbook 这份文档来进行入门

1. [scrapy官方文档](http://scrapy-chs.readthedocs.io/zh_CN/latest/intro/overview.html)
2. [Welcome to scrapy-cookbook](http://scrapy-cookbook.readthedocs.io/zh_CN/latest/)


## 项目部分函数的说明

> 我们会在 scrapy 框架中的 middlewares.py 文件中创建一个新的类 ProxyMiddleware ,使用这个类来完成代理操作. 同时,为了开启这个 ProxyMiddleware 中间件,需要在 setting.py 文件中做如下操作.


``` python
DOWNLOADER_MIDDLEWARES = {
    'douban.middlewares.ProxyMiddleware': 543,
    'scrapy.downloadermiddlewares.httpproxy.HttpProxyMiddleware': None,
}
```

各个函数的详细解释,都在代码中.

## scrapy框架使用代理API构建IP池

``` python
# -*- coding: utf-8 -*-
class ProxyMiddleware(object):

    def __init__(self):

        # 调用代理API获取IP
        self.get_url = "http://api.xdaili.cn/xdaili-api//greatRecharge/getGreatIp?spiderId=6d805b68a2f44f4eb783612b242875b3&orderno=MF201852280832SEXfl&returnType=2&count=5"

        # 我们使用站长工具来查看当前代理的 IP 地址.
        self.temp_url = "http://ip.chinaz.com/getip.aspx"
        
        # 我们构建的IP池,用于存储从代理API中获取的IP
        self.ip_list = []

        # 记录当前使用的 ip 是IP池中的哪一个
        self.ip_count = 0

        # 最大请求数
        self.max_count = 5

    def getIPData(self):
        """
            使用 requests 库去调用代理API获取IP,同时,将所有的 IP 存储到 IP 池  ip_list 中.
        """
        temp_data = requests.get(url=self.get_url).text
        # 清空IP池
        self.ip_list.clear()
        # 填充IP池(这里的处理方法对于不同的代理而言,使用的函数不一样,但是本质都是一样的, 取出IP,填充)
        for every_ip in json.loads(temp_data)["RESULT"]:
            self.ip_list.append({
                "ip": every_ip["ip"],
                "port": every_ip["port"]
            })

    def changeProxy(self, request):
        """
            更换当前的代理.
        """
        request.meta["proxy"] = "http://" + str(self.ip_list[self.ip_count-1]["ip"]) + ":" + str(self.ip_list[self.ip_count-1]["port"])


    def verifyIP(self):
        """
            因为 IP 不是 一定有用的,有时候得到的 IP 是无效的. 所以,我们需要先测试 IP 再使用.
            timeout = 3 用于设置延时,一旦超过延时就不用该 IP 了.
        """
        requests.get(url=self.temp_url, proxies={"http" : str(self.ip_list[self.ip_count-1]["ip"]) + ":" + str(self.ip_list[self.ip_count-1]["port"])}, timeout=3)


    # 判断这个ip是否被使用过
    def verifyAndChange(self, request):
        """
        :param request:
        :return:
        """
        try:
            # 先更换代理,再验证IP
            self.changeProxy(request)
            self.verifyIP()
        except:
            # IP 校验失败,我们更换 IP,如果 IP 池中的IP 都是无效的,那么我们更换 IP 池.
            if self.ip_count == 0 or self.ip_count == self.max_count:
                self.getIPData()
                self.ip_count = 1
            # IP 校验失败,但是 IP 池中还有 IP, 更换新IP
            self.ip_count += 1
            self.verifyAndChange(request)

    def process_request(self, request, spider):
        # ip池为空,我们现在需要先获得代理IP
        if self.ip_count == 0 or self.ip_count == self.max_count:
            self.getIPData()
            self.ip_count = 1

        # IP池填充完成, 校验当前 IP 是否有用.
        # 如果有效就使用,无效则更换.
        self.verifyAndChange(request)

```




