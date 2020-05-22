---
title: "Python 错误处理与测试"
date: 2019-11-12 16:01
tag: python
---

[TOC]

---

## 异常处理

使用 `try ... except ... finally` 来处理异常。

```python
try:
    x = 1 / 0
    print(x)
except ZeroDivisionError as e:
    print('except: ', e)
finally:
    print('finally')
```

try 代码块的代码如果执行出错，则后续代码不会运行，而是跳转到错误处理代码，即 except 代码块，执行完except后，如果有finally语句块，则执行finally语句块。

如果发生了不同类型的错误，应该由不同的except语句块处理，可以有多个except来捕获不同类型的错误。

Python的错误其实也是class，所有的错误类型都继承自BaseException，所以在使用except时需要注意的是，它不但捕获该类型的错误，还把其子类也捕获了。

如果错误没有被捕获，它就会一直往上抛，最后被Python解释器捕获，打印一个错误信息，然后程序退出。

既然错误是 class ，我们也可以自己定义，用raise语句抛出一个错误的实例：

```python
# err_raise.py
class FooError(ValueError):
    pass

def foo(s):
    n = int(s)
    if n==0:
        raise FooError('invalid value: %s' % s)
    return 10 / n

foo('0')
```

捕获错误目的只是记录一下，便于后续追踪；如果当前函数不知道应该怎么处理该错误，可以直接用 raise 语句继续往上抛，让顶层调用者去处理。

## logging

如果调试时只使用 print ，那么代码会慢慢变得很乱，可以使用 logging 模块。

```python
import logging

# 设置日志级别
logging.basicConfig(level=logging.INFO)

s = '0'
n = int(s)
logging.info('n = %d' % n)
print(10 / n)
```

## pdb

类似于 gdb，pdb 是 Python 的调试工具。

```
python -m pdb err.py
```

## 单元测试

Python 自带一个 `unittest` 模块。

```python
import unittest

class MyTest(unittest.TestCase):

    def test1(self):
        a, b = 1, 2
        self.assertEqual(a + b, 3)

    def test2(self):
        d = {'a': 1, 'b': 2}
        self.assertTrue('a' in d)

if __name__ == '__main__':
    unittest.main()
```

可以在单元测试中编写两个特殊的 `setUp()` 和 `tearDown()` 方法。这两个方法会分别在每调用一个测试方法的前后分别被执行。帮助我们减少测试开始前后的重复的准备工作（比如连接、关闭数据库）。

```python
class MyTest(unittest.TestCase):

    def setUp(self):
        print('setUp...')

    def tearDown(self):
        print('tearDown...')
```

## 文档测试

自动测试写在注释文档中的代码，可以利用 Python 中的 `doctest` 模块：

```python
class MyAdd(object):
    '''
    Test docttest

    >>> my_add = MyAdd()
    >>> my_add.add(1, 2)
    3
    '''
    def __init__(self):
        self.sum = 0

    def add(self, a, b):
        self.sum = a - b  # wrong
        assert(self.sum == a + b)
        return self.sum

if __name__=='__main__':
    import doctest
    doctest.testmod()
```

上面的 add 实现错了，直接运行， doctest 会给出错误提示：

```
Failed example:
    my_add.add(1, 2)
Exception raised:
    Traceback (most recent call last):
      File "/Users/yilin/anaconda/envs/py27/lib/python2.7/doctest.py", line 1315, in __run
        compileflags, 1) in test.globs
      File "<doctest __main__.MyAdd[1]>", line 1, in <module>
        my_add.add(1, 2)
      File "tmp.py", line 14, in add
        assert(self.sum == a + b)
    AssertionError
**********************************************************************
1 items had failures:
   1 of   2 in __main__.MyAdd
***Test Failed*** 1 failures.
```


