# python string

<!-- TOC -->

- [python string](#python-string)
    - [`eval`](#eval)
    - [`exec`](#exec)
    - [编码问题](#编码问题)
    - [常用的函数：](#常用的函数)
        - [center](#center)
        - [count](#count)
        - [some funcions](#some-funcions)
        - [replace,strip](#replacestrip)
        - [split,splitlines](#splitsplitlines)
    - [string format](#string-format)
        - [string with color](#string-with-color)
    - [string format attention](#string-format-attention)
        - [`f'{}'`](#f)

<!-- /TOC -->

## `eval`

```bash
python2, raw_input() return string, input() return number;
python3 ,input() retern return string, eval(input()) return number
```

eval可以处理字符串中的**数值与表达式**

```python
num1=eval("12.35")
num2=eval("2*3-9")
print(num1,num2)
```

## `exec`

执行字符串中的语句

```python
myStr='print("hello,world")'
exec(myStr)
```

`repr()` 输出对 Python比较友好，而`str()`的输出对用户比较友好

```python
print("x" in "xorg")#True
print("xo" not in "xorg")#False

num1=100
num2=200
print("%d,%d"%(num1,num2))
myStr="%d,%d"%(num1**2,num2**2)
print(myStr)
#right
print(format("1",">10s"))
print(format("12",">10s"))
print(format("123",">10s"))
#left
print(format("1","<10s"))
print(format("12","<10s"))
print(format("123","<10s"))
#left
print(format("1","10s"))
print(format("12","10s"))
print(format("123","10s"))
```

```bash
#output
True
False
100,200
10000,40000
         1
        12
       123
1         
12        
123       
1         
12        
123       
```

```python
print("%-10.2f %f"%(1.23,1.9))#1.23       1.900000
print("%10.2f %f"%(1.23,1.9))#      1.23 1.900000
print("%010.2f %f"%(1.23,1.9))#0000001.23 1.900000
print("%% %f %f"%(1.23,1.9))#% 1.230000 1.900000
```

在python2中，普通字符串是以8bit ASCII进行存储的，而unicode字符串则存储为16bit的unicode字符串，python2使用unicode要加u,`myStr=u"Hello"`

在python3中，默认都是unicode(utf-8,unicode-8,unicode16)，不用加`u`

`.py`文件保存格式：ASCII,utf-8

- utf-8包括ASCII
- ASCII格式，没有中文，可以编译
- utf-8格式

```python
#r节约时间，不用输入\;不能处理"
print(r"C:\Program Files (x86)\AnyDesk")
# print(r"\"C:\Program Files (x86)\AnyDesk\"")#r不能处理这个
print("C:\\Program Files (x86)\\AnyDesk")
```

```python
# print(dir(""))#查看字符串函数
myList=dir("")
print(myList[0])#__add__
# 在command window里面输入
# help(str.find),help("".find)查看find的帮助，help(str.find)更好，第一个是class，第二个是function
for i in myList:
    print(i)
    help("str."+i)#查看每一个函数的help
    help("\"\"."+i)#查看每一个函数的help
```

## 编码问题

涉及到在内存中的存储问题；

```bash
英文字母：对常用的编码、解码无所谓；但是下面的utf-16等不行

字节数 : 1;编码：GB2312
字节数 : 1;编码：GBK
字节数 : 1;编码：GB18030
字节数 : 1;编码：ISO-8859-1
字节数 : 1;编码：UTF-8
字节数 : 4;编码：UTF-16
字节数 : 2;编码：UTF-16BE
字节数 : 2;编码：UTF-16LE

中文汉字：

字节数 : 2;编码：GB2312
字节数 : 2;编码：GBK
字节数 : 2;编码：GB18030
字节数 : 1;编码：ISO-8859-1
字节数 : 3;编码：UTF-8
字节数 : 4;编码：UTF-16
字节数 : 2;编码：UTF-16BE
字节数 : 2;编码：UTF-16LE
```

- 编码(encode)：字符串到二进制
- 解码(decode)：二进制到字符串

```python
#encode
a1="你好abc".encode("utf-8")
a2="你好abc".encode("gbk")
#decode
b1=a1.decode("utf-8")
b2=a2.decode("gbk")

print(a1,a2)#b'\xe4\xbd\xa0\xe5\xa5\xbdabc' b'\xc4\xe3\xba\xc3abc'，和下面的bytes()结果一样的
print(b1,b2)#你好abc 你好abc
print(type(a1),type(a2))#<class 'bytes'> <class 'bytes'>
print(type(b1),type(b2))#<class 'str'> <class 'str'> 
```

```python
b1=bytes(10)#bytes是一个class
print(b1)#b'\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00',创建一个10bytes的object
b2=bytes(1)
print(b2)#b'\x00',创建一个1bytes的object

#同样的一个东西，不同编码，大小不同
#编码是对一个二进制进行不同的解释，用另外一种解释就会乱码
#encode，字符串到二进制码
a1=bytes("你好abc","utf-8")#一个汉字占3bytes
a2=bytes("你好abc","gbk")#一个汉字占2bytes
a3=bytes("abc","utf-8")
a4=bytes("abc","gbk")
print(a1,a2)#b'\xe4\xbd\xa0\xe5\xa5\xbdabc' b'\xc4\xe3\xba\xc3abc'
print(a3,a4)#b'abc' b'abc'

#decode,二进制码到字符串
print(b'\xe4\xbd\xa0\xe5\xa5\xbdabc'.decode("utf-8"))#你好abc
print(b'\xe4\xbd\xa0\xe5\xa5\xbdabc'.decode("gbk"))#浣犲ソabc，出现了乱码
# print(b'\xc4\xe3\xba\xc3abc'.decode("utf-8"))#error
print(b'\xc4\xe3\xba\xc3abc'.decode("gbk"))#你好abc
print(b'abc'.decode("utf-8"))#abc
print(b'abc'.decode("gbk"))#abc
```

文件编码和字符串编码：[区别](http://blog.csdn.net/sun_abc/article/details/5590983)

文件不存在什么编码(归根结底文件都是二进制文件，用ue打开可以看到都是一个个的16进制数)，只有文件中的字符才可以说编码。通常保存的时候提示对文件编码(utf-8,ANSI)就是对文件中的字符串编码成二进制的方式。

一个用**ANSI**保存的`.py`文件如果有中文，那么`python3 test.py`的时候，python3默认将二进制按照**utf-8**解析为字符串，那么中文的那一段就会乱码，如果中文英文混合，就会decode失败。

而且用**utf-8**保存的文件比**ANSI**保存的文件大

```python
a1="你好abc".encode("utf-8")
a2="你好abc".encode("gbk")
b1=a1.decode("utf-8")
b2=a2.decode("gbk")
b3=a1.decode("gbk")
b4=a2.decode("utf-8","ignore")#解码失败不出错，第二个参数默认是'strict'，还有一个'replace'

print(a1,a2)#b'\xe4\xbd\xa0\xe5\xa5\xbdabc' b'\xc4\xe3\xba\xc3abc'
print(b1,b2)#你好abc 你好abc
print(b3,b4)#浣犲ソabc abc
```

## 常用的函数：

### center

```python
myStr="888"
for i in range(5,25,2):
    print(" "*((23-i)//2),end='')
    print(myStr.center(i,"*"))
```

```bash
#ouput
         *888*
        **888**
       ***888***
      ****888****
     *****888*****
    ******888******
   *******888*******
  ********888********
 *********888*********
**********888**********
```

### count

```python
myStr="hello,grey;hello,world"
print(myStr.count("he"))#2
print(myStr.count("he",1))#1
print(myStr.count("he",1,6))#0,index从1到6
```

### some funcions

```python
#endswith
fileName="C:/NewFolder/my.txt"
print(fileName.endswith(".txt"))#True

#\t8个空格,默认tab是8个空格
print("\ta")
myStr="\ta"
print(myStr.expandtabs(tabsize=4))

#find,rfind
myStr2="As Bitcoin’s price has soared, so too has the energy consumption to produce it—to the point that Bitcoin mining now guzzles"
print(myStr2.find("coin"))#6,从左往右找到第一个匹配的，找不到返回-1
print(myStr2.find("coin",10,20))#-1
print(myStr2.find("coin",-30))#100
print(myStr2.rfind("coin"))#100,找到最后一个匹配的
print(myStr2.rfind("coin",-50,-1))#100
print(myStr2.rfind("coin",50,-1))#100
print(myStr2.rfind("coin",0,50))#6

#index,rindex,不存在返回值为exception
myStr3="As Bitcoin’s price has soared, so too has the energy consumption to produce it—to the point that Bitcoin mining now guzzles"
print(myStr3.index("coin"))#6
print(myStr3.rindex("coin"))#100

#isalnum(),isdigit(),isalpha(),islower(),isupper(),isspace(),istitle(),isdecimal()
print("grey123456".isalnum())#True
#汉字没有大小写，大小写都是汉字
print("abc我".islower())#True
print("ABC我".isupper())#True
print("Grey".istitle())#是否首字符大写,True
print("109".isdecimal())#判断进制,True
print("0x109".isdecimal())#False
```

```python
#join
a1="abc"
b1="123"
#用于制造间隔符
print(a1.join(b1))#1abc2abc3
print(b1.join(a1))#a123b123c
print(",".join("grey"))#g,r,e,y
for i in "grey":
    print(i,end=',')#g,r,e,y,
print()

#ljust,rjust
print("888".ljust(10,"*"))#888*******
print("888".rjust(10,"*"))#*******888

#upper(),lower()

#max()
print(max("abc"))#c
print(min("abc"))#a
myList=[x for x in range(10)]
print(max(myList))#9
print(min(myList))#0
```

### replace,strip

```python
#replace
myStr1="hello,grey,hello, china"
myStr2=myStr1.replace("hello","python")
myStr3=myStr1.replace("hello","python",1)#只替换一次
print(myStr1)#hello,grey,hello, china
print(myStr2)#python,grey,python, china
print(myStr3)#python,grey,hello, china

#strip()去除空格,中间的去不掉
myStr4="    abc 123    "
myStr5=myStr4.strip()
myStr6=myStr4.lstrip()
myStr7=myStr4.rstrip()
print(myStr4)
print(myStr5)
print(myStr6)
print(myStr7)
```

### split,splitlines

```python
#split
myStr="18844091035 # 1601110248 # 19980329"
myList=myStr.split(" # ")
myList3=myStr.split(" # ",1)#只切割一次
print(myList)#['18844091035', '1601110248', '19980329']

#splitlines
myStr2="""As Bitcoin’s price has soared, 
so too has the energy consumption to produce 
it—to the point that Bitcoin mining now guzzles 
more electricity 
than all the electric cars in the world."""
myList2=myStr2.splitlines()#默认为False,去掉每行的\n
myList3=myStr2.splitlines(True)#保留每行的\n
print(myList2)
print(myList3)
```

```python
#startswidh,endswith
fileName="C:/NewFolder/my.txt"
print(fileName.endswith(".txt"))#True
print(fileName.startswith("C:"))#True
```

```python
#swapcase
myStr="Grey"
print(myStr.swapcase())#gREY
print(myStr)#Grey
myStr=myStr.swapcase()
print(myStr)#gREY

#zfill
print("123abc".zfill(10))#0000123abc,用于金融数据，填充0
```

```python
#translate，用于加密，无规则
myStr="hello,python,我"
myTable=myStr.maketrans("abcde我","12345X")
print(myStr.translate(myTable))#h5llo,python,X
```

```python
# maktrans
to_encrypt_str='hell, allen'
pwd_book=str.maketrans('abcdefghi', '123456789')
print(to_encrypt_str.translate(pwd_book)) # 85ll, 1ll5n
```

## string format

不到万不得已不要使用`+`连接字符串，因为每次使用都要开辟新内存

- %
- format
- f
- Template: not recommended

```python
item='apple'
price=16

# method1
print("%s's price is %d"%(item, price))

# method2
print("{_item}'s price is {_price}".format(_item=item, _price=price))
print("{0}'s price is {1}".format(item, price)) # recommended

# method3, >=python3.6, recommended
print(f"{item}'s price is {price}")
```

### string with color

```python
num=1999
print(f'The price is \033[31;1m{num}\033[0m') # num is red
```

```python
product_list = [
    ('Iphone',5800),
    ('Mac Pro',9800),
    ('Bike',800),
    ('Watch',10600),
    ('Coffee',31),
    ('Alex Python',120),
]
shopping_list = []
salary = input("Input your salary:")
if salary.isdigit():
    salary = int(salary)
    while True:
        for index,item in enumerate(product_list):
            print(index,item)
        user_choice = input("选择要买嘛？>>>:")
        if user_choice.isdigit():
            user_choice = int(user_choice)
            if user_choice < len(product_list) and user_choice >=0:
                p_item = product_list[user_choice]
                if p_item[1] <= salary: #买的起
                    shopping_list.append(p_item)
                    salary -= p_item[1]
                    print("Added %s into shopping cart,your current balance is \033[31;1m%s\033[0m" %(p_item,salary) )
                else:
                    print("\033[41;1m你的余额只剩[%s]啦，还买个毛线\033[0m" % salary)
            else:
                print("product code [%s] is not exist!"% user_choice)
        elif user_choice == 'q':
            print("--------shopping list------")
            for p in shopping_list:
                print(p)
            print("Your current balance:",salary)
            exit()
        else:
            print("invalid option")
```

## string format attention

[formatspec](https://docs.python.org/3/library/string.html#formatspec)

```python
str1='hello\tWorld\n'

print('%s'%str1) # s means str()
print('%r'%str1) # r means repr()
```

```bash
# output
hello	World

'hello\tWorld\n'
```

### `f'{}'`

```python
num0=1234567890
str1='hi\tGrey'
print(len(str1)) # 7
print(num0)
# !
print(f'{str1!s}')# str()
print(f'{str1!r}')# repr()
print(f'{str1!a}')# ascii()
```

```bash
# output
7
1234567890
hi	Grey
'hi\tGrey'
'hi\tGrey'
```

```python
num0=1234567890
print('='*12)
num1=123456
print(num0)
# alignment: <,>,^
print(f'{num1:10}')
print(f'{num1:<10}')
print(f'{num1:>10}')
print(f'{num1:^10}')
# padding filling
print(f'{num1:*^10}')
print(f'{num1:->10}')
```

```bash
# output
============
1234567890
    123456
123456    
    123456
  123456  
**123456**
----123456
```

```python
num0=1234567890
print('='*12)
str2='hello, world'
print(num0)
# truncating
print(f'{str2:.4}')
# truncating & padding
print(f'{str2:>10.4}')
```

```bash
# output
============
1234567890
hell
      hell
```

```python
num0=1234567890
print('='*12)
print(num0)
# integer
print(f'{num1}')
print(f'{num1:d}')
print(f'{num1:>10}')
print(f'{num1:>10d}')
print(f'{num1:010d}')
print(f'{num1:+d}')
print(f'{num1:+10d}')
print(f'{-num1:10d}')
print(f'{num1:=+10d}')
print(f'{-num1:=10d}')

# float
pi=3.1415926535
print(f'{pi}')
print(f'{pi:f}')
print(f'{pi:.2f}')
print(f'{pi:>10.2f}')
print(f'{pi:0>10.2f}')
```

```bash
# output
============
1234567890
123456
123456
    123456
    123456
0000123456
+123456
   +123456
   -123456
+   123456
-   123456
3.1415926535
3.141593
3.14
      3.14
0000003.14
```

```python
num0=1234567890
pi=3.1415926535
num3=1234567890

print(num0)
print(f'{pi:>10.2%}')
print(f'{num3:,}')
```

```bash
# output
1234567890
   314.16%
1,234,567,890
```

```python
num=16

print(f'{num:b}')
print(f'{num:o}')
print(f'{num:d}')
print(f'{num:x}')

print(f'{num:#b}')
print(f'{num:#o}')
print(f'{num:#d}')
print(f'{num:#x}')
```

```bash
# ouput
10000
20
16
10

0b10000
0o20
16
0x10
```

```python
myDict={'name':'grey', 'age':20, 'weight':50}
str1='name={name}, age={age}, weight={weight}kg'.format(**myDict)
print(str1) # name=grey, age=20, weight=50kg
```