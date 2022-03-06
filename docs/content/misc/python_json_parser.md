# Python JSON 解析器

### 1. JSON 格式的定义

`JSON` 全称 `JavaScript Object Notation`，是一种轻量级的数据交换格式。其优点是机器能很容易的解析和生成，人类也能轻松的书写和理解。

`JSON` 是基于 `JavaScript` 编程语言标准（ECMA-262 第三版）的一个子集，`JSON` 是一种完全独立于语言的文本格式，被多种编程语言所支持，包括 `C`、`C++`、`Java`、`JavaScript`、`Python`、`Golang` 等。

`JSON` 主要由两种结构构成：

1. 一个由名称/值的集合（键/值），在其他语言中也被称作对象（Object），记录（Record），结构体（Structure），字典（Dictionary），哈希表（Hash Table）等；
2. 一个有序的值的列表，在其他语言中也被称作数组（Array），向量/单行矩阵（Vector），列表（List）等；

这些通用的数据结构几乎所有的现代编程语言都以一种或多种形式支持它们，编程语言可交换的数据格式也基于这些格式。

在 `JSON` 格式中有以下格式：

由一组无序的键/值对组成的对象（以 JavaScript 中的 `object` 为例）。一个对象以左大括号 `{` 开头，以右大括号结尾 `}` 每个 `键` 后面跟着冒号 `:`，每个键/值对之间以逗号 `,` 分隔（除最后一组外）。

