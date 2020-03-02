---
title: "C++11 多线程与异步调用"
date: 2019-11-17 02:23
tag: c++
---

参考：

https://thispointer.com/category/multithreading/

[TOC]

---

## 使用 C++11 线程

`<thread>` 是 C++11 的线程库，构造函数可以传入 `function pointer` / `functor` / `lambda`：

使用 `std::this_thread::get_id()` 可以获取当前线程的 ID 。

```cpp
#include <iostream>
#include <thread>
  
void thread_func1() {
  auto thread_id = std::this_thread::get_id();
  std::cout << "thread_func1, id: " << thread_id << " " << std::endl;
}

int main(int argc, char **argv) {
  // thread 可以接受 function pointer/functor/lambda
  std::thread t1(thread_func1);
  std::thread t2([]() {
    auto thread_id = std::this_thread::get_id();
    std::cout << "thread func lambda, id: " << thread_id << " " << std::endl;
  });
  std::cout << "main thread, id: " << std::this_thread::get_id() << " " << std::endl;

  t1.join();
  t2.join();

  return 0;
}
```

## 线程的 join 和 detach

`join()` 的含义就是等待线程完成工作，然后 *join* 回到当前线程。

```cpp
int main(int argc, char **argv) {
  std::thread t1(func_ptr);

  // do something (main thread)

  t1.join();  // wait thread t1 to finish its job
}
```

简单来说，`t1.join()` 就是等待 `t1` 线程完成，然后才会继续当前线程的操作。

可以用 `std::this_thread::sleep_for()` 让线程等待一定时间：

```cpp
#include <iostream>
#include <thread>
#include <chrono>

int main(int argc, char **argv) {
  std::cout << "before start thread t1\n";
  std::thread t1([]() {
    int i = 0;
    while (i < 200) {
      ++i;
      // sleep 10 ms per iteration

      std::this_thread::sleep_for(std::chrono::milliseconds(10));
    }   
    std::cout << "t1 finished.\n";
  }); 
  std::cout << "do something in main thread.\n";
  std::cout << "wait t1 to finish.\n";
  // 主线程什么都没做，但是由于调用了 t1.join()，需要等待 t1 线程完成任务
  t1.join();
  std::cout << "done.\n";

  return 0;
}
```

可以一次创建多个线程，用一个 `vector`保存，注意下面 `std::mem_fn` 的作用是将一个类的成员函数包装成一个参数为对象的函数：

```cpp
#include <iostream>                                                                                                                                                                                         
#include <vector>
#include <thread>
#include <chrono>
#include <functional>
#include <algorithm>

int main(int argc, char **argv) {
  auto thread_func = []() {
    std::cout << "thread id: " << std::this_thread::get_id() << std::endl;
  };
  std::vector<std::thread> thread_list;
  for (int i = 0; i < 10; ++i) {
    thread_list.push_back(std::thread(thread_func));
  }
  std::cout << "wait for all the worker thread to finish.\n";
  std::for_each(thread_list.begin(), thread_list.end(),
                std::mem_fn(&std::thread::join));
  std::cout << "main thread finished.\n";
  return 0;
}
```

`detach` 的作用是创建分离线程/后台线程/守护线程：

> Detached threads are also called daemon / Background threads.

```cpp
std::thread t1(func);
t1.detach();  // After calling detach(), std::thread object
              // is no longer associated with the actual thread of execution.
```

- join是阻塞当前线程，并等待 thread object 对应的线程结束，当前线程继续执行
- detach 是将线程从当前线程分离出去，即不受阻塞，操作系统会将其独立对待

使用 `detach()` 函数会让线程在后台运行，即说明 **主线程不会等待子线程运行结束才结束** 。

通常称分离线程为守护线程(daemon threads), UNIX中守护线程是指，没有任何显式的用户接口，并在后台运行的线程。这种线程的特点就是长时间运行；线程的生命周期可能会从某一个应用起始到结束，可能会在后台监视文件系统，还有可能对缓存进行清理，亦或对数据结构进行优化。

