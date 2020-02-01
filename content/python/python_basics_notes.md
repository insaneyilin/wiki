---
title: "Python Basics Notes"
date: 2019-07-09 01:01
tag: python
---

[TOC]

> Python（英国发音：/ˈpaɪθən/ 美国发音：/ˈpaɪθɑːn/）是一种广泛使用的解释型、高级编程、通用型编程语言，由吉多·范罗苏姆创造，第一版发布于1991年。可以视之为一种改良（加入一些其他编程语言的优点，如面向对象）的LISP。Python的设计哲学强调代码的可读性和简洁的语法（尤其是使用空格缩进划分代码块，而非使用大括号或者关键词）。相比于C++或Java，Python让开发者能够用更少的代码表达想法。不管是小型还是大型程序，该语言都试图让程序的结构清晰明了。
>  -- [Wikipedia--Python](https://zh.wikipedia.org/wiki/Python)

### Python中的五种基本对象类型：

- 字符串（string），简记为`str`
    - 使用''或""括起来的一系列字符
- 整数（integer），简记为`int`
    - 十进制：21；八进制：025；十六进制：0x15
- 浮点数（float）
    - 1.65, 21.0, 21., .21, 2.1E1
- 布尔数（boolean），简记为`bool`
    - `True`, `False`
- 复数（complex）
    - `1 + 1j`

---

### 强制类型转换：

`int('123')` -> 123

`str(123)` -> '123'

`float('123')` -> 123.0

`float(123)` -> 123.0

`bool(123)` -> `True`

`bool(123)` -> `False`

---

### 算术运算符

注意除法`/`表示向下取整除（floor division），即两个整数相除，结果为舍去小数部分后的整数。

`**`为指数运算符。

---

### 自动类型转换

`bool` -> `int` -> `float` -> `complex`

---

### 模块（module）

```python
import module_name
```

```python
import math
dir(math)  # 查看模块内容
help(math.sin)  # 查看帮助
```

---

### 关系与逻辑运算

`<>`和`!=`都表示不等于。

与：`and`；或：`or`；非：`not`。

---

### 标识符（Identifier）

命名规则：

- 可以任意长
- 可以包含数字、字母、下划线
- 首个字符必须是字母或下划线
- 大小写敏感
- 标识符不能是关键字

---

### 标准键盘输入

`raw_input`函数，读取键盘输入，将所有输入作为字符串看待。

```python
radius = float(raw_input('Radius: '))
```

---

### 分支结构

Python中没有`else if`，而是用`elif`代替。

```python
if score >= 90.0:
    count90 += 1
elif score >= 80.0:
    count80 += 1
# ......
```

---

### 循环结构

`while`语句。

```python
num = 1
Total_Num = 3
count = 0

while (num <= Total_Num):
    grade = input("Input No." + str(num) + " student\'s grade: ")
    if (grade >= 60):
        count += 1
    else:
        pass
    num += 1
    
print "Total number of passed student is ", count
```

`for`语句。

```python
count = 0

for num in range(1, 4):
    grade = input("Input No." + str(num) + " student\'s grade: ")
    if (grade >= 60):
        count += 1
    else:
        pass
        
print "Total number of passed student is ", count
```

`break`与`continue`与C++中意义相同。

#### 同时迭代元素下标和元素

```python
for i, elem in enumerate(['a', 'b', 'c']):
    print(i, elem)
```

---

### 函数

```python
def sum(i1, i2):
    """
    To calculate the sum from i1 to i2
    i1 is low value, i2 is high value
    """
    result = 0
    for i in range(i1, i2 + 1):
        result += i
    return result  # return sum value
```

Python的函数参数支持显示调用，如：

```python
sum(i2 = 2, i1 = 0)
```

#### 函数的变量作用域

局部变量：

- 只能在程序的特定部分使用的变量
- 函数内部

全局变量：

- 为整个程序所使用的变量
- 所有函数均可以使用

#### 函数的参数

Python 函数可以使用默认参数、可变参数和关键字参数。

默认参数：

```python
def foo(a, b=1):
    return a + b
```

一个坑，不要用 `L=[]` 作为默认参数：

```python
def add_end(L=[]):
    L.append('END')
    return L

>>> add_end()
['END', 'END']
>>> add_end()
['END', 'END', 'END']
# 默认参数L也是一个变量，它指向对象[]，每次调用该函数，如果改变了L的内容，则下次调用时，默认参数的内容就变了，不再是函数定义时的[]了

# 正确实现
def add_end(L=None):
    if L is None:
        L = []
    L.append('END')
    return L
```

结论：**默认参数必须指向不变对象！**。

可变参数：

```python
def calc(*numbers):
    sum = 0
    for n in numbers:
        sum = sum + n
    return sum

calc()
# 0
calc(1, 2, 3)
# 6

nums = [1, 2, 3]
calc(*nums)  # * 的作用，在函数调用中，* 能够将元组或者列表解包成不同的参数

# 类似的，** 可以解包一个字典
```

关键字参数：

```python
# 关键字参数允许你传入0个或任意个含参数名的参数，这些关键字参数在函数内部自动组装为一个dict
def person(name, age, **kw):
    print('name:', name, 'age:', age, 'other:', kw)

>>> person('Bob', 35, city='Beijing')
name: Bob age: 35 other: {'city': 'Beijing'}
>>> person('Adam', 45, gender='M', job='Engineer')
name: Adam age: 45 other: {'gender': 'M', 'job': 'Engineer'}

# 字典解包
>>> extra = {'city': 'Beijing', 'job': 'Engineer'}
>>> person('Jack', 24, **extra)
name: Jack age: 24 other: {'city': 'Beijing', 'job': 'Engineer'}
```

**对于任意函数，都可以通过类似func(*args, **kw)的形式调用它，无论它的参数是如何定义的。**

注意函数接口可读性。

- `*args` 是可变参数， `args` 接收的是一个 `tuple` ；
- `**kw` 是关键字参数， `kw` 接收的是一个 `dict` 。

#### 递归函数

```python
# 非尾递归
def fact(n):
    if n==1:
        return 1
    return n * fact(n - 1)

# 尾递归
def fact(n):
    return fact_iter(n, 1)

def fact_iter(num, product):
    if num == 1:
        return product
    return fact_iter(num - 1, num * product)
```

大多数编程语言没有针对尾递归做优化，Python解释器也没有做优化，所以，即使把上面的 `fact(n)` 函数改成尾递归方式，也会导致栈溢出。

---

### Python中的字符串

成对的单引号或双引号括起来的内容。

使用三个引号（单或双）括起来的字符串将保留全部格式信息（如换行等）。

#### 基本的字符串操作：

- `len`，字符串长度

```python
first_name = 'Michael'
len(first_name)
```

- `+`，拼接

```python
name = first_name + 'Jordan'
print name
```

- `*`，重复

```python
name * 3
```

- `in`，判断一个字符串是否是另一个字符串的子串

```python
name = 'Michael Jordan'
'a' int name  ## 值为True
```

- `for`，枚举字符串的每个字符

```python
my_str = 'hello world'
for char in my_str:
    print char
```

---

#### 字符串索引（index）

- 字符串中每个字符都有一个索引值（下标）
- 索引从0（前向）或-1（后向）开始

<br>

#### 字符串切片（Slicing）

语法：

- [start:finish]，start或finish可以省略。
- [start:finish:countBy]，默认countBy为1

<br>

#### 获取逆字符串：

```python
my_str = 'spam'
reverse_str = my_str[::-1]
print reverse_str
```

---

#### **字符串是不可变（Immutable）的**

下面这样写是错的：

```python
my_str = 'hello world'
my_str[1] = 'a'  # 出错！
```

应该通过slicing生成相应的新字符串：

```python
my_str = 'hello world'
my_str = my_str[:1] + 'a' + my_str[2:]
print my_str
```

---

#### 常用字符串方法

`replace`，如`my_str.replace(old, new)`

注意：

- `replace` 方法返回一个新的字符串，原字符串内容不变

```python
my_str = 'hello world'
my_str = my_str.replace('e', 'a')
print my_str
```

`find`

```python
>>> my_str = 'hello world'
>>> my_str.find('l')  # find index of 'l' in my_str
2
```

`split`

```python
>>> my_str = 'hello world'
>>> my_str.split()  # 默认以空格为分隔符
['hello', 'world']
```

---

#### 字符串格式化（Formatting）

`format`方法

```python
>>> print "Hello {} good {}.".format(5, 'Day')
Hello 5 good Day.
```

括号的形式为  
`{field name:align width.precision type} `

```python
>>> print math.pi
3.14159265359
>>> print 'PI is {:.4f}'.format(math.pi)
PI is 3.1416
>>> print 'PI is {:9.4f}'.format(math.pi)
PI is    3.1416
>>> print 'PI is {:e}'.format(math.pi)
PI is 3.141593e+00
```

---

### 列表（List）

与字符串类似，但有如下不同点：

- 使用`[]`生成，元素之间用逗号分隔；
- 可以包含多种类型的对象；
- 内容是可变的；而字符串是不可变的

<br>

#### 列表作函数参数

交换两个变量的值？

```python
def swap(a, b):
    tmp = a
    a = b
    b = tmp

x = 10
y = 20

swap(x, y)
print x, y  # 10, 20
```

上面代码无法起到效果，使用列表作为函数参数可以解决。

```python
def swap(lst, a, b):
    tmp = lst[a]
    lst[a] = lst[b]
    lst[b] = tmp
    
x = [10, 20, 30]
swap(x, 0, 1)
print x  # [20, 10, 30]
```

#### 内建排序函数

- `sorted()`函数

```python
>>> a = [5, 2, 3, 1, 4]
>>> sorted(a)
[1, 2, 3, 4, 5]
```

- `list.sort()`方法

```python
>>> a = [5, 2, 3, 1, 4]
>>> a.sort()
>>> a
[1, 2, 3, 4, 5]
```

---

#### 嵌套列表

```python
x = [[5, 4, 7, 3], [4, 8, 9, 7], [5, 1, 2, 3]]
x[2][1]  # 第三行第二列
```

<br>

#### 列表解析或推导（List Comprehension）

**由原列表创建新列表**

`[表达式 for语句 条件]`

如生成值为{x^2: x in {1...9}}的列表：

```python
lst = []
for x in range(1, 10):
    lst.append(x**2)
    
print lst

# 使用列表推导只需要一行
lst = [x**2 for x in range(1, 10)]
```

### 生成器(generator)

通过 list comprehension 可以直接创建列表，但如果列表过长，会造成空间浪费。

使用 generator 可以边循环边计算列表元素，节省空间。

```python
>>> L = [x * x for x in range(10)]
>>> L
[0, 1, 4, 9, 16, 25, 36, 49, 64, 81]
>>> g = (x * x for x in range(10))
>>> g
<generator object <genexpr> at 0x1022ef630>

# 通过next()函数获得generator的下一个返回值：
next(g) # 0
next(g) # 1
next(g) # 14

# 也可以直接用 in 来循环
for x in g:
    print(x)
```

另一种定义 generator 的方式是使用关键字 `yield`：

```python
def fib(max):
    n, a, b = 0, 0, 1
    while n < max:
        yield(b)
        a, b = b, a + b
        n += 1
    return
# 打印斐波那契数列
for x in fib(6):
    print(x)
```

如果一个函数定义中包含 `yield` 关键字，那么这个函数就不再是一个普通函数，而是一个generator。

使用 generator 打印杨辉三角：

```python
def triangles():
    l_prev = []
    l_cur = [1]
    while True:
        yield(l_cur)
        l_prev = l_cur
        l_cur = [1]
        l_prev_size = len(l_prev)
        for i in xrange(1, l_prev_size):
            l_cur.append(l_prev[i] + l_prev[i-1])
        l_cur.append(1)
    return

n = 0
results = []
for t in triangles():
    results.append(t)
    n = n + 1
    if n == 10:
        break

for t in results:
    print(t)
# [1]
# [1, 1]
# [1, 2, 1]
# [1, 3, 3, 1]
# [1, 4, 6, 4, 1]
# [1, 5, 10, 10, 5, 1]
# [1, 6, 15, 20, 15, 6, 1]
# [1, 7, 21, 35, 35, 21, 7, 1]
# [1, 8, 28, 56, 70, 56, 28, 8, 1]
# [1, 9, 36, 84, 126, 126, 84, 36, 9, 1]
```

### 迭代器(Iterator)

可以使用 `for` 循环的数据类型：

- 集合数据类型，`list`, `dict`, `tuple`, `set`, `str` 等
- `generator`，包括生成器和带 `yield` 的generator function

这些可以直接作用于 `for` 循环的对象统称为可迭代对象：`Iterable`

```python
>>> from collections import Iterable
>>> isinstance([], Iterable)
True
>>> isinstance({}, Iterable)
True
>>> isinstance('abc', Iterable)
True
>>> isinstance((x for x in range(10)), Iterable)
True
>>> isinstance(100, Iterable)
False
```

可以被 `next()` 函数调用并不断返回下一个值的对象称为迭代器：`Iterator` 。

生成器都是 `Iterator` 对象，但 `list` 、 `dict` 、 `str` 虽然是 `Iterable` ，却不是 `Iterator`

把 `list` 、 `dict` 、 `str` 等 `Iterable` 变成 `Iterator` 可以使用 `iter()` 函数：

```python
>>> isinstance(iter([]), Iterator)
True
>>> isinstance(iter('abc'), Iterator)
True
```

Python的 `Iterator` 对象表示的是一个 **数据流** ， `Iterator` 对象可以被 `next()` 函数调用并不断返回下一个数据，直到没有数据时抛出 `StopIteration` 错误。可以把这个数据流看做是一个有序序列，但我们却不能提前知道序列的长度，只能不断通过 `next()` 函数实现按需计算下一个数据，所以 `Iterator` 的计算是 **惰性** 的，只有在需要返回下一个数据时它才会计算。

`Iterator` 甚至可以表示一个无限大的数据流，例如全体自然数。而使用 `list` 是永远不可能存储全体自然数的。

---

### lambda函数

定义匿名函数：

```python
>>> def f(x): return x**2
>>> print f(8)
64
```

```python
>>> g = lambda x: x**2
>>> print g(8)
64
```

---

### 元组（Tuple）

特点：不可变（immutable）

使用逗号（可以加括号）来创建Tuple：

```python
my_tuple = 1, 'a', 3.14, True
my_tuple = (1, 'a', 3.14, True)
```

<br>

#### 函数与元组

- 函数只能有一个返回值，但是可以返回一个元组

```python
def max_min(lst):
    max = min = lst[0]
    for i in lst:
        if i > max:
            max = i
        if i < min:
            min = i
    return max, min  # 返回一个元组
```

---

### 字典（Dictionary）

特点：key-value

使用`{}`创建字典

```python
my_dict = {'John':86411234, 'Bob':86419453, 'Mike':86412387}
print my_dict['Bob']
```

key必须是**不可变**的且不重复，value可以是任意类型。

<br>

#### 常用的dict方法

- `len`，字典长度
- `key in my_dict`，等价于`my_dict.has_key(key)`
- `for key in my_dict:`，枚举字典中的key，注意key是**无序**的。

其他方法：

- `my_dict.items()`，全部的key-value对
- `my_dict.keys()`，全部的key
- `my_dict.values()`，全部的value
- `my_dict.clear()`，清空字典

---

### 集合（set）

特点：无序不重复的元素（key）集

和字典类似，但是无value

创建：

```python
x = set()
x = set([1, 2, 2, 4, 3, 1]) # 会去除重复元素。1, 2, 4, 3
```

添加和删除：

```python
x.add('body')
x.remove('body')
```

集合的运算符：

- `-`，差集
- `&`，交集
- `|`，并集
- `!=`，不等于
- `==`，等于
- `in`，成员
- `for key in set`，枚举

---

### Python 面向对象

上面没有讲面向对象的内容，这里补充一下（http://code.google.com/p/hashao/wiki/ChinesePythonTutor）。

```python
class RenLei:
    def __init__(self, mingzi):  # self 是一个固定参数名，代表这个对象自己。
        self.mingzi = mingzi     # 把名字存到对象自己的 mingzi 变量上。
        self.xingbie = "nv3"     # 性别，预设为女
    def shui2(self):             # 谁函数，提取姓名
        return self.mingzi
    def gai3ming2(self, xin_mingzi):    # 改名函数
        self.mingzi = xin_mingzi
```

`__init__`是构造函数，类的成员变量（属性）在其中定义。

```python
self.属性名 = 初始值
```

类函数的定义，是直接在类里面，缩进四个空格，像一般函数定义就可以了。只是别忘了第一个参数，一定要是 `self` 。

```python
# 建立对象，提供的参数对应__init__() 函数，self 这个参数程序会自动提供，不用我们提供。
xiaozhang1 = RenLei("小张")  # 建立一个叫小张的人
mingzi = xiaozhang1.shui2()  # 使用“对象名.函数名()”的格式来调用类里的函数。
print (mingzi) # 打印 "小张"
```

### Python 程序员编程习惯

重中之重: 大量的使用 `list` 这个列表方式来存储、操作数据。**一字长蛇阵**是蟒蛇的绝招，多用没错。

1. `import`所有会用到的模块 
2. 定义我们自己的类和函数。对于每个函数里要用到的函数，被调用的函数一般放在调用函数的前面定义。这样我们读程序的时候，如果从上到下，就知道调用的函数，在前面是怎么定义的，有什么功能，需要什么参数，返回什么值。
3. 在文件的最下面，调用上面定义的函数和类，开始工作。
4. 命令行的选项是通过 `sys.argv` 这个**列表**得到的。
5. 尽量使用Python提供的内建函数和模块里的函数，所以我们**对Python的模块说明手册要很熟悉**。
6. 程序的清晰比简洁重要，多写注释，表明自己下面要做什么。越清晰的程序越容易修改。否则十天半个月后您想给自己的程序加个新功能，结果根本看不懂原来的程序了，可怎么加？

### 一个完整的小例子

把一个两列的文件input.txt，分割成两个文件col1.txt, col2.txt，一个文件一列。 input.txt 内容:

```
a1 啊阿
ai1 挨埃哀
ao2 熬鳌翱獒
```

程序代码：

```python
def split_file(filename): # 把文件分成两列
    col1 = [] # 存储第一列
    col2 = []
    fd = open(filename) # open 函数用来打开文件，返回一个文件对象
    text = fd.read() # fd.read 读入文件fd 的内容。
    lines = text.splitlines() # 把读入的内容分行
    for line in lines: # 循环每一行
        part = line.split(None, 1) # 分割一行。
        col1.append(part[0]) # 把分割的第一部分放到col1后面。
        col2.append(part[1])

    return col1, col2 # 返回 col1, col2

def write_list(filename, alist): # 把文字列表内容写入文件
    fd = open(filename, 'w') # 打开输出文件col1.txt，'w'指定使用写入模式。
    for line in alist:
         fd.write(line + '\n')

def main(): # 主函数，程序进入点，习惯性叫他 main()
    filename = 'input.txt'            # 把输入文件名 input.txt 放进一个变量
    col1, col2 = split_file(filename) # 调用分割函数，结果存入 col1, col2
    write_list('col1.txt', col1)      # 调用写入函数
    write_list('col2.txt', col2)

main() # 唯一的函数外命令，程序开始执行，调用上面的 main() 函数数。
```

这里，输入的文件名是写死的 input.txt ，我们可以使用模块 `optparse` 来通过命令行读取用户提供的文件，会更灵活些，那些就是您研究过 `optparse` 以后的事了。

先熟悉:

- 内建函数
- 内建数据类型 (字符串，数字，列表，字典，文件对象)
- sys 模块
- re 模块
- os 模块
- optparse 模块

熟悉这些，基本上编程没问题了。

### 中文编码

对于中文的各种编码，我们只要知道字符串的两个函数：encode() 和 decode()，他们可以把各种编码与标准的统一码进行转换。

```python
    # 把统一码转成 GBK 码
    "我们就是花朵".encode("GBK")

    # 把使用 UTF-8 编码的字符串转成统一码
    "你们才是花朵，你们全家都是花朵".decode("UTF-8")
```

---

以下来自[快速入门：十分钟学会Python](http://blog.jobbole.com/43922/)

### 获取帮助

你可以很容易的通过Python解释器获取帮助。如果你想知道一个对象(object)是如何工作的，那么你所需要做的就是调用`help(<object>)`。另外还有一些有用的方法，`dir()`会显示该对象的所有方法，还有`<object>.__doc__`会显示其文档：

```python
>>> help(5)
Help on int object:
(etc etc)

>>> dir(5)
['__abs__', '__add__', ...]

>>> abs.__doc__
'abs(number) -> number

Return the absolute value of the argument.'
```

### 语法

Python中没有强制的语句终止字符，且代码块是通过缩进来指示的。缩进表示一个代码块的开始，逆缩进则表示一个代码块的结束。声明以冒号(:)字符结束，并且开启一个缩进级别。单行注释以井号字符(#)开头，多行注释则以多行字符串的形式出现。赋值（事实上是将对象绑定到名字）通过等号(“=”)实现，双等号(“==”)用于相等判断，”+=”和”-=”用于增加/减少运算(由符号右边的值确定增加/减少的值)。这适用于许多数据类型，包括字符串。你也可以在一行上使用多个变量。

Python具有列表（list）、元组（tuple）和字典（dictionaries）三种基本的数据结构，而集合(sets)则包含在集合库中(但从Python2.5版本开始正式成为Python内建类型)。列表的特点跟一维数组类似（当然你也可以创建类似多维数组的“列表的列表”），字典则是具有关联关系的数组（通常也叫做哈希表），而元组则是不可变的一维数组（Python中“数组”可以包含任何类型的元素，这样你就可以使用混合元素，例如整数、字符串或是嵌套包含列表、字典或元组）。数组中第一个元素索引值(下标)为0，使用负数索引值能够从后向前访问数组元素，**-1表示最后一个元素**。数组元素还能指向函数。

Python支持有限的多继承形式。私有变量和方法可以通过添加至少两个前导下划线和最多尾随一个下划线的形式进行声明（如“`__spam_`”，这只是惯例，而不是Python的强制要求）。当然，我们也可以给类的实例取任意名称。

外部库可以使用 `import [libname]` 关键字来导入。同时，你还可以用 `from [libname] import [funcname]` 来导入所需要的函数。

### 杂项

- 数值判断可以链接使用，例如 1 < a < 3 能够判断变量 a 是否在1和3之间。
- 可以使用 del 删除变量或删除数组中的元素。
- 列表推导式(List Comprehension)提供了一个创建和操作列表的有力工具。列表推导式由一个表达式以及紧跟着这个表达式的for语句构成，for语句还可以跟0个或多个if或for语句，来看下面的例子：

```python
>>> lst1 = [1, 2, 3]
>>> lst2 = [3, 4, 5]
>>> print [x * y for x in lst1 for y in lst2]
[3, 4, 5, 6, 8, 10, 9, 12, 15]
>>> print [x for x in lst1 if 4 > x > 1]
[2, 3]
# Check if an item has a specific property.
# "any" returns true if any item in the list is true.
>>> any([i % 3 for i in [3, 3, 4, 4, 3]])
True
# This is because 4 % 3 = 1, and 1 is true, so any()
# returns True.

# Check how many items have this property.
>>> sum(1 for i in [3, 3, 4, 4, 3] if i == 4)
2
>>> del lst1[0]
>>> print lst1
[2, 3]
>>> del lst1
```

#### 全局变量

- 全局变量在函数之外声明，并且可以不需要任何特殊的声明即能读取，但**如果你想要修改全局变量的值，就必须在函数开始之处用global关键字进行声明，否则Python会将此变量按照新的局部变量处理**（请注意，如果不注意很容易被坑）。例如：

```python
number = 5

def myfunc():
    # This will print 5.
    print number

def anotherfunc():
    # This raises an exception because the variable has not
    # been bound before printing. Python knows that it an
    # object will be bound to it later and creates a new, local
    # object instead of accessing the global one.
    print number
    number = 3

def yetanotherfunc():
    global number
    # This will correctly change the global.
    number = 3
```

### 字符串和编码

参考自 [这里
](https://www.liaoxuefeng.com/wiki/1016959663602400/1017075323632896)。

字符串是一种数据类型，但是，字符串比较特殊的是还有一个编码问题。

计算机是美国人发明的，最早只有127个字符被编码到计算机里，也就是大小写英文字母、数字和一些符号，这个编码表被称为ASCII编码。

要处理中文显然一个字节是不够的，至少需要两个字节，而且还不能和ASCII编码冲突，所以，中国制定了GB2312编码，用来把中文编进去。

Unicode把所有语言都统一到一套编码里，这样就不会再有乱码问题了。

Unicode标准也在不断发展，但最常用的是用两个字节表示一个字符（如果要用到非常偏僻的字符，就需要4个字节）。现代操作系统和大多数编程语言都直接支持Unicode。

ASCII编码和Unicode编码的区别：ASCII编码是1个字节，而Unicode编码通常是2个字节。

新的问题又出现了：如果统一成Unicode编码，乱码问题从此消失了。但是，如果你写的文本基本上全部是英文的话，用Unicode编码比ASCII编码需要多一倍的存储空间，在存储和传输上就十分不划算。

本着节约的精神，又出现了把Unicode编码转化为“可变长编码”的UTF-8编码。UTF-8编码把一个Unicode字符根据不同的数字大小编码成1-6个字节，常用的英文字母被编码成1个字节，汉字通常是3个字节，只有很生僻的字符才会被编码成4-6个字节。如果你要传输的文本包含大量英文字符，用UTF-8编码就能节省空间。

UTF-8编码有一个额外的好处，就是ASCII编码实际上可以被看成是UTF-8编码的一部分，所以，大量只支持ASCII编码的历史遗留软件可以在UTF-8编码下继续工作。

现在计算机系统通用的字符编码工作方式：在计算机内存中，统一使用Unicode编码，当需要保存到硬盘或者需要传输的时候，就转换为UTF-8编码。

对于单个字符的编码，Python提供了 `ord()` 函数获取字符的整数表示， `chr()` 函数把编码转换为对应的字符。

由于Python的字符串类型是 `str` ，在内存中以Unicode表示，一个字符对应若干个字节。如果要在网络上传输，或者保存到磁盘上，就需要把 `str` 变为以字节为单位的 bytes。

```python
x = b'ABC'
```

要注意区分 `'ABC'` 和 `b'ABC'` ，前者是 `str` ，后者虽然内容显示得和前者一样，但 `bytes` 的每个字符都只占用一个字节。

以Unicode表示的 `str` 通过 `encode()` 方法可以编码为指定的bytes，例如：

```python
>>> 'ABC'.encode('ascii')
b'ABC'
>>> '中文'.encode('utf-8')
b'\xe4\xb8\xad\xe6\x96\x87'
```

反过来，如果我们从网络或磁盘上读取了字节流，那么读到的数据就是bytes。要把bytes变为 `str` ，就需要用 `decode()` 方法：

```python
>>> b'ABC'.decode('ascii')
'ABC'
>>> b'\xe4\xb8\xad\xe6\x96\x87'.decode('utf-8')
'中文'
```

`len()` 函数计算的是 `str` 的字符数，如果换成bytes，`len()` 函数就计算 **字节数** ：

```python
>>> len(b'ABC')
3
>>> len(b'\xe4\xb8\xad\xe6\x96\x87')
6
>>> len('中文'.encode('utf-8'))
6
```

可见，1个中文字符经过UTF-8编码后通常会占用3个字节，而1个英文字符只占用1个字节。



