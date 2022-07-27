## yield 语句

python 从 [PEP 255 - 简单生成器](https://peps.python.org/pep-0255/) 引入 yield 关键字，yield 被定义为语句（statement）。该语句只能在方法中使用，并将包含 `yield` 语句的方法称为生成器函数（generator function），调用该方法不在直接执行，而是返回一个生成器迭代器对象（generator-iterator object），既然是一个迭代器，自然需要满足[迭代器协议](https://peps.python.org/pep-0234/)，通过以下方法可以验证：

```python
def fib():
    a, b = 0, 1
    while 1:
       yield b
       a, b = b, a+b

g = fib()
assert hasattr(g, '__next__')
assert hasattr(g, '__iter__')
```

既然生成器对象被称为迭代器自然可以通过 `for` 循环调用
```python
def fib():
    a, b = 0, 1
    while b < 100:
       yield b
       a, b = b, a+b

for i in fib():
    print(i)
```

只要在方法中使用了 `yield` 关键字，那么方法就会转换成生成器函数，就算 `yield` 语句不会执行到，也会返回生成器迭代器对象（以下简称`生成器对象`）。
如果 `yield` 语句无法执行到，当使用 `next(g)` 方法**激活**生成器对象时，会直接抛出 `StopIteration` 异常。当 `yield` 语句能执行到，则 `next(g)` 会返回 `yield` 语句后表达式的结果的值，可以通过 `value = next(g)` 来获取生成器对象返回的值，再次执行 `next(g)`，`yield` 语句后代码就会继续执行，如果后续不会遇到 `yield` 语句，那么在函数执行结束是将会抛出 `StopInteration` 异常。如果后续还有 `yield` 语句则可以继续调用 `next()` 方法来执行迭代器对象。
```python
from typing import Generator

def gen(meet_yield):
    print('start generator')
    if meet_yield:
        print('before yield')
        yield 666
        print('after yield')
    print('end generator')
    
g = gen(False)
assert isinstance(g, Generator)
next(g)
'''
start generator
end generator
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
StopIteration
'''

g = gen(True)
value = next(g)
print(value)
'''
start generator
before yield
666
'''
next(g)
'''
after yield
end generator
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
StopIteration
'''
```

> 注意：早期的生成器函数虽然能使用 `return` 语句，但是 `return` 语句上是不允许使用 `expression_list` 返回一个值的，只能单纯的返回来通知外部，生成器对象执行完成，或直接使用默认的返回 `None` 来达到通知的目的，

在 [迭代器](/content/python/async/iterator/) 章节有一个实例，实现了一个数据生成器，通过 `yield` 语句可以简单的重构该内容：
```python
def counter():
    index = 1
    while True:
      yield index
      index += 1
g_counter = counter()

index = next(g_counter)
index = next(g_counter)
```
