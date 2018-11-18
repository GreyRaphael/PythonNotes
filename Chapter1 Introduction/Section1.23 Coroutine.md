# Coroutine

- [Coroutine](#coroutine)
    - [Introduction](#introduction)
    - [greenlet](#greenlet)
    - [gevent](#gevent)
    - [event driven & asyncIO](#event-driven--asyncio)

## Introduction

> 协程是一种用户态的轻量级线程;  
> Coroutine是用户控制的，CPU并不知道它的存在，CPU只知道线程

协程拥有自己的寄存器上下文和栈。协程调度切换时，将寄存器上下文和栈保存到其他地方，在切回来的时候，恢复先前保存的寄存器上下文和栈。
> 线程是直接使用CPU的寄存器来切换上下文

Coroutine优点：
- 无需线程上下文切换的开销：不同函数的来回切换，没有新建线程的开销
- 无需原子操作锁定及同步的开销: 因为Coroutine是在单线程中，不需要mutex
  - "原子操作(atomic operation)是不需要synchronized"，所谓原子操作是指不会被线程调度机制打断的操作；这种操作一旦开始，就一直运行到结束，中间不会有任何 context switch （切换到另一个线程）。原子操作可以是一个步骤，也可以是多个操作步骤，但是其顺序是不可以被打乱，或者切割掉只执行部分。视作整体是原子性的核心。
- 方便切换控制流，简化编程模型
- 高并发+高扩展性+低成本：一个CPU支持上万的协程都不是问题。所以很适合用于高并发处理。

Coroutine缺点：
- 无法利用多核资源：协程的本质是个单线程,所以不能用于多核,协程需要和进程配合才能运行在多CPU上.当然我们日常所编写的绝大部分应用都没有这个必要，除非是cpu密集型应用。
- 进行阻塞（Blocking）操作（如IO时）会阻塞掉整个程序

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

