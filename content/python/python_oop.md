---
title: "Python 面向对象编程"
date: 2019-11-11 11:01
tag: python
---

[TOC]

OOP把对象作为程序的基本单元，一个对象封装了数据和操作数据的函数。

面向对象的程序设计把计算机程序视为一组对象的集合，而每个对象都可以接收其他对象发过来的消息，并处理这些消息，计算机程序的执行就是一系列消息在各个对象之间传递。

Python中，所有数据类型都可以视为对象，也可以自定义对象。自定义的对象数据类型就是面向对象中的类（Class）的概念。

参考自 [廖雪峰Python教程](https://www.liaoxuefeng.com/wiki/1016959663602400/1017495723838528)

---

## 类和实例

```python
class Student(object):
    # __init__ 是一个特殊方法，相当于 C++ 中的构造函数
    def __init__(self, name, score):
        # self 表示要创建的实例本身，类似于 C++ 中的 this 指针
        self.name = name
        self.score = score
    # 注意类的方法的第一个参数永远是 self
    def print_name(self):
        print(self.name)
    def print_score(self):
        print(self.score)

# stu 是 Student 类(class)的一个实例(instance)
stu = Student("Bob", 80)
stu.print_name()
stu.print_score()
```

```python
# 关于 python 中的访问限制，如果用 __ 开头（而且不以 __ 结尾），则为私有变量，但实际上还是有办法访问
# 比如 Student 类有一个私有变量 __var，可以通过 _Student__var 访问到（依赖于 Python解释器）
# 需要自觉遵守约定不在外部访问私有变量
# __name__ 这种形式的变量是特殊变量，不是私有变量，外部可以访问到
# 直接在类作用域定义的变量是类变量，所有实例都能够访问到
# 类变量既可以通过类名来访问，也可以通过实例名来访问。推荐用类名来访问
# 要注意相同名称的实例变量将屏蔽掉类变量
class Student(object):
    # 类变量
    class_var = "class variable"
    # __init__ 是一个特殊方法，相当于 C++ 中的构造函数
    def __init__(self, name, score):
        # self 表示要创建的实例本身，类似于 C++ 中的 this 指针
        self.name = name
        self.score = score
        self.__var = 'private variable'  # 私有变量
    # 注意类的方法的第一个参数永远是 self
    def print_name(self):
        print(self.name)
    def print_score(self):
        print(self.score)
stu = Student('Bob', 80)
print(stu.class_var)
print(Student.class_var)
print(stu.__var)  # error
```

## 继承和多态

```python
class Animal(object):
    def run(self):
        print('Animal is running...')
class Cat(Animal):
    # 子类方法 override
    def run(self):
        print('Cat is running...')
class Dog(Animal):
    # 子类方法 override
    def run(self):
        print('Dog is running...')

def animal_run_twice(animal):
    animal.run()
    animal.run()

# 注意由于 Python 没有类型检查，传入的 animal 可以不是 Animal 类的实例，只要有 run 方法就行了
# “鸭子类型”：一个对象只要“看起来像鸭子，走起路来像鸭子”，那它就可以被看做是鸭子。
```

## 获取对象信息

```python
# 可以使用 type() 获取对象类型
type(123)
type([1, 2, 3])
# 使用isinstance() 判断继承关系
instance(cat, Animal)
# 使用 dir() 获取一个对象的所有属性和方法，它返回一个包含字符串的list
# 配合 getattr()、setattr() 以及 hasattr()，我们可以直接操作一个对象的状态：
class MyObject(object):
    def __init__(self):
        self.x = 9
    def power(self):
        return self.x * self.x
obj = MyObject()
hasattr(obj, 'x') # 有属性'x'吗？
# True
obj.x
# 9
hasattr(obj, 'y') # 有属性'y'吗？
# False
setattr(obj, 'y', 19) # 设置一个属性'y'
hasattr(obj, 'y') # 有属性'y'吗？
# True
getattr(obj, 'y') # 获取属性'y'
# 19
obj.y # 获取属性'y'
# 19
# 也可以获得对象的方法
hasattr(obj, 'power') # 有属性'power'吗？
# True
getattr(obj, 'power') # 获取属性'power'
# <bound method MyObject.power of <__main__.MyObject object at 0x10077a6a0>>
fn = getattr(obj, 'power') # 获取属性'power'并赋值到变量fn
fn # fn指向obj.power
# <bound method MyObject.power of <__main__.MyObject object at 0x10077a6a0>>
fn() # 调用fn()与调用obj.power()是一样的
# 81
```

## 使用 @property

Python内置的 `@property` 装饰器负责把一个方法变成属性调用：

```python
class Student(object):

    # 把一个getter方法变成属性，只需要加上@property
    @property
    def score(self):
        return self._score

    # @property本身又创建了另一个装饰器@score.setter，负责把一个setter方法变成属性赋值
    @score.setter
    def score(self, value):
        if not isinstance(value, int):
            raise ValueError('score must be an integer!')
        if value < 0 or value > 100:
            raise ValueError('score must between 0 ~ 100!')
        self._score = value

s = Student()
s.score = 60 # OK，实际转化为s.set_score(60)
s.score # OK，实际转化为s.get_score()
# 60
s.score = 9999
# Traceback (most recent call last):
#   ...
# ValueError: score must between 0 ~ 100!

# 可以实现只读属性，只定义getter方法，不定义setter方法就是一个只读属性
```

## 使用 `__slots__`

Python 是动态语言，创建了一个class的实例后，我们可以给该实例绑定任何属性和方法：

```python
class Student(object):
    pass

stu = Student()
stu.name = 'Bob'  # 动态给实例绑定一个属性

def set_age(self, age): # 定义一个函数作为实例方法
    self.age = age

from types import MethodType
s.set_age = MethodType(set_age, s) # 给实例绑定一个方法
s.set_age(25) # 调用实例方法
s.age # 测试结果
# 25

# 为了给所有实例都绑定方法，可以给class绑定方法：
def set_score(self, score):
    self.score = score

Student.set_score = set_score
```

Python允许在定义class的时候，定义一个特殊的 `__slots__` 变量，来限制该class实例能添加的属性：

```python
class Student(object):
    __slots__ = ('name', 'age') # 用tuple定义允许绑定的属性名称
```

使用 `__slots__` 要注意， `__slots__` 定义的属性仅对当前类实例起作用，对继承的子类是不起作用的。

## 多重继承

Python 支持多重继承：

```python
class Animal(object):
    pass

# 大类:
class Mammal(Animal):
    pass

class Bird(Animal):
    pass

class Runnable(object):
    def run(self):
        print('Running...')

class Flyable(object):
    def fly(self):
        print('Flying...')

# 各种动物:
class Dog(Mammal, Runnable):
    pass

class Bat(Mammal, Flyable):
    pass

class Parrot(Bird, Flyable):
    pass

class Ostrich(Bird, Runnable):
    pass

```

多重继承的问题是越往后，层次的设计越困难，因此推荐用 **组合** 来完成扩展额外功能更友好。

### Mixin

MixIn的目的就是给一个类增加多个功能，这样，在设计类的时候，我们优先考虑通过多重继承来组合多个MixIn的功能，而不是设计多层次的复杂的继承关系。

```python
class Dog(Mammal, RunnableMixIn, CarnivorousMixIn):
    pass
```

Python自带的很多库也使用了MixIn。举个例子，Python自带了 `TCPServer` 和 `UDPServer` 这两类网络服务，而要同时服务多个用户就必须使用多进程或多线程模型，这两种模型由 `ForkingMixIn` 和 `ThreadingMixIn` 提供。通过组合，我们就可以创造出合适的服务来。

## 定制(customize)类

看到类似 `__slots__` 这种形如 `__xxx__` 的变量或者函数名就要注意，这些在Python中是有特殊用途的。

`__slots__` 我们已经知道怎么用了， `__len__()` 方法我们也知道是为了能让class作用于 `len()` 函数。

除此之外，Python的class中还有许多这样有特殊用途的函数，可以帮助我们定制类。

### `__str__`

帮助我们打印对象。

```python
class Student(object):
    def __init__(self, name):
        self.name = name
    def __str__(self):
        return 'Student object (name=%s)' % self.name
    __repr__ = __str__

print(Student('Michael'))
# Student object (name: Michael)
```

### `__iter__`

如果一个类想被用于 `for ... in` 循环，类似list或tuple那样，就必须实现一个 `__iter__()` 方法，该方法返回一个迭代对象，然后，Python的for循环就会不断调用该迭代对象的 `__next__()` 方法拿到循环的下一个值，直到遇到StopIteration错误时退出循环。

```python
class Fib(object):
    def __init__(self):
        self.a, self.b = 0, 1 # 初始化两个计数器a，b

    def __iter__(self):
        return self # 实例本身就是迭代对象，故返回自己

    def __next__(self):
        self.a, self.b = self.b, self.a + self.b # 计算下一个值
        if self.a > 100000: # 退出循环的条件
            raise StopIteration()
        return self.a # 返回下一个值

for n in Fib():
    print(n)
```

### `__getitem__`

要实现像list那样按照下标取出元素，需要实现 `__getitem__()` 方法：

```python
class Fib(object):
    def __getitem__(self, n):
        a, b = 1, 1
        for x in range(n):
            a, b = b, a + b
        return a
```

但上面的写法不支持 `slice` 操作，需要再做判断：

```python
class Fib(object):
    def __getitem__(self, n):
        if isinstance(n, int): # n是索引
            a, b = 1, 1
            for x in range(n):
                a, b = b, a + b
            return a
        if isinstance(n, slice): # n是切片
            start = n.start
            stop = n.stop
            if start is None:
                start = 0
            a, b = 1, 1
            L = []
            for x in range(stop):
                if x >= start:
                    L.append(a)
                a, b = b, a + b
            return L

f = Fib()
f[0:5]
# [1, 1, 2, 3, 5]

# 但是上面的实现没有对 slice 中的 step 做处理
f[0:2:5]  # 还是 [1, 1, 2, 3, 5]
```

### `__getattr__`

正常情况下，当我们调用类的方法或属性时，如果不存在，就会报错。

使用 `__getattr__()` 方法，可以动态返回一个属性。

```python
class Student(object):

    def __init__(self):
        self.name = 'Michael'

    def __getattr__(self, attr):
        if attr=='score':
            return 99

s = Student()
s.name
# 'Michael'
s.score
# 99
s.score = 1
# 1
```

注意，只有在没有找到属性的情况下，才调用 `__getattr__` ，已有的属性，比如name，不会在 `__getattr__` 中查找。

实际上可以把一个类的所有属性和方法调用全部动态化处理，好处是可以针对完全动态的情况作调用。

动态调用的好处举例：

现在很多网站都搞REST API，比如新浪微博、豆瓣啥的，调用API的URL类似：

- http://api.server/user/friends
- http://api.server/user/timeline/list

如果要写SDK，给每个URL对应的API都写一个方法，那得累死，而且，API一旦改动，SDK也要改。

```python
class Chain(object):

    def __init__(self, path=''):
        self._path = path

    def __getattr__(self, path):
        return Chain('%s/%s' % (self._path, path))

    def __str__(self):
        return self._path

    __repr__ = __str__

Chain().status.user.timeline.list
# '/status/user/timeline/list'
```

这样，无论API怎么变，SDK都可以根据URL实现完全动态的调用，而且，不随API的增加而改变！

### `__call__`

类似于 C++ 中的 `operator()`：

```python
class Student(object):
    def __init__(self, name):
        self.name = name

    def __call__(self):
        print('My name is %s.' % self.name)

s = Student('Michael')
s() # self参数不要传入
# My name is Michael.
```

`__call__()` 还可以定义参数。对实例进行直接调用就好比对一个函数进行调用一样，所以你完全可以把对象看成函数，把函数看成对象，因为这两者之间本来就没啥根本的区别。

通过 `callable()` 函数，我们就可以判断一个对象是否是“可调用”对象。

## 枚举类

```python
from enum import Enum

Month = Enum('Month', ('Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec'))

print(Month.Jan)

for name, member in Month.__members__.items():
    print(name, '=>', member, ',', member.value)

# 自定义枚举值类型
from enum import Enum, unique

# 装饰器 unique 帮助我们检查没有重复
@unique
class Weekday(Enum):
    Sun = 0 # Sun的value被设定为0
    Mon = 1
    Tue = 2
    Wed = 3
    Thu = 4
    Fri = 5
    Sat = 6
```

## 元类(metaclass)

`type()` 函数既可以返回一个对象的类型，又可以创建出新的类型，比如，我们可以通过 `type()` 函数创建一个 Hello 类而无需通过 `class Hello` 这样的定义：

```python
def fn(self, name='world'): # 先定义函数
    print('Hello, %s.' % name)

Hello = type('Hello', (object,), dict(hello=fn)) # 创建Hello class

h = Hello()
h.hello()
# Hello, world.
print(type(Hello))
# <class 'type'>
print(type(h))
# <class '__main__.Hello'>
```

### metaclass

除了使用 `type()` 动态创建类以外，要控制类的创建行为，还可以使用 `metaclass` 。

我们通过类创建实例，通过元类创建类。

```python
# metaclass是类的模板，所以必须从`type`类型派生：
class ListMetaclass(type):
    def __new__(cls, name, bases, attrs):
        attrs['add'] = lambda self, value: self.append(value)
        return type.__new__(cls, name, bases, attrs)

# 传入关键字参数metaclass
class MyList(list, metaclass=ListMetaclass):
    pass
```

当我们传入关键字参数 `metaclass` 时，魔术就生效了，它指示Python解释器在创建 `MyList` 时，要通过 `ListMetaclass.__new__()` 来创建，在此，我们可以修改类的定义，比如，加上新的方法，然后，返回修改后的定义。

动态修改有什么意义？直接在 `MyList` 定义中写上 `add()` 方法不是更简单吗？正常情况下，确实应该直接写，通过 `metaclass` 修改纯属变态。

但是，总会遇到需要通过metaclass修改类定义的。ORM就是一个典型的例子。

ORM全称“Object Relational Mapping”，即对象-关系映射，就是把关系数据库的一行映射为一个对象，也就是一个类对应一个表，这样，写代码更简单，不用直接操作SQL语句。

要编写一个ORM框架，所有的类都只能动态定义，因为只有使用者才能根据表的结构定义出对应的类来。


