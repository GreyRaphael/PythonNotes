# Complicated Spider

- [Complicated Spider](#complicated-spider)
  - [Spider Acceleration](#spider-acceleration)
    - [Coroutine Spider](#coroutine-spider)
    - [coroutine, threading, multiprocessing](#coroutine-threading-multiprocessing)
  - [Distributed Spider](#distributed-spider)
  - [DFS & BFS Spider](#dfs--bfs-spider)

## Spider Acceleration

Acceleration methods:
- 多线程
- 多进程
- 协程
- 分布式: 消息队列、GPU加速、分布式框架

爬虫框架:
- scrapy，分布式scrap-redis: 只能爬不加密的简单网站
- pyspider: 可以爬简单的加密网站，因为用selenium实现

> 爬虫框架经常会失效

### Coroutine Spider

```python
import gevent

def func(n):
    for i in range(n):
        print(f'{i}=====>{gevent.getcurrent()}')
        gevent.sleep(1) # 碰到等待的操作，自动切换到其他上面去

g1=gevent.spawn(func,3)
g2=gevent.spawn(func,3)
g3=gevent.spawn(func,3)
g1.join()
g2.join()
g3.join()
```

```bash
# output
0=====><Greenlet "Greenlet-0" at 0x2daac3766a8: func(3)>
0=====><Greenlet "Greenlet-1" at 0x2daac376598: func(3)>
0=====><Greenlet "Greenlet-2" at 0x2daac376ae8: func(3)>
1=====><Greenlet "Greenlet-0" at 0x2daac3766a8: func(3)>
1=====><Greenlet "Greenlet-1" at 0x2daac376598: func(3)>
1=====><Greenlet "Greenlet-2" at 0x2daac376ae8: func(3)>
2=====><Greenlet "Greenlet-0" at 0x2daac3766a8: func(3)>
2=====><Greenlet "Greenlet-1" at 0x2daac376598: func(3)>
2=====><Greenlet "Greenlet-2" at 0x2daac376ae8: func(3)>
```

example: coroutine download

```python
import gevent
from gevent import monkey
# monkey要放在靠前的位置
monkey.patch_all()

import requests

def download(url):
    print(f'start download {url}')
    r=requests.get(url).text
    print(f'finish: {url}, length={len(r)}')

urls=['https://www.baidu.com', 'http://www.qq.com', 'https://www.163.com']

gevent_tasks=[]
for url in urls:
    gevent_tasks.append(gevent.spawn(download, url))


gevent.joinall(gevent_tasks)
```

```bash
# output
start download https://www.baidu.com
start download http://www.qq.com
start download https://www.163.com
finish: https://www.baidu.com, length=2443
finish: https://www.163.com, length=684958
finish: http://www.qq.com, length=231238
```

example: 挖掘老赖数据, without coroutine

```python
import requests
from jsonpath import jsonpath
import pandas as pd

total_pages=101
headers={
    "User-Agent":"Mozilla/5.0 (Windows NT 10.0; WOW64; rv:62.0) Gecko/20100101 Firefox/62.0",
    "Referer": "https://www.baidu.com/s?wd=%E8%80%81%E8%B5%96"
}

age=[]
sexy=[]
areaName=[]
duty=[]
iname=[]
publishDateStamp=[]
regDate=[]

for i in range(total_pages+1):
    url=f'https://sp0.baidu.com/8aQDcjqpAAV3otqbppnN2DJv/api.php?resource_id=6899&query=老赖&pn={i*10}&rn=10&ie=utf-8&oe=utf-8&format=json'
    r=requests.get(url, headers=headers).json()
    age.extend(jsonpath(r, '$..age')[:10])
    sexy.extend(jsonpath(r, '$..sexy')[:10])
    areaName.extend(jsonpath(r, '$..areaName')[:10])
    duty.extend(jsonpath(r, '$..duty')[:10])
    iname.extend(jsonpath(r, '$..iname')[:10])
    publishDateStamp.extend(jsonpath(r, '$..publishDateStamp')[:10])
    regDate.extend(jsonpath(r, '$..regDate')[:10])

# save to DataFrame for later analysis
df=pd.DataFrame({'age':age, 'sexy':sexy, 'areaName': areaName, 'duty':duty, 'iname':iname, 'publishDateStamp':publishDateStamp, 'regDate': regDate})
```

example: 挖掘老赖数据, with coroutine

```python
from jsonpath import jsonpath
import pandas as pd
import gevent
from gevent import monkey
monkey.patch_all()

import requests

total_pages=101
headers={
    "User-Agent":"Mozilla/5.0 (Windows NT 10.0; WOW64; rv:62.0) Gecko/20100101 Firefox/62.0",
    "Referer": "https://www.baidu.com/s?wd=%E8%80%81%E8%B5%96"
}

age=[]
sexy=[]
areaName=[]
duty=[]
iname=[]
publishDateStamp=[]
regDate=[]

def download(start, end):
    for i in range(start, end):
        url=f'https://sp0.baidu.com/8aQDcjqpAAV3otqbppnN2DJv/api.php?resource_id=6899&query=老赖&pn={i*10}&rn=10&ie=utf-8&oe=utf-8&format=json'
        r=requests.get(url, headers=headers).json()
        age.extend(jsonpath(r, '$..age')[:10])
        sexy.extend(jsonpath(r, '$..sexy')[:10])
        areaName.extend(jsonpath(r, '$..areaName')[:10])
        duty.extend(jsonpath(r, '$..duty')[:10])
        iname.extend(jsonpath(r, '$..iname')[:10])
        publishDateStamp.extend(jsonpath(r, '$..publishDateStamp')[:10])
        regDate.extend(jsonpath(r, '$..regDate')[:10])

gevent_tasks=[]
for i in range(0, 100, 10):
    gevent_tasks.append(gevent.spawn(download, i, i+10))
gevent_tasks.append(gevent.spawn(download, 100, total_pages+1))
gevent.joinall(gevent_tasks)

# save to DataFrame for later analysis
df=pd.DataFrame({'age':age, 'sexy':sexy, 'areaName': areaName, 'duty':duty, 'iname':iname, 'publishDateStamp':publishDateStamp, 'regDate': regDate})
```

examle: Coroutine Pool

```py
import requests
import re
from gevent import pool
from gevent import monkey

monkey.patch_all() # monkey可能与分布式爬虫冲突，可以关闭

HEADERS={'User-Agent':"Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:73.0) Gecko/20100101 Firefox/73.0"}
pat=re.compile(r'<h2><a href="(.+?)"')

def func(url):
    print('crawling --->', url)
    r=requests.get(url, headers=HEADERS)
    return pat.findall(r.text)

if __name__ == "__main__":
    po=pool.Pool(4)
    urls=[f'https://www.lesmao.co/plugin.php?id=group&page={i+1}' for i in range(10)]
    result=po.map(func, urls)
    po.join()
    print(result)
```

### coroutine, threading, multiprocessing

example1: simple example

```python
import re
import requests
from bs4 import BeautifulSoup

def get_total_numbers(url):
    html=requests.get(url).content.decode('gbk')
    total_numbers=re.search(r'共(\d+)条记录', html).group(1)
    return eval(total_numbers)

def make_urls(N):
    urls=[]
    if N % 30 ==0:
        for i in range(N//30):
            urls.append(f'http://wz.sun0769.com/index.php/question/report?page={i*30}')
    else:
        for i in range(N//30+1):
            urls.append(f'http://wz.sun0769.com/index.php/question/report?page={i*30}')
    return urls

def capture_data(html):
    soup=BeautifulSoup(html)

    id_list=[]
    type_list=[]
    title_list=[]
    location_list=[]
    status_list=[]
    person_list=[]
    time_list=[]

    for tr in soup.select('table[bgcolor="#FBFEFF"] tr'):
        id_list.append(tr.select_one('td:nth-of-type(1)').text)
        type_list.append(tr.select_one('td a:nth-of-type(1)').text)
        title_list.append(tr.select_one('td a:nth-of-type(2)').text)
        location_list.append(tr.select_one('td a:nth-of-type(3)').text)
        status_list.append(tr.select_one('td:nth-of-type(3)').text)
        person_list.append(tr.select_one('td:nth-of-type(4)').text)
        time_list.append(tr.select_one('td:nth-of-type(5)').text)

    data=[]
    for item in zip(id_list, type_list, title_list, location_list, status_list, person_list, time_list):
        data.append(';'.join(item))
    return data

def download_write(urls, file):
    for url in urls:
        try:
            html=requests.get(url).content.decode('gbk', errors='ignore')
            data=capture_data(html)
        except Exception as e:
            print(e, url)
        
        for line in data:
            file.write(line)
            file.write('\n')
            file.flush()

def main():
    total_numbers=get_total_numbers('http://wz.sun0769.com/html/top/report.shtml')
    urls=make_urls(total_numbers)

    with open('test.csv', 'w', encoding='utf8') as file:
        file.write('id;type;title;location;status;person;time\n')
        download_write(urls, file)

if __name__ == '__main__':
    main()
```

example2: simple example with **coroutine**

```python
import re
import gevent
from bs4 import BeautifulSoup
from gevent import monkey
monkey.patch_all()

import requests

def get_total_numbers(url):
    html=requests.get(url).content.decode('gbk')
    total_numbers=re.search(r'共(\d+)条记录', html).group(1)
    return eval(total_numbers)

def make_urls(N):
    urls=[]
    if N % 30 ==0:
        for i in range(N//30):
            urls.append(f'http://wz.sun0769.com/index.php/question/report?page={i*30}')
    else:
        for i in range(N//30+1):
            urls.append(f'http://wz.sun0769.com/index.php/question/report?page={i*30}')
    return urls

def capture_data(html):
    soup=BeautifulSoup(html, features='lxml')

    id_list=[]
    type_list=[]
    title_list=[]
    location_list=[]
    status_list=[]
    person_list=[]
    time_list=[]

    for tr in soup.select('table[bgcolor="#FBFEFF"] tr'):
        id_list.append(tr.select_one('td:nth-of-type(1)').text)
        type_list.append(tr.select_one('td a:nth-of-type(1)').text)
        title_list.append(tr.select_one('td a:nth-of-type(2)').text)
        location_list.append(tr.select_one('td a:nth-of-type(3)').text)
        status_list.append(tr.select_one('td:nth-of-type(3)').text)
        person_list.append(tr.select_one('td:nth-of-type(4)').text)
        time_list.append(tr.select_one('td:nth-of-type(5)').text)

    data=[]
    for item in zip(id_list, type_list, title_list, location_list, status_list, person_list, time_list):
        data.append(';'.join(item))
    return data

def download_write(urls, file):
    for url in urls:
        try:
            html=requests.get(url).content.decode('gbk', errors='ignore')
            data=capture_data(html)
        except Exception as e:
            print(e, url)
        
        for line in data:
            file.write(line)
            file.write('\n')
            file.flush()

def main():
    total_numbers=get_total_numbers('http://wz.sun0769.com/html/top/report.shtml')
    # total_numbers=105*30+5 # test
    urls=make_urls(total_numbers)
    segment_N=10

    file=open('test.csv', 'w', encoding='utf8')
    file.write('id;type;title;location;status;person;time\n')

    url_seg=[[] for _ in range(segment_N)]
    for i, url in enumerate(urls):
        url_seg[i % segment_N].append(url)

    task_list=[]
    for i in range(segment_N):
        task_list.append(gevent.spawn(download_write, url_seg[i], file))
    gevent.joinall(task_list)

    file.close()

if __name__ == '__main__':
    main()
```

example3: simple example with **threading**

```python
import re
from bs4 import BeautifulSoup
import requests
import threading

def get_total_numbers(url):
    html=requests.get(url).content.decode('gbk')
    total_numbers=re.search(r'共(\d+)条记录', html).group(1)
    return eval(total_numbers)

def make_urls(N):
    urls=[]
    if N % 30 ==0:
        for i in range(N//30):
            urls.append(f'http://wz.sun0769.com/index.php/question/report?page={i*30}')
    else:
        for i in range(N//30+1):
            urls.append(f'http://wz.sun0769.com/index.php/question/report?page={i*30}')
    return urls

def capture_data(html):
    soup=BeautifulSoup(html, features='lxml')

    id_list=[]
    type_list=[]
    title_list=[]
    location_list=[]
    status_list=[]
    person_list=[]
    time_list=[]

    for tr in soup.select('table[bgcolor="#FBFEFF"] tr'):
        id_list.append(tr.select_one('td:nth-of-type(1)').text)
        type_list.append(tr.select_one('td a:nth-of-type(1)').text)
        title_list.append(tr.select_one('td a:nth-of-type(2)').text)
        location_list.append(tr.select_one('td a:nth-of-type(3)').text)
        status_list.append(tr.select_one('td:nth-of-type(3)').text)
        person_list.append(tr.select_one('td:nth-of-type(4)').text)
        time_list.append(tr.select_one('td:nth-of-type(5)').text)

    data=[]
    for item in zip(id_list, type_list, title_list, location_list, status_list, person_list, time_list):
        data.append(';'.join(item))
    return data

def download_write(urls, file):
    for url in urls:
        try:
            html=requests.get(url).content.decode('gbk', errors='ignore')
            data=capture_data(html)
        except Exception as e:
            print(e, url)
        
        with g_lock:
            for line in data:
                file.write(line)
                file.write('\n')
                file.flush()

def main():
    total_numbers=get_total_numbers('http://wz.sun0769.com/html/top/report.shtml')
    # total_numbers=105*30+5 # test
    urls=make_urls(total_numbers)
    segment_N=10

    file=open('test.csv', 'w', encoding='utf8')
    file.write('id;type;title;location;status;person;time\n')

    url_seg=[[] for _ in range(segment_N)]
    for i, url in enumerate(urls):
        url_seg[i % segment_N].append(url)

    task_list=[]
    for i in range(segment_N):
        th=threading.Thread(target=download_write, args=(url_seg[i], file))
        task_list.append(th)
        th.start()

    for th in task_list:
        th.join()

    file.close()

if __name__ == '__main__':
    g_lock=threading.Lock()
    main()
```

example4: simpel example with **multiprocessing**
> 读取完毕然后一次性写入，边读边写可能会丢数据

```python
import re
from bs4 import BeautifulSoup
import requests
import multiprocessing

def get_total_numbers(url):
    html=requests.get(url).content.decode('gbk')
    total_numbers=re.search(r'共(\d+)条记录', html).group(1)
    return eval(total_numbers)

def make_urls(N):
    urls=[]
    if N % 30 ==0:
        for i in range(N//30):
            urls.append(f'http://wz.sun0769.com/index.php/question/report?page={i*30}')
    else:
        for i in range(N//30+1):
            urls.append(f'http://wz.sun0769.com/index.php/question/report?page={i*30}')
    return urls

def capture_data(html):
    soup=BeautifulSoup(html, features='lxml')

    id_list=[]
    type_list=[]
    title_list=[]
    location_list=[]
    status_list=[]
    person_list=[]
    time_list=[]

    for tr in soup.select('table[bgcolor="#FBFEFF"] tr'):
        id_list.append(tr.select_one('td:nth-of-type(1)').text)
        type_list.append(tr.select_one('td a:nth-of-type(1)').text)
        title_list.append(tr.select_one('td a:nth-of-type(2)').text)
        location_list.append(tr.select_one('td a:nth-of-type(3)').text)
        status_list.append(tr.select_one('td:nth-of-type(3)').text)
        person_list.append(tr.select_one('td:nth-of-type(4)').text)
        time_list.append(tr.select_one('td:nth-of-type(5)').text)

    data=[]
    for item in zip(id_list, type_list, title_list, location_list, status_list, person_list, time_list):
        data.append(';'.join(item))
    return data

def download_write(urls, q):
    for url in urls:
        try:
            html=requests.get(url).content.decode('gbk', errors='ignore')
            data=capture_data(html)
        except Exception as e:
            print(e, url)
        
        q.put(data)
    print(f'{multiprocessing.current_process().name} finished')

def main():
    # total_numbers=get_total_numbers('http://wz.sun0769.com/html/top/report.shtml')
    total_numbers=105*30+5 # test
    urls=make_urls(total_numbers)
    segment_N=10

    url_seg=[[] for _ in range(segment_N)]
    for i, url in enumerate(urls):
        url_seg[i % segment_N].append(url)

    # python2中multiprocessing.Queue()无法pickle，建议使用multiprocessing.Manager().Queue()；python3都行
    q=multiprocessing.Queue()
    task_list=[]
    for i in range(segment_N):
        p=multiprocessing.Process(target=download_write, args=(url_seg[i], q))
        p.start()
        task_list.append(p)

    for p in task_list:
        p.join()
    
    with open('test.csv', 'w', encoding='utf8') as file:
        file.write('id;type;title;location;status;person;time\n')
        while not q.empty():
            data=q.get()
            for line in data:
                file.write(line)
                file.write('\n')
                file.flush()

if __name__ == '__main__':
    main()
```
> python2 [multiprocessing.Queue() vs multiprocessing.Manager().Queue()](https://stackoverflow.com/questions/43439194)

example5: simpel example with **multiprocessing**
> 读取完毕然后一次性写入，内存压力太大，已知要get的数目，可以实现边读边写  
> [stackoverflow Example](https://stackoverflow.com/questions/32395150)

```python
import re
from bs4 import BeautifulSoup
import requests
import multiprocessing

def get_total_numbers(url):
    html=requests.get(url).content.decode('gbk')
    total_numbers=re.search(r'共(\d+)条记录', html).group(1)
    return eval(total_numbers)

def make_urls(N):
    urls=[]
    if N % 30 ==0:
        for i in range(N//30):
            urls.append(f'http://wz.sun0769.com/index.php/question/report?page={i*30}')
    else:
        for i in range(N//30+1):
            urls.append(f'http://wz.sun0769.com/index.php/question/report?page={i*30}')
    return urls

def capture_data(html):
    soup=BeautifulSoup(html, features='lxml')

    id_list=[]
    type_list=[]
    title_list=[]
    location_list=[]
    status_list=[]
    person_list=[]
    time_list=[]

    for tr in soup.select('table[bgcolor="#FBFEFF"] tr'):
        id_list.append(tr.select_one('td:nth-of-type(1)').text)
        type_list.append(tr.select_one('td a:nth-of-type(1)').text)
        title_list.append(tr.select_one('td a:nth-of-type(2)').text)
        location_list.append(tr.select_one('td a:nth-of-type(3)').text)
        status_list.append(tr.select_one('td:nth-of-type(3)').text)
        person_list.append(tr.select_one('td:nth-of-type(4)').text)
        time_list.append(tr.select_one('td:nth-of-type(5)').text)

    data=[]
    for item in zip(id_list, type_list, title_list, location_list, status_list, person_list, time_list):
        data.append(';'.join(item))
    return data

def download(urls, q):
    for url in urls:
        try:
            html=requests.get(url).content.decode('gbk', errors='ignore')
            data=capture_data(html)
        except Exception as e:
            print(e, url)
        
        q.put(data)
    print(f'{multiprocessing.current_process().name} finished')

def write_data(q, total_pages):
    with open('test.csv', 'w', encoding='utf8') as file:
        file.write('id;type;title;location;status;person;time\n')
        for _ in range(total_pages):
            data=q.get()
            for line in data:
                file.write(line)
                file.write('\n')
                file.flush()
    print('finish writting...')

def main():
    # total_numbers=get_total_numbers('http://wz.sun0769.com/html/top/report.shtml')
    total_numbers=105*30+5 # test
    urls=make_urls(total_numbers)
    total_pages=len(urls) # 很重要，便于得到等待的次数
    segment_N=10

    url_seg=[[] for _ in range(segment_N)]
    for i, url in enumerate(urls):
        url_seg[i % segment_N].append(url)

    q=multiprocessing.Queue()
    task_list=[]
    for i in range(segment_N):
        p=multiprocessing.Process(target=download, args=(url_seg[i], q))
        p.start()
        task_list.append(p)
    
    read_process=multiprocessing.Process(target=write_data, args=(q, total_pages))
    read_process.start()
    task_list.append(read_process)

    for p in task_list:
        p.join()


if __name__ == '__main__':
    main()
```

## Distributed Spider

Distributed:
- 分布式计算
- 分布式控制
- 分布式爬虫

分布式意味着在局域网中多台电脑干活。
> 要分成Server, Client  
> Server: 将任务put到task_queue中，从result_queue中get结果  
> Client: 从task_queue中get任务，计算得到结果，将结果put到result_queue中

example1: 分布式计算

```python
# Server
from multiprocessing import Process, Queue
from multiprocessing.managers import BaseManager

class Worker(Process):
    def __init__(self, tq, rq):
        self.tq = tq
        self.rq = rq
        super().__init__()

    def run(self):
        for i in range(3):
            self.tq.put(i)
        print('waiting for ...')
        for _ in range(3):
            print(self.rq.get())

class QueueManager(BaseManager):pass

def main():
    tq = Queue()
    rq = Queue()
    w = Worker(tq, rq)
    w.start()

    QueueManager.register('task_queue', callable=lambda: tq) # lambda的return不用写
    QueueManager.register('result_queue', callable=lambda: rq)# lambda的return不用写
    m = QueueManager(address=('', 6666), authkey=b'666666')
    s = m.get_server()
    s.serve_forever()


if __name__ == '__main__':
    main()
```

```python
# Client
from multiprocessing.managers import BaseManager

class QueueManager(BaseManager): pass

def main():
    QueueManager.register('task_queue')
    QueueManager.register('result_queue')
    
    m = QueueManager(address=('127.0.0.1', 6666), authkey=b'666666')
    m.connect()

    tq=m.task_queue()
    rq=m.result_queue()

    for _ in range(3):
        data=tq.get()
        print(f'client get: {data}')
        rq.put(data**2)

if __name__ == '__main__':
    main()
```

example2: 分布式控制

```python
# Server在example1基础上修改
class Worker(Process):
    def __init__(self, tq, rq):
        self.tq = tq
        self.rq = rq
        super().__init__()

    def run(self):
        self.tq.put('notepad')
        self.tq.put('mspaint')
        self.tq.put('explorer')
        print('waiting for ...')
        for _ in range(3):
            print(self.rq.get())
````

```python
# Client
from multiprocessing.managers import BaseManager
import os

class QueueManager(BaseManager): pass

def main():
    QueueManager.register('task_queue')
    QueueManager.register('result_queue')
    
    m = QueueManager(address=('127.0.0.1', 6666), authkey=b'666666')
    m.connect()

    tq=m.task_queue()
    rq=m.result_queue()

    for i in range(3):
        cmd=tq.get()
        os.system(cmd)
        rq.put(i**2)

if __name__ == '__main__':
    main()
```

example3: 分布式作业系统

```python
# Server在example1基础上修改
class Worker(Process):
    def __init__(self, tq, rq):
        self.tq = tq
        self.rq = rq
        super().__init__()

    def run(self):
        self.tq.put('python my_work1.py')
        print('waiting for ...')
        for _ in range(3):
            print(self.rq.get())
```

```python
# Client与example2相同
# 另外写一个在client端写一个my_work1.py
```

example4: 分布式爬虫

Server抓取urls并put进入task_queue, Clients get task_queue中的urls并提取data，Clients将data put进入result_queue, Server get result_queue中的data并保存。

因为一般的云端的Linux没有GUI，无法用selenium，所以Server.py放在云端运行，Client.py放在windows运行。

```python
# Server
import re
import multiprocessing
from multiprocessing.managers import BaseManager
import requests

def get_total_numbers(url):
    html=requests.get(url).content.decode('gbk')
    total_numbers=re.search(r'共(\d+)条记录', html).group(1)
    return eval(total_numbers)

def make_urls(N):
    urls=[]
    if N % 30 ==0:
        for i in range(N//30):
            urls.append(f'http://wz.sun0769.com/index.php/question/report?page={i*30}')
    else:
        for i in range(N//30+1):
            urls.append(f'http://wz.sun0769.com/index.php/question/report?page={i*30}')
    return urls

def write_data(q, total_pages):
    print('Server begins writing...')
    with open('test.csv', 'w', encoding='utf8') as file:
        file.write('id;type;title;location;status;person;time\n')
        for _ in range(total_pages):
            data=q.get()
            for line in data:
                file.write(line)
                file.write('\n')
                file.flush()
    print('Server finishes writting!')

class Worker(multiprocessing.Process):
    def __init__(self, tq, rq):
        self.tq = tq
        self.rq = rq
        super().__init__()

    def run(self):
        # total_numbers=get_total_numbers('http://wz.sun0769.com/html/top/report.shtml')
        total_numbers=105*30+5 # test
        urls=make_urls(total_numbers)
        total_pages=len(urls) # 很重要，便于得到等待的次数        

        for url in urls:
            self.tq.put(url)
        print('Server finishes putting urls in task_queue...')
        write_data(self.rq, total_pages)


class QueueManager(BaseManager):pass

def main():
    tq = multiprocessing.Queue()
    rq = multiprocessing.Queue()
    
    w = Worker(tq, rq)
    w.start()

    QueueManager.register('task_queue', callable=lambda: tq) # lambda的return不用写
    QueueManager.register('result_queue', callable=lambda: rq)# lambda的return不用写
    m = QueueManager(address=('', 6666), authkey=b'666666')
    s = m.get_server()
    s.serve_forever()


if __name__ == '__main__':
    main()
```

```python
# Client
from multiprocessing.managers import BaseManager
from bs4 import BeautifulSoup
import requests

def capture_data(html):
    soup=BeautifulSoup(html, features='html5lib')

    id_list=[]
    type_list=[]
    title_list=[]
    location_list=[]
    status_list=[]
    person_list=[]
    time_list=[]

    for tr in soup.select('table[bgcolor="#FBFEFF"] tr'):
        id_list.append(tr.select_one('td:nth-of-type(1)').text)
        type_list.append(tr.select_one('td a:nth-of-type(1)').text)
        title_list.append(tr.select_one('td a:nth-of-type(2)').text)
        location_list.append(tr.select_one('td a:nth-of-type(3)').text)
        status_list.append(tr.select_one('td:nth-of-type(3)').text)
        person_list.append(tr.select_one('td:nth-of-type(4)').text)
        time_list.append(tr.select_one('td:nth-of-type(5)').text)

    data=[]
    for item in zip(id_list, type_list, title_list, location_list, status_list, person_list, time_list):
        data.append(';'.join(item))
    return data

def download(urls, q):
    for url in urls:
        try:
            html=requests.get(url).content.decode('gbk', errors='ignore')
            data=capture_data(html)
        except Exception as e:
            print(e, url)
        
        print('Clients finishes:', url)
        q.put(data)


class QueueManager(BaseManager): pass

def main():
    QueueManager.register('task_queue')
    QueueManager.register('result_queue')
    
    m = QueueManager(address=('222.29.69.149', 6666), authkey=b'666666')
    m.connect()

    tq=m.task_queue()
    rq=m.result_queue()

    urls=[]
    while not tq.empty():
        urls.append(tq.get())
    print(f'client gets: {len(urls)} urls')
    download(urls, rq)

if __name__ == '__main__':
    main()
```

example: 生产者消费者模式爬虫

```py
import re
import requests
from queue import Queue
from threading import Thread


class CrawThread(Thread):
    def __init__(self, ID, pageQ, htmlQ):
        super().__init__()
        self.ID = ID
        self.pageQ = pageQ
        self.htmlQ = htmlQ

    def run(self):
        while not self.pageQ.empty():
            try:
                pg = self.pageQ.get(block=False)
                url = f'https://www.lsmpx.com/plugin.php?id=group&page={pg}'
                r = requests.get(url, headers={
                                 'User-Agent': "Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:71.0) Gecko/20100101 Firefox/71.0"})
                self.htmlQ.put(r.text)
                print(f'{self.ID} finish page-{pg}')
            except:
                pass


class ParseThread(Thread):
    def __init__(self, ID, htmlQ, resultQ):
        super().__init__()
        self.ID = ID
        self.htmlQ = htmlQ
        self.resultQ = resultQ

    def run(self):
        # 最初的时候htmlQ是空的，所以HTML_QUEUE_REAL_EMPTY需要在外部控制
        while not HTML_QUEUE_REAL_EMPTY:
            try:
                html = self.htmlQ.get(block=False)
                img_list = pat.findall(html)
                self.resultQ.put(img_list)
                print(f'{self.ID} finish {len(img_list)}: {img_list[:2]}...')
            except:
                pass


pat = re.compile(r'<img src="(.+?)"', re.S)  # re.S忽略\n
HTML_QUEUE_REAL_EMPTY = False


def main():
    pageQueue = Queue(10)
    htmlQueue = Queue()
    resultQueue = Queue()

    for i in range(10):
        pageQueue.put(i+1)

    craw_list = []
    for i in range(3):
        t = CrawThread(f'craw-{i}', pageQueue, htmlQueue)
        t.start()
        craw_list.append(t)

    parse_list = []
    for i in range(3):
        t = ParseThread(f'parse-{i}', htmlQueue, resultQueue)
        t.start()
        parse_list.append(t)

    for t in craw_list:
        t.join()
    else:
        print('finish craw html, nothing will add to htmlQueue')

    while not htmlQueue.empty():
        pass
    else:
        # htmlQueue really is empty, stop parse_thread loop
        global HTML_QUEUE_REAL_EMPTY
        HTML_QUEUE_REAL_EMPTY = True

    for t in parse_list:
        t.join()
    else:
        print('finish parse html')

    with open('result.txt', 'w') as file:
        while not resultQueue.empty():
            for img in resultQueue.get():
                file.write(img)
                file.write('\n')


if __name__ == "__main__":
    main()
```

## DFS & BFS Spider

example: 深度优先爬虫递归实现

```py
import requests
import re

headers = {'User-Agent': "Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:72.0) Gecko/20100101 Firefox/72.0"}
pat = re.compile(r'href="(thread-.+?html)"')
visited_set = set()

def scrape_urls(url):
    r = requests.get(url, headers=headers)
    return [f'https://www.lesmao.co/{short_url}' for short_url in pat.findall(r.text)]

def scrape(url, file, depth=1):
    visited_set.add(url)
    url_list = scrape_urls(url)
    # duplicate filter
    working_urls = set(url_list)-visited_set

    for url in working_urls:
        file.write(f'depth={depth}, url={url}\n')

        if depth < 2:
            scrape(url, file, depth+1)

if __name__ == "__main__":
    with open('result.txt', 'w') as file:
        scrape('https://www.lesmao.co/', file=file)
```

example: 递归+多线程+无法区分深度优先还是广度优先了

```py
import requests
import re
import threading
import time
import queue

headers = {'User-Agent': "Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:72.0) Gecko/20100101 Firefox/72.0"}
pat1 = re.compile(r'href="(thread-.+?html)"')
pat2 = re.compile(r'src="(.+?)" /></a></li>')

def save_src(src_queue):
    file=open('img_src.txt', 'w')
    while not src_queue.empty():
        src=src_queue.get()
        file.write(f'{src}\n')
        file.flush()
        time.sleep(0.05)
    else:
        file.close()

def scrape(url, depth_dict, src_queue, sem):
    current_depth = depth_dict[url]

    r = requests.get(url, headers=headers).text
    url_list = [f'https://www.lesmao.co/{short_url}' for short_url in pat1.findall(r)]
    src_list= pat2.findall(r)
    for src in src_list:
        src_queue.put(src)

    for link in url_list:
        if link not in depth_dict:
            new_depth = current_depth+1
            depth_dict[link] = new_depth
            print('  '*new_depth, link)

            if current_depth < 2: # max DEPTH = 1
                with sem:
                    threading.Thread(target=scrape, args=(link, depth_dict, src_queue, sem)).start()

if __name__ == "__main__":
    # about img src
    src_quque=queue.Queue()
    # start at 3 seconds later
    write_thread=threading.Timer(interval=3, function=save_src, args=(src_quque, ))
    write_thread.start()
    
    # about page
    start_url='https://www.lesmao.co/'
    depth_dict={start_url:0, }
    sem = threading.Semaphore(4)
    scrape(start_url, depth_dict, src_quque, sem)
```

example: 深度优先爬虫stack实现

```py
import requests
import re

headers = {
    'User-Agent': "Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:72.0) Gecko/20100101 Firefox/72.0"}
pat = re.compile(r'href="(thread-.+?html)"')

def scrape_urls(url):
    r = requests.get(url, headers=headers)
    return [f'https://www.lesmao.co/{short_url}' for short_url in pat.findall(r.text)]

url_stack = [(0, 'https://www.lesmao.co/'), ]
visited_set = set()
while len(url_stack) != 0:
    depth, url = url_stack.pop()
    print("  "*depth, url)
    url_list = scrape_urls(url)
    working_urls = set(url_list)-visited_set
    for url in working_urls:
        url_stack.append((depth+1, url))
        visited_set.add(url)
```

example: 广度优先queue实现

```py
import requests
import re
import queue

headers = {
    'User-Agent': "Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:72.0) Gecko/20100101 Firefox/72.0"}
pat = re.compile(r'href="(thread-.+?html)"')


def scrape_urls(url):
    r = requests.get(url, headers=headers)
    return [f'https://www.lesmao.co/{short_url}' for short_url in pat.findall(r.text)]


url_queue = queue.Queue()
url_queue.put((0, 'https://www.lesmao.co/'))
visited_set = set()
while not url_queue.empty():
    depth, url = url_queue.get()
    print("  "*depth, url)
    url_list = scrape_urls(url)
    working_urls = set(url_list)-visited_set
    for url in working_urls:
        url_queue.put((depth+1, url))
        visited_set.add(url)
```

example: 广度优先+多线程

```py
import requests
import re
import queue
import threading

headers = {'User-Agent': "Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:72.0) Gecko/20100101 Firefox/72.0"}
pat = re.compile(r'href="(https://www.meitulu.com/item/\d+.html)"')
# list, set, dict等container的赋值、添加是线程安全的
visited_url = set()

def func(url_queue):
    while True:
        depth, url = url_queue.get()
        if depth > 2: # 超过2层目录就停止
            break
        print("  "*depth, url)

        r = requests.get(url, headers=headers)
        url_list = pat.findall(r.text)
        working_url = set(url_list) - visited_url
        for url in working_url:
            visited_url.add(url)
            url_queue.put((depth+1, url))


if __name__ == "__main__":
    url_queue = queue.Queue()
    url_queue.put((0, 'https://www.meitulu.com/'))
    for _ in range(4):
        threading.Thread(target=func, args=(url_queue, )).start()
```

```py
import requests
import re
import time
import queue
import threading
from lxml import etree

headers = {'User-Agent': "Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:72.0) Gecko/20100101 Firefox/72.0"}
pat = re.compile(r'href="(https://www.meitulu.com/item/\d+.html)"')

def func(url_queue, visited_url, src_queue):
    while True:
        url = url_queue.get()
        print(threading.current_thread().name, url)
        r = requests.get(url, headers=headers).text

        tree=etree.HTML(r)
        src_list=tree.xpath('//center/img/@src')
        for src in src_list:
            src_queue.put(src)

        url_list = pat.findall(r)
        working_url = set(url_list)-visited_url
        for link in working_url:
            visited_url.add(link)
            url_queue.put(link)

def save_src(src_queue):
    file=open('img_src.txt', 'w')
    while not src_queue.empty():
        src=src_queue.get()
        file.write(f'{src}\n')
        file.flush()
        time.sleep(0.01)
    else:
        file.close()

if __name__ == "__main__":
    # about img src
    src_quque=queue.Queue()
    write_thread=threading.Timer(interval=3, function=save_src, args=(src_quque, ))
    write_thread.start()
    # about page
    url_queue = queue.Queue()
    url_queue.put('https://www.meitulu.com/')
    visited_url = set()

    for _ in range(4):
        threading.Thread(target=func, args=(url_queue, visited_url, src_quque)).start()
```

exmaple: 广度优先+多线程

```py
import requests
import re
import queue
import threading

headers = {
    'User-Agent': "Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:72.0) Gecko/20100101 Firefox/72.0"}
pat = re.compile(r'href="(https://www.meitulu.com/item/\d+.html)"')

class Worker(threading.Thread):
    def __init__(self, url_queue, visited_url, MAX_DEPTH=2):
        super().__init__()
        self.url_queue = url_queue
        self.visited_url = visited_url
        self.MAX_DEPTH = MAX_DEPTH

    def run(self):
        while True:
            depth, url = self.url_queue.get()
            if depth > self.MAX_DEPTH:
                break
            print('  '*depth, url)

            url_list = self.get_urls(url)
            working_url = set(url_list)-self.visited_url
            for url in working_url:
                self.visited_url.add(url)
                self.url_queue.put((depth+1, url))

    def get_urls(self, url):
        r = requests.get(url, headers=headers)
        return pat.findall(r.text)


class Crawler(object):
    def __init__(self, url, thread_num=4):
        self.url_queue = queue.Queue()
        self.url_queue.put((0, url))
        self.visited_url = set()
        self.thread_num = thread_num

    def crawl(self):
        for _ in range(self.thread_num):
            t = Worker(self.url_queue, self.visited_url)
            t.start()


if __name__ == "__main__":
    crawler = Crawler('https://www.meitulu.com/')
    crawler.crawl()
```

example: 多进程+BFS

```py
import requests
import re
import multiprocessing
from lxml import etree
import time

headers = {'User-Agent': "Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:72.0) Gecko/20100101 Firefox/72.0"}
pat = re.compile(r'href="(https://www.meitulu.com/item/\d+.html)"')

def func(url_queue, visit_url, src_queue):
    while True:
        depth, url= url_queue.get()
        if depth > 2:
            break
        print(multiprocessing.current_process().name, url)

        r = requests.get(url, headers=headers).text
        tree=etree.HTML(r)
        src_list=tree.xpath('//center/img/@src')
        for src in src_list:
            src_queue.put(src)

        url_list = pat.findall(r)
        for link in url_list:
            if link not in visit_url:
                visit_url.append(link)
                url_queue.put((depth+1, link))


def save_src(src_queue):
    time.sleep(3)
    file=open('img_src.txt', 'w')
    while not src_queue.empty():
        src=src_queue.get()
        file.write(f'{src}\n')
        file.flush()
        time.sleep(0.01)
    else:
        file.close()


if __name__ == "__main__":
    m=multiprocessing.Manager()

    src_queue=m.Queue()
    write_process=multiprocessing.Process(target=save_src, args=(src_queue,))
    write_process.start()

    start_url='https://www.meitulu.com/'
    url_queue=m.Queue()
    url_queue.put((0, start_url))
    # 多进程通过Manager().dict, Manager().list()共享数据
    visit_url=m.list()
    visit_url.append(start_url)

    process_list=[write_process, ]
    for _ in range(4):
        p=multiprocessing.Process(target=func, args=(url_queue, visit_url, src_queue))
        p.start()
        process_list.append(p)

    for p in process_list:
        # 必须join,否则AttributeError: 'ForkAwareLocal' object has no attribute 'connection'
        p.join()
```

example: 多线程Pool+BFS

```py
import requests
import re
from lxml import etree
import time
import multiprocessing

headers = {'User-Agent': "Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:72.0) Gecko/20100101 Firefox/72.0"}
pat = re.compile(r'href="(https://www.meitulu.com/item/\d+.html)"')

def save_src(src_queue):
    pass

def func(url_queue, visit_url, src_queue):
    pass

if __name__ == "__main__":
    m=multiprocessing.Manager()

    src_queue=m.Queue()
    write_process=multiprocessing.Process(target=save_src, args=(src_queue,))
    write_process.start()

    start_url='https://www.meitulu.com/'
    url_queue=m.Queue()
    url_queue.put((0, start_url))
    # 多进程通过Manager().dict, Manager().list()共享数据
    visit_url=m.list()
    visit_url.append(start_url)

    pool=multiprocessing.Pool(4, func, (url_queue, visit_url, src_queue))
    pool.close()
    pool.join()
    write_process.join()
```