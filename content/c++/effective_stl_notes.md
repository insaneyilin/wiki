---
title: "Effective STL notes"
date: 2019-12-05 23:23
tag: c++
---

[TOC]

Scott Meyers 的 < Effective STL >，有些东西在 Modern C++ 里已经过时。

---

## 1. 慎重选择容器类型

标准STL序列容器：vector、string、deque、list、forward_list(C++11)、array(C++11)。

标准STL关联容器：set、multiset、map、multimap、unordered_set(C++11)、unordered_multiset(C++11)、unordered_map(C++11)、unordered_multimap(C++11)。

标准的非STL容器，包括：`bitset(include <bitset>)`、`valarray(include <valarray>)`。其它STL容器：`stack(include <stack>)`、`queue(include <queue>)`和 `priority_queue(include <queue>)`。

要考虑应用特点选择容器，比如，当大部分插入和删除操作发生在序列的头部和尾部时， deque是应该考虑使用的数据结构。

STL容器的一种分类方法：连续内存容器(contiguous memory container)和基于节点的容器(node-based container)。

选择容器需要考虑：

- 是否关心容器中的元素是如何排序
- 是否有频繁的插入、删除、查找
- 希望使用哪种迭代器
- 容器中数据的布局是否需要和C兼容？如果需要兼容，就只能选择vector

## 2. 不要试图编写独立于容器类型的代码

STL 设计的一个重要思想是泛化(generalization)。数组被泛化为”以其包含的对象的类型为参数”的容器，函数被泛化为”以其使用的迭代器的类型为参数”的算法，指针被泛化为”以其指向的对象的类型为参数”的迭代器。

但不应该滥用泛化，比如对于关联容器和序列容器，编写泛化的查找函数几乎没有意义。可以把容器隐藏到一个类中，并尽量减少那些通过类接口(而使外部)可见的、与容器相关的信息。

## 3. 确保容器中的对象拷贝正确而高效

当(通过如insert或push_back之类的操作)向容器中加入对象时，存入容器的是你所指定的对象的拷贝。当(通过如front或back之类的操作)从容器中取出一个对象时，你所得到的是容器中所保存的对象的拷贝。进去的是拷贝，出来的也是拷贝(copy in, copy out)。这就是STL的工作方式。

使拷贝动作高效、正确，并防止剥离问题发生的一个简单办法是 **使容器包含指针而不是对象** 。

## 4. 调用 `empty()` 而不是检查 `size()` 是否为0

size() 不一定是常数时间。

## 5. 区间成员函数优先于与之对应的单元素成员函数

```cpp
std::vector<Object> v1;
std::vector<Object> v2;
// ...
v1.assign(v2.begin(), v2.end());  // 区间函数

std::copy(v2.begin(), v2.end(), std::back_inserter(v1));  // 效率不如 assign()

// 效率最差的是一个一个 push_back() ...
```

区间函数能更清楚地表达意图。

## 6. 当心C++编译器烦人的分析机制

C++是较为底层的面相对象语言，在底层的语法规则分析中，有很多隐藏的分析机制。

使用临时命名迭代器来分步完成需求。虽然这样做与标准STL使用有点违背了，但是为了没有二义性和提高代码可读性和方便维护是比较提倡的。

## 7. 如果容器中包含了通过new操作创建的指针，切记在容器对象析构前将指针delete掉

## 8. 切勿创建包含auto_ptr的容器对象

## 9. 慎重选择删除元素的方法

要删除容器中有特定值的所有对象：如果容器是 `vector`, `string` 或 `deque` ，则使用 `erase-remove` 惯用法；如果容器是 `list`，则使用 `list::remove` ；如果容器是一个标准关联容器，则使用它的 `erase` 成员函数。

