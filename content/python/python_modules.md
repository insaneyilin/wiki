---
title: "Python 中的模块"
date: 2019-11-09 11:01
tag: python
---

[TOC]

参考自 [廖雪峰Python教程](https://www.liaoxuefeng.com/wiki/1016959663602400)

## 使用模块

自定义一个模块：

```
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

'hello.py: a test module'
# 任何模块代码的第一个字符串都被视为模块的文档注释
# 文档注释可以用特殊变量 __doc__ 访问

__author__ = 'no one'  # 作者

import sys

def test():
    args = sys.argv
    if len(args)==1:
        print('Hello, world!')
    elif len(args)==2:
        print('Hello, %s!' % args[1])
    else:
        print('Too many arguments!')

if __name__=='__main__':
    test()
```

类似 `_xxx` 和 `__xxx` 这样的函数或变量就是非公开的（private），不应该被直接引用，比如 `_abc` ，`__abc` 等；Python并没有一种方法可以完全限制访问private函数或变量，这只是编程规范。

## 安装第三方模块

通过包管理工具 `pip` 完成。

一般来说，第三方库都会在Python官方的pypi.python.org网站注册。

```
pip install <package>
# python -m pip install <package>
```

或者使用 [Anaconda](https://www.anaconda.com/) 进行管理。

## 模块搜索路径

当加载一个模块时，Python会在指定路径下搜索对应的 `.py` 文件，如果找不到，就会报错。

默认情况下，Python解释器会搜索当前目录、所有已安装的内置模块和第三方模块，搜索路径存放在 `sys` 模块的 `path` 变量中：

```python
import sys
print(sys.path)
```

添加自己的搜索目录，有两种方法：

1. 直接修改 `sys.path`：`sys.path.append('/home/user/my_py_codes')`
2. 设置环境变量 `PYTHONPATH`: `export PYTHONPATH=...`

---

## Python 常用内建模块

Python 内置了很多非常有用的模块。

### datetime

```python
import datetime

# 获取当前日期和时间
print(datetime.datetime.now())
# 2019-11-09 10:41:06.959435
print(type(now))
# <class 'datetime.datetime'>

# 获取指定日期和时间
dt = datetime.datetime(2017, 4, 19, 10, 30) # 用指定日期时间创建 datetime
print(dt)  # 2017-04-19 10:30:00

# datetime转换为timestamp
# 我们把1970年1月1日 00:00:00 UTC+00:00时区的时刻称为epoch time，
# 记为0（1970年以前的时间timestamp为负数），当前时间就是相对于
# epoch time的秒数，称为timestamp。
dt = datetime.datetime(2019, 10, 19, 10, 30, 25) # 用指定日期时间创建 datetime
print(str(dt.timestamp()))  # 1571452225.0

# 时间戳转 datetime
t = 1571452225.0
print(datetime.datetime.fromtimestamp(t))

# str转换为datetime
# 注意转换后的datetime是没有时区信息的
dt = datetime.datetime.strptime('2015-6-1 18:19:59', '%Y-%m-%d %H:%M:%S')

# datetime 转 str
datetime.datetime.now().strftime('%Y-%m-%d %H:%M:%S')
```

datetime加减，借助于 `datetime.timedelta`：

```python
import datetime

now = datetime.datetime.now()
now + datetime.timedelta(hours=10)
now - datetime.timedelta(days=2)
```

时区转换：

```python
# 我们可以先通过utcnow()拿到当前的UTC时间，再转换为任意时区的时间

# 拿到UTC时间，并强制设置时区为UTC+0:00:
>>> utc_dt = datetime.utcnow().replace(tzinfo=timezone.utc)
>>> print(utc_dt)
2015-05-18 09:05:12.377316+00:00

# astimezone()将转换时区为北京时间:
>>> bj_dt = utc_dt.astimezone(timezone(timedelta(hours=8)))
>>> print(bj_dt)
2015-05-18 17:05:12.377316+08:00

# astimezone()将转换时区为东京时间:
>>> tokyo_dt = utc_dt.astimezone(timezone(timedelta(hours=9)))
>>> print(tokyo_dt)
2015-05-18 18:05:12.377316+09:00

# astimezone()将bj_dt转换时区为东京时间:
>>> tokyo_dt2 = bj_dt.astimezone(timezone(timedelta(hours=9)))
>>> print(tokyo_dt2)
2015-05-18 18:05:12.377316+09:00
```

### collections

`collections` 提供了很多有用的集合类（类似 C++ 中的容器）。

#### namedtuple

`namedtuple`，用来创建一个自定义的tuple对象，并且规定了tuple元素的个数，并可以用属性而不是索引来引用tuple的某个元素。

```python
import collections

Point = collections.namedtuple('Point', ['x', 'y'])
pt = Point(1, 2)
print(pt.x, pt.y)

# namedtuple 是 tuple 的子类
isinstance(pt, tuple)  # True
```

#### deque

`deque`，使用 `list` 存储数据时，按索引访问元素很快，但是插入和删除元素就很慢了，因为 `list` 是线性存储，数据量大的时候，插入和删除效率很低。 `deque` 是为了高效实现插入和删除操作的双向列表，适合用于队列和栈：

```python
import collections

q = collections.deque(['a', 'b', 'c'])
q.append('x')
q.pop()
q.appendleft('x')
q.popleft()
```

#### defaultdict

`defaultdict`，使用 `dict` 时，如果引用的Key不存在，就会抛出KeyError。如果希望 `key` 不存在时，返回一个默认值，就可以用 `defaultdict`:

```python
import collections

d = collections.defaultdict(lambda: 'N/A')
print(d['new_key'])  # 'N/A'
```

#### OrderedDict

`OrderedDict`, 使用 `dict` 时，Key是无序的。在对 `dict` 做迭代时，我们无法确定Key的顺序。如果要保持Key的顺序，可以用 `OrderedDict`：

```python
import collections

d = OrderedDict()
od['z'] = 1
od['y'] = 2
od['x'] = 3
list(od.keys()) # 按照插入的Key的顺序返回
# ['z', 'y', 'x']
```

#### ChainMap

`ChainMap` 可以把一组 `dict` 串起来并组成一个逻辑上的 `dict` 。 `ChainMap` 本身也是一个 `dict` ，但是查找的时候，会按照顺序在内部的 `dict` 依次查找。

什么时候使用 `ChainMap` 最合适？举个例子：应用程序往往都需要传入参数，参数可以通过命令行传入，可以通过环境变量传入，还可以有默认参数。我们可以用ChainMap实现参数的优先级查找，即先查命令行参数，如果没有传入，再查环境变量，如果没有，就使用默认参数。

```python
import collections

d1 = {'a': 1, 'b': 2}
d2 = {'a': 0, 'b': 3, 'c': 9}

cmap = collections.ChainMap(d1, d2)
print(cmap)
# ChainMap({'a': 1, 'b': 2}, {'a': 0, 'b': 3, 'c': 9})
print(cmap['a'])  # 1
print(cmap['c'])  # 9
```

#### Counter

一个简单的计数器，例如，统计字符出现的个数：

```python
import collections

counter = collections.Counter()
# Counter 实际上是 dict 的一个子类
isinstance(counter, dict)  # True

for ch in 'aabbbcccc':
    counter[ch] += 1
print(counter)
# Counter({'c': 4, 'b': 3, 'a': 2})
```

### base64

Base64是一种用64个字符来表示任意二进制数据的方法。

Base64的原理很简单，首先，准备一个包含64个字符的数组：

```python
['A', 'B', 'C', ... 'a', 'b', 'c', ... '0', '1', ... '+', '/']
```

然后，对二进制数据进行处理，每3个字节一组，一共是 `3x8=24bit` ，划为4组，每组正好6个bit。

```
# b1        b2        b3
  1001 0001 0000 0100 1110 1011
# n1     n2      n3     n4
```

这样我们得到4个数字作为索引，然后查表，获得相应的4个字符，就是编码后的字符串。

所以，Base64编码会把3字节的二进制数据编码为4字节的文本数据，长度增加33%，好处是编码后的文本数据可以在邮件正文、网页等直接显示。

如果要编码的二进制数据不是3的倍数，最后会剩下1个或2个字节怎么办？Base64用 `\x00` 字节在末尾补足后，再在编码的末尾加上1个或2个 `=` 号，表示补了多少字节，解码的时候，会自动去掉。

```python
import base64

encoded = base64.b64encode(b'hello')
print(encoded)  # b'aGVsbG8='
print(type(encoded))  # <class 'bytes'>

decoded = base64.b64decode(encoded)
print(decoded)  # b'hello'
print(type(decoded))  # <class 'bytes'>
print(str(decoded, 'utf-8'))  # hello
```

由于标准的Base64编码后可能出现字符 `+` 和 `/` ，在URL中就不能直接作为参数，所以又有一种"url safe"的base64编码，其实就是把字符 `+` 和 `/` 分别变成 `-`和 `_`:

```python
import base64

base64.b64encode(b'i\xb7\x1d\xfb\xef\xff')
# b'abcd++//'
base64.urlsafe_b64encode(b'i\xb7\x1d\xfb\xef\xff')
# b'abcd--__'
base64.urlsafe_b64decode('abcd--__')
# b'i\xb7\x1d\xfb\xef\xff'
```

还可以自己定义64个字符的排列顺序，这样就可以自定义Base64编码

Base64是一种通过查表的编码方法， **不能用于加密，即使使用自定义的编码表也不行** 。（破解难度太低）

Base64适用于小段内容的编码，比如数字证书签名、Cookie的内容等。

由于 `=` 字符也可能出现在Base64编码中，但 `=` 用在URL、Cookie里面会造成歧义，所以，很多Base64编码后会把 `=` 去掉。去掉 `=` 后怎么解码呢？因为Base64是把3个字节变为4个字节，所以，Base64编码的长度永远是4的倍数，因此，需要加上 `=` 把Base64字符串的长度变为4的倍数，就可以正常解码了。

### struct

Python提供了一个struct模块来解决bytes和其他二进制数据类型的转换。

```python
import struct

b = struct.pack('>I', 10240099)
# b'\x00\x9c@c'

x = struct.unpack('>I', b)
print(x)
# (10240099,)
```

`pack` 的第一个参数是处理指令，`'>I'` 的意思是：

- `>`: 字节顺序是big-endian，也就是网络序
- `I`: 表示4字节无符号整数

`unpack` 把bytes变成相应的数据类型。

### hashlib

Python的 `hashlib` 提供了常见的 Hash 算法（又称摘要算法），如MD5，SHA1等等。

Hash：通过一个函数，把 **任意长度的数据** 转换为一个 **长度固定的数据串** （通常用16进制的字符串表示）。

Hash 可以用来检查原始数据是否被人篡改过。

Hash 函数是一个单向函数，计算出 hash 值容易，反推很难。

```python
import hashlib

md5 = hashlib.md5()
md5.update('hello world'.encode('utf-8'))
print(md5.hexdigest())
# 5eb63bbbe01eeed093cb22bb8f5acdc3
```

如果数据量很大，可以分块多次调用 `update()` ，最后计算的结果是一样的。

MD5是最常见的摘要算法，速度很快，生成结果是固定的128 bit字节，通常用一个32位的16进制字符串表示。

另一种常见的摘要算法是SHA1，`hashlib.sha1()`。

SHA1的结果是160 bit字节，通常用一个40位的16进制字符串表示。

比SHA1更安全的算法是SHA256和SHA512，不过越安全的算法不仅越慢，而且摘要长度更长。

有没有可能两个不同的数据通过某个摘要算法得到了相同的摘要？完全有可能，因为任何摘要算法都是把无限多的数据集合映射到一个有限的集合中。这种情况称为碰撞。

一个摘要算法的例子，网站数据库不应该明文保存用户密码，而应该存储用户密码的哈希值，当用户登录时，首先计算用户输入的明文口令的MD5，然后和数据库存储的MD5对比，如果一致，说明口令输入正确，如果不一致，口令肯定错误。

假设你是一个黑客，已经拿到了存储MD5口令的数据库，如何通过MD5反推用户的明文口令呢？暴力破解费事费力，真正的黑客不会这么干。

很多用户喜欢用123456，888888，password这些简单的口令，于是，黑客可以事先计算出这些常用口令的MD5值，得到一个反推表：

```
'e10adc3949ba59abbe56e057f20f883e': '123456'
'21218cca77804d2ba1922c33e0151105': '888888'
'5f4dcc3b5aa765d61d8327deb882cf99': 'password'
```

这样，无需破解，只需要对比数据库的MD5，黑客就获得了使用常用口令的用户账号。

由于常用口令的MD5值很容易被计算出来，所以，要确保存储的用户口令不是那些已经被计算出来的常用口令的MD5，这一方法通过对原始口令加一个复杂字符串来实现，俗称“加盐”：

```python
def calc_md5(password):
    return get_md5(password + 'the-Salt')
```

但是如果有两个用户都使用了相同的简单口令比如123456，在数据库中，将存储两条相同的MD5值，这说明这两个用户的口令是一样的。有没有办法让使用相同口令的用户存储不同的MD5呢？

如果假定用户无法修改登录名，就可以通过把登录名作为Salt的一部分来计算MD5，从而实现相同口令的用户也存储不同的MD5。

### hmac

为了防止黑客通过彩虹表（rainbow table）根据哈希值反推原始口令，在计算哈希的时候，不能仅针对原始输入计算，需要增加一个salt来使得相同的输入也能得到不同的哈希，这样，大大增加了黑客破解的难度。

Hmac算法：Keyed-Hashing for Message Authentication。它通过一个标准算法，在计算哈希的过程中，把key混入计算过程中。要验证哈希值，必须同时提供正确的口令(key)。

和我们自定义的加salt算法不同，Hmac算法针对所有哈希算法都通用，无论是MD5还是SHA-1。采用Hmac替代我们自己的salt算法，可以使程序算法更标准化，也更安全。

```python
import hmac
message = b'Hello, world!'
key = b'secret'
h = hmac.new(key, message, digestmod='MD5')
# 如果消息很长，可以多次调用h.update(msg)
h.hexdigest()
# 'fa4ee7d173f2d97ee79022d1a7355bcf'
```

### itertools

`itertools` 提供了非常有用的用于操作迭代对象的函数。

```python
import itertools

# 无限的迭代
natuals = itertools.count(1)
for n in natuals:
    print(n)

# cycle()会把传入的一个序列无限重复下去：
cs = itertools.cycle('ABC')  # 注意字符串也是序列的一种
for c in cs:
    print(c)

# repeat()负责把一个元素无限重复下去，不过如果提供第二个参数就可以限定重复次数：
ns = itertools.repeat('A', 3)
for n in ns:
    print(n)

# 通常我们会通过takewhile()等函数根据条件判断来截取出一个有限的序列：
natuals = itertools.count(1)
ns = itertools.takewhile(lambda x: x <= 10, natuals)
print(list(ns))
# [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

# chain()可以把一组迭代对象串联起来，形成一个更大的迭代器：
for c in itertools.chain('ABC', 'XYZ'):
    print(c)
# 迭代效果：'A' 'B' 'C' 'X' 'Y' 'Z'

# groupby()把迭代器中相邻的重复元素挑出来放在一起：
for key, group in itertools.groupby('AAABBBCCAAA'):
    print(key, list(group))
"""
A ['A', 'A', 'A']
B ['B', 'B', 'B']
C ['C', 'C']
A ['A', 'A', 'A']
"""

# groupby 支持传入一个求 key 值的函数，比如我们可以忽略大小写：
for key, group in itertools.groupby('AaaBBbcCAAa', lambda c: c.upper()):
    print(key, list(group))
"""
A ['A', 'a', 'a']
B ['B', 'B', 'b']
C ['c', 'C']
A ['A', 'A', 'a']
"""
```

### contextlib

在Python中，读写文件这样的资源要特别注意，必须在使用完毕后正确关闭它们。正确关闭文件资源的一个方法是使用 `try...finally` ：

```python
try:
    f = open('/path/to/file', 'r')
    f.read()
finally:
    if f:
        f.close()
```

Python的 `with` 语句允许我们非常方便地使用资源，而不必担心资源没有关闭，所以上面的代码可以简化为：

```python`
with open('/path/to/file', 'r') as f:
    f.read()
```

任何对象，只要正确实现了 **上下文管理** ，就可以用于 `with` 语句。

实现上下文管理是通过 `__enter__` 和 `__exit__` 这两个方法实现的。例如，下面的class实现了这两个方法：

```python
class Query(object):

    def __init__(self, name):
        self.name = name

    def __enter__(self):
        print('Begin')
        return self
    
    def __exit__(self, exc_type, exc_value, traceback):
        if exc_type:
            print('Error')
        else:
            print('End')
    
    def query(self):
        print('Query info about %s...' % self.name)


# 把自己写的资源对象用于with语句：
with Query('Alice') as q:
    q.query()
```

#### @contextmanager

编写 `__enter__` 和 `__exit__` 仍然很繁琐，因此Python的标准库 `contextlib` 提供了更简单的写法，上面的代码可以改写如下：

```python
from contextlib import contextmanager

class Query(object):

    def __init__(self, name):
        self.name = name

    def query(self):
        print('Query info about %s...' % self.name)

# @contextmanager这个decorator接受一个generator，
# 用yield语句把with ... as var把变量输出出去，
# 然后，with语句就可以正常地工作了：
@contextmanager
def create_query(name):
    print('Begin')
    q = Query(name)
    yield q
    print('End')

with create_query('Bob') as q:
    q.query()
```

很多时候，我们希望在某段代码执行前后自动执行特定代码，也可以用@contextmanager实现。例如：

```python
@contextmanager
def tag(name):
    print("<%s>" % name)
    yield
    print("</%s>" % name)

with tag("h1"):
    print("hello")
    print("world")
"""
<h1>
hello
world
</h1>
"""
```

#### @closing

如果一个对象没有实现上下文，我们就不能把它用于 `with` 语句。这个时候，可以用 `closing()` 来把该对象变为上下文对象。例如，用 `with` 语句使用 `urlopen()` ：

```python
from contextlib import closing
from urllib.request import urlopen

with closing(urlopen('https://www.python.org')) as page:
    for line in page:
        print(line)
```

`closing` 也是一个经过@contextmanager装饰的generator，这个generator编写起来其实非常简单：

```python
@contextmanager
def closing(thing):
    try:
        yield thing
    finally:
        thing.close()
```

它的作用就是把任意对象变为上下文对象，并支持 `with` 语句。

### urllib

urllib提供了一系列用于操作URL的功能。

#### Get

urllib的 `request` 模块可以非常方便地抓取URL内容，也就是发送一个GET请求到指定的页面，然后返回HTTP的响应：

```python
from urllib import request

with request.urlopen('https://api.douban.com/v2/book/2129650') as f:
    data = f.read()
    print('Status:', f.status, f.reason)
    for k, v in f.getheaders():
        print('%s: %s' % (k, v))
    print('Data:', data.decode('utf-8'))
```

如果我们要想模拟浏览器发送GET请求，就需要使用`Request`对象，通过往`Request`对象添加HTTP头，我们就可以把请求伪装成浏览器。例如，模拟iPhone 6去请求豆瓣首页：

```python
from urllib import request

req = request.Request('http://www.douban.com/')
# 添加HTTP头
req.add_header('User-Agent', 'Mozilla/6.0 (iPhone; CPU iPhone OS 8_0 like Mac OS X) AppleWebKit/536.26 (KHTML, like Gecko) Version/8.0 Mobile/10A5376e Safari/8536.25')
with request.urlopen(req) as f:
    print('Status:', f.status, f.reason)
    for k, v in f.getheaders():
        print('%s: %s' % (k, v))
    print('Data:', f.read().decode('utf-8'))
```

#### Post

如果要以POST发送一个请求，只需要把 `urlopen` 的参数 `data` 以bytes形式传入。

```python
from urllib import request, parse

print('Login to weibo.cn...')
email = input('Email: ')
passwd = input('Password: ')
login_data = parse.urlencode([
    ('username', email),
    ('password', passwd),
    ('entry', 'mweibo'),
    ('client_id', ''),
    ('savestate', '1'),
    ('ec', ''),
    ('pagerefer', 'https://passport.weibo.cn/signin/welcome?entry=mweibo&r=http%3A%2F%2Fm.weibo.cn%2F')
])

req = request.Request('https://passport.weibo.cn/sso/login')
req.add_header('Origin', 'https://passport.weibo.cn')
req.add_header('User-Agent', 'Mozilla/6.0 (iPhone; CPU iPhone OS 8_0 like Mac OS X) AppleWebKit/536.26 (KHTML, like Gecko) Version/8.0 Mobile/10A5376e Safari/8536.25')
req.add_header('Referer', 'https://passport.weibo.cn/signin/login?entry=mweibo&res=wel&wm=3349&r=http%3A%2F%2Fm.weibo.cn%2F')

# 传入 data 参数
with request.urlopen(req, data=login_data.encode('utf-8')) as f:
    print('Status:', f.status, f.reason)
    for k, v in f.getheaders():
        print('%s: %s' % (k, v))
    print('Data:', f.read().decode('utf-8'))
```

#### Handler

如果还需要更复杂的控制，比如通过一个Proxy去访问网站，我们需要利用ProxyHandler来处理，示例代码如下：

```python
proxy_handler = urllib.request.ProxyHandler({'http': 'http://www.example.com:3128/'})
proxy_auth_handler = urllib.request.ProxyBasicAuthHandler()
proxy_auth_handler.add_password('realm', 'host', 'username', 'password')
opener = urllib.request.build_opener(proxy_handler, proxy_auth_handler)
with opener.open('http://www.example.com/login.html') as f:
    pass
```

### xml

XML比JSON复杂，在Web中应用也不如以前多，不过仍有很多地方在用。

操作XML有两种方法：DOM和SAX。DOM会把整个XML读入内存，解析为树，因此占用内存大，解析慢，优点是可以任意遍历树的节点。SAX是流模式，边读边解析，占用内存小，解析快，缺点是我们需要自己处理事件。

正常情况下，优先考虑SAX，因为DOM实在太占内存。

在Python中使用SAX解析XML非常简洁，通常我们关心的事件是 `start_element`，`end_element` 和 `char_data` ，准备好这3个函数，然后就可以解析xml了。

举个例子，当SAX解析器读到一个节点时：

```
<a href="/">python</a>
```

会产生3个事件：

1. start_element事件，在读取<a href="/">时；
2. char_data事件，在读取python时；
3. end_element事件，在读取</a>时。

```python
from xml.parsers.expat import ParserCreate

class DefaultSaxHandler(object):
    def start_element(self, name, attrs):
        print('sax:start_element: %s, attrs: %s' % (name, str(attrs)))

    def end_element(self, name):
        print('sax:end_element: %s' % name)

    def char_data(self, text):
        print('sax:char_data: %s' % text)

xml = r'''<?xml version="1.0"?>
<ol>
    <li><a href="/python">Python</a></li>
    <li><a href="/ruby">Ruby</a></li>
</ol>
'''

handler = DefaultSaxHandler()
parser = ParserCreate()
parser.StartElementHandler = handler.start_element
parser.EndElementHandler = handler.end_element
parser.CharacterDataHandler = handler.char_data
parser.Parse(xml)
```

解析XML时，找出自己感兴趣的节点，响应事件时，把节点数据保存起来。解析完毕后，就可以处理数据。

### HTMLParser

Python提供了HTMLParser来非常方便地解析HTML，只需简单几行代码：

```python
from html.parser import HTMLParser
from html.entities import name2codepoint

class MyHTMLParser(HTMLParser):

    def handle_starttag(self, tag, attrs):
        print('<%s>' % tag)

    def handle_endtag(self, tag):
        print('</%s>' % tag)

    def handle_startendtag(self, tag, attrs):
        print('<%s/>' % tag)

    def handle_data(self, data):
        print(data)

    def handle_comment(self, data):
        print('<!--', data, '-->')

    def handle_entityref(self, name):
        print('&%s;' % name)

    def handle_charref(self, name):
        print('&#%s;' % name)

parser = MyHTMLParser()
parser.feed('''<html>
<head></head>
<body>
<!-- test html parser -->
    <p>Some <a href=\"#\">html</a> HTML&nbsp;tutorial...<br>END</p>
</body></html>''')
```

利用HTMLParser，可以把网页中的文本、图像等解析出来。

---

## 常用第三方模块

### Pillow

PIL：Python Imaging Library，已经是Python平台事实上的图像处理标准库了。PIL功能非常强大，但API却非常简单易用。

由于PIL仅支持到Python 2.7，加上年久失修，于是一群志愿者在PIL的基础上创建了兼容的版本，名字叫Pillow，支持最新Python 3.x，又加入了许多新特性，因此，我们可以直接安装使用Pillow。

安装：

```
pip install pillow
```

常用操作：

```python
from PIL import Image

# 打开一个jpg图像文件，注意是当前路径:
im = Image.open('test.jpg')

# 获得图像尺寸:
w, h = im.size
print('Original image size: %sx%s' % (w, h))

# 缩放到50%:
im.thumbnail((w//2, h//2))
print('Resize image to: %sx%s' % (w//2, h//2))

# 把缩放后的图像用jpeg格式保存:
im.save('thumbnail.jpg', 'jpeg')

# PIL 中提供了一些图像 filter
from PIL import ImageFilter

# 应用模糊滤镜:
im2 = im.filter(ImageFilter.BLUR)
```

PIL的 `ImageDraw` 提供了一系列绘图方法，让我们可以直接绘图。比如要生成字母验证码图片：

```python
from PIL import Image, ImageDraw, ImageFont, ImageFilter

import random

# 随机字母:
def rndChar():
    return chr(random.randint(65, 90))

# 随机颜色1:
def rndColor():
    return (random.randint(64, 255), random.randint(64, 255), random.randint(64, 255))

# 随机颜色2:
def rndColor2():
    return (random.randint(32, 127), random.randint(32, 127), random.randint(32, 127))

# 240 x 60:
width = 60 * 4
height = 60
image = Image.new('RGB', (width, height), (255, 255, 255))
# 创建Font对象:
font = ImageFont.truetype('Arial.ttf', 36)
# 创建Draw对象:
draw = ImageDraw.Draw(image)
# 填充每个像素:
for x in range(width):
    for y in range(height):
        draw.point((x, y), fill=rndColor())
# 输出文字:
for t in range(4):
    draw.text((60 * t + 10, 10), rndChar(), font=font, fill=rndColor2())
# 模糊:
image = image.filter(ImageFilter.BLUR)
image.save('code.jpg', 'jpeg')
```

### requests

`requests` 是一个Python第三方库，处理URL资源比内置的 `urllib` 强大。

```
pip install requests
```

要通过GET访问一个页面，只需要几行代码：

```python
import requests
r = requests.get('https://www.douban.com/') # 豆瓣首页
print(r.status_code)
# 200
print(r.text)
"""
'<!DOCTYPE HTML>\\n<html>\\n<head>\\n<meta name="description" content="提供图书、电影、音乐唱片的推荐、评论和...'
"""
print(r.encoding)
# 'utf-8'
```

### chardet

在处理一些不规范的第三方网页的时候，如果不知道字符串编码，会很难处理。

`chardet` 用于检测编码，简单实用。

```python
import chardet

chardet.detect(b'Hello, world!')
# {'encoding': 'ascii', 'confidence': 1.0, 'language': ''}
```

### psutil

在Linux下，有许多系统命令可以让我们时刻监控系统运行的状态，如ps，top，free等等。要获取这些系统信息，Python可以通过 `subprocess` 模块调用并获取结果。但这样做显得很麻烦，尤其是要写很多解析代码。

psutil = process and system utilities，这个第三方模块可以通过一两行代码实现系统监控，还可以跨平台使用。

```python
import psutil

# 获取 CPU 信息
psutil.cpu_count() # CPU逻辑数量
psutil.cpu_count(logical=False) # CPU物理核心

# 统计CPU的用户／系统／空闲时间：
psutil.cpu_times()

# 实现类似top命令的CPU使用率，每秒刷新一次，累计10次：
for x in range(10):
    psutil.cpu_percent(interval=1, percpu=True)

# 获取内存信息
psutil.virtual_memory() # 物理内存
psutil.swap_memory()  # 交换内存

# 获取磁盘信息
psutil.disk_partitions() # 磁盘分区信息
psutil.disk_usage('/') # 磁盘使用情况
psutil.disk_io_counters() # 磁盘IO

# 获取网络信息
psutil.net_io_counters() # 获取网络读写字节／包的个数
psutil.net_if_stats() # 获取网络接口状态

# 获取进程信息
psutil.pids() # 所有进程ID
p = psutil.Process(3776) # 获取指定进程ID=3776
```

psutil还提供了一个 `test()` 函数，可以模拟出 `ps` 命令的效果。

```python
import psutil
psutil.test()
```

