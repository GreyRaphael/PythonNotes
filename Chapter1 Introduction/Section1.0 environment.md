# Python Environment

<!-- TOC -->

- [Python Environment](#python-environment)
    - [Anaconda & VSCode](#anaconda--vscode)
    - [py2exe](#py2exe)
    - [pywin32](#pywin32)
    - [CGI](#cgi)
    - [python like shell](#python-like-shell)
    - [python project directory structure](#python-project-directory-structure)
        - [examples](#examples)
    - [Python code stype](#python-code-stype)
    - [Python in environment](#python-in-environment)
    - [Python3 encoding](#python3-encoding)

<!-- /TOC -->

## Anaconda & VSCode

配置[Anaconda3](https://mirrors.tuna.tsinghua.edu.cn/anaconda/archive/)环境
- PyCharm中选择Anaconda的python
- VSCode的setting: `"python.pythonPath": "D:/ProgrammingTools/Anaconda3/python",`

Anaconda中`conda update --all`遇到问题，看看结果，一般

```bash
#先安装ipykernel
conda update ipykernel
#then
conda update --all
```

>`D:\ProgrammingTools\Anaconda3\Lib`里面一般是系统lib  
>`D:\ProgrammingTools\Anaconda3\Lib\site-packages`里面一般是安装的lib  
>`D:\ProgrammingTools\Anaconda3\pkgs`里面的可以全部删除  

## py2exe

https://pypi.python.org/pypi/py2exe/0.9.2.0

## pywin32

调用windows自带的软件(text to speech,speech reconition),要用到
[Pywin32 Download](https://sourceforge.net/projects/pywin32/)

## CGI

download like
[httpd-2.2.25-win32-x86-no_ssl](https://archive.apache.org/dist/httpd/binaries/win32/), 自定义安装所有的东西，填入hostname(localhost,localhost,your email)

在浏览器中输入黑窗口提示的ip(或者直接`127.0.0.1`)

修改`C:\Program Files (x86)\Apache Software Foundation\Apache2.2\htdocs\index.html`的内容，在浏览器中的内容也会跟着修改

## python like shell

```python
#! /usr/bin/env python

print('hello, world')
```

将上面保存为`sample.py`, `chmod 755 sample.py`, `./sample.py`就可以运行了;

推荐使用`#! /usr/bin/env python`, 不推荐`#! /usr/bin/python`写死了

## python project directory structure

[stackoverflow suggestion](https://stackoverflow.com/questions/193161/what-is-the-best-project-structure-for-a-python-application)

```bash
Foo/
|-- bin/
|   |-- foo
|
|-- foo/
|   |-- tests/
|   |   |-- __init__.py
|   |   |-- test_main.py
|   |
|   |-- __init__.py
|   |-- main.py
|
|-- docs/
|   |-- conf.py
|   |-- abc.rst
|
|-- setup.py
|-- requirements.txt
|-- README
```

从bin/foo调用main.py; 主入口是`main.py`

### examples

```bash
Atm/
    atm/
        __init__.py
        main.py
    bin/
        __init__.py
        atm.py
    conf/
        __init__.py
        settings.py
    logs/
```

```python
# atm/main.py
def main():
    print('This is main() from main.py')


if __name__ == '__main__':
    main()
```

```python
# bin/atm.py, 这个要调用主入口main.py
import os, sys

# print(__file__)  # Atm/bin/atm.py, PyCharm里面显示的是绝对路径
# print(os.path.abspath(__file__))
# print(os.path.dirname(os.path.abspath(__file__)))
# print(os.path.dirname(os.path.dirname(os.path.abspath(__file__))))

BASE_DIR = os.path.dirname(os.path.dirname(os.path.abspath(__file__)))
sys.path.append(BASE_DIR)

from atm import main

main.main()
```

## Python code stype

[Google code style](https://zh-google-styleguide.readthedocs.io/en/latest/contents/)

`pip install autopep8`
- vscode: **Shift+Alt+F**
- pycharm: **Ctrl+Alt+L**

## Python in environment

```bash
# use python3 in command line instead of python3.6.5
ln -s /usr/bin/python3.6.5 /usr/bin/python3
```

## Python3 encoding

[python3编码之美](https://thief.one/2017/04/18/1/)