# Scrapy-Redis

- [Scrapy-Redis](#scrapy-redis)
  - [Introduction](#introduction)
  - [Spider with localhost Redis](#spider-with-localhost-redis)

## Introduction

scrapy-redis与scrapy的不同
- 存item数据: 对应redis数据库中`spider_name:items`
- 存待爬request: 对应redis数据库中`spider_name:requests`
- 存request的指纹(hash value): 对应redis数据库中`spdier_name:dupefilter`

通过redis的subscribe-publish实现多个客户端之间的通信
> ![](res/scrapy_redis_architecture_01.jpg)

`pip install scrapy-redis`, scrapy-redis 四个组件
- Scheduler: 通过subscribe-publish和redis队列来实现数据的采集，调度
- Duplication Filter: 通过redis的set实现
- Item Pipeline: 如果有优先级，是通过zset实现
- Base Spiders.

分布式使用流程:
- Master只用启动redis-server即可，并提供ip,port
- 一套代码复制到多台机器slave。多个slave端只要有python, scrapy, scrapy-redis并配置ip, port，然后启动爬虫
- 随便一台机器连接redis-server发送给命令: `lpush myspider:start_urls http://www.dmoz.org/`

scrapy-reids实现分布式的两个class: `RedisSpider`, `RedisCrawlSpider`

example: scrapy redis [example](https://github.com/rmax/scrapy-redis/blob/master/example-project/example/spiders/dmoz.py)
> ![](res/scrapy-redis01.png)  
> settings.py中设置redis的ip, port即可使用，但并没有继承scrapy-redis的类，所以只是为了演示scrapy能够连接redis: `scrapy crawl dmoz`;  
> 并且项目停止之后，指纹仍然保存在redis中，下次重新启动爬虫，爬虫从上次的位置开始继续爬取，也就是`SCHEDULER_PERSIST=True`

```py
# settings.py
SPIDER_MODULES = ['example.spiders']
NEWSPIDER_MODULE = 'example.spiders'

DUPEFILTER_CLASS = "scrapy_redis.dupefilter.RFPDupeFilter"
SCHEDULER = "scrapy_redis.scheduler.Scheduler"


# True停止爬虫(比如加个新功能)，再重启爬虫，爬虫继续爬; False爬虫重新爬
SCHEDULER_PERSIST = True

#SCHEDULER_QUEUE_CLASS = "scrapy_redis.queue.SpiderPriorityQueue"
#SCHEDULER_QUEUE_CLASS = "scrapy_redis.queue.SpiderQueue"
#SCHEDULER_QUEUE_CLASS = "scrapy_redis.queue.SpiderStack"

ITEM_PIPELINES = {
    # 加时间戳的优先级300比进入redis数据库的优先级400要高
    # 这两个不要动
    'example.pipelines.ExamplePipeline': 300,
    # 处理item可以使用scrapy_redis的pipeline
    # 如果item数据都放在redis上，会增加redis压力；而且每个slave还得把item通过网络传递给redis, 会降低效率；所以一般slave机器分别保存自己的item到本地的mongodb上，所以一般注释掉下面这行
    'scrapy_redis.pipelines.RedisPipeline': 400,
}

# 这两个不写就是localhost
REDIS_HOST='192.168.1.10'
REDIS_PORT=6379
# REDIS_URL更好用，可以配置redis密码
```

```py
# pipelines.py
# 只是redis处理数据之前，先加入时间戳和name
from datetime import datetime

class ExamplePipeline(object):
    def process_item(self, item, spider):
        item["crawled"] = datetime.utcnow()
        item["spider"] = spider.name
        return item
```

```py
from scrapy.linkextractors import LinkExtractor
from scrapy.spiders import CrawlSpider, Rule

class DmozSpider(CrawlSpider):
    """Follow categories and extract links."""
    name = 'dmoz'
    allowed_domains = ['dmoz.org']
    start_urls = ['http://www.dmoz.org/']

    rules = [
        Rule(LinkExtractor(
            restrict_css=('.top-cat', '.sub-cat', '.cat-item')
        ), callback='parse_directory', follow=True),
    ]

    def parse_directory(self, response):
        for div in response.css('.title-and-desc'):
            yield {
                'name': div.css('.site-title::text').extract_first(),
                'description': div.css('.site-descr::text').extract_first().strip(),
                'link': div.css('a::attr(href)').extract_first(),
            }
```

example: scrapy redis [RedisSpider example](https://github.com/rmax/scrapy-redis/blob/master/example-project/example/spiders/myspider_redis.py)
> ![](res/scrapy-redis02.png)  
> 这个RedisSpider例子可以演示分布式: `scrapy runspider myspider_redis.py`; `lpush`一个url, 程序爬一次， 要爬取多次也需要构造通过`yield scrapy.Request()`, 或者写一个redis client循环lpush url到redis

```py
from scrapy_redis.spiders import RedisSpider

class MySpider(RedisSpider):
    """Spider that reads urls from redis queue (myspider:start_urls)."""
    name = 'myspider_redis'
    redis_key = 'myspider:start_urls' # 启动Slave端爬虫的key

    # 要么采用如下的方式，要么采用下面方式通过url动态获取域名，两种方式没有本质区别
    # allowed_domains = ['wz.sun0769.com']

    def __init__(self, *args, **kwargs):
        # Dynamically define the allowed domains list.
        domain = kwargs.pop('domain', '')
        self.allowed_domains = filter(None, domain.split(','))
        super(MySpider, self).__init__(*args, **kwargs)

    def parse(self, response):
        return {
            'name': response.css('title::text').extract_first(),
            'url': response.url,
        }
```

```py
# main.py和items.py同一层
from scrapy import cmdline

cmdline.execute('scrapy runspider myspider_redis.py'.split())
```

example: `RedisCrawlSpider`
> ![](res/scrapy-redis03.png)  
> 这个RedisSpider例子可以演示分布式: `scrapy runspider mycrawler_redis.py`

```py
from scrapy.spiders import Rule
from scrapy.linkextractors import LinkExtractor
from scrapy_redis.spiders import RedisCrawlSpider

class MyCrawler(RedisCrawlSpider):
    """Spider that reads urls from redis queue (myspider:start_urls)."""
    name = 'mycrawler_redis'
    redis_key = 'mycrawler:start_urls'

    rules = (
        # follow all links
        Rule(LinkExtractor(), callback='parse_page', follow=True),
    )

    def __init__(self, *args, **kwargs):
        # Dynamically define the allowed domains list.
        domain = kwargs.pop('domain', '')
        self.allowed_domains = filter(None, domain.split(','))
        super(MyCrawler, self).__init__(*args, **kwargs)

    def parse_page(self, response):
        return {
            'name': response.css('title::text').extract_first(),
            'url': response.url,
        }
```

tips: 如果slave机器不多，可以每台机器用git同步代码；如果slave机器太多，就需要用到[scrapyd](https://scrapyd.readthedocs.io/en/latest/) or [python-scrapyd-api](https://python-scrapyd-api.readthedocs.io/en/latest/)来部署(deployment)

tip: put file from local to remote

```bash
# in localmachine
tar -cvf your_folder.tar your_foler
# tar and zip: tar -zcvf your_folder.tar.gz your_foler

sftp username@192.168.1.111 # then enter your password
sftp> ls # list remote directory
sftp> lls # local list directory
sftp> put your_folder.tar

# in remote machine
tar -xvf your_foler.tar
# unzip: tar -zxvf your_foler.tar.gz
```

## Spider with localhost Redis

start localhost [redis of windows](https://github.com/microsoftarchive/redis/releases): `redis-server.exe redis.windows.conf`

```bash
# redis.windows.conf

# bind 127.0.0.1
requirepass xxxxxx
```

example: `scrapy.spiders.Spider` with localhost redis

in Anaconda Prompt: `scrapy crawl lsm`

```
.
|-- example
|   |-- __init__.py
|   |-- items.py
|   |-- pipelines.py
|   |-- settings.py
|   `-- spiders
|       |-- __init__.py
|       |-- lsm.py
|-- process_items.py
`-- scrapy.cfg
```

```py
# settings.py
SPIDER_MODULES = ['example.spiders']
NEWSPIDER_MODULE = 'example.spiders'

USER_AGENT = 'Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:72.0) Gecko/20100101 Firefox/72.0'

DUPEFILTER_CLASS = "scrapy_redis.dupefilter.RFPDupeFilter"
SCHEDULER = "scrapy_redis.scheduler.Scheduler"
SCHEDULER_PERSIST = True
#SCHEDULER_QUEUE_CLASS = "scrapy_redis.queue.SpiderPriorityQueue"
#SCHEDULER_QUEUE_CLASS = "scrapy_redis.queue.SpiderQueue"
#SCHEDULER_QUEUE_CLASS = "scrapy_redis.queue.SpiderStack"

ITEM_PIPELINES = {
    'example.pipelines.ExamplePipeline': 300,
    'scrapy_redis.pipelines.RedisPipeline': 400,
}

DOWNLOAD_DELAY = 1
# connect to db=2
REDIS_URL = 'redis://:xxxxxx@localhost:6379/2'
```

```py
# add class to items.py
class LsmItem(Item):
    title=Field()
    src=Field()
    url=Field()
    # 这两个提供给pipelines.py
    spider = Field()
    url = Field()
```

```py
# lsm.py
from scrapy import spiders, Request

class DmozSpider(spiders.Spider):
    name = 'lsm'
    allowed_domains = ['www.lesmao.co']
    pg=1
    start_urls = ['https://www.lesmao.co/plugin.php?id=group&page=1']

    def parse(self, response):
        for data in response.xpath('//div[@class="photo"]/a'):
            yield {
                'url':data.xpath('./@href').extract_first(),
                'src':data.xpath('./img/@src').extract_first(),
                'title':data.xpath('./img/@alt').extract_first(),
            }
        
        if self.pg<5:
            self.pg+=1
        yield Request(f'https://www.lesmao.co/plugin.php?id=group&page={self.pg}', callback=self.parse)
```

example: `scrapy.spiders.CrawlSpider` with localhost redis
> `scrapy crawl lsm2`

```py
# add class to items.py
class Lsm2Item(Item):
    titles=Field()
    srcs=Field()
    urls=Field()
    # 这两个提供给pipelines.py
    spider = Field()
    url = Field()
```

```py
# lsm2.py
from scrapy.linkextractors import LinkExtractor
from scrapy.spiders import CrawlSpider, Rule

class DmozSpider(CrawlSpider):
    name = 'lsm2'
    allowed_domains = ['www.lesmao.co']
    start_urls = ['https://www.lesmao.co/plugin.php?id=group&page=1']

    rules = [
        Rule(LinkExtractor(
            allow=(r'page=\d+')
        ), callback='myparse', follow=True),
    ]

    def myparse(self, response):
        yield {
                'urls':response.xpath('//div[@class="photo"]/a/@href').extract(),
                'srcs':response.xpath('//div[@class="photo"]/a/img/@src').extract(),
                'titles':response.xpath('//div[@class="photo"]/a/img/@alt').extract(),
            }
```

example: `scrapy_redis.spiders.RedisSpider` with localhost redis
> in Anaconda prompt: `scrapy runspider lsm3.py`  
> in redis-cli: `lpush lsm3:start_urls https://www.lesmao.co`

```py
# items.py
# 采用lsm1.py中的class
```

```py
# lsm3.py
from scrapy_redis.spiders import RedisSpider
from scrapy import Request

class MySpider(RedisSpider):
    name = 'lsm3'
    redis_key = 'lsm3:start_urls'
    allowed_domains = ['www.lesmao.co']
    pg=1

    def parse(self, response):
        for item in response.xpath('//div[@class="photo"]/a'):
            yield {
                'url': item.xpath('./@href').extract_first(),
                'src': item.xpath('./img/@src').extract_first(),
                'title': item.xpath('./img/@alt').extract_first(),
            }

        if self.pg < 5:
            self.pg += 1
        yield Request(f'https://www.lesmao.co/plugin.php?id=group&page={self.pg}', callback=self.parse)
```

example: `scrapy_redis.spiders.RedisCrawlSpider` with localhost redis
> in Anaconda prompt: `scrapy runspider lsm4.py`  
> in redis-cli: `lpush lsm4:start_urls https://www.lesmao.co`

```py
# items.py
# 采用lsm2.py中的class
```

```py
# lsm4.py
from scrapy.spiders import Rule
from scrapy.linkextractors import LinkExtractor
from scrapy_redis.spiders import RedisCrawlSpider


class MyCrawler(RedisCrawlSpider):
    name = 'lsm4'
    redis_key = 'lsm4:start_urls'
    allowed_domains = ['www.lesmao.co']

    rules = (
        Rule(LinkExtractor(allow=r'page=\d+'), callback='parse_page', follow=True),
    )

    def parse_page(self, response):
        yield {
            'urls': response.xpath('//div[@class="photo"]/a/@href').extract(),
            'srcs': response.xpath('//div[@class="photo"]/a/img/@src').extract(),
            'titles': response.xpath('//div[@class="photo"]/a/img/@alt').extract(),
        }
```