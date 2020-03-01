---
title: "C++ Standard IO tips"
date: 2020-02-03 15:21
tag: c++
---

[TOC]

---

## 头文件

```cpp
#include <iostream>  // 标准 IO
#include <iomanip>   // IO 格式控制等
#include <fstream>   // 文件 IO 
```

默认情况下，回车、tab 和空格会被 `cin` 忽略。

## std::getline

`std::getline()` 可以用于获取一行输入。

```cpp
std::string str;
std::getline(std::cin, str);
```

`std::getline()` 的第三个参数可以指定一个停止字符，用来输入下面这种有分隔符格式的文件比较有用：

```
John|male|25|programmer
Jane|female|37|teacher
```

```cpp
std::ifstream ifs("input.txt");
std::string name;
std::string gender;
std::string age;
std::string job;
while (!std::getline(ifs, name, '|').eof()) {
  std::getline(inf, gender, '|');
  std::getline(inf, age, '|');
  std::getline(inf, job);
}
```

## 输出格式控制

```cpp
// 输出前先进行设置
std::cout.setf(std::ios::fixed, std::ios::floatfield);
std::cout.setf(std::ios::showpoint);
std::cout.precision(2);
```

```cpp
// 设置输出宽度
int x = 4;
std::cout << x << std::endl;
//output:  "4"

std::cout << setw(2) << x << std::endl;
//output:  " 4"

std::cout.fill('0');
std::cout << setw(4) << x << std::endl;
//output:  "0004"
```

## 参考

http://www.augustcouncil.com/~tgibson/tutorial/iotips.html
