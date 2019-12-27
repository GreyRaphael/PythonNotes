# Scrapy Framework

- [Scrapy Framework](#scrapy-framework)
  - [Introduction](#introduction)
  - [scrapy download image](#scrapy-download-image)
  - [scrapy download files](#scrapy-download-files)
  - [CrawlSpiders](#crawlspiders)

## Introduction

![](res/scrapy01.png)
> 1. Spiders将很多请求交给Scheduler
> 1. Scheduler将请求交给Downloader从网站下载数据; (proxy作为Scheduler和Downloader之间的中间件)
> 1. Downloader下载的数据交给Spiders如果是Requests，那么重复1-3的过程；如果是数据，将数据交给ItermPipeline

in Anaconda prompt: 
- `conda install scrapy`
- `scrapy startproject test1`
- `cd test1`
- `scrapy genspider myspider1 m.huxiu.com`

```
C:\USERS\GREY\TEST1
│  scrapy.cfg
│
└─test1
    │  items.py
    │  middlewares.py
    │  pipelines.py
    │  settings.py
    │  __init__.py
    │
    ├─spiders
    │  │  myspider1.py
    │  │  __init__.py
    │  │
    │  └─__pycache__
    │          __init__.cpython-37.pyc
    │
    └─__pycache__
            settings.cpython-37.pyc
            __init__.cpython-37.pyc
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

in Anaconda Prompt:
- `scrapy crawl myspider1`

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
- `scrapy crawl myspider1 -o huxiu.csv`

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

then in Anaconda Prompt:
- `scrapy crawl myspider1`

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

scrapy with redis与scrapy的不同
- 存item数据
- 存request
- 存request的指纹(hash value)

## CrawlSpiders

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

about LOG

```py
# settings.py
LOG_FILE='myproject.log'

# CRITICAL > ERROR > WARNING > INFO > DEBUG
LOG_LEVEL='CRITICAL' # >=CRITICAL
LOG_LEVEL='ERROR' # >=ERROR
LOG_LEVEL='WARNING' # 显示>=WARNING
LOG_LEVEL='INFO' # 显示>=INFO
LOG_LEVEL='DEBUG' # 只显示DEBUG
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