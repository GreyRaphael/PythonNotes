# Python function

<!-- TOC -->

- [Python function](#python-function)
    - [what is function](#what-is-function)
    - [Function introduction](#function-introduction)
        - [性能比较](#性能比较)
    - [variables](#variables)
    - [`*args` and `**kwargs`](#args-and-kwargs)
    - [Lambda expression](#lambda-expression)
    - [Variable length function](#variable-length-function)
    - [string of python](#string-of-python)
    - [`list`](#list)
    - [`tuple`](#tuple)
    - [`set`](#set)
    - [`dict`](#dict)
    - [summary](#summary)
    - [Convert](#convert)
    - [variables scope](#variables-scope)
    - [built-in function](#built-in-function)
        - [`bytearray`](#bytearray)
        - [`compile`](#compile)
        - [`filter`](#filter)
        - [`sorted`](#sorted)
        - [`zip`](#zip)
        - [`__import__`](#__import__)

<!-- /TOC -->

## what is function

三种编程范式:
- OOP,Object Oriented Programming: `class`
- POP,Procedure Oriented Programming: `def`, 没有返回值, 连None也没有
- FP, Functional Programming: `def`, 有返回值, 但不仅限`return`

```python
def func1():
    print('func1')
    return 0

def func2():
    print('func2')

res1=func1()
res2=func2()
print(res1, res2) # 0 None
# python中看起来是面向过程，其实是函数式，因为有返回值None
```

```python
# python2 cmp(x, y) deprecated
def my_cmp(x, y):
    return (x>y)-(x<y)

print(my_cmp(1, 2)) # -1
print(my_cmp(2, 1)) # 1
```

## Function introduction

```python
#get the max by for
import random
myList=[]
for i in range(10):
    myList.append(random.randint(1,100))
print(myList)

myMax=myList[0]
for i in range(1,10):
    if myList[i]>myMax:
        myMax=myList[i]

print(myMax)
```

```python
#get the second max
import random
myList=[]
for i in range(10):
    myList.append(random.randint(1,100))
# print(myList)
myMax=myList[0]
mySecondMax=myList[1]
if myMax<mySecondMax:
    myMax,mySecondMax=mySecondMax,myMax
# 也就三个区间，比myMax大，中间，比mySecondmax小
for i in range(2,10):
    if myList[i]>myMax:
        mySecondMax=myMax
        myMax=myList[i]
    else:
        if myList[i]>mySecondMax:
            mySecondMax=myList[i]

print(myMax,mySecondMax)
print(sorted(myList))
```

```python
#get the third max
import random
myList=[]
for i in range(10):
    myList.append(random.randint(1,100))

#initialize first 3 max
myMax=myList[0]
mySecondMax=myList[1]
myThirdMax=myList[2]

#assure the first 3 order
if myMax<mySecondMax:
    myMax,mySecondMax=mySecondMax,myMax
if myMax<myThirdMax:
    myMax,myThirdMax=myThirdMax,myMax
if mySecondMax<myThirdMax:
    mySecondMax,myThirdMax=myThirdMax,mySecondMax

#loop to get the first three
for i in range(3,10):
    if myList[i]>myMax:
        myThirdMax=mySecondMax
        mySecondMax=myMax
        myMax=myList[i]
    elif myList[i]>mySecondMax:
        myThirdMax=mySecondMax
        mySecondMax=myList[i]
    elif myList[i]>myThirdMax:
        myThirdMax=myList[i]
    else:
        pass

print(sorted(myList,reverse=True))
print(myMax,mySecondMax,myThirdMax)
```

```python
#画围棋棋盘，并且落10个子
import random
import turtle
#horizontal line
turtle.speed("fastest")
turtle.setup(1080,1080)
turtle.penup()
for i in range(-400,400,80):
    for j in range(-400,400,80):
        turtle.goto(j,i)
        turtle.pendown()
        turtle.write("("+str(j)+","+str(i)+")")
    turtle.penup()

#vertical line
for i in range(-400,400,80):
    for j in range(-400,400,80):
        turtle.goto(i,j)
        turtle.pendown()
    turtle.penup()

#draw dot,size=10,color="red"
myList=[x for x in range(-400,400,80)]
myDotsX=myList.copy()
myDotsY=myList.copy()
random.shuffle(myDotsX)
random.shuffle(myDotsY)
for i in range(10):
    turtle.goto(myDotsX[i],myDotsY[i])
    turtle.dot(20,"red")

print(myDotsX)
print(myDotsY)

turtle.done() 
```

```python
#画国际象棋棋盘
import turtle
step=50
turtle.speed("fastest")
turtle.penup()
for i in range(8):
    for j in range(8):
        turtle.goto(i*step,j*step)
        turtle.pendown()
        # choose color
        if (i+j)%2==0:
            turtle.color("black")
        else:
            turtle.color("white")
        # fill the small rectangle
        turtle.begin_fill()
        for k in range(4):
            turtle.forward(step)
            turtle.right(90)
        turtle.end_fill()
    turtle.penup()
# draw the border
turtle.goto(0,-step)
turtle.pendown()
for i in range(4):
    turtle.forward(step*8)
    turtle.left(90)
turtle.done()
```

### 性能比较

```python
# 判断质数
import time

#is prime,method1
def IsPrime_1(num:int):
    if num<2:
        return False
    if num==2 or num==3:
        # print(num,end=' ')
        return True
    for i in range(2,int(num**0.5)+1):
        if num%i==0:
            return False
    # print(num,end=' ')
    return True
#is prime, method2

def IsPrime_2(num:int):
    if num<2:
        return False
    if num==2 or num==3 or num==5:
        # print(num,end=' ')
        return True
    # 不在6的倍数两侧的一定不是质数(孪生素数)
    if num%6!=1 and num%6!=5:
        return False
    for i in range(5,int(num**0.5)+1,6):
        if num%i==0 or num%(i+2)==0:
            return False
    # print(num,end=' ')
    return True

#compare two model
start_time1=time.time()
for i in range(5000000):
    IsPrime_1(i)
print(time.time()-start_time1)#38.62s

start_time2=time.time()
for i in range(5000000):
    IsPrime_2(i)
print(time.time()-start_time2)#15.08s
```

```c
//c,1359
#include <stdio.h>
#include <stdlib.h>
#include <time.h>
#include <math.h>

int IsPrime_2(unsigned num)
{
    if( num  <2 ){
        return 0;
    }
    if (num == 2 || num == 3)
    {
        //printf("%d ", num);
        return 1;
    }
    if (num % 6 != 1 && num % 6 != 5)
    {
        return 0;
    }
    for (size_t i = 5; i <= sqrt(num); i += 6)
    {
        if (num%i == 0 || num % (i + 2) == 0)
        {
            return 0;
        }
    }
    //printf("%d ", num);
    return 1;
}

int main()
{
    clock_t begin = clock();
    for (size_t i = 0; i < 5000000; i++)
    {
        IsPrime_2(i);
    }
    clock_t end = clock();
    printf("%d\n", end - begin);
    system("pause");
}

```

```cpp
//cpp, 3341
#include <iostream>
#include <ctime>

bool IsPrime_2(unsigned num) {
    if(num<2){
        return false;
    }
    if (num==2||num==3)
    {
        //std::cout << num << " ";
        return true;
    }
    if (num%6!=1 && num%6!=5)
    {
        return false;
    }
    for (size_t i = 5; i <= sqrt(num); i+=6)
    {
        if (num%i==0||num%(i+2)==0)
        {
            return false;
        }
    }
    //std::cout << num << " ";
    return true;
}

int main() {
    clock_t begin = clock();
    for (size_t i = 0; i < 5000000; i++)
    {
        IsPrime_2(i);
    }
    clock_t end = clock();
    std::cout << end - begin << "\n";
    std::system("pause");
}
```

```csharp
//c#, 1076
using System;

namespace ConsoleApp2
{
    class Program
    {
        static void Main(string[] args)
        {
            var watch = System.Diagnostics.Stopwatch.StartNew();
            for (int i = 0; i < 5000000; i++) {
                IsPrime_2(i);
            }
            watch.Stop();
            Console.WriteLine(watch.ElapsedMilliseconds);
        }

        static bool IsPrime_2(int num) {
            if (num < 2) {
                return false;
            }
            if (num == 2 || num == 3) {
                //Console.Write(num + " ");
                return true;
            }
            if (num % 6 != 1 && num % 6 != 5) {
                return false;
            }
            for (int i = 5; i <= Math.Sqrt(num); i += 6) {
                if (num % i == 0 || num%(i+2)==0  ) {
                    return false;
                }
            }
            //Console.Write(num + " ");
            return true;
        }
    }
}
```

```go
//go, 1016
package main

import "fmt"
import "math"
import "time"

func IsPrime(num uint32) bool {
    if num < 2 {
        return false
    }
    if num == 2 || num == 3 {
        // fmt.Printf("%d ", num)
        return true
    }
    if num%6 != 1 && num%6 != 5 {
        return false
    }
    for index := uint32(5); index <= uint32(math.Sqrt(float64(num))); index += 6 {
        if num%index == 0 || num%(index+2) == 0 {
            return false
        }
    }
    // fmt.Printf("%d ", num)
    return true
}

func main() {
    begin := time.Now()
    for index := uint32(0); index < 5000000; index++ {
        IsPrime(index)
    }
    fmt.Println(time.Since(begin))
}
```

```python
## function example
import time

#method1
def IsPrime(num:int):
    if num<2:
        return False
    if num==2 or num==3:
        # print(num,end=' ')
        return True
    for i in range(2,int(num**0.5)+1):
        if num%i==0:
            return False
    # print(num,end=' ')
    return True

#3x+4y=data,x is prime, y is prime
def SumOfTwoPrime(data):
    for x in range(data//3):
        if (data-3*x)%4==0:
            y=(data-3*x)//4
            if IsPrime(x) and IsPrime(y):
                print("3*%d+4*%d=%d"%(x,y,data))
                return True
    else:
        return False

#3x+4y+5z=data,x is prime, y is prime, z is prime
def SumOfThreePrime(data):
    for x in range(data//3):
        for y in range((data-3*x)//4):
            if (data-3*x-4*y)%5==0:
                z=(data-3*x-4*y)//5
                if IsPrime(x) and IsPrime(y) and IsPrime(z):
                    print("3*%d+4*%d+5*%d=%d"%(x,y,z,data))
                    return True
    else:
        return False

for i in range(100,110):
    SumOfTwoPrime(i)

for i in range(100,110):
    SumOfThreePrime(i)
```

## variables

```python
# exchange variables
a=10
b=20

#method1
c=a
a=b
b=c
print(a,b)#20 10

#method2
a,b=b,a
print(a,b)#10 20

#method3
a=a+b
b=a-b
a=a-b
print(a,b)#20 10
```

```python
def MyAdd(a,b):
    return a+b

def MySub(a,b):
    return a-b

def MyMul(a,b):
    return a*b

print(type(MyAdd),id(MyAdd))
#函数变量
go=MyAdd
print(type(go),id(go))
print(go(10.2,5))
go=MySub
print(type(go),id(go))
print(go(10.2,5))
go=MyMul
print(type(go),id(go))
print(go(10.2,5))
```

```bash
#output
<class 'function'> 1574081192144
<class 'function'> 1574081192144
15.2
<class 'function'> 1574081192280
5.199999999999999
<class 'function'> 1574081192416
51.0
```

```python
#python接口
def Test(func,num1,num2):##业务的核心代码
    return func(num1,num2)

def MyAdd(a,b):#业务的逻辑
    return a+b

def MySub(a,b):
    return a-b

def MyMul(a,b):
    return a*b

print(Test(MyAdd,10.2,5))
print(Test(MySub,10.2,5))
print(Test(MyMul,10.2,5))
```

```python
# another example
#测试代码时间,装饰器模式
import time
def GetCostTime(func,data):
    start_time=time.time()
    func(data)
    end_time=time.time()
    print(end_time-start_time)

def go(data):
    res=0
    for i in range(data):
        res+=i


GetCostTime(go,100000000)
```

没有`return`；有`return`但是没有返回数据。默认返回`None`

```python
def show(num1,num2,num3):
    print(num1,num2,num3)

show(1,2,3)
show(num3=3,num1=1,num2=2)
```

基本数据类型，当作参数传递进入函数内部；原来的数据不变(查看`id()`)

```python
def Example(data1,data2):
    return data1+data2,data1-data2,data1*data2

a,b,c=Example(10,20)
print(a,b,c)#30 -10 200
```

```python
#函数的定义可以嵌套,怀孕函数

def go(a,b):
    def MyAdd(a,b):#业务的逻辑
        return a+b
    def MySub(a,b):
        return a-b
    def MyMul(a,b):
        return a*b
    print(MyAdd(a,b),MySub(a,b),MyMul(a,b))

go(10,20)
```

```python
#如果不修改fix

def go(a,b):
    fix=10
    def MyAdd(a,b):
        # nonlocal fix#默认就是这个
        return a+b+fix
    def MySub(a,b):
        # nonlocal fix
        return a-b+fix
    def MyMul(a,b):
        # nonlocal fix
        return a*b+fix
    print(MyAdd(a,b),MySub(a,b),MyMul(a,b))

go(10,20)#40 0 210
```

```python
#如果修改fix
def go(a,b):
    fix=10
    def MyAdd(a,b):
        nonlocal fix
        fix=100
        return a+b+fix
    def MySub(a,b):
        # nonlocal fix
        return a-b+fix
    def MyMul(a,b):
        # nonlocal fix
        return a*b+fix
    print(MyAdd(a,b),MySub(a,b),MyMul(a,b))

go(10,20)#130 90 300
```

```python
# 开房查询

#字符串检索，
#print("hello".find("el")) #find函数找到返回位置
#print("hello".find("ak"))

import codecs  #编码
#第一个参数路径，第二个参数，rb二进制读写 第三个参数汉字编码，第四个参数忽略错误
file=codecs.open("kaifangX.txt","rb","gbk","ignore")

while True:
    mystr = input("输入要查询的数据")
    while True:
        linestr=file.readline()#读取一行
        if linestr.find(mystr)!=-1:
            print(linestr,end="") #显示数据
        if linestr== None: #读取失败返回值为None
            break
```

```python
import os
print(type(print),id(print))#<class 'builtin_function_or_method'> 1667693182840
print(type(os.system),id(os.system))#<class 'builtin_function_or_method'> 2188833368032
```

```python
import os

def MyAdd(num1,num2):
    return num1+num2

def MySub(num1,num2):
    return num1*num2

p=MyAdd
print(p(10,20))#30
p=MySub
print(p(10,20))#200

#下面这个及其不规范，把一个函数赋值给另一个
#即便是built-in-function，也不能幸免
os.system=print
os.system("calc")#打印calc，而不是执行
```

```python
# 劫持原理
# 收费软件也是这个原理
import os
def FilterPrint(string):
    if string.find("goodProgram")!=-1:
        newStr=string[12:]
        os.system("echo "+newStr)
    else:
        pass

print=FilterPrint#替换系统的print
print("Hello,world")#不被流氓360认可的程序，不让通行
print("goodProgram Hello, world")#Hello, world
```

```python
# 假设os.system是收费的
import os

RealOsSytem=os.system

def IsPaided(myStr):
    if myStr.find("paid")!=-1:
        newStr=myStr[4:]
        RealOsSytem(newStr)
    else:
        print("Please pay for this software")

os.system=IsPaided
os.system("paid calc")
```

```python
import os
def go(data):
    print(data)

print(type(os.system),id(os.system),os.system)#<class 'builtin_function_or_method'> 2220510783456 <built-in function system>
print(type(go),id(go),go)#<class 'function'> 2220544024784 <function go at 0x0000020502BCE0D0>
```

```python
#默认参数与位置参数
def show(num1,num2=2,num3=3):#默认参数是从右往左，函数调用的参数是从左往右要覆盖到有默认参数的位置
    print(num1,num2,num3)

show(10)
show(10,20)#从左往右
show(10,num2=10)
show(10,num3=30))#混合填充，位置参数可以不从左往右，而且没有分配的必须放在左边
#错误情况，混合填充不能相间隔
# show(10,num2=10,10)
```

## `*args` and `**kwargs`

```python
# args
def func1(*args):
    print(args)

myList=[1, 2, 3, 4]
func1(1, 2, 3, 4)

func1(myList) # 这里传入的是一个list, ([1, 2, 3, 4])
func1(*myList) # 这里传入的是一个4个变量, (1, 2, 3, 4)

# kwargs
def func2(**kwargs):
    print(kwargs)

myDict={'name':'grey', 'age':23}
func2(name='grey', age=23)

# func2(myDict) # error
func2(**myDict) # {'name': 'grey', 'age': 23}

# args, kwargs
def func3(*args, **kwargs):
    print(f'args: {args}')
    print(f'kwargs: {kwargs}')

func3(1, 2, name='grey', age=23)

myList=[11, 22, 33]
myDict={'sex':'male', 'weight':50}
func3(*myList, **myDict)

# 形参顺序
def func4(x, y, a=100, b=200, *args, **kwargs):
    print(f'args: {args}')
    print(f'kwargs: {kwargs}')

# 注意
def func5(x, a=100, **kwargs):
    print(f'kwargs: {kwargs}', f'a={a}')

func5(10, name='grey', age=10)
func5(10, a=200, name='grey', age=10)
# 不规范写法，虽然可以运行
func5(10, name='grey', age=10, a=111)
func5(10, name='grey', a=222, age=10)
```

## Lambda expression

```python
#simple examle
(lambda myStr: print(myStr))("Hello, Lambda")
```

```python
#Lambda
myAdd=lambda a,b: a+b
print(myAdd(10,20))# 30
print((lambda a,b: a*b)(10,20))# 200
```

```python
#lambda as parameter
def MyInterface(func,num1,num2):
    return func(num1,num2)

print(MyInterface(lambda a,b: a+b,10,20))
print(MyInterface(lambda a,b: a-b,10,20))
print(MyInterface(lambda a,b: a*b,10,20))
```

## Variable length function

```python
def MySum(*sequence):
    res=0
    for i in sequence:#sequence is a tuple
        res+=i
    return res

print(MySum(1,2,3))#6
print(MySum(1,2,3,4))#10
```

## string of python

字符串变量存的是字符串常量的地址，字符串本质是常量(字符串的每一个字符不可以修改，但是可以让字符串变量跳转到其他字符串)

```python
#字符串截取
for ch in "Grey":#len("Grey")is 4
    print(ch)
print("grey"[2])#e
print("grey"[1:3])#re
print("grey"[:3])#gre
#positive index from 0,1,2,3
#negative index from -1,-2,-3
print("grey"[0:-1])#gre
print("grey"[1:])#rey
print("grey"[:])#grey
```

```python
#字符串copy
print("grey"[1:3]*3)#rerere
print("grey"+"hello")
```

## `list`

很多操作类似字符串

```python
#simple exmaple
myList=[]
for i in range(10):
    myList.append(i)

print(len(myList))
for item in myList:
    print(item,end=' ')#0 1 2 3 4 5 6 7 8 9
print()
for item in myList[1:8]:#不包含8
    print(item,end=' ')#1 2 3 4 5 6 7 
print(myList*2)#[0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
print(myList+myList[0:3])#[0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 0, 1, 2]
#List是变量，可以被修改
myList[0]=111
print(myList)#[111, 1, 2, 3, 4, 5, 6, 7, 8, 9]

for i in range(len(myList)):
    print(myList[i],end=',')#111,1,2,3,4,5,6,7,8,9,
```

```python
# 开房List查询,先全部载入内存，然后利用List来处理

import codecs  #编码
file=codecs.open("kaifangX.txt","rb","gbk","ignore")
listname=file.readlines() #多行,返回list
print(type(listname)) #打印类型
print(len(listname))#求行数
while True:
    name=input("输入要查询的负心人")
    i=0
    for  line  in listname: #挨个搜索
        if  line.find(name)!=-1:
            i+=1
            print(line)
    print("一共"+str(i)+"条")
```

## `tuple`

tuple几乎具备List所有功能，就是不能修改内部的值，但是可以跳到新的tuple;

为了数据安全，比如刚才的开房数据查询

```python
myTuple=(1,2,3,)
myTuple=(8,9,10)
print(myTuple)#(8,9,10)
```

```python
myTuple=(0,1,2,3,4,5,6,7,8,9)
for item in myTuple:
    print(item,end=' ')#0 1 2 3 4 5 6 7 8 9 
print()
for item in myTuple[1:8]:
    print(item,end=' ')#1 2 3 4 5 6 7 
print()
print(myTuple*2)#(0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 0, 1, 2, 3, 4, 5, 6, 7, 8, 9)
print(myTuple+myTuple[0:3])#(0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 0, 1, 2)
for i in range(len(myTuple)):
    print(myTuple[i],end=',')#0,1,2,3,4,5,6,7,8,9,
```

```python
myTuple=(1)
print(type(myTuple))#<class 'int'>
myTuple=(1,)
print(type(myTuple))#<class 'tuple'>
```

## `set`

采用的是hashtable

```python
myList=[1,1,2,3]
myTuple=(1,1,2,3)
mySet={1,1,2,3,}
print(myList)#[1, 1, 2, 3]
print(myTuple)#(1, 1, 2, 3)
print(mySet,type(mySet))#数据没有重复的.{1, 2, 3} <class 'set'>
```

set operation:交集、并集、差集、并集-差集(`& | - ^`)

用于数据的串联，比如kaifang数据和dangdang数据进行串联

set不支持索引

```python
mySet1={1,2,3,4,5}
mySet2={3,4,5,6,7}
print(mySet1&mySet2)
print(mySet1|mySet2)
print(mySet1-mySet2)
print(mySet2-mySet1)
print(mySet1^mySet2)
```

```bash
#output
{3, 4, 5}
{1, 2, 3, 4, 5, 6, 7}
{1, 2}
{6, 7}
{1, 2, 6, 7}
```

## `dict`

key-value

```python
#dictionary是set的强化版，key不允许重复，value允许重复()
myDict1={"a":1,"b":2,"c":3,"c":4,"d":5}

print(myDict1,type(myDict1))

print(myDict1["a"])#1

for key in myDict1:
    print(key,myDict1[key])

print(myDict1.keys())
print(myDict1.values())
print(myDict1.keys)
print(myDict1.values)
```

```bash
#output
{'a': 1, 'b': 2, 'c': 4, 'd': 5} <class 'dict'>
1
a 1
b 2
c 4
d 5
dict_keys(['a', 'b', 'c', 'd'])
dict_values([1, 2, 4, 5])
<built-in method keys of dict object at 0x00000231B3B7CBD0>
<built-in method values of dict object at 0x00000231B3B7CBD0>
```

## summary

- list和tuple可以用下标循环，也可以用元素循环
- set,dictionary只能元素循环

```python
#list,tuple,set,dictionary都有in,not in
myList=[]
for i in range(10):
    myList.append(i)
myTuple=tuple(myList)
myAnotherList=list(myTuple)
mySet=set(myList)

print(8 in myList)#True
print(10 not in myTuple)#True
print(2 in mySet)#True
```

```python
#list,tuple,set,dictionary的构造表达式
myList=[x for x in range(10)]#列表构造表达式
myTuple=((x**2 for x in range(10)),(x**0.5 for x in range(10)))
mySet={x**3 for x in range(10)}

print(myList)
print(myTuple)
print(mySet)
print(type(x**0.5 for x in range(10)))
```

```bash
#output
[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
(<generator object <genexpr> at 0x0000023B5C7FA3B8>, <generator object <genexpr> at 0x0000023B5C7FA410>)
{0, 1, 64, 512, 8, 343, 216, 729, 27, 125}
<class 'generator'>
```

```python
myList1=[x for x in range(10)]
myList2=[x**2 for x in range(3)]
myTuple=(myList1,myList2)

for item in myTuple:
    print(item)
```

```bash
#output
[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
[0, 1, 4]
```

## Convert

```python
#按照进制转换
print(int("16",16))#22
print(int("16",10))#16
print(int("16",8))#14=1*8+6
print(hex(100))#0x64
print(oct(100))#0o144
print(bin(100))#0b1100100
```

```python
#frozen list
myList=[x for x in range(10)]
myList2=frozenset(myList)
print(myList)
print(type(myList2),myList2)
```

```bash
#output
[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
<class 'frozenset'> frozenset({0, 1, 2, 3, 4, 5, 6, 7, 8, 9})
```

内存1s可以检索1G的数据，SSD也可以500M/s

## variables scope

Python 中只有`module`, `class`, `def`, `lambda`才会引入新的作用域，其它的代码块(`if...elif...else`、`try...except..`、`for`, `while`, `with`, etc)是不会引入新的作用域的，也就是说这这些语句内定义的变量，外部也可以访问

```python
if True:
    msg='inner message'
print(msg)# inner message

def func1():
    num=100
print(num) # error
```

Python 使用 LEGB 的顺序来查找一个符号对应的对象：

`locals -> enclosing function -> globals -> builtins`

```python
x=int(2.9)# built-ins: int

g_count=0 # global
def outer():
    e_count=1 # enclosing
    def inner():
        l_count=2 # local
```

要修改全局变量要用`global`;要修改闭包外部的非全局的变量要用`nonlocal`; **谨慎使用**

一般都是将mutable的类型(list, dict, set)作为函数参数，然后再函数内修改; 而immutable类型(number, str, tuple)只能借助`global`修改, 然而并不推荐

```python
# local vs global
num=100

def go(data):
    num=data
    print(num,id(num))#200 1508935360 

go(200)
print(num,id(num))#100 1508932160
```

```python
# attention
num=100

def go():
    print(num) # error，python会自动扫描一遍，认为num是局部变量,而num屏蔽全局的num, 然而并没有给num的数值，所以会报错
    num=200
    print(num)

go()
```

```python
# 修改全局变量
num=100

def go():
    global num
    num=200
    print(num,id(num))#200 1508935360

go()
print(num,id(num))#200 1508935360
```

```python
# 不规范的做法
def func1():
    global name
    name='grey'

func1()
print(name)
```

```python
# 修改enclosing
def outer():
    num=10
    def inner():
        nonlocal num
        num=100
    inner()
    print(num)

outer() # 100
```

```python
# global is readonly
a = 10
def test():
    a = a + 1 # error, 将左边换成b就不会报错
    print(a)
test()
```

## built-in function

[built-ins](https://docs.python.org/3/library/functions.html)

### `bytearray`

字符串不可修改，然而bytearray是修改的

```python
arr1=bytearray('abc', encoding='utf8')
print(arr1[0])#97
arr1[0]=98
print(arr1)# bytearray(b'bbc')
```

### `compile`

简单表达式用`eval`, 复杂表达式(循环、判断)用`exec`

```python
code='1+2**10'
code_obj=compile(code, '', 'eval')
eval(code_obj)
```

```python
code="""
for i in range(10):
    print(f'---{i}---')
    print(f'***{i}***')"""
code_obj=compile(code, '', 'exec')
exec(code_obj)
```

也可以直接执行: 用于网络传递代码, 并动态执行代码, 不用`import`, `import`需要是在本地

```python
eval('1+2**10)

code="""
for i in range(10):
    print(f'---{i}---')
    print(f'***{i}***')"""
exec(code)
```

### `filter`

python的lambda只能处理简单的语句， 循环和判断都不行，判断只能用三元运算符;

```python
myList1=[x for x in range(10)]
filter1=filter(lambda x: x**2>9, myList1)
print(type(filter1))

for item in filter1:
    print(item, item**2)
```

```python
myList1=[x for x in range(5)]
myMap=map(lambda x:x**2, myList1) # type(myMap, map)==True

print(myList1)# [0, 1, 2, 3, 4]
print([i for i in myMap])# [0, 1, 4, 9, 16]
```

```python
import functools

myList1=[1, 2, 3, 4, 5]
res=functools.reduce(lambda x, y:x*y, myList1)
print(res)# 120
```

### `sorted`

dict默认是无序的，如果要排序，先要变成list

```python
dict1={4:'grey', 11:'alpha', 6:'beta', 1:'gamma'}

print(sorted(dict1.items()))
print(sorted(dict1.items(), reverse=True))
print(sorted(dict1.items(), key=lambda x: x[1] , reverse=True))
print(dict1)
```

```bash
# res
[(1, 'gamma'), (4, 'grey'), (6, 'beta'), (11, 'alpha')]
[(11, 'alpha'), (6, 'beta'), (4, 'grey'), (1, 'gamma')]
[(4, 'grey'), (1, 'gamma'), (6, 'beta'), (11, 'alpha')]
{4: 'grey', 11: 'alpha', 6: 'beta', 1: 'gamma'}
```

iterable排序有两种方式:
- 内建的`.sort()`
- `sorted()`

```python
# example1: number
# sorted不改变原数据
L1=[36, 5, 12, -9, 21]
print(sorted(L1, reverse=True)) # [36, 21, 12, 5, -9]
print(sorted(L1, key=lambda x: x**2)) # [5, -9, 12, 21, 36]

# .sort改变原数据
L1.sort(reverse=True)
print(L1) # [36, 21, 12, 5, -9]

# example2: string
L2=['bob', 'about', 'Zoo', 'Credit']
print(sorted(L2)) # 根据ASCII排序，['Credit', 'Zoo', 'about', 'bob']
print(sorted(L2, key=str.lower)) # ['about', 'bob', 'Credit', 'Zoo']

# example3: tuple
L3=[('b', 6), ('a', 1), ('c', 3), ('d', 4), ('a', 7)]
print(sorted(L3)) # [('a', 1), ('a', 7), ('b', 6), ('c', 3), ('d', 4)]
print(sorted(L3, key=lambda x: x[1])) # [('a', 1), ('c', 3), ('d', 4), ('b', 6), ('a', 7)]

# example4: tuple with itemgetter
import operator
print(sorted(L3, key=operator.itemgetter(1))) # [('a', 1), ('c', 3), ('d', 4), ('b', 6), ('a', 7)]

# examle5: 多个关键字排序，先按照其中一个排序，相同的话按照另一个排序
print(sorted(L3, key=lambda x: (x[1], x[0]))) # [('a', 1), ('c', 3), ('d', 4), ('b', 6), ('a', 7)]
print(sorted(L3, key=operator.itemgetter(1, 0))) # [('a', 1), ('c', 3), ('d', 4), ('b', 6), ('a', 7)]
```

### `zip`

zip的两个参数，长度可以不一致

```python
def func1(*args):
    print(args)

x=[1, 2, 3]
y=[4, 5, 6]
print(*zip(x, y))# (1, 4) (2, 5) (3, 6)
func1(*zip(x, y))# ((1, 4),(2, 5),(3, 6))

for item in zip(x, y):
    print(item)
```

```python
myList1=[1, 2, 3]
myList2=[4, 5, 6]
# 使用map实现zip
map1=map(lambda x, y:(x, y), myList1, myList2)
for item in map1:
    print(item)

print(*map(lambda x, y:(x, y), myList1, myList2))
```

```bash
# res
(1, 4)
(2, 5)
(3, 6)

(1, 4) (2, 5) (3, 6)
```
​
```python
l1 = [1, 2, 3, 4]
l2 = ['a', 'b', 'c', 'd']
# zip只能用一次
for i in zip(l1, l2):
    print(i)

# zip得到原序列
print('='*20)
zip1 = zip(l1,l2)
for item in zip(*zip1):
    print(item)

# zip构造字典
print(dict(l1, l2))
```

```bash
# res
(1, 'a')
(2, 'b')
(3, 'c')
(4, 'd')
====================
(1, 2, 3, 4)
('a', 'b', 'c', 'd')

{1: 'a', 2: 'b', 3: 'c', 4: 'd'}
```

### `__import__`

通常在动态加载时可以使用到这个函数，比如你希望加载某个文件夹下的所用模块，但是其下的模块名称又会经常变化时，就可以使用这个函数动态加载所有模块了