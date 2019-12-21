# Scrapy Framework

- [Scrapy Framework](#scrapy-framework)
  - [Introduction](#introduction)

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
# -*- coding: utf-8 -*-
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