---
title: "Python 函数式编程"
date: 2019-11-09 01:01
tag: python
---

[TOC]

Functional Programming, 函数式编程就是一种抽象程度很高的编程范式，纯粹的函数式编程语言编写的函数没有变量，因此，任意一个函数，只要输入是确定的，输出就是确定的，这种纯函数我们称之为没有副作用。而允许使用变量的程序设计语言，由于函数内部的变量状态不确定，同样的输入，可能得到不同的输出，因此，这种函数是有副作用的。

函数式编程的一个特点就是，允许把函数本身作为参数传入另一个函数，还允许返回一个函数。

Python对函数式编程提供部分支持。由于Python允许使用变量，因此，Python不是纯函数式编程语言。

参考自[廖雪峰Python教程](https://www.liaoxuefeng.com/wiki/1016959663602400/1017328525009056)。

---

## 高阶函数

Higher-order function.

将函数作为参数的函数称为高阶函数。

```python
def f1(x):
    return x

def f2(x):
    return x**2

# g 是一个 high-order function
def g(xs, f):
    return sum([f(x) for x in xs])

xs = [1, 2, 3]

print(g(xs, f1))  # 6
print(g(xs, f2))  # 14
```

### map/reduce

`map()` 函数接收两个参数，一个是函数，一个是 `Iterable` ， `map` 将传入的函数依次作用到序列的每个元素，并把结果作为新的 `Iterator` 返回。

```python
def f(x):
    return x**2

xs = [1, 2, 3]
map(f, xs)  ## [1, 4, 9]
```

`reduce()` 把一个函数作用在一个序列 `[x1, x2, x3, ...]` 上，这个函数必须接收两个参数， `reduce` 把结果继续和序列的下一个元素做累积计算:

```python
reduce(f, [x1, x2, x3, x4]) = f(f(f(x1, x2), x3), x4)
```

```python
def add(x, y):
    return x + y

reduce(add, [1, 2, 3])  ## 6
```

这些高阶函数配合 lambda 使用可以写出非常简洁的代码。

### filter

Python内建的 `filter()` 函数用于过滤序列。

`filter()` 把传入的函数依次作用于每个元素，然后根据返回值是 `True` 还是 `False` 决定保留还是丢弃该元素。

```python
# 过滤偶数
def is_odd(x):
    return x % 2 == 1

list(filter(is_odd, [1, 2, 3, 4, 5, 6]))  # 1 3 5

# 删掉一个序列中的空字符串
def not_empty(s):
    return s and s.strip()

list(filter(not_empty, ['A', '', 'B', None, 'C', '  ']))
# 结果: ['A', 'B', 'C']
```

注意到 `filter()` 函数返回的是一个 `Iterator` ，也就是一个惰性序列，所以要强迫 `filter()` 完成计算结果，需要用 `list()` 函数获得所有结果并返回 `list` 。

### sorted

Python内置的 `sorted()` 函数可以对 `list` 进行排序。

```python
>>> sorted([36, 5, -12, 9, -21])
[-21, -12, 5, 9, 36]
```

`sorted()` 是一个高阶函数，它还可以接收一个 `key` 函数来实现自定义的排序，例如按绝对值大小排序：

```python
>>> sorted([36, 5, -12, 9, -21], key=abs)
[5, 9, -12, ,-21, 36]
```

`sorted()` 还有一个参数是 `reverse`，为 `True` 时表示反向排序：

```python
>>> sorted([36, 5, -12, 9, -21], key=abs, reverse=True)
[36, -21, -12, 9, 5]
```

## 函数作为返回值

高阶函数除了可以接受函数作为参数外，还可以把函数作为结果值返回。

```python
def lazy_sum(*args):
    def sum():
        ax = 0
        for n in args:
            ax = ax + n
        return ax
    return sum  # 返回一个函数

# 当我们调用lazy_sum()时，返回的并不是求和结果，而是求和函数：

>>> f = lazy_sum(1, 3, 5, 7, 9)
>>> f
<function lazy_sum.<locals>.sum at 0x101c6ed90>

# 调用lazy_sum()时，每次调用都会返回一个新的函数
>>> f1 = lazy_sum(1, 3, 5, 7, 9)
>>> f2 = lazy_sum(1, 3, 5, 7, 9)
>>> f1==f2
False
```

这里 sum 是一个闭包（Closure），它可以访问到 lazy_sum 中定义的变量。

### 闭包

> 在计算机科学中，闭包（Closure）是词法闭包（Lexical Closure）的简称，是引用了自由变量的函数。这个被引用的自由变量将和这个函数一同存在，即使已经离开了创造它的环境也不例外。所以，有另一种说法认为闭包是由函数和与其相关的引用环境组合而成的实体。

```python
def count():
    n = 0
    def f():
        # 注意 nonlocal 关键字，表示 n 是外部作用域定义的
        nonlocal n
        n += 1
        return n
    return f

f1 = count()
f2 = count()
f3 = count()
# f1 f2 f3 互相之间不会干扰
print(f1(), f2(), f3())  # 1 1 1
print(f1(), f2(), f3())  # 2 2 2
print(f1(), f2(), f3())  # 3 3 3
```

使用闭包要注意，返回函数不要引用任何循环变量，或者后续会发生变化的变量：

```python
# 错误的写法
def count():
    fs = []
    for i in range(1, 4):
        def f():
             return i*i
        fs.append(f)
    return fs

f1, f2, f3 = count()
f1(), f2(), f3()  # 9 9 9
# 因为 `return fs` 时，变量 i 的值为 3。

# 正确的写法
# 再创建一个函数，用该函数的参数绑定循环变量当前的值
# 无论该循环变量后续如何更改，已绑定到函数参数的值不变
def count():
    fs = []
    def f(j):
        def g():
            return j * j
        return g
    for i in range(1, 4):
        fs.append(f(i))
    return fs

f1, f2, f3 = count()
print(f1(), f2(), f3())
# 1 4 9
```

## lambda

Python 中对 lambda 表达式提供了有限的支持，lambda 表达式可以看成一个匿名函数：

```python
>>> list(map(lambda x: x * x, [1, 2, 3, 4, 5, 6, 7, 8, 9]))
[1, 4, 9, 16, 25, 36, 49, 64, 81]
```

Python 因为本身设计思想，仅仅将 lambda 定位成一个辅助用的短函数，比如不能换行，不能用来实现闭包（无法访问外部作用域变量）。

## 装饰器

在代码运行期间动态增加功能的方式，称之为“装饰器”（Decorator）。

本质上，decorator就是 **一个返回函数的高阶函数** 。

例子：

```python
def now():
    print('2019-09-09')

# 我们希望调用 now() 时能自动打一行 log
# 定义一个 log 装饰器
def log(func):
    # 把传入的函数“包装/装饰”了一下
    def wrapper(*args, **kw):
        print('call %s():' % func.__name__)
        return func(*args, **kw)
    return wrapper

# Python的@语法，把decorator置于函数的定义处
@log
def now():
    print('2019-09-09')

now()  # 实际上执行的是 log(now)()
# call now():
# 2019-09-09
```

如果装饰器函数本身也有参数，那就需要编写一个返回decorator的高阶函数：

```python
def log(text):
    def decorator(func):
        def wrapper(*args, **kw):
            print('%s %s():' % (text, func.__name__))
            return func(*args, **kw)
        return wrapper
    return decorator

@log('call')
def now():
    print('2019-09-09')

now()  # 相当于 log('call')(now)()

# 注意此时 now 的函数名已经变成了 'wrapper'
print(now.__name__)
```

定义装饰器时，一般需要把原始函数的__name__等属性复制到wrapper()函数中，否则，有些依赖函数签名的代码执行就会出错。Python内置的 `functools.wraps` 可以帮我们完成这个任务。

```python
import functools

def log(func):
    @functools.wraps(func)
    def wrapper(*args, **kw):
        print('call %s():' % func.__name__)
        return func(*args, **kw)
    return wrapper
```

## Partial function

`functools.partial` 帮助我们绑定函数的一些参数，返回一个新的函数：

```python
import functools
int2 = functools.partial(int, base=2)
int2('0010')  # 2
int2('0100')  # 4
```