---
title: "GDB 调试"
date: 2019-07-22 00:01
---

[TOC]

> GDB（GNU Debugger）是 UNIX 及类 UNIX 系统下的强大调试工具，可以调试 C、C++ 等语言编写的程序。

### 生成 core 文件

```
# 查看 core 文件大小限制
ulimit -c

# 将 core 文件大小限制设为无限（不限制大小）
ulimit -c unlimited
```

### 使用 gdb 查看 core 文件堆栈信息

```
gdb <bin> <corefile>

(gdb) bt
```

`bt` 会显示堆栈信息，一般能够直接定位到 coredump 的具体代码行。

### 多线程程序查看 core 文件错误信息

```
(gdb) bt

(gdb) info threads
```

`info threads` 会列出各个线程信息，找到有 abort 的一行，前面是线程号：

```
(gdb) thread <THREADNUMBER>

(gdb) bt
```

### 查看具体变量值

使用 `bt` 后，可以利用 `f` 和 `p` 查看具体某个栈帧中某个变量的值：

```
(gdb) f <frame_num>
(gdb) p <variable_name>
```

### 编译选项支持调试模式

编译时加上 `-g`

```
gcc -g -o main main.c
```

对于 CMake：

```
cmake -DCMAKE_BUILD_TYPE=Debug <src_dir_path>

# or
set(CMAKE_BUILD_TYPE "Debug")
set(CMAKE_CXX_FLAGS_DEBUG "$ENV{CXXFLAGS} -O0 -Wall -g -ggdb")
set(CMAKE_CXX_FLAGS_RELEASE "$ENV{CXXFLAGS} -O3 -Wall")
```


### gdb 调试可执行程序

```
gdb <bin>

(gdb) r
```

```
gdb --args <bin> [args...]

(gdb) r
```