![Object JSON](https://www.json.org/img/object.png) 

一组有序的值组成的数组（以 JavaScript 中的 `Array` 为例）。一个数组以左中括号开始 `[`，右中括号 `]` 结束，值之间由逗号 `,` 分隔。

![Array JSON](https://www.json.org/img/array.png)

值可以是

1. 带双引号 `"` 的字符串
2. 数字
3. 对象（键/值对集合）
4. 数组（有序列表）
5. true / false
6. null（nil，None）

字符串是由0个或多个Unicode字符组成的有序序列，使用双引号 `"unicode"` 包围，特殊字符需使用反斜杠 `\` 转义，如 `"`，`/`，`\` 等。

![String](https://www.json.org/img/string.png)

数字类似于现代编程语言中的整型 `int`，JSON 格式中的数字只能是十进制数字

![Number](https://www.json.org/img/number.png)

除了键和字符串值之内的空格一般是无意义的

### 2. JSON 在 Python 语言中的使用

#### 2.1 JSON 数据类型与 Python 数据类型的比较

在 Python 中，JSON 格式一般由 `dict` 和 `list` 数据格式定义，其他格式也有其对应的数据格式

| JSON       | Python     |
| ---------- | ---------- |
| object     | dict       |
| array      | list       |
| string     | str        |
| number     | int，float |
| true/false | True/False |
| null       | None       |

#### 2.2 json 标准库

在 Python 中一般使用标准库 `json` ，来完成 `JSON` 格式和 `dict` 和 `list` 之间的转义。也有一些第三方开发者上传的库来完成 JSON 的解析，常用的如 ujson。

```
import json

# 符合 JSON 标准格式的 JSON 字符串
JSON_STR = """
{
    "key1": "value1",
    "key2": ["value2", "value3"],
    "key3": {
        "key4": "value4",
        "key5": ["value5", "value6"]
    },
    "key6": true,
    "key7": null
}
"""
# 解析为标准 dict 格式
json_dict = json.loads(JSON_STR)
# 压缩为 JSON 字符串
json_str = json.dumps(json_dict)
```

### 3. 实现 JSON 解析类

```
class ParseJson(object):
    def __init__(self, jsonstr):
        # 字符源串
        self._str = jsonstr.replace("'", '"')
        # 当前处理字符的位置
        self._pos = 0

    def _blank(self):
        # 跳过空格字符
        if self._pos < len(self._str) and self._str[self._pos] in [' ', '\n', '\r', '\t']:
            self._pos += 1
            self._blank()

    def _current_char(self):
        # 当前定位所在的字符
        return self._str[self._pos]

    def parse(self):
        # 入口函数
        self._blank()
        ch = self._current_char()
        if ch == '{':
            # 当前需要处理的字符为 {，则说明当前结构为 object，以 object 方式处理
            self._pos += 1
            return self._parse_object()
        elif ch == '[':
            # 当前需要处理的字符为 [，则说明当前结构为 array，以 array方式处理
            self._pos += 1
            return self._parse_array()
        else:
            print('JSON parse ERROR')

    def _parse_string(self):
        # 解析出由 "" 包围的 字符串 数据类型
        start = end = self._pos
        while self._str[end] != '"':
            if self._str[end] == '"':
                break
            end += 1
            if self._str[end] == '\\':
                end += 1
                if self._str[end] in '"\\/rntubf':
                    end += 1
        self._pos = end
        self._pos += 1
        return self._str[start: end]

    def _parse_number(self):
        start = end = self._pos
        # 解析出数字数据类型结束的坐标
        while self._str[end] not in ' \n\t\r,}]':   # 数字结束的字符串
            end += 1
        # 完整的数字字符串
        number = self._str[start:end]
        try:
            # 尝试进行转换
            if '.' in number or 'e' in number or 'E' in number:
                res = float(number)
            else:
                res = int(number)
            self._pos = end
        except ValueError as e:
            # 错误处理
            print(e.with_traceback())
        return res

    def _parse_value(self):
        # 核心处理逻辑
        c = self._current_char()
        if c == '{':
            # 当前值为 object
            self._pos += 1
            return self._parse_object()
        elif c == '[':
            # 当前值为 array
            self._pos += 1
            return self._parse_array()
        elif c == '"':
            # 当前值为 字符串
            self._pos += 1
            return self._parse_string()
        elif c == 't' and self._str[self._pos:self._pos + 4] == 'true':
            # 当前值为 true
            self._pos += 4
            self._blank()
            return True
        elif c == 'f' and self._str[self._pos:self._pos + 5] == 'false':
            # 当前值为 false
            self._pos += 5
            self._blank()
            return False
        elif c == 'n' and self._str[self._pos:self._pos + 4] == 'null':
            # 当前值为 null
            self._pos += 4
            self._blank()
            return None
        else:
            # 当前值为 数字，因为数字无特殊标识，所以最后处理
            self._blank()
            return self._parse_number()

    def _parse_array(self):
        # 解析 list
        array = []
        self._blank()
        # 如果当前字符为 ]，则说明为空列表 处理完成
        if self._current_char() == ']':
            self._pos += 1
            return array
        while True:
            item = self._parse_value()
            array.append(item)
            self._blank()
            if self._current_char() is ',':
                self._pos += 1
                self._blank()
            elif self._current_char() is ']':
                self._pos += 1
                return array
            else:
                print('array json parse error')
                return None

    def _parse_object(self):
        obj = {}
        self._blank()
        # 如果当前字符为 }，则说明为空字典 处理完成
        if self._current_char() == '}':
            self._pos += 1
            return obj
        self._pos += 1
        while True:
            # 得到 键
            key = self._parse_string()
            self._blank()
            # 跳过 ":" 字符，可多做一个判断 如果该字符不是 `:` 则说明不是正确的 json 格式
            self._pos += 1
            self._blank()
            # 解析值
            value = self._parse_value()
            obj[key] = value
            self._blank()
            if self._current_char() == ',':
                self._pos += 1
                self._blank()
                self._pos += 1
            if self._current_char() == '}':
                self._pos += 1
                break
        return obj


def parse(s):
    _p = ParseJson(s)
    return _p.parse()

if __name__ == "__main__":
    JSON_STR = """
    {
        "key1": "value1",
        "key2": ["value2", "value3"],
        "key3": {
            "key4": "value4",
            "key5": ["value5", "value6"]
        },
        "key6": true,
        "key7": null
    }
    """

    json_dict = parse(JSON_STR)
    assert isinstance(json_dict, dict)
    
    JSON_STR = """
    ["value2", "value3"]
    """
    json_dict = parse(JSON_STR)
    assert isinstance(json_dict, list)

```

 