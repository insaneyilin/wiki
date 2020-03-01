---
title: "< Accelerated C++ > 读书笔记"
date: 2019-07-09 01:01
collection: Technical Books
tag: C++
---

[Accelerated C++中文版](https://book.douban.com/subject/2280545/)


```cpp
std::string::size_type num = str.size();  // 用stl库给出的特定类型，不用纠结 int 还是 unsigned int
std::vector<double>::size_type vec_size = vec.size();
```

学会组织计算和数据：

- 计算模块化
- 数据结构化
- 计算和数据隔离

容器，迭代器

```cpp
std::vector<int>::iterator iter = vec.begin();
std::vector<int>::const_iterator iter = vec.begin();

iter = vec.erase(iter);  // 用迭代器实现删除容器中的元素，比索引更安全
```

思考数据结构合理性，如果不需要 random access（不需要通过索引访问容器内元素），用 list 替代 vector 性能更高。

多用标准库已有的实现：

```cpp
<cctype>
std::isspace()
```

迭代器，算法；很多容器有类似的操作（find、insert等），提取公共接口形成算法。

STL利用公共接口提供一个算法集合：

```cpp
for (vector<string>::const_iterator it = str.begin(); it != str.end(); ++it) {
  ret.push_back(*it);
}

ret.insert(ret.end(), str.begin(), str.end());

std::copy(str.begin(), str.end(), std::back_inserter(ret));
```

copy 泛型算法（generic，不属于特定容器）

back_inserter 迭代器适配器（传入一个容器，返回一个迭代器）

利用泛型算法和迭代器适配器实现 split：

```cpp
std::vector<std::string> split(const std::string &str) {
  typedef std::string::const_iterator iter_t;
  std::vector<std::string> ret;
  iter_t i = str.begin();
  while (i != str.end()) {
    // 忽略前端空格
    i = std::find_if(i, str.end(), [](const char &ch) {
      return !std::isspace(ch);
    });
    // 下一个单词结尾空格
    iter_t j = std::find_if(i, str.end(), [](const char &ch) {
      return std::isspace(ch);
    });
    // 复制 [i, j)
    if (i != str.end()) {
      ret.push_back(std::string(i, j));
    }
    i = j;
  }
  return ret;
}
```

回文字符串判断：

```cpp
bool is_palindrome(const std::string &s) {
  return std::equal(s.begin(), s.end(), s.rbegin());
}
```

map/reduce :

```
std::transform() / std::accumulate()
```

**算法作用于容器的元素，而不是容器**

顺序容器: vector, list 等

关联容器：map 等

---

## 泛型函数

模板函数，模板实例化

数据结构独立性，`find(v.begin(), v.end(), x)` 对比 `v.find(x)` ，前者更通用

用迭代器提高适应性：

```cpp
template <typename Out>
void split(const std::string &str, Out out) {
  typedef std::string::const_iterator iter_t;
  iter_t i = str.begin();
  while (i != str.end()) {
    // 忽略前端空格
    i = std::find_if(i, str.end(), [](const char &ch) {
      return !std::isspace(ch);
    });
    // 下一个单词结尾空格
    iter_t j = std::find_if(i, str.end(), [](const char &ch) {
      return std::isspace(ch);
    });
    // 复制 [i, j)
    if (i != str.end()) {
      *out++ = std::string(i, j);
    }
    i = j;
  }
}

std::list<std::string> word_list;
split(s, std::back_inserter(word_list));
```

三种内存分配方法：局部、静态、动态

---

## allocator

ch11 完整的 Vec 类实现，其中 allocator 部分需要好好理解

频繁使用 new delete 会增加资源开销

allocator<T>

分配一块预备用来存储 T 类型对象的内存块

---

## Handle Class

handle class ，主程序中频繁使用基类、派生类指针，容易出错，通过一个类封装基类指针来隐藏内存分配、释放：

```cpp
class Core {
 public:
  string name();
  virtual double grade();
  // ...
};
class Grad : public Core {
 public:
  virtual double grade() override;
  // ...  
};
class Student_info {
 public:
  Student_info() : ptr_(nullptr) {}
  ~Student_info() {
    delete ptr_;
  }
  void read(string s) {
    if (s == "U") {
      ptr_ = new Core;
    } else {
      ptr_ = new Grad;
    }
  }
  string name() {
    return ptr_->name();
  }
  double grade() {
    return ptr_->grade();
  }
 private:
  Core *ptr_;  // 封装一个基类指针
};
```

使用 handle class 时需要注意 handle class 对象的拷贝，如何确定拷贝的是含有基类/子类实例的对象？

```cpp
class Core {
  friend class Student_info;
 protected:
  virtual Core* clone() const {
    return new Core(*this);
  }
};
class Grad : public Core {
 protected:
  Grad* clone() const {
    return new Grad(*this);
  }
};
Student_info::Student_info(const Student_info& info) : ptr_(nullptr) {
  if (info.ptr_ != nullptr) {
    ptr_ = s.ptr_->clone();
  }
}
Student_info& Student_info::operator=(const Student_info& info) {
  if (&info != this) {
    delete ptr_;
    ptr_ = nullptr;
    if (info.ptr_ != nullptr) {
      ptr_ = info.ptr_.clone();
    }
  }
  return *this;
}
```

注意上面代码的巧妙之处，通过 protected 虚函数 clone 实现实例拷贝，声明 handle class 为友元

通用的 handle class （智能指针雏形）：

- 一个 Handle 对象是指向某个对象的值
- 能够通过 handle 对象进行复制
- 能够判断 handle 对象是否指向另一个对象
- 能够触发多态行为

```cpp
template <typename T>
class Handle {
 public:
   Handle() : p_(nullptr) {}
   Handle(const Handle &that) : p_(nullptr) {
     if (that.p_) {
       p_ = new T(*(that.p_));  // clone
     }
   }
   Handle& operator=(const Handle&);
   ~Handle {
     delete p_;
   }
   Handle(T *p) : p_(p) {}
   operator bool() {
     return p_ != nullptr;
   }
   T& operator&() const;
   T* operator->() const;
 private:
   T *p_;
};
```

上面的 handle 类总是会进行数据拷贝，可以使用引用计数来记录有多少个对象指向底层对象：

```cpp
template <typename T>
class RefHandle {
 public:
   RefHandle() : ref_cnt_(1), p_(nullptr) {}
   RefHandle(T *p) : ref_cnt_(1), p_(p) {}
   RefHandle(const RefHandle &h) : ref_cnt_(h.ref_cnt_), p_(h.p_) {
     ++ref_cnt_;
   }
   RefHandle& operator=(const RefHandle &h) {
     ++*h.ref_cnt_;
     --*this.ref_cnt_;
     if (*this.ref_cnt_ == 0) {
       delete ref_cnt_;
       delete p_;
     }
     ref_cnt_ = h.ref_cnt_;
     p_ = h.p_;
     return *this;
   }
   ~RefHandle() {
     --*this.ref_cnt_;
     if (*this.ref_cnt_ == 0) {
       delete ref_cnt_;
       delete p_;
     }
   }
 private:
   T *p_;
   size_t *ref_cnt_;
};
```

可以增加一个 make_unique() 方法来取消共享数据：

```cpp
void RefHandle::make_unique() {
  if (*ref_cnt_ != 1) {
    --*ref_cnt_;
    ref_cnt_ = new size_t(1);
    p_ = p_ ? new T(*p_) : nullptr;  // clone
  }
}
```

软件工程中的基本思想：所有问题都可以通过引入一个额外的间接层来解决：

```cpp
tempate <typename T>
T* clone(const T* p) {
  return p->clone();
}

// 模板特化
// 对 Vec<char> 类型特化
template <>
Vec<char>* clone(const Vec<char> *p) {
  return new Vec<char>(*p);
}
```