看一个例子：

```cpp
#include <iostream>
#include <vector>
#include <thread>
#include <chrono>
#include <functional>
#include <algorithm>

int main(int argc, char **argv) {
  auto thread_func = []() {
    std::cout << "thread id: " << std::this_thread::get_id() << std::endl;
    for (int i = 0; i < 20; ++i) {
      std::this_thread::sleep_for(std::chrono::milliseconds(10));                     
    }
  };
  std::thread t1(thread_func);
  t1.detach();
  std::cout << "main thread finished.\n";
  return 0;
}
// main thread finished.
// thread id: 140529165158144
// 可以看到主线程没有等待 t1 线程完成就结束了
```

要注意，一旦调用 `join()` 或者 `detach()`，那么线程对象就不再与实际线程有关联了，要避免重复调用 `join()` / `detach()` 。

可以用 `joinable()` 判断：

```cpp
std::thread t1(func1);
if(t1.joinable()) {
  std::cout << "Detaching Thread." << std::endl;
  t1.detach();
}

std::thread t2(func2);
if(t2.joinable()) {
  std::cout << "Joining Thread." << std::endl;
  t2.join();
}
```

还有一点要注意，不要忘记调用线程对象的  `join()` / `detach()` 而直接 return:

```cpp
#include <iostream>
#include <vector>
#include <thread>
#include <chrono>
#include <functional>
#include <algorithm>

int main(int argc, char **argv) {
  auto thread_func = []() {
    std::cout << "thread id: " << std::this_thread::get_id() << std::endl;
    for (int i = 0; i < 20; ++i) {
      std::this_thread::sleep_for(std::chrono::milliseconds(10));
    }
  };
  std::thread t1(thread_func);
  // Program will terminate as we have't called either join or detach with the std::thread object.
  // Hence std::thread's object destructor will terminate the program
  std::cout << "main thread finished.\n";
  return 0;
}
```

可以使用 RAII(Resource Acquisition Is Initialization) 对 thread 进行封装：

```cpp
#include <iostream>
#include <thread>

class ThreadRAII {
 public:
  ThreadRAII(std::thread &t) : t_(t) {}
  ~ThreadRAII() {
    if (t_.joinable()) {
      t_.detach();
    }
  }

 private:
  std::thread& t_;
};

int main(int argc, char **argv) {
  std::thread t([]() {
    std::cout << "thread id: " << std::this_thread::get_id() << std::endl;
  });

  ThreadRAII wrapper_t(t);  // If we comment this Line, then program will crash
  return 0;
}

```

## 传递参数给线程

```cpp
#include <iostream>
#include <string>
#include <thread>

void thread_callback(int x, const std::string &str) {
  std::cout << "x: " << x << " str: " << str << std::endl;
}

int main(int argc, char **argv) {
  int x = 0;
  std::string str = "hello, world";

  // directly pass params to std::thread's constructor
  std::thread t1(thread_callback, x, str);
  t1.join();
  return 0;
}
```

注意，小心传递局部变量或者动态内存的地址和指针，不同线程难以保证指针是否有效。对于真正要共享的内容，要记得 **加锁** 。

### 传递引用

默认传参给线程会拷贝一份参数的值到线程，即使声明了引用参数也无法改变外部变量，如何传递引用？可以用 `std::ref()`：

```cpp
#include <iostream>
#include <string>
#include <thread>

void thread_callback(const int &x) {
  int &y = const_cast<int&>(x);  // 强制转换为非 const
  ++y;
  std::cout << "inside thread x: " << x << std::endl;
}

int main(int argc, char **argv) {
  int x = 9;
  std::cout << "main thread, before t1 starts, x: " << x << std::endl;
  std::thread t1(thread_callback, x);
  t1.join();
  std::cout << "main thread, after t1 joins, x: " << x << std::endl;
  return 0;
}

/*
main thread, before t1 starts, x: 9
inside thread x: 10
main thread, after t1 joins, x: 9
*/
// 可以看到，即使声明了引用参数，也没有改变 main thread 中 x 的值
```

