---
tip: translate by baidu@2023-10-25 08:30:11
---
---
metaTitle: "C++ | RAII: Resource Acquisition Is Initialization"
description: "Locking, Finally/ScopeExit, ScopeSuccess (c++17), ScopeFail (c++17)"
---

# RAII: Resource Acquisition Is Initialization



## Locking


Bad locking:

```cpp
std::mutex mtx;

void bad_lock_example() {
    mtx.lock();
    try
    {
        foo();
        bar();
        if (baz()) {
            mtx.unlock();   // Have to unlock on each exit point.
            return;
        }
        quux();
        mtx.unlock();       // Normal unlock happens here.
    }
    catch(...) {
        mtx.unlock();       // Must also force unlock in the presence of
        throw;              // exceptions and allow the exception to continue.
    }
}

```


That is the wrong way to implement the locking and unlocking of the mutex. To ensure the correct release of the mutex with `unlock()` requires the programer to make sure that all the flows resulting in the exiting of the function result in a call to `unlock()`. As shown above this is a brittle processes as it requires any maintainers to continue following the pattern manually.

> 这是实现互斥锁的锁定和解锁的错误方法。为了确保使用“unlock（）”正确释放互斥锁，需要程序员确保导致函数退出的所有流都会调用“unlock”（）。如上所示，这是一个脆弱的过程，因为它需要任何维护人员继续手动遵循模式。


Using an appropriately crafted class to implement RAII, the problem is trivial:

> 使用一个精心编制的类来实现RAII，问题很简单：

```cpp
std::mutex mtx;

void good_lock_example() {
    std::lock_guard<std::mutex> lk(mtx);   // constructor locks.
                                           // destructor unlocks. destructor call
                                           // guaranteed by language.
    foo();
    bar();
    if (baz()) {
        return;
    }
    quux();
}

```

`lock_guard` is an extremely simple class template that simply calls `lock()` on its argument in its constructor, keeps a reference to the argument, and calls `unlock()` on the argument in its destructor. That is, when the `lock_guard` goes out of scope, the `mutex` is guaranteed to be unlocked. It doesn't matter if the reason it went out of scope is an exception or an early return - all cases are handled; regardless of the control flow, we have guaranteed that we will unlock correctly.



## Finally/ScopeExit



For cases when we don't want to write special classes to handle some resource, we may write a generic class:

> 对于我们不想写特殊类来处理某些资源的情况，我们可以写一个泛型类：

```cpp
template<typename Function>
class Finally final
{
public:
    explicit Finally(Function f) : f(std::move(f)) {}
    ~Finally() { f(); } // (1) See below

    Finally(const Finally&) = delete;
    Finally(Finally&&) = default;
    Finally& operator =(const Finally&) = delete;
    Finally& operator =(Finally&&) = delete;
private:
    Function f;
};
// Execute the function f when the returned object goes out of scope.
template<typename Function>
auto onExit(Function &&f) { return Finally<std::decay_t<Function>>{std::forward<Function>(f)}; }

```

And its example usage

```cpp
void foo(std::vector<int>& v, int i)
{
    // ...

    v[i] += 42;
    auto autoRollBackChange = onExit([&](){ v[i] -= 42; });

    // ... code as recursive call `foo(v, i + 1)`
}

```


Note (1): Some discussion about destructor definition has to be considered to handle exception:

> 注（1）：为了处理异常，必须考虑对析构函数定义的一些讨论：

- `~Finally() noexcept { f(); }`: `std::terminate` is called in case of exception

- `~Finally() noexcept(noexcept(f())) { f(); }`: terminate() is called only in case of exception during stack unwinding.

> -`~Finally（）noexcept（noexcept（f（）））｛f（）；｝`：terminate（）仅在堆栈展开过程中出现异常时调用。

- `~Finally() noexcept { try { f(); } catch (...) { /* ignore exception (might log it) */} }` No `std::terminate` called, but we cannot handle error (even for non stack unwinding).

> -`~Finally（）noexcept｛try｛f（）；｝catch（…）｛/*忽略异常（可能会记录它）*/｝｝`没有调用'std:：terminate'，但我们无法处理错误（即使是非堆栈展开）。



## ScopeSuccess (c++17)



Thanks to `int std::uncaught_exceptions()`, we can implement action which executes only on success (no thrown exception in scope).

> 多亏了“int std:：uncaught_exceptions（）”，我们可以实现只在成功时执行的操作（范围内没有抛出异常）。
Previously `bool std::uncaught_exception()` just allows to detect if **any** stack unwinding is running.

```cpp
#include <exception>
#include <iostream>

template <typename F>
class ScopeSuccess
{
private:
    F f;
    int uncaughtExceptionCount = std::uncaught_exceptions();
public:
    explicit ScopeSuccess(const F& f) : f(f) {}
    ScopeSuccess(const ScopeSuccess&) = delete;
    ScopeSuccess& operator =(const ScopeSuccess&) = delete;

    // f() might throw, as it can be caught normally.
    ~ScopeSuccess() noexcept(noexcept(f())) {
        if (uncaughtExceptionCount == std::uncaught_exceptions()) {
            f();
        }
    }
};

struct Foo {
    ~Foo() {
        try {
            ScopeSuccess logSuccess{[](){std::cout << "Success 1\n";}};
            // Scope succeeds,
            // even if Foo is destroyed during stack unwinding
            // (so when 0 < std::uncaught_exceptions())
            // (or previously std::uncaught_exception() == true)
        } catch (...) {
        }
        try {
            ScopeSuccess logSuccess{[](){std::cout << "Success 2\n";}};

            throw std::runtime_error("Failed"); // returned value
                                                // of std::uncaught_exceptions increases
        } catch (...) { // returned value of std::uncaught_exceptions decreases
        }
    }

};

int main()
{
    try {
        Foo foo;

        throw std::runtime_error("Failed"); // std::uncaught_exceptions() == 1
    } catch (...) { // std::uncaught_exceptions() == 0
    }
}

```

