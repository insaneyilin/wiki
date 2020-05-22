---
title: "C++ 类型转换"
date: 2020-02-02 12:21
tag: c++
---

[TOC]

---

## static_cast

一般情况下类型之间的转化用static_cast，能完成大部分工作，不保证安全。

## dynamic_cast

针对基类和派生类指针和引用的转换，基类和派生类之间必须要继承关系，是安全的。

在运行时尝试 **将基类指针转化为派生类指针** ，如果失败，会返回 nullptr。

子类到父类的转换是 implicit 的。

## const_cast

用于转换 const/非const 指针和引用。常用于去除 const 属性。

## reinterpret_cast

允许将任何指针类型转为其他指针类型，用来处理无关类型之间的转换；它会产生一个新的值，这个值会有与原始参数（expressoin）有完全相同的比特位。

> reinterpret_cast 运算符并不会改变括号中运算对象的值，而是对该对象从位模式上进行重新解释。

一个例子(处理不透明的数据类型)：

https://stackoverflow.com/questions/573294/when-to-use-reinterpret-cast

> One case when reinterpret_cast is necessary is when interfacing with **opaque data types**. This occurs frequently in vendor APIs over which the programmer has no control. Here's a contrived example where a vendor provides an API for storing and retrieving arbitrary global data:

```cpp
// vendor.hpp
typedef struct _Opaque * VendorGlobalUserData;
void VendorSetUserData(VendorGlobalUserData p);
VendorGlobalUserData VendorGetUserData();


// main.cpp
#include "vendor.hpp"
#include <iostream>
using namespace std;

struct MyUserData {
    MyUserData() : m(42) {}
    int m;
};

int main() {
    MyUserData u;

    // store global data
    VendorGlobalUserData d1;
//  d1 = &u;                                          // compile error
//  d1 = static_cast<VendorGlobalUserData>(&u);       // compile error
    d1 = reinterpret_cast<VendorGlobalUserData>(&u);  // ok
    VendorSetUserData(d1);

    // do other stuff...

    // retrieve global data
    VendorGlobalUserData d2 = VendorGetUserData();
    MyUserData * p = 0;
//  p = d2;                                           // compile error
//  p = static_cast<MyUserData *>(d2);                // compile error
    p = reinterpret_cast<MyUserData *>(d2);           // ok

    if (p) { cout << p->m << endl; }
    return 0;
}
```

---

## 参考

https://zhuanlan.zhihu.com/p/33040213

https://stackoverflow.com/questions/573294/when-to-use-reinterpret-cast
