# Python Environment

- [Python Environment](#python-environment)
    - [window环境变量](#window%E7%8E%AF%E5%A2%83%E5%8F%98%E9%87%8F)
    - [Anaconda](#anaconda)
    - [Jython](#jython)
    - [winpython](#winpython)
    - [py2exe](#py2exe)
    - [pywin32](#pywin32)
    - [wmi(Windows Management Instrumentation)](#wmiwindows-management-instrumentation)
    - [CGI](#cgi)
    - [深度学习](#%E6%B7%B1%E5%BA%A6%E5%AD%A6%E4%B9%A0)

尹成python:

- 环境地址
链接:http://yun.baidu.com/s/1hs2yXuc 密码:vhy9
- 基础视频地址
http://pan.baidu.com/s/1c2lFQww

## window环境变量

环境变量的意义在于可以在cmd中不用输入路径就打开一个外部的exe(比如`firefox`)

## Anaconda

机器学习要用到anaconda工具(最好用64bit),也可以用winpython

安装anaconda完毕之后，在pyCharm中选择Anaconda的python，然后会自动添加Anaconda的包到pycharm的External Liberaries中

Anaconda中`conda update --all`遇到问题，看看结果，一般

```bash
#先安装ipykernel
conda update ipykernel
#then
conda update --all
```

## Jython

- CPython是Python语言在C中的完全实现(官方版本);
- Jython是Python语言在Java中的完全实现;
- IronPython是Python语言在C#中的完全实现;
- Pyston是Python语言在C++中的完全实现;

首先需要安装JDK,将`jdk/bin`添加进入环境变量(即cmd中的输入`java`,`javac`要有效)

一般情况：

变量名|变量值
---|---
JAVA_HOME|JDK安装路径
CLASSPATH|%JAVA_HOME%\lib
path|%JAVA_HOME%\bin

然后`java -jar jython-installer-2.7.0.jar`进行安装

最后将`jython\bin`添加大环境变量

## winpython

Datascience,science: Windows下建议使用
[WinPython](https://sourceforge.net/projects/winpython/)
,Linux下则使用Anaconda(更加强大，而且跨平台)

winpython，它轻巧，包含了常用的科学计算工具包`numpy`，`scipy`，`sklearn`，`matplotlib`，还有可以调用C动态库的扩展包`ctypes`

Scipy是一个高级的科学计算库，它和Numpy联系很密切，Scipy一般都是操控Numpy数组来进行科学计算，所以可以说是基于Numpy之上了。Scipy有很多子模块可以应对不同的应用，例如插值运算，优化算法、图像处理、数学统计等。

winpython实际上是整合了IDE工具spyder和一些科学计算包，默认包含了以下工具包，有了这些工具包，完全可以替代MATLAB做科学计算：

- numpy，scipy： 数值计算工具包，里面包含了各种矩阵算，MATLAB有的，它基本上都有。不过，里面有array和matrix两种类型，最好是用array类型的，因为它的功能最全，大部分函数处理的类型都是array。scipy实际上包含了numpy的功能，并且还有2D绘图子工具包pylab，里面的plot用法很像matlab的。scipy里有各种最优化算法，比如约束最优化，非约束最优化等等，它的官方在线文档： http://docs.scipy.org/doc/scipy/reference/
- matplotlib： 2D和3D绘图工具，绘图功能强大，各种数据可视化表现方式，没有做不到的，只有你想不到的。
- sklearn： 各种学习算法，聚类算法都在里面，比如svm，k-means，KNN，PCA，随机森林等等一大堆。官方网站： http://scikit-learn.org/stable/
- ctypes： 能使python和c交流的工具包有好几个，但是我认这个最好用，因为，你可以用VS生成一个动态库，而ctypes则可以直接去调用动态库中的函数。当你要处理复杂运算时，用纯粹的python实现出来的会慢的有如世界末日，但是用C实现无疑是最快的办法，而ctypes则可以帮你轻松做到这一点。想想matlab和c的混编，光是数据提取和类型转换就是一堆，估计很多人会有种想死的感觉。由于ctypes实现了python便捷访问c动态库的功能，你会觉得python和c的混编是一件非常轻松快乐的事情。它的方便之处还在于，numpy或scipy的数据成员中是默认包含ctypes的，这使python到c函数的各种数据类型的参数传递变得异常简单。

## py2exe

https://pypi.python.org/pypi/py2exe/0.9.2.0

## pywin32

调用windows自带的软件(text to speech,speech reconition),要用到
[Pywin32 Download](https://sourceforge.net/projects/pywin32/)

## wmi(Windows Management Instrumentation)

https://pypi.python.org/pypi/WMI

获取一些系统的信息(流量、内存、cpu...)

## CGI

download like
[httpd-2.2.25-win32-x86-no_ssl](https://archive.apache.org/dist/httpd/binaries/win32/), 自定义安装所有的东西，填入hostname(localhost,localhost,your email)

在浏览器中输入黑窗口提示的ip(或者直接`127.0.0.1`)

修改`C:\Program Files (x86)\Apache Software Foundation\Apache2.2\htdocs\index.html`的内容，在浏览器中的内容也会跟着修改

## 深度学习

