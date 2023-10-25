---
tip: translate by baidu@2023-10-25 08:31:14
---
---

metaTitle: "C++ | Singleton Design Pattern"
description: "Lazy Initialization, Static deinitialization-safe singleton., Subclasses, Thread-safe Singeton"
-------------------------------------------------------------------------------------------------------------

# Singleton Design Pattern

## Lazy Initialization

This example has been lifted from the `Q & A` section here:[http://stackoverflow.com/a/1008289/3807729](http://stackoverflow.com/a/1008289/3807729)

> 此示例摘自此处的“问答”部分：[http://stackoverflow.com/a/1008289/3807729](http://stackoverflow.com/a/1008289/3807729)

See this article for a simple design for a lazy evaluated with guaranteed destruction singleton:<br />

> 请参阅本文，了解一个使用保证销毁单例进行评估的懒惰的简单设计：<br/>

[Can any one provide me a sample of Singleton in c++?](http://stackoverflow.com/questions/270947/can-any-one-provide-me-a-sample-of-singleton-in-c/271104#271104)

> [有人能给我一个 c++ 中 Singleton 的例子吗？](http://stackoverflow.com/questions/270947/can-any-one-provide-me-a-sample-of-singleton-in-c/271104#271104)

**The classic lazy evaluated and correctly destroyed singleton.**

```cpp
class S
{
    public:
        static S& getInstance()
        {
            static S    instance; // Guaranteed to be destroyed.
                                  // Instantiated on first use.
            return instance;
        }
    private:
        S() {};                   // Constructor? (the {} brackets) are needed here.

        // C++ 03
        // ========
        // Dont forget to declare these two. You want to make sure they
        // are unacceptable otherwise you may accidentally get copies of
        // your singleton appearing.
        S(S const&);              // Don't Implement
        void operator=(S const&); // Don't implement

        // C++ 11
        // =======
        // We can use the better technique of deleting the methods
        // we don't want.
    public:
        S(S const&)               = delete;
        void operator=(S const&)  = delete;

        // Note: Scott Meyers mentions in his Effective Modern
        //       C++ book, that deleted functions should generally
        //       be public as it results in better error messages
        //       due to the compilers behavior to check accessibility
        //       before deleted status
};

```

See this article about when to use a singleton: (not often)<br />

[Singleton: How should it be used](http://stackoverflow.com/questions/86582/singleton-how-should-it-be-used)

> [Singleton:应该如何使用它](http://stackoverflow.com/questions/86582/singleton-how-should-it-be-used)

See this two article about initialization order and how to cope:<br />

[Static variables initialisation order](http://stackoverflow.com/questions/211237/c-static-variables-initialisation-order/211307#211307)<br />

> [静态变量初始化顺序](http://stackoverflow.com/questions/211237/c-static-variables-initialisation-order/211307#211307)<br/>

[Finding C++ static initialization order problems](http://stackoverflow.com/questions/335369/finding-c-static-initialization-order-problems/335746#335746)

> [查找 C++ 静态初始化顺序问题](http://stackoverflow.com/questions/335369/finding-c-static-initialization-order-problems/335746#335746)

See this article describing lifetimes:<br />

[What is the lifetime of a static variable in a C++ function?](http://stackoverflow.com/questions/246564/what-is-the-lifetime-of-a-static-variable-in-a-c-function)

> [C++ 函数中静态变量的生存期是多少？](http://stackoverflow.com/questions/246564/what-is-the-lifetime-of-a-static-variable-in-a-c-function)

See this article that discusses some threading implications to singletons:<br />

> 请参阅本文，该文章讨论了 singleton 的一些线程含义：<br/>

[Singleton instance declared as static variable of GetInstance method](http://stackoverflow.com/questions/449436/singleton-instance-declared-as-static-variable-of-getinstance-method/449823#449823)

> [声明为 GetInstance 方法的静态变量的 Singleton 实例](http://stackoverflow.com/questions/449436/singleton-instance-declared-as-static-variable-of-getinstance-method/449823#449823)

See this article that explains why double checked locking will not work on C++:<br />

> 请参阅这篇文章，解释为什么双重检查锁定在 C++ 上不起作用：<br/>

[What are all the common undefined behaviours that a C++ programmer should know about?](http://stackoverflow.com/questions/367633/what-are-all-the-common-undefined-behaviour-that-c-programmer-should-know-about/367690#367690)

> [C++ 程序员应该知道的所有常见的未定义行为是什么？](http://stackoverflow.com/questions/367633/what-are-all-the-common-undefined-behaviour-that-c-programmer-should-know-about/367690#367690)

## Static deinitialization-safe singleton.

There are times with multiple static objects where you need to be able to guarantee that the **singleton** will not be destroyed until all the static objects that use the **singleton** no longer need it.

> 有时，在使用多个静态对象时，您需要能够保证在所有使用 **singleton** 的静态对象不再需要它之前，**singleton***不会被破坏。

In this case `std::shared_ptr` can be used to keep the **singleton** alive for all users even when the static destructors are being called at the end of the program:

> 在这种情况下，即使在程序结束时调用静态析构函数，也可以使用“std:：shared_ptr”为所有用户保持 **singleton** 的活动状态：

```cpp
class Singleton
{
public:
    Singleton(Singleton const&) = delete;
    Singleton& operator=(Singleton const&) = delete;

    static std::shared_ptr<Singleton> instance()
    {
        static std::shared_ptr<Singleton> s{new Singleton};
        return s;
    }

private:
    Singleton() {}
};

```

NOTE: [This example appears as an answer in the Q&A section here.](http://stackoverflow.com/a/40337728/3807729)

> 注：[此示例显示为问答部分的答案。](http://stackoverflow.com/a/40337728/3807729)

## Subclasses

```cpp
class API
{
public:
    static API& instance();
    
    virtual ~API() {}
    
    virtual const char* func1() = 0;
    virtual void func2() = 0;
    
protected:
    API() {}
    API(const API&) = delete;
    API& operator=(const API&) = delete;
};

class WindowsAPI : public API
{
public:
    virtual const char* func1()  override { /* Windows code */ }
    virtual void func2() override { /* Windows code */ }    
};

class LinuxAPI : public API
{
public:
    virtual const char* func1() override { /* Linux code */ }
    virtual void func2() override { /* Linux code */ }    
};

API& API::instance() {
#if PLATFORM == WIN32
    static WindowsAPI instance;
#elif PLATFORM = LINUX
    static LinuxAPI instance;
#endif
    return instance;
}

```

In this example, a simple compiler switch binds the `API` class to the appropriate subclass. In this way, `API` can be accessed without being coupled to platform-specific code.

> 在本例中，一个简单的编译器开关将“API”类绑定到适当的子类。通过这种方式，可以访问“API”，而无需耦合到特定于平台的代码。

## Thread-safe Singeton

The C++11 standards guarantees that the initialization of function scope objects are initialized in a synchronized manner. This can be used to implement a thread-safe singleton with [lazy initialization](http://stackoverflow.com/documentation/c%2B%2B/2713/singleton-design-pattern/9075/lazy-initialization).

> C++11 标准保证函数作用域对象的初始化是以同步的方式初始化的。这可以用于实现具有[惰性初始化]的线程安全单例([http://stackoverflow.com/documentation/c%2B%2B/2713/singleton](http://stackoverflow.com/documentation/c%2B%2B/2713/singleton)-设计模式/9075/惰性初始化）。

```cpp
class Foo
{
public:
    static Foo& instance()
    {
        static Foo inst;
        return inst;
    }        
private:
    Foo() {}
    Foo(const Foo&) = delete;
    Foo& operator =(const Foo&) = delete;
};

```

#### Remarks

A **Singleton** is designed to ensure a class only has one instance **and** provides a global point of access to it. If you only require one instance **or** a convenient global point of access, but not both, consider other options before turning to the singleton.

> **Singleton** 旨在确保一个类只有一个实例**，并且**为其提供全局访问点。如果您只需要一个实例***或**方便的全局访问点，但不能同时需要这两个，请在转向 Singleton 之前考虑其他选项。

Global variables **can** make it harder to reason about code. For example, if one of the calling functions isn't happy with the data it's receiving from a Singleton, you now have to track down what is first giving the singleton bad data in the first place.

> 全局变量**可能会使对代码进行推理变得更加困难。例如，如果其中一个调用函数对它从 Singleton 接收的数据不满意，那么现在必须首先跟踪是什么给了 Singleton 坏数据。

Singletons also encourage [coupling](https://en.wikipedia.org/wiki/Coupling_(computer_programming)), a term used to describe two components of code that are joined together thus reducing each components own measure of self-containment.

> 单身汉也鼓励[结合]([https://en.wikipedia.org/wiki/Coupling](https://en.wikipedia.org/wiki/Coupling)_（computer_programing），一个术语，用于描述代码的两个组成部分连接在一起，从而减少每个组成部分自身的自我包容程度。

Singletons aren't concurrency friendly. When a class has a global point of access, every thread has the ability to access it which can lead to deadlocks and race conditions.

> Singleton 不支持并发。当一个类具有全局访问点时，每个线程都有能力访问它，这可能会导致死锁和竞争条件。

Lastly, lazy initialization can cause performance issues if initialized at the wrong time. Removing lazy initialization also removes some of the features that do make Singleton's interesting in the first place, such as polymorphism (see Subclasses).

> 最后，如果在错误的时间初始化，延迟初始化可能会导致性能问题。删除惰性初始化还删除了一些让 Singleton 有趣的特性，比如多态性（请参见子类）。

Sources: [Game Programming Patterns](http://gameprogrammingpatterns.com/singleton.html) by [Robert Nystrom](https://twitter.com/munificentbob)

> 来源：[游戏编程模式](http://gameprogrammingpatterns.com/singleton.html)作者 [Robert Nystrom](https://twitter.com/munificentbob)
