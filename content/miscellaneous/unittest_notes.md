---
title: "TDD 与单元测试"
date: 2019-07-26 01:01
tag: development skills
---

[TOC]

## 什么是 TDD

TDD，测试驱动开发(Test Driven Development)，是一种敏捷开发模式，具体指：

- 开发者先编写接口的 **测试用例**
- 然后再编写实现代码通过测试
- 不断增加测试用例、**重构** 代码来改善设计，完成开发

TDD 的优点：从测试角度来驱动，提升代码质量。

No silver bullet. TDD 的缺点也很明显，时间成本增加，降低开发速度。

## 什么是单元测试

软件工程中的测试：

- unit tests
- integration tests
- regression tests
- acceptance tests
- performance tests

单元测试(unit test)是指程序员对代码中的 **单元** 进行正确性检验的测试。

单元的定义可以是一个函数，一组函数，一个类等。

单元测试虽然叫测试，但实际是开发者的行为。

## 为什么要写单测

- 保证代码运行结果符合预期，更早发现问题
- 可重复运行，减少修改代码后出 bug 的风险
- 提高代码的可读性、可维护性，好的单测可以起到程序文档的作用，阅读代码顺序：单测 -> 头文件 -> 源文件

在 TDD 中，我们可以通过单测来优化函数/类/模块的设计。

## 怎么写单测

基本原则

- 针对需求写，而不是针对代码写
- 能够引导出更好的接口设计，以及模块化
- 好的代码应该是易于测试的

### 单测四步走模板

- Setup，初始化，准备数据等
- Execute，调用要测试的方法、函数
- Verify，验证结果是否符合预期
- Teardown，重置状态

```cpp
TEST_F(MyTest, test1) {
  // Setup
  int state = 0;
  Init();
  SetDefaultParams();

  // Execute
  state = GetState();

  // Verify
  EXPECT_EQ(state, 1);

  // Teardown
  ResetAllData();
}
```

这里 `TEST_F` 是 GTest 中的宏，用于测试 Testsuite 类：当一批用例存在很大的相似性时，通过定义一个 Testsuite 类，其中实现公共的数据构造，清理数据的功能，如常见的 SetUp()和 TearDown()函数。

如果需要构造复杂的输入，可以借助 DSL 。

数据驱动：对于流程相同，只是数据不同的测试用例，我们可以把数据参数化，利用测试框架帮我们生成多个测试用例。

### 单测技巧

#### 如何测试 private 函数

在 UT Makefile 中定义：

```
-Dprivate=public -Dprotected=public
```

#### Mock 技术

mocking 有假装、嘲讽的意思。

mock 对象就是可以模拟其他对象行为的一种对象，它可以使单元测试易于编写。

看这样一个例子：

```cpp
class A {
 public:
  int func1() {
    if (func2() > 0) {
      return 1;
    } else {
      return -1;
    }
  }
 protected:
  virtual int func2();
};
```

函数 `func1` 的返回值依赖于 `func2` 的实现，如果我们希望单测能够覆盖到 `func1` 的每一个分支，同时又不想关心 `func2` 的具体实现，应该怎么做呢？这种情况我们可以使用 mock 技术来 mock 掉 `func2` ，比如分别模拟 `func2` 返回值大于 0 、不大于 0 的情况。

##### Stub 技术

针对与上下游之间的网络通信，数据发送/接收等功能做单测时，会用到 stub 技术。可以开发一个简单的桩程序 stub 来代替下游模块，并提供接口用来在 UT 用例中设置 stub 返 回的结果数据。但考虑该方法具有一定开发成本，使用起来不是很方便，加上，网络通信， 数据发送/接收等功能的测试更像是接口，联调类测试，放在单测中进行本身就不是很合适。 故一般不建议采用。

##### 子类化重写

利用虚函数特性进行 mock，缺点是被 mock 的函数必须是 virtual 函数。

```cpp
class MockA1 : public A {
 protected:
  virtual int func2() override {
    return 100;
  }
};

TEST(test_func1, func2_result_positive) {
  MockA1 *a1 = new MockA1();
  EXPECT_EQ(a1->func1, 1);
  delete a1;
}
```

##### HOOK 技术

HOOK 技术不要求被 mock 的函数是虚函数，其通过在运行中动态修改函数地址的方式，达到 mock 的效果。

##### GMock

Google 的 Mock 框架，本质上是基于虚函数重载进行的二次开发。其提供了很多封装好的 mock 宏接口。

详细用法请参考 [Googletest](https://github.com/google/googletest/)。