使用 `std::ref`：

```cpp
#include <iostream>
#include <string>
#include <thread>

void thread_callback(const int &x) {
  int &y = const_cast<int&>(x);  // 强制转换为非 const
  ++y;
  std::cout << "inside thread x: " << x << std::endl;
}

int main(int argc, char **argv) {
  int x = 9;
  std::cout << "main thread, before t1 starts, x: " << x << std::endl;
  std::thread t1(thread_callback, std::ref(x));
  t1.join();
  std::cout << "main thread, after t1 joins, x: " << x << std::endl;
  return 0;
}

/*
main thread, before t1 starts, x: 9
inside thread x: 10
main thread, after t1 joins, x: 10
*/
```

### 传递成员函数

成员函数传递给线程，注意第一个函数参数要传实例的地址：

```cpp
#include <iostream>
#include <thread>
#include <string>

class A {
 public:
  void say_hello(const std::string &str) {
    std::cout << "hello, " << str << std::endl;
  }
};

int main(int argc, char **argv) {
  A a;
  std::thread t1(&A::say_hello, &a, "stranger");
  t1.join();
  return 0;
}
```

## 使用 Lock 机制处理多线程数据共享

### Race condition

> Race condition is a kind of a bug that occurs in multithreaded applications.

多个线程并发地访问同一个地址时，如果不做一些特殊处理，可能会引发意想不到的结果。

### Lock 机制

解决 race condition 可以使用 Lock 机制。

```cpp
#include <iostream>
#include <thread>
#include <mutex>
#include <vector>
#include <algorithm>
#include <functional>

class Count {
 public:
  // 多线程写操作，要加锁
  void add() {
    mutex_for_x_.lock();
    ++x_;
    mutex_for_x_.unlock();
  }

  void reduce() {
    mutex_for_x_.lock();
    --x_;
    mutex_for_x_.unlock();
  }

  int get_count() {
    return x_;
  }

 private:
  int x_ = 0;
  std::mutex mutex_for_x_;
};

int main(int argc, char **argv) {
  Count count;
  std::vector<std::thread> thread_list;
  for (int i = 0; i < 100; ++i) {
    if (i % 2 == 1) {
      thread_list.push_back(std::thread(&Count::add, &count));
    } else {
      thread_list.push_back(std::thread(&Count::reduce, &count));
    }
  }
  std::for_each(thread_list.begin(), thread_list.end(),
      std::mem_fn(&std::thread::join));
  std::cout << "final count: " << count.get_count() << std::endl;
  return 0;
}
```

或者使用 `std::lock_guard`，实现了对 `mutex` 对象的 RAII：

```cpp
  void add() {
    // lock_guard 对象出作用域会自动 unlock mutex
    std::lock_guard<std::mutex> lock_guard(mutex_for_x_);
    ++x_;
  }
```

## 使用 Condition variable 处理事件响应

假设要实现一个简单的消费者生产者模型，一个线程往队列中放入数据，一个线程从队列中取数据，取数据前需要判断一下队列中确实有数据，由于这个队列是线程间共享的，所以，需要使用互斥锁进行保护，一个线程在往队列添加数据的时候，另一个线程不能取，反之亦然。

`mutex` 可以完成这个任务，但是却存在着性能问题。消费者每次都要等待锁释放。如果生产者处理开销较大，那么会有不必要的等待开销。

更合适的模型是，生产者往队列中添加完数据后，立刻通知消费者干活，如何实现这种“通知”的机制？C++11 提供了 Condition variable 来帮我们实现多线程之间的 signal 机制。

