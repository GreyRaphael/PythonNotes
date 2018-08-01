# Python Examples

<!-- TOC -->

- [Python Examples](#python-examples)
    - [Add number to image](#add-number-to-image)
    - [get random serial](#get-random-serial)
    - [count world](#count-world)
    - [batch change images's resolution](#batch-change-imagess-resolution)
    - [count code](#count-code)
    - [generate verify code](#generate-verify-code)
    - [network](#network)
        - [http request](#http-request)
        - [socket](#socket)
    - [python extending](#python-extending)
        - [python extending with c](#python-extending-with-c)
        - [python extending with .so file](#python-extending-with-so-file)
        - [python extending with c++](#python-extending-with-c)
        - [python extending with cython](#python-extending-with-cython)

<!-- /TOC -->

## Add number to image

`conda install pillow`

```python
from PIL import Image, ImageFont, ImageDraw

im=Image.open('Pig.png')
# print(im.size)
draw=ImageDraw.Draw(im)

font_size=min(im.size)//5
# font=ImageFont.truetype(r'C:\Windows\Fonts\arial.ttf', font_size)
font=ImageFont.truetype('arial.ttf', font_size)

draw.text((im.size[0]-font_size, 0), '8', font=font, fill=(255,0,0))
im.save('modified.png', 'png')
```

## get random serial

Method1:

```python
import string, random

def get_random():
    # 返回字母与数字的随机组合，共4个字符
    return ''.join(random.sample(string.ascii_letters + string.digits, 4))

def concatenate(group):
    return '-'.join([get_random() for _ in range(group)])

def generate(N):
    return [concatenate(4) for _ in range(N)]

print(generate(3))
# ['QOk9-cMdw-B7Y3-ijsV', 'cofO-XYnZ-2nGs-idxW', 'LqsF-yVpI-uSGH-FCIp']
```

Method2: `pip install rstr`

```python
from rstr import xeger

def concatenate(group):
    return xeger(r'[A-Za-z0-9]{4}(-[A-Za-z0-9]{4})'+f'{{{group-1}}}')

def generate(N):
    return [concatenate(4) for _ in range(N)]

generate(3)
#['frjD-Mby3-QeXC-kRVp', 'bEjp-36b7-BUIh-gMib', 'PwEf-ndLn-nUI3-3jMe']
```

## count world

single file:

```python
import re
from collections import Counter

def get_word_frequency(file_path):
    regex=re.compile(r'[A-Za-z]+')
    with open(file_path) as file:
        words_list=regex.findall(file.read())
        print(words_list)
        # return Counter(words_list)
        # sort with freq
        return Counter(words_list).most_common()    

get_word_frequency('songLyric.txt')
```

multiple files:

```python
import re, os
from collections import Counter

def get_word_frequency(file_name):
    regex=re.compile(r'[A-Za-z]+')
    with open(file_name) as file:
        words_list=regex.findall(file.read())
        return Counter(words_list)

black_list=['the', 'in', 'of', 'and', 'to', 'has', 'that', 's' 'is', 'are', 'a', 'with', 'as', 'an', 'I']

def run(file_dir):
    os.chdir(file_dir)
    total_counter=Counter()
    
    for filename in os.listdir('.'):
        if filename.endswith('.txt'):
            total_counter+=get_word_frequency(filename)
    for word in black_list:
        total_counter[word]=0
    return total_counter.most_common()

run(r'D:\SomeLyrics')
```

## batch change images's resolution

```python
import os
from PIL import Image

def process_img(source_dir, dest_dir, filename, img_type):
    imtype='jpeg' if img_type=='.jpg' else 'png'
    im=Image.open(source_dir + filename)
    rate=max(im.size[0]/1920.0 if imsize[0]>1920 else 0, im.size[1]/1080.0 if imsize[1]>1080 else 0)
    if rate:
        im.thumbnail((im.size[0]/rate, im.size[1]/rate))
    im.save(dest_dir+file_name, imtype)

def run(file_dir, dest_dir):
    os.chdir(file_dir)
    for filename in os.listdir('.'):
        file_ext=os.path.splitext(filename)[1]
        if filename.endswith('.png') or filename.endswith('.jpg'):
            process_img(file_dir, dest_dir, file_name, file_ext)

run(r'D:\src_img', r'D:\dest_img')
```

## count code

```python
import os, re

def analyze_code(filename):
    """open a .py file, count all_lines, comment_lines, blank_lines"""
    total_lines, comment_lines, blank_lines = 0, 0, 0
    with open(filename) as file:
        lines = file.readlines()
        total_lines = len(lines)
        line_index = 0
        while line_index < total_lines:
            line = lines[line_index]
            if re.match(r'\s*#', line):
                comment_lines += 1
            elif re.match(r'\s*"""', line) or re.match(r"\s*'''", line):
                # 其中的""", '''也是注释
                comment_lines += 1
                while not re.match(r'.*"""', line) or not re.match(r".*'''", line):
                    line = lines[line_index]
                    comment_lines += 1
                    line_index += 1
            elif line == '\n':
                blank_lines += 1
            line_index += 1

        print(f'{filename} contains:')
        print(f'total lines:{total_lines}')
        print(f'comment lines: {comment_lines}, ratio={comment_lines/total_lines:.2%}')
        print(f'blank lines: {blank_lines}, ratio={blank_lines/total_lines:.2%}')
        return total_lines, comment_lines, blank_lines


def run(file_dir):
    os.chdir(file_dir)
    total_lines, comment_lines, blank_lines = 0, 0, 0
    for filename in os.listdir('.'):
        if filename.endswith('.py'):
            single_file = analyze_code(filename)
            total_lines += single_file[0]
            comment_lines += single_file[1]
            blank_lines += single_file[2]
    print(f'total lines:{total_lines}')
    print(f'comment lines: {comment_lines}, ratio={comment_lines/total_lines:.2%}')
    print(f'blank lines: {blank_lines}, ratio={blank_lines/total_lines:.2%}')            

run('D:\\pythonfiles')
```

## generate verify code

```python
import string, random
from PIL import Image, ImageDraw, ImageFilter, ImageFont

def get_random_string():
    return [random.choice(string.ascii_letters+string.digits) for _ in range(4)]

def get_random_color():
    return (random.randint(30, 100), random.randint(30, 100),random.randint(30, 100))

def get_verify_code():
    width, height=240, 60
    im=Image.new('RGB', (width, height), (180, 180, 180))
    font=ImageFont.truetype('arial.ttf', 40)
    draw=ImageDraw.Draw(im)
    
    # add string
    code_string=get_random_string()
    for index, ch in enumerate(code_string):
        draw.text((60*index+10, 0), ch, font=font, fill=get_random_color())
    
    # add noise
    for _ in range(random.randint(1500, 3000)):
        draw.point((random.randint(0, width), random.randint(0, height)), fill=get_random_color())
    
    # blur image
    img=im.filter(ImageFilter.BLUR)
    
    img.save(''.join(code_string)+'.png', 'png')

get_verify_code()
```

## network

一般要么是基于原生socket请求, 要么就是http请求;

### http request

现在一般用的requests, 而不用urllib; [requests vs urllib](https://www.cnblogs.com/znyyy/p/7868511.html)

`pip install requests`

http中包含了get, post
- get: 下载得到网页的内容(地址栏上面)
- post: 提交表单给网站后台(F12, 在request里面)

```python
# request without parameter
imprt requests

# 一般网站
# r=requests.get('http://www.fortunechina.com/investing/c/2018-07/30/content_312935.htm')
headers={
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; WOW64; rv:61.0) Gecko/20100101 Firefox/61.0",
}
# 这个网站的response给出的就是request的各种信息
response=requests.get('http://httpbin.org/get', headers=headers)
print(response.text) # utf8 decoded content
print(response.content) # binary content
print(response.url) # http://httpbin.org/get
print(response.encoding) # utf-8
print(response.status_code) # 200
```

>使用response.text 时，Requests 会基于 HTTP 响应的文本编码自动解码响应内容，大多数 Unicode 字符集都能被无缝地解码。
>
>使用response.content 时，返回的是服务器响应数据的原始二进制字节流，可以用来保存图片等二进制文件。

```python
# request with parameters
import requests

headers={
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; WOW64; rv:61.0) Gecko/20100101 Firefox/61.0",
}
params={
    'q':'hello',
}
response=requests.get("https://cn.bing.com/search", headers=headers, params=params)

print(response.text)
```

```python
# get请求
import requests
url='http://127.0.0.1:1990/login'
data={"username":"admin","password":123456}
res=requests.get(url,data)
#res=res.text#text方法是获取到响应为一个str，也不需要对res进行转换等处理
res=res.json()#当返回的数据是json串的时候直接用.json即可将res转换成字典
print(res)

#post请求
import requests
url='http://127.0.0.1:1990/login'
data={"username":"admin","password":123456}
res=requests.post(url,data)
#res=res.text#text方法是获取到响应为一个str，也不需要对res进行转换等处理
res=res.json()#当返回的数据是json串的时候直接用.json即可将res转换成字典
print(res)

#当传参格式要求为json串时
import requests
url='http://127.0.0.1:1990/login'
data={"username":"admin","password":123456}
res=requests.post(url,json=data)#只需要在这里指定data为json即可
#res=res.text#text方法是获取到响应为一个str，也不需要对res进行转换等处理
res=res.json()#当返回的数据是json串的时候直接用.json即可将res转换成字典
print(res)

#传参含cookie
import requests
url='http://127.0.0.1:1990/login'
data={"username":"admin","password":123456}
cookie={"sign":"123abc"}
res=requests.post(url,json=data,cookies=cookie)#只需要在这里指定cookies位cookie即可，headers，files等类似
res=res.json()
print(res)
```

上传文件

```python
import requests

url='http://httpbin.org/post/'
myfile={'file': open('pig.jpg', 'rb')}
response=requests.post(url, files=myfile)
print(response.text)
```

模拟用户登陆;

>在 requests 里，session对象是一个非常常用的对象，这个对象代表一次用户会话：从客户端浏览器连接服务器开始，到客户端浏览器与服务器断开。
>
>会话能让我们在跨请求时候保持某些参数，比如在同一个 Session 实例发出的所有请求之间保持 cookie

```python
import requests

url='https://www.yunpanjingling.com/'
user={'email':'vip.gewei@foxmail.com', 'password':'xQfheYU9HRPVMX'}
headers={
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; WOW64; rv:61.0) Gecko/20100101 Firefox/61.0",
}
session=requests.Session()
response=session.post(url, data=user, headers=headers)

print(response.text)
```

[proxy](https://blog.csdn.net/qq_37616069/article/details/80376776)

```python
import requests
 
# 根据协议类型，选择不同的代理
proxies = {
  "http": "http://12.34.56.79:9527",
  "https": "http://12.34.56.79:9527",
}
 
response = requests.get("http://www.baidu.com", proxies = proxies)
print(response.text)
```

处理https请求

```python
import requests
# verify默认是True
response = requests.get("https://www.baidu.com/", verify=True)
print(response.text)
```

如果SSL证书验证不通过，或者不信任服务器的安全证书，则会报出SSLError; 比如12306

```python
import requests
response = requests.get("https://www.12306.cn/mormhweb/", verify=False)
print response.text
```

### socket

高并发服务器一般用的是socket

```python
# tcp/ip server
import socket
# ip+port可以唯一确定互联网中的进程
myhost='' # 多个网卡的时候会绑定多个网卡
port=9990
# 第一参数是IP协议， 第二个参数是TCP协议
sock_obj=socket.socket(socket.AF_INET, socket.SOCK_STREAM)
sock_obj.bind((myhost, port))
sock_obj.listen(128) # 同时可以有128个

while True:
    conn, addr = sock_obj.accept()
    print(f'{addr} is online...')
    while True:
        data=conn.recv(1024).decode('gbk')
        print(data)
        if not data:
            break
        reply=('echo'+data).encode('gbk')
        conn.send(reply)
    conn.close()
```

要么使用[NetAssistant](http://www.cmsoft.cn/reslink.php?id=205)来连接上面的服务器， 要么使用Linux上面的`nc 222.29.69.149 9990`来连接

或者将上面的代码拷贝到阿里云服务器上: `scp myserver.py grey@39.106.18.97:/home/grey/`, 并且配置规则，允许端口;就可以作为服务器了

查看端口占用情况`netstat -apn|grep 999`

单客户端服务器(断开client的时候有bug)

```python
# python已经封装的服务器
from socketserver import TCPServer, BaseRequestHandler
# 出错处理
import traceback

class my_string_request_handler(BaseRequestHandler):
    def handle(self):
        while True:
            # 客户端端口时，抛出异常
            try:
                # 读取1024Bytes，去掉前后的空白
                data=self.request.recv(1024).strip()
                if not data:
                    break
                print(f'{self.client_address}: {data}')
                self.request.sendall(data.upper())
            except:
                traceback.print_exc()
                break

addr=('', 9990)
server=TCPServer(addr, my_base_request_handler)
server.serve_forever()
```

多客户端服务器(多线程并发服务器)

```python
from socketserver import ThreadingTCPServer, StreamRequestHandler
import traceback

class my_base_request_handler(StreamRequestHandler):
    def handle(self):
        while True:
            # 客户端端口时，抛出异常
            try:
                data=self.rfile.readline().strip()
                if not data:
                    break
                print(f'{self.client_address}: {data}')
                self.wfile.write(data.upper())
            except:
                traceback.print_exc()
                break

addr=('', 9990)
server=ThreadingTCPServer(addr, my_string_request_handler)
server.serve_forever()
```

## python extending

why:
- 性能
- 源代码的私密性

procedure:
- write c/c++ code
- python类型适配, 包装c/c++ code
- 编译、测试

一般网络开发: 业务逻辑， 性能计算， 存储(与数据库交互); 除了性能，其他用python都是极好的;

常考:字符串解析+数据结构算法(crud, sort)
- 字符串逆序
- 字符串查找子串
- crud, sort

### python extending with c

```bash
./
    fac.c
    facHead.h
    kkkwrapper.c
    setup.py
```

step1: write c/c++ code

```c
//fac.c
#include<stdio.h>
#include<stdlib.h>
#include<string.h>
int fac(int n)
{
   if(n<2)
       return 1;
   return n*fac(n-1);
}

//字符串反转
char *reverse(char *s)
{
  char t,*p=s,*q=(s+(strlen(s)-1));
  while(s && (p<q))
  {
   t=*p;
   *p++=*q;
   *q--=t;
  }
  return s;
}

int test(void)
//int main()
//main()是为了测试 gcc test.c -o testc
// ./testc
{
  char s[1024];
  printf("4!==%d\n",fac(4));
  printf("8!==%d\n",fac(8));

  strcpy(s,"mynameid kkk");
  printf("reversing,get '%s'\n",reverse(s));
  return 0;
}
```

step2: write header file

```c
//facHead.h
#ifndef KKK_H_
#define KKK_H_

int fac(int n);
char *reverse(char *s);
int test(void);

#endif
```

step3:手写包裹函数(包裹函数名不严格限制)

必须借助中间的东西才能访问c/c++库;
- `#include <Python.h>`
- 为每一个函数增加一个`PyObject *Module_func()`的包裹函数
- 为module增加一个PyMethod DefModuleMethods[]的数组
- 增加module的初始化函数`void initModule()`

```c
//kkkwrapper.c for python2
#include "facHead.h"
#include "Python.h"
#include <stdlib.h>
#include <string.h>

static PyObject *kkk_fac(PyObject *self,PyObject *args)
{
  int num;
  if(!PyArg_ParseTuple(args,"i",&num)) //python类型变成c类型
      return NULL;
  return (PyObject *)Py_BuildValue("i",fac(num));//c返回类型变成python类型
}

static PyObject *kkk_rever(PyObject *self,PyObject *args)
{
  char *src;
  char *mstr;
  PyObject *retVal;

  if(!PyArg_ParseTuple(args,"s",&src))
      return NULL;

  mstr=malloc(strlen(src)+1);
  strcpy(mstr,src);
  reverse(mstr);
  retVal=(PyObject *)Py_BuildValue("ss",src,mstr);
  free(mstr);

  return retVal;
}

static PyObject *kkk_test(PyObject *self,PyObject *args)
{
  test();
  return (PyObject *)Py_BuildValue("");
  //return Py_None;
}

static PyMethodDef kkkMethods[]=
{
  {"fac",kkk_fac,METH_VARARGS},
  {"rever",kkk_rever,METH_VARARGS},
  {"test",kkk_test,METH_VARARGS},
  {NULL,NULL},
};

//初始化操作
void initkkk(void)
{
  Py_InitModule("kkk",kkkMethods);
}
```

```c
//kkkwrapper.c for python3
#include "facHead.h"
#include "Python.h"
#include <stdlib.h>
#include <string.h>

static PyObject *kkk_fac(PyObject *self,PyObject *args)
{
  int num;
  if(!PyArg_ParseTuple(args,"i",&num))//python类型变成c类型
      return NULL;
  return (PyObject *)Py_BuildValue("i",fac(num));//c类型变成python类型
}

static PyObject *kkk_rever(PyObject *self,PyObject *args)
{
  char *src;
  char *mstr;
  PyObject *retVal;

  if(!PyArg_ParseTuple(args,"s",&src))
      return NULL;

  mstr=malloc(strlen(src)+1);
  strcpy(mstr,src);
  reverse(mstr);
  retVal=(PyObject *)Py_BuildValue("ss",src,mstr);
  free(mstr);

  return retVal;
}

static PyObject *kkk_test(PyObject *self,PyObject *args)
{
  test();
  return (PyObject *)Py_BuildValue("");
}
//==============================
static PyMethodDef kkkMethods[]=
{
  {"fac",kkk_fac,METH_VARARGS},
  {"rever",kkk_rever,METH_VARARGS},
  {"test",kkk_test,METH_VARARGS},
  {NULL,NULL},
};

static struct PyModuleDef kkk =
{
    PyModuleDef_HEAD_INIT,
    "kkk", //name of module
    "this is c/++ moudle for python",//module documentation, may be NULL
    -1,/* size of per-interpreter state of the module, or -1 if the module keeps state in global variables. */
    kkkMethods
};

//初始化操作
PyMODINIT_FUNC PyInit_kkk(void)
{
    return PyModule_Create(&kkk);
}
```

python与c/c++数据转换

格式代码|python类型|c类型
---|---|---
s|str|char *
z|str/None|char */NULL
i|int|int
l|long|long
c|str|char
d|float|double
D|complex|Py_Complex*
O|any|PyObject*
S|str|PyStringObject

step4: 编译安装到python环境

- 创建setup.py
- 运行setup.py编译、链接c的扩展代码
- import
- 测试

```python
# setup.py
from distutils.core import setup, Extension
module_name='kkk'
kkk_module=Extension(module_name, sources=['kkkwrapper.c', 'fac.c'])
setup(name=module_name,
      version='0.1.0',
      description='c/cpp module for python test',
      ext_modules=[kkk_module])
```

`python setup.py build`会生成**kkk.cpython-36m-x86_64-linux-gnu.so**文件; 将该文件拷贝到任意位置, 然后同目录下使用**ipython**测试

```python
# in ipython
import kkk
kkk.test()
kkk.fac(5)
kkk.rever('hello')
```

如果有需要的话, `python setup.py install`, 可以在任意位置`import kkk`; 或者发布给其他人
`python setup.py sdist`

>recommened: `pip install .`来代替`python setup.py install`

### python extending with .so file

```bash
./
    libfac.so
    facHead.h
    kkkwrapper.c
    setup.py
```

其中`gcc -o libfac.so -shared -fPIC fac.c`生成了`libfac.so`; 名字必须是lib开头;

并且`export LD_LIBRARY_PATH=.`

```python
# setup.py
from distutils.core import setup, Extension
module_name='kkk'
kkk_module=Extension(module_name,
                    sources=['kkkwrapper.c'],
                    libraries=["fac"],
                    library_dirs=["."])
setup(name=module_name,
      version='0.1.0',
      description='c/cpp module for python test',
      ext_modules=[kkk_module])
```

`python setup.py build`得到的`kkk.cpython-36m-x86_64-linux-gnu.so`; 将`kkk.cpython-36m-x86_64-linux-gnu.so`与`libfac.so`拷贝到当前目录， 才能用**ipython**

比较麻烦; 

>推荐做法是用`cdll.LoadLibrary()`, [tuturial](http://intermediate-and-advanced-software-carpentry.readthedocs.io/en/latest/c++-wrapping.html)

```python
from ctypes import cdll
kkk=cdll.LoadLibrary('./libfac.so') # 写上路径，否则不在当前目录查找
kkk.test()
```

### python extending with c++

pass

### python extending with cython

```bash
./
    facHead.h
    fac.c
    kkkwrapper.pyx
    setup.py
```

```python
# kkkwrapper.pyx
cdef extern from "facHead.h":
    int fac(int n)
    char* reverse(char* s)
    int test()


def rever_fn(s):
    return reverse(s)
def fac_fn(n):
    return fac(n)
def test_fn():
    return test()
```

```python
# setup.py
from distutils.core import setup
from distutils.extension import Extension
from Cython.Build import cythonize

ext_modules = [
    Extension("kkk",sources=["kkkwrapper.pyx", 'fac.c'],)
]

setup(name="kkk",ext_modules=cythonize(ext_modules))
```

```python
In []: import kkk

In []: kkk.fac_fn(5)
Out[]: 120

In []: kkk.test_fn()
4!==24
8!==40320
reversing,get 'kkk diemanym'

# cython严格区分string, bytes;
In []: kkk.rever_fn(b'hello')
Out[]: b'olleh'
```