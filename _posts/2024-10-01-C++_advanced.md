---
title: C++ advanced
tags:
    - basis
---

## operator overloading
在C++中，运算符重载（Operator Overloading）是一种允许程序员为用户自定义的数据类型（通常是类）定义新的操作符行为的特性。这使得用户定义的类型可以像内置类型一样使用运算符。运算符重载的一个常见用途是为类对象定义加法、减法、比较等操作。

基本规则
- 不能创建新的运算符：只能重载已有的运算符。
- 不能改变运算符的优先级和结合性。
- 某些运算符不能被重载：如 .、.*、::、?:、sizeof、typeid、alignof、noexcept 等。
- 运算符重载函数可以是成员函数，也可以是非成员函数：如果是成员函数，左操作数必须是类的对象。
成员函数重载
对于二元运算符，成员函数的第一个参数是隐式的 this 指针：

```c++
class Complex {
public:
    double real, imag;
    
    Complex(double r, double i) : real(r), imag(i) {}

    // 重载加法运算符
    Complex operator+(const Complex& other) const {
        return Complex(real + other.real, imag + other.imag);
    }
};
```
## functor
functor（也称为函数对象）是一个可以像函数一样调用的对象。它通常是通过重载类的 operator() 运算符实现的。

在YAKL中，functors可以用于定义并行计算的内核。使用functors的主要好处是，它们允许将计算逻辑封装在一个对象中，从而可以更灵活地管理状态和行为。以下是YAKL与functor互动的一些关键点：
- 封装计算逻辑：通过functors，你可以将复杂的计算逻辑封装在一个类中。这使得代码更模块化和可重用。
- 状态管理：functors可以拥有成员变量，这意味着你可以在不同的调用之间保持状态。这对于某些需要记住先前计算结果的算法特别有用。
- 内核定义：在YAKL中，functors可以用于定义内核函数。你可以通过重载operator()来实现内核的计算部分。
- 类型安全：使用functors可以提高类型安全性，因为它们是基于类的，并且可以使用模板来实现类型安全的多态行为。
- 与YAKL的结合：在YAKL中，functors可以与其并行执行模型结合使用。通过将functor传递给YAKL的并行执行函数（如parallel_for），你可以在CPU或GPU上高效地执行计算。




## lambda
C++ 中的 lambda 表达式是一种用于定义匿名函数的语法，可以在本地范围内定义和使用。lambda 表达式非常适合用于需要短小的函数对象的场合，例如在算法中传递自定义的操作。lambda 表达式的基本语法如下：

```c++
[capture](parameters) -> return_type {
    // function body
};
```
各个部分的含义如下：
- capture：捕获列表，用于捕获周围作用域中的变量。可以按值（=）或引用（&）捕获变量。也可以指定具体的变量名。
- parameters：参数列表，类似于普通函数的参数列表。
- return_type：返回类型，可以省略，如果编译器可以自动推导出返回类型。
- 函数体：包含函数实际执行的代码。

闭包与捕获
https://blog.csdn.net/u012456479/article/details/101479821

## error handling
## 资源管理
智能指针
- shared_ptr
- unique_ptr
- weak_ptr 解决循环引用问题


## 右值引用 move： 优化不必要的拷贝

## 多进程与多线程
多进程是指同时运行多个进程，每个进程拥有独立的内存空间。多进程通常用于需要隔离内存的场景，如提高程序的安全性或稳定性。
- 创建进程：在C++中，可以使用POSIX的fork()函数（在类Unix系统上）或Windows的CreateProcess()函数来创建新进程。
- 进程间通信（IPC）：由于进程之间不共享内存，必须使用进程间通信机制进行数据交换，如管道（pipes）、消息队列、共享内存、信号量等。
- 优点和缺点：多进程提供更好的隔离和稳定性，但由于每个进程有独立的内存空间，创建和切换进程的开销通常比线程大。

