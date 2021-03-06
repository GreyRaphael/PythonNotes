# Scrapy Framework

- [Scrapy Framework](#scrapy-framework)
  - [Introduction](#introduction)
  - [scrapy download image](#scrapy-download-image)
  - [scrapy download files](#scrapy-download-files)
  - [CrawlSpiders](#crawlspiders)
  - [download middleware](#download-middleware)
  - [Other Examples](#other-examples)

## Introduction

![](res/scrapy_architecture_01.png)
> [doc](https://docs.scrapy.org/en/latest/topics/architecture.html)  
> 1. Spiders将很多请求交给Scheduler
> 1. Scheduler将请求交给Downloader从网站下载数据; (proxy作为Scheduler和Downloader之间的中间件)
> 1. Downloader下载的数据交给Spiders如果是Requests，那么重复1-3的过程；如果是数据，将数据交给ItermPipeline

in Anaconda prompt: 
> [scrapy command line](https://docs.scrapy.org/en/latest/topics/commands.html)
- `conda install scrapy`
- `scrapy startproject test1`
- `cd test1`
- `scrapy genspider myspider1 m.huxiu.com`
- when finish file edit, then `scrapy crawl myspider1`

```
C:\USERS\GREY\TEST1
│  scrapy.cfg
└─test1
    │  items.py
    │  middlewares.py
    │  pipelines.py
    │  settings.py
    │  __init__.py
    │
    ├─spiders
       │  myspider1.py
       │  __init__.py
```

```py
# settings.py
# settings.py中可以设置UA, COOKIE, CONCURRENT_REQUESTS....
COOKIES_ENABLED = False

DEFAULT_REQUEST_HEADERS = {
  'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8',
#   'Accept-Language': 'en',
  'User-Agent':"Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:71.0) Gecko/20100101 Firefox/71.0",
}

# ITEM_PIPELINES = {
#     # 数字表示优先级，数字越小优先级越高
#    'test1.pipelines.Test1Pipeline': 300,
# }

# 导出数据格式，默认格式utf8但是这种格式'\u4e00'
FEED_EXPORT_ENCODING='utf8'
```




```py
# myspider1.py
import scrapy

class Myspider1Spider(scrapy.Spider):
    name = 'myspider1'
    allowed_domains = ['m.huxiu.com']
    start_urls = ['https://m.huxiu.com/']

    def parse(self, response):
        with open('index.html', 'wb') as file:
            file.write(response.body)
```

```py
# items.py
import scrapy

class Test1Item(scrapy.Item):
    # define the fields for your item here like:
    link=scrapy.Field()
    content=scrapy.Field()
```

example: get data without pipeline

```py
# myspider1.py
import scrapy
from test1 import items

class Myspider1Spider(scrapy.Spider):
    name = 'myspider1'
    allowed_domains = ['m.huxiu.com']
    start_urls = ['https://m.huxiu.com/']

    def parse(self, response):
        ItemList = []

        # scrapy自带xpath
        div_list = response.xpath('//div[@class="rec-article-info"]')
        for div in div_list:
            MyItem = items.Test1Item()

            link_list = div.xpath('./a/@href').extract()
            content_list = div.xpath('normalize-space(.//a)').extract()

            MyItem['link'] = link_list[0]
            MyItem['content'] = content_list[0]

            ItemList.append(MyItem)
        return ItemList
```

then in Anaconda Prompt:
- `scrapy crawl myspider1 -o huxiu.json`
- `scrapy crawl myspider1 -o huxiu.jl`, json不包含开头结尾的`[]`
- `scrapy crawl myspider1 -o huxiu.csv`
- `scrapy crawl myspider1 -o huxiu.xml`
- `scrapy crawl myspider1 -o ftp://user:pass@ftp.example.com/path/to/huxiu.csv`

example: HelloItem的本质是dictionary
> `scrapy crawl lsm -o data.json`  
> `yield`是为了支持协程

```py
# items.py
import scrapy

class HelloItem(scrapy.Item):
    title=scrapy.Field()
    url=scrapy.Field()
```

```py
# spiders/lsm.py
import scrapy

class LsmSpider(scrapy.Spider):
    name = 'lsm'
    allowed_domains = ['www.lsmpx.com']
    pg=1
    start_urls = ['https://www.lsmpx.com/']

    def parse(self, response):
        for i in response.xpath('//h2/a'):
            data={}
            data['url']=i.xpath('./@href').extract_first()
            data['title']=i.xpath('./text()').extract_first()
            yield data

        if self.pg<5:
            self.pg+=1
        yield scrapy.Request(f'https://www.lsmpx.com/plugin.php?id=group&page={self.pg}', callback=self.parse)
```

example: `spiders.Spider` 模仿`spiders.CrawlSpider`

```py
# spiders/lsm.py
import scrapy

class LsmSpider(scrapy.Spider):
    name = 'lsm'
    allowed_domains = ['www.lsmpx.com']
    start_urls = ['https://www.lsmpx.com/']

    def parse(self, response):
        for i in response.xpath('//h2/a'):
            data={}
            data['url']=i.xpath('./@href').extract_first()
            data['title']=i.xpath('./text()').extract_first()
            yield data

        for j in response.xpath('//div[@class="pg"]//a//@href').extract():
            url=f'https://www.lsmpx.com/{j}'
            yield scrapy.Request(url, callback=self.parse)
```

example: crawl with pipeline
> 可以不写pipeline，使用上个例子`return`+`scrapy crawl myspider -o xxx`的方法

```py
# setttings.py
ITEM_PIPELINES = {
   'test1.pipelines.Test1Pipeline': 300,
}
```

```py
# pipelines.py
import json

class Test1Pipeline(object):
    def __init__(self):
        # utf8和后面的ensure_ascii很重要
        self.file = open('test.json', 'w', encoding='utf8')

    def process_item(self, item, spider):
        json.dump(dict(item), self.file, ensure_ascii=False)
        self.file.write('\n')
        return item

    def close_spider(self, spider):
        self.file.close()
```

```py
# myspider1.py
import scrapy
from test1 import items

class Myspider1Spider(scrapy.Spider):
    name = 'myspider1'
    allowed_domains = ['m.huxiu.com']
    start_urls = ['https://m.huxiu.com/']

    def parse(self, response):
        # scrapy自带xpath
        div_list = response.xpath('//div[@class="rec-article-info"]')
        for div in div_list:
            MyItem = items.Test1Item()

            link_list = div.xpath('./a/@href').extract()
            content_list = div.xpath('normalize-space(.//a)').extract()

            MyItem['link'] = link_list[0]
            MyItem['content'] = content_list[0]

            yield MyItem
```

example: pipeline去重

```py
# pipelines.py
class Test1Pipeline(object):
    def __init__(self):
        self.urls = []

    def process_item(self, item, spider):
        if  spider.name=="lsm":
                if item['url'] in self.urls:
                    raise DropItem(f"Duplicate item found: {item}")
                else:
                    self.urls.add(item['url'])
            return item
```

example: scrapy with pagination

```py
# settings.py
# ITEM_PIPELINES = {
#    'test1.pipelines.Test1Pipeline': 300,
# }

DOWNLOAD_DELAY = 1
```

```py
# items.py
import scrapy

class Test1Item(scrapy.Item):
    ip = scrapy.Field()
    port = scrapy.Field()
    anonymous = scrapy.Field()
    protocol_type = scrapy.Field()
```

```py
# myspider1.py
import scrapy
from test1 import items

class Myspider1Spider(scrapy.Spider):
    name = 'myspider1'
    allowed_domains = ['www.kuaidaili.com']
    pg = 1
    start_urls = [f'https://www.kuaidaili.com/free/inha/{pg}']

    def parse(self, response):
        # yield data
        proxy_list = response.xpath('//tbody/tr')
        for p in proxy_list:
            MyItem = items.Test1Item()

            ip = p.xpath('./td[@data-title="IP"]/text()').extract()
            port = p.xpath('./td[@data-title="PORT"]/text()').extract()
            anonymous = p.xpath('./td[@data-title="匿名度"]/text()').extract()
            protocol_type = p.xpath('./td[@data-title="类型"]/text()').extract()

            MyItem['ip'] = ip[0]
            MyItem['port'] = port[0]
            MyItem['anonymous'] = anonymous[0]
            MyItem['protocol_type'] = protocol_type[0]

            yield MyItem
        # yield request
        if self.pg < 5:
            self.pg += 1
        yield scrapy.Request(f'https://www.kuaidaili.com/free/inha/{self.pg}', callback=self.parse)
```

then in Anaconda Prompt: `scrapy crawl myspider1 -o proxy.json`

## scrapy download image

example: scrapy downlod image
> then `scrapy crawl myspider1`  
> or `scrapy crawl myspider1 -o data.json`

```
C:.
│  scrapy.cfg
├─res
└─test1
    │  items.py
    │  middlewares.py
    │  pipelines.py
    │  settings.py
    │  __init__.py
    │
    └─spiders
            myspider1.py
            __init__.py
```

```py
# settings.py
BOT_NAME = 'test1'

SPIDER_MODULES = ['test1.spiders']
NEWSPIDER_MODULE = 'test1.spiders'

DEFAULT_REQUEST_HEADERS = {
  'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8',
  'User-Agent':"Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:71.0) Gecko/20100101 Firefox/71.0",
}

ITEM_PIPELINES = {
   'test1.pipelines.Test1Pipeline': 1,
}

# 这个必须写
IMAGES_STORE = 'res'
```

```py
# items.py
import scrapy

class Test1Item(scrapy.Item):
    ID = scrapy.Field()
    ImgLink = scrapy.Field()
```

```py
# myspider1.py
import scrapy
from test1 import items
import json

class Myspider1Spider(scrapy.Spider):
    name = 'myspider1'
    allowed_domains = ['www.douyu.com']
    pg = 1
    start_urls = [f'https://www.douyu.com/gapi/rknc/directory/yzRec/{pg}']

    def parse(self, response):
        data_list=json.loads(response.text)['data']['rl']
        for p in data_list:
            MyItem = items.Test1Item()

            MyItem['ID'] = p['rid']
            MyItem['ImgLink'] = p['rs1']

            yield MyItem

        if self.pg < 2:
            self.pg += 1
        yield scrapy.Request(f'https://www.douyu.com/gapi/rknc/directory/yzRec/{self.pg}', callback=self.parse)
```

```py
# pipelines.py
import scrapy
from scrapy.pipelines.images import ImagesPipeline
from scrapy.exceptions import DropItem
import os
from scrapy.utils.project import get_project_settings


class Test1Pipeline(ImagesPipeline):

    def get_media_requests(self, item, info):
        img_url = item['ImgLink']
        # 这个Requests可以加header
        yield scrapy.Request(img_url)

    def item_completed(self, results, item, info):
        # rename completed image
        # image_paths[0]是IMAGES_STORE/full/hashvalue.jpg
        image_paths = [x['path'] for ok, x in results if ok]
        if not image_paths:
            raise DropItem("Item contains no images")

        image_dir = get_project_settings().get('IMAGES_STORE')
        os.rename(f'{image_dir}/{image_paths[0]}', f'{image_dir}/{item["ID"]}.jpg')
        return item
```

example: 多层目录结构下载图片

```
C:.
│  scrapy.cfg
├─daili
│  │  items.py
│  │  middlewares.py
│  │  pipelines.py
│  │  settings.py
│  │  __init__.py
│  │
│  └─spiders
│          myspider.py
│          __init__.py
└─res
```

```py
# settings.py
BOT_NAME = 'daili'

SPIDER_MODULES = ['daili.spiders']
NEWSPIDER_MODULE = 'daili.spiders'

ROBOTSTXT_OBEY = False

DEFAULT_REQUEST_HEADERS = {
    'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8',
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:71.0) Gecko/20100101 Firefox/71.0',
}

ITEM_PIPELINES = {
   'daili.pipelines.DailiPipeline': 300,
}
IMAGES_STORE='res'

FEED_EXPORT_ENCODING='utf8'
```

```py
# items.py
import scrapy

class DailiItem(scrapy.Item):
    name=scrapy.Field()
    srcs=scrapy.Field()
```

```py
# spider.py
import scrapy
from daili import items
import re

pat1=re.compile(r'<h2><a href="thread-(\d+)-1-1.html"')
pat2=re.compile(r'<li><img alt=".+?" src="(.+?)"')

class MyspiderSpider(scrapy.Spider):
    name = 'myspider'
    allowed_domains = ['www.lsmpx.com']
    pg=1
    start_urls = [f'https://www.lsmpx.com/plugin.php?id=group&page={pg}']

    def parse(self, response):
        id_list=pat1.findall(response.text)
        for ID in id_list:
            for i in range(5):
                yield scrapy.Request(f'https://www.lsmpx.com/thread-{ID}-{i+1}-1.html', callback=self.myparse)
        
        if self.pg < 1:
            self.pg+=1
        yield scrapy.Request(f'https://www.lsmpx.com/plugin.php?id=group&page={self.pg}/', callback=self.parse)

    def myparse(self, response):
        MyItem=items.DailiItem()
        MyItem['name']=response.url[22:-7]
        MyItem['srcs']=pat2.findall(response.text)
        
        yield MyItem
```

```py
# pipelines.py
import scrapy
from scrapy.pipelines import images
from scrapy import exceptions


class DailiPipeline(images.ImagesPipeline):
    def get_media_requests(self, item, info):
        srcs = item['srcs']
        for i, src in enumerate(srcs):
            filename = f'{item["name"]}-{i}.jpg'
            yield scrapy.Request(src, headers={'Referer': 'https://www.lsmpx.com'}, meta={'filename': filename})

    def file_path(self, request, response=None, info=None):
        # rename file
        return request.meta['filename']
```

## scrapy download files

example: download files
> ImagePipelilne可能导致图片size变小，所以FilePipeling更加合适

```py
# settings.py
BOT_NAME = 'daili'

SPIDER_MODULES = ['daili.spiders']
NEWSPIDER_MODULE = 'daili.spiders'

ROBOTSTXT_OBEY = False

DEFAULT_REQUEST_HEADERS = {
    'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8',
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:71.0) Gecko/20100101 Firefox/71.0',
}

ITEM_PIPELINES = {
   'daili.pipelines.DailiPipeline': 300,
}
# 这个必须有
FILES_STORE='res'

FEED_EXPORT_ENCODING='utf8'
```

```py
# items.py
import scrapy

class DailiItem(scrapy.Item):
    name=scrapy.Field()
    srcs=scrapy.Field()
```

```py
# myspider.py
import scrapy
from daili import items
import re

pat1=re.compile(r'<h2><a href="thread-(\d+)-1-1.html"')
pat2=re.compile(r'<li><img alt=".+?" src="(.+?)"')

class MyspiderSpider(scrapy.Spider):
    name = 'myspider'
    allowed_domains = ['www.lsmpx.com']
    pg=1
    start_urls = [f'https://www.lsmpx.com/plugin.php?id=group&page={pg}']

    def parse(self, response):
        id_list=pat1.findall(response.text)
        for ID in id_list:
            for i in range(5):
                yield scrapy.Request(f'https://www.lsmpx.com/thread-{ID}-{i+1}-1.html', callback=self.myparse)
        
        if self.pg < 1:
            self.pg+=1
        yield scrapy.Request(f'https://www.lsmpx.com/plugin.php?id=group&page={self.pg}/', callback=self.parse)

    def myparse(self, response):
        MyItem=items.DailiItem()
        MyItem['name']=response.url[29:-7]
        MyItem['srcs']=pat2.findall(response.text)
        
        yield MyItem
```

```py
# pipelines.py
import scrapy
from scrapy.pipelines import files

class DailiPipeline(files.FilesPipeline):
    def get_media_requests(self, item, info):
        for i, src in enumerate(item['srcs']):
            filename = f'{item["name"]}-{i}.jpg'
            yield scrapy.Request(src, headers={'Referer': 'https://www.lsmpx.com'}, meta={'filename': filename})

    def file_path(self, request, response=None, info=None):
        # rename file
        return request.meta['filename']
```

## CrawlSpiders

in Anaconda Prompt
- `scrapy view https://www.taobao.com`下载pagesource并用浏览器打开下载的html

in Anaconda Prompt debug:
- `scrapy parse https://www.baidu.com -c parse_func`: 直接调试函数
- `scrapy parse https://www.baidu.com -c parse_func --pipelines`: 带pipeline的调试

in Anaconda Prompt
- `scrapy shell url`
then in IPython3

```py
from scrapy.linkextractors import LinkExtractor

le1=LinkExtractor(allow=(r'id=group&page=\d+'))
# 提取link
le1.extract_links(response)
```

in Anaconda Prompt
- `scrapy startproject hello`
- `scrapy genspider -t crawl myspider www.lsmpx.com`
- `scrapy crawl myspider -o data.json`

```
C:.
│  scrapy.cfg
└─hello
    │  items.py
    │  middlewares.py
    │  pipelines.py
    │  settings.py
    │  __init__.py
    │
    └─spiders
            myspider.py
            __init__.py
```

```py
# settings.py
BOT_NAME = 'hello'

SPIDER_MODULES = ['hello.spiders']
NEWSPIDER_MODULE = 'hello.spiders'

ROBOTSTXT_OBEY = False

DEFAULT_REQUEST_HEADERS = {
    'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8',
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:71.0) Gecko/20100101 Firefox/71.0',
}

FEED_EXPORT_ENCODING='utf8'
```

```py
# items.py
import scrapy

class HelloItem(scrapy.Item):
    cover_id = scrapy.Field()
    src = scrapy.Field()
```

```py
# myspider.py
import scrapy
from scrapy.linkextractors import LinkExtractor
from scrapy.spiders import CrawlSpider, Rule
from hello import items

class MyspiderSpider(CrawlSpider):
    name = 'myspider'
    allowed_domains = ['www.lsmpx.com']
    pg=1
    start_urls = [f'https://www.lsmpx.com/plugin.php?id=group&page={pg}']

    rules = (
        # 提取link, 并配置规则
        # 匹配url并follow进入
        Rule(LinkExtractor(allow=r'id=group&page=\d+'), callback='parse_item', follow=True),
    )

    def parse_item(self, response):
        data_list=response.xpath('//div[@class="photo"]')
        for data in data_list:
            MyItem=items.HelloItem()
            
            MyItem['cover_id']=data.xpath('./a/@href').extract()[0]
            MyItem['src']=data.xpath('.//img/@src').extract()[0]

            yield MyItem
```

example: 层级结构CrawlSpiders
> `scrapy crawl myspider -o data.json`

```py
# items.py
import scrapy

class HelloItem(scrapy.Item):
    name = scrapy.Field()
    srcs = scrapy.Field()
```

```py
# myspider.py
import scrapy
from scrapy.linkextractors import LinkExtractor
from scrapy.spiders import CrawlSpider, Rule
from hello import items
import re

pat=re.compile(r'<li><img alt=".+?" src="(.+?)"')

class MyspiderSpider(CrawlSpider):
    name = 'myspider'
    allowed_domains = ['www.lsmpx.com']
    start_urls = ['https://www.lsmpx.com/plugin.php?id=group&page=1']

    rules = (
        # 第一个rule False, 第二个rule True; 表示只爬https://www.lsmpx.com/plugin.php?id=group&page=1
        Rule(LinkExtractor(allow=r'id=group&page=\d+'), follow=False),
        Rule(LinkExtractor(allow=r'thread-\d+-\d-1.html'), callback='parse_item', follow=True),
        
    )

    def parse_item(self, response):
        MyItem=items.HelloItem()
        MyItem['name']=response.url[29:-7]
        MyItem['srcs']=pat.findall(response.text)

        yield MyItem
```

example: field trick
> 其实也快不到哪里去

```py
def parse_item(self, response):
    name=response.url[29:-7]
    srcs=pat.findall(response.text)
    MyItem=items.HelloItem()
    for field in MyItem.fields:
        try:
            MyItem[field]=eval(field)
        except:
            self.logger.debug('Field is Not Defined', field)
    yield MyItem
```

about LOG

```py
# settings.py
LOG_FILE='myproject.log'

# CRITICAL > ERROR > WARNING > INFO > DEBUG
LOG_LEVEL='CRITICAL' # >=CRITICAL
LOG_LEVEL='ERROR' # >=ERROR
LOG_LEVEL='WARNING' # 显示>=WARNING
LOG_LEVEL='INFO' # 显示>=INFO
LOG_LEVEL='DEBUG' # 显示>=DEBUG
```

example: loger

```py
import scrapy


class HttpbinSpider(scrapy.Spider):
    name = 'httpbin'
    allowed_domains = ['httpbin.org']
    start_urls = ['http://httpbin.org/get']

    def parse_post(self, response):
        self.logger.debug('This is DEBUG')
        print('*'*50)
        print(response.status)
```

example: process link
> 反爬虫: html的link是错误的，但通过js使得动态的url是正确的，所以需要先对url进行处理

```py
# myspider.py
import scrapy
from scrapy.linkextractors import LinkExtractor
from scrapy.spiders import CrawlSpider, Rule
from hello import items
import re

class MyspiderSpider(CrawlSpider):
    name = 'myspider'
    allowed_domains = ['wz.sun0769.com']
    start_urls = ['http://wz.sun0769.com/index.php/question/questionType?type=4&page=0']

    rules = (
        Rule(LinkExtractor(allow=r'type=4'), follow=True, process_links='deal_links'),
        Rule(LinkExtractor(allow=r'html/question/\d+/\d+'), callback='parse_item', follow=False),
    )

    def parse_item(self, response):
        MyItem=items.HelloItem()
        MyItem['url']=response.url
        MyItem['name']=response.xpath('//td/span[2]//text()').extract()[0]

        yield MyItem
    
    def deal_links(self, links):
        print('*'*50)
        # 对link进行修正
        for link in links:
            print(link)
            link.url=link.url.replace('?', '&').replace('Type&', 'Type?')
        print('*'*50)

        return links
```

about 反反爬虫策略
- User-Agent
- `COOKIES_ENABLED = False`
- `DOWNLOAD_DELAY = 1`
- Proxy
- Baidu cache, Google cache

example: scrapy with database
> [connect some database](https://zhuanlan.zhihu.com/p/33816647)

```sql
-- create table in sqlite3
CREATE TABLE "info" (
  "cover_id" TEXT NOT NULL,
  "url" TEXT NOT NULL
);
```

```
C:.
│  data.db
│  myproject.log
│  scrapy.cfg
│
└─hello
    │  items.py
    │  middlewares.py
    │  pipelines.py
    │  settings.py
    │  __init__.py
    │
    └─spiders
            myspider2.py
            __init__.py
```

```py
# settings.py
BOT_NAME = 'hello'

SPIDER_MODULES = ['hello.spiders']
NEWSPIDER_MODULE = 'hello.spiders'

ROBOTSTXT_OBEY = False

DEFAULT_REQUEST_HEADERS = {
    'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8',
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:71.0) Gecko/20100101 Firefox/71.0',
}

ITEM_PIPELINES = {
   'hello.pipelines.HelloPipeline': 300,
}

FEED_EXPORT_ENCODING='utf8'
```

```py
# items.py
import scrapy

class TestItem(scrapy.Item):
    cover_id = scrapy.Field()
    url = scrapy.Field()
```

```py
# myspider.py
import scrapy
import re
from hello import items

pat=re.compile(r'<h2><a href="thread-(\d+)-1-1.html"')

class Myspider2Spider(scrapy.Spider):
    name = 'myspider2'
    allowed_domains = ['www.lsmpx.com']
    pg=1
    start_urls = [f'https://www.lsmpx.com/plugin.php?id=group&page={pg}']

    def parse(self, response):
        id_list=pat.findall(response.text)
        for ID in id_list:
            MyItem=items.TestItem()
            MyItem['cover_id']=ID
            MyItem['url']=f'thread-{ID}-1-1.html'
            yield MyItem
        
        if self.pg<5:
            self.pg+=1
        
        yield scrapy.Request(f'https://www.lsmpx.com/plugin.php?id=group&page={self.pg}', callback=self.parse)
```

```py
# pipelines.py
import sqlite3
from scrapy import spiders

class HelloPipeline(object):
    def open_spider(self, spider):
        self.db_conn = sqlite3.connect('data.db')
        self.db_cur = self.db_conn.cursor()

    def close_spider(self, spider):
        self.db_conn.close()

    def process_item(self, item, spider):
        self.insert_db(item)
        self.db_conn.commit()
        return item

    def insert_db(self, item):
        sql='INSERT INTO info VALUES(?,?)'
        values=(item['cover_id'], item['url'])
        self.db_cur.execute(sql, values)
```

example: get settings

```py
# pipelines.py

class MongoPipeline(object):
    def __init__(self, mongo_uri, mongo_db):
        self.mongo_uri=mongo_uri
        self.mongo_db=mongo_db
    
    @classmethod
    def from_crawler(cls, crawler):
        return cls(
            mongo_uri=crawler.settings.get('MONGO_URI'),
            mongo_db=crawler.settings.get('MONGO_DB')
        )
    
    def open_spider(self, spider):
        self.client=pymongo.MongoClient(self.mongo_uri)
        self.db=self.client[self.mongo_db]
    
    def process_item(self, item, spider):
        name=item.__class__.__name__
        self.db[name].insert(dict(item))
        return item
    
    def close_spider(self, spider):
        self.client.close()
```

```py
# spider1.py
import scrapy
from mytest import items

class Spider1Spider(scrapy.Spider):
    name = 'spider1'
    allowed_domains = ['www.81mbmb.com']
    pg = 1
    start_urls = [f'https://www.81mbmb.com/lunli/']

    # 自定义settings
    custom_settings = {
        'DEFAULT_REQUEST_HEADERS': {
            'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:72.0) Gecko/20100101 Firefox/72.0',
            'Accept-Language': 'en',
        },
    }

    # 先通过from_crawler这个classmethod生成带变量的class, 这个class __init__的时候自动获取了参数并交给self
    def __init__(self, var, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.var = var
        print(self.var)

    @classmethod
    def from_crawler(cls, crawler):
        return cls(
            var=crawler.settings.get('FEED_EXPORT_ENCODING', 'utf8')
        )

    def parse(self, response):
        names = response.xpath('//h5/a/@title').extract()
        urls = response.xpath('//h5/a/@href').extract()

        for name, url in zip(names, urls):
            MyItem = items.MytestItem()
            MyItem['name'] = name
            MyItem['url'] = f'https://www.81mbmb.com{url}'
            yield MyItem

        if self.pg < 231:
            self.pg += 1

        yield scrapy.Request(f'https://www.81mbmb.com/lunli/index-{self.pg}.html', callback=self.parse)
```

example: 如果start_urls必须post, 那么`start_requests`必须重写

```py
# spider1.py
import scrapy

class HttpbinSpider(scrapy.Spider):
    name = 'httpbin'
    allowed_domains = ['httpbin.org']

    def start_requests(self):
        yield scrapy.Request('http://httpbin.org/post', callback=self.parse_post, method='POST')

    def parse_post(self, response):
        print('*'*50)
        print(response.status)
        print('*'*50)
```

## download middleware

在Engine与Download之间

```py
# settings.py
BOT_NAME = 'mytest'

SPIDER_MODULES = ['mytest.spiders']
NEWSPIDER_MODULE = 'mytest.spiders'

ROBOTSTXT_OBEY = False

DOWNLOAD_DELAY = 0.25

COOKIES_ENABLED = False

DEFAULT_REQUEST_HEADERS = {
    'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8',
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:71.0) Gecko/20100101 Firefox/71.0',
}

DOWNLOADER_MIDDLEWARES = {
   'mytest.middlewares.RandomProxy': 543,
   'mytest.middlewares.RandomAgent': 544,
}
```

```py
# items.py
import scrapy

class MytestItem(scrapy.Item):
    url = scrapy.Field()
    title = scrapy.Field()
```

```py
# myspider.py
import scrapy
import random
from mytest import items


class MyspiderSpider(scrapy.Spider):
    name = 'myspider'
    allowed_domains = ['wz.sun0769.com']
    pg = 0
    start_urls = [f'http://wz.sun0769.com/index.php/question/questionType?type=4&page={pg}']

    def parse(self, response):
        data_list=response.xpath('//td/a[@class="news14"]')
        for data in data_list:
            MyItem = items.MytestItem()
            MyItem['url'] = data.xpath('./@href').extract()[0]
            MyItem['title'] = data.xpath('./@title').extract()[0]
            yield MyItem

        if self.pg < 90:
            self.pg += 30

        yield scrapy.Request(f'http://wz.sun0769.com/index.php/question/questionType?type=4&page={self.pg}', callback=self.parse)
```

```py
# middlewares.py
import random

UA_LIST = [
    'Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:71.0) Gecko/20100101 Firefox/71.0',
    'Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:70.0) Gecko/20100101 Firefox/70.0',
    'Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:69.0) Gecko/20100101 Firefox/69.0',
    'Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:68.0) Gecko/20100101 Firefox/68.0',
    'Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:67.0) Gecko/20100101 Firefox/67.0',
    'Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:66.0) Gecko/20100101 Firefox/66.0',
]

PROXY_LIST = [
    'HTTP://113.195.168.87:9999', 
    'HTTP://1.162.123.164:80', 
    'HTTP://75.151.213.85:8080', 
    'HTTP://187.95.125.71:3128', 
    'HTTP://91.121.162.173:80', 
    'HTTP://185.148.218.246:8081', 
    'HTTP://49.89.84.83:9999', 
    'HTTP://123.163.96.122:9999', 
    'HTTP://187.53.60.82:8080', 
    'HTTP://187.95.125.71:3128', 
    'HTTP://91.121.162.173:80', 
    'HTTP://114.226.162.7:9999', 
    'HTTP://187.95.125.71:3128', 
    'HTTP://91.121.162.173:80', 
    'HTTP://185.148.218.246:8081', 
]


class RandomAgent(object):
    def process_request(self, request, spider):
        ua = random.choice(UA_LIST)
        request.headers.setdefault('User-Agent', ua)


class RandomProxy(object):
    def process_request(self, request, spider):
        proxy= random.choice(PROXY_LIST)
        request.meta['proxy']=proxy
```

example: `process_response`, 修改status code

```py
# spider2.py
import scrapy

class HttpbinSpider(scrapy.Spider):
    name = 'httpbin'
    allowed_domains = ['httpbin.org']
    start_urls = ['http://httpbin.org/get']

    def parse(self, response):
        self.logger.debug('This is DEBUG')
        print('*'*50)
        print(response.status)
```

```py
# middlewares.py
class MytestDownloaderMiddleware(object):
    def process_response(self, request, response, spider):
        response.status=201
        return response
```

example: `process_exception`
> 第一次访问exception之后调用代理访问

查看settings: `scrapy settings --get=DOWNLOADER_MIDDLEWARES_BASE`

```py
# settings.py
DOWNLOADER_MIDDLEWARES = {
   'mytest.middlewares.MytestDownloaderMiddleware': 543,
    # 禁用Retry只是为了方便演示
   'scrapy.downloadermiddlewares.retry.RetryMiddleware':None
}
```

```py
# spider3.py
import scrapy

class GoogleSpider(scrapy.Spider):
    name = 'google'
    allowed_domains = ['www.google.com']

    def start_requests(self):
        self.logger.debug('Try First Time')
        # dont_filter不参与去重
        yield scrapy.Request('https://www.google.com/', callback=self.parse, meta={'download_timeout':5}, dont_filter=True)

    def parse(self, response):
        print(response.text)
```

```py
# middlewares.py
class MytestDownloaderMiddleware(object):
    logger=logging.getLogger(__name__)
    def process_exception(self, request, exception, spider):
        self.logger.debug('Get Exception')
        self.logger.debug('Try Second Time')
        request.meta['proxy']='https://127.0.0.1:9743'
        return request
```

example: 302反爬虫

```py
import requests

headers={'User-Agent':'Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:71.0) Gecko/20100101 Firefox/71.0'}

r1=requests.get('url', headers=headers, allow_redirects=False)

real_url=r1.headers['Location']
r2.requests.get(real_url, headers=headers)
```

example: 通过用weibo.cn查询股票代码的信息获取舆情，进而决策
> 缺点：噪音太大

## Other Examples

各种[spider](https://docs.scrapy.org/en/latest/topics/spiders.html):
- Spider
- CrawlSpider
- XMLFeedSpider
- CSVFeedSpider

example: xmlspider
> 因为response.xpath()就会将response.text变成tree, 适用于html, 同样适用于xml

```py
# spiders.py/xmlspider.py
import scrapy

# 或者继承scrapy.spiders.XMLFeedSpider
class XmlspiderSpider(scrapy.Spider):
    name = 'xmlspider'
    allowed_domains = ['blog.sina.com.cn']
    start_urls = ['http://blog.sina.com.cn/rss/liuyuhaoxy.xml']

    def parse(self, response):
        for i in response.xpath('//item'):
            title=i.xpath('./title/text()').extract_first()
            link=i.xpath('./link/text()').extract_first()
            print(title, link)
```

examle: csvspider

```py
import scrapy

class XmlspiderSpider(scrapy.spiders.CSVFeedSpider):
    name = 'csvspider'
    allowed_domains = ['quotes.money.163.com']
    start_urls = ['http://quotes.money.163.com/service/chddata.html?code=1300133&end=20180202&fields=TCLOSE;HIGH;LOW;TOPEN;LCLOSE;CHG;PCHG;TURNOVER;VOTURNOVER;VATURNOVER;TCAP;MCAP']
    headers=['日期','股票代码','名称','收盘价','最高价','最低价','开盘价','前收盘','涨跌额','涨跌幅','换手率','成交量','成交金额','总市值','流通市值']
    delimiter=','


    def parse_row(self, response, row):
        print(row['股票代码'].encode('cp1252').decode('gbk'), row['名称'].encode('cp1252').decode('gbk'))
```

tip: scrapy模拟登录对于同一个链接先get, 然后post; 那么`Request(dont_filter=True)`或者post直接使用`FormRequest()`

example: 利用`FormRequest`模拟登录
> scrapy能够模拟js加密: fiddler监控的post的数据与下面`formdata=`数据不同, 所以scrapy进行了本地加密处理；对于复杂的网站加密，scrapy仍然无法解决  
> 更加通用的方式是selenium获取cookie得到cookie pool, 通过api提供给程序，程序中间件在headers中加cookie访问

```py
# spiders/renren.py
import scrapy

class RenrenSpider(scrapy.Spider):
    name = 'renren'
    allowed_domains = ['www.renren.com']
    # 只有这个接口可以成功
    start_urls = ['http://www.renren.com/PLogin.do']
    # # 如下登录接口无法成功
    # start_urls = ['http://www.renren.com/SysHome.do']

    def parse(self, response):
        yield scrapy.FormRequest.from_response(response, formdata={'email':'xxxxxx@163.com', 'password':'xxxxxx'}, callback=self.parse_page)

    def parse_page(self, response):
        yield scrapy.Request('http://www.renren.com/973121717/profile', callback=self.parse_profile)
    
    def parse_profile(self, response):
        if response.text.find('boot'):
            print('login success')
```

```py
# spiders.py/renren.py
import scrapy

class RenrenSpider(scrapy.Spider):
    name = 'renren'
    allowed_domains = ['www.renren.com']
    start_urls = ['http://www.renren.com/973121717/profile']

    def parse(self, response):
        cookie = {
            "anonymid": "k57rnfzf-9vf59c",
            " depovince": "GW",
            " _r01_": "1",
            " JSESSIONID": "abcNq7kRGbepk8EA3fo_w",
            " ick_login": "eb8213a2-ac40-4411-805d-a55b61080ce0",
            " taihe_bi_sdk_uid": "6e883fbe12ac02761a24aeda65a31293",
            " taihe_bi_sdk_session": "680271b4f995e38a262f87af3fc6312a",
            " id": "973121717",
            " ver": "7.0",
            " ick": "755e68b8-d22b-4c75-a749-8a7877a245bd",
            " jebecookies": "b3672c21-9827-4e54-83ea-bf031e59b1a7|||||",
            " _de": "5F410E5B6D93032B5E28A9DF15E724D26DEBB8C2103DE356",
            " p": "7d4a8a81fa610c33e11b23b7cebc416b7",
            " first_login_flag": "1",
            " ln_uact": "pku_gewei@163.com",
            " ln_hurl": "http://hdn.xnimg.cn/photos/hdn521/20191219/2100/h_main_3rDB_c69400003484195a.jpg",
            " t": "d024bf313ee762be3711f5cf6387d7f67",
            " societyguester": "d024bf313ee762be3711f5cf6387d7f67",
            " xnsid": "7192a1d8",
            " loginfrom": "null",
            "wp_fold": "0"
        }
        yield scrapy.Request('http://www.renren.com/973121717/profile', cookies=cookie, callback=self.parse_profile)

    def parse_profile(self, response):
        if response.text.find('boot'):
            print('login success')
```

example: scrapy with selenium in spider

```py
from selenium import webdriver
import scrapy


class CtoSpider(scrapy.Spider):
    name = 'cto'
    allowed_domains = ['51cto.com']
    start_urls = ['https://home.51cto.com/mobile/client-login/']

    def parse(self, response):
        my_cookie = self.my_cookie(response.url)
        yield scrapy.Request('https://home.51cto.com/space?uid=9472075', callback=self.parse_page, cookies=my_cookie)

    def my_cookie(self, login_url):
        browser = webdriver.Firefox(firefox_binary=r"D:\Browser\Firefox\firefox.exe",
                                    executable_path=r"D:\Browser\BrowserDriver\geckodriver.exe")
        browser.get(login_url)
        user = browser.find_element_by_id('loginform-username')
        user.send_keys('xxxxxx@163.com')
        pwd = browser.find_element_by_id('loginform-password')
        pwd.send_keys('xxxxxx')
        btn_submit = browser.find_element_by_id('bind_old_login')
        btn_submit.click()

        cookie_dict = browser.get_cookies()
        browser.close()
        return cookie_dict

    def parse_page(self, response):
        if response.text.find('RacherSasuke') != -1:
            print('login success', '*'*20)
        else:
            print('login fail', '*'*20)
```

example: scarpy with selenium in middleware

```py
# spiders/cto.py
import scrapy

class CtoSpider(scrapy.Spider):
    name = 'cto'
    allowed_domains = ['51cto.com']
    start_urls = ['https://home.51cto.com/mobile/client-login/']

    cookies = None

    def parse(self, response):
        if response.text == 'success':
            yield scrapy.Request('https://home.51cto.com/space?uid=9472075',
                                 callback=self.parse_page,
                                 cookies=self.cookies)
        else:
            print('getting cookie fails')

    def parse_page(self, response):
        if response.text.find('RacherSasuke') != -1:
            print('login success', '*'*20)
        else:
            print('login fail', '*'*20)
```

```py
# middlewares.py
from selenium import webdriver
from scrapy import http

class Test1DownloaderMiddleware(object):
    def process_request(self, request, spider):
        if spider.name == 'cto':
            if request.url.find('client-login') != -1:
                browser = webdriver.Firefox(firefox_binary=r"D:\Browser\Firefox\firefox.exe",
                                            executable_path=r"D:\Browser\BrowserDriver\geckodriver.exe")
                browser.get(request.url)
                user = browser.find_element_by_id('loginform-username')
                user.send_keys('xxxxxx@163.com')
                pwd = browser.find_element_by_id('loginform-password')
                pwd.send_keys('xxxxxx')
                btn_submit = browser.find_element_by_id('bind_old_login')
                btn_submit.click()

                spider.cookies = browser.get_cookies()
                browser.close()
                return http.HtmlResponse(url=request.url, body=b'success')
```

example: save and load cookies
> 可以买多个账户密码，然后登陆，添加进入database, 形成cookie pool

```py
from selenium import webdriver
import pickle

browser = webdriver.Firefox(firefox_binary=r"D:\Browser\Firefox\firefox.exe",
                            executable_path=r"D:\Browser\BrowserDriver\geckodriver.exe")
browser.get('https://home.51cto.com/mobile/client-login/')

user = browser.find_element_by_id('loginform-username')
user.send_keys('xxxxxx@163.com')
pwd = browser.find_element_by_id('loginform-password')
pwd.send_keys('xxxxxx')
btn_submit = browser.find_element_by_id('bind_old_login')
btn_submit.click()

cookies=browser.get_cookies()

with open('cookies.dat', 'wb') as file:
    pickle.dump(cookies, file)

# browser.close()
```

```py
from selenium import webdriver
import pickle

with open('cookies.dat', 'rb') as file:
    cookies=pickle.load(file)

browser = webdriver.Firefox(firefox_binary=r"D:\Browser\Firefox\firefox.exe",
                            executable_path=r"D:\Browser\BrowserDriver\geckodriver.exe")
browser.get('https://home.51cto.com')

for cookie in cookies:
    # print(cookie['name'],cookie['value'])
    browser.add_cookie(cookie)

browser.get('https://home.51cto.com/space?uid=9472075')
if browser.page_source.find('RacherSasuke')!=-1:
    print('cookie is good')
else:
    print('cookie is bad')

browser.close()
```

exmaple: selenium cookies for requests

```py
import requests
import pickle

s=requests.session()
for cookie in pickle.load(open('cookies.dat', 'rb')):
    s.cookies.set(cookie['name'], cookie['value'])

r=s.get('https://home.51cto.com/space?uid=9472075')
if r.text.find('RacherSasuke')!=-1:
    print('cookie is good')
else:
    print('cookie is bad')
```

example: scrape proxies with mongodb

```bash
C:.
│  scrapy.cfg
│
└─hello
    │  items.py
    │  middlewares.py
    │  pipelines.py
    │  settings.py
    │
    └─spiders
            xici.py
```

```py
# settings.py
BOT_NAME = 'hello'

SPIDER_MODULES = ['hello.spiders']
NEWSPIDER_MODULE = 'hello.spiders'

ROBOTSTXT_OBEY = False

DEFAULT_REQUEST_HEADERS = {
    'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8',
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:71.0) Gecko/20100101 Firefox/71.0',
}

ITEM_PIPELINES = {
   'hello.pipelines.HelloPipeline': 300,
}
MONGO_URI='mongodb://grey:xxxxxx@xx.xx.xx.xx'
MONGO_DB='xici'
FEED_EXPORT_ENCODING='utf8'
```

```py
# pipelines.py
import pymongo

class HelloPipeline(object):
    def __init__(self, mongo_uri, mongo_db):
        self.mongo_uri=mongo_uri
        self.mongo_db=mongo_db
    
    @classmethod
    def from_crawler(cls, crawler):
        return cls(
            mongo_uri=crawler.settings.get('MONGO_URI'),
            mongo_db=crawler.settings.get('MONGO_DB')
        )
    
    def open_spider(self, spider):
        self.client=pymongo.MongoClient(self.mongo_uri)
        self.db=self.client[self.mongo_db]
    
    def process_item(self, item, spider):
        self.db['proxies'].update(item, {'$set':item}, upsert=True)
        return item
    
    def close_spider(self, spider):
        self.client.close()
```

```py
# items.py
import scrapy

class HelloItem(scrapy.Item):
    proxy=scrapy.Field()
```

```py
# xici.py
import scrapy
import requests


class XiciSpider(scrapy.Spider):
    name = 'xici'
    allowed_domains = ['www.xicidaili.com']
    start_urls = ['https://www.xicidaili.com/nn/1']
    pg = 1

    def parse(self, response):
        protocol_list = response.xpath('//tr/td[6]//text()').extract()
        ip_list = response.xpath('//tr/td[2]//text()').extract()
        port_list = response.xpath('//tr/td[3]//text()').extract()

        for data in zip(protocol_list, ip_list, port_list):
            proxy = {f'{data[0]}': f'{data[0]}://{data[1]}:{data[2]}'}
            if self.check_proxy(proxy):
                yield {'proxy': proxy}

        if self.pg < 2:
            self.pg += 1

        yield scrapy.Request(f'https://www.xicidaili.com/nn/{self.pg}', callback=self.parse)

    def check_proxy(self, proxy):
        try:
            requests.get('https://www.baidu.com',
                        proxies=proxy,
                        timeout=2,
                        headers={
                            'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:72.0) Gecko/20100101 Firefox/72.0'})
            return True
        except:
            return False
```

example: sitemap example
> `scrapy crawl apple -o data.json`

```py
# apple.py
import scrapy

class AppleSpider(scrapy.spiders.SitemapSpider):
    name = 'apple'
    allowed_domains = ['www.apple.com']
    # spider会请求sitemap中的所有urls
    sitemap_urls =  ['https://www.apple.com/sitemap.xml']


    def parse(self, response):
        yield {
            # 获取新Url的url和title
            'url':response.url,
            'title':response.xpath('//head/title/text()').extract_first()
        }
```

example: use `meta`

```bash
.
│  scrapy.cfg
└─lesmao
    │  items.py
    │  middlewares.py
    │  pipelines.py
    │  settings.py
    │
    └─spiders
            lsm.py
```

```py
# settings.py
BOT_NAME = 'lesmao'

SPIDER_MODULES = ['lesmao.spiders']
NEWSPIDER_MODULE = 'lesmao.spiders'

DEFAULT_REQUEST_HEADERS = {
    'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8',
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:71.0) Gecko/20100101 Firefox/71.0',
}

FEED_EXPORT_ENCODING='utf8'
```

```py
# items.py
import scrapy

class LesmaoItem(scrapy.Item):
    pass
```

```py
# lsm.py
import scrapy

class LsmSpider(scrapy.Spider):
    name = 'lsm'
    allowed_domains = ['www.lesmao.co']
    start_urls = ['https://www.lesmao.co/plugin.php?id=group&page=1']
    pg=1

    def parse(self, response):
        urls=response.xpath('//h2/a/@href').extract()
        titles=response.xpath('//h2/a/text()').extract()
        for short_url, title in zip(urls, titles):
            url=f'https://www.lesmao.co/{short_url}'
            item={'url':url, 'title':title}
            yield scrapy.Request(url,meta={"item":item}, callback=self.parse_page)
        
        if self.pg<5:
            self.pg+=1
        yield scrapy.Request(f'https://www.lesmao.co/plugin.php?id=group&page={self.pg}', callback=self.parse)

    def parse_page(self, response):
        item=response.meta['item']
        date=response.xpath('//div[@id="thread-title"]/em/text()').extract_first()
        item['date']=date
        yield item
```

example: scrapy log

```py
# lsm.py
import scrapy
from scrapy import log

class LsmSpider(scrapy.Spider):
    name = 'lsm'
    allowed_domains = ['www.lesmao.co']
    start_urls = ['https://www.lesmao.co/plugin.php?id=group&page=1']
    pg=1

    def parse(self, response):
        # log由lsm spider发出
        self.logger.warning('------------This is warning----------------')
        # log由scrapy.log发出
        log.logger.warning('------------This is warning----------------')
```

example: stats collection

```py
import scrapy
from scrapy import log

class LsmSpider(scrapy.Spider):
    name = 'lsm'
    allowed_domains = ['www.lesmao.co']
    start_urls = ['https://www.lesmao.co/plugin.php?id=group&page=1']
    pg=1

    def parse(self, response):
        self.crawler.stats.inc_value('crawl_count')
        self.crawler.stats.set_value('url', response.url)
        print('-'*30, self.crawler.stats.get_value('crawl_count'))
        print('-'*30, self.crawler.stats.get_value('url'))
```

example: signal dispatch

```py
# lsm.py
# method1
import scrapy
from scrapy import signals
from scrapy.xlib.pydispatch import dispatcher

class LsmSpider(scrapy.Spider):
    name = 'lsm'
    allowed_domains = ['www.lesmao.co']
    start_urls = ['https://www.lesmao.co/plugin.php?id=group&page=1']
    pg=1

    def __init__(self):
        dispatcher.connect(self.myclose, signal=signals.spider_closed)

    def parse(self, response):
        pass
    
    def myclose(self):
        print('-'*30,'spider is closed')
```

```py
# lsm.py
# method2
import scrapy
from scrapy import signals

class LsmSpider(scrapy.Spider):
    name = 'lsm'
    allowed_domains = ['www.lesmao.co']
    start_urls = ['https://www.lesmao.co/plugin.php?id=group&page=1']
    pg=1

    @classmethod
    def from_crawler(cls, crawler, *args, **kwargs):
        spider = super(LsmSpider, cls).from_crawler(crawler, *args, **kwargs)
        crawler.signals.connect(spider.myclose, signal=signals.spider_closed)
        return spider

    def parse(self, response):
        pass
    
    def myclose(self, **kwargs):
        print('-'*30,'spider is closed')
        print(kwargs)
```

example: scrapy send mail

```py
# ke.py
import scrapy
from scrapy.xlib.pydispatch import dispatcher
from scrapy import signals
from scrapy import mail


class KeSpider(scrapy.Spider):
    name = 'ke'
    allowed_domains = ['www.kepu.cn']
    start_urls = ['http://www.kepu.cn/zt/']
    pg = 1

    def __init__(self):
        dispatcher.connect(self.send_mail, signal=signals.spider_closed)
        self.mailer = mail.MailSender(
            smtphost='mail.pku.edu.cn',
            mailfrom='xxxxxx@pku.edu.cn',
            smtpuser='xxxxxx@pku.edu.cn',
            smtppass='xxxxxx',
            smtpport=465,
            smtpssl=True)

    def parse(self, response):
        titles = response.xpath('//div[@class="cont"]/h5/a/text()').extract()
        urls = response.xpath('//div[@class="cont"]/h5/a/@href').extract()
        for title, url in zip(titles, urls):
            yield {'title': title, 'url': url}
        if self.pg < 10:
            self.pg += 1
        yield scrapy.Request(f'http://www.kepu.cn/zt/index_{self.pg}.html', callback=self.parse)

    def send_mail(self):
        # 必须要return, 否则报错
        return self.mailer.send(to=["vip.gewei@foxmail.com"], subject="SpiderReport", body="The spider is closed", cc=["pku_gewei@163.com"])
```

example: scrpay with `__setitem__`

```
.
|-- coolapk
|   |-- items.py
|   |-- settings.py
|   `-- spiders
|       `-- cool.py
`-- scrapy.cfg
```

```py
# settings.py
BOT_NAME = 'coolapk'

SPIDER_MODULES = ['coolapk.spiders']
NEWSPIDER_MODULE = 'coolapk.spiders'

ROBOTSTXT_OBEY = False

DEFAULT_REQUEST_HEADERS = {
    'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8',
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:71.0) Gecko/20100101 Firefox/71.0',
}

FEED_EXPORT_ENCODING='utf8'
```

```py
# items.py
import scrapy

class CoolapkItem(scrapy.Item):
    def __setitem__(self, key, value):
        if key not in self.fields:
            self.fields[key] = scrapy.Field()
        self._values[key] = value
```

```py
# cool.py
import scrapy
from coolapk import items

class CoolSpider(scrapy.Spider):
    name = 'cool'
    allowed_domains = ['www.coolapk.com']
    start_urls = [f'https://www.coolapk.com/game?p={i+1}' for i in range(10)]

    def parse(self, response):
        myItem = items.CoolapkItem()
        names = response.xpath("//p[@class='list_app_title']/text()").extract()
        for name in names:
            myItem['game_name'] = name
            yield myItem
```

example: scrapy with `ItemLoader`
> 方便大型项目的管理

```py
# items.py
import scrapy

class CoolapkItem(scrapy.Item):
    games = scrapy.Field()
    current_time = scrapy.Field()
```

```py
# cool.py
import scrapy
from coolapk import items
from scrapy import loader
import time

class CoolSpider(scrapy.Spider):
    name = 'cool'
    allowed_domains = ['www.coolapk.com']
    start_urls = [f'https://www.coolapk.com/game?p={i+1}' for i in range(10)]

    def parse(self, response):
        myLoader=loader.ItemLoader(item=items.CoolapkItem(), response=response)
        myLoader.add_xpath(field_name='games', xpath="//p[@class='list_app_title']/text()")
        myLoader.add_value(field_name='current_time', value=time.asctime())
        yield myLoader.load_item()
```