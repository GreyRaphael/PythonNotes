# Python Introduction

Tutorial: [Alex](https://www.cnblogs.com/alex3714/articles/5465198.html)

学习Tips:

- 用到才会印象深刻
- 经常精确思维

写汇编语言的工具：RadASM

编译型语言vs解释性语言：

- c,c++是编译型语言，先要编译然后运行
- python是解释性语言，解释器(编译且执行)

python只认可`utf-8`编码的`.py`文本；python要严格对齐(前面多一个空格都不行)

```python
# python函数中'等价于"
# '不能解决换行问题，用'''
print('Hello')
print("Hello")

print('''Hello,
world''')
print("""Hello,
world""")
```

```python
#单行注释

'''
多行
注释
'''
```

python中整除是`//`

python's error

- syntax error
- 运行错误(divide by zero, canot open file)
- 逻辑错误(用错operator...)

```python
#on Linux
import os
os.system("gedit")
os.system("pkill gedit")

#on Windows
import os
os.system("notepad")
os.system("taskkill /f /im notepad.exe")
#windows下结果出现乱码
#File/Setting/Editor/FileEncoding/GBK
```

pycharm安装module:Setting/Project interpreter/+

```bash
#fedora
sudo dnf install python3-tkinter.x86_64
```

```python
import turtle
#
turtle.showturtle()
turtle.write("Hello")
turtle.forward(200)
turtle.right(90)
turtle.color("red")
turtle.forward(200)
turtle.write("Another Text")

#保持窗口
turtle.done()
```

```python
#another python example
import turtle
#draw a square
turtle.showturtle()
#抬起笔
turtle.penup()
turtle.goto(100,100)
#放下笔
turtle.pendown()
turtle.goto(100,-100)
turtle.goto(-100,-100)
turtle.goto(-100,100)
turtle.goto(100,100)

#保持窗口
turtle.done()
```

```python
#画奥运五环
import turtle
#draw 5 circle
turtle.showturtle()
#first one
turtle.penup()
turtle.goto(200,100)
turtle.pendown()
turtle.color("red")
turtle.circle(100)
#second one
turtle.penup()
turtle.goto(0,100)
turtle.pendown()
turtle.color("black")
turtle.circle(100)
#third one
turtle.penup()
turtle.goto(-200,100)
turtle.pendown()
turtle.color("blue")
turtle.circle(100)
#forth one
turtle.penup()
turtle.goto(100,-100)
turtle.pendown()
turtle.color("green")
turtle.circle(100)
#fifth one
turtle.penup()
turtle.goto(-100,-100)
turtle.pendown()
turtle.color("yellow")
turtle.circle(100)
#hold the window
turtle.done()
```

```python
#画正方体
import turtle
#first square
turtle.goto(200,0)
turtle.goto(200,200)
turtle.goto(0,200)
turtle.goto(0,0)
#second square
turtle.penup()
turtle.goto(100,-100)
turtle.pendown()
turtle.goto(100,100)
turtle.goto(-100,100)
turtle.goto(-100,-100)
turtle.goto(100,-100)
#connect two squares
turtle.goto(200,0)
#
turtle.penup()
turtle.goto(100,100)
turtle.pendown()
turtle.goto(200,200)
#
turtle.penup()
turtle.goto(-100,100)
turtle.pendown()
turtle.goto(0,200)
#
turtle.penup()
turtle.goto(-100,-100)
turtle.pendown()
turtle.goto(0,0)

turtle.done()
```

```python
#画五角星
import turtle
for i in range(1,6):
    turtle.forward(200)
    turtle.right(144)


turtle.done()
```

```python
#画六边形
import turtle
for i in range(1,7):
    turtle.forward(100)
    turtle.right(60)
turtle.done()
```

```python
#画多边形
import turtle
turtle.circle(100,steps=6)
turtle.done()
```

```bash
以动手实践为荣 , 以只看不练为耻;
以打印日志为荣 , 以单步跟踪为耻;
以空格缩进为荣 , 以制表缩进为耻;
以单元测试为荣 , 以人工测试为耻;

以模块复用为荣 , 以复制粘贴为耻;
以多态应用为荣 , 以分支判断为耻;
以Pythonic为荣 , 以冗余拖沓为耻;
以总结分享为荣 , 以跪求其解为耻;
```

## token & keyword

```python
import keyword
print(keyword.kwlist)
```

```bash
#output
['False', 'None', 'True', 'and', 'as', 'assert', 'break', 'class', 'continue', 'def', 'del', 'elif', 'else', 'except', 'finally', 'for', 'from', 'global', 'if', 'import', 'in', 'is', 'lambda', 'nonlocal', 'not', 'or', 'pass', 'raise', 'return', 'try', 'while', 'with', 'yield']
```

```python
#input,eval,type,id example
myInput=input("enter a number:")
print(type(myInput))#<class 'str'>
print(type(eval(myInput)))#<class 'int'>
print(id(myInput))#内存地址
```

```python
#变量类型可以不断改变,因为num可以看作是基类指针
num=1
print(type(num))#<class 'int'>
num="grey"
print(type(num))#<class 'str'>
```

```python
num=1
print(type(num))#<class 'int'>
print(id(num))#139885503076032
num="grey"
print(type(num))#<class 'str'>
print(id(num))#139885504426592
#地址赋值
num1=3
num2=3
print(id(num1),id(num2))#139885503076096 139885503076096,地址居然相同
num1=10
print(id(num1),id(num2))#139885503076320 139885503076096
```

```python
#complex example
complex1=1+2j
print(type(complex1),complex1)#<class 'complex'> (1+2j)
#自适应数据大小
num=1
print(type(num))#<class 'int'>
#自适应
num=99999999999999999999999999999999
print(type(num))#<class 'int'>
```

```python
num=1
print(num)
#删除变量
del num
# print(num)#error
```

```python
#连续赋值
num1=num2=10
#对称赋值
num3,num4=1,2
print(num3,num4)#1 2
num5,num6=eval(input("Enter two numbers:"))#要用,间隔
```

```python
import math
print(math.pi*pow(10,2))#314.1592653589793
print(math.pi*10**2)#314.1592653589793
#运算符自动可以拆分多行
print(1+2+3+4+5+6
      +7+8+9+
      10)
#
print("what the \
hell")#what the hell
#end,sep
num1=1
num2=2
print(num1,end="")
print(num2)#12
print(num1,num2,sep="")#12

#多行合成一行代码，用;
num1=1;num2=2;print(num1,end="");print(num2);print(num1,num2,sep="")
```

```python
#convert
data=1.5e3#1500
str1=str(data)
num=1.5
print(int(num))#1
print(round(num))#2
```

```python
#datetime
import datetime
print(datetime.datetime.now())#2018-01-15 01:05:42.130947
print(datetime.datetime.now().date())#2018-01-15
print(datetime.datetime.now().time())#01:06:12.043553
```

```python
num=None
print(num,type(num))#None <class 'NoneType'>
```

```python
#查函数使用
import math
print(dir(math))
help(math.acos)
```

```bash
#output
['__doc__', '__file__', '__loader__', '__name__', '__package__', '__spec__', 'acos', 'acosh', 'asin', 'asinh', 'atan', 'atan2', 'atanh', 'ceil', 'copysign', 'cos', 'cosh', 'degrees', 'e', 'erf', 'erfc', 'exp', 'expm1', 'fabs', 'factorial', 'floor', 'fmod', 'frexp', 'fsum', 'gamma', 'gcd', 'hypot', 'inf', 'isclose', 'isfinite', 'isinf', 'isnan', 'ldexp', 'lgamma', 'log', 'log10', 'log1p', 'log2', 'modf', 'nan', 'pi', 'pow', 'radians', 'sin', 'sinh', 'sqrt', 'tan', 'tanh', 'tau', 'trunc']
```

```python
#字符串的三种风格
#python没有char类型
import os
os.system('calc')
os.system("calc")
os.system('''calc''')#'''可以换行
##
str1="A"
str2="a"
print(ord(str1),ord(str2))#65 97
print(chr(65),chr(97))#A a
##
help(ord)#Return the Unicode code point for a one-character string.
help(chr)#Return a Unicode string of one character with ordinal i; 0 <= i <= 0x10ffff.
str3="我"
print(ord(str3))#25105,16进制为6211
print(chr(25105))#我
print("\u6211")#我,u意味着unicode
##转义字符
os.system("\"C:\Program Files\Shadow Defender\"")
```

```python
#print
print("a"*5)
print(dir(""))#显示所有字符串的方法
```

python中，所有的数据(包括数字、字符串)实际上都是对象，所以有下面的例子

```python
print(type("abc"))
print("abc".upper())
print("abc".find("a"))#0
```

```python
#string format;数据对齐
#format返回值是<class 'str'>
print(format(10.2,"10.3f"))#总宽度为10,小数3位
print(format(10.2,".3f"))#小数3位,右对齐
print(format(10.2,"<.3f"))#小数3位，左对齐
##
print("{0:b}".format(0b10000))#10000
print("{0:o}".format(0b10000))#20
print("{0:d}".format(0b10000))#16
print("{0:x}".format(0b10000))#10
print("{0:b}".format(16))#10000
print("{0:o}".format(16))#20
print("{0:d}".format(16))#16
print("{0:x}".format(16))#10
```

```python
str1="Grey"
str2="grey"
print(str1<str2)#True,根据ASCII
```

```python
# random example
import random
for i in range(1,50):
    num=random.randint(65,90)
    print(chr(num),end='')
```

```python
randrange(0,10)#不包含10
randint(0,10)#包含10
```

```python
num1=10
num2=10
if num1==num2:
    print("same value")
else:
    print("different value")

if num1 is num2:
    print("same address")
else:
    print("different address")

if num1 is not num2:
    print("different address")
else:
    print("same address")
```

```bash
#output
same value
same address
same address
```

```python
#比较的特例，和其他语言不同
print(5>3>1)#True,因为5>3 and 3>1
```

```python
#结果全是4
import random
for i in range(1,100):
    num=random.randint(1,5)
    if 3<num<5:#3<num and num<5
        print(num)
    else:
        pass#什么都不做，空语句，占坑的
```

[pywin32 Download](https://github.com/mhammond/pywin32/releases)

```python
#先安装pywin32
import win32com.client
speaker=win32com.client.Dispatch("SAPI.SPVOICE")#window的发音接口(Easy access)
for i in range(1,10):
    speaker.Speak("I love you")
```

```python
num=1
while num<5:
    print("I love you, "+str(num)+" times")
    num+=1
else:#必定执行
    print("GUN")
```

```python
#关于float的比较要谨慎
import time

num=2.0
while abs(num)>1e-6:
    print(num)
    num-=0.1
    time.sleep(0.5)
```

```python
#内嵌判断
num=10
str1="hello" if num>5 else "James"
print(str1)#hello
```

```python
import os
for i in range(1,10):
    os.system("start notepad")
```

```python
#二元一次方程，穷举
#3*x+4*y=52
x=0
while x<=52//3:
    if (52-3*x)%4==0:
        print("x=",x,"y=",(52-3*x)//4)
    x+=1
```

需要安装pyWin32包，[pywin32 Download](https://github.com/mhammond/pywin32/releases)

```python
#植物大战僵尸，修改内存数据
import win32process #进程模块
import win32con#系统定义
import win32api#调用系统模块
import ctypes#C语言类型
import win32gui #界面

#一个常量，标识最高权限打开一个进程
PROCESS_ALL_ACCESS=(0x000F0000|0x00100000|0xFFF) # |位运算，0x十六进制
window=win32gui.FindWindow("MainWindow","植物大战僵尸中文版")#查找窗体
hid,pid=win32process.GetWindowThreadProcessId(window) #根据窗体抓取进程编号
phand=win32api.OpenProcess(PROCESS_ALL_ACCESS,False,pid)#用最高权限打开进程编号
date=ctypes.c_long()#C语言的整数类型，读取数据
#加载内核模块
mydll=ctypes.windll.LoadLibrary("C:\\Windows\\System32\\kernel32.dll")

#读取内存，  int(phand)打开的进程编号  244866760,内存地址， 写入结果ctypes.byref(date)
#整数4个字节
mydll.ReadProcessMemory(int(phand),244866760,ctypes.byref(date),4,None)#读取内存
print(date.value)
newdata=ctypes.c_long(2048)#设定修改数据为2048
mydll.WriteProcessMemory(int(phand),244866760,  ctypes.byref(newdata),4,None )
```

```python
#植物大战僵尸，自动加血
import win32process #进程模块
import win32con#系统定义
import win32api#调用系统模块
import ctypes#C语言类型
import win32gui #界面
import time

#一个常量，标识最高权限打开一个进程
PROCESS_ALL_ACCESS=(0x000F0000|0x00100000|0xFFF) # |位运算，0x十六进制
window=win32gui.FindWindow("MainWindow","植物大战僵尸中文版")#查找窗体
hid,pid=win32process.GetWindowThreadProcessId(window) #根据窗体抓取进程编号
phand=win32api.OpenProcess(PROCESS_ALL_ACCESS,False,pid)#用最高权限打开进程编号
date=ctypes.c_long()#C语言的整数类型，读取数据
#加载内核模块
mydll=ctypes.windll.LoadLibrary("C:\\Windows\\System32\\kernel32.dll")

while  True:
    mydll.ReadProcessMemory(int(phand), 244866760, ctypes.byref(date), 4, None)  # 读取内存
    print(date.value)
    if  date.value <300:
        newdata = ctypes.c_long(500)  # 设定修改数据为2048
        mydll.WriteProcessMemory(int(phand), 244866760, ctypes.byref(newdata), 4, None)
    time.sleep(1)

#读取内存，  int(phand)打开的进程编号  244866760,内存地址， 写入结果ctypes.byref(date)
#整数4个字节
```

```python
#for example
for i in range(10):
    print(i,end='')

print()

for i in range(0,10):
    print(i,end='')

print()

for i in range(0,10,2):
    print(i,end='')
```

```bash
#output
0123456789
0123456789
02468
```

```python
#for的速度比while快
import time

startTime=time.time()

#跑分代码...
num=0
for i in range(1000000000):#range(integer)里面只能放整数
    num+=1
print(num)

endTime=time.time()
print(endTime-startTime)
```

```python
#定时操作
import time
import os

num=0
while True:
    time.sleep(1)
    print("第"+str(num)+"秒")
    num+=1

    if num==10:
        os.system("start notepad")
    elif num==20:
        os.system("taskkill /f /im notepad.exe")
    else:
        pass
```

```python
for i in range(100):
    print(i)
#i并没有消亡
print(i)#99
```

python循环总结：

- 任何`while`都可以变成`for`，反过来不行
- `for`不能处理float，不能死循环(step不能=0)

```python
import turtle

for i in range(0,300,100):
    for j in range(0,400,100):
        turtle.goto(j,i);
        turtle.pendown()
        turtle.write(str(j)+","+str(i))
    turtle.penup()
turtle.done()
```

```python
# 窗口闪烁
import win32con #定义
import win32gui  #界面
import time  #时间

QQ=win32gui.FindWindow("TXGuiFoundation","QQ")#找出QQ窗体编号
for  num  in  range(120):
    time.sleep(1)
    if num%2==0:
        win32gui.ShowWindow(QQ,win32con.SW_HIDE) #设置隐藏
    else:
        win32gui.ShowWindow(QQ, win32con.SW_SHOW)#设置显示
```

```python
# 窗口移动
import win32gui
import win32con
import time

'''
notepad=win32gui.FindWindow("Notepad","无标题 - 记事本")
for  i  in  range(10):
    for  size  in  range(0,800,10):
        win32gui.SetWindowPos(notepad,  #操作记事本
                          win32con.HWND_TOPMOST, #最上方
                          0, #位置x
                          0,#位置y
                          size,#长度
                          size,#宽度
                          win32con.SWP_SHOWWINDOW)
    for  size  in  range(800,0,-10):
        win32gui.SetWindowPos(notepad,  #操作记事本
                          win32con.HWND_TOPMOST, #最上方
                          0, #位置x
                          0,#位置y
                          size,#长度
                          size,#宽度
                          win32con.SWP_SHOWWINDOW)

'''


wangwang=win32gui.FindWindow("StandardFrame","阿里旺旺")
while True:
    for  size  in  range(0,800,10):
        win32gui.SetWindowPos(wangwang,  #操作记事本
                          win32con.HWND_TOPMOST, #最上方
                          size, #位置x
                          0,#位置y
                          300,#长度
                          300,#宽度
                          win32con.SWP_SHOWWINDOW)
    for  size  in  range(800,0,-10):
        win32gui.SetWindowPos(wangwang,  #操作记事本
                          win32con.HWND_TOPMOST, #最上方
                          size, #位置x
                          0,#位置y
                          300,#长度
                          300,#宽度
                          win32con.SWP_SHOWWINDOW)
```

```python
# 对角线移动
import win32gui
import win32con
import time

'''
notepad=win32gui.FindWindow("Notepad","无标题 - 记事本")
for  i  in  range(10):
    for  size  in  range(0,800,10):
        win32gui.SetWindowPos(notepad,  #操作记事本
                          win32con.HWND_TOPMOST, #最上方
                          0, #位置x
                          0,#位置y
                          size,#长度
                          size,#宽度
                          win32con.SWP_SHOWWINDOW)
    for  size  in  range(800,0,-10):
        win32gui.SetWindowPos(notepad,  #操作记事本
                          win32con.HWND_TOPMOST, #最上方
                          0, #位置x
                          0,#位置y
                          size,#长度
                          size,#宽度
                          win32con.SWP_SHOWWINDOW)
'''


wangwang=win32gui.FindWindow("StandardFrame","阿里旺旺")
while True:
    for  size  in  range(0,800,10):
        win32gui.SetWindowPos(wangwang,  #操作记事本
                          win32con.HWND_TOPMOST, #最上方
                          size, #位置x
                          size*768//1024,#位置y
                          300,#长度
                          300,#宽度
                          win32con.SWP_SHOWWINDOW)
    for  size  in  range(800,0,-10):
        win32gui.SetWindowPos(wangwang,  #操作记事本
                          win32con.HWND_TOPMOST, #最上方
                          size, #位置x
                          size * 768 // 1024,#位置y
                          300,#长度
                          300,#宽度
                          win32con.SWP_SHOWWINDOW)
```

```python
# 窗口画园
import win32gui
import win32con
import time
import math
wangwang=win32gui.FindWindow("StandardFrame","阿里旺旺")

while True:
    SE = 0.0
    while  SE-3.1415926535*2  <0.000001:
        SE+=0.1
        newx = int(400+400*math.cos(SE))
        newy = int(400+400*math.sin(SE))
        win32gui.SetWindowPos(wangwang,  # 操作记事本
                          win32con.HWND_TOPMOST,  # 最上方
                          newx,  # 位置x
                          newy,  # 位置y
                          300,  # 长度
                          300,  # 宽度
                          win32con.SWP_SHOWWINDOW)
    print(SE)
```

```python
# 窗口画口
import win32gui
import win32con
import time
import math
wangwang=win32gui.FindWindow("StandardFrame","阿里旺旺")
while True:
    for  size  in  range(0,800,10):
        win32gui.SetWindowPos(wangwang,  #操作记事本
                          win32con.HWND_TOPMOST, #最上方
                          size, #位置x
                          0,#位置y
                          300,#长度
                          300,#宽度
                          win32con.SWP_SHOWWINDOW)
    for  size  in  range(0,600,10):
        win32gui.SetWindowPos(wangwang,  #操作记事本
                          win32con.HWND_TOPMOST, #最上方
                          800, #位置x
                          size,#位置y
                          300,#长度
                          300,#宽度
                          win32con.SWP_SHOWWINDOW)
    for  size  in  range(800,0,-10):
        win32gui.SetWindowPos(wangwang,  #操作记事本
                          win32con.HWND_TOPMOST, #最上方
                          size, #位置x
                          600,#位置y
                          300,#长度
                          300,#宽度
                          win32con.SWP_SHOWWINDOW)
    for size in range(600, 0, -10):
        win32gui.SetWindowPos(wangwang,  # 操作记事本
                              win32con.HWND_TOPMOST,  # 最上方
                              0,  # 位置x
                            size,#置y
                              300,  # 长度
                              300,  # 宽度
                              win32con.SWP_SHOWWINDOW)
```

循环等价：

- while
- for
- 死循环中的if条件里面加上`break`

```python
#on Linux, open programs sychronically
import os
for i in range(5):
    os.system("gedit")
```

```python
#on Linux, open programs asychronically
import os
for i in range(5):
    os.system("gedit &")
```

```python
#on window, open programs synchronically
import os
for i in range(5):
    os.system("notepad")
```

```python
#on window, open programs asynchronically
import os
for i in range(5):
    os.system("start notepad")
```

for ... else语句中, for中的语句和普通的没有区别，else中的语句会在循环**正常执行完**(即for不是通过`break`跳出而中断的)的情况下执行，while ... else 也是一样