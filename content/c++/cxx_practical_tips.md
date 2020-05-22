---
title: "C++ Practical Tips"
date: 2020-01-23 15:23
tag: c++
---

[TOC]

---

## 使用 nullptr

C++11 之后的代码，不要用 NULL 和 0，使用 nullptr。nullptr的类型为nullptr_t，能够隐式的转换为任何指针，防止歧义。

## 头文件不使用 using namespace

头文件里不要用 `using xxx`，防止名字空间污染。

## 变量记得初始化

无论是局部变量还是类的成员变量，初始化都是个好习惯。

## 使用 namespace

尽量不要写不带名字空间的代码。

## 单参数的构造函数，要小心隐式转换（implicit conversions）

用 `explicit` 标记单参数构造函数。

## 尽可能使用 const

const 变量，const 指针，const 成员函数。

## 关于函数返回值的选择

- 局部变量，临时变量，必须 return by value.
- 对于成员变量，使用指针和引用/常引用可以减少性能开销

## 对于基本类型，没必要用常引用参数

基本类型，函数参数直接传值能更好地利用编译器优化。

## 尽可能使用智能指针

C++11 之后尽量避免用 raw pointer 直接获取/释放内存。

## 尽量少用 C-style 数组

std::array 是个不错的选择。

## 合理利用 override 和 final 关键字

让开发者更清楚虚函数的使用情况。

## 尽可能使用标准库完成任务

C++11 后线程使用 std::thread。C++17 文件操作有 std::filesystem。

## 考虑编译效率

使用前向声明（如果头文件里只用了指针/引用）；避免不必要的模板实例化（大量模板会让编译效率降低）；避免无意义的头文件引用。

## 简化代码

- 使用初始化列表
- 尽可能减少临时变量的使用

## 开启移动语义

```cpp
ModelObject(ModelObject &&) = default;
```

## 禁止 shared_ptr 拷贝

shared_ptr 的拷贝涉及到引用计数的原子操作和线程安全，会有额外开销。

## 利用三目运算符和lambda减少拷贝/再赋值

```cpp
// Bad Idea
std::string somevalue;

if (caseA) {
  somevalue = "Value A";
} else {
  somevalue = "Value B";
}

// Better Idea
const std::string somevalue = caseA ? "Value A" : "Value B";
```

```cpp
// Bad Idea
std::string somevalue;

if (caseA) {
  somevalue = "Value A";
} else if(caseB) {
  somevalue = "Value B";
} else {
  somevalue = "Value C";
}

// Better Idea
const std::string somevalue = [&](){
    if (caseA) {
      return "Value A";
    } else if (caseB) {
      return "Value B";
    } else {
      return "Value C";
    }
  }();
```

## 使用 make_shared 和 make_unique

使用智能指针时如果使用 new 来创建对象，会有分配引用计数的额外开销。

## 关注构造函数和析构函数的性能

构造函数/析构函数的性能可能会比较差（继承的情况，会调用父类的构造函数/析构函数；组合的情况，会调用成员对象的构造函数/析构函数），编码时要能够意识到这种“silent execution”的情况。

## 优先使用 unique_ptr 而不是 shared_ptr

unique_ptr 不需要维护引用计数，性能会高一些；对于工厂模式，也建议返回 unique_ptr，有必要时通过类型转换获取 shared_ptr：

```cpp
std::unique_ptr<ModelObject_Impl> factory();

auto shared = std::shared_ptr<ModelObject_Impl>(factory());
```

## 尽量不要使用 std::endl

`std::endl` 相当于 `"\n" << std::flush`，多了一次强制刷新缓冲区。

## 谨慎选择 double 和 float

float 精度低，但向量化计算可能比 double 性能高；频繁的 double - float 转换也会导致性能下降，根据场景取舍。

## Prefer ++i to i++

老生常谈的问题，i++ 会多一次拷贝。

## 注意区分 char 和 string

```cpp
// Bad Idea
std::cout << someThing() << "\n";

// Good Idea
std::cout << someThing() << '\n';
```

前者是 string，于是需要 range check(tail \0)，而后者是单个 char，会省一些开销。

