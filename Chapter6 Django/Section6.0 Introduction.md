# Django Introduction

- [Django Introduction](#django-introduction)
  - [Framework](#framework)
    - [MVC](#mvc)
    - [MTV](#mtv)
  - [Django on Windows](#django-on-windows)

## Framework

example: 所有的web请求本质是socket; 所有web框架的本质是如下的程序;

```python
# DIY socket service
import socket

def handle_request(client):
    buf = client.recv(1024).decode('utf8')
    client.send("HTTP/1.1 200 OK\r\n\r\n".encode('utf8'))
    client.send(f"Hello, Grey\n{buf}".encode('utf8'))

if __name__ == '__main__':
    server = socket.socket()
    server.bind(('', 9999))
    server.listen(5)

    while True:
        connection, addr = server.accept()
        handle_request(connection)
        connection.close()
```

```python
# 只是简化上面的过程
from wsgiref.simple_server import make_server

def simple_app(environ, start_response):
    start_response('200 OK', [('Content-type', 'text/html')])
    # environ.items()客户端传递过来的所有东西
    return [f'Hello, Grey'.encode('utf8'), ]

with make_server('', 9999, simple_app) as httpd:
    httpd.serve_forever()
```

example: DIY web MVC framework
> MVC: Models, Views, Controllers

```bash
Models/
    # get data from DB
Views/
    test.html
Controllers/
    handle.py
test.py
```

```python
# test.py
from wsgiref.simple_server import make_server
from Controllers import handle

URL_DICT = {
    '/index': handle.handle_index,
    '/register': handle.handle_register,
}

def simple_app(environ, start_response):
    start_response('200 OK', [('Content-type', 'text/html')])
    path = environ['PATH_INFO']

    data = b'404'
    if path in URL_DICT:
        func = URL_DICT[path]
        data = func()

    return [data, ]

with make_server('', 9999, simple_app) as httpd:
    httpd.serve_forever()

```

```python
# handle.py
def handle_index():
    return b'index'

def handle_register():
    with open('Views/test.html', 'rb') as file:
        return file.read()
```

python的web框架有:
- flask:小型应用
- django: 大型应用
- tornado

[the Django book](http://djangobook.py3k.cn/)

### MVC

MVC框架的核心思想：**解耦**; 降低各功能模块之间的耦合性，方便变更，更容易重构代码，最大程度上实现代码的重用；根本目的是实现高可扩展性，向后兼容；
> ![](res/mvc01.png)
- m表示model，主要用于对数据库层的封装
- v表示view，用于向用户展示结果
- c表示controller，是核心，用于处理请求、获取数据、返回结果

### MTV

Django是python web框架, 属于MTV框架，本质和MVC一样，只是没有严格按照MVC的结构来写;
> ![](res/mvt01.png)
- m表示model，负责与数据库交互; 要么操作object, 要么操作list
- t表示template，负责呈现内容到浏览器
- v表示view，是核心，负责接收请求(request)、逻辑处理、获取数据、返回响应结果(response)

model中一般规范不让编写sql语句；model中维护的是一个object与Database中的一行对应；对于python, 一个list与Database的一个table对应；model直接将代码转换为sql语句(也就是ORM)

## Django on Windows

```bash
# Anaconda Prompt
# Create a environment
(base) C:\Users\Administrator>conda create --name Test
(base) C:\Users\Administrator>activate Test
(Test) C:\Users\Administrator>python --version
Python 3.6.7 :: Anaconda, Inc.
(Test) C:\Users\Administrator>conda list
# nothing

(Test) C:\Users\Administrator>conda install django
# Anaconda3\envs\Test\Scripts目录下会出现django-admin.exe，将该目录添加进入environments

(Test) C:\Users\Administrator>pip list
Package      Version
------------ ----------
certifi      2018.11.29
Django       2.1.5
pip          18.1
pytz         2018.7
setuptools   40.6.3
wheel        0.32.3
wincertstore 0.2

(Test) C:\Users\Administrator>pip freeze
certifi==2018.11.29
Django==2.1.5
pytz==2018.7
wincertstore==0.2

# start project
(Test) C:\Users\Administrator>django-admin startproject mysite
(Test) C:\Users\Administrator>tree /f mysite
C:\USERS\ADMINISTRATOR\MYSITE
│  manage.py
└─mysite
        settings.py
        urls.py
        wsgi.py
        __init__.py

# running server
(Test) C:\Users\Administrator>cd mysite
(Test) C:\Users\Administrator\mysite>python manage.py runserver
Django version 2.1.5, using settings 'mysite.settings'
Starting development server at http://127.0.0.1:8000/
Quit the server with CTRL-BREAK.
```

example: simple modify easy mysite project

```bash
│ manage.py
| Views
|       v1.py
└─mysite
        settings.py
        urls.py
        wsgi.py
        __init__.py
```

```python
# v1.py
from django.shortcuts import HttpResponse

def index(request):
    return HttpResponse('Hello')
```

```python
# urls.py
from django.contrib import admin
from django.urls import path

from Views import v1

urlpatterns = [
    path('admin/', admin.site.urls),
    path('index/', v1.index),
]
```

