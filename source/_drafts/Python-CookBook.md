---
title: Python CookBook
date: 2020-05-07 15:31:33
tags: Python
categories: 读书笔记
---
Python是一门非常好作为入门学习的编程语言，语言简洁干练，拥有强大的第三方库，可以用来做机器学习、数据分析、爬虫等等。但是，很多人入门之后对Python的认知就停留在了比较浅显的认知上，成为了调包侠， Python本身有一些特性和库（如collections、future、generator等）是值得并且需要仔细来琢磨的。[Python Cookbook](https://book.douban.com/subject/4828875/) 就是一本可以作为进阶学习Python的书籍，里面的很多技巧和方法论非常实用。
<!-- more -->

# 数据结构和算法
## 迭代器
迭代器有两个基本的方法：iter()和next()。我们可以实现自己的迭代器，通过实现__iter__和__next__方法。
```python
class MyNumbers:
    def __iter__(self):
        self.start = 0
        return self
    def __next__(self):
        self.start += 1
        if self.start > 10:
            raise StopIteration
        return self.start
my_number = MyNumbers()
my_iter = iter(my_number)
for x in my_iter:
    print(x)
```    
## 序列分解
任何的序列（或者是可迭代对象）可以通过一个简单的赋值操作来分解为单独的变量, 如果变量总数和序列里的元素数目不匹配会抛出ValueError，不过我们可以用*操作符来处理这种情况。
```Python
>>> a = [1, 2 , 3]
>>> x, y, z = a
>>> x
1
>>> x, y = a
ValueError: too many values to unpack (expected 2)
>>> *x, y = a
>>> x
[1, 2]
>>> x, *_, y = a  #如果你想丢弃部分元素
>>> x
1
>>> y
3
```
## Deque
Deque可以以O(1)的时间复杂度在头部和尾部进行append和pop的操作，既可以作为queue也可以作为stack来使用。
> Deques support thread-safe, memory efficient appends and pops from either side of the deque with approximately the same O(1) performance in either direction

```Python
>>> from collections import deque
>>> q = deque(maxlen=3)
>>> q.append(1)
>>> q.append(2)
>>> q.appendleft(3)
>>> q
deque([3,1,2])
>>> q.pop()
2
>>> q.popleft()
3
```
## Heapq
Heapq就是堆数据结构（也叫优先队列），他满足**heap[k] <= heap[2*k+1] and heap[k] <= heap[2*k+2]**, heapq是最小堆， heapq[0]得到的总是最小值。
```python
import heapq
nums = [1, 8, 2, 23, 7, -4, 18, 23, 42, 37, 2]
print(heapq.nlargest(3, nums)) # Prints [42, 37, 23]
print(heapq.nsmallest(3, nums)) # Prints [-4, 1, 2]
#优先队列
class PriorityQueue:
    def __init__(self):
        self._queue = []
        self._index = 0
    def push(self, item, priority):
        heapq.heappush(self._queue, (-priority, item, self._index))
        self._index += 1
    def pop(self):
        return heapq.heappop(self._queue)[-1]
```
heap[0]和heapq.heappop()操作永远得到当前最小的元素，每次操作的时间复杂度仅仅为logN，所以当需要得到n个最小的元素并且n远远小于列表的长度的时候，通过heapq来实现是比较好的。

## 字典
常常我们想要一个key对应多个值，容易写出
```python
d = {}
if v in d:
    d[v].append()
else:
    d[v] = []
#或者
d.setdefault(v, []).append()
```
这样的代码块，setdefault相对方便一点，但是每次都要调用很繁琐，其实我们可以直接用**defaultdict**
```python
from collections import defaultdict
d = defaultdict()
d['test'].append('test')
```
而有的时候当我们迭代字典的时候希望能够保留插入的顺序，那么可以使用**OrderedDict**
```python
from collections import OrderedDict
d = OrderedDict()
d['test1'] = 1
d['test2'] = 2
for k, v in d.items():
    print(k, v)
```
如果要比较字典的键值的相同或者不同，也有方便的集合操作符
```python
a.keys() & b.keys()   #intersection
a.keys() - b.keys()   #difference
a.items() & b.items()
a.items() - b.items()
```
需要注意的是**a.values()**方法并不支持这样的集合操作。

## 命名切片
在做数据处理的时候，常常需要频繁的对序列进行切片，满屏幕的切片操作常常会让代码可读性大大降低，命名切片可以提高代码可读性和维护性。
```python
>>> a = slice(0, 10, 2) #start, stop, step
>>> b = [1,2,3,4,5]
>>> b[a]
[1,3,5]
>>> a.indices(5)       #适配一个合适的范围
(0, 5, 2)
```
## 制表或者计数
对于字典而言，一个常见的用法是，我们经常用来统计元素的频率
```python
for e in dic:
    if e in dic:
        dic[e] += 1
    else:
        dic[e] = 1
```
不过其实python的**Counter**可以直接用来解决此类问题
```python
from collections import Counter
a = [1,2,3,4,3]
fre = Counter(a)
fre.most_common(3)          #top 3 出现的元素
#一个有意思的地方是，Counter可以直接做数学操作
fre2 = Counter([1,2,3,4])
fre - fre2                 #difference
fre + fre2                 #union
```
## 字典和对象排序
如果需要对数组字典进行排序的话，我们常常的做法是
```python
sorted(data, key=lambda...)
```
python其实提供一个专门的函数来处理这种情况，**itemgetter**
```python
from operator import itemgetter
sroted(data, key=itemgetter('field1', 'field2'))
#min,max也可以使用
max(l, key=itemgetter('field1))
```
类似得，如果是对数组对象进行排序的话，我们可以使用**attrgetter**
```python
from operator import attrgetter
sorted(users, key=attrgetter('user_id'))
```
值得注意是，虽然两种做法都可以，但是来自operator的做法会更快一点
## 数据分组
常用的数据分组我们会用pandas来进行，其实python的itertools包也提供类似的功能
```python
from operator import itemgetter
from itertools import groupby
data.sort(key=itemgetter('user_id'))
groupby(data, key=itemgetter('user_id'))
```
值得注意的地方是，**groupby**的逻辑是检查连续的元素，所以事先我们需要对数据进行第一轮排序。
## 数据过滤
数据过滤有很多做法，除去简单的for loop，我们可以使用过滤器、迭代器、生成器以及itertool里面的工具
```python
[n for n in l if n > 0]        #list comprehension
(n for n in l if n > 0)        #generator 
list(filter(lambda x:.., l))   #filter
[n if n >0 else -1 for n in l] #transformation

from itertools import compress
a = [1,2,3,4,5]
b = [ i > 5 for i in a]
list(compress(a, b))            #根据b中的布尔值过滤对应的元素 
```
## namedtuple
当需要频繁大量的通过下标来访问tuple的时候，我们可以使用namedtuple来提高程序的可读性
```python
>>> from collections import namedtuple
>>> User = namedtuple('User', ['user_id', 'username'])
>>> u = User('no1', 'jacky')
>>> u.username
jacky
```
## 合并字典
合并字典也是常见的操作之一，我们经常会写if代码块来做这样的工作，或者使用update方法，这样会创建一个新的字典，但是其实可以直接使用 ChainMap来处理字典的合并查询，合成的新对象只是两个字典逻辑上的合并，并没有新的字典对象被创造出来
```python
a = {'x':1, 'z':3}
b = {'y':2, 'z':4}
b.update(a)        #用a来覆盖b，相同的key的话，会采用a中的值  

from collections import ChainMap
a = {'x':1, 'z':3}
b = {'y':2, 'z':4}
c = ChainMap(a, b)   #用a来覆盖b

#ChainMap一个有意思的地方是可以很方便的作为编程语言作用域的处理
env = ChainMap()
env['x'] = 1
env = env.new_child()
env['x'] = 2
env = env.new_child()
#现在env['x']等于2
env = env.parents
#现在env['x']等于1
```

#字符串处理
几乎所有有用的程序都会涉及到某些文本处理，不管是解析数据还是产生输出。 这一章将重点关注文本的操作处理，比如提取字符串，搜索，替换以及解析等。
##字符串分割匹配和搜索
**string**对象的**split**方法适用范围比较狭窄，而**re.split**则可以适用更广范围的字符串分割
```python
>>> a = "a   b   c"
>>> a.split(' ')
['a', '', '', 'b', '', '', 'c'] 
>>> re.split(r'\s+', a)
['a','b','c']
#re.split还可以进行多字符分割
>>> a = 'asdf fjdk; afed, fjek,asdf, foo'
>>> re.split(r'[;,\s]\s*', a)
['asdf', 'fjdk', 'afed', 'fjek', 'asdf', 'foo']
```
同理，对于**replace**方法，我们也可以使用**re.sub**来实现更高级的文本替代
```python
>>> a = 'Today is 11/27/2012. PyCon starts 3/13/2013.'
>>> re.sub(r'(\d+)/(\d+)/(\d+)', r'\3-\1-\2', a)
'Today is 2012-11-27. PyCon starts 2013-3-13.'
#我们还可以使用命名分组来实现上述的功能
>>> re.sub(r'(?P<month>\d+)/(?P<day>\d+)/(?P<year>\d+)', r'\g<year>-\g<month>-\g<day>', a)
'Today is 2012-11-27. PyCon starts 2013-3-13.'
#更高级点的用法， 是可以传入回掉函数来处理sub
>>> re.sub(r'(\d+)\(\d+)\(\d+)', lambda x:x, a)
```
在匹配中我们可以用?如(.*?)来做最短匹配，也可以使用?:如(?:\d+)来指定一个非捕获匹配。
##字符串格式化
我们可以使用**rjust()**，**ljust()** 以及 **center()** 来做字符串对齐，比如
```python
>>> text = 'Hello World'
>>> text.ljust(20)
'Hello World         '
>>> text.rjust(20)
'         Hello World'
>>> text.center(20, '*')
'****Hello World*****'
```

**format()** 也可以做到这一点
```python
>>> format(text, '>20')
'         Hello World'
>>> format(text, '<20')
'Hello World         '
>>> format(text, '^20')
'    Hello World     '
>>> format(text, '=>20s')
'=========Hello World'
>>> '{:>10s} {:>10s}'.format('Hello', 'World')
'     Hello      World'
```

而且，我们还可以通过**textwrap** 模块来控制输出字符串的列宽
```python
import textwrap
textwrap.fill(s, 40)
#还可以配合os.get_terminal_size()来格式化输出
import os
col = os.get_terminal_size().columns
textwrap.fill(s, col)
```
#数字日期和时间
##数字的格式化
如果需要对浮点数进行四舍五入，我们可以直接使用**round**
```python
>>> round(1.23, 1)
1.2
>>> round(1.27, 1)
1.3
>>> round(1644, -1)
1640
>>> round(1655, -2)
1700
```
众所周知，浮点数计算是会存在误差的，所以如果需要更精确的浮点数计算，我们可以使用**Decimal**
```python
>>> from decimal import Decimal
>>> a = Decimal('4.2')
>>> b = Decimal('2.1')
>>> a / b
Decimal('3.333333333333333333333333333')
#还可以控制计算的规范
>>> from decimal import localcontext
>>>  with localcontext() as ctx:
>>>    ctx.prec = 3
>>>    print(a / b)
3.33
```
数字进制进行转换的时候，我们常常使用**bin()** ， **orc()** 以及 **hex()** ， 但**format**也可以完成
```python
>>> x = 1234
>>> bin(x)
'0b10011010010'
>>> oct(x)
'0o2322'
>>> hex(x)
'0x4d2'
>>> format(x, 'b')
'10011010010'
>>> format(x, 'o')
'2322'
>>> format(x, 'x')
'4d2'
#反过来可以直接用int
>>>  int('10011010010', 2)
1234
```
## 随机选择
**random** 模块有很多函数来进行随机选择或者生成元素
```python
>>> import random
>>> values = [1,2,3,4]
>>> random.choice(values)  #随机选择
2
>>> random.sample(values, 3) #随机抽样
[3, 1, 2]
>>> random.shuffle()   #随机打乱
>>> random.randint(0, 10) #随机数
>>> random.random()  # 0 - 1
>>> random.getrandbits(2) #N bit 随机数
```
##日期和时间
python的datetime是一个模块，有好几个类用来进行时间和日期的操作
* date 日期类
* time 时间类
* datetime 日期时间
* timedelta 表示时间差
* tzinfo  时区信息
* timezone  时区

用datetime和timedelta可以进行时间计算
```python
>>> from datetime import timedelta, datetime
>>> a = datetime(2012, 9, 23)
>>> print(a + timedelta(days=10))
2012-10-03 00:00:00
>>> b = datetime(2012, 12, 21)
>>> b - a
datetime.timedelta(days=89)

```
字符串转为时间
```python
>>> cday = datetime.strptime('2020-12-01 19:23:23', '%Y-%m-%d, %H:%M:%S')
```
时间格式化
```python
>>> now = datetime.now()
>>> print(now.strftime('%a, %b %d %H:%M'))
```
时区的设置
```python
>>> tz = timezone(timedelta(hours=8))
>>> now = datetime.now()
>>> dt = now.astimezone(tz)
```
 需要注意的一点是, **strptime** 性能并不好，并且关于时区的问题，最好是使用**pytz** 模块
 ```python
>>> from pytz import timezone
>>> central = timezone('US/Central)
>>> now = central.localize(datetime.now()) #得到美国时区的当前时间
>>>  now.astimezone(timezone('Asia/Shanghai')) #得到上海时区
 ```
**pytz** 是一个第三方库，需要安装
#迭代器与生成器 
迭代器其实就是实现了迭代协议的对象，任何一个实现了__next__方法的对象都是一个迭代器，我们常常会把__iter__也实现了，当调用iter(obj)的时候，其实就是调用了对象的__iter__方法 
```python
class Iter:
    def __init__(self):
        self.val = 0
    def __iter__(self):
        return self
    def __next__(self):
        self.val += 1
        if self.val > 10:
            raise StopIteration
        return self.val
it  = iter(Iter())
for v in it:
    print(it)
```
那生成器是什么呢？**yield**就构建了一个生成器，生成器实现了迭代器协议
```python
def my_range():
    i = 0
    while i < 100:
        yield i
for i in range_():
    print(i)
```
上面的**my_range**就是一个生成器，调用**my_range()** 之后会返回一个Generator对象，这个对象就实现了迭代器协议，可以进行各种迭代器的操作。如果我们想要对实现了迭代器协议的对象进行切片操作的话，那么就需要使用**itertools.islice()**
```python
import itertools
for i in itertools.islice(g, 10, 20):
    print(i)
```
**itertools** 中还有用于过滤数据的**dropwhile** 方法
```python
a = (i for i in range(10))
for i in dropwhile(lambda x: x < 5, a):
    print(i)
#会打印出来5, 6, 7, 8, 9
#dropwhile会从前往后读数据，直到遇到第一个False，那么就会输出后续的所有元素
#值得注意的是这里只会处理第一个False
```
**itertools** 中还有有很多有用的处理迭代器对象的方法
```python
from itertools import permutations, combinations, zip_longest, chain
>>> a = [1,2,3]
>>> list(permutations(a))    #排列
[(1, 2, 3), (1, 3, 2), (2, 1, 3), (2, 3, 1), (3, 1, 2), (3, 2, 1)]
>>> list(combinations(a, 2))    #组合
[(1, 2), (1, 3), (2, 3)]
>>> zip_longest([1,2], [3,4,5]) #返回最长的zip结果，默认的zip是最短的
>>>chain([1,2,3], ['x', 'y', 'z']) #从逻辑上组合多个迭代器对象，然后进行统一的操作
```



