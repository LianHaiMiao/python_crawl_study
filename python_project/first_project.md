# scrapy　初体验

## 0.说明

很久没有碰 scrapy 了，现在又因为某个项目，要去简单的爬取一些淘宝的数据，所以就记录一下。

(本项目基于 scrapy tutorial 扩展而来)

## 1. 准备工作

1. 下载 pycharm，用于 IDE
2. 安装好 scrapy，具体的安装办法，上官网自己看去


## 2. 第一步：创建项目和调试预备

```
scrapy startproject tutorial
```

然后，我们就会创建一个 tutorial 文件夹，文件夹的目录文件如下所示：

```
tutorial/
    scrapy.cfg            # deploy configuration file

    tutorial/             # project's Python module, you'll import your code from here
        __init__.py

        items.py          # project items definition file

        middlewares.py    # project middlewares file

        pipelines.py      # project pipelines file

        settings.py       # project settings file

        spiders/          # a directory where you'll later put your spiders
            __init__.py
```

> 为了便于调试，我们还需要做另一件事。

我们要在 `tutorial/` 文件夹下面新建一个文件叫 `entrypoint.py` 。这样做的好处是可以使用 pycharm 来进行调试，文件里面的具体内容如下所示：

``` python
# 第三个参数是你爬虫的名字
from scrapy.cmdline import execute
execute(['scrapy', 'crawl', 'quotes'])

```

## 3. 第二步：确定我们需要爬取的网页的数据

因为我们需要爬取的网站是比较简单的名言网站，因此，我们在这里只设定3个字段。

于是，我们在 `items.py` 写入如下代码：

```
import scrapy


class QuoteItem(scrapy.Item):
    # define the fields for your item here like:
    # name = scrapy.Field()
    # 名言内容
    quote_text = scrapy.Field()
    # 作者
    quote_author = scrapy.Field()
    # 名言标签
    quote_tags = scrapy.Field()

```


## 4. 第三步：编写爬虫

我们在 `tutorial/spiders` 文件夹下面新建一个文件叫  `quotes_spider.py`, 用来处理爬取到的网页内容。

```
# -*- coding: utf-8 -*-
import scrapy
from scrapy.http import Request
from tutorial.items import QuoteItem

class QuotesSpider(scrapy.Spider):
    name = "quotes"
    base_url = 'http://quotes.toscrape.com'

    # 爬虫开始的函数
    def start_requests(self):
        # 为什么要这样写呢？
        # 因为通常，我们的起始页面是很多个网页组成的，我们会用一个 list 来保存这些页面，为了统一格式，我们采取这样的写法。
        urls = ['http://quotes.toscrape.com/']
        for url in urls:
            # 回调函数，用来处理 url
            yield Request(url, callback=self.parse)

    def parse(self, response):
        # 获取 class quote 的内容 获取名言数据
        quotes = response.css('.quote')
        for quote in quotes:
            item = QuoteItem()
            item['quote_text'] = quote.css('.text::text').extract_first()
            item['quote_author'] = quote.css('.author::text').extract_first()
            item['quote_tags'] = ",".join(quote.css('.tags .tag::text').extract())
            yield item

        # 获取接下来每一个页面的 url 链接, 如果我们爬取到最后一页，那么我们就立刻提示并且退出。
        try:
            nextpage = response.xpath('//li[@class="next"]/a/@href').extract()[0]
            next_url = self.base_url + nextpage

            yield Request(next_url, callback=self.parse)
        except:
            print("claw over， claw over， claw over， claw over，claw over， claw over！！！")
            return
        
```


## 5. 第四步：处理爬取到的数据

我们在 `tutorial` 文件夹下面新建一个文件夹叫 `data` 用来保存数据，然后在 `data` 目录下新建一个文件叫 `items.jl`, 将爬取到的数据保存在这个文件之中。最后在 `tutorial/pipelines.py` 文件中，我们会写几个函数用来处理我们从网页上爬取到的数据。

```
import json

class TutorialPipeline(object):
    def __init__(self):
        self.file = open('./data/items.jl', 'w', encoding='utf-8')

    # 将item写入JSON文件
    def process_item(self, item, spider):

        line = json.dumps(dict(item)) + "\n"
        self.file.write(line)
        return item

    def close_spider(self, spider):
        self.file.close()

```


## 6. 第五步：打开 setting.py 开启我们需要的配置

因为在这里我们只使用了 ITEM_PIPELINES 所以在 `setting.py` 文件中我们取消 如下的注释,然后运行爬虫就可以开始爬取数据了。

```
# Configure item pipelines
# See https://doc.scrapy.org/en/latest/topics/item-pipeline.html
ITEM_PIPELINES = {
   'tutorial.pipelines.TutorialPipeline': 300,
}

```


