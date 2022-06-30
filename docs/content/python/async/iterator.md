## 可迭代对象

在 Python 一般将可用于 for 循环和其他许多需要序列的地方（zip()，map()，...）的对象称为可迭代对象，对象一般都实现了 `__iter__()` 或 `__getitem__()`方法来实现序列语义。python 常见的可迭代对象包含 `lsit, tuple, set, str, dict, 文件对象` 等。

为了寻找到可迭代对象的共通之处，可以通过以下方法来实现

``` python
def common_attrs(*objs):
    """ 计算对象共同属性 """
    assert len(objs) > 0
    attrs = set(dir(objs[0]))
    for obj in objs[1:]:
        attrs.intersection_update(set(dir(obj)))        # 取交集，获取到可迭代对象所有相同的属性
    attrs.difference_update(set(dir(object)))           # 取差集，去除这些属性中和父对象 object 相同的属性
    return attrs


if __name__ == "__main__":
    # 常见的 python 可迭代对象，包含 str, list, tuple, set, dict, file
    iterables = [
        "abc", ['a','b','c'], ('a', 'b'), {'a', 'b', 'c'}, {1: 'a'}, open('1.txt', 'r')
    ]
    iterable_common_atts = common_attrs(*iterables)
    print(iterable_common_atts)
    # {'__iter__'}

```

通过上述方法，我们最终得到了一个属性（接口） `__iter__`，python 提供了一个内置方法 `iter()` 来调用 `__iter__()`方法，除文件对象较为特殊外 `iter()` 方法返回了一个 `<type>_iterator` 的对象，这也是我们常说的 **迭代器**。

```python
for iterable in iterables:
    print(iter(iterable))

# <str_iterator object at 0x0000018E80039710>
# <list_iterator object at 0x0000018E80039710>
# <tuple_iterator object at 0x0000018E80039710>
# <set_iterator object at 0x0000018E800354C8>
# <dict_keyiterator object at 0x0000018EFFF05688>
# <_io.TextIOWrapper name='1.txt' mode='r' encoding='cp936'>
```

### 迭代器

为了了解迭代器的共通性，我们继续使用 `common_attrs()` 方法来获取迭代器的共同属性，这次我们得到了两个相同的属性（接口） `__iter__` 和 `__next__`

```python
iterators = [iter(iterable) for iterable in iterables]
iterator_common_atts = common_attrs(*iterators)
print(iterator_common_atts)

# {'__iter__', '__next__'}
```

#### 迭代器的使用

```python
iterable = ['a', 'b', 'c']	# 可迭代对象
iterator = iter(iterable)	# 转换迭代器
item = next(iterator)		# 通过 next() 方法调用 __next__ 接口
print(item)
# a
item = next(iterator)
print(item)
# b
item = next(iterator)
print(item)
# c
next(iterator)
# Traceback (most recent call last):
#   File ".\test2.py", line 10, in <module>
#     item = next(iterator)
# StopIteration
```

可以发现，当迭代器中所有元素都被获取完之后继续调用 `next()` 会得到一个异常 `StopIteration`（迭代终止） 。

针对这个特性，可以通过 `while`  循环来模仿 `for` 循环遍历迭代器

```python
# 创建迭代器
iterator = iter(iterable)
while True:
    try:
        # 获取迭代器下一个对象
        print(next(iterator))
    except StopIteration:	# 捕获结束迭代异常终止迭代
        break
```

#### 自定义迭代器

``` python
from typing import Iterable

class MyIterator:
    def __init__(self, items: Iterable):
        # 记录迭代元素
        self.items = items
        # 初始化起始位置
        self.index = 0
    
    def __next__(self):
        while self.index < len(self.items):
            # 先获取元素，否则最后会导致下标越界
            item = self.items[self.index]
            # 更新索引
            self.index += 1
            # 只返回奇数位的内容
            if self.index % 2 == 0:
                continue
            return item
        raise StopIteration

iterms = [0, 1, 2, 3, 4, 5, 6]
my_iterator = MyIterator(iterms)
while True:
    try:
        print(next(my_iterator))
    except StopIteration:
        break


for item in MyIterator(iterms):
    print(item)

# Traceback (most recent call last):
#   File ".\test3.py", line 27, in <module>
#     for item in MyIterator(iterms):
# TypeError: 'MyIterator' object is not iterable

```

