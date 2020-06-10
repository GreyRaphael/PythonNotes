# Simple Spider

- [Simple Spider](#simple-spider)
  - [Introdution](#introdution)
  - [HTTP request/response](#http-requestresponse)
  - [Proxy](#proxy)
  - [`urllib`](#urllib)
  - [`requests`](#requests)
    - [requests HTTP Auth](#requests-http-auth)
    - [request json](#request-json)
    - [requests session](#requests-session)
  - [`BeautifulSoup`](#beautifulsoup)
    - [BeautifulSoup selector](#beautifulsoup-selector)
  - [selenium](#selenium)
    - [selenium + chrome](#selenium--chrome)
    - [selenium + phantomjs](#selenium--phantomjs)
    - [selenium with firefox](#selenium-with-firefox)
    - [selenium keys & click](#selenium-keys--click)
    - [selenium with dynamic page](#selenium-with-dynamic-page)
    - [selenium mobile emulation](#selenium-mobile-emulation)
  - [login with cookie](#login-with-cookie)
    - [method1: only with session](#method1-only-with-session)
    - [method2&3: cookie with request](#method23-cookie-with-request)
  - [word cloud](#word-cloud)
  - [Periodic Sign Task](#periodic-sign-task)
  - [XPath](#xpath)
  - [OCR vs verify code](#ocr-vs-verify-code)
  - [requests vs selenium](#requests-vs-selenium)
  - [pyquery](#pyquery)
  - [Onlilne File Spider](#onlilne-file-spider)

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
> get,post本质上没有区别，仅仅是一个表现为地址栏，一个藏了起来，都是提交的同样dict格式的数据  
> get提交的时候数据在请求行中，post提交的时候数据在请求体中; 所以post并不会比get更加安全，抓包都能弄出来  
> ![](res/http_msg01.png)

HTTP response messages:
> ![](res/http_msg02.png)

HTTP request/response:
> ![https://developer.mozilla.org/en-US/docs/Web/HTTP/Messages](res/http_msg03.png)

## Proxy

淘宝买一个**vps秒换ip**, 将写好的程序挪到那个vps上运行就行。

Some reference:
- [抓取证券之星的股票数据](http://www.cnblogs.com/sjzh/archive/2016/09/24/5899716.html)
- [抓取代理IP并多线程验证](https://www.cnblogs.com/sjzh/p/5990152.html)
- [基础爬虫架构及爬取证券之星全站行情数据](https://www.cnblogs.com/sjzh/p/7657882.html)

example1: 随机切换User-Agent

```python
import json
import pickle
import random
import multiprocessing
import requests
from bs4 import BeautifulSoup

url = 'http://quote.stockstar.com/webhandler/rank.ashx'
user_agents = [
    "Mozilla/5.0 (Windows NT 10.0; WOW64; rv:53.0) Gecko/20100101 Firefox/53.0",
    "Mozilla/5.0 (Windows NT 10.0; WOW64; rv:54.0) Gecko/20100101 Firefox/54.0",
    "Mozilla/5.0 (Windows NT 10.0; WOW64; rv:55.0) Gecko/20100101 Firefox/55.0",
    "Mozilla/5.0 (Windows NT 10.0; WOW64; rv:56.0) Gecko/20100101 Firefox/56.0",
    "Mozilla/5.0 (Windows NT 10.0; WOW64; rv:57.0) Gecko/20100101 Firefox/57.0",
    "Mozilla/5.0 (Windows NT 10.0; WOW64; rv:58.0) Gecko/20100101 Firefox/58.0",
    "Mozilla/5.0 (Windows NT 10.0; WOW64; rv:59.0) Gecko/20100101 Firefox/59.0",
    "Mozilla/5.0 (Windows NT 10.0; WOW64; rv:60.0) Gecko/20100101 Firefox/60.0",
    "Mozilla/5.0 (Windows NT 10.0; WOW64; rv:61.0) Gecko/20100101 Firefox/61.0",
    "Mozilla/5.0 (Windows NT 10.0; WOW64; rv:62.0) Gecko/20100101 Firefox/62.0",
]


def add_data(queue, page_number):
    params = {
        'type': 'a',
        'sortfield': 3,
        'direction': 1,
        'pageid': page_number
    }
    headers = {"User-Agent": random.choice(user_agents)}
    try:
        res = requests.get(url, params=params, headers=headers)
        print(f'page-{page_number} is got')
        temp_dict = json.loads(res.text[12:-1]) # response是变量，并不是json
        queue.put(temp_dict['html'])
    except Exception as e:
        print(f'page={page_number}', e)


if __name__ == '__main__':
    queue_list = []
    process_list = []
    data_list = []
    for i in range(118): # 总共118页，为了便于理解，这里没有进行进程分组
        queue_list.append(multiprocessing.Queue())
        process_list.append(multiprocessing.Process(
            target=add_data, args=(queue_list[i], i+1)))
        process_list[i].start()
    for queue in queue_list:
        data_list.append(queue.get())
    # pickle.dump(data_list, open('data.dat', 'wb'))
    print('===finished capture data===')

    # write to file(也可以写入数据库)
    with open('final.txt', 'w', encoding='utf8') as file:
        for html in data_list:
            soup=BeautifulSoup(html, features='lxml')
            for i, td in enumerate(soup.select('td')):
                file.write(f'{td.string:>12}')
                if (i+1) % 13 ==0:
                    file.write('\n')
            file.write('\n')
```

example: [proxy](https://blog.csdn.net/qq_37616069/article/details/80376776)
> 付费的proxy一般带有用户名密码

```py
proxies = {
    "http": "http://username:pwd@183.129.207.73:13016",
    "https": "https://username:pwd@183.129.207.73:13016",
}
```

```python
import requests
 
# 根据协议类型，选择不同的代理
proxies = {
    "http": "https://183.129.207.73:13016",
    "https": "https://183.129.207.73:13016",
}
 
response = requests.get("https://m.huxiu.com/", proxies = proxies)
print(response.text)
```

example: sock5 proxy

```py
import socket
import socks
import requests

#洋葱浏览器9150默认端口的代理
socks.set_default_proxy(socks.SOCKS5,"127.0.0.1",9150)
socket.socket = socks.socksocket

print(requests.get("http://checkip.amazonaws.com").text)
```

example: visit deep web
> 通过CCProxy代理1080端口，然后Vidalia设置Tor的proxy为`socks://127.0.0.1:1080`， 然后使用cow代理Tor的9050端口到7777；也可以配合selenium+PhantomJS的`--proxy`访问带js的页面  
> ![](res/tor01.png)

```py
# visit deep web
import  requests
proxies={"http":"http://127.0.0.1:7777","https":"https://127.0.0.1:7777"}
s=requests.Session()
print(s.get("http://hcutffpecnc44vef.onion/",proxies=proxies).text)
```

example: iteract with tor browser by [stem](https://stem.torproject.org/api.html)
> `pip install stem`

```py
import socks#设置全局端口代理
import requests
import time
import socket
import stem #换ip
from stem import Signal #信号
from stem.control import  Controller #控制器

controller=Controller.from_port(port=9151) #9151时控制端口
controller.authenticate() #控制器，换ip

socks.set_default_proxy(socks.SOCKS5,"127.0.0.1",9150)
socket.socket=socks.socksocket

for   i   in range(10):
    print(i,requests.get("http://checkip.amazonaws.com").text)
    controller.signal(Signal.NEWNYM)
    time.sleep(15)
```

example2: www.kuaidaili.com抓取代理ip
> [免费代理IP地址列表](https://www.jianshu.com/p/93fd64a2747b)

```python
# www.kuaidaili.com
import time
import requests
from bs4 import BeautifulSoup

headers = {
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; WOW64; rv:61.0) Gecko/20100101 Firefox/61.0",
}
# www.kuaidaili.com必须使用Session才能克服反爬虫
session = requests.Session()

htmls = []
for page_number in range(1, 6):
    url = f'https://www.kuaidaili.com/free/inha/{page_number}/'
    try:
        res = session.get(url, headers=headers)
        htmls.append(res.text)
    except Exception as e:
        print(f'page={page_number}', e)
    time.sleep(0.5)
else:
    print('finished capture response')

all_ips = []
all_ports = []
all_types = []

for html in htmls:
    soup = BeautifulSoup(html, features="lxml")
    for td in soup.select('td[data-title="IP"]'):
        all_ips.append(td.string)
    for td in soup.select('td[data-title="PORT"]'):
        all_ports.append(td.string)
    for td in soup.select('td[data-title="类型"]'):
        all_types.append(td.string)

all_addr = []
for item in zip(all_types, all_ips, all_ports):
    all_addr.append(f'{item[0]}://{item[1]}:{item[2]}')

print(all_addr)
```

example3: www.xicidaili.com抓取代理ip

```python
import requests
from bs4 import BeautifulSoup

headers = {
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; WOW64; rv:61.0) Gecko/20100101 Firefox/61.0",
}
session = requests.Session()
htmls = []

for page_number in range(1, 6):
    url = f'http://www.xicidaili.com/nn/{page_number}'
    try:
        res = session.get(url, headers=headers)
        htmls.append(res.text)
    except Exception as e:
        print(f'page={page_number}', e)
else:
    print('finished capture response')


all_ips = []
all_ports = []
all_types = []

for html in htmls:
    soup=BeautifulSoup(html, features='lxml')
    for td in soup.select('tr > td:nth-of-type(2)'):
        all_ips.append(td.string)
    for td in soup.select('tr > td:nth-of-type(3)'):
        all_ports.append(td.string)
    for td in soup.select('tr > td:nth-of-type(6)'):
        all_types.append(td.string)

all_addr = []
for item in zip(all_types, all_ips, all_ports):
    all_addr.append(f'{item[0]}://{item[1]}:{item[2]}')

print(len(all_addr))
```

example4: 同example3的套路

```python
url = f'http://www.ip3366.net/free/'

params = {
    "stype": 1,
    "page": page_number
}

# 后面只需要微调
```

```python
url=f'http://www.mimiip.com/gngao/{page_number}'
# 后面微调
```

```python
url=f'http://www.66ip.cn/areaindex_1/{page_number}.html'
# 后面微调
```

example5: with regex

```python
import re
import requests

headers = {
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; WOW64; rv:61.0) Gecko/20100101 Firefox/61.0",
}
session = requests.Session()

home_url='http://www.xsdaili.com'
pat0=re.compile(r'<a href="/dayProxy/ip/(\d+)\.html">')
res0=session.get(home_url, headers=headers)
url=f'http://www.xsdaili.com/dayProxy/ip/{pat0.search(res0.text)}.html'

res=session.get(url, headers=headers)
pat=re.compile(r'(\d+\.\d+\.\d+\.\d+:\d+)@(\w+)#')
all_ip_port=pat.findall(res.text)

all_addr=[]
for item in all_ip_port:
    all_addr.append(f'{item[1]}://{item[0]}')

print(all_addr)
```

```py
import requests
import random
import re

pat1 = re.compile(r'href="(thread-.+?html)"')
pat2 = re.compile(r'src="(.+?)" /></a></li>')

def random_headers():
    version = random.randint(60, 72)
    random_ua = f'Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:{version}.0) Gecko/20100101 Firefox/{version}.0'
    return {'User-Agent': random_ua}

r = requests.get('https://www.lesmao.co/', headers=random_headers()).text
url_list = [f'https://www.lesmao.co/{short_url}' for short_url in pat1.findall(r)]

with open('img_src.txt', 'w') as file:
    for url in set(url_list):
        print(url)
        r = requests.get(url, headers=random_headers()).text
        for src in pat2.findall(r):
            file.write(f'{src}\n')
            file.flush()
```

example6: check ip

```python
import re
import multiprocessing
import requests

# get proxy ip
headers = {
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; WOW64; rv:61.0) Gecko/20100101 Firefox/61.0",
}
session = requests.Session()

home_url = 'http://www.xsdaili.com'
res0 = session.get(home_url, headers=headers)
pat0 = re.compile(r'<a href="/dayProxy/ip/(\d+)\.html">')
url = f'http://www.xsdaili.com/dayProxy/ip/{pat0.search(res0.text)}.html'

res = session.get(url, headers=headers)
pat = re.compile(r'(\d+\.\d+\.\d+\.\d+:\d+)@(\w+)#')
all_ip_port = pat.findall(res.text)

all_addr = []
for item in all_ip_port:
    all_addr.append(f'{item[1]}://{item[0]}')

all_proxies = []
for addr in all_addr:
    # 每一个dict就是一个proxies
    all_proxies.append({'https': addr})

# 多进程check proxies
test_url = 'http://quote.stockstar.com/stock'


def check_ip(proxies, q):
    try:
        session.get(test_url, headers=headers, proxies=proxies)
        q.put(proxies['https'])
        print(f'{proxies} is good')
    except Exception as e:
        print(f'{proxies} is bad', e)


if __name__ == '__main__':
    queue_list = []
    process_list = []
    for i, proxies in enumerate(all_proxies):
        queue_list.append(multiprocessing.Queue())
        process_list.append(multiprocessing.Process(
            target=check_ip, args=(proxies, queue_list[i])))
    for process in process_list:
        process.start()
    # write to file
    with open('good_ip.txt', 'w') as file:
        for queue in queue_list:
            file.write(f'{queue.get()}\n')
    # # 主进程默认会等待，下面可以不写
    # for process in process_list:
    #     process.join()
```

example7: 一个简单的爬虫框架

```python
import os
from multiprocessing import Pool
import random
import re
import time
import requests
import pandas as pd
from bs4 import BeautifulSoup
import sqlalchemy

'''
1.url获取器: get_catalog, get_urls
2.url管理器: url_manager
3.html下载器: spider_proxy, html_downloader
4.html解析器: html_parser
5.数据存储器: data_saver
6.爬虫管理器: spider_manager
'''


class catalog_getter(object):
    '''左边栏菜单'''

    def __init__(self):
        self.catalog = None

    def save_catalog(self):
        '''证券之星左侧导航的内容和网址并保存为csv'''
        headers = {
            "User-Agent": "Mozilla/5.0 (Windows NT 10.0; WOW64; rv:61.0) Gecko/20100101 Firefox/62.0",
        }
        session = requests.Session()
        res = session.get('http://quote.stockstar.com/', headers=headers)
        html = res.content.decode('gbk')
        soup = BeautifulSoup(html, features='html5lib')

        # 一级菜单列表+二级菜单列表
        catalog1 = pd.DataFrame(columns=["cata1", "cata2", "url2"])
        catalog2 = pd.DataFrame(columns=["url2", "cata3", "url3"])

        for submenu in soup.select('.subMenuBox .list'):
            for dl in submenu.select('.subNav dl'):
                cata2 = dl.dt.a.string.strip(' ·\n\r\t')
                url2 = dl.dt.a["href"]
                catalog1 = catalog1.append(
                    {"cata1": submenu.a.string, "cata2": cata2, "url2": url2}, ignore_index=True)
                for li in dl.select('dd li'):
                    cata3 = li.a.string.strip('·')
                    catalog2 = catalog2.append(
                        {"url2": url2, "cata3": cata3, "url3": li.a["href"]}, ignore_index=True)

        # 合并成完整菜单列表
        self.catalog = pd.merge(catalog1, catalog2, on='url2', how='left')
        self.catalog.to_csv('catalog.csv', index=False)

    def load_catalog(self):
        '''导入csv'''
        if 'catalog.csv' not in os.listdir():
            self.save_catalog()
            print('catalog.csv generated')
        else:
            print('catalog.csv already exists')
            self.catalog = pd.read_csv('catalog.csv')
        print('catalog.csv loaded')

    def get_info(self, index):
        '''创建每行的行名，作为存入数据库的表名，并获取每行终端的网址链接'''
        if str(self.catalog.loc[index]['cata3']) == 'nan':
            table_name = self.catalog.loc[index]['cata1'] + \
                '_' + self.catalog.loc[index]['cata2']
            url = f'http://quote.stockstar.com{self.catalog.loc[index]["url2"]}'
        else:
            table_name = self.catalog.loc[index]['cata1'] + '_' + \
                self.catalog.loc[index]['cata2'] + '_' + \
                self.catalog.loc[index]['cata3']
            url = f'http://quote.stockstar.com{self.catalog.loc[index]["url3"]}'
        return table_name, url


class urls_getter(object):
    '''获取每个menu的链接列表'''

    def __init__(self, url):
        self.url = url

    def get_urllist(self):
        headers = {
            "User-Agent": "Mozilla/5.0 (Windows NT 10.0; WOW64; rv:61.0) Gecko/20100101 Firefox/62.0",
        }
        session = requests.Session()
        res = session.get(self.url, headers=headers)
        html = res.content.decode('gbk')
        soup = BeautifulSoup(html, features='html5lib')

        page_size = int(soup.select_one(
            '#ClientPageControl1_hdnPageSize')['value'])
        total_count = int(soup.select_one(
            '#ClientPageControl1_hdnTotalCount')['value'])
        # ceiling division
        page_count = (total_count+page_size-1)//page_size
        # http://quote.stockstar.com/stock/sha_3_1_2.html中3, 1, 2分别是是sort_field, direction, page_index
        sort_field = int(soup.select_one(
            '#ClientPageControl1_hdnSortField')['value'])
        direction = int(soup.select_one(
            '#ClientPageControl1_hdnDirection')['value'])

        # construct urllist
        dot_index = self.url.rindex('.')
        url_prefix = self.url[:dot_index]
        urllist = [
            f'{url_prefix}_{sort_field}_{direction}_{i+1}.html' for i in range(page_count)]
        return urllist


class url_manager(object):
    def __init__(self):
        # 未爬
        self.new_urls = set()
        # 已爬
        self.old_urls = set()

    def has_new_url(self):
        '''判断是否有未爬取的URL'''
        return len(self.new_urls) > 0

    def get_new_url(self):
        '''获取一个未爬取的URL'''
        new_url = self.new_urls.pop()
        self.old_urls.add(new_url)
        return new_url

    def add_new_urls(self, urls):
        '''将新的URL列表添加到未爬取的URL集合中'''
        if urls is None or len(urls) == 0:
            return
        for url in urls:
            if url is None:
                return
            if url not in self.new_urls and url not in self.old_urls:
                self.new_urls.add(url)


class spider_proxy(object):
    '''获取有效proxies并保存'''

    @staticmethod
    def get_proxies():
        '''get proxy ips'''
        headers = {
            "User-Agent": "Mozilla/5.0 (Windows NT 10.0; WOW64; rv:61.0) Gecko/20100101 Firefox/61.0",
        }
        session = requests.Session()
        home_url = 'http://www.xsdaili.com'

        res0 = session.get(home_url, headers=headers)
        pat0 = re.compile(r'<a href="/dayProxy/ip/(\d+)\.html">')
        url = f'http://www.xsdaili.com/dayProxy/ip/{pat0.search(res0.text)}.html'

        res = session.get(url, headers=headers)
        pat = re.compile(r'(\d+\.\d+\.\d+\.\d+:\d+)@(\w+)#')
        all_ip_port = pat.findall(res.text)

        all_proxies = []
        for item in all_ip_port:
            all_proxies.append({'https': f'{item[1]}://{item[0]}'})
        print('get all_proxies')
        return all_proxies


class html_downloader(object):
    '''get html text'''

    def __init__(self, all_proxies):
        self.all_proxies = all_proxies

    def download(self, url):
        headers = {
            "User-Agent": "Mozilla/5.0 (Windows NT 10.0; WOW64; rv:61.0) Gecko/20100101 Firefox/61.0",
        }
        session = requests.Session()
        proxies = random.choice(self.all_proxies)

        print(proxies)

        try:
            res = session.get(url, headers=headers, proxies=proxies)
            html = res.content.decode('gbk')
            time.sleep(0.5)
        except Exception as e:
            print(f'get "{url}" error"', e)

        # 一般返回的比较短都是被forbidden
        if len(html) > 100:
            return html
        else:
            return None


class html_parser(object):
    '''解析html获取数据'''

    def __init__(self, html):
        self.html = html
        self.soup = BeautifulSoup(self.html, features='html5lib')

    def get_date(self):
        pat = re.compile(r'数据时间：(.*?)<')
        try:
            # 刨除后面的时间，只留下2018-09-14这样的日期
            date_string = pat.search(self.html).group(1)[:10]
        except Exception as e:
            print(f'get date error', e)
            return ''
        return date_string

    def get_header(self):
        '''获取表的标题'''
        header_name = []
        try:
            for td in self.soup.select('thead.tbody_right td'):
                header_name.append(td.string)
            header_name.append('数据时间')
        except Exception as e:
            print('get header name error', e)
            header_name = []
        finally:
            return header_name

    def get_datalist(self):
        '''获取表的数据'''
        header_name = self.get_header()
        # 如果表标题长度与数据长度不一致
        if len(header_name)-1 != len(self.soup.select('#datalist tr:nth-of-type(1) > td')):
            return [], []

        datalist = []
        date = self.get_date()
        try:
            for tr in self.soup.select('#datalist tr'):
                row = []
                for td in tr.select('td'):
                    row.append(td.string)
                row.append(date)
                datalist.append(row)
        except Exception as e:
            print('get datalist error', e)
            datalist = []
        return header_name, datalist

    def get_dataframe(self):
        '''用pandas组合成一个table'''
        header_name, datalist = self.get_datalist()
        table = pd.DataFrame(datalist, columns=header_name)
        return table


class data_saver(object):
    '''将数据存入sqlite3数据库'''

    def __init__(self, engine, table, table_name):
        self.engine = engine
        # table本质是pandas.DataFrame()
        self.table = table
        self.table_name = table_name

    def sava_to_db(self):
        # DataFrame的to_sql()方法
        self.table.to_sql(name=self.table_name, con=self.engine,
                          if_exists='append', index_label='id')


class spider_manager(object):
    '''爬虫调度器'''

    def __init__(self, engine, table_name, all_proxies):
        self.engine = engine
        self.table_name = table_name
        self.urlman = url_manager()
        self.downloader = html_downloader(all_proxies)

    def crawl_single(self, url):
        '''爬取单一网址'''
        html=self.downloader.download(url)
        parser=html_parser(html)
        if parser.get_header():
            table=parser.get_dataframe()
            saver=data_saver(self.engine, table, self.table_name)
            saver.sava_to_db()
            print(f"{url} saved to {self.table_name}")

    def crawl_menu(self, urllist):
        '''爬取一个menu'''
        self.urlman.add_new_urls(urllist)

        # pool = Pool(10)
        while self.urlman.has_new_url():
            new_url = self.urlman.get_new_url()
            # apply_async的时候已经开始运行了
            # pool.apply_async(func=self.crawl_single, args=(new_url, ))
            self.crawl_single(new_url)

        # pool.close()
        # pool.join()  # 主进程卡在这里


if __name__ == '__main__':
    obj = catalog_getter()
    obj.load_catalog()

    engine = sqlalchemy.create_engine("sqlite:///Data.db")
    all_proxies=spider_proxy.get_proxies()

    for index, row in obj.catalog.iterrows():
        table_name, url = obj.get_info(index)
        urlsGetter = urls_getter(url)
        urllist = urlsGetter.get_urllist()
        print(f"{index}:{table_name}'s urllist got")

        spider_man = spider_manager(engine, table_name, all_proxies)
        spider_man.crawl_menu(urllist)
        print(f"{index}: {table_name} crawled")
```

## `urllib`

python3 urllib modules:
- `urllib.request`
- `urllib.error`
- `urllib.parse`: url解析模块
- `urllib.robotparse`: robots.txt解析模块

> python3中urllib, urllib2合并为[urllib](https://docs.python.org/3/howto/urllib2.html#urllib-howto); 现在一般用的requests, 而不用urllib; [requests vs urllib](https://www.cnblogs.com/znyyy/p/7868511.html)

```py
# python2
import urllib2
response = urllib2.urlopen('http://www.baidu.com')

# python3
import urllib.request
response = urllib.request.urlopen('http://www.baidu.com')
```

```py
import urllib.parse
import urllib.request

data = bytes(urllib.parse.urlencode({'word': 'hello'}), encoding='utf8')
response = urllib.request.urlopen('http://httpbin.org/post', data=data, timeout=0.1)
print(response.read())
```

```py
import socket
import urllib.request
import urllib.error

try:
    response = urllib.request.urlopen('http://httpbin.org/get', timeout=0.1)
except urllib.error.URLError as e:
    if isinstance(e.reason, socket.timeout):
        print('TIME OUT')
```

```py
import urllib.request

response = urllib.request.urlopen('https://www.python.org')
print(response.status)
print(response.getcode()) # 200
print(response.getheaders())
print(response.getheader('Server'))
```

`response.getcode()` return:
- 2开头，网页正常
- 4开头，网页不存在
- 5开头，服务器bug

```py
from urllib import request, parse

url = 'http://httpbin.org/post'
headers = {
    'User-Agent': 'Mozilla/4.0 (compatible; MSIE 5.5; Windows NT)',
    'Host': 'httpbin.org'
}
dict = {
    'name': 'Grey'
}
data = bytes(parse.urlencode(dict), encoding='utf8')
req = request.Request(url=url, data=data, headers=headers, method='POST')
response = request.urlopen(req)
print(response.read().decode('utf-8'))
```

```py
from urllib import request, parse

url = 'http://httpbin.org/post'
dict = {
    'name': 'Germey'
}
data = bytes(parse.urlencode(dict), encoding='utf8')
req = request.Request(url=url, data=data, method='POST')
req.add_header('User-Agent', 'Mozilla/4.0 (compatible; MSIE 5.5; Windows NT)')
response = request.urlopen(req)
print(response.read().decode('utf-8'))
```

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
# simple example
import requests

r=requests.get('https://www.python.org/')
html=r.text
# binary_html=r.content
print(r.status_code) # 200
```

>使用response.text 时，Requests 会基于 HTTP 响应的文本编码自动解码响应内容，大多数 Unicode 字符集都能被无缝地解码。  
>使用response.content 时，返回的是服务器响应数据的原始二进制字节流，可以用来保存图片等二进制文件。

```python
# request without parameter
imprt requests

headers={
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; WOW64; rv:61.0) Gecko/20100101 Firefox/61.0",
}
# 这个网站的response给出的就是request的各种信息
response=requests.get('http://httpbin.org/get', headers=headers)
print(response.text) # utf8 decoded content
print(response.content) # binary content
print(response.url) # http://httpbin.org/get
print(response.encoding) # utf-8
print(response.status_code) # 200
```

```python
# request with parameters
import requests

headers={
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; WOW64; rv:61.0) Gecko/20100101 Firefox/61.0",
}
params={
    'q':'hello',
}
response=requests.get("https://cn.bing.com/search", headers=headers, params=params)

print(response.text)
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

[Other Image](https://www.lsmpx.com/): 多线程下载图片

```py
import requests
import re
import concurrent.futures

s=requests.Session()
s.headers.update({"User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:70.0) Gecko/20100101 Firefox/70.0",})

pat1=re.compile(r'<h2><a href="thread-(\d+)-1-1.html"')

def get_ids(pg):
    r=s.get(f'https://www.lsmpx.com/plugin.php?id=group&page={pg}').text
    return pat1.findall(r)

def get_id_list(total_pages):
    with concurrent.futures.ThreadPoolExecutor(max_workers=64) as executor:
        forecast_futures = [executor.submit(get_ids, pg) for pg in range(1, total_pages+1)]
        id_list=[]
        for f in forecast_futures:
            id_list.extend(f.result())
        return id_list

pat2=re.compile(r'<li><img alt=".+?" src="(.+?)"')

def get_img_urls(ID):
    img_urls=[]
    for i in range(5):
        url=f'https://www.lsmpx.com/thread-{ID}-{i+1}-1.html'
        r=s.get(url).text
        img_urls.extend(pat2.findall(r))
    return url, img_urls

def download_img(ID):  
    url, img_urls=get_img_urls(ID)
    for i, url in enumerate(img_urls):
        with open(f'test/{ID}-{i}.jpg', 'wb') as file:
            r=s.get(url, headers={'Referer': url}).content
            file.write(r) 

def download_images(id_list):
    with concurrent.futures.ThreadPoolExecutor(max_workers=128) as executor:
        forecast_futures = [executor.submit(download_img, ID) for ID in id_list]

if __name__ == "__main__":
    id_list=get_id_list(264)
    print(len(id_list))
    download_images(id_list)
```

example: 多线程下载视频

```py
import requests
import concurrent.futures

s=requests.Session()
s.headers.update({"Referer": "https://avgle.com/video/Z4FrtreTcrr/mywife-696-%E4%BB%B2%E9%87%8C%E7%B5%B5%E9%87%8C%E5%AD%90-%E5%88%9D%E6%9C%83%E7%AF%87",})

def download(index):
    url=f'https://ip78770016.cdn.qooqlevideo.com/key=zD8vdkaOHIjJCD25erI9Dg,s=,end=1575904855,limit=2/data=1575904855/state=Wfip/referer=force,.avgle.com/reftag=56109644/media=hlsA/ssd4/177/6/62849406.mp4/seg-{index}-v1-a1.ts'
    r=s.get(url, stream=True)
    with open(f'test/{index:03d}.ts', 'wb') as file:
        for chunk in r.iter_content(chunk_size=1024 * 1024):
            if chunk:
                file.write(chunk)

with concurrent.futures.ThreadPoolExecutor(max_workers=64) as executor:
    for i in range(400):
        executor.submit(download, i+1)
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

### requests HTTP Auth

> 访问网页的时候弹窗要输入user, password。比如路由器的登陆，basic auth容易破解。

[Authentication](http://docs.python-requests.org/en/master/user/authentication/), [Usage](https://www.jianshu.com/p/18fb07f2f65e):
- Basic Authentication
- Digest Authentication
- OAuth Authentication

```python
import requests

s=requests.Session()
# basic authentication
r=s.get('http://pythonscraping.com/pages/auth/login.php', auth=('user', '123456'))
r.text
```

### request json

```python
# get请求
import requests
url='http://127.0.0.1:1990/login'
data={"username":"admin","password":123456}
res=requests.get(url,data)
#res=res.text#text方法是获取到响应为一个str，也不需要对res进行转换等处理
res=res.json()#当返回的数据是json串的时候直接用.json即可将res转换成字典
print(res)

#post请求
import requests
url='http://127.0.0.1:1990/login'
data={"username":"admin","password":123456}
res=requests.post(url,data)
#res=res.text#text方法是获取到响应为一个str，也不需要对res进行转换等处理
res=res.json()#当返回的数据是json串的时候直接用.json即可将res转换成字典
print(res)

#当传参格式要求为json串时
import requests
url='http://127.0.0.1:1990/login'
data={"username":"admin","password":123456}
res=requests.post(url,json=data)#只需要在这里指定data为json即可
#res=res.text#text方法是获取到响应为一个str，也不需要对res进行转换等处理
res=res.json()#当返回的数据是json串的时候直接用.json即可将res转换成字典
print(res)

#传参含cookie
import requests
url='http://127.0.0.1:1990/login'
data={"username":"admin","password":123456}
cookie={"sign":"123abc"}
res=requests.post(url,json=data,cookies=cookie)#只需要在这里指定cookies位cookie即可，headers，files等类似
res=res.json()
print(res)
```

```python
# simple example
import requests

r=requests.get('https://www.lagou.com/lbs/getAllCitySearchLabels.json').json()
print(type(r)) # <class 'dict'>
```

[jsonpath](https://github.com/json-path/JsonPath)类似xpath,语法不同

```python
import requests
import jsonpath

r=requests.get('https://www.lagou.com/lbs/getAllCitySearchLabels.json').json()
jsonpath.jsonpath(r, '$..name')  # 找出根节点下所有的key='name'的value
jsonpath.jsonpath(r, '$..allCitySearchLabels.A') # allCity下面的所有A节点
jsonpath.jsonpath(r, '$..A.*') # A下面所有的节点
```

上传文件

```python
import requests

url='http://httpbin.org/post/'
myfile={'file': open('pig.jpg', 'rb')}
response=requests.post(url, files=myfile)
print(response.text)
```

### requests session

模拟用户登陆;

>在 requests 里，session对象是一个非常常用的对象，这个对象代表一次用户会话：从客户端浏览器连接服务器开始，到客户端浏览器与服务器断开。
>
>会话能让我们在跨请求时候保持某些参数，比如在同一个 Session 实例发出的所有请求之间保持 cookie

```python
import requests

s=requests.Session()
data={'username':'Grey', 'password':'password'}

# session可以记录cookie
r=s.post('http://pythonscraping.com/pages/cookies/welcome.php', data=data)
print(s.cookies.get_dict()) # {'loggedin': '1', 'username': 'Grey'}

# visit another site
s.get('http://pythonscraping.com/pages/cookies/welcome.php').text
# login success!
```

处理https请求

```python
import requests
# verify默认是True
response = requests.get("https://www.baidu.com/", verify=True)
print(response.text)
```

如果SSL证书验证不通过，或者不信任服务器的安全证书，则会报出SSLError; 比如12306

```python
import requests
response = requests.get("https://www.12306.cn/mormhweb/", verify=False)
print response.text
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

## selenium

### selenium + chrome

> 先要下载[chromedriver](http://chromedriver.chromium.org/downloads)

```python
from selenium import webdriver

options = webdriver.ChromeOptions()
options.binary_location = 'D:/Cent/chrome.exe'
driver = webdriver.Chrome('chromedriver.exe', chrome_options=options)

url = 'http://www.cmsoft.cn/'
driver.get(url)
page_source = driver.page_source
driver.close()

print(page_source) # 包含了动态生成的html
```

### selenium + phantomjs

先要下载[phantomjs](http://phantomjs.org/)

```python
from selenium import webdriver

driver=webdriver.PhantomJS('phantomjs.exe')

url = 'http://www.cmsoft.cn/'
driver.get(url)
page_source = driver.page_source
driver.close()

print(page_source)
```

```python
# phantom with user-agents
from selenium import webdriver
from selenium.webdriver.common.desired_capabilities import DesiredCapabilities

dcap = dict(DesiredCapabilities.PHANTOMJS)
dcap["phantomjs.page.settings.userAgent"] = ("Mozilla/5.0 (Windows NT 10.0; WOW64; rv:62.0) Gecko/20100101 Firefox/62.0")
driver = webdriver.PhantomJS('phantomjs.exe', desired_capabilities=dcap)

url='https://httpbin.org/get'
driver.get(url)
print(driver.page_source)

driver.close()
```

### selenium with firefox

```python
from selenium import webdriver

opt=webdriver.FirefoxOptions()
opt.headless=False
# # firefox disable js
# opt.set_preference("javascript.enabled", False)

driver = webdriver.Firefox(firefox_binary="D:/Browser/Firefox/firefox.exe", executable_path='geckodriver.exe',firefox_profile=r'D:\Browser\Firefox\Profile\Mozilla\Firefox\Profiles\04q5ed1u.default-release', options=opt)
driver.get('https://m.huxiu.com')
driver.save_screenshot('huxiu.png')
driver.quit()
```

example: selenium with proxy
> 获取Network Panel中的所有文件并且提取文件

```py
import time
import json
import pandas as pd
from selenium import webdriver
from browsermobproxy import Server

# create proxy server
server = Server(r"D:\Browser\BrowserDriver\browsermob-proxy\bin\browsermob-proxy.bat")
server.start()
proxy = server.create_proxy()
proxy.new_har('simuwang', options={'captureHeaders': False, 'captureContent': True})

# config firefox
opt = webdriver.FirefoxOptions()
opt.set_preference('network.proxy.type', 1)
# proxy all http & https requests
opt.set_preference('network.proxy.http', '127.0.0.1')
opt.set_preference('network.proxy.http_port', proxy.port)
opt.set_preference('network.proxy.ssl', '127.0.0.1')
opt.set_preference('network.proxy.ssl_port', proxy.port)

driver = webdriver.Firefox(
    firefox_binary="D:/Browser/Firefox/firefox.exe",
    executable_path=r"D:\Browser\BrowserDriver\geckodriver.exe",
    firefox_profile=r'D:\Browser\Firefox\Profile\Mozilla\Firefox\Profiles\04q5ed1u.default-release',
    options=opt)

# visit target site(因为设置了firefox_profile， 所以登录之后cookie能够保持)
driver.get('https://dc.simuwang.com/product/HF00003LPV.html')
time.sleep(5)
driver.quit()

# get all data in proxy server(json)
result = proxy.har

# filter to get target response
response = None
for entry in result['log']['entries']:
    url = entry['request']['url']
    if 'index.php?c=Chart&a=jzdb_fund' in url:
        response = entry['response']['content']['text']

response_json = json.loads(response)

# get date
date = response_json['categories']
# get 沪深300
hs300 = response_json['data'][1]
# get 融智-中性优选20指数
index20 = response_json['data'][2]
# 累计增长率
data0 = response_json['data'][0]
accu_growth_rate = [d['value'] for d in data0]

# summary to a DataFrame
df = pd.DataFrame({
    'Date': date,
    'accu_growth_rate': accu_growth_rate,
    'hs300': hs300,
    'index20': index20
})
df['accu_netvalue'] = df.accu_growth_rate + 1.0
df['accu_growth_rate'] = df.accu_growth_rate * 100.0
df['hs300'] = df.hs300 * 100
df['index20'] = df.index20 * 100

# post processing
df = df.round({'accu_netvalue': 4, 'accu_growth_rate': 1,'hs300': 2, 'index20': 2})
df.set_index('Date', inplace=True)

# save to file
df.to_csv('firefox.csv')
```

### selenium keys & click

> 可用于点击登陆，自动注册，也可用于普通的点击`next`翻页

example1: selenium send keys

```python
from selenium import webdriver
from selenium.webdriver.common.keys import Keys

options = webdriver.ChromeOptions()
options.binary_location = 'D:/Cent/chrome.exe'

driver = webdriver.Chrome('chromedriver.exe', chrome_options=options)
driver.get('https://www.baidu.com/')

kw=driver.find_element_by_id('kw') # F12找到输入框的位置
kw.send_keys('python3')
kw.send_keys(Keys.RETURN)

print(driver.page_source)

driver.close()
```

example2: selenium click

```python
from selenium import webdriver
from selenium.webdriver.common.keys import Keys

options = webdriver.ChromeOptions()
options.binary_location = 'D:/Cent/chrome.exe'

driver = webdriver.Chrome('chromedriver.exe', chrome_options=options)
driver.get('https://www.baidu.com/')
kw=driver.find_element_by_id('kw')
kw.send_keys('python3')

btn=driver.find_element_by_id('su')
btn.click()

print(driver.page_source)
driver.close()
```

example3: selenium with action

```python
from selenium import webdriver
from selenium.webdriver.common.keys import Keys

driver=webdriver.Firefox()
driver.get('https://www.baidu.com')

setting_link=driver.find_element_by_link_text('设置')
action=webdriver.ActionChains(driver)
action.move_to_element(setting_link)
action.perform()

kw=driver.find_element_by_id('kw')
kw.send_keys('hello')
kw.send_keys(Keys.CONTROL, 'a')
```

example4: selenium explit wait

```python
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC

ff = webdriver.Firefox()
ff.get('https://www.baidu.com')

kw=WebDriverWait(ff, timeout=5, poll_frequency=0.5).until(EC.presence_of_element_located((By.ID, "kw")))
kw.send_keys('hello')

ff.close()
```

example5: selenium implicit wait

> **Implicit wait** is applicable for all the web elements where as **Explicit wait** is applicable only for the element

```python
from selenium import webdriver

ff = webdriver.Firefox()
ff.get('https://www.baidu.com')

ff.implicitly_wait(5)

kw=ff.find_element_by_id('kw')
kw.send_keys('hello')

ff.close()
```

example6: find_elements

```python
from selenium import webdriver

ff = webdriver.Firefox()
ff.get('https://www.baidu.com')

kw=ff.find_element_by_id('kw')
kw.send_keys('hello')

ff.implicitly_wait(3)

for a in ff.find_elements_by_css_selector('.t a'):
    print(a.text)
```

example7: multi-windows

```python
from selenium import webdriver

ff = webdriver.Firefox()
ff.get('https://www.baidu.com')

login=ff.find_element_by_link_text('登录')
login.click()
ff.implicitly_wait(3)

register=ff.find_element_by_link_text('立即注册')
register.click()
ff.implicitly_wait(3)

ff.switch_to_window(ff.window_handles[1])

user=ff.find_element_by_id('TANGRAM__PSP_3__userName')
phone=ff.find_element_by_id('TANGRAM__PSP_3__phone')
user.send_keys('DeltaGrey')
phone.send_keys('110')

ff.close() # close page
ff.quit() # quit browser
```

example8: dropdown menu & alert window

```python
from selenium import webdriver

ff = webdriver.Firefox()
ff.get('https://www.baidu.com')

setting_link=ff.find_element_by_link_text('设置')
webdriver.ActionChains(ff).move_to_element(setting_link).perform()

search_setting=ff.find_element_by_link_text('搜索设置')
search_setting.click()
ff.implicitly_wait(3)

drop_menu=ff.find_element_by_id('nr')
select=webdriver.support.select.Select(drop_menu)
select.select_by_index(2)

ff.find_element_by_link_text('保存设置').click()
ff.switch_to_alert().accept()
```

example9: scroll window

```python
import time
from selenium import webdriver

ff = webdriver.Firefox()
ff.get('https://www.baidu.com')

kw=ff.find_element_by_id('kw')
kw.send_keys('python3')
kw.submit()

time.sleep(1)

# execute js
ff.execute_script('window.scrollTo(0, 800);')
```

example10: video

```python
from selenium import webdriver

ff = webdriver.Firefox()
ff.get('https://videojs.com/')
ff.implicitly_wait(3)

ff.find_element_by_class_name('vjs-big-play-button').click()
# ff.execute_script('document.getElementsByClassName("vjs-big-play-button")[0].click();')
```

example11: selenium download

```python
# firefox download
from selenium import webdriver

options=webdriver.FirefoxOptions()
options.set_preference("browser.download.dir", "c:\\") 
options.set_preference("browser.download.folderList",2) # use lastdownload dir
options.set_preference("browser.helperApps.neverAsk.saveToDisk", "binary/octet-stream") # specify file type

ff = webdriver.Firefox(options=options)
ff.get('https://pypi.org/project/selenium/#files')

# click link to dowload
ff.find_element_by_partial_link_text('tar.gz').click()
```

```python
# chrome download
from selenium import webdriver

options=webdriver.ChromeOptions()
options.add_experimental_option('prefs', {'download.prompt_for_download': False, 'download.default_directory': 'c:\\'})
options.binary_location='D:/Cent/chrome.exe'

chrome=webdriver.Chrome(chrome_options=options)

chrome.get('https://pypi.org/project/selenium/#files')
chrome.implicitly_wait(5)
chrome.find_element_by_partial_link_text('tar.gz').click()
```

### selenium with dynamic page

如果碰到frame, 需要先切换frame

```python
# qzone login
from selenium import webdriver

driver=webdriver.Firefox()
driver.get('https://qzone.qq.com/')

driver.switch_to_frame('login_frame') # switch frame
# 动态网页必须使用css_selector, 其他的无效
driver.find_element_by_css_selector('#switcher_plogin').click() # 账号密码登录

user=driver.find_element_by_css_selector('#u')
password=driver.find_element_by_css_selector('#p')
login_btn=driver.find_element_by_css_selector('#login_button')

user.send_keys('666666666')
password.send_keys('999999999')
login_btn.click()
# driver.close()
```

### selenium mobile emulation

```python
# firefox
from selenium import webdriver

options=webdriver.FirefoxOptions()
options.set_preference("general.useragent.override", 'iPhone X')

driver=webdriver.Firefox(options=options)
driver.set_window_size(375, 750)

driver.get('https://www.baidu.com')
driver.close()
```

```python
# chrome
import time
from selenium import webdriver

options = webdriver.ChromeOptions()
options.binary_location='D:/Cent/chrome.exe'
options.add_experimental_option("mobileEmulation", { "deviceName": "iPhone X" })

driver=webdriver.Chrome(chrome_options=options)

driver.get('https://www.taobao.com')
driver.get('https://www.jd.com')

time.sleep(3)
driver.back()
time.sleep(3)
driver.forward()
time.sleep(3)
driver.refresh()
driver.close()
```

> Tips: selenium doesn't load picture

```python
from selenium import webdriver

options=webdriver.FirefoxOptions()
options.set_preference('permissions.default.image', 2)
ff=webdriver.Firefox(options=options)

ff.get('https://www.baidu.com')
```

## login with cookie

浏览器访问baidu的标准Request Headers

```
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3
Accept-Encoding: gzip, deflate, br
Accept-Language: en-US,en;q=0.9,zh-CN;q=0.8,zh-TW;q=0.7,zh;q=0.6,ja;q=0.5
Cache-Control: max-age=0
Connection: keep-alive
Cookie: xxxx
Host: www.baidu.com
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/74.0.3729.169 Safari/537.36
```

Attention:
- Accept: 可以不写
- Accept-Encoding: 不能写，否则还需要解压数据
- Accept-Language: 可以不写
- Cache-Control: 可以不写
- Connection: keep-alive，模拟登录的时候需要
- Cookie: 模拟登录的时候需要
- Host:可以不写
- Upgrade-Insecure-Requests: 可以不写
- User-Agent必须写

[requests模拟登录的原理](https://blog.csdn.net/zwq912318834/article/details/79571110)

[requests模拟登录的三种方法](https://blog.csdn.net/hui1788/article/details/79944102):
- method1利用session模拟登陆
- method2将cookie放在headers中一起发送请求就可以了
- method3将cookie值拿出来直接在requests中发送就可以了，不过在发送时我们需要把cookie字符串转化为字典。

### method1: only with session

> 标准做法是用Fiddler来抓取post的内容: Find: post→Inspector→TextView  
> Fiddler也可以获取header: 一般写上User-Agent, Refer就行了  
> Fiddler抓https的时候,同时开python爬虫，可能报错，python需要忽略ssl安全: `requests.get('xxx', verify=False)`

[百度登录加密协议分析](https://www.jianshu.com/p/8e83974d2b04), [后台数据库密文](https://www.zhihu.com/question/55202145)

```python
# method1: only with session, 不稳定
import requests

headers = {
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; WOW64; rv:61.0) Gecko/20100101 Firefox/61.0",
}

# login
s = requests.session()
login_url = 'https://passport.baidu.com/'
username = 'XXXXXX'
password = 'YYYYYY'
raw_post = {"username": username, "password": password}
s.post(login_url, headers=headers, data=raw_post)

# visited other sites
url = 'https://www.baidu.com/'
r = s.get(url, headers=headers)
print(r.content.decode('utf8'))
```

### method2&3: cookie with request

F12/Network/Headers/RequestHeaders/Cookie复制字符串
> ![](res/cookie_login01.png)

```python
# method2: cookie in header, 最简单
import requests

cookie_str = 'BIDUPSID=9E547011EFA2D219E2FEC5BEDF7B9475; PSTM=1537057280; BD_UPN=12314753; BAIDUID=4594E9308E15EE0085797C1212051F67:FG=1; BDUSS=WdCMGQwRFJHUHo1YjFoa3VXa1JIRDQ0bjRNUmMzTXd-TU5rd2ljLUVlSG1OOGRiQVFBQUFBJCQAAAAAAAAAAAEAAAC6p~WiQWxwaGFHcmV5AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAOaqn1vmqp9bd; cflag=15%3A3; delPer=1; BD_HOME=1; H_PS_PSSID=1438_26965_21116_22157; sug=3; sugstore=0; bdime=0; ORIGIN=2; ISSW=1'
headers = {
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; WOW64; rv:61.0) Gecko/20100101 Firefox/61.0",
    "Cookie": cookie_str
}
url = 'https://www.baidu.com/'


r = requests.get(url, headers=headers)
print(r.content.decode('utf8'))
```

```python
# method3: cookie in requests
import requests

headers={
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; WOW64; rv:61.0) Gecko/20100101 Firefox/61.0",
}
url='https://www.baidu.com/'


cookie_str='BIDUPSID=9E547011EFA2D219E2FEC5BEDF7B9475; PSTM=1537057280; BD_UPN=12314753; BAIDUID=4594E9308E15EE0085797C1212051F67:FG=1; BDUSS=WdCMGQwRFJHUHo1YjFoa3VXa1JIRDQ0bjRNUmMzTXd-TU5rd2ljLUVlSG1OOGRiQVFBQUFBJCQAAAAAAAAAAAEAAAC6p~WiQWxwaGFHcmV5AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAOaqn1vmqp9bd; cflag=15%3A3; delPer=1; BD_HOME=1; H_PS_PSSID=1438_26965_21116_22157; sug=3; sugstore=0; bdime=0; ORIGIN=2; ISSW=1'
cookie_dict={item.split('=')[0]:item.split('=')[1] for item in cookie_str.split('; ')}

r=requests.get(url, headers=headers, cookies=cookie_dict)
print(r.content.decode('utf8'))
```

## word cloud

因为英文单词之间有空格分隔，中文没有， 所以需要用[jieba](https://github.com/fxsjy/jieba)来分开词语。然后用[wordcloud](https://github.com/amueller/word_cloud)来生成词云

- `pip install jieba`
- `pip install wordcloud`

```python
# english simple
from wordcloud import WordCloud
import matplotlib.pyplot as plt

file_content=open('alice.txt').read()
wc=WordCloud(width=1000, height=860).generate(file_content)

plt.imshow(wc)
plt.axis('off')
plt.show()

wc.to_file('test.png')
```

```python
# english with picture example
from wordcloud import WordCloud
import matplotlib.pyplot as plt
import numpy as np
from PIL import Image

file_content=open('alice.txt').read()
alice_coloring=np.array(Image.open('alice_color.png'))

wc=WordCloud(width=1000, height=860, mask=alice_coloring).generate(file_content)    
plt.imshow(wc)
plt.axis('off')
plt.show()

wc.to_file('test.png')
```

```python
# chinese with picture example
import jieba
from wordcloud import WordCloud
import matplotlib.pyplot as plt
import numpy as np
from PIL import Image

file_content=open('xiao.txt', encoding='utf8').read()
seg_list=jieba.cut(file_content, cut_all=False)
# 构造成类似英文分隔
text=' '.join(seg_list)

alice_coloring=np.array(Image.open('alice_color.png'))

# 必须有font_path, 使用的是环境变量中的Microsoft Yahei或者font_path='simkai.ttf'
wc=WordCloud(width=1000, height=860, mask=alice_coloring,font_path="msyh.ttc").generate(text)

plt.imshow(wc)
plt.axis('off')
plt.show()

wc.to_file('test.png')
```

## Periodic Sign Task

Linux: `nohup python task.py &`
> kill task: `lsof nohup.out`, `kill -9 32767`

```python
# task.py
import re
import time
import threading
import requests

file = open('log.txt', 'w')

# Construct headers
headers = {
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; WOW64; rv:61.0) Gecko/20100101 Firefox/61.0",
    # Copy Cookie from browser
    "Cookie": 'xxxx'
}


def task():
    # get normal coin
    sign_url = 'https://www.pdawiki.com/forum/dsu_paulsign-sign.html'
    r1 = requests.get(sign_url, headers=headers)
    pat1 = re.compile('formhash=(.+)"')

    data = {
        "formhash": pat1.search(r1.text).group(1),
        "qdxq": 'kx',
        "qdmode": 1,
        "todaysay": '我又来签到了',
        "fastreply": 0,
    }

    post_url = 'https://www.pdawiki.com/forum/plugin.php?id=dsu_paulsign:sign&operation=qiandao&infloat=1&inajax=1'
    r2 = requests.post(post_url, headers=headers, data=data)

    if r2.text.find('恭喜你签到成功'):
        file.write(f'心情签到成功:{time.ctime()}\n')
        file.flush()

    # get another 20 coins
    panel_url = 'https://www.pdawiki.com/forum/plugin.php?id=nimba_days:html&formhash=8b7c3748&infloat=yes&handlekey=nimba_days&inajax=1&ajaxtarget=fwin_content_nimba_days'
    r3 = requests.get(panel_url, headers=headers)
    pat2 = re.compile(r"NimbaAjax\('(.+)'\);return false;")
    coin_url = f'https://www.pdawiki.com/forum/{pat2.search(r3.text).group(1)}'
    r4 = requests.get(coin_url, headers=headers)

    if r4.text.find('积分奖励领取成功'):
        file.write(f'20金币签到成功:{time.ctime()}\n')
        file.flush()

    # periodic task
    threading.Timer(24*3600, task).start()

task()
```

## XPath

BeautifulSoup, XPath适合于表格的数据提取; regex适合精确的提取小部分位置的数据；
> BeautifulSoup比xpath慢，BeautifulSoup更加user-friendly
> xpath对[empty string](https://stackoverflow.com/questions/2420705)支持不好

example1: xpath selector

```python
from lxml import etree

html='''
<div>
<ul>
    <li class="item-0"><a href="link1.html">0 item</a></li>
    <li class="item-1"><a href="link2.html">1 item</a></li>
    <li class="item-inactive"><a href="link3.html">2 item</li>
    <li class="item-3"><a href="link4.html">3 item</a></li>
    <li class="item-4"><a href="link5.html">4 item</a></li>    
</ul>
<ul>
    <li class="item-10"><a href="link11.html">10 item</a></li>
    <li class="item-11"><a href="link12.html">11 item</a></li>
    <li class="item-inactive"><a href="link13.html">12 item</li>
    <li class="item-13"><a href="link14.html">13 item</a></li>
    <li class="item-14"><a href="link15.html">14 item</a></li>    
</ul>
</div>
'''

tree=etree.HTML(html) # or lxml.etree.parse('index.html') # parse file

for ul in tree.xpath('//ul'):
    for a in ul.xpath('.//li/a'):
        print(a.text, end=';')
# 0 item;1 item;2 item;3 item;4 item;10 item;11 item;12 item;13 item;14 item;

print(tree.xpath('//li/@class'))
# ['item-0', 'item-1', 'item-inactive', 'item-3', 'item-4', 'item-10', 'item-11', 'item-inactive', 'item-13', 'item-14']

print(tree.xpath('//li/a/@href'))
print(tree.xpath('//ul/li//@href'))
# ['link1.html', 'link2.html', 'link3.html', 'link4.html', 'link5.html', 'link11.html', 'link12.html', 'link13.html', 'link14.html', 'link15.html']

print(tree.xpath('//li/a/text()')) # text is particular
# ['0 item', '1 item', '2 item', '3 item', '4 item', '10 item', '11 item', '12 item', '13 item', '14 item']

print(tree.xpath('//li/a/@href="link3.html"')) # True

tree.xpath('//li[@class="itme-10"]/a/@href') # link11.html

# get the first one
print(tree.xpath('//li[1]/a/@href'))
# ['link1.html', 'link11.html']

# get the last one
print(tree.xpath('//li[last()]'))
# [<Element li at 0x23231e04c08>, <Element li at 0x23231e040c8>]
print(tree.xpath('//li[last()]/a/@href'))
# ['link5.html', 'link15.html']

# match: *表示匹配所有
tree.xpath('//*[@class="item-inactive"]/a/@href')
# ['link3.html', 'link13.html']
```

example2: xpath get proxy ip

```python
import requests
from lxml import etree

headers = {
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; WOW64; rv:61.0) Gecko/20100101 Firefox/61.0",
}

html=requests.get('http://www.ip3366.net/free/', headers=headers).content.decode('gbk')
tree=etree.HTML(html)
tree.xpath('//tr/td/text()')
```

xpath模糊查询(id属性模糊包含某些字符串): `//div[contains(@id, "qiushi_tag")]//h2`

example: requests with xpath

```py
from lxml import html
import requests
import time

for i in range(5):
    url=f'https://www.kuaidaili.com/free/inha/{i+1}/'
    r=requests.get(url, headers={'User-Agent':"Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:71.0) Gecko/20100101 Firefox/71.0"})
    time.sleep(1)
    tree=html.fromstring(r.text)
    proxy_list = tree.xpath('//tbody/tr')
    for p in proxy_list:
        ip = p.xpath('./td[@data-title="IP"]/text()')
        port = p.xpath('./td[@data-title="PORT"]/text()')
        anonymous = p.xpath('./td[@data-title="匿名度"]/text()')
        protocol_type = p.xpath('./td[@data-title="类型"]/text()')

        print(ip, port, anonymous, protocol_type)
```

## OCR vs verify code

最简单的做法
1. 将图片下载到本地
2. 程序暂停等待用户输入
3. 手动或者自动打开图片
4. 用户输入验证码，继续程序

[tesseract](https://github.com/tesseract-ocr/tesseract)使用: `tesseract.exe c:\3.jpg c:\result`, 将识别的结果放到了`c:\result.txt`
> [tesseract for chinese](https://www.cnblogs.com/wzben/p/5930538.html), 或者用[baiduAI](https://ai.baidu.com/)  
> 人工智能: 人工训练出来的智能

```python
import subprocess

p=subprocess.Popen(['C:\\Program Files (x86)\\Tesseract-OCR\\tesseract.exe','c:\\3.jpg','c:\\result'], 
                    stdout=subprocess.PIPE,
                    stderr=subprocess.PIPE)
p.wait()
with open('c\\result.txt', 'r') as file:
    print(file.read())
```

## requests vs selenium

|  | selenium | requests |
|---|----|-----|
| 模拟登录 | 不需要复杂的加密过程 | 面对高端网站模拟登陆需要分析登陆协议 |
| 速度 | 慢 | 快 |

> 低端网站只用requests  
> 高端网站用selenium获取cookies, 然后使用requests带着cookies访问

jd, tmall存在验证码，所以需要手动，无法全自动;

```python
# baidu: selenium + requests
from selenium import webdriver
import requests

ff=webdriver.Firefox()
ff.get('https://www.baidu.com')

ff.find_element_by_link_text('登录').click()
ff.find_element_by_id('TANGRAM__PSP_10__footerULoginBtn').click()

user=ff.find_element_by_id('TANGRAM__PSP_10__userName')
password=ff.find_element_by_id('TANGRAM__PSP_10__password')
user.send_keys('13810455459')
password.send_keys('1kdlsJBVguW720')
ff.find_element_by_id('TANGRAM__PSP_10__submit').click()

cookies={}
for cookie in ff.get_cookies():
    cookies[cookie['name']]=cookie['value']

# request use selenium's cookies
s=requests.Session()
headers={
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; WOW64; rv:61.0) Gecko/20100101 Firefox/62.0",
}
r=s.get('https://www.baidu.com', cookies=cookies, headers=headers)
```

## pyquery

`conda install pyquery`, [Tutorials](https://www.cnblogs.com/zhaof/p/6935473.html)

4 methods:
- re: 最快
- xpath: 中等
- BeautifulSoup：最慢，易用
- pyquery: easy, 类似jQuery的语法


```python
import pyquery

html='''
<div>
<h3>this is header</h3>
<p id="p1">paragraph 1</p>
<ul>
    <li class="item-0"><a href="link1.html">0 item</a></li>
    <li class="item-1"><a href="link2.html">1 item</a></li>
    <li class="item-inactive"><a href="link3.html">2 item</li>
    <li class="item-3"><a href="link4.html">3 item</a></li>
    <li class="item-4"><a href="link5.html">4 item</a></li>    
</ul>
<ul>
    <li class="item-10"><a href="link11.html">10 item</a></li>
    <li class="item-11"><a href="link12.html">11 item</a></li>
    <li class="item-inactive"><a href="link13.html">12 item</li>
    <li class="item-13"><a href="link14.html">13 item</a></li>
    <li class="item-14"><a href="link15.html">14 item</a></li>    
</ul>
</div>
'''

doc=pyquery.PyQuery(html)

print(doc('h3')) # <h3>this is header</h3>
print(doc('h3').text()) # this is header
print(doc('#p1')) # <p id="p1">paragraph 1</p>

print(doc('[class]')) # find tag with class attribute
print(doc('[class=item-inactive]')) # find class=item-inactive

print(doc('p').attr['id']) # p1
print(doc('p').attr('id')) # p1
print(doc('p').attr.id) # p1

# Traversal
for li in doc('li').items():
    print(li.text(), end='; ')
# 0 item; 1 item; 2 item; 3 item; 4 item; 10 item; 11 item; 12 item; 13 item; 14 item; 

for li in doc('li'):
    print(doc(li).text(), end='; ')
# 0 item; 1 item; 2 item; 3 item; 4 item; 10 item; 11 item; 12 item; 13 item; 14 item; 

for a in doc('li a'):
    print(a.text, end='; ')
# 0 item; 1 item; 2 item; 3 item; 4 item; 10 item; 11 item; 12 item; 13 item; 14 item; 

for a in doc('li').find('a'):
    print(a.text, end='; ')
# 0 item; 1 item; 2 item; 3 item; 4 item; 10 item; 11 item; 12 item; 13 item; 14 item; 

for li in doc('ul').children():
    print(doc(li).text(), end='; ')
# 0 item; 1 item; 2 item; 3 item; 4 item; 10 item; 11 item; 12 item; 13 item; 14 item;   
```

## Onlilne File Spider

example: read online pdf
> `pip install pdfminer`

```py
from pdfminer.pdfinterp import PDFResourceManager, PDFPageInterpreter
from pdfminer.layout import LAParams
from pdfminer.converter import TextConverter
from pdfminer.pdfpage import PDFPage
import io
import urllib.request
import requests

def pdf_to_text(pdf_file):
    memory_file = io.StringIO()

    rsrcmgr = PDFResourceManager()
    device=TextConverter(rsrcmgr, memory_file, laparams=LAParams())
    interpreter = PDFPageInterpreter(rsrcmgr, device)
    for page in PDFPage.get_pages(pdf_file, pagenos=(0, 1, 2)):
        interpreter.process_page(page)
    text=memory_file.getvalue()
    memory_file.close()
    return text

# # local pdf to text
# local_pdf_file=open('UnderstandingGIL.pdf', 'rb')
# print(pdf_to_text(local_pdf_file))

# # online pdf to text by urllib
# online_pdf_file=urllib.request.urlopen('http://www.dabeaz.com/python/UnderstandingGIL.pdf')
# pdf_memory_file=io.BytesIO(online_pdf_file.read())
# print(pdf_to_text(pdf_memory_file))

# online pdf to text by requests
response = requests.get('http://www.dabeaz.com/python/UnderstandingGIL.pdf')
pdf_memory_file = io.BytesIO(response.content)
print(pdf_to_text(pdf_memory_file))
```

example: read online txt

```py
import requests

r=requests.get('http://www.pythonscraping.com/pages/warandpeace/chapter1.txt')
print(r.text)
```

example: read online csv

```py
# csv text reader
import requests
import io
import csv

response=requests.get('http://pythonscraping.com/files/MontyPythonAlbums.csv')
memory_file=io.StringIO(response.text)
for line in csv.reader(memory_file):
    print(line)
```

```py
# csv dictionary reader
import requests
import io
import csv

response=requests.get('http://pythonscraping.com/files/MontyPythonAlbums.csv')
memory_file=io.StringIO(response.text)
reader=csv.DictReader(memory_file) # ['Name', 'Year']
print(reader.fieldnames)
for line in reader: # every line is OrderedDict
    print(line['Name'], '--->', line['Year'])
```

example: read online docx

```py
import requests
import io
import zipfile
import re

response=requests.get('http://pythonscraping.com/pages/AWordDocument.docx')
memory_file=io.BytesIO(response.content)
doc=zipfile.ZipFile(memory_file)
xml=doc.read('word/document.xml').decode('utf8')
pat=re.compile(r'<w:t>(.+?)</w:t>')
print(pat.findall(xml))
```

> `pip install python-docx`

```py
import requests
import io
import docx

response=requests.get('http://pythonscraping.com/pages/AWordDocument.docx')
memory_file=io.BytesIO(response.content)
doc=docx.Document(memory_file)
for p in doc.paragraphs:
    print(p.text)
```