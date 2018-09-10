# Simple Spider

<!-- TOC -->

- [Simple Spider](#simple-spider)
    - [Introdution](#introdution)
    - [HTTP request/response](#http-requestresponse)
    - [`urllib`](#urllib)
    - [`requests`](#requests)
    - [`BeautifulSoup`](#beautifulsoup)
        - [BeautifulSoup selector](#beautifulsoup-selector)

<!-- /TOC -->

## Introdution

复杂的涉及分布式爬虫+ Elasticsearch

友情链接作用：搜索引擎，会访问友情链接，所以为了让自己的搜索结果靠前，需要和其他网站大量交换友情链接

应用场景:
- 咨询报告
- 抢票、投票作弊器
- 预测（股市、票房）
- 国民情感分析
- 社交关系网络
- 政府部分舆情监测

浏览器输入网址后的动作:
- 浏览器首先访问的是DNS服务器，查找域名对应的IP地址
- 向IP对应的服务器发送请求(get, post)
- 服务器响应请求，发回response
- 浏览器渲染response

> 爬虫其实就是为了得到response; 然后进一步得到数据

> 浏览器和服务器要发送、接受数据，就需要遵循HTTP或HTTPS协议，协议规定了request, response的各种格式

## HTTP request/response

[HTTP request/response](https://blog.csdn.net/lzghxjt/article/details/51458540)

HTTP request messages:
> ![](res/http_msg01.png)

HTTP response messages:
> ![](res/http_msg02.png)

HTTP request/response:
> ![https://developer.mozilla.org/en-US/docs/Web/HTTP/Messages](res/http_msg03.png)

## `urllib`

python3中urllib, urllib2合并为[urllib](https://docs.python.org/3/howto/urllib2.html#urllib-howto); 现在一般用的requests, 而不用urllib; [requests vs urllib](https://www.cnblogs.com/znyyy/p/7868511.html)

```python
import urllib.request

with urllib.request.urlopen("https://www.python.org/") as response:
   html = response.read().decode('utf8')
   print(response.getcode()) # 200
```

`response.getcode()` return:
- 2开头，网页正常
- 4开头，网页不存在
- 5开头，服务器bug

```python
# download response-body
import urllib.request

with urllib.request.urlopen('https://www.python.org/') as response:
    with open('test.html', 'wb') as file:
        file.write(response.read())
```

```python
# douyu example: 很容易被forbidden
import re
import time
import urllib.request

def get_img(html):
    regex=re.compile(r'data-original="(.*?\.jpg)"')
    img_urls=regex.findall(html)
    imgs_sum=len(img_urls)
    for index, img_url in enumerate(img_urls):
        print(f'downing image-{index}/{imgs_sum}')
        with urllib.request.urlopen(img_url) as response, open(f'./imgs/image-{index}.jpg', 'wb') as file:
            file.write(response.read())
            # time.sleep(1)

# add agent
opener=urllib.request.build_opener()
opener.addheaders = [('User-agent', "Mozilla/5.0 (Windows NT 10.0; WOW64; rv:61.0) Gecko/20100101 Firefox/61.0")]
with opener.open('https://www.douyu.com/g_yz') as response:
    html=response.read().decode('utf8')

# begin
get_img(html)
```

```python
import urllib

params=urllib.urlencode({'t':1,'eggs':2,'bacon':0})

# urllib get
f1=urllib.urlopen("http://python.org/query?%s" % params)
print(f1.read())

# urllib post
f2=urllib.urlopen("http://python.org/query",parmas)
print(f2.read())
```

```python
# urllib download file
urllib.urlretrieve('https://rpic.douyucdn.cn/20180831104533_small.jpg', './images/1.jpg')
```

## `requests`

`pip install requests`, 如果对性能有要求可以采用`pip install pycurl`

> [requests vs pycurl](https://github.com/yudazilian/Pycurl-vs-Requests)

```python
import requests

r=requests.get('https://www.python.org/')
html=r.text
# binary_html=r.content
print(r.status_code) # 200
```

```python
# 没有forbidden
# douyu 单页
import re
import requests

def get_img(html):
    regex=re.compile(r'data-original="(.*?\.jpg)"')
    img_urls=regex.findall(html)
    img_sum=len(img_urls)
    for index, img_url in enumerate(img_urls):
        print(f'downing image-{index}/{img_sum}')
        r=requests.get(img_url)
        with open(f'./imgs/image-{index}.jpg', 'wb') as file:
            file.write(r.content)

html=requests.get('https://www.douyu.com/g_yz').text
get_img(html)
```

```python
# with agent & session
import re
import requests

def get_img(html):
    regex=re.compile(r'data-original="(.*?\.jpg)"')
    img_urls=regex.findall(html)
    img_sum=len(img_urls)
    for index, img_url in enumerate(img_urls):
        print(f'downing image-{index}/{img_sum}')
        r=requests.get(img_url)
        with open(f'./imgs/image-{index}.jpg', 'wb') as file:
            file.write(r.content)

headers={
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; WOW64; rv:61.0) Gecko/20100101 Firefox/61.0",
}
session=requests.Session()
r=session.get('https://www.douyu.com/g_yz', headers=headers)
get_img(r.text)
```

斗鱼多页: 通过F12/ViewSource得知[斗鱼](https://www.douyu.com/g_yz)采用了JS生成pagenator, 然后切换页码的时候采用的Ajax; 所以点击翻页地址栏不变;

F12/Network/XHR, 点击切换页码，发现了两个request(一个post, 一个get), post请求的`FormData`里面包含了`pg`, get请求的[url](https://www.douyu.com/gapi/rkc/directory/2_201/2)为json;

所以两个思路: (采用思路一)
- 直接通过get的url,获取json
- 通过`requests`模块的`post`方法来翻页


```python
# 斗鱼多页
import re
import os
import sys
import requests

if 'imgs' not in os.listdir():
    os.mkdir('imgs')

html=requests.get('https://www.douyu.com/g_yz').text
# 通过html的<script>xxx</script>获取页码

regex=re.compile(r'count: "(\d+)"')
total_pages=eval(regex.search(html).group(1))# 4

# 这个地方可以采用多进程进行，分配任务; 为了简单没有采用
json_urls=[f'https://www.douyu.com/gapi/rkc/directory/2_201/{i+1}' for i in range(total_pages)]

# 通过分析json的结构得到
img_urls=[]
for url in json_urls:
    r=requests.get(url)
    temp_dict=r.json()
    for item in temp_dict['data']['rl']:
        img_urls.append(item['rs16'])

# download all imgs
def get_img(img_urls):
    img_sum=len(img_urls)
    for index, url in enumerate(img_urls):
        # vscode, jupyter通用的进度条
        sys.stdout.write(f'\r{index+1}/{img_sum}')
        r=requests.get(url)
        with open(f'./imgs/image-{index}.jpg', 'wb') as file:
            file.write(r.content)

get_img(img_urls)
```

[头条街拍](https://www.jianshu.com/p/d67b1d4b99ad)

```python
import re
import os
import sys
import requests

# download imgs
def get_img(img_urls, img_dir):
    img_sum=len(img_urls)
    for index, url in enumerate(img_urls):
        sys.stdout.write(f'\r{index+1}/{img_sum}')
        r=requests.get(url)
        with open(f'./{img_dir}/image-{index}.jpg', 'wb') as file:
            file.write(r.content)

# begin
if 'imgs' not in os.listdir():
    os.mkdir('imgs')

url='https://www.toutiao.com/search_content'
offset=0

while True:
    # query param
    param={'offset':offset, 'format':'json', 'keyword':'街拍', 'autoload':'true', 'count':20, 'cur_tab':3, 'from':'gallery'}
    res=requests.get(url, param)
    data=res.json().get('data')
    if not data:
        # data为空的时候表示没有链接了
        break
    
    # get image urls
    img_urls=[]
    for article in data:
        img_list=article.get('image_list')
        if img_list:
            for img in img_list:
                origin_url=img['url'].replace('list', 'origin')
                complete_url=f'http:{origin_url}'
                img_urls.append(complete_url)
    
    # download imgages
    img_dir=f'imgs/offset{offset}'
    os.mkdir(img_dir)
    get_img(img_urls, img_dir)
    
    # offset + 20
    offset+=20
```

[九派新闻](https://ask.hellobi.com/blog/linjichu/sitemap/)

```python
import requests

param={'page':1, 'pageSize':15, 'type':1}
url='http://appjph.jiupaicn.com/app/content/recommend_pc/list'
res=requests.get(url, param)
data=res.json().get('resultData')

news_urls=[]

for item in data:
    title=item.get('title')
    if title:
        new_url=f'http://jphao.jiupaicn.com/index.php?m=content&c=jiupaihao&a=article&id={item.get("id")}&memberId={item.get("memberId")}'
        news_urls.append(new_url)
```

## `BeautifulSoup`

`pip install beautifulsoup4`

[Official Document](https://beautifulsoup.readthedocs.io/zh_CN/latest/)

```python
# 获取html的所有img_url
import requests
from bs4 import BeautifulSoup

html=requests.get('https://www.douyu.com/g_yz').text
soup=BeautifulSoup(html)

img_urls=[]
for item in soup.find_all('img', {'class': 'JS_listthumb'}):
    img_urls.append(item['data-original'])

print(img_urls)
```

[Python Salary](https://ask.hellobi.com/blog/linjichu/6479)

```python
import requests
from bs4 import BeautifulSoup

params={'query':'Python', 'scity':101010100, 'page':1} # 101010100代表北京
headers={
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; WOW64; rv:61.0) Gecko/20100101 Firefox/61.0",
}
url='https://www.zhipin.com/job_detail/'
res=requests.get(url,params, headers=headers)
data=res.text
soup=BeautifulSoup(data)

# for item in soup.find_all('span', class_='red'):
for item in soup.find_all('span', {'class': 'red'}):
    print(item.get_text())
```

### BeautifulSoup selector

```python
import requests
from bs4 import BeautifulSoup

html=requests.get('https://www.douyu.com/g_yz').text

soup=BeautifulSoup(html)
# format html
# print(soup.prettify())

# Select by tag
# get text stirng: soup.tag
# 获取第一个
print(soup.title) # <title>This is title</title>
print(soup.div.a) # <a class="head-logo fl" href="/"></a>
print(soup.title.string)
print(soup.name) # [document]
print(soup.title.name) # title
print(soup.div.attrs) # {'id': 'container', 'class': ['container']}
print(soup.a['href']) # ...


# CSS选择器
print(soup.select('title')) # 标签选择器
print(soup.select('.container .a-pop .list-wrap')) # class选择器
# print(soup.select('#main-col')) # id选择器
# print(soup.select('body #main-col')) # 三者组合

# 特殊
print(soup.select('head > title')) # 子标签选择器
# print(soup.select('div[class="container"]'))
# print(soup.select('a[name="grey"]'))
```