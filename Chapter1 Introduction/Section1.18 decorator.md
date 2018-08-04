# Python Decorator

<!-- TOC -->

- [Python Decorator](#python-decorator)
    - [Closures(闭包)](#closures闭包)
    - [decorator](#decorator)
    - [decorator with parameters](#decorator-with-parameters)
        - [`property`](#property)
    - [静态语言vs动态语言](#静态语言vs动态语言)
    - [`__slots__`](#__slots__)
    - [元类(metaclass)](#元类metaclass)
        - [`__metaclass__`属性](#__metaclass__属性)
        - [customize metaclass](#customize-metaclass)

<!-- /TOC -->

## Closures(闭包)

```python
def test1():
    print("---test1---")

test1()
test2=test1
print(test1,test2)
test2()
```

```bash
#output
---test1---
<function test1 at 0x000002DB79D6AB70> <function test1 at 0x000002DB79D6AB70>
---test1---
```

在函数内部再定义一个函数，并且这个函数用到了外边函数的变量，那么将这个函数以及用到的一些变量称之为闭包

```python
def test(number):
    print("---1---")
    
    #这就是一个闭包
    def test_in(number_in):
        print("Hello,---2---")
        return number+number_in
    print("---3---")
    return test_in

#闭包的应用，有一个默认的值
ret = test(20)
print(ret(100))
print(ret(200))
```

```bash
---1---
---3---
Hello,---2---
120
Hello,---2---
220
```

```python
def counter(start=0):
    count=[start]
    def incr():
        count[0] += 1
        return count[0]
    return incr

c1=counter(5)
print(c1())#6
print(c1())#7
print(c1())#8
```

```python
def counter(start=0):
    def incr():
        nonlocal start
        start += 1
        return start
    return incr

c1 = counter(5)
print(c1())
print(c1())

c2 = counter(50)
print(c2())
print(c2())

#保存了结果，类似global，可以修改
print(c1())
print(c1())

print(c2())
print(c2())
```

```bash
#output
6
7
51
52
8
9
53
54
```

```python
def line_conf(a, b):
    def line(x):
        return a*x + b
    return line

#内存中开辟了两个空间
line1 = line_conf(1, 1)
line2 = line_conf(4, 5)
print(line1(5))#6
print(line2(5))#25
```

```python
x=0
def grandpa():
    x=1
    def dad():
        x=2
        def son():
            x=3
            print(x)
        son()
    dad()

# 使用的局部变量3
grandpa() # 3
```

Closure summary:

1. 闭包似优化了变量，原来需要类对象完成的工作，闭包也可以完成
2. 由于闭包引用了外部函数的局部变量，则外部函数的局部变量没有及时释放，消耗内存

## decorator

decorator本质是函数, 装饰其他函数(为其他函数添加附加功能). 对使用者而言, decorator相当于是透明的(使用者不知道是否加入了装饰器)

decorator原则: 
- 不能修改被修改函数的源码
- 不能修改函数的调用方式

python解释器执行的时候，认为函数就是一个变量名指向了函数体；相当于新建了一个变量，没有用；可以中途换指向

```python
#避免这种情况
def test1():
    print("---test1---")
def test1():
    print("---test2---")

test1()#---test2---
```

写代码要遵循**开放封闭原则**，虽然在这个原则是用的面向对象开发，但是也适用于函数式编程，简单来说，它规定已经实现的功能代码不允许被修改，但可以被扩展，即：

- 封闭：已实现的功能代码块
- 开放：对扩展开发

可以加东西，不要修改已经弄好的；

```python
#without decoration
def w1(func):
    def inner():
        print("---added function---")
        func()
    return inner

#下面几个不让动
def func1():
    print("---test1---")

def func2():
    print("---test2---")

def func3():
    print("---test3---")

func1=w1(func1)
func2=w1(func2)
func3=w1(func2)

#consume the functions
func1()
func2()
func3()
```

这个就是装饰器模式：用w1给func1,func2,func3装饰一下

```python
#with decoration, 只是为了简化上面的代码
def w1(func):
    def inner():
        print("---added function---")
        func()
    return inner

#下面几个不让动
@w1 #等价于func1=w1(func1)
def func1():
    print("---test1---")

@w1
def func2():
    print("---test2---")

@w1
def func3():
    print("---test3---")

func1()
func2()
func3()
```

```bash
#output
---added function---
---test1---
---added function---
---test2---
---added function---
---test3---
```

多个装饰器的例子：

```python
#定义函数：完成包裹数据
def makeBold(fn):
    print("---bold begin---")
    def wrapped():
        print("---bold wrap---")
        return "<b>" + fn() + "</b>"
    print("---bold end---")    
    return wrapped

#定义函数：完成包裹数据
def makeItalic(fn):
    print("---italic begin---")
    def wrapped():
        print("---italic wrap---")
        return "<i>" + fn() + "</i>"
    print("---italic end---")
    return wrapped

@makeBold
def test1():
    return "hello world-1"

@makeItalic
def test2():
    return "hello world-2"

@makeBold
@makeItalic
def test3():
    return "hello world-3"
```

```bash
#即便是没有调用test1(),test2(),test3()
---bold begin---
---bold end---
---italic begin---
---italic end---
---italic begin---
---italic end---
---bold begin---
---bold end---
```

```python
#定义函数：完成包裹数据
def makeBold(fn):
    print("---bold begin---")
    def wrapped():
        print("---bold wrap---")
        return "<b>" + fn() + "</b>"
    print("---bold end---")    
    return wrapped

#定义函数：完成包裹数据
def makeItalic(fn):
    print("---italic begin---")
    def wrapped():
        print("---italic wrap---")
        return "<i>" + fn() + "</i>"
    print("---italic end---")
    return wrapped

@makeBold
@makeItalic
def test3():
    return "hello world-3"

#开始调用
print("="*15)
print(test3())
```

```bash
#output, 存在堆栈结构
---italic begin---
---italic end---
---bold begin---
---bold end---
===============
---bold wrap---
---italic wrap---
<b><i>hello world-3</i></b>
```

不使用装饰器，来得到相同的结果

```python
#定义函数：完成包裹数据
def makeBold(fn):
    print("---bold begin---")
    def wrapped():
        print("---bold wrap---")
        return "<b>" + fn() + "</b>"
    print("---bold end---")    
    return wrapped

#定义函数：完成包裹数据
def makeItalic(fn):
    print("---italic begin---")
    def wrapped():
        print("---italic wrap---")
        return "<i>" + fn() + "</i>"
    print("---italic end---")
    return wrapped

def test3():
    return "hello world-3"

#先调用的是makeItalic()
test3=makeItalic(test3)
test3=makeBold(test3)

#开始调用
print("="*15)
print(test3())
```

```bash
#output，装饰的时候是从下往上，调用的时候从上往下
---italic begin---
---italic end---
---bold begin---
---bold end---
===============
---bold wrap---
---italic wrap---
<b><i>hello world-3</i></b>
```

## decorator with parameters

1.1无参数的函数

```python
from time import ctime, sleep

def timefun(func):
    def wrappedfunc():
        print(f"{func.__name__} called at {ctime()}")
        func()
    return wrappedfunc

@timefun
def foo():
    print("I am foo")

foo()
sleep(2)
foo()
```

```bash
#output
foo called at Mon Mar 12 17:28:27 2018
I am foo
foo called at Mon Mar 12 17:28:29 2018
I am foo
```

```python
import time

def decorator(func):
    def wrapper(*args,**kwargs):
        start=time.time()
        func(*args,**kwargs)
        stop=time.time()
        print(f'run time is {stop-start}')
    return wrapper
 
@decorator
def test(list_test):
    for i in list_test:
        time.sleep(0.1)
        print(f'---{i}---')
  
 
# test这个时候本质是在执行wrapper()
# decorator(test)(range(10)) 
test(range(10))
print(test(range(10)))# None
```

2.带参数的函数

```python
from time import ctime, sleep

def timefun(func):
    def wrappedfunc(a, b):
        print(f"{func.__name__} called at {ctime()}")
        print(a, b)
        func(a, b)
    return wrappedfunc

@timefun
def foo(a, b):
    print(a+b)

foo(3,5)
sleep(2)
foo(2,4)
```

```bash
#output
foo called at Mon Mar 12 17:33:42 2018
3 5
8
foo called at Mon Mar 12 17:33:44 2018
2 4
6
```

3.不定长参数的函数装饰

```python
from time import ctime, sleep

def timefun(func):
    def wrappedfunc(*args, **kwargs):
        print(f"{func.__name__} called at {ctime()}")
        func(*args, **kwargs)#这里要拆包
    return wrappedfunc

@timefun
def foo(a, b, c):
    print(a+b+c)

foo(3,5,7)
sleep(2)
foo(2,4,9)
```

```bash
#output
foo called at Mon Mar 12 17:38:58 2018
15
foo called at Mon Mar 12 17:39:00 2018
15
```

4.带return函数的装饰

```python
# method1:
import time

def decorator(func):
    def wrapper(*args,**kwargs):
        start=time.time()
        func(*args,**kwargs)
        stop=time.time()
        print(f'run time is {stop-start}')
        return sum(*args)
    return wrapper
 
@decorator
def test(list_test):
    for i in list_test:
        time.sleep(0.1)
        print(f'---{i}---')
  
# test(range(10))
print(test(range(10)))# 45
```

```python
# method2:
import time

def decorator(func):
    def wrapper(*args,**kwargs):
        start=time.time()
        res=func(*args,**kwargs)
        stop=time.time()
        print(f'run time is {stop-start}')
        return res
    return wrapper
 
@decorator
def test(list_test):
    for i in list_test:
        time.sleep(0.1)
        print(f'---{i}---')
    return sum(list_test)
  
# test(range(10))
print(test(range(10)))
```

```python
from time import ctime, sleep

def timefun(func):
    def wrappedfunc():
        print(f"{func.__name__} called at {ctime()}")
        func()
    return wrappedfunc

@timefun
def foo():
    print("I am foo")

@timefun
def getInfo():
    return '----hahah---'

foo()
print(getInfo())
```

```bash
#output, 没有拿到最里面的结果
foo called at Mon Mar 12 17:42:37 2018
I am foo
getInfo called at Mon Mar 12 17:42:37 2018
None
```

```python
from time import ctime, sleep

def timefun(func):
    def wrappedfunc():
        print(f"{func.__name__} called at {ctime()}")
        return func() #修改这个地方
    return wrappedfunc

@timefun
def foo():
    print("I am foo")

@timefun
def getInfo():
    return '----hahah---'

foo()
print(getInfo())
```

```bash
#output
foo called at Mon Mar 12 17:45:36 2018
I am foo
getInfo called at Mon Mar 12 17:45:36 2018
----hahah---
```

比较通用的装饰器结构

```python
def func(functionName):
    def func_in(*args, **kwargs):
        print("-----新功能放在这儿，比如记录日志-----")
        ret = functionName(*args, **kwargs)
        return ret
    return func_in
```

5.装饰器带参数,在原有装饰器的基础上，设置外部变量

无参数的decorator需要两层嵌套, 带参数的decorator需要三层嵌套;

```python
from time import ctime, sleep

def timefun_arg(pre="hello"):
    def timefun(func):
        def wrappedfunc():
            print(f"{func.__name__} called at {ctime()},pre={pre}")
            return func()
        return wrappedfunc
    return timefun

@timefun_arg("itcast")
def foo():
    print("I am foo")

@timefun_arg("python")
def too():
    print("I am too")

foo()
sleep(2)
foo()

too()
sleep(2)
too()
```

```bash
#可以理解为
timefun=timefun_arg("itcast")
foo=timefun(foo)
foo()
#或者
foo()==timefun_arg("itcast")(foo)()
#output
foo called at Mon Mar 12 18:00:27 2018,pre=itcast
I am foo
foo called at Mon Mar 12 18:00:29 2018,pre=itcast
I am foo
too called at Mon Mar 12 18:00:29 2018,pre=python
I am too
too called at Mon Mar 12 18:00:31 2018,pre=python
I am too
```

6.类装饰器

装饰器函数其实是这样一个接口约束，它必须接受一个`callable`对象作为参数，然后返回一个`callable对`象。在Python中一般callable对象都是函数，但也有例外。只要某个对象重写了 `__call__()` 方法，那么这个对象就是callable的。

```python
class Test():
    def __call__(self):
        print('call me!')

t = Test()
t()  # call me
```

```python
#without decoration
class Test(object):
    def __init__(self, func):
        print("---initializing---")
        print(f"func name is {func.__name__}")
        self.__func = func
    def __call__(self):
        print("---装饰器中的功能---")
        self.__func()

def func1():
    print("---do func1---")

func1=Test(func1)
#开始调用
print("="*15)
func1()
````

```python
#with decoration
class Test(object):
    def __init__(self, func):
        print("---initializing---")
        print(f"func name is {func.__name__}")
        self.__func = func
    def __call__(self):
        print("---装饰器中的功能---")
        self.__func()

@Test
def func1():
    print("---do func1---")

#开始调用
print("="*15)
func1()
```

```bash
#output
---initializing---
func name is func1
===============
---装饰器中的功能---
---do func1---
```

类装饰器的作用：用来简化getter,setter

`property`就是一个重写了`__call__`方法的class

### `property`

```python
class Student(object):
    def __init__(self):
        self.__age=22
    def getAge(self):
        return self.__age
    def setAge(self, value):
        if isinstance(value,int):
            self.__age=value
        else:
            print('not a integer age')
    # 简化调用的时候书写getter,setter
    Age=property(getAge,setAge)

stu1=Student()
print(stu1.Age)#22,会调用getAge函数
stu1.Age=20
print(stu1.Age)#20
```

使用装饰器的方法，达到同样的效果

```python
class Student(object):
    def __init__(self):
        self.__age=22
    
    @property
    def Age(self):
        return self.__age
    @Age.setter
    def Age(self, value):
        if isinstance(value,int):
            self.__age=value
        else:
            print('not a integer age')

stu1=Student()
print(stu1.Age)#22
stu1.Age=20
print(stu1.Age)#20
```

具体的演化方式

```python
class Student(object):
    def __init__(self):
        self.__age=22
    def getAge(self):
        return self.__age
    def setAge(self, value):
        if isinstance(value,int):
            self.__age=value
        else:
            print('not a integer age')
    #创建一个property()对象p1
    #p1两个函数Age.getter(),p1.setter()分别包装了getAge,和setAge;返回值一个property对象，返回的对象当stu1有赋值重载和.的时候，发生调用；
    p1=property()
    getAge=p1.getter(getAge)
    setAge=p1.setter(setAge)

stu1=Student()
print(stu1.getAge)#这里直接就调用了getAge函数
stu1.setAge=20#这里调用了setAge函数
print(stu1.getAge)#20
```

```python
#装饰成这个样子
class Student(object):
    def __init__(self):
        self.__age=22
    p1=property()
    @p1.getter
    def getAge(self):
        return self.__age
    @p1.setter
    def setAge(self, value):
        if isinstance(value,int):
            self.__age=value
        else:
            print('not a integer age')
```

```python
#进一步进化
class Student(object):
    def __init__(self):
        self.__age=22
    def getAge(self):
        return self.__age
    def setAge(self, value):
        if isinstance(value,int):
            self.__age=value
        else:
            print('not a integer age')
    p1=property()
    p2=p1.getter(getAge)#对getter就是对p1本身的操作，返回property对象
    Age=p2.setter(setAge)# property对象

stu1=Student()
print(stu1.Age)#22
stu1.Age=20
print(stu1.Age)#20
```

```python
#装饰成这个样子
class Student(object):
    def __init__(self):
        self.__age=22
    p1=property()
    @p1.getter
    def p2(self):
        return self.__age
    @p2.setter
    def Age(self, value):
        if isinstance(value,int):
            self.__age=value
        else:
            print('not a integer age')

stu1=Student()
print(stu1.Age)#22
stu1.Age=20
print(stu1.Age)#20
```

```python
#引入__init__的考虑
class Student(object):
    def __init__(self):
        self.__age=22
    def getAge(self):
        return self.__age
    def setAge(self, value):
        if isinstance(value,int):
            self.__age=value
        else:
            print('not a integer age')
    p1=property(getAge)
    Age=p1.setter(setAge)
```

```python
class Student(object):
    def __init__(self):
        self.__age=22
    @property
    def Age(self):
        return self.__age
    @Age.setter
    def Age(self, value):
        if isinstance(value,int):
            self.__age=value
        else:
            print('not a integer age')
    ## 等价形式
    # Age=property(Age)
    # Age=Age.setter(Age)
stu1=Student()
print(stu1.Age)#22
stu1.Age=20
print(stu1.Age)#20
```


## 静态语言vs动态语言

- 静态语言，不允许在运行过程中修改代码;C,C++,C#
- 动态语言，可以在运行过程中修改代码;PHP,Ruby,Python

比如

```python
def printInfo():
    print("Hello")

a=100
a=printInfo#直接从一种类型指向另一种类型
```

```python
import types

class Student():
    def __init__(self, name, age):
        self.name=name
        self.__age=age

def go():
    print("gogogo")

def eat(self):
    print(self.name+" eating")

#动态添加属性
stu1=Student("grey",44)
stu1.sex="man"
stu1.weight=100

#甚至添加类属性
Student.AvergeScores=90

#参数必须是空的，才能这么做
stu1.run=go
stu1.run()#gogoto

# method1:
xxx=types.MethodType(eat,stu1)
xxx()#grey eating
stu1.eatEat=types.MethodType(eat,stu1)
stu1.eatEat()#grey eating

# method2:
types.MethodType(eat,stu1)
stu1.eatEat=eat
stu1.eatEat(stu1)#grey eating
```

```python
#动态添加静态方法
import types

class Student():
    def __init__(self, name, age):
        self.name=name
        self.__age=age

@staticmethod
def Laugh():
    print("hahaha")

Student.Haha=Laugh
Student.Haha()#hahaha
```

```python
#动态添加类方法
import types

class Student():
    def __init__(self, name, age):
        self.name=name
        self.__age=age

@classmethod
def Laugh(cls):
    print("hahaha")

Student.Haha=Laugh
Student.Haha()#hahaha
```

summary:

- 给instance添加属性，添加实例方法
- 给class添加属性，添加静态方法，类方法

比如天猫的客户端没有更新，但是页面的按钮经常更新，应该是先下载了代码，然后动态加载进去了；

## `__slots__`

只允许对Person实例添加name和age属性：为了达到限制的目的，Python允许在定义class的时候，定义一个特殊的`__slots__`变量，来限制该class实例能添加的属性：

```python
class Student():
    __slots__=("name","age")

stu1=Student()
stu1.name="grey"
try:
    stu1.score=100
except Exception as e:
    print(e)
```

```bash
#output
'Student' object has no attribute 'score'
```

## 元类(metaclass)

python认为，类同样也是一种对象;python中一切皆对象

ORM映射会用到metaclass

```python
#即便没有创建实例，也会运行
class Person(object):
    num=0
    print("hello,world")
    def __init__(self):
        pass
print(type(Person), Person)
p1=Person()
print(p1)
```

```bash
#outputhello,world
<class 'type'> <class '__main__.Person'>
<__main__.Person object at 0x0000017020398208>
hello,world
```

动态创建class, method1: 一般不要这么干，因为class是一等公民

```python
def choose_class(name):
    if name == 'foo':
        class Foo(object):
            pass
        return Foo     # 返回的是类，不是类的实例
    else:
        class Bar(object):
            pass
        return Bar

fooClass=choose_class("foo")
fooInstance=fooClass()
print(fooClass,fooInstance)
barClass=choose_class("other")
barInstance=barClass()
print(barClass,barInstance)
```

```bash
#output
<class '__main__.choose_class.<locals>.Foo'> <__main__.choose_class.<locals>.Foo object at 0x000001CCF6C6E0F0>
<class '__main__.choose_class.<locals>.Bar'> <__main__.choose_class.<locals>.Bar object at 0x000001CCF6C6E1D0>
```

python中类型(int, str, float)本质是一个class，用来创建instance; 而class又是由type创建的，而一般把type称为**metaclass**(元类)；

动态创建class, method2:`type`法(type既可以检查类型，也可以创建class，为了兼容旧版本，一般不这么做)

`type(类名, 由父类名称组成的元组（针对继承的情况，可以为空），包含属性的字典（名称和值）)`

```python
Student = type("Student", (object,), {"name":None,"age":55})

stu1 = Student()
print(stu1)
stu1.name="grey"
print(stu1.name, stu1.age)
```

```bash
#output
<__main__.Student object at 0x000001DAFCB6D080>
grey 55
```

```python
#动态绑定方法
def printInfo(self):
    print(f"name={self.name}, age={self.age}")

Student = type("Student", (object,), {"name":None, "age":55, "printStu":printInfo})

stu1 = Student()
stu1.printStu()
```

```bash
#output
name=None, age=55
```

```python
class Student(object):
    pass

stu1=Student()
print(stu1.__class__)
print(Student.__class__)
print(type.__class__)#神奇
```

```bash
#output
<class '__main__.Student'>
<class 'type'>
<class 'type'>
```

### `__metaclass__`属性

Python会在类的定义中寻找`__metaclass__`属性，如果找到了，Python就会用它来创建类Foo，如果没有找到，就会用内建的type来创建这个类

- Foo中有`__metaclass__`这个属性吗？如果是，Python会通过`__metaclass__`创建一个名字为Foo的类(对象)
- 如果Python没有找到`__metaclass__`，它会继续在Bar（父类）中寻找`__metaclass__`属性，并尝试做和前面同样的操作。
- 如果Python在任何父类中都找不到`__metaclass__`，它就会在模块层次中去寻找`__metaclass__`，并尝试做同样的操作。
- 如果还是找不到`__metaclass__`,Python就会用内置的type来创建这个类对象

### customize metaclass

元类的主要目的就是为了当创建类时能够自动地改变类。通常，你会为API做这样的事情，你希望可以创建符合当前上下文的类


```python
#主要考虑python3中的metaclass
def upper_attr(future_class_name, future_class_parents, future_class_attr):
    #遍历属性字典，把不是__开头的属性名字变为大写
    newAttr = {}
    for name,value in future_class_attr.items():
        if not name.startswith("__"):
            newAttr[name.upper()] = value

    #调用type来创建一个类, 里面的属性都大写
    return type(future_class_name, future_class_parents, newAttr)

class Foo(object, metaclass=upper_attr):#关键就在这里
    bar = 'bip'

print(hasattr(Foo, 'bar'))#false
print(hasattr(Foo, 'BAR'))#True

f1 = Foo()
print(f1.BAR)#bip
```

```bash
#output
False
True
bip
```

```python
#in python2
def upper_attr(future_class_name, future_class_parents, future_class_attr):
    #遍历属性字典，把不是__开头的属性名字变为大写
    newAttr = {}
    for name,value in future_class_attr.items():
        if not name.startswith("__"):
            newAttr[name.upper()] = value
    #调用type来创建一个类
    return type(future_class_name, future_class_parents, newAttr)

class Foo(object, metaclass=upper_attr):
    bar = 'bip'

print(hasattr(Foo, 'bar'))
print(hasattr(Foo, 'BAR'))

f = Foo()
print(f.BAR)
```

```python
#final example
class UpperAttrMetaClass(type):
    # __new__ 是在__init__之前被调用的特殊方法
    # __new__是用来创建对象并返回之的方法
    # 而__init__只是用来将传入的参数初始化给对象
    # 你很少用到__new__，除非你希望能够控制对象的创建
    # 这里，创建的对象是类，我们希望能够自定义它，所以我们这里改写__new__
    # 如果你希望的话，你也可以在__init__中做些事情
    # 还有一些高级的用法会涉及到改写__call__特殊方法，但是我们这里不用
    def __new__(cls, future_class_name, future_class_parents, future_class_attr):
        #遍历属性字典，把不是__开头的属性名字变为大写
        newAttr = {}
        for name,value in future_class_attr.items():
            if not name.startswith("__"):
                newAttr[name.upper()] = value
        # 方法1：通过'type'来做类对象的创建
        # return type(future_class_name, future_class_parents, newAttr)

        # 方法2：复用type.__new__方法
        # 这就是基本的OOP编程，没什么魔法
        # return type.__new__(cls, future_class_name, future_class_parents, newAttr)

        # 方法3：使用super方法
        # return super(UpperAttrMetaClass, cls).__new__(cls, future_class_name, future_class_parents, newAttr)
        return super().__new__(cls, future_class_name, future_class_parents, newAttr)

# #python2的用法
# class Foo(object):
#     __metaclass__ = UpperAttrMetaClass
#     bar = 'bip'

# python3的用法
class Foo(object, metaclass = UpperAttrMetaClass):
    bar = 'bip'

print(hasattr(Foo, 'bar'))#False
print(hasattr(Foo, 'BAR'))#True

f = Foo()
print(f.BAR)#bip
```

python性能低的原因在于GIL, 可以用C语言突破这种限制；