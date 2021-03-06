# Coroutine

- [Coroutine](#coroutine)
  - [Introduction](#introduction)
  - [greenlet](#greenlet)
  - [gevent](#gevent)
  - [event driven & asyncIO](#event-driven--asyncio)
  - [IO模式](#io%E6%A8%A1%E5%BC%8F)
  - [IO Multiplexing](#io-multiplexing)
  - [RabbitMQ](#rabbitmq)
  - [RabbitMQ RPC](#rabbitmq-rpc)
  - [Cache](#cache)

## Introduction

> 协程是一种用户态的轻量级线程;  
> Coroutine是用户控制的，CPU并不知道它的存在，CPU只知道线程

协程拥有自己的寄存器上下文和栈。协程调度切换时，将寄存器上下文和栈保存到其他地方，在切回来的时候，恢复先前保存的寄存器上下文和栈。
> 线程是直接使用CPU的寄存器来切换上下文

Coroutine优点：
- 无需线程上下文切换的开销：不同函数的来回切换，没有新建线程的开销
- 无需原子操作锁定及同步的开销: 因为Coroutine是在单线程中，不需要mutex
  - "原子操作(atomic operation)是不需要synchronized"，所谓原子操作是指不会被线程调度机制打断的操作；这种操作一旦开始，就一直运行到结束，中间不会有任何 context switch (切换到另一个线程)。原子操作可以是一个步骤，也可以是多个操作步骤，但是其顺序是不可以被打乱，或者切割掉只执行部分。视作整体是原子性的核心。
- 方便切换控制流，简化编程模型
- 高并发+高扩展性+低成本：一个CPU支持上万的协程都不是问题。所以很适合用于高并发处理。

Coroutine缺点：
- 无法利用多核资源：协程的本质是个单线程,所以不能用于多核,协程需要和进程配合才能运行在多CPU上.当然我们日常所编写的绝大部分应用都没有这个必要，除非是cpu密集型应用。
- 进行阻塞(Blocking)操作(如IO时)会阻塞掉整个程序

nginx默认是单线程，一个线程支持10k的并发，也是用的Coroutine；配置成多个线程，可以支持更多并发。

```python
# 单线程并发，不是并行; 这种就是异步IO的雏形
import time

def consumer(name):
    print(f"{name}准备吃包子啦!")
    while True:
        baozi = yield
        print(f"包子[{baozi}]来了,被[{name}]吃了!")

def producer(name):
    c1 = consumer('A') # type is generator
    c2 = consumer('B') # type is generator
    c1.__next__()
    c2.__next__()
    print("老子开始准备做包子啦!")
    for i in range(5):
        time.sleep(1)
        print("做了1个包子， 分两半!")
        c1.send(i)
        c2.send(i)

producer("alex")
```

> Coroutine最核心的地方：碰到IO操作就切换  
> Problem: IO操作执行完毕如何再切换回去，所以需要检测IO操作的执行

Coroutine特点：
- 必须在只有一个单线程里实现并发
- 修改共享数据不需加锁
- 用户程序里自己保存多个控制流的上下文栈
- 一个协程遇到IO操作自动切换到其它协程，因此`yield`不是标准的Coroutine

## greenlet

greenlet是一个用C实现的协程模块，相比与python自带的yield，它可以使你在任意函数之间随意切换，而不需把这个函数先声明为generator; 然而因为只是切换，没有碰到IO自动切换，并不是标准的Coroutine

```python
from greenlet import greenlet

def test1():
    print(12)
    gr2.switch()
    print(34)
    gr2.switch()

def test2():
    print(56)
    gr1.switch()
    print(78)

gr1 = greenlet(test1)
gr2 = greenlet(test2)
gr1.switch()
```

```bash
12
56
34
78
```

## gevent

gevent是对greenlet进一步封装得到的可以自动IO切换的Coroutine


example: gevent.sleep()导致的自动切换

```python
import gevent

def func1():
    print('func1 begins')
    gevent.sleep(2)
    print('func1 ends')

def func2():
    print('func2 begins')
    gevent.sleep(1)
    print('func2 ends')

def func3():
    print('func3 begins')
    gevent.sleep(0) # 只是为了触发切换
    print('func3 ends')

task_list=[gevent.spawn(func1), gevent.spawn(func2), gevent.spawn(func3)]
gevent.joinall(task_list)
```

```bash
func1 begins
func2 begins
func3 begins
func3 ends # 这里卡住1s
func2 ends # 这里卡住<1s
func1 ends
```

example: IO导致的自动切换
> 首先需要gevent检测到IO操作, `monkey.patch_all()`将程序的所有IO标记，相当于`gevent.sleep()`

```python
import time
import gevent
from gevent import monkey

# 标记所有IO操作，为了实现碰到IO自动gevent.sleep()
monkey.patch_all()

import requests

def downloader(url):
    r=requests.get(url).content
    print(f'Recv {len(r)}bytes from {url}')

url_list=['https://m.huxiu.com/', 'http://www.10tiao.com/', 'https://www.423down.com/']

# Async
t1=time.clock()
task_list=[gevent.spawn(downloader, url) for url in url_list]
gevent.joinall(task_list)
t2=time.clock()
print('async time: ', t2-t1)

# Sync
for url in url_list:
    downloader(url)
print('sync time: ', time.clock()-t2)
```

example: gevent server

```python
import gevent
# gevent重写的socket
from gevent import socket

client_dict = {}

def handle_request(sock):
    while True:
        # 当这个函数耗时的时候，就切换给tcp_server()
        recv_data = sock.recv(1024).decode('gbk')
        if not recv_data:
            print(f'{client_dict[sock]} disconnected!')
            sock.close()
            break
        print(f'>>{client_dict[sock]}:{recv_data}')
        sock.send((recv_data+' checked').encode('gbk'))


def tcp_server(port):
    server_socket = socket.socket()
    server_socket.bind(('', port))
    server_socket.listen(10)
    while True:
        # 重写了的socket, 一旦耗时就切换，如果只有一个协程，就不切换，死等
        client_socket, client_info = server_socket.accept()
        client_dict[client_socket] = client_info
        print(f'{client_info} connected!')
        # 如果是socketserver模块，这里是新建线程
        gevent.spawn(handle_request, client_socket)


if __name__ == '__main__':
    tcp_server(9999)
```

```python
# client
import socket
import threading

def client():
    s = socket.socket()
    s.connect(("localhost", 9999))
    for i in range(3):
        s.send(f"hello {i}".encode('utf8'))
        data = s.recv(1024)
        print(f"{threading.get_ident()} Recv", data.decode('utf8'))
    else:
        s.close()

for _ in range(10): # 并发10个client
    threading.Thread(target=client).start()
```

## event driven & asyncIO

比如其中一个Coroutine切换到其他Coroutine, 如何切换回来？
> 采用**事件驱动模型**实现, [Details](http://www.cnblogs.com/alex3714/articles/5248247.html)  

事件驱动模型：很类似**生产者-消费者模型**
1. 有一个事件(消息)队列；
2. 比如鼠标按下时，往这个队列中增加一个点击事件(消息)；
3. 事件(消息)一般都各自保存各自的回调函数指针；
4. 有个循环，不断从队列取出事件，根据不同的事件，调用不同的的回调函数

> 回调分为：同步回调，异步回调；AsyncIO当然是采用的异步回调。[Details](https://www.zhihu.com/question/19801131/answer/27459821)

常见的编程范式：
- 多进程编程
- 多线程编程
- 事件驱动编程: 包含一个事件循环，当外部事件发生时使用**回调机制**来触发相应的处理

适合事件驱动模型的场景:
- 多任务
- 任务之间高度独立，不需要通信
- 任务中存在阻塞
> 所以事件驱动模型很适合网络应用程序

具体到Coroutine的AsyncIO过程:
> 假设有CoroutineA, CoroutineB。CoroutineA触发IO操作事件，将该事件加入消息队列，然后切换到CoroutineB，CoroutineB触发IO操作事件，也将该事件加入消息队列；等待信号。

> OS从消息队列取出事件和回掉函数指针，处理完IO操作，并调用回调函数，给主程序一个信号；主程序根据信号切换到对应的Coroutine

> IO操作都是OS完成的，不适用消息队列就会卡住主程序，使用消息队列就不会卡住

基础知识：

- 操作系统的核心是内核，独立于普通的应用程序，可以访问受保护的内存空间，也有访问底层硬件设备的所有权限。为了保证用户进程不能直接操作内核，保证内核的安全，操心系统将虚拟空间划分为两部分，一部分为内核空间，一部分为用户空间。
  > 用户访问网卡实际上是要通过OS的接口才能访问

- 进程的阻塞是进程自身的一种主动行为，也因此只有处于运行态的进程(获得CPU)，才可能将其转为阻塞状态。当进程进入阻塞状态，是不占用CPU资源的。

- 内核为每个进程维护一个打开文件的记录表，文件描述符就是一个index，根据这个index可以在记录表中找到文件对象。当程序打开一个现有文件或者创建一个新文件时，内核向进程返回一个文件描述符。
  > 比如文件读取过程：程序拿着index给OS，然后OS从文件记录表中将文件对象从内核空间拷贝到用户空间

- Linux会将 I/O 的数据缓存在文件系统的页缓存(page cache)中，也就是说，数据会先被拷贝到操作系统内核的缓冲区中，然后才会从操作系统内核的缓冲区拷贝到应用程序的地址空间。
  > 比如接收到的socket数据，先从网卡到了内核空间缓冲区，然后才拷贝到用户空间  
  > 缓存IO缺点：数据在传输过程中需要在应用程序地址空间和内核进行多次数据拷贝操作，这些数据拷贝操作所带来的 CPU 以及内存开销是非常大的。

## IO模式

对于一次IO访问(比如read)，数据会先被拷贝到操作系统内核的缓冲区中，然后才会从操作系统内核的缓冲区拷贝到应用程序的地址空间。也就是两个阶段：
1. 等待数据准备 (Waiting for the data to be ready)
2. 将数据从内核拷贝到进程中 (Copying the data from the kernel to the process)

因为这两个阶段，linux系统产生了下面五种网络模式的方案。[Details](https://notes.shichao.io/unp/ch6/)
- 阻塞 I/O(blocking IO)
- 非阻塞 I/O(nonblocking IO)
- I/O 多路复用( IO multiplexing)
- 信号驱动 I/O( signal driven IO)
- 异步 I/O(asynchronous IO)

## IO Multiplexing

select，poll，epoll都是IO多路复用的机制。I/O多路复用就是通过一种机制，一个进程可以监视多个描述符，一旦某个描述符就绪（一般是读就绪或者写就绪），能够通知程序进行相应的读写操作。
> `select`, `poll`, `epoll`作用就是检测, [Details](http://www.cnblogs.com/alex3714/p/4372426.html)；从Linux2.6之后用的都是epoll

nginx, tornado, 异步网络框架twisted都是使用epoll; windows只支持select()
- select: 只告知N个中有n个就绪，但是具体是哪n个需要扫描N个；最大文件描述符数目为1024
- poll: 只告知N个中有n个就绪，但是具体是哪n个需要扫描N个；没有最大文件描述符数目限制
- epoll: 直接告知N个中的哪n个就绪；没有最大检测数目的限制；

IO多路复用没有AsncIO厉害，但是IO多路复用使用场景更多；AsyncIO的使用场景较少、实现复杂，而且内核对异步IO的支持也不太好。
> 实际上nginx, tornado, twisted只是叫做异步IO，本质其实是IO多路复用。

Python3有一个AsyncIO模块: asyncio

服务器开发中可能用到AsyncIO, 比如几十万玩家的游戏服务器，异步爬虫等。

example: 用`select`实现socket server

```python
import select
import socket

server = socket.socket()
server.bind(('', 9999))
server.listen(10)
server.setblocking(False)  # recv(), accept()不会阻塞;这样才能并发

rlist = [server, ] # waiting until ready reading
wlist = [] # waiting until ready writing

while True:
    # select()会卡住
    readable, writable, exceptional=select.select(rlist, wlist, rlist)
    # print(readable, writable, exceptional)
    for r in readable:
        # 来新连接，server变成活动的；客户端发消息，对应的socket变成活动的
        if r is server:
            client_sock, addr=server.accept()
            print(f'{addr} connected!')
            rlist.append(client_sock)
        else:
            data=r.recv(1024).decode('gbk')
            
            if not data:
                print('client disconnected')
                r.close()
                rlist.remove(r)
                continue

            print(f'>>:{data}')
            r.send(data.upper().encode('gbk'))
```

example: `select`分开readable, writable, exceptional
> [details](http://www.cnblogs.com/alex3714/p/4372426.html)

```python
import select
import socket
import queue

server = socket.socket()
server.bind(('', 9999))
server.listen(10)
server.setblocking(False)

rlist = [server, ]
wlist = []

msg_dict = {}

while True:
    readable, writable, exceptional = select.select(rlist, wlist, rlist)
    print(readable, writable, exceptional)
    for r in readable:
        if r is server:
            client_sock, addr = server.accept()
            print(f'{addr} connected!')
            rlist.append(client_sock)
            # 每一个client给一个待发送消息的队列
            msg_dict[client_sock] = queue.Queue()
        else:
            data = r.recv(1024).decode('gbk')

            if not data:
                print('client disconnected')
                # r.close()
                rlist.remove(r)
                continue

            print(f'>>:{data}')
            msg_dict[r].put(data)
            # 放到wlist再发，再经过writable发送
            wlist.append(r)

    for w in writable:
        data = msg_dict[w].get()
        w.send(data.upper().encode('gbk'))
        wlist.remove(w)  # 下次循环writable不再有处理完的socket

    for e in exceptional:
        rlist.remove(e) # 出错的socket一定在rlist
        if e in wlist:
            wlist.remove(e)
        del msg_dict[e]
```

example: `epoll`实现的socket server, 过于底层
> [Details](http://www.cnblogs.com/alex3714/articles/5876749.html)

example: 有封装好的`selectors`

```python
import selectors
import socket

sel = selectors.DefaultSelector() 
# 默认使用epoll;如果OS不支持epoll就会使用select ,比如windows

def accept(server, mask):
    conn, addr = server.accept()  # Should be ready
    print('accepted', conn, 'from', addr)
    conn.setblocking(False)
    sel.register(conn, selectors.EVENT_READ, read)

def read(conn, mask):
    data = conn.recv(1000)  # Should be ready
    if data:
        print('echoing', repr(data), 'to', conn)
        conn.send(data)  # Hope it won't block
    else:
        print('closing', conn)
        sel.unregister(conn)
        conn.close()

server = socket.socket()
server.bind(('', 9999))
server.listen(10)
server.setblocking(False)
sel.register(server, selectors.EVENT_READ, accept)

while True:
    events = sel.select() # 有活动就不会阻塞
    for key, mask in events:
        callback = key.data
        callback(key.fileobj, mask)
        # fileobj就是socket实例
```

example: selectors实现ftp文件上传、下载，即两个client同时在下载、上传

summary:

coroutine与select, poll, epoll都是单线程;

epoll在linux底层通过libevent.so实现; gevent在linux底层也是通过libevent.so实现;

游戏后端有些用twisted这个异步网络框架，一般用不到

## RabbitMQ

[RabbitMQ](https://github.com/rabbitmq/rabbitmq-server/releases): Rabit Message Queue，是使用Erlang开发的，[RabbitMQ原理](https://www.jianshu.com/p/ff665a17473a), 默认端口5671, 5672

queue分类:
- 线程queue: python普通的queue, 多个线程之间
- 进程queue: multiprocessing中的queue, 父子进程之间或者同父的不同子进程之间通信
- RabbitMQ: 两个陌生进程之间通信需要中间代理broker，也就是RabbitMQ。比如不同的python程序之间，java与python程序之间通信。

![](res/RabbitMQ01.png)

陌生进程通信:
- socket
  - 两个进程之间直接socket
  - 两个进程分别与中间代理建立socket
- 一个进程写入磁盘文件，另一个读取磁盘文件

![](res/RabbitMQ02.png)

RabbitMQ+Python: [devtools](https://www.rabbitmq.com/devtools.html), [tutorials](https://www.rabbitmq.com/getstarted.html)
> pip install pika

RabbitMQ configuration:
- `sudo apt install rabbitmq-server`
- 配置管理用户
  - `sudo rabbitmq-plugins enable rabbitmq_management`
  - 添加admin用户: `sudo rabbitmqctl add_user admin admin`, `sudo rabbitmqctl set_user_tags admin administrator`
  - 登录`http://127.0.0.1:15672`
- 配置remote access(为了安全一般不这么干)
  - `vim /etc/rabbitmq/rabbitmq.config`添加`[{rabbit, [{loopback_users, []}]}].`
  - `sudo service rabbbitmq-server restart`

example: Producer

```python
import pika

# 建立socket
connection = pika.BlockingConnection(pika.ConnectionParameters('xx.xx.xx.xx'))
# 声明一个管道
channel = connection.channel()
# 声明queue，给queue一个名字
channel.queue_declare(queue='hello')

channel.basic_publish(exchange='', routing_key='hello', body='Hello World!')
print(" [x] Sent 'Hello World!'")
connection.close()
```

example: Consumer

```python
import pika

def callback(ch, method, properties, body):
    # ch就是channel对象; body是内容
    print(" [x] Received %r" % body)

connection = pika.BlockingConnection(pika.ConnectionParameters('xx.xx.xx.xx'))
channel = connection.channel()
# 因为不清楚Consumer,Producer哪个先启动，所以重新声明了queue
channel.queue_declare(queue='hello')
 
channel.basic_consume('hello', callback)

print(' [*] Waiting for messages. To exit press CTRL+C')
channel.start_consuming()  # 一直收消息，没有就卡住
```

```bash
# Consumer output
[*] Waiting for messages. To exit press CTRL+C
[x] Received b'Hello World!'
[x] Received b'Hello World!'
[x] Received b'Hello World!'
[x] Received b'Hello World!'
```

RabbitMQ消费分发模式
- 1 Producer N Consumers: 生产者生产的消息被轮询的分别发给不同消费者
- 1 Producer N Consumers: 当`basic_consumer()`默认值`auto_ack=False`, 如果Consumer没有处理完消息，而socket断了，那么RabbitMQ会将消息给下一个Consumer，一直轮询下去，如果所有的Consumer都没有处理完这个消息而断了，那么消息丢失

```python
# 演示Consumer没有处理完消息
def callback(ch, method, properties, body):
    print('******')
    time.sleep(10)
    print(" [x] Received %r" % body)
```

Linux查看Rabbitmq中的queue中有多少消息: `sudo rabbitmqctl list_queues`; 删除queue: `rabbitmqadmin delete queue name='hello'`

```bash
$ sudo rabbitmqctl list_queues
Listing queues
# 消息数量为1
hello	1
```

```python
def callback(ch, method, properties, body):
    print('******')
    time.sleep(10)
    print(" [x] Received %r" % body)
    # 手动确认，保证sudo rabbitmqctl list_queues中的queue消息数量为0
    ch.basic_ack(delivery_tag=method.delivery_tag)
```

example: list_queue里面里面的queue还有消息，但是server重启
> 一般情况，消息在server的内存中，server挂了，消息就丢了

step1: 让队列持久化:
> Producer, Consumer都修改`channel.queue_declare(queue='hello', durable=True)`

step2: 让消息持久化
> Producer修改`channel.basic_publish(exchange='', routing_key='hello', body='Hello World!', properties=pika.BasicProperties(delivery_mode=2))`

example: RabbitMQ负载均衡
> server给Consumer发消息的时候，先检查现在还有多少消息: 如果queue中的消息>1，server就不给consumer发消息; 如果没有消息，server就给consumer发消息  
> Producer在`basic_consumer()`添加`channel.basic_qos(prefetch_count=1)`

演示: consumerA `time.sleep(10)`, consumerB `time.sleep(30)`; 如果有源源不断的message, consumerB没有处理完，就会反复给consumerA发; 而不是默认的轮询的方式

example: RabbitMQ不是轮询地发消息，而是所有人收到同样的消息

RabbitMQ的exchange模式
- fanout: 所有bind到此exchange的queue都可以接收消息
- direct: 通过routingKey和exchange决定的那个唯一的queue可以接收消息
- topic:所有符合routingKey(此时可以是一个表达式)的routingKey所bind的queue可以接收消息
- headers: 通过headers 来决定把消息发给哪些queue

[RabbitMQ mode](http://www.cnblogs.com/alex3714/articles/5248247.html)

example: Broadcast
> server为每个consumer生成临时queue, 绑定到某个exchange, 然后producer将消息发到所有临时queue就实现了广播  
> 广播的临时queue中不保留消息，consumer错过就错过了，和听广播类似的原理

```python
# Producer
import sys
import pika

# 建立socket
connection = pika.BlockingConnection(pika.ConnectionParameters('xx.xx.xx.xx'))
# 声明一个管道
channel = connection.channel()
# 广播不需要queue_declare, 随机生成queue
# 声明exchange，给exchange一个名字
channel.exchange_declare(exchange='ex1', exchange_type='fanout')
message = ' '.join(sys.argv[1:]) or "info: Hello World!"

channel.basic_publish(exchange='ex1', routing_key='', body=message)
print(f" [x] Sent '{message}'")
connection.close()
```

```python
# Consumer
import pika


def callback(ch, method, properties, body):
    print('******')
    print(" [x] Received %r" % body)
    ch.basic_ack(delivery_tag=method.delivery_tag)

connection = pika.BlockingConnection(pika.ConnectionParameters('xx.xx.xx.xx'))
channel = connection.channel()
channel.exchange_declare(exchange='ex1', exchange_type='fanout')
# exclusive=True会在使用此queue的消费者断开后,自动将queue删除
result = channel.queue_declare(queue='', exclusive=True)
queue_name=result.method.queue

channel.queue_bind(queue_name, exchange='ex1')
print(' [*] Waiting for ex1. To exit press CTRL+C')

channel.basic_consume(queue_name, callback)
channel.start_consuming()  # 一直收消息，没有就卡住
```

example: direct mode, 具有过滤作用

```python
# Producer
import sys
import pika

connection = pika.BlockingConnection(pika.ConnectionParameters('xx.xx.xx.xx'))
channel = connection.channel()
channel.exchange_declare(exchange='ex2', exchange_type='direct')
severity = sys.argv[1] if len(sys.argv) > 1 else 'info'
message = ' '.join(sys.argv[2:]) or "Hello World!"

channel.basic_publish(exchange='ex2', routing_key=severity, body=message)
print(f" [x] Sent {severity}:'{message}'")
connection.close()
```

```python
# Consumer
import sys
import pika


def callback(ch, method, properties, body):
    print('******')
    print(" [x] Received %r" % body)
    ch.basic_ack(delivery_tag=method.delivery_tag)


connection = pika.BlockingConnection(pika.ConnectionParameters('xx.xx.xx.xx'))
channel = connection.channel()
channel.exchange_declare(exchange='ex2', exchange_type='direct')
result = channel.queue_declare(queue='', exclusive=True)
queue_name = result.method.queue

severities = sys.argv[1:]
if not severities:
    sys.stderr.write(f"Usage: {sys.argv[0]} [info] [warning] [error]\n")
    sys.exit(1)

for severity in severities:
    channel.queue_bind(queue_name, exchange='ex2', routing_key=severity)

print(' [*] Waiting for ex2. To exit press CTRL+C')

channel.basic_consume(queue_name, callback)
channel.start_consuming()
```

example: detail filter
> 可以使用关键词来过滤

```python
# Producer
import sys
import pika

connection = pika.BlockingConnection(pika.ConnectionParameters('xx.xx.xx.xx'))
channel = connection.channel()
channel.exchange_declare(exchange='ex3', exchange_type='topic')
routing_key = sys.argv[1] if len(sys.argv) > 1 else 'anonymous.info'
message = ' '.join(sys.argv[2:]) or "Hello World!"

channel.basic_publish(exchange='ex3', routing_key=routing_key, body=message)
print(f" [x] Sent {routing_key}:'{message}'")
connection.close()
```


```python
# Consumer
import sys
import pika

def callback(ch, method, properties, body):
    print('******')
    print(" [x] Received %r" % body)
    ch.basic_ack(delivery_tag=method.delivery_tag)

connection = pika.BlockingConnection(pika.ConnectionParameters('xx.xx.xx.xx'))
channel = connection.channel()
channel.exchange_declare(exchange='ex3', exchange_type='topic')
result = channel.queue_declare(queue='', exclusive=True)
queue_name = result.method.queue

binding_keys = sys.argv[1:]
if not binding_keys:
    sys.stderr.write(f"Usage: {sys.argv[0]} [binding key]\n")
    sys.exit(1)

for binding_key in binding_keys:
    channel.queue_bind(queue_name, exchange='ex3', routing_key=binding_key)

print(' [*] Waiting for ex3. To exit press CTRL+C')

channel.basic_consume(queue_name, callback)
channel.start_consuming()
```

```bash
# Producer
$ python producer.py
[x] Sent anonymous.info:'Hello World!'

# Consumer
$ python consumer.py *.info
[*] Waiting for ex3. To exit press CTRL+C
******
[x] Received b'Hello World!'

# Consumer receive all
$ python consumer.py #
```

OpenStack默认使用RabbitMQ, SaltStack可以从ZeroMQ修改为RabbitMQ

## RabbitMQ RPC

RPC:Remote Procedure Call
> client发一条指令，server执行，返回client结果  
> 实现: client和server既是Producer又是Consumer, 并且有两个queue

![](res/rpc01.png)

```python
# Client
import time
import pika
import uuid

class FibonacciRpcClient(object):
    def __init__(self):
        self.connection = pika.BlockingConnection(
            pika.ConnectionParameters('xx.xx.xx.xx'))
        self.channel = self.connection.channel()
        result = self.channel.queue_declare(queue='', exclusive=True)
        self.callback_queue = result.method.queue
        # 这里只是register
        self.channel.basic_consume(self.callback_queue, self.on_response)

    def on_response(self, ch, method, props, body):
        # 高并发的时候区分不同的request
        if self.corr_id == props.correlation_id:
            self.response = body

    def call(self, n):
        self.response = None
        self.corr_id = str(uuid.uuid4())
        self.channel.basic_publish(exchange='', routing_key='rpc_queue', properties=pika.BasicProperties(
            reply_to=self.callback_queue, correlation_id=self.corr_id,), body=str(n))
        while self.response is None:
            # 非阻塞的start_consuming(), 没有消息直接通过，有消息触发on_response
            self.connection.process_data_events()
            print('no msg....')
            time.sleep(1)
        return int(self.response)

fibonacci_rpc = FibonacciRpcClient()
print(" [x] Requesting fib(30)")
response = fibonacci_rpc.call(30)
print(f" [.] Got {response}")
```

```python
# Server
import pika


def fib(n):
    if n == 0:
        return 0
    elif n == 1:
        return 1
    else:
        return fib(n-1) + fib(n-2)


def on_request(ch, method, props, body):
    n = int(body)
    print(f" [.] fib({n})")
    response = fib(n)

    ch.basic_publish(exchange='', routing_key=props.reply_to, properties=pika.BasicProperties(
        correlation_id=props.correlation_id), body=str(response))
    ch.basic_ack(delivery_tag=method.delivery_tag)


conn = pika.BlockingConnection(pika.ConnectionParameters('xx.xx.xx.xx'))
channel = conn.channel()
channel.queue_declare(queue='rpc_queue')
channel.basic_qos(prefetch_count=1)
channel.basic_consume('rpc_queue', on_request)

print(" [x] Awaiting RPC requests")
channel.start_consuming()
```

work: 一个client发送命令给多个server, 然后server返回给client;
> 使用`direct`, `topic`模式

## Cache

![](res/cache01.png)
> Redis默认存内存，配置后可以存在磁盘  
> Memcached比较轻量级

Redis是通过异步IO epoll实现的高并发: 单线程, get/set: 80000 requests/second

```bash
$ redis-cli
127.0.0.1:6379> set name grey
OK
127.0.0.1:6379> keys *
1) "name"
127.0.0.1:6379> get name
"grey"
# 2s expire
127.0.0.1:6379> set age 22 ex 2
OK
127.0.0.1:6379> get age
(nil)
```

example: connect redis with python

```python
import redis

r=redis.Redis(host='xx.xx.xx.xx')
r.set('name', 'moriaty')
print(r.get('name')) # b'moriaty'
```

example: redis pool

```python
import redis

po = redis.ConnectionPool(host='xx.xx.xx.xx')
r = redis.Redis(connection_pool=po)
r.set('age', 22)
print(r.get('age')) # b'22'
````

[Redis Operation](http://www.cnblogs.com/alex3714/articles/6217453.html)
- `string` operation
- `hash` operation
- `list` operation
- `set` operation
- `sort set` operation

MySQL数据库超过500w条就会变慢，需要分表；假设从14亿数据统计在线的用户数，如下操作

```bash
# 比如id为1000和55的用户上线，那么
setbit N 1000 1
setbit N 55 1
bitcount N # 2
```

```bash
# 或者当用户登录的时候
incr N
incr N
incr N
# 当用户下线的时候
decr N
get N 2
```

`brpoplpush`: redis有timeout的时候，可以用一个进程往redis的list中放入数据，另一个进程从该list中get数据并放入自己的redis的list
> 类似生产者-消费者

example: pipeline

```python
import time
import redis

po = redis.ConnectionPool(host='xx.xx.xx.xx')
r = redis.Redis(connection_pool=po)

pipe = r.pipeline(transaction=True)

pipe.set('name', 'grey')
time.sleep(5)
pipe.set('age', 22)

# 最后一次性执行
pipe.execute()
```

example: publish-subscribe
> publish-subscribe本质就是广播

```python
# RedisHelper
import redis


class RedisHelper:
    def __init__(self):
        self.__conn = redis.Redis(host='xx.xx.xx.xx')
        self.chan_sub = 'fm104.5'
        self.chan_pub = 'fm104.5'

    def public(self, msg):
        self.__conn.publish(self.chan_pub, msg)
        return True

    def subscribe(self):
        pub = self.__conn.pubsub()
        pub.subscribe(self.chan_sub)
        pub.parse_response()
        return pub
```

```python
# Publisher
from MyRedisHelper import RedisHelper

obj=RedisHelper()
obj.public('hello')
```

```python
# Subscriber
from MyRedisHelper import RedisHelper

obj=RedisHelper()
subscriber=obj.subscribe()

while True:
    msg=subscriber.parse_response()
    print(msg)
```