Output:

```cpp
Success 1

```



## ScopeFail (c++17)



Thanks to `int std::uncaught_exceptions()`, we can implement action which executes only on failure (thrown exception in scope).

> 多亏了“int std:：uncaught_exceptions（）”，我们可以实现只在失败时执行的操作（在作用域中抛出异常）。
Previously `bool std::uncaught_exception()` just allows to detect if **any** stack unwinding is running.

```cpp
#include <exception>
#include <iostream>

template <typename F>
class ScopeFail
{
private:
    F f;
    int uncaughtExceptionCount = std::uncaught_exceptions();
public:
    explicit ScopeFail(const F& f) : f(f) {}
    ScopeFail(const ScopeFail&) = delete;
    ScopeFail& operator =(const ScopeFail&) = delete;

    // f() should not throw, else std::terminate is called.
    ~ScopeFail() {
        if (uncaughtExceptionCount != std::uncaught_exceptions()) {
            f();
        }
    }
};

struct Foo {
    ~Foo() {
        try {
            ScopeFail logFailure{[](){std::cout << "Fail 1\n";}};
            // Scope succeeds,
            // even if Foo is destroyed during stack unwinding
            // (so when 0 < std::uncaught_exceptions())
            // (or previously std::uncaught_exception() == true)
        } catch (...) {
        }
        try {
            ScopeFail logFailure{[](){std::cout << "Failure 2\n";}};

            throw std::runtime_error("Failed"); // returned value
                                                // of std::uncaught_exceptions increases
        } catch (...) { // returned value of std::uncaught_exceptions decreases
        }
    }

};

int main()
{
    try {
        Foo foo;

        throw std::runtime_error("Failed"); // std::uncaught_exceptions() == 1
    } catch (...) { // std::uncaught_exceptions() == 0
    }
}

```

Output:

```cpp
Failure 2

```



#### Remarks



RAII stands for **R**esource **A**cquisition **I**s **I**nitialization. Also occasionally referred to as SBRM (Scope-Based Resource Management) or RRID (Resource Release Is Destruction), RAII is an idiom used to tie resources to object lifetime. In C++, the destructor for an object always runs when an object goes out of scope - we can take advantage of that to tie resource cleanup into object destruction.

> RAII代表**R**资源**A**获取**I**s**I**初始化。RAII有时也被称为SBRM（基于范围的资源管理）或RRID（资源释放即销毁），是一种用于将资源与对象生存期联系起来的习惯用法。在C++中，对象的析构函数总是在对象超出范围时运行——我们可以利用这一点将资源清理与对象销毁联系起来。


Any time you need to acquire some resource (e.g. a lock, a file handle, an allocated buffer) that you will eventually need to release, you should consider using an object to handle that resource management for you. Stack unwinding will happen regardless of exception or early scope exit, so the resource handler object will clean up the resource for you without you having to carefully consider all possible current and future code paths.

> 任何时候，当你需要获取一些最终需要释放的资源（例如锁、文件句柄、分配的缓冲区）时，你都应该考虑使用一个对象来为你处理资源管理。无论异常或早期范围退出，堆栈展开都会发生，因此资源处理程序对象将为您清理资源，而无需仔细考虑所有可能的当前和未来代码路径。


It's worth noting that RAII doesn't completely free the developer of thinking about the lifetime of resources.

> 值得注意的是，RAII并没有完全解放开发者对资源使用寿命的思考。

One case is, obviously, a crash or exit() call, which will prevent destructors from being called. Since the OS will clean up process-local resources like memory after a process ends, this is not a problem in most cases.

> 一种情况显然是崩溃或exit（）调用，它将阻止析构函数被调用。由于操作系统会在进程结束后清理进程本地资源（如内存），因此在大多数情况下这不是问题。

However with system resources (i.e. named pipes, lock files, shared memory) you still need facilities to deal with the case where a process didn't clean up after itself, i.e. on startup test if the lock file is there, if it is, verify the process with the pid actually exists, then act accordingly.

> 然而，对于系统资源（即命名管道、锁定文件、共享内存），您仍然需要一些设施来处理进程本身没有清理的情况，即在启动测试时，如果锁定文件存在，则验证具有pid的进程是否真的存在，然后采取相应行动。


Another situation is when a unix process calls a function from the exec-family, i.e. after a fork-exec to create a new process. Here, the child process will have a full copy of the parents memory (including the RAII objects) but once exec was called, none of the destructors will be called in that process.

> 另一种情况是unix进程调用exec家族的函数，即在fork-exec之后创建新进程。在这里，子进程将拥有父内存（包括RAII对象）的完整副本，但一旦调用exec，该进程中就不会调用任何析构函数。

On the other hand, if a process is forked and neither of the processes call exec, all resources are cleaned up in both processes.

> 另一方面，如果一个进程被分叉，并且两个进程都不调用exec，那么两个进程中的所有资源都会被清理掉。

This is correct only for all resources that were actually duplicated in the fork, but with system resources, both processes will only have a reference to the resource (i.e. the path to a lock file) and will both try to release it individually, potentially causing the other process to fail.

> 这只适用于在fork中实际复制的所有资源，但对于系统资源，两个进程都将只有对资源的引用（即锁定文件的路径），并且都将尝试单独释放它，这可能会导致另一个进程失败。

