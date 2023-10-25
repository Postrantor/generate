---
tip: translate by baidu@2023-10-25 08:30:39
---
---
metaTitle: "C++ | Resource Management"
description: "Resource Acquisition Is Initialization, Mutexes & Thread Safety"
---

# Resource Management




## Resource Acquisition Is Initialization



Resource Acquisition Is Initialization (RAII) is a common idiom in resource management. In the case of dynamic memory, it uses [smart pointers](http://stackoverflow.com/documentation/c%2B%2B/509/smart-pointers#t=201612210638370818186) to accomplish resource management. When using RAII, an acquired resource is immediately given ownership to a smart pointer or equivalent resource manager. The resource is only accessed through this manager, so the manager can keep track of various operations. For example, `std::auto_ptr` automatically frees its corresponding resource when it falls out of scope or is otherwise deleted.

> 资源获取即初始化（RAII）是资源管理中的一个常见习语。在动态内存的情况下，它使用[智能指针](http://stackoverflow.com/documentation/c%2B%2B/509/smart-指针#t=201612210638370818186）来实现资源管理。当使用RAII时，所获取的资源立即被赋予智能指针或等效资源管理器的所有权。资源只能通过该管理器访问，因此管理器可以跟踪各种操作。例如，当“std:：auto_ptr”超出作用域或被删除时，它会自动释放相应的资源。

```cpp
#include <memory>
#include <iostream>
using namespace std;

int main() {
    {
        auto_ptr ap(new int(5)); // dynamic memory is the resource
        cout << *ap << endl; // prints 5
    } // auto_ptr is destroyed, its resource is automatically freed
}

```

`std::auto_ptr`'s main problem is that it can't copied without transferring ownership:

```cpp
#include <memory>
#include <iostream>
using namespace std;

int main() {
    auto_ptr ap1(new int(5));
    cout << *ap1 << endl; // prints 5
    auto_ptr ap2(ap1); // copy ap2 from ap1; ownership now transfers to ap2
    cout << *ap2 << endl; // prints 5
    cout << ap1 == nullptr << endl; // prints 1; ap1 has lost ownership of resource
}

```


Because of these weird copy semantics, `std::auto_ptr` can't be used in containers, among other things. The reason it does this is to prevent deleting memory twice: if there are two `auto_ptrs` with ownership of the same resource, they both try to free it when they're destroyed. Freeing an already freed resource can generally cause problems, so it is important to prevent it. However, `std::shared_ptr` has a method to avoid this while not transferring ownership when copying:

> 由于这些奇怪的复制语义，“std:：auto_ptr”不能在容器中使用。它这样做的原因是为了防止两次删除内存：如果有两个“auto_ptr”拥有同一资源的所有权，它们都会在被销毁时尝试释放内存。释放已经释放的资源通常会导致问题，因此防止它很重要。但是，“std:：shared_ptr”有一种方法可以避免这种情况，同时在复制时不转移所有权：

```cpp
#include <memory>
#include <iostream>
using namespace std;

int main() {
    shared_ptr sp2;
    {
        shared_ptr sp1(new int(5)); // give ownership to sp1
        cout << *sp1 << endl; // prints 5
        sp2 = sp1; // copy sp2 from sp1; both have ownership of resource
        cout << *sp1 << endl; // prints 5
        cout << *sp2 << endl; // prints 5
    } // sp1 goes out of scope and is destroyed; sp2 has sole ownership of resource
    cout << *sp2 << endl;        
} // sp2 goes out of scope; nothing has ownership, so resource is freed

```



## Mutexes & Thread Safety



Problems may happen when multiple threads try to access a resource. For a simple example, suppose we have a thread that adds one to a variable. It does this by first reading the variable, adding one to it, then storing it back. Suppose we initialize this variable to 1, then create two instances of this thread. After both threads finish, intuition suggests that this variable should have a value of 3. However, the below table illustrates what might go wrong:

> 当多个线程试图访问一个资源时，可能会出现问题。举个简单的例子，假设我们有一个线程向变量添加一个。它首先读取变量，向其中添加一个变量，然后将其存储回。假设我们将这个变量初始化为1，然后创建这个线程的两个实例。在两个线程都完成后，直觉表明这个变量的值应该为3。但是，下表说明了可能出现的问题：

||Thread 1|Thread 2
|---|---|---|---|---|---|---|---|---|---
|Time Step 1|Read 1 from variable|
|Time Step 2||Read 1 from variable
|Time Step 3|Add 1 plus 1 to get 2|
|Time Step 4||Add 1 plus 1 to get 2
|Time Step 5|Store 2 into variable|
|Time Step 6||Store 2 into variable


As you can see, at the end of the operation, 2 is in the variable, instead of 3. The reason is that Thread 2 read the variable before Thread 1 was done updating it. The solution? Mutexes.

> 正如您所看到的，在操作结束时，变量中有2，而不是3。原因是线程2在线程1更新变量之前读取了该变量。解决方案是什么？Mutex。


A mutex (portmanteau of **mut**ual **ex**clusion) is a resource management object designed to solve this type of problem. When a thread wants to access a resource, it "acquires" the resource's mutex. Once it is done accessing the resource, the thread "releases" the mutex. While the mutex is acquired, all calls to acquire the mutex will not return until the mutex is released. To better understand this, think of a mutex as a waiting line at the supermarket: the threads go into line by trying to acquire the mutex and then waiting for the threads ahead of them to finish up, then using the resource, then stepping out of line by releasing the mutex. There would be complete pandemonium if everybody  tried to access the resource at once.

> 互斥对象（**mut**ual**ex**clusion的组合）是一种资源管理对象，旨在解决这类问题。当线程想要访问资源时，它会“获取”该资源的互斥对象。一旦访问完资源，线程就会“释放”互斥对象。当获取互斥量时，所有获取互斥量的调用都不会返回，直到释放互斥量。为了更好地理解这一点，可以将互斥体想象成超市里的等待队列：线程通过尝试获取互斥体进入队列，然后等待前面的线程完成，然后使用资源，然后通过释放互斥体退出队列。如果每个人都试图同时访问该资源，那将是一片混乱。

`std::mutex` is C++11's implementation of a mutex.

```cpp
#include <thread>
#include <mutex>
#include <iostream>
using namespace std;

void add_1(int& i, const mutex& m) { // function to be run in thread
    m.lock();
    i += 1;
    m.unlock();
}

int main() {
    int var = 1;
    mutex m;

    cout << var << endl; // prints 1
    
    thread t1(add_1, var, m); // create thread with arguments
    thread t2(add_1, var, m); // create another thread
    t1.join(); t2.join(); // wait for both threads to finish
    
    cout << var << endl; // prints 3
}

```

