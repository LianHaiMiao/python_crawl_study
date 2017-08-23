
# python爬虫学习（二）Beautiful Soup

## 0.说明

该笔记本主要根据 北京理工大学嵩天老师的课程 [Python网络爬虫与信息提取](http://www.icourse163.org/course/BIT-1001870001) 记录而成

Beautiful Soup库的官方文档请参考 [Beautiful Soup](https://www.crummy.com/software/BeautifulSoup/bs4/doc/)



## Beautiful Soup首次尝试


```python
import requests
from bs4 import BeautifulSoup

url = 'http://python123.io/ws/demo.html'
r = requests.get(url)
page = r.text
soup = BeautifulSoup(page, "html.parser")

# 展示soup标签树
# print(soup.prettify())
# 结果 (此处不展示)

# 展示第一个a标签
# print(soup.a)
# 结果：<a class="py1" href="http://www.icourse163.org/course/BIT-268001" id="link1">Basic Python</a>

# 展示第一个a标签的属性值
# print(soup.a.attrs)
# 结果 {'id': 'link1', 'href': 'http://www.icourse163.org/course/BIT-268001', 'class': ['py1']}

# 展示链接属性
# print(soup.a.attrs['href'])
# 结果 http://www.icourse163.org/course/BIT-268001

# 显示a标签的内容
# print(soup.a.string)
# 结果： Basic Python

# 显示标签名字
# print(soup.a.name)
# 结果：a
```

## Beautiful Soup库的介绍、理解、使用

Beautiful Soup库是解析、遍历、维护“标签树”的功能库

使用方式：
    from bs4 import BeatifulSoup
    # 或者
    import bs4
    soup = BeatifulSoup('<html>data</html>', 'html.parser')
简单的说，我们可以将BeatifulSoup类当成一个HTML/XML文档的全部内容

**Beautiful Soup库的解析器**

<table>
    <tr>
        <th>解析器</th>
        <th>使用方法</th>
        <th>条件</th>
    </tr>
    <tr>
        <td>bs4的HTML解析器</td>
        <td>BeatifulSoup(mk, 'html.parser')</td>
        <td>安装bs4库</td>
    </tr>
    <tr>
        <td>lxml的HTML解析器</td>
        <td>BeatifulSoup(mk, 'lxml')</td>
        <td>pip install lxml</td>
    </tr>
    <tr>
        <td>lxml的XML解析器</td>
        <td>BeatifulSoup(mk, 'xml')</td>
        <td>pip install lxml</td>
    </tr>
    <tr>
        <td>html5lib的解析器</td>
        <td>BeatifulSoup(mk, 'html5lib')</td>
        <td>pip install html5lib</td>
    </tr>
</table>

**Beautiful Soup类的基本元素**

<table>
    <tr>
        <th>基本元素</th>
        <th>说明</th>
    </tr>
    <tr>
        <td>Tag</td>
        <td>标签，最基本的信息组织单元，分别用<>和</>标明开头和结尾</td>
    </tr>
    <tr>
        <td>Name</td>
        <td>标签的名字， < p>...< /p>的名字是'p', 格式:< tag>.name</td>
    </tr>
    <tr>
        <td>Attributes</td>
        <td>标签的属性，字典形式组织，格式: < tag>.attrs</td>
    </tr>
    <tr>
        <td>NavigableString</td>
        <td>标签内非属性字符串，<> ...</>中字符串，格式：< tag>.string</td>
    </tr>
    <tr>
        <td>Comment</td>
        <td>标签内字符串的注释部分，一种特殊的Comment类型</td>
    </tr>
</table>

**标签树的下行遍历**

<table>
    <tr>
        <th>属性</th>
        <th>说明</th>
    </tr>
    <tr>
        <td>.contents</td>
        <td>子节点的列表，将< tag>所有儿子节点存入列表</td>
    </tr>
    <tr>
        <td>.children</td>
        <td>标签的名字， < p>...< /p>的名字是'p', 格式:< tag>.name </td>
    </tr>
    <tr>
        <td>.descendants</td>
        <td>子孙节点的迭代类型，包含所有子孙节点，用于循环遍历 </td>
    </tr>

</table>


**标签树的上行遍历**

<table>
    <tr>
        <th>属性</th>
        <th>说明</th>
    </tr>
    <tr>
        <td>.parent</td>
        <td>节点的父亲标签</td>
    </tr>
    <tr>
        <td>.parents</td>
        <td>节点先辈标签的迭代类型，用于循环遍历先辈节点 </td>
    </tr>
</table>

**标签树的平行遍历**

> 请注意，所有的平行便利必须发生在同一个父节点下。

<table>
    <tr>
        <th>属性</th>
        <th>说明</th>
    </tr>
    <tr>
        <td>.next_sibling</td>
        <td>返回按照HTML文本顺序的下一个平行节点标签</td>
    </tr>
    <tr>
        <td>.previous_sibling</td>
        <td>返回按照HTML文本顺序的上一个平行节点标签 </td>
    </tr>
    <tr>
        <td>.next_siblings</td>
        <td>迭代类型，返回按照HTML文本顺序的后续的所有平行节点标签 </td>
    </tr>
    <tr>
        <td>.previous_siblings</td>
        <td>迭代类型，返回按照HTML文本顺序的前序的所有平行节点标签</td>
    </tr>
</table>

**更加“友好”的输出html页面**

使用prettify方法

print(soup.prettify())


**总结：Beautiful Soup库**

bs4库的基本元素：

Tag, Name, Attributes, NavigableString, Comment

bs4库的遍历功能：

下行遍历：
.contents .children .descendents

上行遍历：
.parent .parents

中行遍历：
.next_sibling .previous_sibling .next_siblings .previous_siblings



```python
import requests
from bs4 import BeautifulSoup

url = 'http://python123.io/ws/demo.html'
r = requests.get(url)
page = r.text
soup = BeautifulSoup(page, "html.parser")

# 标签树的遍历

# 查看head标签里面的内容
# print(soup.head)
# 结果：<head><title>This is a python demo page</title></head>

# 查看body标签里面的内容
# print(soup.body)
# 结果：body标签的内容

# 查看body标签里面的内容，以列表形式返回
# print(soup.body.contents)
# 结果：...(自行尝试)

# 查看body标签里面的内容的长度
# print(len(soup.body.contents))
# 结果： 5


# 便利儿子节点/便利子孙节点类似
#for child in soup.body.children:
    #print(child)
# 结果： 自行查看

# 查看a标签的父亲节点
# print(soup.a.parent)
# 结果： <p class="course">Python is a wonderful general-purpose programming language. You can learn Python from novice to professional by tracking the following courses:
#       <a class="py1" href="http://www.icourse163.org/course/BIT-268001" id="link1">Basic Python</a> and <a class="py2" href="http://www.icourse163.org/course/BIT-1001870001" id="link2">Advanced Python</a>.</p>

# 查看a标签的所有父亲节点，会一层一层，直至最后的html
# for parent in soup.a.parents:
#     if parent is None:
#         因为便利过程会一直持续到soup本身，但是soup本身没有.name是属性，所以我们直接打印它，就是[document]
#         print(parent)
#     else:
#         print(parent.name)
# 结果   p body html [document]


# 查看a标签后续的平行标签们
#for slibing in soup.a.next_siblings:
    #print(slibing)
# 结果：(3个)  and   <a class="py2" href="http://www.icourse163.org/course/BIT-1001870001" id="link2">Advanced Python</a>  .

# 更加“友好”的输出html页面
# print(soup.prettify())

```

#### 通过Beautiful Soup库对信息进行提取和处理

**find_all(name, attrs, recursive, string, \*\*kwargs)**

返回一个列表类型，存储查找结果。

name: 对标签名称的检索字符串。【可以是列表，可以是True】

attrs：对标签属性值的检索字符串，可以标注属性检索

recursive：是否对子孙节点全部检索，默认为True

string：< >...< />中字符串区域的检索字符串


### 另外，请谨记

> <tag>(...)等价于 <tag>.find_all(..)

> soup(...)等价于 soup.find_all(..)

**扩展方法**

<table>
    <tr>
        <th>方法</th>
        <th>说明</th>
    </tr>
    <tr>
        <td><>.find()</td>
        <td>搜索且只返回一个结果，字符串类型，同.find_all()参数</td>
    </tr>
    <tr>
        <td><>.find_parents()</td>
        <td>在先辈节点中搜索，返回列表类型，同.find_all()参数</td>
    </tr>
    <tr>
        <td><>.find_parent()</td>
        <td>在先辈节点中搜索，只返回一个结果，字符串类型，同.find_all()参数</td>
    </tr>
    <tr>
        <td><>.find_next_siblings()</td>
        <td>在后续平行节点中搜索，返回列表类型，同find_all()参数</td>
    </tr>
    <tr>
        <td><>.find_next_sibling()</td>
        <td>在后续平行节点中搜索，只返回一个结果，同.find_all()参数</td>
    </tr>
    <tr>
        <td><>.find_previous_siblings()</td>
        <td>在前序平行节点中搜索，返回列表类型，同.find_all()参数</td>
    </tr>
    <tr>
        <td><>.find_previous_sibling()</td>
        <td>在前序平行节点中搜索，只返回一个结果，同.find_all()参数</td>
    </tr>
</table>


```python
import requests
import re
from bs4 import BeautifulSoup

url = 'http://python123.io/ws/demo.html'
r = requests.get(url)
page = r.text
soup = BeautifulSoup(page, "html.parser")

# 通过find_all方法找出所有的a，b标签

# for link in soup.find_all(['a', 'b']):
#     print(link)

# 找到所有p标签中，含有course字符串的标签

# for link in soup.find_all('p', 'course'):
#     print(link)

# 直接查找id = link1的标签
# print(soup.find_all(id='link1'))

# recursive置位False表示soup的儿子节点中没有a标签，而在其子孙节点中才有
# print(soup.find_all('a',recursive=False))
# 结果 []

# 检索soup中含有python字符串的文本内容
# print(soup.find_all(string=re.compile("python")))
# 结果：['This is a python demo page', 'The demo python introduces several python courses.']
```

## 实战：中国大学排名定向爬取

### 思路分析

**功能分析**

输入：大学排名URL链接

输出：大学排名信息的屏幕输出

技术路线：requests-bs4

定向爬虫：仅对输入URL进行爬取，不扩展爬取

### format函数粗略了解



### 小技巧

对于中文不对齐问题，我们可以采用中文字符的空格填充chr(12288)|


```python
import requests
from bs4 import BeautifulSoup
import bs4

# 定义获取soup的函数
def getHTML(url):
    try:
        r = requests.get(url, timeout=30)
        # 产生异常信息
        r.raise_for_status()
        r.encoding = r.apparent_encoding
        return r.text
    except:
        print('爬取失败！')

# 处理获得的soup,获取我们需要的content
def dealSoup(ulist, page):
    count = 1;
    temp = [];
    soup = BeautifulSoup(page, "html.parser")
    # 获取了所有大学排名的列表，但是还是html格式，接下来，我们需要解析这些格式的数据   
    tr = soup.find_all('tr')[1:]
    for tds in tr:
        td = tds.find_all('td')[1:]
        temp.append(count)
        # 此时的每个td都是单独的一个学校，包含了排名、分数等信息
        for i in td:
            temp.append(i.string)
        ulist.append(temp)
        temp = []
        count = count + 1
    return

# 定义一个显示函数 主要使用fomat函数
def printUniverList(ulist, num):
    # 定义一个输出模板的变量  
    tplt = "{0:^10}\t{1:{13}^10}\t{2:^10}\t{3:^10}\t{4:^10}\t{5:^10}\t{6:^10}\t{7:^10}\t{8:^10}\t{9:^10}\t{10:^10}\t{11:^10}\t{12:^10}"
    print(tplt.format("排名", "学校名称", "省份", "总分", "生源质量", "培养结果", "科研规模", "科研质量", "顶尖成果", "顶尖人才", "科技服务", "成果转化", "学生国际化",chr(12288)))
#     tplt = "{0:^10}\t{1:{4}^10}\t{2:^10}\t{3:^10}\t{4:^10}"
#     print(tplt.format("排名","学校名称","省份", "总分", chr(12288)))
    for i in range(num):
        u = ulist[i]
        print(tplt.format(u[0], u[1], u[2], u[3], u[4], u[5], u[6], u[7], u[8], u[9], u[10], u[11], u[12], chr(12288)))

# 入口函数
if __name__ == "__main__":
    uinfo = []
    url = "http://zuihaodaxue.cn/zuihaodaxuepaiming2017.html"
    page = getHTML(url)
    dealSoup(uinfo, page)
    printUniverList(uinfo, 10)
```

        排名    	　　　学校名称　　　	    省份    	    总分    	   生源质量   	   培养结果   	   科研规模   	   科研质量   	   顶尖成果   	   顶尖人才   	   科技服务   	   成果转化   	  学生国际化   
        1     	　　　清华大学　　　	    北京    	  94.0    	  100.0   	  97.70%  	  40938   	  1.381   	   1373   	   111    	 1263428  	  613524  	  7.04%   
        2     	　　　北京大学　　　	    北京    	  81.2    	  96.1    	  97.61%  	  39853   	  1.327   	   1137   	    90    	  470237  	  31652   	  6.16%   
        3     	　　　浙江大学　　　	    浙江    	  77.8    	  87.2    	  97.91%  	  44509   	  1.109   	   921    	    88    	 1107921  	  23240   	  4.67%   
        4     	　　上海交通大学　　	    上海    	  77.5    	  89.4    	  98.62%  	  44674   	  1.134   	   836    	    79    	  796668  	  94339   	  6.30%   
        5     	　　　复旦大学　　　	    上海    	  71.1    	  91.8    	  97.85%  	  28205   	  1.355   	   758    	    57    	  246629  	   6221   	  6.78%   
        6     	　中国科学技术大学　	    安徽    	  65.9    	  91.9    	  95.20%  	  20169   	  1.502   	   749    	    40    	  69617   	    0     	  1.02%   
        7     	　　　南京大学　　　	    江苏    	  65.3    	  87.1    	  98.60%  	  23308   	  1.344   	   610    	    34    	  408221  	   647    	  4.63%   
        8     	　　华中科技大学　　	    湖北    	  63.0    	  80.6    	  95.58%  	  26199   	  1.202   	   645    	    34    	  486917  	   5401   	  4.19%   
        9     	　　　中山大学　　　	    广东    	  62.7    	  81.1    	  92.77%  	  26422   	  1.247   	   565    	    46    	  212530  	   1373   	  4.15%   
        10    	　哈尔滨工业大学　　	   黑龙江    	  61.6    	  76.4    	  95.20%  	  26672   	  1.005   	   550    	    23    	 1666191  	   4886   	  1.98%   
    