```cpp
#include <iostream>
#include <thread>
#include <functional>
#include <mutex>
#include <condition_variable>

class Application {
 public:
  Application() = default;
  ~Application() = default;

  void load_data() {
    std::cout << "loading data...\n";
    std::this_thread::sleep_for(std::chrono::milliseconds(1000));
    // lock the data
    std::unique_lock<std::mutex> lock(mutex_);
    data_loaded_ = true;
    cond_var_.notify_one();
  }

  void main_proc() {
    std::cout << "main processing...\n";
    std::unique_lock<std::mutex> lock(mutex_);
    // Start waiting for the Condition Variable to get signaled
    // wait() will internally release the lock and make the thread to block
    // as soon as condition variable get signaled, resume the thread and
    // again acquire the lock. Then check if condition is met or not
    // if condition is met then continue else again go in wait.
    cond_var_.wait(lock, [&]() {
      return data_loaded_;
    });
    std::cout << "main processing done.\n";
  }

 private:
  bool data_loaded_ = false;
  std::mutex mutex_;
  std::condition_variable cond_var_;
};

int main(int argc, char **argv) {
  Application app;
  std::thread t1(&Application::load_data, &app);
  std::thread t2(&Application::main_proc, &app);
  t2.join();
  t1.join();
  return 0;
}
```

注意 `cond_var_.wait()` 要和 `std::unique_lock` 配合使用，不能用 `std::lock_guard`，因为 `wait()` 函数会先调用互斥锁的 `unlock()` 函数，然后再将自己睡眠，在被唤醒后，又会继续持有锁，保护后面的队列操作。而 `lock_guard` 没有 `lock` 和 `unlock` 接口，而 `unique_lock` 提供了。这就是必须使用 `unique_lock` 的原因。

关于 `std::condition_variable` 的成员函数：

- `wait()`：It makes the current thread to block until the condition variable get signaled or a spurious wake up happens.
- `notify_one()`：If any threads are waiting on same conditional variable object then  notify_one unblocks one of the waiting threads.
- `notify_all()`：If any threads are waiting on same conditional variable object then  notify_all unblocks all of the waiting threads.

再看一个例子：

```cpp
#include <iostream>
#include <deque>
#include <thread>
#include <mutex>
#include <condition_variable>

std::deque<int> q;
std::mutex mu;
std::condition_variable cond;

void function_1() {
  int count = 10;
  while (count > 0) {
    std::unique_lock<std::mutex> locker(mu);
    q.push_front(count);
    locker.unlock();  // 注意控制锁的粒度，q.push_front() 之后就不需要保护了，可以提前 unlock
    cond.notify_one();  // Notify one waiting thread, if there is one.
    std::this_thread::sleep_for(std::chrono::seconds(1));
    --count;
  }
}

void function_2() {
  int data = 0;
  while ( data != 1) {
    std::unique_lock<std::mutex> locker(mu);
    while(q.empty()) {
      cond.wait(locker); // Unlock mutex and wait to be notified
    }
    data = q.back();
    q.pop_back();
    locker.unlock();
    std::cout << "t2 got a value from t1: " << data << std::endl;
  }
}

int main() {
  std::thread t1(function_1);
  std::thread t2(function_2);
  t1.join();
  t2.join();
  return 0;
}
```

## 使用 std::future 和 std::promise 处理线程的返回值

假设我们要写一个程序，从主线程创建一个线程来压缩一个文件夹，压缩完成后在主线程打印压缩文件名和大小，要求压缩线程返回压缩文件名和文件大小。

第一种方式：通过共享指针实现，向压缩线程传入一个指针变量，利用 mutex，condition variable 完成。

第二种方式：使用 std::future 和 std::promise。

- `std::future` is a class template and its object **stores the future value** .
- `std::promise` is also a class template and its object promises to **set the value in future** . Each `std::promise` object has an associated `std::future` object that will give the value once set by the `std::promise` object.
- A `std::promise` object shares data with its associated std::future object.

如图所示：

![std::promise and std::future](/wiki/attach/images/cxx11_multi_thread_and_async/cxx11_promise_future.png)

