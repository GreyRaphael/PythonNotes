# Python OOP

<!-- TOC -->

- [Python OOP](#python-oop)
    - [OOP的意义](#oop的意义)
        - [`==` vs `is`](#-vs-is)
    - [ctor & destructor](#ctor--destructor)
        - [class property vs instance property](#class-property-vs-instance-property)
        - [`super()`](#super)
        - [`__init__` vs `__new__`](#__init__-vs-__new__)
        - [`__del__`](#__del__)
    - [python GUI](#python-gui)
    - [动态增加属性、方法](#动态增加属性方法)
    - [overload](#overload)
    - [副本机制](#副本机制)
    - [访问控制](#访问控制)
    - [send SMS](#send-sms)
    - [send Email](#send-email)
    - [`import` local .py file](#import-local-py-file)
        - [import method 1](#import-method-1)
        - [import method 2](#import-method-2)
    - [组合多种功能(send email,send sms)](#组合多种功能send-emailsend-sms)
    - [访问控制](#访问控制-1)

<!-- /TOC -->

## OOP的意义

函数只是实现了**代码的重用**，而class还实现了**数据的封装**(数据独立性、权限)和**数据的重用**

```python
#class type
class Student:
    pass

print(type(Student))#<class 'type'>
stu1=Student()
print(type(stu1))#<class '__main__.Student'>
```

```python
class Student(object):#继承自object，也可以不写(object)
    #ctor
    def __init__(self,name,score):#第一个参数永远是实例变量self
        print("instance is created")
        self.Name=name
        self.Score=score
    #destructor
    def __del__(self):
        print("instance is destructed")
    def AddScore(self,num):
        self.Score+=num
    def SubScore(self,num):
        self.Score-=num
    def printSelfAddr(self):
        print("self addr:",id(self))

print(type(Student))#<class 'type'>
#stu1
stu1=Student("grey",100.5)#instance is created
print(stu1.Name,stu1.Score)#grey 100.5
stu1.AddScore(100)
print(stu1.Name,stu1.Score)#grey 200.5
stu1.SubScore(200)
print(stu1.Name,stu1.Score)#grey 0.5
#stu2
stu2=Student("chris",500)#instance is created
print(stu2.Name,stu2.Score)#chris 500
stu2.AddScore(100)
print(stu2.Name,stu2.Score)#chris 600
stu2.SubScore(200)
print(stu2.Name,stu2.Score)#chris 400
#不同的instance,数据地址不同，函数地址相同(用self来区分是谁来调用的)
print(id(stu1.Name),id(stu1.AddScore))#2258064818896 2258031228296
print(id(stu2.Name),id(stu2.AddScore))#2258064819008 2258031228296
#对应每一个instance,instance的地址与其self的地址相同
print(id(stu1))#1531340244472
stu1.printSelfAddr()#self addr: 1531340244472
print(id(stu2))#1531340244416
stu2.printSelfAddr()#self addr: 1531340244416
#instance is destructed
#instance is destructed
```

### `==` vs `is`

- `is` 是比较两个引用是否指向了同一个对象（引用比较）。
- `==` 是比较两个对象是否相等。

```python
a = [11, 22, 33]
a = [11, 22, 33]
b = [11, 22, 33]
c = a
print(a == b)  # True
print(a is b)  # False

print(a == c)  # True
print(a is c)  # True

print(b == c)  # True
print(b is c)  # False
print(id(a), id(b), id(c))  # 2408832912904 2408832972424 2408832912904
```

```python
b1 = 10000000000
b2 = 10000000000

print(b1 == b2) #True
print(b1 is b2) #True
```

## ctor & destructor

```python
class Student(object):
    Count=0
    def __init__(self):#class中只能在函数中使用self,其他地方不行
        Student.Count+=1#不能用self，而要用Student；用self结果是0不是5(对象会自己新建同名变量，并覆盖；所以外边同名变量的一般省略了，这里是要用到外部的所以没有省略)
        print("ctor")
        
    def __del__(self):
        Student.Count-=1
        print("destructor")

myStuList=[]
for i in range(5):
    stu=Student()
    myStuList.append(stu)

print(Student.Count)
```

```bash
#ouput
ctor
ctor
ctor
ctor
ctor
5
destructor
destructor
destructor
destructor
destructor
```

### class property vs instance property

```python
class Person:
    name='class name'
    def __init__(self, name, age):
        self.name=name
        self.age=age

p1=Person('grey', 26)
# 同名变量: 实例变量先于类变量
print(p1.name) # grey
```

```python
class Person:
    name='class name'
    def __init__(self, name, age):
        self.name=name
        self.age=age

p1=Person('grey', 26)
# del a property
del p1.age
# add a property
p1.weapon='ak47'
print(p1.name, p1.age, p1.weapon) # has no attribut 'age'
```

```python
class Person:
    Count=0
    def __init__(self, name, age):
        self.name=name
        self.age=age

p1=Person('grey', 26)
p2=Person('james', 23)

# 这里并没有修改类变量，只是造了一个实例变量与类变量同名而已
p1.Count=100
print(p1.Count, p2.Count) # 100, 0
```

```python
class Person:
    Count=0
    def __init__(self, name, age):
        self.name=name
        self.age=age

p1=Person('grey', 26)
p2=Person('james', 23)

p1.Count=100

Person.Count=999
print(p1.Count, p2.Count) # 100 999
```

```python
class Person:
    list1=[]
    def __init__(self, name, age):
        self.name=name
        self.age=age

p1=Person('grey', 26)
p2=Person('james', 23)

p1.list1.append('p1')

Person.list1.append('Person')
print(p1.list1, p2.list1) # ['p1', 'Person'] ['p1', 'Person']
```

```python
class Person:
    _Count=0
    def __init__(self, name, age):
        Person._Count+=1
        self.name=name
        self.age=age

p1=Person('grey', 26)
p2=Person('james', 23)

print(p1._Count, p2._Count)# 2 2
```

### `super()`

```python
class Person:
    def __init__(self, name, age):
        self.name=name
        self.age=age
    def eat(self):
        print('I can eat!')

class Man(Person):
    def eat(self):
        print('man is eating...')

# 默认继承父类的__init__
m1=Man('grey', 28)
m1.eat()# man is eating
```

```python
class Person:
    def __init__(self, name, age):
        self.name=name
        self.age=age
    def eat(self):
        print('I can eat!')

class Man(Person):
    def eat(self):
        # 先调用父类方法，然后调用子类的方法;
        # super() is recommended
        super().eat()
        # Person.eat(self)
        print('man is eating...')

# 默认继承父类的__init__
m1=Man('grey', 28)
m1.eat()
# I can eat!
# man is eating...
```

```python
class Person:
    def __init__(self, name, age):
        self.name=name
        self.age=age
    def eat(self):
        print('I can eat!')

class Man(Person):
    def __init__(self, name, age, weight):
        super().__init__(name, age)
        self.weight=weight
    def eat(self):
        super().eat()
        print('man is eating...')

m1=Man('grey', 28, 50)
```

### `__init__` vs `__new__`

流程：

1. 创建一个对象(`__new__`)，将返回值给self，将其他参数也给`__init__`的其他参数
2. 对应`__new__`的参数，初始化(`__init__`)
3. 返回对象的引用

- `__new__`:创建对象时调用，会返回当前对象的一个实例
- `__init__`:创建完对象后调用，对当前对象的一些实例初始化，无返回值

两个方法合起来相当于**ctor**

python不会主动的调用父类的ctor(对应于`__init__()`), 所以主动调用父类的ctor对应的需要`super()`; c++会主动调用父类的ctor;

```python
class Dog(object):
    def __init__(self):
        print("__init__() called")
    def __new__(cls):
        print(id(cls))#cls是类对象

        print("__new__() called")
        return super().__new__(cls)#其实返回值，就是传递给了self,让self指向__new__()创建的对象

print(id(Dog))
dog1=Dog()
```

```bash
#output
1574827469672
1574827469672
__new__() called
__init__() called
```

```python
#单例(单个实例)
class Dog(object):

    __instance=None

    def __new__(cls):
        if cls.__instance==None:
            cls.__instance=super().__new__(cls)
            return cls.__instance
        else:
            return cls.__instance

dog1=Dog()
dog2=Dog()
#单例，创建的两个都指向同一个对象
print(id(dog1),id(dog2))#2755947049536 2755947049536
```

```python
#带参数的单例
#单例(单个实例)
class Dog(object):

    __instance=None

    def __new__(cls,name):#这里的name只是为了对应参数，进而保存参数
        if cls.__instance==None:
            cls.__instance=super().__new__(cls)
            return cls.__instance
        else:
            return cls.__instance
    def __init__(self, name):
        self.name=name

dog1=Dog("grey")
dog2=Dog("chris")
#单例，创建的两个都指向同一个对象
print(id(dog1),id(dog2))#2755947049536 2755947049536
print(dog1.name,dog2.name)
```

```bash
#output
1981537641248 1981537641248
chris chris
```

```python
#单例，只初始化一次
#单例(单个实例)
class Dog(object):

    __instance=None
    __init_flag=False

    def __new__(cls,name):
        if cls.__instance==None:
            cls.__instance=super().__new__(cls)
            return cls.__instance
        else:
            return cls.__instance
    def __init__(self, name):
        if Dog.__init_flag==False:
            self.name=name
            Dog.__init_flag=True

dog1=Dog("grey")
dog2=Dog("chris")
#单例，创建的两个都指向同一个对象
print(id(dog1),id(dog2))#2755947049536 2755947049536
print(dog1.name,dog2.name)
```

```bash
#output
2243901186792 2243901186792
grey grey
```

### `__del__`

对象被销毁的时候调用

```python
class A:
    def __del__(self):
        print('count=0, this will be collected')

a=A()
b=a
del a
print('='*20)
del b # 计数为0, 实例调用__del__
print('='*20)
# ====================
# count=0, this will be collected
# ====================
```

## python GUI

```python
import tkinter

mytk=tkinter.Tk()#Tk is a class
mytk.title("Untitled")
mytk.geometry('500x400+0+100')#width=500,height=400,x=0,y=100
mytk.mainloop()#新建一个窗口
```

```python
import tkinter
class  mywin:
    def  __init__(self,text,height,width,x,y):
        self.mytk = tkinter.Tk()  # Tk是一个类，tk()构造函数
        self.mytk.title(text)  # 调用类的行为
        self.height=height
        self.width=width
        self.x=x
        self.y=y
        laststr="%dx%d+%d+%d"%(self.height,self.width,self.x,self.y)
        self.mytk.geometry(laststr)  # 800宽度，800高度，x,y坐标，左上角
    def show(self):
        self.mytk.mainloop()  # 运行起来

win1=mywin("hello python1",300,400,0,0)
win1.show()
win2=mywin("hello python2",300,400,0,0)
win2.show()
win3=mywin("hello python3",300,400,0,0)
win3.show()
```

```python
import tkinter
class  mywin:
    def  __init__(self,text,height,width,x,y):
        self.mytk = tkinter.Tk()
        self.mytk.title(text)
        self.height=height
        self.width=width
        self.x=x
        self.y=y
        laststr="%dx%d+%d+%d"%(self.height,self.width,self.x,self.y)
        self.mytk.geometry(laststr)
    def show(self):
        self.mytk.mainloop()

myList=[]
for i in range(5):
    title="window"+str(i)
    myTemp=mywin(title,500+i*20,300+i*20,10*i,10*i)
    myList.append(myTemp)

for i in range(5):
    myList[i].show()
```

## 动态增加属性、方法

如果前辈已经写好了一个class,而该class非常难改；就采用**动态增加属性**、**动态增加方法**，也就是所谓的**迭代开发**

```python
#empty class
class Student(object):
    pass

stu1=Student()
stu1.Name="James"#动态增加属性
stu1.Score=95.6#动态增加属性
#动态增加方法，无法再使用self
stu1.AddScore=lambda num:print(stu1.Score,"Add",num)##动态增加method
#
stu1.AddScore(20)
print(stu1.Name,stu1.Score)
```

## overload

```python
#without return
#overload operator
class MyComplex(object):
    def __init__(self, x,y):
        self.x=x
        self.y=y
    def show(self):
        print(self.x,'+',self.y,'i')
    def __add__(self, other):
        self.x+=other.x
        self.y+=other.y

c1=MyComplex(1,2)
c2=MyComplex(3,4)
c1.show()#1+2i
c2.show()#3+4i
c1+c2
c1.__add__(c2)#与上一句效果一样
c1.show()#7+10i
c2.show()#3+4i
```

```python
#with return
#overload operator
class MyComplex(object):
    def __init__(self, x,y):
        self.x=x
        self.y=y
    def show(self):
        print(self.x,'+',self.y,'i')
    def __add__(self, other):
        return MyComplex(self.x+other.x,self.y+other.y)

c1=MyComplex(1,2)
c2=MyComplex(3,4)
c1.show()#1+2i
c2.show()#3+4i
c3=c1+c2
c1.show()#1+2i
c2.show()#3+4i
c3.show()#4+6i
```

```python
#with different parameter
#overload operator
class MyComplex(object):
    def __init__(self, x,y):
        self.x=x
        self.y=y
    def show(self):
        print(self.x,'+',self.y,'i')
    def __add__(self, other):
        #不同类型的重载，只能这么做
        if type(other)==type(self):
            return MyComplex(self.x+other.x,self.y+other.y)
        elif type(other)==type(10):
            return MyComplex(self.x+other,self.y+other)

c1=MyComplex(1,2)
c2=MyComplex(3,4)
c3=c1+c2
c1.show()#1+2i
c2.show()#3+4i
c3.show()#4+6i
c4=c3+10
c4.show()#14+16i
```

## 副本机制

```python
#浅拷贝
#overload operator
class MyComplex(object):
    def __init__(self, x,y):
        self.x=x
        self.y=y
    def show(self):
        print(self.x,'+',self.y,'i')

c1=MyComplex(1,2)
c2=c1#浅拷贝
c1.show()
c2.show()
print(id(c1),id(c2))#140644246524312 140644246524312
```

```python
#深拷贝
#overload operator
class MyComplex(object):
    def __init__(self, x,y):
        self.x=x
        self.y=y
    def show(self):
        print(self.x,'+',self.y,'i')

c1=MyComplex(1,2)
c2=MyComplex(c1.x,c1.y)#深拷贝
c1.show()
c2.show()
print(id(c1),id(c2))#139718618935816 139718618935872
```

函数的参数，副本机制，浅拷贝，不能修改原来变量的地址；

```python
#函数副本机制
def change(myStr):
    print("func",id(myStr))#func 140452856766736
    myStr="grey"#到这里之后才改变了id,也就是说到这里才新建了一个变量，而且这里的str是整体改变，改变不了其中的元素
    print("func",id(myStr))#func 140452856766624

myStr="hello"
print("main",id(myStr))#main 140452856766736
change(myStr)
print("main",id(myStr))#main 140452856766736
```

```python
#list修改
#不能修改mylist的地址，但是可以改变其中的元素的指向
def change(myList):
    print("func",myList,id(myList))#func ['h', 'e', 'l', 'l', 'o'] 140293166807752
    myList[1]='x'
    print("func",myList,id(myList))#func ['h', 'x', 'l', 'l', 'o'] 140293166807752

myList=list("hello")
print("main",myList,id(myList))#main ['h', 'e', 'l', 'l', 'o'] 140293166807752
change(myList)
print("main",myList,id(myList))#main ['h', 'x', 'l', 'l', 'o'] 140293166807752
```

```python
#list整体修改的时候，会创建一个新的，其他时候不会创建新的
def change(myList):
    print("func",myList,id(myList))#func ['h', 'e', 'l', 'l', 'o'] 140293166807752
    myList=tuple("hxllo")
    print("func",myList,id(myList))#func ('h', 'x', 'l', 'l', 'o') 140004140756400

myList=list("hello")
print("main",myList,id(myList))#main ['h', 'e', 'l', 'l', 'o'] 140293166807752
change(myList)
print("main",myList,id(myList))#main ['h', 'x', 'l', 'l', 'o'] 140293166807752
```

```python
#instance as parameter
class Student(object):
    Score=0

def  Reallychange(stu):
    print(stu,id(stu))    
    stu.Score=100
    print(stu,id(stu))    

def FakeChangge(stu):
    print(stu,id(stu))
    stu=Student()
    stu.Score=99
    print(stu,id(stu))    

stu1=Student()
print(stu1,id(stu1))
FakeChangge(stu1)
print(stu1,id(stu1))
#
print()
stu2=Student()
print(stu2,id(stu2))
Reallychange(stu2)
print(stu2,id(stu2))
```

## 访问控制

python是解释性语言；对于instance，变量存在，解释为**引用**, 变量不存在，解释为**动态绑定**

```python
class Asset(object):
    def __init__(self):
        self.__money=10000#只是用__来当作是private
    def show(self):
        print(self.__money)

myAsset1=Asset()
myAsset1.show()#10000
# print(myAsset1.__money)#不合法，外部无法访问
#这一个是动态绑定
myAsset2=Asset()
myAsset2.__money=1000000
print(myAsset2.__money)#1000000
myAsset2.show()#10000
```

```python
class Asset(object):
    def __init__(self):
        self.__money=10000#只是用__来当作是private
    def show(self):
        print(self.__money)
    def AddMoney(self, num):
        self.__money+=num
        print("Add Success")
    def RetriveMoney(self, num):
        i=0
        while i<3:
            password=input("enter your password: ")
            if password=="123456":
                self.__money-=num
                break
            else:
                print("pass word error")
            i+=1
        if i==3:
            print("card is locked")
        else:
            print("retrive success")
            
myAsset1=Asset()
myAsset1.show()
myAsset1.AddMoney(10000)
myAsset1.show()
myAsset1.RetriveMoney(2000)
```

```python
#private method
class Asset(object):
    def __init__(self):
        self.__money=10000#只是用__来当作是private
    def show(self):
        print(self.__GetMoney())
    def __SetMoney(self, num):#private method
        self.__money=num
    def __GetMoney(self):#private method
        return self.__money
    
myAsset1=Asset()
myAsset1.show()#10000
```

```python
class Asset(object):
    def __init__(self):
        self.__money=10000#只是用__来当作是private
    def __SetMoney(self, num):#private method
        self.__money=num
    def __GetMoney(self):#private method
        return self.__money
    def __CheckPassword(self, password):
        if password=="123456":
            return True
        else:
            return False
    def Query(self, password):
        if self.__CheckPassword(password):
            return self.__GetMoney()
        else:
            return None

myAsset1=Asset()
print(myAsset1.Query("123456"))#10000
#查看这个class有多少方法
print(dir(myAsset1))
```

```python
#修改private variable,调用private method
class Asset(object):
    def __init__(self):
        self.__money=10000#只是用__来当作是private
    def __SetMoney(self, num):#private method
        self.__money=num
    def __GetMoney(self):#private method
        return self.__money
    def show(self):
        print(self.__GetMoney())

myAsset1=Asset()
myAsset1.show()#10000
myAsset1._Asset__money=0
myAsset1.show()#0,修改private variable
myAsset1._Asset__SetMoney(1000000)
myAsset1.show()#1000000,调用private method
```

用class来实现，两个概率密码本的归并

```python
#获取每个概率密码本的行数
class  filegetlines:
    def __init__(self,filepath):
        self.filepath=filepath
        self.file=open(filepath,"rb")
        self.Lines=-1#表示没有读入
    def GetLineCount(self):
        i=0
        while True:
            linestr=self.file.readline()
            if  linestr:#不为空，
                print(linestr)
                i+=1
            else:
                #为空
                break
        self.Lines=i
        return i

f1= filegetlines("data/password1.txt")
print(f1.GetLineCount())#3
f2= filegetlines("data/password2.txt")
print(f2.GetLineCount())#4
```

```python
#实现概率密码本的归并，下标法
#密码次数排序，假定密码不一样
def  merge(path1,path2,path3):
    file1=open(path1,"rb")
    file2=open(path2, "rb")
    file3=open(path3,"wb")
    lines1=4
    lines2=6
    #
    i1=0
    i2=0
    linstr1 = file1.readline() #读一行
    linstr2 = file2.readline() #读一行
    while i1 < lines1  and i2 < lines2 : #取file1
        linelist1 = (linstr1.decode("utf-8")).split(' ')
        linelist2 = (linstr2.decode("utf-8")).split(' ')
        if  eval(linelist1[0]) >eval(linelist2[0]) :
            file3.write(linstr1)
            i1+=1
            linstr1 = file1.readline()
        elif  eval(linelist1[0]) <eval(linelist2[0]) :
            file3.write(linstr2)
            i2+=1
            linstr2 = file2.readline()
        else: #==
            file3.write(linstr1)
            file3.write(linstr2)
            i1+=1
            i2+=1
            linstr1 = file1.readline()
            linstr2 = file2.readline()
    while i1<lines1:
        file3.write(linstr1)
        i1+=1
        linstr1=file1.readline()
    while i2<lines2:
        file3.write(linstr2)
        i2+=1
        linstr2=file2.readline()
    file1.close()
    file2.close()
    file3.close()

merge("data/password1.txt","data/password2.txt","data/passwordMerged.txt")
```

```bash
#password1.txt
1000 123456
800 123456789
500 asdfghjkl
100 zxcvbnm

#password2.txt
2000 123456
1800 123456789
500 asdfghjkl
300 qwertyuiop
50 888888
20 666666

#passwordMerged.txt
2000 123456
1800 123456789
1000 123456
800 123456789
500 asdfghjkl
500 asdfghjkl
300 qwertyuiop
100 zxcvbnm
50 888888
20 666666
```

```python
#如果有重复的情况，需要先按照字符串排序，进行合并；然后再按照次数排序
#Tips:
#先将密码次数=1的清洗到其他文件中
#先将密码次数小的清洗到其他文件中
```

```python
#a Test
file=open("data/password1.txt","rb")
myList=file.readlines()
file.close()

newList=[]
for line in myList:
    line=line.decode("utf-8")
    linesp=line.split(" ")
    newList.append(linesp)
newList.sort(key=lambda x: x[1])#按照第二列排序
print(newList)
#output
#[['1000', '123456\n'], ['800', '123456789\n'], ['500', 'asdfghjkl\n'], ['100', 'zxcvbnm']]
```

```python
file=open("data/pass2.txt","rb")
myList=file.readlines()
file.close()

newList=[]
for line in myList:
    line=line.decode("utf-8")
    linesp=line.split(" ")
    newList.append(linesp)
newList.sort(key=lambda x: x[1])
file=open("data/pass22.txt","wb")
for item in newList:
    file.write((item[0]+' '+item[1]).encode("utf-8"))
```

```bash
#output
#pass1.txt
1000 123456
300 123456789
200 asdfghjkl
100 zxcvbnm

#pass2.txt
2000 123456
1800 123456789
600 asdfghjkl
400 qwertyuiop
50 888888
20 666666

#pass11.txt
1000 123456
300 123456789
200 asdfghjkl
100 zxcvbnm

#pass22.txt
2000 123456
1800 123456789
20 666666
50 888888
600 asdfghjkl
400 qwertyuiop
```


对于一个很大的文件：先分割成多个文件，然后进行多个文件的**密码字符串**的排序，然后统计次数；得到较小的文件

对于若干个较小的文件，进行归并(仍然按照**密码字符串**的顺序)，然后次数相加，生成最后的概率密码本

```python
def Merge(path1,path2,path3):
    file1=open(path1,"rb")
    file2=open(path2,"rb")
    file3=open(path3,"wb")
    lines1=4
    lines2=6
    #
    i1=0
    i2=0
    lineStr1=file1.readline()
    lineStr2=file2.readline()
    while i1<lines1 and i2<lines2:
        lineList1=(lineStr1.decode("utf-8")).split(' ')
        lineList2=(lineStr2.decode("utf-8")).split(' ')
        if lineList1[1].rstrip('\n')<lineList2[1].rstrip('\n'):
            file3.write(lineStr1)
            i1+=1
            lineStr1=file1.readline()
        elif lineList1[1].rstrip('\n')>lineList2[1].rstrip('\n'):
            file3.write(lineStr2)
            i2+=1
            lineStr2=file2.readline()
        else:
            lineStr1=("%d"%(eval(lineList1[0])+eval(lineList2[0]))+' '+lineList1[1]).encode("utf-8")
            file3.write(lineStr1)
            i1+=1
            i2+=1
            lineStr1=file1.readline()
            lineStr2=file2.readline()
    while i1<lines1:
        file3.write(lineStr1)
        i1+=1
        lineStr1=file1.readline()
    while i2<lines2:
        file3.write(lineStr2)
        i2+=1
        lineStr2=file2.readline()
    file1.close()
    file2.close()
    file3.close()

Merge("data/pass1.txt","data/pass2.txt","data/passMerged.txt")
```

```bash
#output,最后还需要sort第一列
3000 123456
2100 123456789
800 asdfghjkl
400 qwertyuiop
50 888888
20 666666
100 zxcvbnm
```

## send SMS

```python
#互亿无线或者阿里短信的api
import http.client
import urllib

host = "106.ihuyi.com"
sms_send_uri = "/webservice/sms.php?method=Submit"

# 用户名请登录用户中心->验证码、通知短信->帐户及签名设置->APIID
account = "C56220776"
# 密码 查看密码请登录用户中心->验证码、通知短信->帐户及签名设置->APIKEY
password = "e2a9ca9a4b287da535180a1ec74aaa89"

def send_sms(text, mobile):
    params = urllib.parse.urlencode(
        {'account': account, 'password': password, 'content': text, 'mobile': mobile, 'format': 'json'})
    headers = {"Content-type": "application/x-www-form-urlencoded", "Accept": "text/plain"}
    conn = http.client.HTTPConnection(host, port=80, timeout=30)
    conn.request("POST", sms_send_uri, params, headers)
    response = conn.getresponse()
    response_str = response.read()
    conn.close()
    return response_str

#Test
list= [13295800121,18503088297,13810811154,13419990303,17090412975,15210681197,18774678240,18630126892,17346533310,17703819995,13672169651,18085637896,17600364335,15931269973]
text = "这是短信的内容"
for mobile  in list:
    print(send_sms(text, str(mobile)))
```

```python
#将api包装到class中
import http.client
import urllib

class SendSMS:
    #账号初始化
    def  __init__(self,account,password):
        self.host = "106.ihuyi.com"
        self.sms_send_uri = "/webservice/sms.php?method=Submit"
        self.account = account
        self.password = password
    #发送短信
    def send_sms(self,text,mobile):
        params = urllib.parse.urlencode(
            {'account': self.account, 'password': self.password, 'content': text, 'mobile': mobile, 'format': 'json'})
        headers = {"Content-type": "application/x-www-form-urlencoded", "Accept": "text/plain"}
        conn = http.client.HTTPConnection(self.host, port=80, timeout=30)
        conn.request("POST", self.sms_send_uri, params, headers)
        response = conn.getresponse()
        response_str = response.read()
        conn.close()
        return response_str
    #批量发送
    def send_smss(self,text,mobilelist):
        for mobile in mobilelist:
            print(self.send_sms(text, str(mobile)))


sender1=SendSMS("C29408441","42d111e84be3ca82032dbfea49f36dba")
print(sender1.send_sms("这是短信内容","18890506318"))
```

## send Email

发送消息给Email需要给定端口，要不然不清楚是wechat, qq还是邮箱接受了消息

```python
#尹成email:yincheng8848@163.com,tsinghua8848
import smtplib  #发邮件
from email.mime.text import MIMEText #邮件文本

class  SendMail:
    def __init__(self,SMTPsever,Sender,password ):
        self.SMTPsever = SMTPsever  # 服务器
        self.Sender = Sender  # 发送邮件的地址
        self.password = password  # 密码
        self.mailsever = smtplib.SMTP(self.SMTPsever, 25)  # 邮件服务器25端口
        self.mailsever.login(self.Sender, self.password)  # 登陆
    def Send(self,Message,title,maillist):
        msg = MIMEText(Message)  # 转化邮件文本
        msg["Subject"] =title  # 邮件标题
        msg["From"] = self.Sender  # 邮件发送者
        msg["To"] = "zhouyongguo2009@163.com"  # 谁来收

        self.mailsever.sendmail(self.Sender,
                           maillist,#收件人
                           msg.as_string())
    def exit(self):
        self.mailsever.quit()


sender1= SendMail("mail.pku.edu.cn","gewei@pku.edu.cn","GUN")
sender1.Send("这是内容","这是标题",["zhouyongguo2009@163.com"])
sender1.exit()
```

## `import` local .py file

### import method 1

```bash
#文件结构
/Python/
    test.py
    another.py
```

```python
#another.py
class AnotherClass(object):
    def __init__(self):
        print("this is in ctor")
```

```python
#test.py
import another

go=another.AnotherClass()#this is in ctor
```

### import method 2

```python
#test.py
from another import AnotherClass

go=AnotherClass()#this is in ctor
```

## 组合多种功能(send email,send sms)

```bash
#文件结构
Test/
    QQPerson.py
    SMS.py
    Email.py
```

```python
#Email.py
import smtplib  #发邮件
from email.mime.text import MIMEText #邮件文本

class  SendMail:
    def __init__(self,SMTPsever,Sender,password ):
        self.SMTPsever = SMTPsever  # 服务器
        self.Sender = Sender  # 发送邮件的地址
        self.password = password  # 密码
        self.mailsever = smtplib.SMTP(self.SMTPsever, 25)  # 邮件服务器25端口
        self.mailsever.login(self.Sender, self.password)  # 登陆
    def Send(self,Message,title,maillist):
        msg = MIMEText(Message)  # 转化邮件文本
        msg["Subject"] =title  # 邮件标题
        msg["From"] = self.Sender  # 邮件发送者
        msg["To"] = "zhouyongguo2009@163.com"  # 谁来收

        self.mailsever.sendmail(self.Sender,
                           maillist,#收件人
                           msg.as_string())
    def exit(self):
        self.mailsever.quit()
```

```python
#SMS.py
import http.client
import urllib

class SendSMS:
    #账号初始化
    def  __init__(self,account,password):
        self.host = "106.ihuyi.com"
        self.sms_send_uri = "/webservice/sms.php?method=Submit"
        self.account = account
        self.password = password
    #发送短信
    def send_sms(self,text,mobile):
        params = urllib.parse.urlencode(
            {'account': self.account, 'password': self.password, 'content': text, 'mobile': mobile, 'format': 'json'})
        headers = {"Content-type": "application/x-www-form-urlencoded", "Accept": "text/plain"}
        conn = http.client.HTTPConnection(self.host, port=80, timeout=30)
        conn.request("POST", self.sms_send_uri, params, headers)
        response = conn.getresponse()
        response_str = response.read()
        conn.close()
        return response_str
    #批量发送
    def send_smss(self,text,mobilelist):
        for mobile in mobilelist:
            print(self.send_sms(text, str(mobile)))
```

```python
#QQPerson.py
import Email
import SMS

class  QQperson:
    def  __init__(self,
                  Nickname,
                  Name,
                  Phone,
                  Mobile,
                  PagerHome,
                  HomePhone,
                  QQ,
                  BQQ,
                  Email):
        self.Nickname=Nickname
        self.Name=Name
        self.Phone=Phone
        self.Mobile=Mobile
        self.PagerHome=PagerHome
        self.HomePhone=HomePhone
        self.QQ=QQ
        self.BQQ=BQQ
        self.Email=Email
    def  sendsms(self,Text):
        sender1 = SMS.SendSMS("C29408441", "42d111e84be3ca82032dbfea49f36dba")
        print(sender1.send_sms(Text, self.Mobile))
    def  sendmail(self,Text,Title):
        sender1 = Email.SendMail("smtp.163.com", "yincheng8848@163.com", "tsinghua8848")
        sender1.Send(Text, Title, [self.Email, "yincheng8848@163.com"])
        sender1.exit()
```

## 访问控制

- `xx`: 公有变量、函数、类
- `_x`: 变量、函数、类在使用`from module1 import *`时都不会被导入，但是`import module1`然后`print(module1._num)`可以访问
- `__xx`：双前置下划线,避免与子类中的属性命名冲突，无法在外部直接访问(名字重整所以访问不到)；变量、函数、类在使用from xxx import *时都不会被导入
- `__xx__`:双前后下划线,用户名字空间的魔法对象或属性。例如:`__init__` ,不要自己发明这样的名字
- xx_:单后置下划线,用于避免与Python关键词的冲突

通过name mangling（名字重整(目的就是以防子类意外重写基类的方法或者属性)修改了名字，如：_Class__object）机制就可以访问private了。

```python
class Student(object):
    def __init__(self):
        self.__num = 100


stu = Student()
print(dir(stu))
print(dir(Student))
```

```bash
#output
['_Student__num', '__class__', '__delattr__', '__dict__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__gt__', '__hash__', '__init__', '__init_subclass__', '__le__', '__lt__', '__module__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', '__weakref__']
['__class__', '__delattr__', '__dict__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__gt__', '__hash__', '__init__', '__init_subclass__', '__le__', '__lt__', '__module__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', '__weakref__']
```