## 不要使用 std::bind

在 C++11 之后，std::bind 实现的功能可以用 lambda 来实现：

```cpp
// Bad Idea
auto f = std::bind(&my_function, "hello", std::placeholders::_1);
f("world");

// Good Idea
auto f = [](const std::string &s) { return my_function("hello", s); };
f("world");
```

## 虚函数的性能开销

虚函数由于要维护虚函数表及相对应的指针，比普通函数会多一些开销（构造函数必须初始化vptr(虚函数表)；虚函数是通过指针间接调用的，所以必须先得到指向虚函数表的指针，然后再获得正确的函数偏移量）；虚函数的额外性能开销比较小，一般可忽略不计，但对于 getter/setter 这种本身就没什么开销的方法要注意是否需要用虚函数。

相比较而言，模板比继承提供了更好的性能，它把对类型的解析提前到编译期间。但使用模板的开发难度增加了。

## 返回值优化

Return Value Optimization (RVO)，属于编译器级别优化，对于返回一个对象的函数，消除临时对象的构造和析构成本。更好的习惯是不返回对象，把要返回的对象作为函数参数（引用或者指针）。或者可以用 Computational Constructor，在构造函数中完成计算。

## inline functions

- Small methods (for example, accessors) should be inlined and large methods should not be inlined. 
- Excessive inlining can drastically increase code size, which can result in increased execution times because of a resulting lower cache hit rate.
- 只会被调用一次的方法最适合用内联函数，因为不会导致 code size 膨胀；
- 递归函数不能用内联

## 使用内存池

频繁地分配和回收内存会严重地降低程序的性能。

通过开发专用的内存管理器可以解决这个问题。对专用内存管理器的设计可以从多个角度考虑，常见的两个角度是大小和并发。

大小：

- 固定大小：分配固定大小内存块的内存管理器。
- 可变大小：分配任意大小内存块的内存管理器。所请求分配的大小事先是未知的。

并发：

- 单线程：内存管理器局限在一个线程内。内存被一个线程使用，并且不越出该线程的界限。这种内存管理器不涉及相互访问的多线程。
- 多线程：内存管理器被多个线程并发地使用。这种实现需要包含互斥执行的代码段。无论什么时候，只能有一个线程在执行一个代码段。

大部分情况下，灵活性会以速度的较低为代价，随着内存管理的功能和灵活性的增强，执行速度将降低。一般来说，只考虑单线程的固定大小内存管理器性能最高。

## 减少分支结构

最快的代码是直线型的代码：没有条件判断，没有循环，没有调用，没有返回。一般来说，程序的关键路径越像一条直线，执行速度就越快。请记住：短小而带有很多分支的代码要比长而没有分支的代码所用的执行时间长。

使用简单计算代替小分支：分支是性能的敌人。

## 提高可扩展性

使用单个锁来保护多个不相关的共享资源，会扩大临界区的范围（临界区指的是一个访问共用资源的程序片段），并造成其它不相关的线程间冲突。

实现可扩展性的技巧是减少或者消除顺序化的代码。下面是一些具体建议：

- 任务分解：将大的任务分为小任务，使线程并发地执行这些小任务。
- 代码移出：临界区应该只包含关键代码，不直接操作共享资源的代码不要放在临界区内。
- 利用缓存：有时通过缓存之前访问过的数据，可以消除对临界区的访问。
- 锁粒度：不要用同样的锁来保护所有资源，除非这些资源是同时更新的。
- 惊群现象：仔细分析你的锁调用的特征。当锁被释放时，是所有的等待线程都被唤醒还是只唤醒一个线程？唤醒所有线程会威胁到应用的可扩展性。
- 使用读/写锁：以读为主的共享数据会从这种锁中获益，使用这种锁，可以消除读者线程之间的竞争。

---

## 参考

[C++ Best Practices](https://lefticus.gitbooks.io/cpp-best-practices)

[C++ Performance Tips](http://www.cs.cmu.edu/~gilpin/c%2B%2B/performance.html)

https://zhuanlan.zhihu.com/p/102419965

https://blog.csdn.net/fengbingchun/article/details/86562213



