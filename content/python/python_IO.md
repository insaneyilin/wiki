---
title: "Python IO 编程"
date: 2019-11-12 18:01
tag: python
---

IO在计算机中指Input/Output，也就是输入和输出。由于程序和运行时数据是在内存中驻留，由CPU这个超快的计算核心来执行，涉及到数据交换的地方，通常是磁盘、网络等，就需要IO接口。

IO编程中，Stream（流）是一个很重要的概念，可以把流想象成一个水管，数据就是水管里的水，但是只能单向流动。Input Stream就是数据从外面（磁盘、网络）流进内存，Output Stream就是数据从内存流到外面去。对于浏览网页来说，浏览器和新浪服务器之间至少需要建立两根水管，才可以既能发数据，又能收数据。

由于CPU和内存的速度远远高于外设的速度，所以，在IO编程中，就存在速度严重不匹配的问题。举个例子来说，比如要把100M的数据写入磁盘，CPU输出100M的数据只需要0.01秒，可是磁盘要接收这100M数据可能需要10秒，怎么办呢？有两种办法：

- 第一种是CPU等着，也就是程序暂停执行后续代码，等100M的数据在10秒后写入磁盘，再接着往下执行，这种模式称为同步IO；
- 另一种方法是CPU不等待，只是告诉磁盘，“您老慢慢写，不着急，我接着干别的事去了”，于是，后续代码可以立刻接着执行，这种模式称为异步IO。

**同步和异步的区别就在于是否等待IO执行的结果** 。好比你去麦当劳点餐，你说“来个汉堡”，服务员告诉你，对不起，汉堡要现做，需要等5分钟，于是你站在收银台前面等了5分钟，拿到汉堡再去逛商场，这是同步IO。

使用异步IO来编写程序性能会远远高于同步IO，但是异步IO的缺点是编程模型复杂(回调/轮询/...)。

参考 [廖雪峰Python教程](https://www.liaoxuefeng.com/wiki/1016959663602400/1017606916795776)。

[TOC]

---

## 文件读写

```python
f = open('111.txt', 'r')

f.read()  # 调用read()方法可以一次读取文件的全部内容，Python把内容读到内存，用一个str对象表示

f.close()  # 关闭文件，释放内存

# 推荐使用 with 语句
with open('/path/to/file', 'r') as f:
    print(f.read())

```

### file-like object

像open()函数返回的这种有个read()方法的对象，在Python中统称为file-like Object。除了file外，还可以是内存的字节流，网络流，自定义流等等。file-like Object不要求从特定类继承，只要写个read()方法就行。

`StringIO` 就是在内存中创建的file-like Object，常用作临时缓冲。

### 读二进制文件

```python
f = open('/Users/michael/test.jpg', 'rb')  # rb 表示读取二进制文件
f.read()
# b'\xff\xd8\xff\xe1\x00\x18Exif\x00\x00...' # 十六进制表示的字节
```

### 写文件

```python
f = open('/Users/michael/test.txt', 'w')
f.write('Hello, world!')
f.close()

with open('/Users/michael/test.txt', 'w') as f:
    f.write('Hello, world!')
```

---

## StringIO

`StringIO` 顾名思义就是在内存中读写 `str`。

```python
from io import StringIO
f = StringIO()
f.write('hello')
# 5
f.write(' ')
# 1
f.write('world!')
# 6
print(f.getvalue())  # 获取写入后的字符串
# hello world!
```

可以像读文件一样读取：

```python
from io import StringIO
f = StringIO('Hello!\nHi!\nGoodbye!')
while True:
    s = f.readline()
    if s == '':
        break
    print(s.strip())

# Hello!
# Hi!
# Goodbye!
```

## BytesIO

如果要操作二进制数据，需要使用BytesIO。

BytesIO实现了在内存中读写bytes，我们创建一个BytesIO，然后写入一些bytes：

```python
from io import BytesIO
f = BytesIO()
f.write('中文'.encode('utf-8'))
# 6
print(f.getvalue())
# b'\xe4\xb8\xad\xe6\x96\x87'
```

和StringIO类似，可以用一个bytes初始化BytesIO，然后，像读文件一样读取：

```python
from io import BytesIO
f = BytesIO(b'\xe4\xb8\xad\xe6\x96\x87')
f.read()
# b'\xe4\xb8\xad\xe6\x96\x87'
```

## 操作文件和目录

`os` 模块提供了很多系统相关的功能

```python
import os

os.name  # 操作系统类型

os.environ  # 操作系统环境变量

os.environ.get('PATH')
```

文件、目录相关操作：

```python
os.path.abspath('.')  # 绝对路径

os.path.join('/Users/michael', 'testdir')  # 拼接路径

os.mkdir('/Users/michael/testdir')  # 创建目录

os.rmdir('/Users/michael/testdir')  # 删除目录

os.path.split('/Users/michael/testdir/file.txt')  # 拆分目录和文件名
# ('/Users/michael/testdir', 'file.txt')

os.path.splitext('/path/to/file.txt')  # 拆分扩展名
# ('/path/to/file', '.txt')

os.rename('test.txt', 'test.py')  # 重命名文件

# os 中没有拷贝文件的函数，可以用 shutil.copyfile()
import shutil
shutil.copyfile('test.txt', 'test.py')

# 列出当前目录下所有 .jpg 文件
[x for x in os.listdir('.') if os.path.isfile(x) and\
        os.path.splitext(x)[1] == ".jpg"]
```

## 序列化

我们把变量从内存中变成可存储或传输的过程称之为序列化，在Python中叫 **pickling** ，在其他语言中也被称之为 **serialization** ，marshalling，flattening等等，都是一个意思。

序列化之后，就可以把序列化后的内容写入磁盘，或者通过网络传输到别的机器上。

反过来，把变量内容从序列化的对象重新读到内存里称之为反序列化，即 **unpickling** 。

Python提供了 `pickle` 模块来实现序列化。

```python
import pickle
d = dict(name='Bob', age=20, score=88)
d_bytes = pickle.dumps(d)

# pickle.dumps()方法把任意对象序列化成一个bytes，
# 然后就可以把这个bytes写入文件。
# 或者用另一个方法pickle.dump()直接把对象序列化后写入一个file-like Object：
f = open('dump.txt', 'wb')
pickle.dump(d, f)
f.close()

# 用 pickle.load() 读取
f = open('dump.txt', 'rb')
d = pickle.load(f)
f.close()
```

Pickle的问题和所有其他编程语言特有的序列化问题一样，就是 **它只能用于Python** ，并且可能不同版本的Python彼此都不兼容，因此，只能用Pickle保存那些不重要的数据，不能成功地反序列化也没关系。

### JSON

最好使用标准格式进行序列化，便于跨平台、跨编程语言。

JSON表示的对象就是标准的JavaScript语言的对象，JSON和Python内置的数据类型对应如下：

```
SON类型        Python类型
{}            dict
[]            list
"string"      str
1234.56       int或float
true/false    True/False
null          None
```

Python内置的json模块提供了非常完善的Python对象到JSON格式的转换。

```python
import json

# python dict 转 json
d = dict(name='Bob', age=20, score=88)
json.dumps(d)
# '{"age": 20, "score": 88, "name": "Bob"}'

# json to python dict
json_str = '{"age": 20, "score": 88, "name": "Bob"}'
json.loads(json_str)
# {'age': 20, 'score': 88, 'name': 'Bob'}
```



