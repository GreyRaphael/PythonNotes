# Python Algorithm

- [Python Algorithm](#python-algorithm)
  - [Time & Space Complecity](#time--space-complecity)
  - [Search](#search)
  - [Sort](#sort)
    - [Bubble Sort](#bubble-sort)
    - [Selection sort](#selection-sort)
    - [Insertion Sort](#insertion-sort)
    - [Quick sort](#quick-sort)
    - [heap sort](#heap-sort)
    - [merge sort](#merge-sort)
    - [Shell Sort](#shell-sort)
    - [sort sunnmary](#sort-sunnmary)
  - [Tree](#tree)
    - [BFS](#bfs)
    - [DFS](#dfs)
    - [tree summary](#tree-summary)
  - [Other Algorithm](#other-algorithm)
  - [BigData Algorithm](#bigdata-algorithm)

## Time & Space Complecity

时间复杂度：用来评估算法运行效率
> 常见时间复杂度: `O(1)<O(logn)<O(n)<O(nlogn)<O(n2)<O(n2logn)<O(n3)`  
> eg. quick-sort: `O(nlogn)`

直接判断时间复杂度:
- 循环减半: O(logn)
- x重循环嵌套: O(n^x)

```py
# O(logn)
n=128

while n>1:
    print(n)
    n=n//2
```

空间复杂度：用来评估算法内存占用大小
- 创建一个变量: O(1)
- 创建一个列表: O(n)
- 创建一个二维列表: O(n2)

## Search

Problem:
- 列表查找：从列表中查找指定元素
  - 输入：列表、待查找元素
  - 输出：元素下标或未查找到元素

Solutions:
- 顺序查找
  - 从列表第一个元素开始，顺序进行搜索，直到找到为止。
    - 最劣时间复杂度$O(n)$
    - 最优时间复杂度$O(1)$
- 二分查找
  - 从有序列表的候选区data[0:n]开始，通过对待查找的值与候选区中间值的比较，可以使候选区减少一半。
    - 序列必须有序
    - 支持下标索引(顺序表)
    - 最劣时间复杂度$O(logn)$
    - 最优时间复杂度$O(1)$

example: 递归与尾递归

```py
# 有一个进入然后返回的过程
def recursive_sum(x):
    return x if x==0 else x+recursive_sum(x-1)

# 只有进入过程，没有返回过程
def tail_recursive_sum(x, result=0):
    if x==0:
        return result
    else:
        return tail_recursive_sum(x-1, result+x)

print(recursive_sum(5))
print(tail_recursive_sum(5))
```

example: 比较顺序查找、二分查找循环实现、二分查找递归实现的运行效率

```py
import time

def calc_time(func):
    def wrapper(*args, **kwargs):
        t1=time.time()
        result=func(*args, **kwargs)
        t2=time.time()
        print(f'{func.__name__} running time: {t2-t1} s')
        return result
    return wrapper

@calc_time
def linear_search(data_set, val):
    for i, d in enumerate(data_set):
        if d==val:
            return i

@calc_time
def bin_search(data_set, val):
    """非递归实现"""
    low = 0
    high = len(data_set)-1

    while low <= high:
        mid = (low+high)//2
        if data_set[mid] == val:
            return mid
        elif val < data_set[mid]:
            high = mid-1
        else:
            low = mid+1
        # print(f'{low},{high}')
    return False

def _recur_bin_search(data_set, val):
    """
    递归实现
    缺点1: 是无法定位目标val的index
    缺点2: 频繁创建list导致内存占用大，还占用了大量运行时间
    """

    n = len(data_set)

    # 递归终止条件
    if n == 0:
        return False
    
    mid = n//2
    # print(f'mid={mid}')
    if data_set[mid] == val:
        return data_set[mid]
    elif val < data_set[mid]:
        return _recur_bin_search(data_set[:mid], val)
    else:
        return _recur_bin_search(data_set[mid+1:], val)

@calc_time
def recur_bin_search(data_set, val):
    # 递归不能加装饰器，所以需要新函数
    return _recur_bin_search(data_set, val)


data=list(range(200000000))
print(linear_search(data, 173320))
print(bin_search(data, 173320))
print(recur_bin_search(data, 173320))
# 速度: bin_search> linear_search > recur_bin_search
```

example: search struct

```py
import time
import random

def generate_data(n):
    result = []
    ids = list(range(1001, 1001+n))
    a1 = ['zhao', 'qian', 'sun', 'li']
    a2 = ['li', 'hao', '', '']
    a3 = ['qiang', 'guo']
    for i in ids:
        name = random.choice(a1)+random.choice(a2)+random.choice(a3)
        student = {'id': i, 'age': random.randint(18, 60), 'name': name}
        result.append(student)
    return result

def bin_search(data_set, val):
    """非递归实现"""
    low = 0
    high = len(data_set)-1

    while low <= high:
        mid = (low+high)//2
        if data_set[mid]['id'] == val:
            return data_set[mid]
        elif val < data_set[mid]['id']:
            high = mid-1
        else:
            low = mid+1
    return False

data = generate_data(1000)
print(bin_search(data, 2000))
```

## Sort

排序算法分类:
- easy: 重点在于**有序区**和**无序区**
  - bubble sort
  - selection sort
  - insertion sort
- 常用
  - quick sort
  - heap sort
  - merge sort
- rarely used
  - shell sort: 希尔排序
  - raidx sort: 基数排序
  - bucket sort: 桶排序

### Bubble Sort

> ![](res/bubble_sort.gif)

```py
import random
import time

def calc_time(func):
    def wrapper(*args, **kwargs):
        t1=time.time()
        result=func(*args, **kwargs)
        t2=time.time()
        print(f'{func.__name__} running time: {t2-t1} s')
        return result
    return wrapper

@calc_time
def bubble_sort(data_set):
    '''时间复杂度是O(n^2)'''
    n=len(data_set)
    for i in range(n-1): # n个元素，只需要n-1趟
        for j in range(n-1-i): # 第1趟循环到index=n-2就ok, 所以是range(n-1-i)
            if data_set[j]>data_set[j+1]:
                data_set[j], data_set[j+1]=data_set[j+1], data_set[j]

@calc_time
def bubble_sort_x(data_set):
    '''
    针对初始就有序的序列进行优化
    最好时间复杂度为O(n), 只扫描一遍
    最坏时间复杂度为O(n^2)
    '''
    n=len(data_set)
    for i in range(n-1):
        exchange=False
        for j in range(n-1-i):
            if data_set[j]>data_set[j+1]:
                data_set[j], data_set[j+1]=data_set[j+1], data_set[j]
                exchange=True
        if not exchange:
            break


# example1: sort
data=list(range(10))
random.shuffle(data)
print(data)
bubble_sort_x(data)
print(data)


# # example2: 比较优化前后版本的效率
# data1=list(range(10000))
# data2=data1.copy() # deep copy

# bubble_sort(data1)
# print(data1)
# bubble_sort_x(data2)
# print(data2)
```

### Selection sort

认为一个序列分成两部分，前部分是排号顺序的，后部分是待遍历的；先找到最小值
- 最优时间复杂度$O(n^2)$, 没有办法优化
- 最劣时间复杂度$O(n^2)$

```py
import random
import time

def calc_time(func):
    def wrapper(*args, **kwargs):
        t1 = time.time()
        result = func(*args, **kwargs)
        t2 = time.time()
        print(f'{func.__name__} running time: {t2-t1} s')
        return result
    return wrapper

@calc_time
def selection_sort(data_set):
    n = len(data_set)
    for i in range(n-1):
        min_loc = i
        for j in range(i+1, n):
            if data_set[j] < data_set[min_loc]:
                min_loc = j
        if min_loc != i:  # 加入这一条能够将不稳定的选择排序变为稳定的
            data_set[i], data_set[min_loc] = data_set[min_loc], data_set[i]

data = list(range(10))
random.shuffle(data)
print(data)
selection_sort(data)
print(data)
```

### Insertion Sort

认为一个序列分成两部分,每次从后部分挑一个到前面部分按顺序的位置
> ![](res/insertion_sort.gif)
- 最优时间复杂度$O(n)$，已经处于有序状态
- 最劣时间复杂度$O(n^2)$
- 排序算法稳定

```py
import random
import time


def calc_time(func):
    def wrapper(*args, **kwargs):
        t1 = time.time()
        result = func(*args, **kwargs)
        t2 = time.time()
        print(f'{func.__name__} running time: {t2-t1} s')
        return result
    return wrapper


@calc_time
def insertion_sort_1(data_set):
    '''method1'''
    n = len(data_set)
    # 从第二个位置，即下标为1的元素开始向前插入
    for i in range(1, n):
        for j in range(i, 0, -1):  # 从右往左判断
            if data_set[j] < data_set[j-1]:
                data_set[j], data_set[j-1] = data_set[j-1], data_set[j]
            else: #这一句是优化，如果判断不成立，再往前的就不用比较了，因为肯定是大于它们的
                break


@calc_time
def insertion_sort_2(data_set):
    '''method2: 优化版本，因为没有进入循环而进行while判断，当有序时，时间复杂度为O(n)'''
    n = len(data_set)
    for i in range(1, n):
        tmp = data_set[i]
        j = i - 1
        while j >= 0 and data_set[j] > tmp:
            data_set[j+1] = data_set[j]
            j = j - 1
        data_set[j + 1] = tmp


data1 = list(range(10000000))
# random.shuffle(data1)
data2=data1.copy()
# print(data1, data2)
insertion_sort_1(data1)
insertion_sort_2(data2)
# print(data1, data2)
```

### Quick sort

关键: 整理分区+递归
> 好写的排序算法里最快的, 快的排序算法里最好写的
- 排序不稳定, 因为有多个相同的元素的时候，会出现左右移动的情况
- 最优时间复杂度O(nlogn), 每一层是n, 共logn层,所以时间复杂度为O(nlogn), 正好mid_value是中间值的情况
- 最劣时间复杂度O(n^2), 正好是顺序导致无法整理分区, logn无法发生作用，每一层为n, 共n层,

快排思路：
> <img src='res/quicksort01.png' width=400>
- 取一个元素p（第一个元素），使元素p归位；
- 列表被p分成两部分，左边都比p小，右边都比p大；
- 递归完成排序。
  > ![](res/quick_sort.gif)

```py
import random
import time
import sys

# 调整递归深度
sys.setrecursionlimit(10000)

def calc_time(func):
    def wrapper(*args, **kwargs):
        t1 = time.time()
        result = func(*args, **kwargs)
        t2 = time.time()
        print(f'{func.__name__} running time: {t2-t1} s')
        return result
    return wrapper

def _quck_sort(data_set, left, right):
    if left < right: # 递归终止条件 left >= right
        mid = partition(data_set, left, right)
        _quck_sort(data_set, left, mid-1)
        _quck_sort(data_set, mid+1, right)

def partition(data_set, left, right):
    tmp = data_set[left]
    while left < right:
        # 下面两个循环使得tmp的左边都<tmp, tmp的右边>=tmp
        # 如果逆序: data_set[right]<tmp, 下面data_set[left]>=tmp
        while left < right and data_set[right] >= tmp:
            right -= 1
        data_set[left] = data_set[right]
        
        while left < right and data_set[left] < tmp:
            left += 1
        data_set[right] = data_set[left]
    
    data_set[left] = tmp
    return left # 此时left==right

@calc_time
def quick_sort(data_set):
    return _quck_sort(data_set, 0, len(data_set)-1)

@calc_time
def sys_sort(data_set):
    '''python自带的sort是Timsort平均复杂度也是O(nlogn)，但是因为调用c, 比自己实现的快速排序要更快'''
    data_set.sort()

data1 = list(range(10000))
random.shuffle(data1)
data2= data1.copy()
print(id(data1)==id(data2)) # False, because is deep copy

quick_sort(data1)
sys_sort(data2)
# print(data1, data2)
```

### heap sort

树是一种可以递归定义的数据结构。树是由n个节点组成的集合：
- 如果n=0，那这是一棵空树；
- 如果n>0，那存在1个节点作为树的根节点，其他节点可以分为m个集合，每个集合本身又是一棵树。

一些概念：
- 父节点、子节点、根节点、叶子节点
- 树的高度(深度)、树的度
- 子树

二叉树(binary tree)：度不超过2的树(包括完全二叉树、满二叉树)
> 二叉树可以用列表来存储，通过规律可以从父亲找到孩子或从孩子找到父亲

二叉树存储方式:
- 链式存储
- 顺序存储：即列表

顺序存储父子节点关系：对于父节点i, 左子节点为2i+1, 右子节点为2i+2
> <img src='res/binary_tree_linear_storage.png' width=350>

堆：
- 大根堆：一棵完全二叉树，满足任一节点都比其孩子节点大
- 小根堆：一棵完全二叉树，满足任一节点都比其孩子节点小

节点的左右子树都是堆，但自身不是堆: 当根节点的左右子树都是堆时，可以通过一次向下的调整来将其变换成一个堆。
> <img src='res/heap_adjust.gif' width=350>

堆排序过程:
1. 建立堆
1. 得到堆顶元素，为最大元素
1. 去掉堆顶，将堆最后一个元素放到堆顶，此时可通过**一次调整**重新使堆有序。
1. 堆顶元素为第二大元素。
1. 重复步骤3，直到堆变空。

建立堆: 多次调整得到大根堆
> ![](res/construct_heap.gif)

堆排序:(直接取出堆顶最大)
> ![](res/head_sort01.gif)

堆排序:为了节约内存，交换堆顶最大和完全二叉树最后一个
> ![](res/heap_sort02.gif)

```py
import random
import time

def calc_time(func):
    def wrapper(*args, **kwargs):
        t1 = time.time()
        result = func(*args, **kwargs)
        t2 = time.time()
        print(f'{func.__name__} running time: {t2-t1} s')
        return result
    return wrapper

def sift(data_set, low, high):
    i = low
    j = 2 * i + 1
    tmp = data_set[i]
    while j <= high:  # 孩子在堆里
        if j < high and data_set[j] < data_set[j+1]:  # 如果有右孩子且比左孩子大
            j += 1  # j指向右孩子
        if data_set[j] > tmp:  # 孩子比最高领导大
            data_set[i] = data_set[j]  # 孩子填到父亲的空位上
            i = j  # 孩子成为新父亲
            j = 2 * i + 1  # 新孩子
        else:
            break
    data_set[i] = tmp  # 将堆顶换到这个位置

@calc_time
def heap_sort(data_set):
    n = len(data_set)
    # 初次建堆
    for i in range(n // 2 - 1, -1, -1):
        sift(data_set, i, n - 1)
    # 每次调整之后，将堆顶从树的最后节点一直往前放
    for i in range(n-1, -1, -1):  # i指向未排序堆部分的最后
        data_set[0], data_set[i] = data_set[i], data_set[0]  # 堆顶逐步换到后面
        sift(data_set, 0, i - 1)  # 调整出新堆顶最大


data1 = list(range(10))
random.shuffle(data1)

print(data1)
heap_sort(data1)
print(data1)
```

example: 逆序heap sort

```py
def sift(data, low, high):
    '''一次调整'''
    i = low
    j = 2 * i + 1
    tmp = data[i]
    while j <= high:
        if j < high and data[j] > data[j + 1]:
            j += 1
        if tmp > data[j]:
            data[i] = data[j]
            i = j
            j = 2 * i + 1
        else:
            break
    data[i] = tmp
```

### merge sort

一次归并: 列表分**两段有序**，通过**一次归并**将其**合成为一个有序列表**
> <img src='res/merge_once.gif' width=350>

归并排序：先拆开再合并(用到了两个游标，然后互相比较)；也要用到递归
> <img src='res/merge_sort.png' width=350>
- 分解：将列表越分越小，直至分成一个元素。
- 一个元素是有序的。
- 合并：将两个有序列表归并，列表越来越大。


拆开的时候需要$logn$层，每一层n次；合并的时候也需要合并$logn$层，每层循环n次，时间复杂度$2nlogn$也就是$O(nlogn)$
- 最优时间复杂度$O(nlogn)$
- 最劣时间复杂度$O(nlogn)$
- 排序算法稳定
- 空间复杂度O(n)

```py
import random
import time

def calc_time(func):
    def wrapper(*args, **kwargs):
        t1 = time.time()
        result = func(*args, **kwargs)
        t2 = time.time()
        print(f'{func.__name__} running time: {t2-t1} s')
        return result
    return wrapper

def merge(data, low, mid, high):
    '''一次归并'''
    i = low
    j = mid+1
    ltmp = []
    # 两边有序序列分别比较添加进入ltmp
    while i <= mid and j <= high:
        if data[i] <= data[j]:
            ltmp.append(data[i])
            i += 1
        else:
            ltmp.append(data[j])
            j += 1

    # 剩下的序列ltmp补充道ltmp
    while i <= mid:
        ltmp.append(data[i])
        i += 1
    while j <= high:
        ltmp.append(data[j])
        j += 1
    data[low:high+1] = ltmp

def _merge_sort(data, low, high):
    if low < high:
        mid = (low+high)//2
        _merge_sort(data, low, mid)
        _merge_sort(data, mid+1, high)
        merge(data, low, mid, high)

@calc_time
def merge_sort(data):
    _merge_sort(data, 0, len(data)-1)


data1 = list(range(10))
random.shuffle(data1)

print(data1)
merge_sort(data1)
print(data1)
```

### Shell Sort

希尔排序:分组插入排序算法。希尔排序每趟并不使某些元素有序，而是使整体数据越来越接近有序；最后一趟排序使得所有数据有序。
> <img src='res/shell_sort.gif' width=350>
- 首先取一个整数d1=n/2，将元素分为d1个组，每组相邻量元素之间距离为d1，在各组内进行直接插入排序；
- 取第二个整数d2=d1/2，重复上述分组排序过程，直到di=1，即所有元素在同一组内进行直接插入排序。

希尔排序性质:
-  最坏时间复杂度$O(n^2)$, gap=1的时候
-  最优时间复杂度$O(n^{5/4})$, 统计结果
-  排序算法不稳定

```python
# 直接在插入排序基础上直接修改1 为gap
import random
import time

def calc_time(func):
    def wrapper(*args, **kwargs):
        t1 = time.time()
        result = func(*args, **kwargs)
        t2 = time.time()
        print(f'{func.__name__} running time: {t2-t1} s')
        return result
    return wrapper

@calc_time
def shell_sort_1(data_set):
    '''method1'''
    n = len(data_set)
    gap = n//2
    while gap >= 1:
        for i in range(gap, n):
            for j in range(i, 0, -1):  # 从右往左判断
                if data_set[j] < data_set[j-gap]:
                    data_set[j], data_set[j-gap] = data_set[j-gap], data_set[j]
                else:
                    break
        gap = gap//2

@calc_time
def shell_sort_2(data_set):
    '''method2: 优化版本'''
    n = len(data_set)
    gap = n//2
    while gap >= 1:
        for i in range(gap, n):
            tmp = data_set[i]
            j = i - gap
            while j >= 0 and data_set[j] > tmp:
                data_set[j+gap] = data_set[j]
                j -= gap
            data_set[j + gap] = tmp
        gap //= 2


data1 = list(range(20))
random.shuffle(data1)
data2 = data1.copy()

print(data1, data2)
shell_sort_1(data1)
shell_sort_2(data2)
print(data1, data2)
```

### sort sunnmary

排序算法的稳定性: 同一序列的两个相同数值，排序完毕后，相对位置是否发生变化

| Name           | Average | Best  | Worst | Memory  | Stable  |
|----------------|---------|-------|-------|---------|---------|
| bubble sort    | n^2     | n     | n^2   | 1       | Yes     |
| insertion sort | n^2     | n     | n^2   | 1       | Yes     |
| selection sort | n^2     | n^2   | n^2   | 1       | depends |
| quick sort     | nlogn   | nlogn | n^2   | logn    | depends |
| heap sort      | nlogn   | nlogn | nlogn | 1       | No      |
| merge sort     | nlogn   | nlogn | nlogn | depends | Yes     |
| shell sort     | n^(5/4) | nlogn |       |         | No      |
| Timsort        | nlogn   | n     | nlogn | n       | Yes     |
| Introsort      | nlogn   | nlogn | nlogn | logn    | No      |

快速排序、归并排序、堆排序的时间复杂度都是O(nlogn)j; 但是一般运行时间quick-sort < merge-sort < heap-sort, 因为快排的常数项比其他两个小
- 快速排序：极端情况下排序效率低, O(n^2)
- 归并排序：需要额外的内存开销
- 堆排序：在快的排序算法中相对较慢

## Tree

树的顺序存储比较少，而链式存储比较常见;

树的实现需要队列，从右边加入，从左边读取;实现广度遍历;

### BFS

```python
class Node(object):
    def __init__(self, item):
        self.elem = item
        self.lchild = None
        self.rchild = None


class Tree(object):
    def __init__(self):
        self.root = None

    def add(self, item):
        node = Node(item)
        if self.root is None:
            self.root = node
            return

        queue = [self.root, ]  # bool([None, ])结果是True

        while queue:
            cur_node = queue.pop(0)
            if cur_node.lchild is None:
                cur_node.lchild = node
                return
            else:
                queue.append(cur_node.lchild)

            if cur_node.rchild is None:
                cur_node.rchild = node
                return
            else:
                queue.append(cur_node.rchild)

    def breadth_travel(self):
        if self.root is None:
            return
        queue = [self.root, ]
        while queue:
            cur_node = queue.pop(0)
            print(f'current elem={cur_node.elem}')
            if cur_node.lchild:
                queue.append(cur_node.lchild)
            if cur_node.rchild:
                queue.append(cur_node.rchild)

    def deepth_travel(self):
        pass


def main():
    tree = Tree()
    for i in range(10):
        tree.add(i)
    tree.breadth_travel()


if __name__ == '__main__':
    main()
```

```bash
#output
current elem=0
current elem=1
current elem=2
current elem=3
current elem=4
current elem=5
current elem=6
current elem=7
current elem=8
current elem=9
```

### DFS

```python
class Node(object):
    def __init__(self, item):
        self.elem = item
        self.lchild = None
        self.rchild = None


class Tree(object):
    def __init__(self):
        self.root = None

    def add(self, item):
        node = Node(item)
        if self.root is None:
            self.root = node
            return

        queue = [self.root, ]  # bool([None, ])结果是True

        while queue:
            cur_node = queue.pop(0)
            if cur_node.lchild is None:
                cur_node.lchild = node
                return
            else:
                queue.append(cur_node.lchild)

            if cur_node.rchild is None:
                cur_node.rchild = node
                return
            else:
                queue.append(cur_node.rchild)

    def breadth_travel(self):
        if self.root is None:
            return
        queue = [self.root, ]
        while queue:
            cur_node = queue.pop(0)
            print(f'current elem={cur_node.elem}')
            if cur_node.lchild:
                queue.append(cur_node.lchild)
            if cur_node.rchild:
                queue.append(cur_node.rchild)

    def pre_order(self, node):
        """根左右，先序遍历"""
        # 递归终止条件
        if node is None:
            return
        print(node.elem, end=',')
        self.pre_order(node.lchild)
        self.pre_order(node.rchild)

    def mid_order(self, node):
        """左根右，中序遍历"""
        # 递归终止条件
        if node is None:
            return
        self.mid_order(node.lchild)
        print(node.elem, end=',')
        self.mid_order(node.rchild)

    def pos_order(self, node):
        """左右根，后序遍历"""
        # 递归终止条件
        if node is None:
            return
        self.pos_order(node.lchild)
        self.pos_order(node.rchild)
        print(node.elem, end=',')

def main():
    tree = Tree()
    for i in range(10):
        tree.add(i)
    tree.pre_order(tree.root)
    print()
    tree.mid_order(tree.root)
    print()
    tree.pos_order(tree.root)

if __name__ == '__main__':
    main()
```

```bash
#output
0,1,3,7,8,4,9,2,5,6,
7,3,8,1,9,4,0,5,2,6,
7,8,3,9,4,1,5,6,2,0,
```

### tree summary

广度遍历:
- 已知一个tree→广度遍历
- 已知一个广度遍历的结果→获得tree的结构

深度遍历:
- 已知一个tree→前中后序遍历
- 单独知道前中后序遍历中的一个不能得到tree的结构
- 前中序遍历→获得tree的结构
- 中后序遍历→获得tree的结构
- 前后序遍历→无法获得tree的结构, 因为左右子树分不开

那么可以先通过前中序的遍历结果得到tree的结构，然后写出后序遍历的结果；

## Other Algorithm

example: 现在有一个列表，列表中的数范围都在0到100之间，列表长度大约为100万。设计算法在O(n)时间复杂度内将列表进行排序。
- 这种情况排序速度: 计数排序>归并排序>快排  
- 因为数据范围小，重复的多，容易出现一边特别少，logn失效

因为下面$\Sigma count = 1000000$, 所以时间复杂度为O(n)

```py
# key: 创建一个列表，用来统计每个数出现的次数
import random
import time

def calc_time(func):
    def wrapper(*args, **kwargs):
        t1 = time.time()
        result = func(*args, **kwargs)
        t2 = time.time()
        print(f'{func.__name__} running time: {t2-t1} s')
        return result
    return wrapper

@calc_time
def count_sort(data, low, high):
    count_list=[0 for _ in range(high-low)]
    for d in data:
        count_list[d-low]+=1
    
    i=0
    for j, count in enumerate(count_list):
        for _ in range(count):
            data[i]=j+low
            i+=1

@calc_time
def sys_sort(data):
    data.sort()

# 0~100
data1=random.choices(range(0, 101), k=1000000)
data2=data1.copy()
count_sort(data1, 0, 101)
sys_sort(data2)
```

example: 现在有n个数（n>10000），设计算法，按大小顺序得到前10大的数

method1: 插入排序修改
- 构造一个新列表来容纳top 10(增加一个冗余位置)
- 从待排序列表中抽出每一个数，看看是否能够插入新列表
- 所以时间复杂度为O(10n), 也就是O(n)

```py
# 找出列表中最大的k个数
import random

def topk(data, k):
    ltmp = [0 for _ in range(k+1)]  # 给j+1留下一个冗余位置

    for val in data:  # 抽一个数
        j = k-1  # j 从 k-1开始一直往左移动
        while j >= 0 and val > ltmp[j]:
            ltmp[j+1] = ltmp[j]
            j -= 1
        ltmp[j+1] = val  # 将抽到的数插入ltmp合适的位置
    return ltmp[:k]

data1 = list(range(1000))
random.shuffle(data1)
print(topk(data1, 10))
```

```py
# 兼容能够找出最大的或者最小的k个数: easy understand
import random

def topk(data, k, reverse=False):
    max_num, min_num = max(data), min(data)
    # k+1是为了给j+1留下一个冗余位置
    ltmp = [min_num if reverse else max_num for _ in range(k+1)]

    for val in data:  # 抽一个数
        j = k-1  # j 从 k-1开始一直往左移动
        while j >= 0:
            if reverse:
                # True: 找出最大的k个
                if val < ltmp[j]:
                    break
            else:
                # False: 找出最小的k个
                if val > ltmp[j]:
                    break
            ltmp[j+1] = ltmp[j]
            j -= 1
        ltmp[j+1] = val  # 将抽到的数插入ltmp合适的位置
    return ltmp[:k]


data1 = list(range(1000))
random.shuffle(data1)
print(topk(data1, 10, reverse=False))
print(topk(data1, 10, reverse=True))
```

```py
import random


def topk(data, k, reverse=False):
    top = data[:k+1]
    # 随便用一个排序使得top有序，方便后续插入
    top.sort(reverse=reverse)
    # print(top)

    for i in range(k+1, len(data)):
        j = k-1
        while j >= 0:
            if reverse:
                # True: 找出最大的k个数
                if data[i] < top[j]:
                    break
            else:
                # False 找出最小的k个数
                if data[i] > top[j]:
                    break
            top[j+1] = top[j]
            j -= 1
        top[j+1] = data[i]

    return top[:-1]


def insert(li, i, reverse=False):
    tmp = li[i]
    j = i-1
    while j >= 0:
        if reverse:
            # True: 找出最大的k个数
            if tmp < li[j]:
                break
        else:
            # False 找出最小的k个数
            if tmp > li[j]:
                break
        li[j+1] = li[j]
        j -= 1
    li[j+1] = tmp


def insert_sort(data, reverse=False):
    for i in range(1, len(data)):
        insert(data, i, reverse=reverse)


def topk2(data, k, reverse=False):
    top = data[:k+1]
    insert_sort(top, reverse=reverse) # 采用插入排序
    # print(top)

    for i in range(k+1, len(data)):
        top[k] = data[i]
        insert(top, k, reverse=reverse)
    return top[:-1]


data1 = list(range(1000))
random.shuffle(data1)

print(topk(data1, 10, reverse=False))
print(topk(data1, 10, reverse=True))

print(topk2(data1, 10, reverse=False))
print(topk2(data1, 10, reverse=True))
```

method2: 堆排序修改
- 初始10个数构造一个堆: 最大的k个数是小根堆，最小的k个数是大根堆
- 后面的数逐个与根节点比较: 并进行调整
- 所以时间复杂度为O(log(10)*n)也就是O(n)

```py
# 找出最大的10个数
import random

def sift(data, low, high):
    i = low
    j = 2 * i + 1
    tmp = data[i]
    while j <= high:
        if j < high and data[j] > data[j + 1]:
            j += 1
        if tmp > data[j]:
            data[i] = data[j]
            i = j
            j = 2 * i + 1
        else:
            break
    data[i] = tmp


def topk(data, k):
    heap = data[:k]
    # 构造size=k的小根堆
    for i in range(k//2-1, -1, -1):
        sift(heap, i, k-1)
    # print(heap)

    # 遍历data剩下的数据
    for j in range(k, len(data)):
        if data[j] > heap[0]:
            heap[0] = data[j]
            sift(heap, 0, k-1)
    print(heap)
    # 从堆里面出数
    for i in range(k-1, -1, -1):
        heap[0], heap[i] = heap[i], heap[0]
        sift(heap, 0, i-1)
    return heap

data1 = list(range(100))
random.shuffle(data1)
print(topk(data1, 10))
```

```py
# 找出最小的10个数: 上面基础上修改3个符号
import random


def sift(data, low, high):
    i = low
    j = 2 * i + 1
    tmp = data[i]
    while j <= high:
        if j < high and data[j] < data[j + 1]:
            j += 1
        if tmp < data[j]:
            data[i] = data[j]
            i = j
            j = 2 * i + 1
        else:
            break
    data[i] = tmp


def topk(data, k):
    heap = data[:k]
    # 构造size=k的大根堆
    for i in range(k//2-1, -1, -1):
        sift(heap, i, k-1)
    # print(heap)

    # 遍历data剩下的数据
    for j in range(k, len(data)):
        if data[j] < heap[0]:
            heap[0] = data[j]
            sift(heap, 0, k-1)
    print(heap)
    # 从堆里面出数
    for i in range(k-1, -1, -1):
        heap[0], heap[i] = heap[i], heap[0]
        sift(heap, 0, i-1)
    return heap

data1 = list(range(100))
random.shuffle(data1)
print(topk(data1, 10))
```

优先队列(本质是heap)：一些元素的集合，POP操作每次执行都会从优先队列中弹出最大（或最小）的元素

example: python内置的`heapq`可以实现优先队列

```py
import heapq
import random

data1 = list(range(30))
random.shuffle(data1)

# heapq实现top k
min10 = heapq.nsmallest(10, data1)
max10 = heapq.nlargest(10, data1)
print(min10, max10)

print('-'*30)
data2 = data1.copy()
print(data2)
heapq.heapify(data2) # 小根堆
print(data2)

print('-'*30)
heap=[]
for i in data1:
    heapq.heappush(heap, i)
print(heap)

print('-'*30)
for i in range(len(heap)):
    # 每次pop最小
    print(heapq.heappop(heap), end=',')
print(heap) # []
```

example: heapq实现堆排序

```py
import heapq
import random

def heap_sort(data, reverse=False):
    heap = []
    for i in data:
        if reverse:
            # True: 逆序
            heapq.heappush(heap, -i)
        else:
            # 正序
            heapq.heappush(heap, i)
    return [-heapq.heappop(heap) if reverse else heapq.heappop(heap) for _ in range(len(data))]

data1 = list(range(30))
random.shuffle(data1)
print(data1)

data2 = heap_sort(data1)
# 通过相反数实现逆序
data3 = heap_sort(data1, reverse=True)
print(data2)
print(data3)
```

example: 给定一个升序列表和一个整数，返回该整数在列表中的下标范围。
> 例如：列表[1,2,3,3,3,4,4,5]，若查找3，则返回(2,4)；若查找1，则返回(0,0)

```py
# 在binary_search基础上修改
def bin_search(data_set, val):
    """非递归实现"""
    n = len(data_set)
    low = 0
    high = n-1

    while low <= high:
        mid = (low+high)//2
        if data_set[mid] == val:
            left = right = mid
            while left >= 0 and data_set[left] == val:
                left -= 1
            while right < n and data_set[right] == val:
                right += 1
            return (left+1, right-1)
        elif val < data_set[mid]:
            high = mid-1
        else:
            low = mid+1
    return -1

data = [1, 2, 3, 3, 3, 4, 4, 5]
print(bin_search(data, 1)) # (0,0)
print(bin_search(data, 3)) # (2, 4)
print(bin_search(data, 6)) # -1
```

example: 给定一个列表和一个整数，设计算法找到两个数的下标，使得两个数之和为给定的整数。保证肯定仅有一个结果。[Ref in leetcode](https://leetcode.com/problems/two-sum/?tab=Description)
> 例如，列表[1,2,5,4]与目标整数3，1+2=3，结果为(0, 1)

```py
# 搜索全部
def func1(data, target):
    n = len(data)
    for i in range(n):
        for j in range(i+1, n):
            if data[i]+data[j] == target:
                return (i, j)
    return -1


def binary_search(data_set, val, low, high):
    while low <= high:
        mid = (low+high)//2
        if data_set[mid] == val:
            return mid
        elif data_set[mid] < val:
            low = mid+1
        else:
            high = mid-1

# 剩下的部分采用二分查找
def func2(data, target):
    tmp = data.copy()
    tmp.sort()
    for i, val in enumerate(tmp):
        b = binary_search(tmp, target-val, i+1, len(tmp)-1)
        if b:
            return (data.index(val), data.index(tmp[b]))


data = [1, 2, 5, 4]
target = 7
print(func1(data, target))
print(func2(data, target))
```

## BigData Algorithm

problem1: 海量数据的topk问题
- 如果能够加载如内存，就用quick-sort
- 如果无法放入内存，采用heap-sort
- 如果所有种类是可以枚举的，那么采用bitmap(状态压缩，桶排序的思想), e.g. 如何判断一个整数是否在40亿个不重复的unsigned int中(unsigned int总共2^32=42亿个),采用1个bit表示整数,那么需要2^32需要512MB就行，然后就可以判断了
- Bloom Filter(状态压缩)检测存在还是不存在，只要哈希方程选的好，哈希碰撞小，就可以进行状态压缩，将不能装入内存的数据进而可以装入内存. e.g. A, B两个文件各存放50亿个url, 每个url占64Bytes, 内存限制是4G，找出A, B共同的url
- 外排序，多台电脑对不同文件进行排序，将多个已经排好序的文件进行归并排序，多路归并复杂度是O(n)
- 胜者树，败者树

