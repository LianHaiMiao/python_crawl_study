# 爬取股票信息

## 0.说明

该项目主要根据 北京理工大学嵩天老师的课程 [Python网络爬虫与信息提取](http://www.icourse163.org/course/BIT-1001870001) 记录而成

## 项目简介

**项目目的：**

这是一个简单的练手项目、主要是为了熟悉 re库 、 BeautifulSoupk库的基本使用方法

**项目说明：**

股票一直以来都是我们生活中所谈论的事务、所以做一个爬虫、爬取一下所有的深市、沪市的股票岂不美哉？

在此提供两个网站

1、[东方财富网深市、沪市的股票大全](http://quote.eastmoney.com/stocklist.html)




网站如图所示：


![东方财富网](/assets/socket/eastmoney.jpg)




2、[百度股市通](https://gupiao.baidu.com/stock/)




![百度股市通显示股票的页面](/assets/socket/baidusocket.jpg)




思路：

从上面两张图中，我们可以看到，东方财富网给我们提供了所有深市、沪市上市公司的股票代码。

而根据百度股市通的url最后一个斜杠 / 是股票代码，它会根据不同的股票为我们进行不同的页面跳转

所以，我们会这样做：

1. 先爬取东方财富网，将所有的股票代码保存在一个list中。

2. 循环遍历上一步中获取的list，然后再利用requests获取 https://gupiao.baidu.com/stock/股票代码 中的html页面

3. 对获取的的html进行处理，获取我们需要的信息，并以键值对的形式进行保存

4. 将该键值对写入文件中

5. 重复2-4步、直到程序不再运行。

**注意事项：**

如果对request库、re库、beautifulsoup库有疑惑、可以查阅

1. [request笔记](/python_package/request.md)
2. [beautifulsoup笔记](/python_package/BeautifulSoup.md)
3. [re库官方文档](https://docs.python.org/3/library/re.html)


## 项目部分函数的说明

``` python
def getHTMLText(url):
    try:
        r = requests.get(url, timeout=30)
        r.raise_for_status()
        r.encoding = r.apparent_encoding
        return r.text
    except:
        return "程序出错"
```

    r.raise_for_status() 

这个方法是用来获取异常的，如果响应状态码不是 200，就主动抛出异常

    r.encoding = r.apparent_encoding

这句话可以保证requests抓取下的text的编码格式正确


``` python
def getStockList(lst, url):
    page = getHTMLText(url)
    # 解析页面
    soup = BeautifulSoup(page, 'html.parser')
    # 找到所有的li标签，然后进行条件匹配
    for link in soup.select("li > a"):
        try:
            rex = re.search(r's[h|z]\d{6}', link['href'])
            if rex:
                lst.append(rex.group(0))
        except:
            continue
    return
```
传入一个用于保存股票代码的lst和东方财富网的url链接，然后经过这个函数的处理，我们可以获取所有的股票代码


``` python
# 定义获取股票信息的函数、同时存入文件中
def writeIntoFile(lst, url, fpath):
    count = 0
    for co in lst:
        try:
            dic = {}
            # 用resquests库爬取股票页面
            page = getHTMLText(url + co)
            # 用BeautifulSoup库处理股票页面
            soup = BeautifulSoup(page, 'html.parser')
            name = soup.find(class_='bets-name')
            s = name.text.strip()
            name = s.split(' ')[0]
            dic.update({'股票名字': name})
            # 解析bets-content中的内容，以键值对的形式，dt和dd
            info = soup.find(class_='bets-content')
            key = info.find_all('dt')
            value = info.find_all('dd')
            
            for i in range(len(key)):
                dic[key[i].text.strip()] = value[i].text.strip()
            
            # 将字典内容写入文件
            with open(fpath, 'a') as f:
                f.write(str(dic) + '\n')
                count = count + 1
            
            print('\r当前进度:  {:.2f}%'.format(count*100/len(lst)), end='')
            print('\n')
            print(dic)
        except:
            traceback.print_exc()
            count = count+1
            continue
    return ''
```

遍历装有股票代码的list,然后拼接成百度股市通的url,最后再对该页面进行处理，同时用一个dict保存股票的信息，因为随时有可能抛出异常，所以我们会需要 try catch语句。

    traceback.print_exc()

是用来查看在哪里抛出了异常的


## python 爬取股票信息源码

``` python
# -*- coding: utf-8 -*-
"""
@author: Shane_Miao

爬取股票信息
"""
import re
import requests
from bs4 import BeautifulSoup
import traceback
# 定义解析页面获取html代码的函数
def getHTMLText(url):
    try:
        r = requests.get(url, timeout=30)
        r.raise_for_status()
        r.encoding = r.apparent_encoding
        return r.text
    except:
        return "程序出错"

# 定义获取股票列表的函数
def getStockList(lst, url):
    page = getHTMLText(url)
    # 解析页面
    soup = BeautifulSoup(page, 'html.parser')
    # 找到所有的li标签，然后进行条件匹配
    for link in soup.select("li > a"):
        try:
            rex = re.search(r's[h|z]\d{6}', link['href'])
            if rex:
                lst.append(rex.group(0))
        except:
            continue
    return

# 定义获取股票信息的函数、同时存入文件中
def writeIntoFile(lst, url, fpath):
    count = 0
    for co in lst:
        try:
            dic = {}
            # 用resquests库爬取股票页面
            page = getHTMLText(url + co)
            # 用BeautifulSoup库处理股票页面
            soup = BeautifulSoup(page, 'html.parser')
            name = soup.find(class_='bets-name')
            s = name.text.strip()
            name = s.split(' ')[0]
            dic.update({'股票名字': name})
            # 解析bets-content中的内容，以键值对的形式，dt和dd
            info = soup.find(class_='bets-content')
            key = info.find_all('dt')
            value = info.find_all('dd')
            
            for i in range(len(key)):
                dic[key[i].text.strip()] = value[i].text.strip()
            
            # 将字典内容写入文件
            with open(fpath, 'a') as f:
                f.write(str(dic) + '\n')
                count = count + 1
            
            print('\r当前进度:  {:.2f}%'.format(count*100/len(lst)), end='')
            print('\n')
            print(dic)
        except:
            traceback.print_exc()
            count = count+1
            continue
    return ''

if __name__ == '__main__':
    stock_list_url = 'http://quote.eastmoney.com/stocklist.html'
    stock_info_url = 'https://gupiao.baidu.com/stock/'
    fpath = 'D://gupiao.txt'
    # 股票列表
    stock_list = []    
    getStockList(stock_list, stock_list_url)
    # 已经获取了所有的股票代码，接下来就是遍历这个列表，然后根据百度股票查找所有的信息，同时写入文件
    writeIntoFile(stock_list, stock_info_url, fpath)

```




