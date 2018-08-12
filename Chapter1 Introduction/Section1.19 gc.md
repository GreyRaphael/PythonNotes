# pythonm GC

- [pythonm GC](#pythonm-gc)
    - [gabage collection](#gabage-collection)
        - [`intern`机制](#intern%E6%9C%BA%E5%88%B6)
    - [垃圾回收机制](#%E5%9E%83%E5%9C%BE%E5%9B%9E%E6%94%B6%E6%9C%BA%E5%88%B6)
    - [对比Ruby和python的GC](#%E5%AF%B9%E6%AF%94ruby%E5%92%8Cpython%E7%9A%84gc)
    - [gc module](#gc-module)
    - [builtin property](#builtin-property)
        - [`__getattribute__`](#getattribute)
    - [builtin functions](#builtin-functions)
        - [`range()`](#range)
        - [`map(function, sequence[, sequence, ...]) -> map object`](#mapfunction-sequence-sequence---map-object)
        - [`filter(function or None, sequence)`](#filterfunction-or-none-sequence)
        - [`sorted(iterable, cmp=None, key=None, reverse=False) --> new sorted list`](#sortediterable-cmpnone-keynone-reversefalse----new-sorted-list)
    - [`functools`](#functools)
        - [`reduce(function, sequence[, initial])`](#reducefunction-sequence-initial)
        - [`partial()`](#partial)
        - [`wraps()`](#wraps)

## gabage collection

对象池：一片内存，里面有很多东西；

Python为了优化速度，使用了小整数对象池，避免为整数频繁申请和销毁内存空间。

Python 对小整数的定义是 `[-5, 257)` 这些整数对象是提前建立好的（python一运行就创建），不会被垃圾回收。在一个 Python 的程序中，所有位于这个范围内的整数使用的都是同一个对象. 超过该范围，每一个大整数，均创建一个新的对象。

同理，单个字母也是这样的。

但是当定义2个相同的字符串时，引用计数为0，触发垃圾回收

```python
#[-5,257)
In [1]: a1,a2=-6,-6

In [2]: print(id(a1),id(a2))
2513494445520 2513494445392

In [3]: b1,b2=-5,-5

In [4]: print(id(b1),id(b2))
1931373856 1931373856

In [5]: c1,c2=256,256

In [6]: print(id(c1),id(c2))
1931382208 1931382208

#神奇
In [1]: d1=257

In [2]: d2=257

In [3]: print(id(d1),id(d2))
2006227070064 2006227070288

In [4]: e1,e2=257,257

In [5]: print(id(e1),id(e2))
2006227070128 2006227070128
```

```python
#[-5,257)无法被销毁
In [1]: a1,a2=100,100

In [2]: print(id(a1),id(a2))
1931377216 1931377216

In [3]: del a1

In [4]: print(id(a2))
1931377216

In [5]: del a2

In [6]: a3=100

In [7]: print(id(a3))
1931377216
```

```python
# for str
# 单个单词，不可修改，默认开启intern机制，共用对象，引用计数为0，则销毁 
In [1]: str1,str2='hello','hello'

In [2]: print(id(str1),id(str2))
2033004034456 2033004034456

In [3]: del(str1)

In [4]: del(str2)

In [5]: str3='hello'

In [6]: print(id(str3))
2033015361408
```

### `intern`机制

中间没有包含空格的字符串，只保存一份(eg.`"HelloWorld"`)；包含空格的，保存不同份(eg.`"Hello world"`)

```python
In [1]: str1='helloworld'

In [2]: str2='helloworld'

In [3]: print(id(str1),id(str2))
1902627113072 1902627113072

In [4]: str3='hello world'

In [5]: str4='hello world'

In [6]: print(id(str3),id(str4))
1902627174320 1902627174704
#另外，大整数不共用内存，引用计数为0，销毁 
```

数值类型和字符串类型在 Python 中都是不可变的，这意味着你无法修改这个对象的值，每次对变量的修改，实际上是创建一个新的对象 

## 垃圾回收机制

引用计数可以解决大部分的问题，但是对于相互引用，无法解决；

以引用计数为主，以**分代收集(Generation Zero)**为辅；而**标记清除**来自于Ruby

引用计数无法解决循环引用的问题

```python
import gc

class ClassA(object):
    def __init__(self):
        print(f"object born id={hex(id(self))}")

def func():
    while True:
        obj1=ClassA()
        obj2=ClassA()
        obj1.t=obj2
        obj2.t=obj1
        del obj1
        del obj2
        #剩下一个循环引用

gc.disable()
# 内存飙升，注释掉上面一句，程序变得正常
func()
```

```python
#循环引用
import gc

def func():
    while True:
        list1=[1,2,3]
        list2=[11,22,33]
        list1.append(list2)
        print(list1)
        list2.append(list1)
        print(list2)
        print("="*20)

#内存同时也在飙升
gc.disable()
func()
```

## 对比Ruby和python的GC

[Visualizing Garbage Collection in Ruby and Python](http://patshaughnessy.net/2013/10/24/visualizing-garbage-collection-in-ruby-and-python)

Ruby: 先申请一个链表的空间，用完了之后，根据标记来清楚垃圾节点；并将节点重新弄成链表，继续使用(**标记清除**)

python: 用的时候申请，引用计数为0之后，销毁；但是无法解决循环引用的问题；

python的分代收集(Generation Zero)可以解决循环引用：

- 三个链表(0,1,2),Generation 0清理最为频繁，Generation2清理最少；
- 每一个Generation都有一个检查的阈值，到达阈值之后才会检查；

Generation0先检查有没有相互引用，没有相互引用的转移到Generation1，有相互引用的，将引用计数减1(如果减1之后还有相互引用，要等到Generation0的下一个阈值的时候才会再减1)；如果引用计数减1之后 为0，那么清楚掉

## gc module

python3默认gc是`gc.is_enable()==True`

导致引用计数+1的情况:

- 对象被创建，例如a=23
- 对象被引用，例如b=a
- 对象被作为参数，传入到一个函数中，例如func(a)
- 对象作为一个元素，存储在容器中，例如list1=[a,a]

导致引用计数-1的情况:

- 对象的别名被显式销毁，例如del a
- 对象的别名被赋予新的对象，例如a=24
- 一个对象离开它的作用域，例如f函数执行完毕时，func函数中的局部变量（全局变量不会）
- 对象所在的容器被销毁，或从容器中删除对象

有三种情况会触发垃圾回收：

- 调用gc.collect(),
- 当gc模块的计数器达到阀值的时候。
- 程序退出的时候

```python
import gc

class ClassA():
    def __init__(self):
        print(f'object born,id:{hex(id(self))}')

def f3():
    print("-----0------")
    c1 = ClassA()
    c2 = ClassA()
    c1.t = c2
    c2.t = c1
    print("-----1------")
    del c1
    del c2
    print("-----2------")
    #垃圾回收后的对象会放在gc.garbage列表里面
    print(gc.garbage)#所以，这里是空的
    print("-----3------")
    #gc.collect()会返回不可达的对象数目
    print(gc.collect()) #显式执行垃圾回收
    print("-----4------")
    print(gc.garbage)
    print("-----5------")

if __name__ == '__main__':
    gc.set_debug(gc.DEBUG_LEAK) #设置gc模块的日志
    f3()
```

```bash
#output
-----0------
object born,id:0x1b9aef58278
object born,id:0x1b9aef58358
-----1------
-----2------
[]
-----3------
32
-----4------
[<__main__.ClassA object at 0x000001B9AEF58278>, <__main__.ClassA object at 0x000001B9AEF58358>, {'t': <__main__.ClassA object at 0x000001B9AEF58358>}, {'t': <__main__.ClassA object at 0x000001B9AEF58278>}, <class 'operator.attrgetter'>, (<class 'operator.attrgetter'>, <class 'object'>), <class 'operator.itemgetter'>, (<class 'operator.itemgetter'>, <class 'object'>), <class 'operator.methodcaller'>, (<class 'operator.methodcaller'>, <class 'object'>), {'__module__': 'operator', '__doc__': "\n    Return a callable object that fetches the given attribute(s) from its operand.\n    After f = attrgetter('name'), the call f(r) returns r.name.\n    After g = attrgetter('name', 'date'), the call g(r) returns (r.name, r.date).\n    After h = attrgetter('name.first', 'name.last'), the call h(r) returns\n    (r.name.first, r.name.last).\n    ", '__slots__': ('_attrs', '_call'), '__init__': <function attrgetter.__init__ at 0x000001B9AED37048>, '__call__': <function attrgetter.__call__ at 0x000001B9AED370D0>, '__repr__': <function attrgetter.__repr__ at 0x000001B9AED37158>, '__reduce__': <function attrgetter.__reduce__ at 0x000001B9AED371E0>, '_attrs': <member '_attrs' of 'attrgetter' objects>, '_call': <member '_call' of 'attrgetter' objects>}, {'__module__': 'operator', '__doc__': '\n    Return a callable object that fetches the given item(s) from its operand.\n    After f = itemgetter(2), the call f(r) returns r[2].\n    After g = itemgetter(2, 5, 3), the call g(r) returns (r[2], r[5], r[3])\n    ', '__slots__': ('_items', '_call'), '__init__': <function itemgetter.__init__ at 0x000001B9AED37268>, '__call__': <function itemgetter.__call__ at 0x000001B9AED372F0>, '__repr__': <function itemgetter.__repr__ at 0x000001B9AED37378>, '__reduce__': <function itemgetter.__reduce__ at 0x000001B9AED37400>, '_call': <member '_call' of 'itemgetter' objects>, '_items': <member '_items' of 'itemgetter' objects>}, {'__module__': 'operator', '__doc__': "\n    Return a callable object that calls the given method on its operand.\n    After f = methodcaller('name'), the call f(r) returns r.name().\n    After g = methodcaller('name', 'date', foo=1), the call g(r) returns\n    r.name('date', foo=1).\n    ", '__slots__': ('_name', '_args', '_kwargs'), '__init__': <function methodcaller.__init__ at 0x000001B9AED37488>, '__call__': <function methodcaller.__call__ at 0x000001B9AED37510>, '__repr__': <function methodcaller.__repr__ at 0x000001B9AED37598>, '__reduce__': <function methodcaller.__reduce__ at 0x000001B9AED37620>, '_args': <member '_args' of 'methodcaller' objects>, '_kwargs': <member '_kwargs' of 'methodcaller' objects>, '_name': <member '_name' of 'methodcaller' objects>}, <function attrgetter.__init__ at 0x000001B9AED37048>, <function attrgetter.__call__ at 0x000001B9AED370D0>, <function attrgetter.__repr__ at 0x000001B9AED37158>, <function attrgetter.__reduce__ at 0x000001B9AED371E0>, <member '_attrs' of 'attrgetter' objects>, <member '_call' of 'attrgetter' objects>, <function itemgetter.__init__ at 0x000001B9AED37268>, <function itemgetter.__call__ at 0x000001B9AED372F0>, <function itemgetter.__repr__ at 0x000001B9AED37378>, <function itemgetter.__reduce__ at 0x000001B9AED37400>, <member '_call' of 'itemgetter' objects>, <member '_items' of 'itemgetter' objects>, <function methodcaller.__init__ at 0x000001B9AED37488>, <function methodcaller.__call__ at 0x000001B9AED37510>, <function methodcaller.__repr__ at 0x000001B9AED37598>, <function methodcaller.__reduce__ at 0x000001B9AED37620>, <member '_args' of 'methodcaller' objects>, <member '_kwargs' of 'methodcaller' objects>, <member '_name' of 'methodcaller' objects>]
-----5------
gc: collectable <ClassA 0x000001B9AEF58278>
gc: collectable <ClassA 0x000001B9AEF58358>
gc: collectable <dict 0x000001B9AEF28A68>
gc: collectable <dict 0x000001B9AEF51EA0>
gc: collectable <type 0x000001B9ACD54208>
gc: collectable <tuple 0x000001B9AED333C8>
gc: collectable <type 0x000001B9ACD53608>
gc: collectable <tuple 0x000001B9AED33448>
gc: collectable <type 0x000001B9ACD16F78>
gc: collectable <tuple 0x000001B9AED33488>
gc: collectable <dict 0x000001B9AED2F990>
gc: collectable <dict 0x000001B9AED2FA68>
gc: collectable <dict 0x000001B9AED2FB40>
gc: collectable <function 0x000001B9AED37048>
gc: collectable <function 0x000001B9AED370D0>
gc: collectable <function 0x000001B9AED37158>
gc: collectable <function 0x000001B9AED371E0>
gc: collectable <member_descriptor 0x000001B9AED2F9D8>
gc: collectable <member_descriptor 0x000001B9AED2FA20>
gc: collectable <function 0x000001B9AED37268>
gc: collectable <function 0x000001B9AED372F0>
gc: collectable <function 0x000001B9AED37378>
gc: collectable <function 0x000001B9AED37400>
gc: collectable <member_descriptor 0x000001B9AED2FAB0>
gc: collectable <member_descriptor 0x000001B9AED2FAF8>
gc: collectable <function 0x000001B9AED37488>
gc: collectable <function 0x000001B9AED37510>
gc: collectable <function 0x000001B9AED37598>
gc: collectable <function 0x000001B9AED37620>
gc: collectable <member_descriptor 0x000001B9AED2FBD0>
gc: collectable <member_descriptor 0x000001B9AED2FC18>
gc: collectable <member_descriptor 0x000001B9AED2FC60>
gc: collectable <dict 0x000001B9AEA16B88>
gc: collectable <SourceFileLoader 0x000001B9AEB8D2B0>
gc: collectable <function 0x000001B9AEA02E18>
gc: collectable <function 0x000001B9AEF3A400>
gc: collectable <function 0x000001B9AEF3A9D8>
gc: collectable <function 0x000001B9AEF3AA60>
gc: collectable <function 0x000001B9AEF3AAE8>
gc: collectable <socket 0x000001B9AEF35590>
```

常用函数：

- `gc.set_debug(flags)` 设置gc的debug日志，一般设置为`gc.DEBUG_LEAK`
- `gc.collect([generation])` 显式进行垃圾回收，可以输入参数，0代表只检查第一代的对象，1代表检查一，二代的对象，2代表检查一，二，三代的对象，如果不传参数，执行一个full collection，也就是等于传2。 返回不可达（unreachable objects）对象的数目
- `gc.get_threshold(`) 获取的gc模块中自动执行垃圾回收的频率。
- `gc.set_threshold(threshold0[, threshold1[, threshold2])` 设置自动执行垃圾回收的频率。
- `gc.get_count()` 获取当前自动执行垃圾回收的计数器，返回一个长度为3的列表


```python
In [1]: import gc

#获取默认3个Generation的阈值
#也就是Generation0到700的时候清理一次；Generation0清理10次清理Generation1;Generation1清理10次清理Generation2;
In [2]: gc.get_threshold()
Out[2]: (700, 10, 10)

In [3]: gc.get_threshold()
Out[3]: (700, 10, 10)

#获取当前到没到阈值（也就是到了上面阈值的哪一位置了）,比如546快到700了
In [4]: gc.get_count()
Out[4]: (546, 5, 1)

In [5]: gc.get_count()
Out[5]: (556, 5, 1)

In [6]: gc.get_count()
Out[6]: (567, 5, 1)
```

```python
#获取引用计数
In [1]: import sys

In [2]: a='hello, grey'
#因为调用函数的时候传入a，这会让a的引用计数+1
In [3]: sys.getrefcount(a)
Out[3]: 2

In [4]: list1=[a,a]

In [5]: sys.getrefcount(a)
Out[5]: 4
```

```python
#因为是在程序中运行的，比较快，所以可以看到结果
import gc

class ClassA(object):
    pass

print(gc.get_count())#(438, 9, 1)
a = ClassA()
print(gc.get_count())#(438, 9, 1)
del a
print(gc.get_count())#(438, 9, 1)
```

```python
#因为是手动输入的，无法看到上面的情况
In [1]: import gc

In [2]: class ClassA(object):
   ...:     pass
   ...: 

In [3]: gc.get_count()
Out[3]: (677, 5, 1)

In [4]: a=ClassA()

In [5]: gc.get_count()
Out[5]: (451, 7, 1)

In [6]: del a

In [7]: gc.get_count()
Out[7]: (467, 7, 1)
```

gc模块唯一处理不了的是循环引用的类都重写`__del__`方法，所以项目中要避免定义`__del__`方法, python2无法解决，python3解决了；所谓的垃圾清理，本质是调用对象的`__del__`方法

```python
import gc

class ClassA():
    pass
    # 如果取消注释，python2会出现uncollectable,python3不会
    # def __del__(self):
    #     print('object die,id:%s'%str(hex(id(self))))

gc.set_debug(gc.DEBUG_LEAK)
a = ClassA()
b = ClassA()

a.next = b
b.prev = a

print("--1--")
print(gc.collect())
print("--2--")
del a
print("--3--")
del b
print("--3-1--")
print(gc.collect())
print("--4--")
```

```bash
--1--
28
--2--
--3--
--3-1--
gc: collectable <type 0x00000289CE059D38>
gc: collectable <tuple 0x00000289D00133C8>
gc: collectable <type 0x00000289CE05A538>
gc: collectable <tuple 0x00000289D0013448>
gc: collectable <type 0x00000289CE02CD68>
gc: collectable <tuple 0x00000289D0013488>
gc: collectable <dict 0x00000289D000F990>
gc: collectable <dict 0x00000289D000FA68>
gc: collectable <dict 0x00000289D000FB40>
gc: collectable <function 0x00000289D0017048>
gc: collectable <function 0x00000289D00170D0>
gc: collectable <function 0x00000289D0017158>
gc: collectable <function 0x00000289D00171E0>
gc: collectable <member_descriptor 0x00000289D000F9D8>
gc: collectable <member_descriptor 0x00000289D000FA20>
gc: collectable <function 0x00000289D0017268>
gc: collectable <function 0x00000289D00172F0>
gc: collectable <function 0x00000289D0017378>
gc: collectable <function 0x00000289D0017400>
gc: collectable <member_descriptor 0x00000289D000FAB0>
gc: collectable <member_descriptor 0x00000289D000FAF8>
gc: collectable <function 0x00000289D0017488>
gc: collectable <function 0x00000289D0017510>
gc: collectable <function 0x00000289D0017598>
gc: collectable <function 0x00000289D0017620>
gc: collectable <member_descriptor 0x00000289D000FBD0>
gc: collectable <member_descriptor 0x00000289D000FC18>
gc: collectable <member_descriptor 0x00000289D000FC60>
gc: collectable <ClassA 0x00000289D023F1D0>
gc: collectable <ClassA 0x00000289D023F208>
gc: collectable <dict 0x00000289D0208A68>
gc: collectable <dict 0x00000289D0232E10>
4
--4--
gc: collectable <dict 0x00000289CFCF6B88>
gc: collectable <SourceFileLoader 0x00000289CFE6D2B0>
gc: collectable <function 0x00000289CFCE2E18>
gc: collectable <function 0x00000289D021A400>
gc: collectable <function 0x00000289D021A9D8>
gc: collectable <function 0x00000289D021AA60>
gc: collectable <function 0x00000289D021AAE8>
gc: collectable <socket 0x00000289D0215590>
```

```python
#手动回收
import gc

class ClassA(object):
    def __init__(self):
        print(f"object born id={hex(id(self))}")

def func():
    while True:
        obj1=ClassA()
        obj2=ClassA()
        obj1.t=obj2
        obj2.t=obj1
        del obj1
        del obj2
        #手动
        gc.collect()

gc.disable()
# 内存不会飙升，注释掉上面一句，程序变得正常
func()
```

实际开发不会用到gc, 了解即可；

## builtin property

```python
class Student(object):
    '''for __doc__ to show'''
    def __init__(self):
        self.name="grey"
        self.__age=55
    def func(self):
        print("Hello, func")

stu1=Student()
stu2=stu1.__class__()#相当于stu2=Student()
print(stu1,stu2)

#__class__
print(stu1.__class__,Student.__class__)

#__dict__
print(stu1.__dict__)

#__doc__
print(stu1.__doc__)
# print(help(stu1))

#__bases__
print(Student.__bases__)

#__dir__
print(stu1.__dir__())
print(dir(stu1))
print(dir(Student))

#__getattribute__,属性拦截器
print(stu1.__getattribute__("name"))
```

```bash
#output
<__main__.Student object at 0x00000194504481D0> <__main__.Student object at 0x00000194504482E8>
<class '__main__.Student'> <class 'type'>
{'name': 'grey', '_Student__age': 55}
for __doc__ to show
(<class 'object'>,)
['name', '_Student__age', '__module__', '__doc__', '__init__', 'func', '__dict__', '__weakref__', '__repr__', '__hash__', '__str__', '__getattribute__', '__setattr__', '__delattr__', '__lt__', '__le__', '__eq__', '__ne__', '__gt__', '__ge__', '__new__', '__reduce_ex__', '__reduce__', '__subclasshook__', '__init_subclass__', '__format__', '__sizeof__', '__dir__', '__class__']
['_Student__age', '__class__', '__delattr__', '__dict__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__gt__', '__hash__', '__init__', '__init_subclass__', '__le__', '__lt__', '__module__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', '__weakref__', 'func', 'name']
['__class__', '__delattr__', '__dict__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__gt__', '__hash__', '__init__', '__init_subclass__', '__le__', '__lt__', '__module__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', '__weakref__', 'func']
grey
```

### `__getattribute__`

访问属性之前，一定会先调用这个方法；

```python
class ClassA(object):
    def __init__(self, name):
        self.name=name
        self.age=55
    #属性访问时拦截器，打log
    def __getattribute__(self, obj):
        print(f"======>{obj}")
        if obj=='name':
            print('log name')
            return 'redicted'
        else:
            temp=super().__getattribute__(obj)
            print(f"===>{temp}")
            #如果没有下面的return，show的结果是None
            return temp
    def show(self):
        print("show func")

s1=ClassA("grey")
print(s1.name)
print(s1.age)
#连方法名也会被检查
#python对象中的方法本质上都是函数指针，所以可以动态链接外部的函数；
s1.show()
```

```bash
#output
======>name
log name
redicted
======>age
===>55
55
======>show
===><bound method ClassA.show of <__main__.ClassA object at 0x0000027D2D6C8400>>
show func
```

`__getattribute__()`的坑

```python
class Person(object):
    def show(self):
        print("Heihei")
    def __getattribute__(self, obj):
        print("---test---")
        if obj.startswith('a'):
            return "haha"
        else:
            return self.show

p1=Person()
print(p1.apple)#haha, 正常
#因为进入else, 返回self.show, 而self.show又会检查__getattribute__,所以进入一个递归，直到stackoverflow
try:
    print(p1.pear)
except Exception as e:
    print(e)
```

```python
#修改后的情况
class Person(object):
    def show(self):
        print("Heihei")
    def __getattribute__(self, obj):
        print("---test---")
        if obj.startswith('a'):
            return "haha"
        elif obj=="show":
            return super().__getattribute__(obj)
        else:
            #为了防止这个坑，__getattribute__要禁止使用self
            return self.show

p1=Person()
print(p1.apple)#haha, 正常
#因为进入else, 返回self.show, 而self.show又会检查__getattribute__,所以进入一个递归，直到stackoverflow
try:
    p1.pear()
except Exception as e:
    print(e)
```

```bash
#output
---test---
haha
---test---
---test---
Heihei
```

## builtin functions

`dir(__builtins__)`, 可以看到很多python解释器启动后默认加载的属性和函数

### `range()`

python2中range返回列表(换成`xrange()`)，python3中range返回一个迭代值

```python
a=range(10)
print(list(a))#[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
```

### `map(function, sequence[, sequence, ...]) -> map object`

- function:是一个函数
- sequence:是一个或多个序列,取决于function需要几个参数
- 返回值是一个list

```python
map1=map(lambda x: x**2,[i for i in range(10)])
print(map1)
for item in map1:
    print(item,end=', ')

print()
map2=map(lambda x,y,z: x+y+z, [x for x in range(5)], [y for y in range(6)], [z for z in range(7)])
#会选择最短的截取其他的sequence
for item in map2:
    print(item,end=', ')

#数据混合
print()
map3=map(lambda num,day: (num, day), [x for x in range(7)], ['Sun', 'Mon', 'Tue', 'Wed', 'Thur', "Fir", 'Sat'])
print(list(map3))
```

```bash
#output
<map object at 0x000001E2B41F94A8>
0, 1, 4, 9, 16, 25, 36, 49, 64, 81, 
0, 3, 6, 9, 12, 
[(0, 'Sun'), (1, 'Mon'), (2, 'Tue'), (3, 'Wed'), (4, 'Thur'), (5, 'Fir'), (6, 'Sat')]
```

### `filter(function or None, sequence)`

```python
filter1=filter(lambda x: x%2==0, range(10)) #过滤掉false
print(filter1)
for item in filter1:
    print(item, end=', ')

print()
filter2=filter(None,"hello")
print(filter2)
for item in filter2:
    print(item, end=', ')
```

```bash
#output
<filter object at 0x0000026DE8068160>
0, 2, 4, 6, 8, 
<filter object at 0x0000026DE8068320>
h, e, l, l, o, 
```

### `sorted(iterable, cmp=None, key=None, reverse=False) --> new sorted list`

```python
list1=[5,2,3,1,0]
sorted1=sorted(list1, key=lambda x: x**2, reverse=True)#key的作用就是先要计算，用计算的临时结果排序原来的sequence
print(list1)
print(sorted1)
```

```bash
#output
[5, 2, 3, 1, 0]
[5, 3, 2, 1, 0]
```

## `functools`

```python
#in python3
import functools
print(dir(functools))
```

```bash
#output
['MappingProxyType', 'RLock', 'WRAPPER_ASSIGNMENTS', 'WRAPPER_UPDATES', 'WeakKeyDictionary', '_CacheInfo', '_HashedSeq', '__all__', '__builtins__', '__cached__', '__doc__', '__file__', '__loader__', '__name__', '__package__', '__spec__', '_c3_merge', '_c3_mro', '_compose_mro', '_convert', '_find_impl', '_ge_from_gt', '_ge_from_le', '_ge_from_lt', '_gt_from_ge', '_gt_from_le', '_gt_from_lt', '_le_from_ge', '_le_from_gt', '_le_from_lt', '_lru_cache_wrapper', '_lt_from_ge', '_lt_from_gt', '_lt_from_le', '_make_key', 'cmp_to_key', 'get_cache_token', 'lru_cache', 'namedtuple', 'partial', 'partialmethod', 'recursive_repr', 'reduce', 'singledispatch', 'total_ordering', 'update_wrapper', 'wraps']
```

常用的是`partial()`, `reduce()`, `wraps`

### `reduce(function, sequence[, initial])`

- function:该函数有两个参数
- sequence:序列可以是str，tuple，list
- initial:固定初始值

在Python3里,reduce函数已经被从全局名字空间里移除了, 它现在被放置在fucntools模块里用的话要先`from functools import reduce`

```python
from functools import reduce

reduce1=reduce(lambda x,y: x+y, range(5))
print(reduce1)#10

print()
# with initial value
reduce2=reduce(lambda x, y: x+y, range(5), 100)
print(reduce2)#110

print()
reduce3=reduce(lambda x, y: x.upper()+y.upper(), 'hello','11')
print(reduce3)#11HELLO
```

### `partial()`

```python
import functools

def showarg(*args, **kw):
    print(args)
    print(kw)

p1=functools.partial(showarg, 1,2,3)
p1()
p1(4,5,6)
p1(a='python', b='itcast')

p2=functools.partial(showarg, a=3,b='linux')
p2()
p2(1,2)
p2(a='python', b='itcast')
```

```bash
#output
#就是设定了默认值
(1, 2, 3)
{}
(1, 2, 3, 4, 5, 6)
{}
(1, 2, 3)
{'a': 'python', 'b': 'itcast'}
()
{'a': 3, 'b': 'linux'}
(1, 2)
{'a': 3, 'b': 'linux'}
()
{'a': 'python', 'b': 'itcast'}
```

### `wraps()`

使用装饰器时，有一些细节需要被注意。例如，被装饰后的函数其实已经是另外一个函数了（函数名等函数属性会发生改变）。

```python
def note(func):
    "note function"
    def wrapper():
        "wrapper function"
        print('note something')
        func()
    return wrapper

@note
def test():
    "test function"
    print('I am test')

test()
print(test.__doc__)
```

```bash
#output
note something
I am test
wrapper function #这不是想要的结果
```

```python
#修改后的程序
import functools
def note(func):
    "note function"
    #简单地添加一句
    @functools.wraps(func)
    def wrapper():
        "wrapper function"
        print('note something')
        func()
        # return func()
    return wrapper

@note
def test():
    "test function"
    print('I am test')

test()
print(test.__doc__)
```

```bash
#output, doc的结果正常
note something
I am test
test function
```