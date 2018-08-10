# Simple Spider

<!-- TOC -->

- [Simple Spider](#simple-spider)
    - [HTTP request/response](#http-requestresponse)
    - [`urllib`](#urllib)
    - [`requests`](#requests)
    - [`BeautifulSoup`](#beautifulsoup)

<!-- /TOC -->

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

## `requests`

`pip install requests`

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
import sys
import requests

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

## `BeautifulSoup`

`pip install beautifulsoup4`

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