要删除容器中满足特定判别式(条件)的所有对象：如果容器是 `vector`, `string` 或 `deque`，则使用 `erase-remove_if` 习惯用法；如果容器是 `list` ，则使用 `list::remove_if` ；如果容器是一个标准关联容器，则使用 `remove_copy_if` 和 `swap` ，或者写一个循环来遍历容器中的元素，记住当把迭代器传给 `erase` 时，要对它进行后缀递增。

要在循环内做某些(除了删除对象之外的)操作：如果容器是一个标准序列容器，则写一个循环来遍历容器中的元素，记住每次调用 `erase` 时，要 **用它的返回值更新迭代器** ；如果容器是一个标准关联容器，则写一个循环来遍历容器中的元素，记住当把迭代器传给 `erase` 时，要对迭代器做后缀递增。

## 10. 了解 allocator 的约定和限制

使用 allocator 可以将内存分配、对象构造分离开。

allocator类将new的内存分配、对象构造，视作两个独立的过程，并由独立的函数负责。

参考 [动态内存管理allocator类C++ STL标准模板库vector实现](https://blog.csdn.net/qingdujun/article/details/84727714)。

## 11. 理解自定义 allocator 的合理用法

## 12. 切勿对STL容器的线程安全性有不切实际的依赖

## 13. vector和string优先于动态分配的数组

很多 `string` 实现在背后使用了引用计数技术，这种策略可以消除不必要的内存分配和不必要的字符拷贝，从而可以提供很多应用程序的效率。如果你在多线程环境下使用了引用计数的string，那么注意一下因支持线程安全而导致的性能问题。

## 14. 使用reserve来避免不必要的重新分配

对于vector和string，增长过程是这样来实现的：每当需要更多空间时，就调用与realloc类似的操作。这一类似于realloc的操作分为四部分：

1. 分配一块大小为当前容量的某个倍数的新内存。在大多数实现中，vector和string的容量每次以2的倍数增长，即，每当容器需要扩张时，它们的容量即加倍。
2. 把容器的所有元素从旧的内存拷贝到新的内存中。
3. 析构掉就内存中的对象。
4. 释放旧内存。

`reserve` 能使你 **把重新分配的次数减少到最低限度** ，从而避免了重新分配和指针/迭代器/引用失效带来的开销。避免重新分配的关键在于，尽早地使用reserve，把容器的容量设为足够大的值， **最好是在容器刚被构造出来之后就使用reserve** 。

## 15. 注意string实现的多样性

1. string的值可能会被引用计数，也可能不会。很多实现在默认情况下会使用引用计数，但它们通常提供了关闭默认选择的方法，往往是通过预处理宏来做到这一点。
2. string对象大小的范围可以是一个char*指针大小的1倍到7倍。
3. 创建一个新的字符串值可能需要零次、一次或两次动态分配内存。
4. string对象可能共享，也可能不共享其大小和容量信息。
5. string可能支持，也可能不支持针对单个对象的分配子。
6. 不同的实现对字符内存的最小分配单位有不同的策略。

## 16. 了解如何把vector和string数据传给旧的API

```cpp
void foo(int *arr);

// ...

std::vector<int> v1;
foo(&v1[0]);
foo(v1.data());  // c++ 11
```

## 17. 使用 **swap技巧** 除去多余的容量

对vector或string进行shrink-to-fit操作时，考虑”swap”技巧。C++11中增加了 `shrink_to_fit` 成员函数。

```cpp
std::vector<int> v1;
v1.push_back(1); // 
// ... 增加 v1 大小，然后删除它的大部分元素
std::vector<int>(v1).swap(v1);  // swap 技巧
                                // vector的拷贝构造函数只为所拷贝的元素分配所需要的内存

v1.shrink_to_fit();  // c++ 11

std::vector<int>().swap(v1);  // 清除 v1 并将其容量设为最小
```

## 18. 避免使用 `vector<bool>`

- `vector<bool>` 不是标准容器
- `vector<bool>` 并不是以单个元素为 `bool` 来存储的。

`vector<bool>` 是一个假的容器，它并不真的存储 `bool` ，相反，为了节省空间，它存储的是 `bool` 的紧凑表示。

> `vector<bool>` 是一个奇葩的存在，它的 `[]` 返回的不是 `bool` ，是一个表示单独 `bool` 引用的 `proxy class` ，于是你得到是这个引用（而你平常使用 `bool` 没事，是因为完成了一个隐式转换），而你用 `auto` 的话，想要得到意想中的类型，需要使用 `static_cast<bool>` ，再给 `auto` 。而这也不是特例，其适用于含有 `proxy class` 的 `class` 。

也不是完全不能用 `vector<bool>`，注意上面的坑就好。（把 `vector<bool>` 作为 mask 数组来用完全没问题）

`bitset` 其实是可以看做是一个 `bool` 类型的数组，只不过 `bitset` 的一个元素只占 1 bit 的空间，而 `bool` 里面则是占 1 byte。

使用 `bitset` 可以轻松存10^8个数字并查询，而 `bool arr[10]` 这样的 bool 数组空间是它的八倍。

bitset 的缺点是它的大小是模板参数确定的，必须提前设定，而 `vector<bool>` 可以运行时调整。

## 19. 理解相等(equality)和等价(equivalence)的区别

等价是以“在已排序的区间中对象值的相对顺序”为基础的。

对于关联容器(set multiset map multimap)应该优先选用自己的成员函数而不是与之对应的算法，这样才能保证都是使用的是用于插入的排序函数来进行的相应处理，保证结果的一致性。

在一般情形下，一个关联容器的比较函数并不是operator <， 甚至也不是less，它是用户定义的判别式(predicate)。
每个标准关联容器都通过key_compare成员函数是排序判别式可被外部使用。

一个例子，我们可以自定义一个 key_compare 实现大小写 key 等价。

## 20. 为包含指针的关联容器指定比较类型

如果关联容器的 key 类型为指针，记住自定义比较类型（否则会按地址排序，这大部分情况不是我们需要的）

## 21. 总是让比较函数在等值情况下返回 false

比较函数的返回值表明的是按照该函数定义的排列顺序，一个值是否在另一个之前。相等的值从来不会有前后顺序关系，所以，对于相等的值，比较函数应当始终返回false。

这个地方有个经典的 [`std::sort` core dump 问题](https://www.jianshu.com/p/c5c9e32618db) ，标准库算法里面很多都要求满足 [Strict_weak_orderings](https://en.wikipedia.org/wiki/Weak_ordering#Strict_weak_orderings) ，使用时需要格外注意，避免翻车。

## 22. 切勿直接修改set或multiset中的键

像所有的标准关联容器一样，set和multiset按照一定的顺序来存放自己的元素，而这些容器的正确行为也是建立在其元素保持有序的基础之上的。如果你把关联容器中的一个元素的值改变了(比如把10改为1000)，那么，新的值可能不在正确的位置上，这将会打破容器的有序性。

## 23. 考虑用排序的vector替代关联容器

标准关联容器通常被实现为平衡的二叉查找树。在排序的vector中存储数据可能比在标准关联容器中存储同样的数据要耗费更少的内存，而考虑到页错误的因素，通过二分搜索法来查找一个排序的vector可能比查找一个标准关联容器要更快一些。

注意，如果一个 vector 经常被修改，那使用排序的 vector 来查找不合适。

## 24. 当效率至关重要时，请在 `map::operator[]` 与 `map::insert` 之间谨慎做出选择

结论，插入新元素时，用 `insert()`，访问已有元素再考虑用 `operator[]`。

```cpp
std::map<int, std::string> m;
m.insert(std::map<int, std::string>::value_type(1, "hello"));
```

## 25. 熟悉非标准的哈希容器

std::map对应的数据结构是红黑树。红黑树是一种近似于平衡的二叉查找树，里面的数据是有序的。在红黑树上做查找、插入、删除操作的时间复杂度为O(logN)。而std::unordered_map对应哈希表，哈希表的特点就是查找效率高，时间复杂度为常数级别O(1)， 而额外空间复杂度则要高出许多。所以对于需要高效率查询的情况，使用std::unordered_map容器，但是std::unordered_map对于迭代器遍历效率并不高。而如果对内存大小比较敏感或者数据存储要求有序的话，则可以用std::map容器。

## 26. iterator优先于const_iterator、reverse_iterator以及const_reverse_iterator

STL中的所有标准容器都提供了4中迭代器类型。对容器类container<T>而言，iterator类型的功效相当于T*，而const_iterator则相当于const T*。

## 27. 使用distance和advance将容器的const_iterator转换成iterator

std::distance用以取得两个迭代器(它们指向同一个容器)之间的距离；std::advance则用于将一个迭代器移动指定的距离。

```cpp
std::vector<int> v(10);
auto iter = v.begin();
auto citer = v.cbegin() + 4;
std::advance(iter,
    std::distance<std::vector<int>::const_iterator>(iter, citer));
```

## 28. 正确理解由reverse_iterator的base()成员函数所产生的iterator的用法

注意反向迭代器的删除操作：

```
// 插入，先转为正向迭代器
std::vector<int>::iterator i(ri.base());
v.insert(i, 100);

// 注意删除需要在ri.base()前面的位置上执行
v.erase((++ri).base());
```

## 29. 对于逐个字符的输入请考虑使用istreambuf_iterator

## 30. 确保目标区间足够大

无论何时，如果所使用的算法需要指定一个目标区间，那么必须确保目标区间足够大，或者确保它会随着算法的运行而增大。 **要在算法执行过程中增大目标区间，请使用插入型迭代器** ，比如 `ostream_iterator` 或者由 `back_inserter` 、 `front_inserter` 和 `inserter` 返回的迭代器。

## 31. 了解各种与排序有关的选择

稳定排序：保证排序前 2 个相等的数在序列中的位置顺序和排序后的位置顺序相同。

- `std::sort`：对给定区间所有元素进行排序。
- `std::stable_sort`：对给定区间所有元素进行稳定排序。
- `std::partial_sort`：对给定区间元素进行部分排序。
- `std::nth_element`: 用于排序一个区间，它使得位置n上的元素正好是全排序情况下的第n个元素。当 `nth_element` 返回的时候，所有按全排序规则(即sort的结果)排在位置n之前的元素也都被排在位置n之前，而所有按全排序规则排在位置n之后的元素则都被排在位置n之后。
- `std::partition`: 对序列重排，使得满足条件的元素位于容器最前面。
- `std::stable_partiton`: 稳定版本 partition

## 32. 如果确实需要删除元素，则需要在remove这一类算法之后调用erase

`std::list` 的 `remove` 成员函数是STL中唯一一个名为remove并且确实删除了容器中元素的函数。

```cpp
std::vector<int> v;
v.erase(std::remove(v.begin(), v.end(), 99), v.end()); // 真正删除所有值等于99的元素
```

`std::remove` 并不是唯一一个适用于这种情形的算法，其它还有两个属于”remove类”的算法：`remove_if` 和 `unique`。

## 33. 对包含指针的容器使用remove这一类算法时要特别小心

## 34. 了解哪些算法要求使用排序的区间作为参数

一些算法要求事先排序。

要求排序区间的STL算法：

- binary_search
- lower_bound
- upper_bound
- equal_range
- set_union
- set_intersection
- set_difference
- set_symmetric_difference
- merge
- inplace_merge
- includes

## 35. 通过mismatch或lexicographical_compare实现简单的忽略大小写的字符串比较

mismatch算法是比较两个序列，找出首个不匹配元素的位置。

std::lexicographical_compare是strcmp的一个泛化版本。不过，strcmp只能与字符数组一起工作，而lexicographical_compare则可以与任何类型的值的区间一起工作。而且，strcmp总是通过比较两个字符来判断它们的关系相等、小于还是大于，而lexicographical_compare则可以接受一个判别式，由该判别式来决定两个值是否满足一个用户自定义的准则。

strcmp通常是被优化过的，它们在字符串的处理上一般要比通用算法mismatch和lexicographical_compare快。

## 36. 理解copy_if算法的正确实现

C++11 增加了 copy_if。

## 37. 使用accumulate或者for_each进行区间统计

## 38. 遵循按值传递的原则来设计仿函数(functor)类

STL函数对象是函数指针的一种抽象和建模形式，所以，按照惯例，在STL中，函数对象在函数之间来回传递的时候也是按值传递(即被拷贝)的。标准库中一个最好的证明是for_each算法，它需要一个函数对象作为参数，同时其返回值也是一个函数对象，而且都是按值传递的。

首先，你的函数对象必须尽可能地小，否则拷贝的开销会非常昂贵；其次，函数对象必须是单态的(不是多态的)，也就是说，它们不得使用虚函数。这是因为，如果参数的类型是基类类型，而实参是派生类对象，那么在传递过程中会产生剥离问题(slicing problem)：在对象拷贝过程中，派生部分可能会被去掉，而仅保留了基类部分。

## 39. 确保判别式是”纯函数”

确保判别式是纯函数可以防止一些副作用。

一个判别式(predicate)是一个返回值为bool类型(或者可以隐式地转换为bool类型)的函数。在STL中，判别式有着广泛的用途。标准关联容器的比较函数就是判别式；对于像find_if以及各种与排序有关的算法，判别式往往也被作为参数来传递。

一个纯函数(pure function)是指返回值仅仅依赖于其参数的函数。在C++中，纯函数所能访问的数据应该仅局限参数以及常量(在函数生命期内不会被改变，自然地，这样的常量数据应该被声明为const)。如果一个纯函数需要访问那些可能在两次调用之间发生变化的数据，那么用相同的参数在不同的时刻调用该函数就有可能会得到不同的结果，这将与纯函数的定义相矛盾。

## 40. 若一个类是 functor，则应使它 adaptable

functor adaptor，包括bind、negate、compose以及对一般函数或成员函数的修饰。其价值体现在，通过它们之间的绑定、组合和修饰能力，几乎可以无限制地创造出各种可能的表达式，搭配 STL 算法进行使用。

C++11 直接提供了更强大的 `std::bind`，过去的 `bind1st` 、 `bind2nd` 已经不推荐使用。

## 41. 理解ptr_fun、men_fun和mem_fun_ref的来由

- std::ptr_fun：将函数指针转换为函数对象。
- std::mem_fun：将成员函数转换为函数对象(指针版本)。
- std::mem_fun_ref：将成员函数转换为函数对象(引用版本)。

C++11中已不再推荐使用ptr_fun、mem_fun、mem_fun_ref等相关函数。

## 42. 确保less<T>与operator<具有相同的语义

## 43. 算法调用优先于手写的循环

## 44. 容器的成员函数优先于同名的算法

比如，关联容器提供了count、find、lower_bound、upper_bound和equal_range，而list则提供了remove、remove_if、unique、sort、merge和reverse。

有两个理由：第一，成员函数往往速度快；第二，成员函数通常与容器(特别是关联容器)结合得更加紧密，这是算法所不能比的。

## 45. 正确区分count、find、binary_search、lower_bound、upper_bound和equal_range

都是查找相关的，要考虑容器区间元素是否已排序。

## 46. 考虑使用 functor 而不是函数作为STL算法的参数

## 47. 避免产生“直写型”(write-only)的代码

考虑可读性。

## 48. 总是包含(#include)正确的头文件

## 49. 学会分析与STL相关的编译器诊断信息

## 50. 熟悉与STL相关的Web站点

http://www.cplusplus.com/reference/

https://en.cppreference.com/w/cpp



