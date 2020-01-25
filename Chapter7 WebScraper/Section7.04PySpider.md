# PySpider Framework

`pip install pyspider`
> python <3.7, 不需要修改源码  
> python>=3.7, 需要修改pyspider的源码(因为async是python3.7的keyword)，将async替换为其他名称  
- 配置phantomjs到环境变量，可以渲染js
- 不配置phantomjs，直接请求

[PySpider](https://github.com/binux/pyspider) Features:
- 多进程处理
- 去重处理
- pyquery提取
- 结果监控
- 错误重试
- WebUI管理(scrapy没有)
- JS渲染(scrapy没有): 通过phantomjs添加入环境变量来实现
- 代码简洁

example: lsm by pyspider
1. `pyspider all` or `pyspider`
2. 浏览器访问`localhost:5000`
3. 浏览器GUI, create a project
4. 通过gui以及手动添加内容, save
5. 在GUI界面STATUS修改为DEBUG或者RUNNING, 然后run

```py
#!/usr/bin/env python
# -*- encoding: utf-8 -*-
# Created on 2020-01-09 13:05:40
# Project: demo

from pyspider.libs.base_handler import *
import pymongo

class Handler(BaseHandler):
    crawl_config = {
        # global setting
        'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:72.0) Gecko/20100101 Firefox/72.0',
    }
    
    # add by user
    client=pymongo.MongoClient("mongodb://grey:pwd@ip:27017")
    db=client['test']

    @every(minutes=24 * 60)
    def on_start(self):
        self.crawl('https://www.lsmpx.com/', callback=self.index_page)

    @config(age=10 * 24 * 60 * 60)
    def index_page(self, response):
        for each in response.doc('h2 > a').items():
            self.crawl(each.attr.href, callback=self.detail_page)
        
        # add by user
        next=response.doc('.pg a:last-child').attr.href
        self.crawl(next, callback=self.index_page)
        
    @config(priority=2)
    def detail_page(self, response):
        data= {
            "url": response.url,
            "date": response.doc('#thread-title > em').text(),
            "source":response.doc('#thread-title > a').text(),
            "views":response.doc('#thread-title > span').eq(0).text(),
            "comments":response.doc('#thread-title > span:nth-child(5)').text(),
        }
    
        return data
    
    # add by user
    def on_result(self, result):
        if result:
            self.save_to_mongo(result)
    
    # add by user
    def save_to_mongo(self, result):
        # 第一个参数是query用于定位
        # 第二个参数是用于update的数据
        # upsert: insert a new document if a matching document does not exist.
        if self.db['lsm'].update({'url': result['url']}, {'$set': result}, upsert=True):
            print('save to mongodb')
```

tip: GUI中的rate表示每秒发送多少request, burst是令牌桶的数量, 后面几个progressbar是统计信息
> ![](res/pyspider01.png)

example: pyspider with douban json

```py
from pyspider.libs.base_handler import *

class Handler(BaseHandler):
    def on_start(self):
        self.crawl('http://movie.douban.com/j/search_subjects?type=movie&tag=%E7%83%AD%E9%97%A8&sort=recommend&page_limit=20&page_start=0',
                   callback=self.json_parser)

    def json_parser(self, response):
        return [{
            "title": x['title'],
            "rate": x['rate'],
            "url": x['url']
        } for x in response.json['subjects']]
```

example: pyspider with pagination

```py
from pyspider.libs.base_handler import *

class Handler(BaseHandler):
    @every(minutes=24 * 60)
    def on_start(self):
        pg=1
        while pg <= 30:
            url = f'https://mm.taobao.com/json/request_top_list.htm?page={pg}'
            self.crawl(url, callback=self.index_page)
            pg += 1

    @config(age=10 * 24 * 60 * 60)
    def index_page(self, response):
        for each in response.doc('a[href^="http"]').items():
            self.crawl(each.attr.href, callback=self.detail_page)

    @config(priority=2)
    def detail_page(self, response):
        return {
            "url": response.url,
            "title": response.doc('title').text(),
        }
```

```py
import re
from pyspider.libs.base_handler import *

class Handler(BaseHandler):
    @every(minutes=24 * 60)
    def on_start(self):
        self.crawl('http://movie.douban.com/tag/', callback=self.index_page)

    @config(age=24 * 60 * 60)
    def index_page(self, response):
        for each in response.doc('a[href^="http"]').items():
            if re.match("http://movie.douban.com/tag/\w+", each.attr.href, re.U):
                self.crawl(each.attr.href, callback=self.list_page)

    @config(age=10 * 24 * 60 * 60, priority=2)
    def list_page(self, response):
        for each in response.doc('HTML>BODY>DIV#wrapper>DIV#content>DIV.grid-16-8.clearfix>DIV.article>DIV>TABLE TR.item>TD>DIV.pl2>A').items():
            self.crawl(each.attr.href, priority=9, callback=self.detail_page)
        # 翻页
        for each in response.doc('HTML>BODY>DIV#wrapper>DIV#content>DIV.grid-16-8.clearfix>DIV.article>DIV.paginator>A').items():
            self.crawl(each.attr.href, callback=self.list_page)

    @config(priority=3)
    def detail_page(self, response):
        return {
            "url": response.url,
            "title": response.doc('HTML>BODY>DIV#wrapper>DIV#content>H1>SPAN').text(),
            "导演": [x.text() for x in response.doc('a[rel="v:directedBy"]').items()],
        }
```

example: pyspider with phantomjs
> phantomjs already in environment variable

```py
class Handler(BaseHandler):
    def on_start(self):
        # 设置fetch_type来使用phantomjs
        self.crawl('http://movie.douban.com/explore',fetch_type='js', callback=self.phantomjs_parser)

    def phantomjs_parser(self, response):
        return [{
            "rate": x('p strong').text(),
            "url": x.attr.href,
        } for x in response.doc('a.item').items()]
```

```py
class Handler(BaseHandler):
    def on_start(self):
        # 设置js_script来执行翻页
        self.crawl('http://movie.douban.com/explore#more',fetch_type='js', js_script="""
                function() {
                    setTimeout("$('.more').click()", 1000);
                }""", callback=self.phantomjs_parser)

    def phantomjs_parser(self, response):
        return [{
            "rate": x('p strong').text(),
            "url": x.attr.href,
        } for x in response.doc('a.item').items()]
```

example: pyspider download image

```py
from pyspider.libs.base_handler import *
import os

PAGE_START = 1
PAGE_END = 30
DIR_PATH = '/var/py/mm'

class Handler(BaseHandler):
    def __init__(self):
        self.base_url = 'https://mm.taobao.com/json/request_top_list.htm?page='
        self.page_num = PAGE_START
        self.total_num = PAGE_END
        self.deal = Deal()

    def on_start(self):
        while self.page_num <= self.total_num:
            url = self.base_url + str(self.page_num)
            self.crawl(url, callback=self.index_page)
            self.page_num += 1

    def index_page(self, response):
        for each in response.doc('.lady-name').items():
            self.crawl(each.attr.href, callback=self.detail_page, fetch_type='js')

    def detail_page(self, response):
        domain = response.doc('.mm-p-domain-info li > span').text()
        if domain:
            page_url = 'https:' + domain
            self.crawl(page_url, callback=self.domain_page)

    def domain_page(self, response):
        name = response.doc('.mm-p-model-info-left-top dd > a').text()
        dir_path = self.deal.mkDir(name)
        brief = response.doc('.mm-aixiu-content').text()
        if dir_path:
            self.deal.saveBrief(brief, dir_path, name)
            imgs = response.doc('.mm-aixiu-content img').items()
            count = 1
            for img in imgs:
                url = img.attr.src
                if url:
                    extension = self.deal.getExtension(url)
                    file_name = name + str(count) + '.' + extension
                    count += 1
                    self.crawl(img.attr.src, callback=self.save_img,save={'dir_path': dir_path, 'file_name': file_name})

    def save_img(self, response):
        content = response.content
        dir_path = response.save['dir_path']
        file_name = response.save['file_name']
        file_path = dir_path + '/' + file_name
        self.deal.saveImg(content, file_path)


class Deal:
    def __init__(self):
        self.path = DIR_PATH
        if not self.path.endswith('/'):
            self.path = self.path + '/'
        if not os.path.exists(self.path):
            os.makedirs(self.path)

    def mkDir(self, path):
        path = path.strip()
        dir_path = self.path + path
        exists = os.path.exists(dir_path)
        if not exists:
            os.makedirs(dir_path)
            return dir_path
        else:
            return dir_path

    def saveImg(self, content, path):
        f = open(path, 'wb')
        f.write(content)
        f.close()

    def saveBrief(self, content, dir_path, name):
        file_name = dir_path + "/" + name + ".txt"
        f = open(file_name, "w+")
        f.write(content.encode('utf-8'))

    def getExtension(self, url):
        extension = url.split('.')[-1]
        return extension
```