```cpp
#include <iostream>
#include <thread>
#include <future>
 
void init(std::promise<int> * promise_obj) {
  std::cout << "Inside Thread" << std::endl;
  promise_obj->set_value(42);
}
 
int main(int argc, char **argv) {
  std::promise<int> promise_obj;
  std::future<int> future_obj = promise_obj.get_future();
  std::thread t(init, &promise_obj);
  std::cout << future_obj.get() << std::endl;  // blocked on the std::future::get() function
  t.join();

  return 0;
}
```

## C++11 异步调用

### std::async

异步操作的主要目的是 **让调用方法的主线程不需要同步等待调用函数，从而可以让主线程继续执行它下面的代码** 。因此异步操作无须额外的线程负担，使用回调的方式进行处理。在设计良好的情况下，处理函数可以不必或者减少使用共享变量，减少了死锁的可能。当需要执行I/O操作时，使用异步操作比使用线程+同步I/O操作更合适。

异步和多线程并不是一个同等关系， **异步是目的，多线程是实现异步的一个手段** 。实现异步可以采用多线程或交给另外的进程来处理。

- `std::future` 可以从异步任务中获取结果，一般与 `std::async` 配合使用， `std::async` 用于创建异步任务，实际上就是创建一个线程执行相应任务。
- `std::async` 就是异步编程的高级封装，封装了 `std::future` 的操作，基本上可以代替 `std::thread` 的所有事情
- `std::async` 的操作，其实相当于封装了 `std::promise` 、 `std::packaged_task` 加上 `std::thread` 。

> `std::async`： `std::async()` is a function template that accepts a callback(i.e. function or function object) as an argument and potentially executes them asynchronously.

`std::async` 的声明：

```cpp
template <class Fn, class... Args>
future<typename result_of<Fn(Args...)>::type> async (launch policy, Fn&& fn, Args&&... args);
```

> `std::async` returns a `std::future<T>`, that stores the value returned by function object executed by `std::async()`. Arguments expected by function can be passed to `std::async()` as arguments after the function pointer argument.
> First argument in `std::async` is launch policy, it control the asynchronous behaviour of `std::async` .

- `std::launch::async` ，调用即创建线程，在另一个 thread 中执行任务
- `std::launch::deferred` ，延迟加载方式创建线程，调用时不创建线程，直到调用 `future` 的 `get` 或者 `wait` 时才创建线程(lazy evaluation)

看一个例子：

```cpp
#include <iostream>
#include <string>
#include <chrono>
#include <thread>
#include <future>

using namespace std::chrono;

std::string fetch_data_from_db(std::string recvd_data) {
  std::this_thread::sleep_for(seconds(5));
  return "DB_" + recvd_data;
}

std::string fetch_data_from_file(std::string recvd_data) {
  std::this_thread::sleep_for(seconds(5));
  return "File_" + recvd_data;
}

int main() {
  system_clock::time_point start = system_clock::now();

  // 异步调用，std::launch::async 策略，会立即创建一个线程工作
  // 如果改成 std::launch::deferred，那么这个例子就看不到使用异步加速的效果了
  std::future<std::string> result_from_db =
      std::async(std::launch::async, fetch_data_from_db, "Data");

  // Fetch Data from File
  std::string file_data = fetch_data_from_file("Data");

  // Fetch Data from DB
  // Will block till data is available in future<std::string> object.
  std::string db_data = result_from_db.get();

  auto end = system_clock::now();
  auto diff = duration_cast < std::chrono::seconds > (end - start).count();
  std::cout << "Total Time Taken = " << diff << " Seconds" << std::endl;
  std::string data = db_data + " :: " + file_data;

  std::cout << "Data = " << data << std::endl;
  return 0;
}

```

### std::packaged_task

`std::promise` 通过 `set_value` 可以使得与之关联的 `std::future` 获取数据。 `std::packaged_task` 则更为强大，它允许传入一个函数，并将函数计算的结果传递给 `std::future`。

`std::packaged_task<>` is a class template and represents a asynchronous task. It encapsulates:

- A callable entity i.e either function, lambda function or function object.
- A shared state that stores the value returned or thrown exception by associated callback.

看一个例子：

