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