多线程是指在同一个进程内同时运行多个线程。线程是程序执行的最小单位，共享进程的内存空间和资源。
- std::thread：C++11引入了std::thread类来创建和管理线程。可以通过创建std::thread对象并传递可调用对象（如函数、函数对象、lambda表达式）来启动新线程。
- 同步机制：由于线程共享内存，可能会产生竞争条件（race conditions）。C++提供了多种同步机制来避免这些问题，包括std::mutex、std::lock_guard、std::unique_lock、std::condition_variable等。
- 线程局部存储：使用thread_local关键字可以为每个线程创建独立的变量副本，避免使用全局变量时的竞争条件。

线程池是一种优化多线程应用程序性能的技术。它通过创建一组预先初始化的线程来处理任务，从而避免了频繁创建和销毁线程的开销。
- 工作原理：线程池维护一个任务队列和一组工作线程。任务被放入队列中，空闲线程从队列中取出任务进行处理。当任务完成后，线程返回池中等待下一个任务。
- 实现：可以手动实现一个简单的线程池，也可以使用现有的库，如C++17引入的std::async和std::future提供了类似线程池的功能。第三方库如Boost.Asio也提供了线程池的支持。
- 优点：线程池通过重用线程减少了线程创建和销毁的开销，提高了程序的性能和响应速度。
## 锁
- 原子操作atomic<int>
- mutex互斥锁
- shared_mutex共享锁，共享读，独占写
- 自旋锁：适用于锁的持有时间短且线程切换开销较高的场景, use TestAndSet， 争到flag再操作
- RAII lock_guard<mutex> lock(g_mutex);
- memory_order_acquire: 它确保在当前线程中，所有后续的内存读取和写入操作不会被重排到这个操作之前。
- volatile: 变量每次都需地址读取，不能cache
- 无锁CAS ： compare and swap 取内容完成操作后 检测当前是否可处理入（即如自己一开始所见）否则重新取
```c++
class Spinlock {
public:
    Spinlock() : flag{ATOMIC_FLAG_INIT} {}

    void lock() {
        while (flag.test_and_set(std::memory_order_acquire)) {
            // 自旋等待，直到锁被释放
            std::this_thread::yield(); // 提示调度器可以调度其他线程
        }
    }

    void unlock() {
        flag.clear(std::memory_order_release);
    }

private:
    std::atomic_flag flag;
};
```
## 内存分配结构
堆区、栈区、全局/静态存储区、常量存储区、代码区

malloc delete

## 内存池
allocator and boost and tcmalloc
https://zhuanlan.zhihu.com/p/691547646
```c++
class MemoryPool {
public:
    MemoryPool(size_t chunkSize, size_t chunkCount)
        : chunkSize_(chunkSize), chunkCount_(chunkCount) {
        // 预分配内存块
        pool_.resize(chunkSize_ * chunkCount_);
        freeList_.reserve(chunkCount_);

        // 初始化空闲列表
        for (size_t i = 0; i < chunkCount_; ++i) {
            freeList_.push_back(pool_.data() + i * chunkSize_);
        }
    }

    void* allocate() {
        std::lock_guard<std::mutex> lock(mutex_);
        if (freeList_.empty()) {
            throw std::bad_alloc();
        }
        void* chunk = freeList_.back();
        freeList_.pop_back();
        return chunk;
    }

    void deallocate(void* chunk) {
        std::lock_guard<std::mutex> lock(mutex_);
        freeList_.push_back(static_cast<char*>(chunk));
    }

private:
    size_t chunkSize_;
    size_t chunkCount_;
    std::vector<char> pool_;
    std::vector<void*> freeList_;
    std::mutex mutex_;
};

int main() {
    const size_t chunkSize = 64; // 每个块64字节
    const size_t chunkCount = 100; // 总共100个块

    MemoryPool pool(chunkSize, chunkCount);

    // 分配和释放内存示例
    void* ptr = pool.allocate();
    std::cout << "Allocated memory at: " << ptr << std::endl;

    pool.deallocate(ptr);
    std::cout << "Deallocated memory at: " << ptr << std::endl;

    return 0;
}
```