```cpp
#include <thread>
#include <future>
#include <iostream>

int sum(int a, int b) {
  return a + b;
}

int main() {
  std::packaged_task<int(int, int)> task(sum);
  std::future<int> future = task.get_future();

  // std::promise 一样，std::packaged_task 支持 move，但不支持拷贝
  std::thread t(std::move(task), 1, 2);
  // 等待异步计算结果
  std::cout << "1 + 2 => " << future.get() << std::endl;

  t.join();
  return 0;
}
```

## 使用 std::future 和 std::promise 实现终止线程功能

我们希望能显式控制一个进程的终止，该如何实现？前面提到过了 `std::promise` 和 `std::future`，只有当 promise 设置值的时候 future 才会真正获取值，那么可以把 promise::set_value 作为一个通知信号来使用，，检查 future 的状态就知道是不是要结束线程了。

```cpp
#include <thread>
#include <iostream>
#include <assert.h>
#include <chrono>
#include <future>

void thread_callback(std::future<void> future_obj) {
  std::cout << "Thread Start" << std::endl;
  // 利用 std::future::wait_for() 来控制循环是否终止
  // 如果相应的 promise object 调用了 set_value()，那么结束循环
  while (future_obj.wait_for(std::chrono::milliseconds(0)) ==
      std::future_status::timeout) {
    std::cout << "Doing Some Work" << std::endl;
    std::this_thread::sleep_for(std::chrono::milliseconds(1000));
  }
  std::cout << "Thread End" << std::endl;
}

int main(int argc, char **argv) {
  // Create a std::promise object
  std::promise<void> exit_signal;

  // Fetch std::future object associated with promise
  std::future<void> future_obj = exit_signal.get_future();

  // Starting Thread & move the future object in lambda function by reference
  std::thread t(&thread_callback, std::move(future_obj));

  // Wait for 10 sec
  std::this_thread::sleep_for(std::chrono::seconds(10));
  std::cout << "Asking Thread to Stop" << std::endl;

  // Set the value in promise
  exit_signal.set_value();

  // Wait for thread to join
  t.join();
  std::cout << "Exiting Main Function" << std::endl;

  return 0;
}
```

使用面向对象封装一个 Stoppable 类：

```cpp
#include <thread>
#include <iostream>
#include <assert.h>
#include <chrono>
#include <future>

class Stoppable {
 public:
  Stoppable() : future_obj_(exit_signal_.get_future()) {}
  Stoppable(Stoppable &&obj) : exit_signal_(std::move(obj.exit_signal_)),
      future_obj_(std::move(obj.future_obj_)) {}
  virtual ~Stoppable() = default;

  Stoppable& operator=(Stoppable &&obj) {
    exit_signal_ = std::move(obj.exit_signal_);
    future_obj_ = std::move(obj.future_obj_);
    return *this;
  }

  virtual void run() = 0;
  void operator()() {
    run();
  }

  bool is_stop_requested() {
    if (future_obj_.wait_for(std::chrono::milliseconds(0)) ==
        std::future_status::timeout) {
      return false;
    }
    return true;
  }

  void stop() {
    exit_signal_.set_value();
  }

 private:
  std::promise<void> exit_signal_;
  std::future<void> future_obj_;
};

class MyTask : public Stoppable {
 public:
  void run() override {
    std::cout << "Task Start" << std::endl;
    while (!is_stop_requested()) {
      std::cout << "Doing Some Work" << std::endl;
      std::this_thread::sleep_for(std::chrono::milliseconds(1000));
    }
    std::cout << "Task End" << std::endl;
  }
};

int main(int argc, char **argv) {
  MyTask task;
  std::thread t([&]() {
    task.run();
  });

  std::this_thread::sleep_for(std::chrono::seconds(10));
  std::cout << "Asking Task to Stop" << std::endl;
  task.stop();
  t.join();

  std::cout << "Thread Joined" << std::endl;
  std::cout << "Exiting Main Function" << std::endl;

  return 0;
}
```
