## yield 关键字

### PEP 255
python 从 [PEP 255 - 简单生成器](https://peps.python.org/pep-0255/) 引入 yield 关键字，yield 被定义为语句（statement），并在 [PEP 342 - 通过增强型发生器进行协程](https://peps.python.org/pep-0342/) `yield` 被正式定义为[表达式](https://peps.python.org/pep-0342/#specification-summary)（expression）。该关键字只能在方法中使用，并将包含 `yield` 关键字的方法称为生成器函数（generator function），调用该方法不在直接执行，而是返回一个生成器迭代器对象（generator-iterator object），既然是一个迭代器，自然需要满足[迭代器协议](https://peps.python.org/pep-0234/)，通过以下方法可以验证：

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

只要在方法中使用了 `yield` 关键字，那么方法就会转换成生成器函数，就算 `yield` 表达式不会执行到，也会返回生成器迭代器对象（以下简称`生成器对象`）。
如果 `yield` 表达式无法执行到，当使用 `next(g)` 方法**激活**生成器对象时，会直接抛出 `StopIteration` 异常。当 `yield` 表达式能执行到，则 `next(g)` 会返回 `yield` 表达式后表达式的结果的值，可以通过 `value = next(g)` 来获取生成器对象返回的值，再次执行 `next(g)`，`yield` 表达式后代码就会继续执行，如果后续不会遇到 `yield` 表达式，那么在函数执行结束是将会抛出 `StopInteration` 异常。如果后续还有 `yield` 表达式则可以继续调用 `next()` 方法来执行迭代器对象。
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

### 简单使用
在 [迭代器](/content/python/async/iterator/) 一节中有一个实例，实现了一个数据生成器，通过 `yield` 表达式可以简单的重构该内容：
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

## 生成器背后原理
要了解生成器为何能在 `yiled` 处暂停代码，并在调用 `next` 后继续该处执行，就需要先了解 Python 的一些运行机制

### 1. 代码对象
在 Python 中，定义函数时会得到一个 `function` 对象（函数对象）。在对象中有一个 `__code__` 属性，其指向了一个实例，该实例类型为一个**代码对象**，代码对象中保存了函数的一些代码内容以及参数等信息，代码对象的一些重要属性均以 `co_` 开头。
```python
from types import FunctionType

def function():
    ...

assert type(function) is FunctionType
assert '__code__' in dir(function)
```
打印 `__code__` 对象的信息，可以得到以下内容（仅针对当前的 `function` 函数），`co_code` 为代码的二进制字节码，`co_flags` 为代码的对应的变量类型等等
```python
for f in dir(function.__code__):
    if f.startswith('co_'):
        print(f, getattr(function.__code__, f))
# co_argcount 0
# co_cellvars ()
# co_code b'd\x00S\x00'
# co_consts (None,)
# co_filename <stdin>
# co_firstlineno 1
# co_flags 67
# co_freevars ()
# co_kwonlyargcount 0
# co_lnotab b'\x00\x01'
# co_name function
# co_names ()
# co_nlocals 0
# co_posonlyargcount 0
# co_stacksize 1
# co_varnames ()
```

### 2. 执行函数
Python 中有一个重要概念 `栈帧`。Python 解释器维护一个栈帧的列表（函数运行栈），在解释器运行时会首先生成一个全局栈帧对象，该对象内存储所有的 方法等。
![image-20220328233043761](/img/Snipaste_2022-07-31_19-35-52.png)

当遇到方法调用时，会生成一个新的栈帧对象（函数运行帧），并将其压入栈帧列表，并在执行完成后，将其出栈（可以在[该网站](https://pythontutor.com/render.html#mode=display)中更直观的查看函数的调用栈）
![image-20220328233043761](/img/Snipaste_2022-07-31_19-40-15.png)
![image-20220328233043761](/img/Snipaste_2022-07-31_19-40-46.png)
![image-20220328233043761](/img/Snipaste_2022-07-31_19-41-03.png)
![image-20220328233043761](/img/Snipaste_2022-07-31_19-41-38.png)
![image-20220328233043761](/img/Snipaste_2022-07-31_19-42-18.png)

栈帧对象可以在方法运行时，通过模块 `inspect` 获取，栈帧对象主要属性以 `f_` 开头，其中 `f_code` 指向函数对象中的 `__code__`，`f_back` 为调用该方法的上一个栈帧，`f_builtins` 为 Python 内置的可用的全局变量的列表，`f_globals` 为当前模块的全局变量列表，`f_lasti` 为当前帧运行到的代码的二进制字节码（代码对象）位置，`f_locals` 存储局部变量信息等等。
```python
import inspect

def func():
    f = inspect.currentframe()
    return f

frame = func()
assert frame.f_code == func.__code__
for f in dir(frame):
    if f.startswith('f_'):
        print(f, getattr(frame, f))

# f_back <frame at 0x7f75da173cc0, file '<stdin>', line 1, code <module>>
# f_builtins {'__name__': 'builtins', '__doc__': "Built-in functions, exceptions, and other objects.\n\nNoteworthy: None is the `nil' object; Ellipsis represents `...' in slices.", '__package__': '', '__loader__': <class '_frozen_importlib.BuiltinImporter'>, '__spec__': ModuleSpec(name='builtins', loader=<class '_frozen_importlib.BuiltinImporter'>), '__build_class__': <built-in function __build_class__>, '__import__': <built-in function __import__>, 'abs': <built-in function abs>, 'all': <built-in function all>, 'any': <built-in function any>, 'ascii': <built-in function ascii>, 'bin': <built-in function bin>, 'breakpoint': <built-in function breakpoint>, 'callable': <built-in function callable>, 'chr': <built-in function chr>, 'compile': <built-in function compile>, 'delattr': <built-in function delattr>, 'dir': <built-in function dir>, 'divmod': <built-in function divmod>, 'eval': <built-in function eval>, 'exec': <built-in function exec>, 'format': <built-in function format>, 'getattr': <built-in function getattr>, 'globals': <built-in function globals>, 'hasattr': <built-in function hasattr>, 'hash': <built-in function hash>, 'hex': <built-in function hex>, 'id': <built-in function id>, 'input': <built-in function input>, 'isinstance': <built-in function isinstance>, 'issubclass': <built-in function issubclass>, 'iter': <built-in function iter>, 'len': <built-in function len>, 'locals': <built-in function locals>, 'max': <built-in function max>, 'min': <built-in function min>, 'next': <built-in function next>, 'oct': <built-in function oct>, 'ord': <built-in function ord>, 'pow': <built-in function pow>, 'print': <built-in function print>, 'repr': <built-in function repr>, 'round': <built-in function round>, 'setattr': <built-in function setattr>, 'sorted': <built-in function sorted>, 'sum': <built-in function sum>, 'vars': <built-in function vars>, 'None': None, 'Ellipsis': Ellipsis, 'NotImplemented': NotImplemented, 'False': False, 'True': True, 'bool': <class 'bool'>, 'memoryview': <class 'memoryview'>, 'bytearray': <class 'bytearray'>, 'bytes': <class 'bytes'>, 'classmethod': <class 'classmethod'>, 'complex': <class 'complex'>, 'dict': <class 'dict'>, 'enumerate': <class 'enumerate'>, 'filter': <class 'filter'>, 'float': <class 'float'>, 'frozenset': <class 'frozenset'>, 'property': <class 'property'>, 'int': <class 'int'>, 'list': <class 'list'>, 'map': <class 'map'>, 'object': <class 'object'>, 'range': <class 'range'>, 'reversed': <class 'reversed'>, 'set': <class 'set'>, 'slice': <class 'slice'>, 'staticmethod': <class 'staticmethod'>, 'str': <class 'str'>, 'super': <class 'super'>, 'tuple': <class 'tuple'>, 'type': <class 'type'>, 'zip': <class 'zip'>, '__debug__': True, 'BaseException': <class 'BaseException'>, 'Exception': <class 'Exception'>, 'TypeError': <class 'TypeError'>, 'StopAsyncIteration': <class 'StopAsyncIteration'>, 'StopIteration': <class 'StopIteration'>, 'GeneratorExit': <class 'GeneratorExit'>, 'SystemExit': <class 'SystemExit'>, 'KeyboardInterrupt': <class 'KeyboardInterrupt'>, 'ImportError': <class 'ImportError'>, 'ModuleNotFoundError': <class 'ModuleNotFoundError'>, 'OSError': <class 'OSError'>, 'EnvironmentError': <class 'OSError'>, 'IOError': <class 'OSError'>, 'EOFError': <class 'EOFError'>, 'RuntimeError': <class 'RuntimeError'>, 'RecursionError': <class 'RecursionError'>, 'NotImplementedError': <class 'NotImplementedError'>, 'NameError': <class 'NameError'>, 'UnboundLocalError': <class 'UnboundLocalError'>, 'AttributeError': <class 'AttributeError'>, 'SyntaxError': <class 'SyntaxError'>, 'IndentationError': <class 'IndentationError'>, 'TabError': <class 'TabError'>, 'LookupError': <class 'LookupError'>, 'IndexError': <class 'IndexError'>, 'KeyError': <class 'KeyError'>, 'ValueError': <class 'ValueError'>, 'UnicodeError': <class 'UnicodeError'>, 'UnicodeEncodeError': <class 'UnicodeEncodeError'>, 'UnicodeDecodeError': <class 'UnicodeDecodeError'>, 'UnicodeTranslateError': <class 'UnicodeTranslateError'>, 'AssertionError': <class 'AssertionError'>, 'ArithmeticError': <class 'ArithmeticError'>, 'FloatingPointError': <class 'FloatingPointError'>, 'OverflowError': <class 'OverflowError'>, 'ZeroDivisionError': <class 'ZeroDivisionError'>, 'SystemError': <class 'SystemError'>, 'ReferenceError': <class 'ReferenceError'>, 'MemoryError': <class 'MemoryError'>, 'BufferError': <class 'BufferError'>, 'Warning': <class 'Warning'>, 'UserWarning': <class 'UserWarning'>, 'DeprecationWarning': <class 'DeprecationWarning'>, 'PendingDeprecationWarning': <class 'PendingDeprecationWarning'>, 'SyntaxWarning': <class 'SyntaxWarning'>, 'RuntimeWarning': <class 'RuntimeWarning'>, 'FutureWarning': <class 'FutureWarning'>, 'ImportWarning': <class 'ImportWarning'>, 'UnicodeWarning': <class 'UnicodeWarning'>, 'BytesWarning': <class 'BytesWarning'>, 'ResourceWarning': <class 'ResourceWarning'>, 'ConnectionError': <class 'ConnectionError'>, 'BlockingIOError': <class 'BlockingIOError'>, 'BrokenPipeError': <class 'BrokenPipeError'>, 'ChildProcessError': <class 'ChildProcessError'>, 'ConnectionAbortedError': <class 'ConnectionAbortedError'>, 'ConnectionRefusedError': <class 'ConnectionRefusedError'>, 'ConnectionResetError': <class 'ConnectionResetError'>, 'FileExistsError': <class 'FileExistsError'>, 'FileNotFoundError': <class 'FileNotFoundError'>, 'IsADirectoryError': <class 'IsADirectoryError'>, 'NotADirectoryError': <class 'NotADirectoryError'>, 'InterruptedError': <class 'InterruptedError'>, 'PermissionError': <class 'PermissionError'>, 'ProcessLookupError': <class 'ProcessLookupError'>, 'TimeoutError': <class 'TimeoutError'>, 'open': <built-in function open>, 'quit': Use quit() or Ctrl-D (i.e. EOF) to exit, 'exit': Use exit() or Ctrl-D (i.e. EOF) to exit, 'copyright': Copyright (c) 2001-2021 Python Software Foundation.
# All Rights Reserved.

# Copyright (c) 2000 BeOpen.com.
# All Rights Reserved.

# Copyright (c) 1995-2001 Corporation for National Research Initiatives.
# All Rights Reserved.

# Copyright (c) 1991-1995 Stichting Mathematisch Centrum, Amsterdam.
# All Rights Reserved., 'credits':     Thanks to CWI, CNRI, BeOpen.com, Zope Corporation and a cast of thousands
#     for supporting Python development.  See www.python.org for more information., 'license': Type license() to see the full license text, 'help': Type help() for interactive help, or help(object) for help about object., '_': True}
# f_code <code object func at 0x7f75da0ce920, file "<stdin>", line 1>
# f_globals {'__name__': '__main__', '__doc__': None, '__package__': None, '__loader__': <class '_frozen_importlib.BuiltinImporter'>, '__spec__': None, '__annotations__': {}, '__builtins__': <module 'builtins' (built-in)>, 'function': <function function at 0x7f75dba21280>, 'FunctionType': <class 'function'>, 'f': 'f_globals', 'inspect': <module 'inspect' from '/usr/lib/python3.8/inspect.py'>, 'func': <function func at 0x7f75da13d8b0>, 'frame': <frame at 0x7f75da0a3d40, file '<stdin>', line 3, code func>}
# f_lasti 10
# f_lineno 3
# f_locals {'f': <frame at 0x7f75da0a3d40, file '<stdin>', line 3, code func>}
# f_trace None
# f_trace_lines True
# f_trace_opcodes False
```

### 3. 生成器的栈帧
Python 解释器在遇到生成器函数调用时，不是将该方法压入栈帧（判断只要来源于代码对象的 `co_flags` 属性），而是返回一个生成器对象，而在生成器对象中有一些重要的属性 `gi_frame`，`gi_code` 等。`gi_frame` 也是一个帧对象，也就是说生成器对象自己维护了一个帧对象，只有在调用生成器对象时，该帧对象才会被重新压入运行栈中，然后根据运行状态修改 `gi_frame` 中的信息。
