# Python Module

<!-- TOC -->

- [Python Module](#python-module)
    - [`sys` & `os` module](#sys--os-module)
    - [`getpass` module](#getpass-module)
    - [module storage](#module-storage)
        - [diy module](#diy-module)
    - [`__all__`](#__all__)
    - [package](#package)
        - [标准格式](#标准格式)
    - [制作package并发布](#制作package并发布)
        - [method 1：手动](#method-1手动)
        - [method2: pycharm 制作](#method2-pycharm-制作)
    - [给程序传参数](#给程序传参数)
    - [循环导入](#循环导入)
    - [standard library](#standard-library)
        - [`hashlib`](#hashlib)
    - [extend library](#extend-library)
    - [pdb](#pdb)

<!-- /TOC -->

## `sys` & `os` module

```python
import sys
# python env
print(sys.path)
print(sys.argv)
```

```python
import os

res=os.system('dir') # 执行命令不保存结果
print(res) # 0, 表示成功执行

# 显示结果
os.popen('dir').read()
```

## `getpass` module

```python
import getpass

upwd = getpass.getpass('enter your password:')
print(upwd)
```

## module storage

```python
#import的顺序按照sys.path的顺序来搜索module，也可以自己添加
import sys

print(sys.path)#是一个list
sys.path.append('/home')
```

```bash
#output
['',#表示当前路径
 'D:\\ProgrammingTools\\Anaconda3\\python36.zip',
 'D:\\ProgrammingTools\\Anaconda3\\DLLs',
 'D:\\ProgrammingTools\\Anaconda3\\lib',
 'D:\\ProgrammingTools\\Anaconda3',
 'D:\\ProgrammingTools\\Anaconda3\\lib\\site-packages',
 'D:\\ProgrammingTools\\Anaconda3\\lib\\site-packages\\win32',
 'D:\\ProgrammingTools\\Anaconda3\\lib\\site-packages\\win32\\lib',
 'D:\\ProgrammingTools\\Anaconda3\\lib\\site-packages\\Pythonwin',
 'D:\\ProgrammingTools\\Anaconda3\\lib\\site-packages\\IPython\\extensions',
 'C:\\Users\\Administrator\\.ipython']
```

重新导入，如果已经import了一个module，程序没有退出，又去修改了这个module，那么如果要用修改后的东西

- 重启程序
- reload():

```python
#imp is deprecated
from imp import *

reload(sys)
```

```python
import os

osPath=os.__file__ #str
# print(osPath,type(osPath))
pos=osPath.rfind('\\')
path=osPath[:pos]
print(path)#D:\ProgrammingTools\Anaconda3\lib
print(os.listdir(path))#random.py, os.py都在这个目录下
```

所谓的`import random`其实就是导入`random.py`文件

```bash
sudo pip install pygame
sudo pip3 install pygame
```

### diy module

先搜索当前路径，然后去系统路径搜索

同一个目录下面：

```bash
./
    __pycache__
        mySend.cpython-36.pyc #这个文件是python编译器将import的先编译得到的二进制，方便反复修改过程中的反复编译，简称为“字节码”
    mySend.py
    test.py
```

```python
#mySend.py
def SendMsg():
    print("function in mySend.py")
def Test():
    print("another function")
```

```python
#test.py
import mySend

mySend.SendMsg()
```

```python
#test.py
from mySend import SendMsg,Test
# from mySend import * #尽量少用这个，因为引入多个模块可能函数名冲突，后面import会代替前面导入的

Test()
SendMsg()
```

```python
#别名
import mySend as myS #别名

myS.SendMsg()
```

当你`import`的时候，会把模块从头到尾执行一边，所以测试玩之后，要注释，或者用`__main__`来调控

```python
#mySend.py
def SendMsg():
    print("function in mySend.py")
def Test():
    print("another function")

#for test
if __name__ == '__main__':#被引用的时候为模块名，__name__==mySend
    SendMsg()
    Test()
```

然后被import也没有关系了；

## `__all__`

It has two purposes:

1. Anybody who reads the source will know what the exposed public API is. It doesn't prevent them from poking around in private declarations, but does provide a good warning not to.
2. When using `from mod import *`, only names listed in `__all__` will be imported. This is not as important, in my opinion, because importing everything is a really bad idea.

```python
#mySend.py
#全局变量, 其中的才能使用，其他的函数不能给别人使用
__all__=["SendMsg",]

def SendMsg():
    print("function in mySend.py")
def Test():
    print("another function")

#for test
if __name__ == '__main__':
    SendMsg()
    Test()
```

```python
#test.py
from mySend import *

SendMsg()
# Test()#Test is not defined
```

```python
#直接的import不受__all__的影响
import mySend

mySend.SendMsg()
mySend.Test()
```

## package

下面的**TestMsg**文件夹就是package


```bash
./
    InfoDisplay.py
    TestMsg
        __init__.py
        SendMsg.py
        ReceiveMsg.py
```

python3不添加`__init__.py`文件可以做到`import TestMsg`,但是用不了；python2不添加`__init__.py`文件无法`import TestMsg`；


```python
#SendMsg.py
def Send():
    print("---sending msg")
```

```python
#ReceiveMsg.py
def Receive():
    print("---receive msg")
```

```python
#__init__.py
__all__=["SendMsg",]
```

```python
import TestMsg #import后面不能使package

# TestMsg.SendMsg.Send()#报错
# TestMsg.ReceiveMsg.Receive()#报错
```

```python
import TestMsg.SendMsg
import TestMsg.ReceiveMsg

TestMsg.SendMsg.Send()
TestMsg.ReceiveMsg.Receive()
```

```python
from TestMsg import SendMsg
from TestMsg import ReceiveMsg

SendMsg.Send()
ReceiveMsg.Receive()
```

```python
from TestMsg.SendMsg import Send
from TestMsg.ReceiveMsg import Receive

Send()
Receive()
```

```python
from TestMsg import *

SendMsg.Send()#warning
# ReceiveMsg.Receive()#报错
```

`__init__.py`中有其他内容，导入时也会执行一次

```python
__all__=["SendMsg",]

print("he")
```

### 标准格式

python2,python3都可以用

```python
#__init__.py
__all__=["SendMsg",]

from . import SendMsg
```

```python
#InfoDisplay.py
import TestMsg #因为在__init__.py中import SendMsg

TestMsg.SendMsg.Send()
# TestMsg.ReceiveMsg.Receive()#报错
```

```python
from TestMsg import *

SendMsg.Send()
# ReceiveMsg.Receive()#报错
```

[标准](https://chrisyeh96.github.io/2017/08/08/definitive-guide-python-imports.html)

任何情况都不会影响绝对路径import；

相对import组合比较复杂

## 制作package并发布

制作好的package:

```bash
parent/
    one/
        __init__.py
        bar.py
        foo.py
    two/
        __init__.py
        test.py
    __init__.py
```

### method 1：手动

手写`setup.py`文件放到package外部

```bash
setup.py
parent/
```

```python
#setup.py
# from setuptools import setup
from distutils.core import setup #默认的，但是没有上面的好

setup(
    name='untitled1',
    version='0.0.1',
    packages=['parent', 'parent.one', 'parent.two'],
    url='',
    license='',
    author='grey',
    author_email='gewei@foxmail.com',
    description='helloTest'
)
```

```bash
#build，还是文件夹，没有setup.py
python3 setup.py build
#打包成xxx.tar.gz
python3 setup.py sdist
```

生成的压缩包可以放到github,或者直接复制给别人用；

```bash
#解压xxx.tar.gz
tar -zxvf xxx.tar.gz

cd

python3 setup.py install

#然后就可以在IDE中import了

#要删除这个包，需要进入
"Python36\Lib\site-packages"直接删除就行了，不方便管理
```

### method2: pycharm 制作

```bash
#可以使用pycharm的File/NewPackage自动生成带有__init__.py的文件夹
parent/
    one/
        __init__.py
        bar.py
        foo.py
    two/
        __init__.py
        test.py
    __init__.py
```

然后，选中parenet文件夹，`Tools/Creat setup.py`, 自动生成

```python
#setup.py
from setuptools import setup

setup(
    name='untitled1',
    version='0.0.1',
    packages=['parent', 'parent.one', 'parent.two'],
    url='',
    license='',
    author='grey',
    author_email='gewei@foxmail.com',
    description='helloTest'
)
```

- 如果要安装到本机,选中`setup.py`，直接`Tools/run setup.py tasks`,选择`install`
- 如果要分享源码,选中`setup.py`，直接`Tools/run setup.py tasks`,选择`sdist`
- 如果要分享包,选中`setup.py`，直接`Tools/run setup.py tasks`,选择`bdist_egg`

安装到本机：略

分享源码：与上面的安装方法相同

分享egg安装包到其他机器上面：

```bash
#进入python的Scripts文件夹

D:\ProgrammingTools\Python36\Scripts>easy_install.exe xxx-py36.egg
```

## 给程序传参数

```bash
#example
ping 127.0.0.1

python3 setup.py install
```

```python
#test.py
import sys

print(sys.argv)

print(f"Hello, {sys.argv[1]}")
```

```bash
#test
python test.py Grey

#output
['test.py', 'Grey']
Hello, Grey
```

python2中的`rang(0,1000000000)`返回值是list，有超出内存的风险;python3中返回值是对象；

```python
list1=[x for x in range(10)]
list2=[11 for x in range(10)]#后面的for只是负责循环而已
list3=[x*x for x in range(10) if x%2==0]#带if
list4=[x for x in range(10) for y in range(3)]
list5=[y for x in range(3) for y in range(10)]
print(list1)
print(list2)#[11, 11, 11, 11, 11, 11, 11, 11, 11, 11]
print(list3)#[0, 4, 16, 36, 64]
print(list4)#[0, 0, 0, 1, 1, 1, 2, 2, 2, 3, 3, 3, 4, 4, 4, 5, 5, 5, 6, 6, 6, 7, 7, 7, 8, 8, 8, 9, 9, 9]
print(list5)#[0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
```

## 循环导入

设计顶层框架的人要设计好，避免这样的事情发生；

a.py中`import` b.py, b.py中`import` a.py

```bash
./
    fileA.py
    fileB.py
```

```python
#fileA.py
from fileB import funcB1

def funcA1():
    print("---A1---")
def funcA2():
    print("---A2---")
    funcB1()

funcA2()
```

```python
#fileB.py
from fileA import funcA1

def funcB1():
    print("---B1---")
    funcA1()
```

解决方法：用一个主模块fileC去调用fileA, fileB，防止这样的循环调用

## standard library

Python有一套很有用的标准库(standard library)

| 标准库          | 说明                           |
| --------------- | ------------------------------ |
| builtins        | 内建函数默认加载               |
| os              | 操作系统接口                   |
| sys             | Python自身的运行环境           |
| functools       | 常用的工具                     |
| json            | 编码和解码 JSON 对象           |
| logging         | 记录日志，调试                 |
| multiprocessing | 多进程                         |
| threading       | 多线程                         |
| copy            | 拷贝                           |
| time            | 时间                           |
| datetime        | 日期和时间                     |
| calendar        | 日历                           |
| hashlib         | 加密算法                       |
| random          | 生成随机数                     |
| re              | 字符串正则匹配                 |
| socket          | 标准的 BSD Sockets API         |
| shutil          | 文件和目录管理                 |
| glob            | 基于文件通配符搜索(没有re好用) |

### `hashlib`

```python
import hashlib

# m=hashlib.md5()
m=hashlib.sha256()
print(m)
#待加密文本
m.update(b'grey')
print(m.hexdigest())
m.update(b'james')
print(m.hexdigest())
```

```bash
#output
<sha256 HASH object @ 0x0000022F9D91BF08>
67fc36385bb47db091a4d850d450021a8b7ba3b58615fc918808d4e636d8ad46
c78b897e41ae4a3d8b0630e248d74c4673f030a9f6a37456ff4fc8f64a0e3965
```

md5被破解的原理：用已经存在的密码去生成32位的md5串，然后用生成的md5串去和你的比较，也就是彩虹表；

```python
import hashlib
import datetime

KEY_VALUE = 'grey'
now = datetime.datetime.now()
m = hashlib.md5()
str1 = f'{KEY_VALUE}{now.strftime("%Y%m%d")}'
print(str1)
m.update(str1.encode('utf-8'))#必须是二进制
value = m.hexdigest()
print(value)
```

```bash
#output
grey20180316
44257a6f9e0761caa1884507dae5841e
```

## extend library

| 扩展库               | 说明                                      |
| -------------------- | ----------------------------------------- |
| requests             | 使用的是 urllib3，继承了urllib2的所有特性 |
| urllib               | 基于http的高层库                          |
| scrapy               | 爬虫                                      |
| beautifulsoup4       | HTML/XML的解析器                          |
| celery               | 分布式任务调度模块                        |
| redis                | 缓存                                      |
| Pillow(PIL)          | 图像处理                                  |
| xlsxwriter           | 仅写excle功能,支持xlsx                    |
| xlwt                 | 仅写excle功能,支持xls ,2013或更早版office |
| xlrd                 | 仅读excle功能                             |
| elasticsearch        | 全文搜索引擎                              |
| pymysql              | 数据库连接库                              |
| mongoengine/pymongo  | mongodbpython接口                         |
| matplotlib           | 画图                                      |
| numpy/scipy          | 科学计算                                  |
| django/tornado/flask | web框架                                   |
| xmltodict            | xml 转 dict                               |
| SimpleHTTPServer     | 简单地HTTP Server,不使用Web框架           |
| gevent               | 基于协程的Python网络库                    |
| fabric               | 系统管理                                  |
| pandas               | 数据处理库                                |
| scikit-learn         | 机器学习库                                |

爬虫：requests, urllib, scrapy, beautifulsoup4,redis

```bash
#-m就是module的意思
D:\ProgrammingTools\Python36>python.exe -m http.server 8888
#本机访问
#然后再浏览器里面输入,就可以访问了
http://localhost:8888/

#其他机器访问，必须知道本机的ip
http://10.128.160.66:8888/
```

爬虫获取的数据，配合django, tornado可以图形化数据；

## pdb

| 命令              | 简写命令      | 作用                              |
| ----------------- | ------------- | --------------------------------- |
| list              | l             | 查看当前行的代码段                |
| next              | n             | 执行下一行                        |
| continue          | c             | 继续执行程序                      |
| break lineno      | b lineno      | 在指定行设置断点                  |
| break file:lineno | b file:lineno | 在指定文件的行设置断点            |
| clear num         |               | 删除指定断点                      |
| break             | b             | 显示所有断点                      |
| step              | s             | 进入函数                          |
| args              | a             | 查看传入参数                      |
| return            | r             | 快速执行到函数最后的一行，然后按n |
| quit              | q             | 中止并退出                        |
| print             | p             | 打印变量的值                      |
| help              | h             | 帮助                              |
|                   | 回车          | 重复上一条命令                    |
| bt                |               | 查看函数调用栈帧                  |

python命令行调试; 当然用IDE(Edit,Run,Debug功能集成)更好啊；

gdb也是这么调试的；也是命令行调试；

```bash
# better
python -m pdb some.py
```

```bash
#交互界面调试一个函数，比如ipython3中, 其中第一步就要使用s进入函数
import pdb
pdb.run('testfun(args)')
```

```python
#程序中埋点
import pdb

def getAverage(a, b):
	result = a+b
	print("result=%d"%result)
	return result

a = 100
b = 200
c = a+b
#埋点
pdb.set_trace()
ret = getAverage(a, b)
print(ret)
```

pdb调试的三种方式：

- `python -m pdb some.py`
- 交互界面，调试函数
- 程序埋点

python调试：

- pdb
- print(日志调试，装饰器可以打印日志，try可以打印日志，然后热修复(程序不停))
- IDE

pdb 调试有个明显的缺陷就是对于多线程，远程调试等支持得不够好，同时没有较为直观的界面显示，不太适合大型的 python 项目