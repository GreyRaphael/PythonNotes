# Python Datastructures & Algorithm

- [Python Datastructures & Algorithm](#python-datastructures-algorithm)
    - [时间复杂度](#%E6%97%B6%E9%97%B4%E5%A4%8D%E6%9D%82%E5%BA%A6)
    - [顺序表](#%E9%A1%BA%E5%BA%8F%E8%A1%A8)
    - [链表](#%E9%93%BE%E8%A1%A8)
        - [单向链表](#%E5%8D%95%E5%90%91%E9%93%BE%E8%A1%A8)
        - [双向链表](#%E5%8F%8C%E5%90%91%E9%93%BE%E8%A1%A8)
        - [单向循环链表(Single Cycle LinkList)](#%E5%8D%95%E5%90%91%E5%BE%AA%E7%8E%AF%E9%93%BE%E8%A1%A8single-cycle-linklist)
        - [双向循环链表](#%E5%8F%8C%E5%90%91%E5%BE%AA%E7%8E%AF%E9%93%BE%E8%A1%A8)
        - [summary](#summary)
    - [stack](#stack)
    - [queue](#queue)
        - [单端队列](#%E5%8D%95%E7%AB%AF%E9%98%9F%E5%88%97)
        - [双端队列](#%E5%8F%8C%E7%AB%AF%E9%98%9F%E5%88%97)

## 时间复杂度

函数调用不能认为是基本操作，而是对基本操作的封装，计算时间复杂度的时候需要考虑函数内部的情况；

`timeit`模块可以用来测试一小段Python代码的执行速度；

```python
import timeit


def append_test():
    list_temp = []
    for i in range(10000):
        list_temp.append(i)


def add_test():
    list_temp = []
    for i in range(10000):
        list_temp += [i, ]
        # list_temp = list_temp + [i, ] # 效率比上面的低，十分慢


def generator_test():
    list_temp = [x for x in range(10000)]


def generator_trans_test():
    list_temp = list(range(10000))


def insert_test():
    list_temp = []
    for i in range(10000):
        list_temp.insert(0, i)  # 倒序而已


def extend_test():
    list_temp = []
    for i in range(10000):
        list_temp.extend([i, ])


# timeit测试的时候并不是在当前文件中执行;所以参数是字符串;而且是在别的地方运算的，所以需要import
timer1 = timeit.Timer('append_test()', 'from __main__ import append_test')
# 测试100次
print(f'{timer1.timeit(1000):>8.4f} cost by append')
timer2 = timeit.Timer('add_test()', 'from __main__ import add_test')
print(f'{timer2.timeit(1000):>8.4f} cost by add')
timer3 = timeit.Timer('generator_test()', 'from __main__ import generator_test')
print(f'{timer3.timeit(1000):>8.4f} cost by generator')
timer4 = timeit.Timer('generator_trans_test()', 'from __main__ import generator_trans_test')
print(f'{timer4.timeit(1000):>8.4f} cost by transform')
timer5 = timeit.Timer('extend_test()', 'from __main__ import extend_test')
print(f'{timer5.timeit(1000):>8.4f} cost by extend')
timer6 = timeit.Timer('insert_test()', 'from __main__ import insert_test')
print(f'{timer6.timeit(1000):>8.4f} cost by insert')
```

```bash
#output
  1.0130 cost by append
  0.9431 cost by add
  0.4621 cost by generator
  0.2741 cost by transform
  1.3409 cost by extend
 22.7166 cost by insert
```

```python
#pop最后一个元素的效率远远高于pop第一个元素
import timeit

list1 = list(range(2000000))
timer1 = timeit.Timer("list1.pop(0)", "from __main__ import list1")
print(f'{timer1.timeit(1000):>8.4f} cost by pop_begin')
timer2 = timeit.Timer("list1.pop()", "from __main__ import list1")
print(f'{timer2.timeit(1000):>8.4f} cost by pop_end')
```

```bash
#output
  1.1288 cost by pop_begin
  0.0001 cost by pop_end
```

python list时间复杂度:

Operation|Big-O Efficiency|Explanation
---|---|---
indexx[ ]|$O(1)$|直接计算偏移量
index assignment|$O(1)$
len()|$O(1)$|直接访问顺序表头
append|$O(1)$
pop()|$O(1)$
pop(i)|$O(n-i)$|有返回值
del l[i]|$O(n-i)$|无返回值
remove|$O(n)$|无返回值
insert(i, item)|$O(n-i)$
iteration|$O(n)$
contains(in)|$O(n)$
get slice [x:y]|$O(k)$
del slice|$O(n)$|pop所有
set slice|$O(n+k)$|先pop所有，然后添加k个
reverse|$O(n)$
concatenate|$O(k)$
sort|$O(nlogn)$|depends on sort-Algorithm
multiply|$O(nk)$
extend()|$O(k)$

python dict时间复杂度:

Operation|Big-O Efficiency
---|---
iteration|$O(n)$
copy|$O(n)$
get item|$O(1)$
set item|$O(1)$
delete item|$O(1)$
contains(in)|$O(1)$

## 顺序表

python中的顺序表将表头和数据区分离(list的id不变)，然后数据区部分还是外置的(不同类型都可以添加)



## 链表

### 单向链表

```python
class Node(object):
    def __init__(self, elem):
        self.elem = elem
        self.next = None


class SingleLinkList(object):
    """单链表"""

    def __init__(self, node=None):
        self.__head = node

    def is_empty(self):
        return self.__head == None

    def length(self):
        # cursor
        cur = self.__head
        count = 0
        while cur != None:
            count += 1
            cur = cur.next
        return count

    def travel(self):
        cur = self.__head
        while cur != None:
            print(cur.elem, end=' ')
            cur = cur.next
        print()

    def add(self, item):
        '''add to begin'''
        node = Node(item)
        node.next = self.__head
        self.__head = node

    def append(self, item):  # 传进来的应该是数据而不是node
        """add to end"""
        node = Node(item)
        if self.is_empty():
            self.__head = node
        else:
            cur = self.__head
            while cur.next != None:
                cur = cur.next
            cur.next = node

    def insert(self, pos, item):
        if pos <= 0:
            self.add(item)
        elif pos >= self.length():
            self.append(item)
        else:
            node = Node(item)
            cur = self.__head
            current_pos = 0
            while current_pos != pos-1:
                current_pos += 1
                cur = cur.next
            node.next = cur.next
            cur.next = node

    def remove(self, item):
        """只是删除第一个"""
        pre = None
        cur = self.__head
        while cur != None:
            if cur.elem == item:
                if cur == self.__head:
                    """如果是开头"""
                    self.__head = cur.next
                else:
                    pre.next = cur.next
                return
            pre = cur
            cur = cur.next
        print(f"{item} not exist")

    def search(self, item):
        cur = self.__head
        while cur != None:
            if cur.elem == item:
                return True
            cur = cur.next
        return False


def main():
    list1 = SingleLinkList()
    list1.append(100)
    list1.append(200)
    list1.add(111)
    print(list1.length())
    list1.travel()
    list1.insert(2, 222)
    list1.travel()
    print(list1.search(333))
    list1.remove(111)
    list1.travel()
    list1.remove(222)
    list1.travel()


if __name__ == '__main__':
    main()
```

```bash
#output
3
111 100 200
111 100 222 200
False
100 222 200
100 200
```

### 双向链表

![](res/linklist01.png)

```python
class Node(object):
    def __init__(self, item):
        self.elem = item
        self.pre = None
        self.next = None


class DoubleLinkList(object):
    """双链表"""

    def __init__(self, node=None):
        self.__head = node

    def is_empty(self):
        return self.__head is None

    def length(self):
        cur = self.__head
        count = 0
        while cur != None:
            count += 1
            cur = cur.next
        return count

    def travel(self):
        cur = self.__head
        while cur != None:
            print(cur.elem, end=' ')
            cur = cur.next
        print()

    def add(self, item):
        node = Node(item)
        self.__head.pre = node
        node.next = self.__head
        self.__head = node

    def append(self, item):
        node = Node(item)
        if self.is_empty():
            self.__head = node
        else:
            cur = self.__head
            while cur.next != None:
                cur = cur.next
            cur.next = node
            node.pre = cur

    def insert(self, pos, item):
        if pos <= 0:
            self.add(item)
        elif pos >= self.length():
            self.append(item)
        else:
            cur = self.__head
            current_pos = 0
            while current_pos != pos:
                current_pos += 1
                cur = cur.next
            node = Node(item)
            # add node
            node.next = cur
            node.pre = cur.pre
            # break
            cur.pre.next = node
            cur.pre = node

    def search(self, item):
        cur = self.__head
        while cur != None:
            if cur.elem == item:
                return True
            cur = cur.next
        return False

    def remove(self, item):
        """只是删除找到的第一个"""
        cur = self.__head
        while cur != None:
            if cur.elem == item:
                if cur == self.__head:
                    """如果是开头"""
                    self.__head = cur.next
                    if cur.next:
                        #链表只有一个node
                        self.__head.pre = None
                else: 
                    cur.pre.next = cur.next
                    if cur.next:
                        # 尾节点
                        cur.next.pre = cur.pre
                return
            cur = cur.next
        print(f"{item} not exist")


def main():
    list1 = DoubleLinkList()
    list1.append(100)
    list1.append(200)
    list1.add(111)
    print(list1.length())
    list1.travel()
    list1.insert(2, 222)
    list1.travel()
    print(list1.search(333))
    list1.remove(111)
    list1.travel()
    list1.remove(222)
    list1.travel()


if __name__ == '__main__':
    main()
```

```bash
#output
3
111 100 200
111 100 222 200
False
100 222 200
100 200
```

### 单向循环链表(Single Cycle LinkList)

![](res/linklist02.png)

```python
class Node(object):
    def __init__(self, elem):
        self.elem = elem
        self.next = None


class SingleLinkList(object):
    """单向循环链表"""

    def __init__(self, node=None):
        self.__head = node
        if node:
            node.next = node

    def is_empty(self):
        return self.__head == None

    def length(self):
        # cursor
        if self.is_empty():
            return 0
        # not empty list
        cur = self.__head
        count = 1
        while cur.next != self.__head:
            count += 1
            cur = cur.next
        return count

    def travel(self):
        if self.is_empty():
            return
        cur = self.__head
        while cur.next != self.__head:
            print(cur.elem, end=' ')
            cur = cur.next
        # 注意最后一个node被遗漏
        print(cur.elem)

    def add(self, item):
        '''add to begin'''
        node = Node(item)
        if self.is_empty():
            node.next = node
            self.__head = node
            return
        # 先找到尾节点
        cur = self.__head
        while cur.next != self.__head:
            cur = cur.next
        cur.next = node
        node.next = self.__head
        self.__head = node

    def append(self, item):  # 传进来的应该是数据而不是node
        """add to end"""
        node = Node(item)
        if self.is_empty():
            node.next = node
            self.__head = node
            return
        cur = self.__head
        while cur.next != self.__head:
            cur = cur.next
        cur.next=node
        node.next=self.__head

    def insert(self, pos, item):
        # 中间位置与单链表相同，不需要改
        if pos <= 0:
            self.add(item)
        elif pos >= self.length():
            self.append(item)
        else:
            node = Node(item)
            cur = self.__head
            current_pos = 0
            while current_pos != pos-1:
                current_pos += 1
                cur = cur.next
            node.next = cur.next
            cur.next = node

    def remove(self, item):
        """只是删除第一个"""
        if self.is_empty():
            print(f"{item} not exist")
            return
        pre = None
        cur = self.__head
        while cur.next != self.__head:
            if cur.elem == item:
                if cur == self.__head:
                    """如果是开头"""
                    # 先找到尾部节点
                    rear=self.__head
                    while rear.next!=self.__head:
                        rear=rear.next
                    self.__head = cur.next
                    rear.next=self.__head
                else:
                    # 中间部分和单链表一样
                    pre.next = cur.next
                return
            pre = cur
            cur = cur.next
        if cur.elem==item:
            if cur==self.__head:
                # only one node
                self.__head=None
            else:
                pre.next=self.__head
            return
        print(f"{item} not exist")

    def search(self, item):
        if self.is_empty():
            return False
        cur = self.__head
        while cur.next != self.__head:
            if cur.elem == item:
                return True
            cur = cur.next
        if cur.elem==item:
            return True
        else:
            return False


def main():
    list1 = SingleLinkList()
    list1.append(100)
    list1.append(200)
    list1.add(111)
    print(list1.length())
    list1.travel()
    list1.insert(2, 222)
    list1.travel()
    print(list1.search(333))
    list1.remove(111)
    list1.travel()
    list1.remove(222)
    list1.travel()


if __name__ == '__main__':
    main()
```

```bash
#output
3
111 100 200
111 100 222 200
False
100 222 200
100 200
```

### 双向循环链表



### summary

对于链表的操作，要考虑

- `self.__head==None`
- 只有一个node
- 最后一个node
- 中间node的情况

甚至上，链表也可以像顺序表一样增加表头信息(表节点)；一般是紧挨着head,然后真正的都在这个表节点的后面

线性表:

- 顺序表
- 链表

## stack

如果将上面的线性表只允许一端操作，禁用其他的操作，那么就可以构成一个stack

顺序表和链表解决存储，而stack, queue是容器;

stack的操作:

- push(item)
- pop()
- peek(), 返回栈顶元素
- is_empty()
- size()

```python
#顺序表实现stack
class Stack(object):
    """借助list来实现stack"""

    def __init__(self):
        self.__list = []

    def push(self, item):
        # 可以选择头部插入，但是list是顺序表实现的，所以使用append效率更高
        # 如果是单链表，选择头部插入效率更高
        self.__list.append(item)

    def pop(self):
        if self.__list:
            return self.__list.pop()
        else:
            return None

    def peek(self):
        if self.__list:
            return self.__list[-1]
        else:
            return None

    def is_empty(self):
        return self.__list == []

    def size(self):
        return len(self.__list)


def main():
    s = Stack()
    print(f'is_empty={s.is_empty()}')
    for i in range(6):
        s.push(i)
    print(s.peek())
    print(f'is_empty={s.is_empty()}, size={s.size()}')
    while not s.is_empty():
        print(s.pop(), end=',')


if __name__ == '__main__':
    main()
```

```bash
#output
is_empty=True
5
is_empty=False, size=6
5,4,3,2,1,0,
```

## queue

### 单端队列

```python
class Queue(object):
    """借助list来实现queue"""

    def __init__(self):
        self.__list = []

    def enqueue(self, item):
        # 头部添加，尾部取；尾部添加，头部取；都是可以得因为总是一个O(1),一个O(n)
        # 具体看那个操作更加频繁，如果尾部添加更加频繁用list, 头部添加更加频繁用链表
        self.__list.append(item)

    def dequeue(self):
        #判断可以省略，因为list内部会返回None
        if self.__list:
            return self.__list.pop(0)
        else:
            return None

    def is_empty(self):
        return self.__list == []

    def size(self):
        return len(self.__list)


def main():
    q = Queue()
    print(f'is_empty={q.is_empty()}')
    for i in range(6):
        q.enqueue(i)
    print(f'is_empty={q.is_empty()}, size={q.size()}')
    while not q.is_empty():
        print(q.dequeue(), end=',')


if __name__ == '__main__':
    main()
```

```bash
#output
is_empty=True
is_empty=False, size=6
0,1,2,3,4,5,
```

### 双端队列

相当于两个stack底部拼接在一起;

```python
class Deque(object):
    """借助list来实现deque"""

    def __init__(self):
        self.__list = []

    def add_front(self, item):
        self.__list.insert(0, item)

    def add_rear(self, item):
        self.__list.append(item)

    def remove_front(self):
        return self.__list.pop(0)

    def remove_rear(self):
        return self.__list.pop()

    def is_empty(self):
        return self.__list == []

    def size(self):
        return len(self.__list)


def main():
    q = Deque()
    print(f'is_empty={q.is_empty()}')
    for i in range(6):
        q.add_rear(i)
    for i in range(11, 55, 11):
        q.add_front(i)
    print(f'is_empty={q.is_empty()}, size={q.size()}')
    while not q.is_empty():
        print(q.remove_front(), end=',')
    # while not q.is_empty():
    #     print(q.remove_rear(), end=',')

if __name__ == '__main__':
    main()
```

```bash
#output
is_empty=True
is_empty=False, size=10
44,33,22,11,0,1,2,3,4,5,
```