# Python Module

<!-- TOC -->

- [Python Module](#python-module)
    - [`import`](#import)
        - [`import` module](#import-module)
        - [`import` package](#import-package)
        - [module classification](#module-classification)
    - [`getpass` module](#getpass-module)
    - [`time` & `datetime`](#time--datetime)
    - [`sys` & `os` module](#sys--os-module)
    - [`shutil` module](#shutil-module)
    - [`pickle` , `json`, `shelve`](#pickle--json-shelve)
    - [`xml` module](#xml-module)
    - [`pyyaml` & `configparser` module](#pyyaml--configparser-module)
    - [`hashlib` &`hmac` module](#hashlib-hmac-module)
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
    - [extend library](#extend-library)
    - [pdb](#pdb)

<!-- /TOC -->

## `import`

`import`导入顺序是按照`sys.path`的list顺序导入

```bash
# import sys
# print(sys.path)
['.', 
'D:\\ProgrammingTools\\empty_py3\\Scripts\\python36.zip', 'D:\\ProgrammingTools\\Anaconda3\\DLLs'
'D:\\ProgrammingTools\\Anaconda3\\lib', 
'D:\\ProgrammingTools\\Anaconda3',
'D:\\ProgrammingTools\\empty_py3',
'D:\\ProgrammingTools\\empty_py3\\lib\\site-packages',
'D:\\ProgrammingTools\\empty_py3\\lib\\site-packages\\setuptools-39.1.0-py3.6.egg']
```

可以通过如下方法添加其他目录

```python
import os, sys

BASE_DIR = os.path.dirname(os.path.dirname(os.path.abspath(__file__)))
# sys.path.append(BASE_DIR)
# 下面这种，从BASE_DIR开始搜索
sys.path.insert(0, BASE_DiR)
```

### `import` module

moudle本质是实现一定功能的`.py`

vscode下测试, pycharm需要**Mark as Sysroot**

```bash
./
    moduleA.py
    main.py
```

```python
# moduleA.py
num=100

def say_hello():
    print("hello, I'm moduleA")
```

```python
# main.py
# import method1:
import moduleA

print(moduleA.num)
moduleA.say_hello()
```

```python
# import method2:
from moduleA import *
```

```python
# import method3:
from moduleA import num, say_hello
```

```python
# import xxx as yyy
import numpy as np
import matplotlib.pyplot as plt
from matplotlib import pyplot as plt
```

> `import` **模块(.py)** 本质是将`moduleA.py`中的代码全部解释了一遍, 然后赋值给变量`moduleA`, 然后利用这个变量来进行调用; 
>
> `from xxx import yyy`是直接将`moduleA`中的变量拿过来执行, 不用中间的变量来进行调用

`import`优化

```python
# main.py
import moduleA

def func1():
    # 如果有大量的moduleA.say_hello(), 就会重复查找，效率低
    # 换用from moduleA import say_hello即可
    moduleA.say_hello()
    print("func1")

def func2():
    moduleA.say_hello()
    print("func2")
```

### `import` package 

package: 一个包含空`__init__.py`文件的目录, 用来组织一堆`.py`

> `import` **Package** 的本质是执行`__init__.py`

```bash
main.py
PackageA/
    __init__.py
    moduleA.py
```

```python
# __init__.py
from . import moduleA
```

```python
# main.py
import PackageA

PackageA.moduleA.say_hello()
```

### module classification

- [standard library](https://docs.python.org/3/py-modindex.html)
- open source library: [Pypi](https://pypi.org/), github
- custom library

## `getpass` module

```python
import getpass

upwd = getpass.getpass('enter your password:')
print(upwd)
```

## `time` & `datetime`

[time & datetime](http://blog.51cto.com/egon09/1840425)

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

## `shutil` module

可以查看源码了解功能, [simple source](http://www.cnblogs.com/wupeiqi/articles/4963027.html)

```python
import shutil

# copyfile
shutil.copyfile('test.txt', 'newnew.txt')

# copy directory and files
shutil.copytree('testDir', 'newDir')

# remove directory and files
shutil.rmtree('testDir')

# move file
shutil.move('outer.txt', 'newDir')

# archive dir
shutil.make_archive('newZip','zip','newDir/')
```

## `pickle` , `json`, `shelve`

pickle vs json:
- pickle: bytes与python类型直接转换: `dumps`, `dump`, `loads`, `load`
- json: 字符串与python类型之间转换: `dumps`, `dump`, `loads`, `load`
- shelve: 对pickle的高级封装，可以进行key-value访问

上面的python类型只能是数据类型，不能是类似`<function>`这种类型

`json`只能处理str, list, tuple, set, dict这中简单的; `json`主要是不同语言中的交换数据, 主要用的是dictionary; `xml`被`json`淘汰;`pickle`是python专用的;

```python
# pickle
import pickle

data1=[1, 2, 3, 4, 5]
data2='grey'

# dumps
pickle1_repr=pickle.dumps(data1)
pickle2_repr=pickle.dumps(data2)
print(pickle1_repr, pickle2_repr)

# loads
data1_frompickle1=pickle.loads(pickle1_repr)
data2_frompickle2=pickle.loads(pickle2_repr)
print(data1_frompickle1, data2_frompickle2)

# dump: 二进制文件
with open('temp1.dat', 'wb') as file1, open('temp2.dat', 'wb') as file2:
    pickle.dump(data1, file1)
    pickle.dump(data2, file2)
# load
with open('temp1.dat', 'rb') as file1, open('temp2.dat', 'rb') as file2:
    data3=pickle.load(file1)
    data4=pickle.load(file2)
print(data3, data4)
```

```python
# json
import json

data1=[1, 2, 3, 4, 5]
data2='grey'

# dumps: 返回值是str
json1_repr=json.dumps(data1)
json2_repr=json.dumps(data2)
print(json1_repr, json2_repr) 

# loads： 返回值是原类型
data1_fromjson1=json.loads(json1_repr)
data2_fromjson2=json.loads(json2_repr)
print(data1_fromjson1, data2_fromjson2)

# dump: 文本文件
with open('temp1.dat', 'w') as file1, open('temp2.dat', 'w') as file2:
    json.dump(data1, file1)
    json.dump(data2, file2)
# load： 返回值是原类型
with open('temp1.dat', 'r') as file1, open('temp2.dat', 'r') as file2:
    data3=json.load(file1)
    data4=json.load(file2)
print(data3, data4) 
```

```python
#查询dangdang的数据，并简单统计
import codecs

def  loaddata():
    filepath = r"C:\dangdang.txt"
    file = codecs.open(filepath, "rb", encoding="gbk", errors="ignore")
    global datalist #引用全局
    datalist=file.readlines() #读取文件到list,疯狂占用内存
    file.close()
def  search(namestr):
    savefilepath="C:\\data\\"+namestr+".txt"
    savefile=open(savefilepath,"wb")
    numbers=0#统计查询的人的记录数
    for  line  in datalist:
        if line.find(namestr)!=-1:
            print(line,end="") #显示数据
            numbers +=1
            savefile.write(line.encode("utf-8"))#写入
    savefile.write(("数量"+str(numbers)).encode("utf-8"))
    savefile.close()

#program begin
datalist=[]
print("load  file start")
loaddata()
print("load  file end")
while True:
    searchname=input("要查询的数据")
    search(searchname)
```

```python
# shelve
import shelve

name='grey'
info={'age':23, 'job':'student'}

# write to file
with shelve.open('shelve_test') as db:
    db['name']=name
    db['info']=info

# read from file
with shelve.open('shelve_test') as db:
    print(db.get('name'))
    print(db.get('info'))
    print(db.get('date'))# None
```

## `xml` module

```bash
./
    test.xml
    main.py
```

```xml
<?xml version="1.0"?>
<data>
    <country name="Singapore">
        <rank updated="yes">5</rank>
        <year>2011</year>
        <gdppc>59900</gdppc>
    </country>
    <country name="Panama">
        <rank updated="yes">69</rank>
        <year>2011</year>
        <gdppc>13600</gdppc>
    </country>
</data>
```

```python
# main.py
# read xml
import xml.etree.ElementTree as ET
 
tree = ET.parse("test.xml")
root = tree.getroot()
print(root.tag)
 
#遍历xml文档
for child in root:
    print(child.tag, child.attrib)
    for i in child:
        print(i.tag,i.text, i.attrib)
 
#只遍历year节点
print('*'*20)
for node in root.iter('year'):
    print(node.tag,node.text)
```

```python
# modify xml
import xml.etree.ElementTree as ET
 
tree = ET.parse("test.xml")
root = tree.getroot()
 
#修改
for node in root.iter('year'):
    new_year = int(node.text) + 1
    node.text = str(new_year)
    node.set("updated","yes")
 
tree.write("test.xml")
 
 
#删除node
for country in root.findall('country'):
   rank = int(country.find('rank').text)
   if rank > 50:
     root.remove(country)
 
tree.write('output.xml')
```

```python
# create xml
import xml.etree.ElementTree as ET
 
 
new_xml = ET.Element("namelist")

name = ET.SubElement(new_xml,"personinfo",attrib={"enrolled":"yes"})
age = ET.SubElement(name,"age",attrib={"checked":"no"})
sex = ET.SubElement(name,"sex")
sex.text = '33'

name2 = ET.SubElement(new_xml,"personinfo",attrib={"enrolled":"no"})
age = ET.SubElement(name2,"age")
age.text = '19'
 
et = ET.ElementTree(new_xml) #生成文档对象
et.write("test.xml", encoding="utf-8",xml_declaration=True)
 
ET.dump(new_xml)
```

```xml
<?xml version='1.0' encoding='utf-8'?>
<namelist>
    <personinfo enrolled="yes">
        <age checked="no" />
        <sex>33</sex>
    </personinfo>
    <personinfo enrolled="no">
        <age>19</age>
    </personinfo>
</namelist>
```

## `pyyaml` & `configparser` module

`pip install pyyaml`, then ...

yaml是做配置文件的;现在常用config, nginx, MySQL就是这种格式的配置文件;

```python
import configparser
 
# write to file
config = configparser.ConfigParser()
config["DEFAULT"] = {'ServerAliveInterval': '45',
                      'Compression': 'yes',
                     'CompressionLevel': '9'}
 
config['bitbucket.org'] = {}
config['bitbucket.org']['User'] = 'hg'

config['topsecret.server.com'] = {}
topsecret = config['topsecret.server.com']
topsecret['Host Port'] = '50022'     # mutates the parser
topsecret['ForwardX11'] = 'no'  # same here

# add one to "DEFAULT"
config['DEFAULT']['ForwardX11'] = 'yes'

with open('example.ini', 'w') as configfile:
   config.write(configfile)

# read file
config2=configparser.ConfigParser()
config2.read('example.ini')
print(config2.sections) # ['bitbucket.org', 'topsecret.server.com']
print(congig2.defaults) # ...

# get value
config['DEFAULT']['forwardx11']
config['DEFAULT'].get('forwardx11')
config2['bitbucket.org']['user']
config2['bitbucket.org'].get('user')

# remove section
config2.remove_section('bitbucket.org')
with open('new_config.conf', 'w') as file:
    config2.write(file)
```

## `hashlib` &`hmac` module

md5被破解的原理：用已经存在的密码去生成32位的md5串，然后用生成的md5串去和你的比较，也就是彩虹表；

hashlib用途:
- 网页防篡改
- 加密

```python
import hashlib

m1=hashlib.sha1()
print(m1.digest_size) # 20, so length=40
m1.update(b'hello')
print(m1.hexdigest()) # aaf4c61ddcc5e8a2dabede0f3b482cd9aea9434d
m1.update(b'it is me')
print(m1.hexdigest()) # d728c718b8c58cc2da072753abc2103f6d2b8705
m1.update(b'it has been a long time')

m2=hashlib.sha1()
m2.update(b'helloit is me') # 结果与上面第二个相同
print(m2.hexdigest())# d728c718b8c58cc2da072753abc2103f6d2b8705
```

```python
import hmac

h1=hmac.new(b'this is key', b'this is msg')
print(h1.digest_size) # length=32
print(h1.hexdigest())

h2=hmac.new(b'this is key')
h2.update(b'this is msg')
print(h2.hexdigest())
```

```python
import hashlib
import datetime

KEY_VALUE = 'grey'
now = datetime.datetime.now()
m = hashlib.md5()
str1 = f'{KEY_VALUE}{now.strftime("%Y%m%d")}'
print(str1) # grey20180316
m.update(str1.encode('utf-8'))#必须是二进制
value = m.hexdigest()
print(value) # 44257a6f9e0761caa1884507dae5841e
```

## module storage

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