我们构建了一个迭代器，并使用 while 循环获取到了奇数位的值，但是当使用 for 循环调用时，得到一个异常 `MyIterator` 不是一个可迭代的对象。

我们知道，可迭代对象是实现了 `__iter__` 接口的对象，为了 for 循环获取数据，难道我们需要在定义一个可迭代对象，然后通过 `__iter__` 接口返回迭代器？

```python
class MyItemIterable():
    def __init__(self, items):
        self.items = items
       
    def __iter__(self):
        return MyIterator(self.items)


for item in MyItemIterable(iterms):
    print(item)
```

但是这回导致代码过分冗余，`MyItemIterable` 的作用仅仅只是为了返回迭代器而存在的一个对象，既然 实现了 `__iter__` 接口的对象就是可迭代对象，而其作用是为了返回一个迭代器，那我们是否可以为迭代器 `MyIterator`  实现 `__iter__` 接口呢

```python
from typing import Iterable

class MyIterator:
    def __init__(self, items: Iterable):
        # 记录迭代元素
        self.items = items
        # 初始化起始位置
        self.index = 0
    
    def __next__(self):
        while self.index < len(self.items):
            item = self.items[self.index]
            # 更新索引
            self.index += 1
            # 只返回奇数位的内容
            if self.index % 2 == 0:
                continue
            return item
        raise StopIteration
    
    def __iter__(self):
        # __iter__ 接口的作用就是为了返回迭代器，而 self 就是一个迭代器，所以直接返回 self
        return self
    
for item in MyIterator(iterms):
    print(item)
```

这样，我们就实现了一个可迭代的迭代器

> 在 [Python 官方文档](https://docs.python.org/3/glossary.html#term-iterator)中明确指出了，迭代器必须同时实现 `__next__` 和 `__iter__` 两个接口，这称为  **迭代器协议**。
>
> 根据这个协议，迭代器必须是可迭代的，换言之，**迭代器** 是一种 **可迭代对象

#### 迭代器的意义

- 通过 `next()` 方法获取数据，来屏蔽底层不同的数据获取方式，简化编程
- 容器类数据结构只须关心数据的静态存储，由迭代器来记录数据迭代过程中的状态信息

可迭代对象

- 容器类
  - 列表，元组，字段，集合等
  - 只有 `__iter__` 接口
  - 存储静态数据
  - 需要额外的迭代器来迭代数据
  - 可多次迭代
- 迭代器类
  - 文件，`StringIO` 等
  - 需同时实现 `__iter__` 和 `__next__` 接口
  - 只能迭代一次
  - 动态

同过手动实现迭代器，将在 for 循环中由后台构建的方式，转换到了前台，并且让可迭代对象脱离了循环。也就是说迭代器和当前循环操作解耦了，于是下面的情况便成为了可能

- 一个可迭代对象可以构建出任意多个不同的迭代器
- 一种迭代器可以应用于任意多个可迭代对象（包括其他迭代器）

基于以上特性，就可以实现一个由迭代器构成的数据管道

[for] <- [iterator] <- [iterator] <- [iterator] <- ... <- [iterator]

- 由多个迭代器串联起来，形成一个处理数据的管道
- 在这个管道中，每一次只通过一份数据，避免了一次性加载所有数据
- 迭代器不在仅仅是按顺序返回数据那么简单，它开始承担处理数据的责任
- 通过迭代器存储数据时，远离了数据存储，可以开始不关心数据的存储方式

### 生成器？

``` python
class Counter:
    def __init__(self):
        self.index = 0

    def __iter__(self):
        return self
    
    def __next__(self):
        index = self.index
        self.index += 1
        return index

counter = Counter()
index = next(counter)
index = next(counter)
```

上面这个类实现了一个计数器，每次通过 next() 调用返回调用计数，连 `StopIteration` 也不再关心，因为迭代器将永远能获取到新的数据。这是一个名副其实的 **数据生成器**。

