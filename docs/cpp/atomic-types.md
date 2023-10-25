---
tip: translate by baidu@2023-10-25 08:20:31
---
---

metaTitle: "C++ | Atomic Types"
description: "Multi-threaded Access"
------------------------------------

# Atomic Types

## Multi-threaded Access

An atomic type can be used to safely read and write to a memory location shared between two threads.

> 原子类型可以用于安全地读取和写入两个线程之间共享的内存位置。

A Bad example that is likely to cause a data race:

```cpp
#include <thread>
#include <iostream>


//function will add all values including and between 'a' and 'b' to 'result'
void add(int a, int b, int * result) {
    for (int i = a; i <= b; i++) {
        *result += i;
    }
}

int main() {
    //a primitive data type has no thread safety
    int shared = 0;

    //create a thread that may run parallel to the 'main' thread
    //the thread will run the function 'add' defined above with paramters a = 1, b = 100, result = &shared
    //analogous to 'add(1,100, &shared);'
    std::thread addingThread(add, 1, 100, &shared);

    //attempt to print the value of 'shared' to console
    //main will keep repeating this until the addingThread becomes joinable
    while (!addingThread.joinable()) {
        //this may cause undefined behavior or print a corrupted value
        //if the addingThread tries to write to 'shared' while the main thread is reading it
        std::cout << shared << std::endl;  
    }


    //rejoin the thread at the end of execution for cleaning purposes
    addingThread.join();
    
    return 0;
}

```

The above example may cause a corrupted read and can lead to undefined behavior.

> 上面的示例可能会导致损坏的读取，并可能导致未定义的行为。

An example with thread safety:

```cpp
#include <atomic>
#include <thread>
#include <iostream>


    //function will add all values including and between 'a' and 'b' to 'result'
void add(int a, int b, std::atomic<int> * result) {
    for (int i = a; i <= b; i++) {
        //atomically add 'i' to result
        result->fetch_add(i);
    }
}

int main() {
    //atomic template used to store non-atomic objects
    std::atomic<int> shared = 0;

    //create a thread that may run parallel to the 'main' thread
    //the thread will run the function 'add' defined above with paramters a = 1, b = 100, result = &shared
    //analogous to 'add(1,100, &shared);'
    std::thread addingThread(add, 1, 10000, &shared);

    //print the value of 'shared' to console
    //main will keep repeating this until the addingThread becomes joinable
    while (!addingThread.joinable()) {
        //safe way to read the value of shared atomically for thread safe read
        std::cout << shared.load() << std::endl;  
    }


    //rejoin the thread at the end of execution for cleaning purposes
    addingThread.join();
    
    return 0;
}

```

The above example is safe because all `store()` and `load()` operations of the `atomic` data type protect the encapsulated `int` from simultaneous access.

> 上面的例子是安全的，因为“atomic”数据类型的所有“store（）”和“load（）”操作都保护封装的“int”不被同时访问。

#### Syntax

- std::atomic<T>
- std::atomic_flag

#### Remarks

`std::atomic` allows atomic access to a TriviallyCopyable type, it is implementation-dependent if this is done via atomic operations or by using locks. The only guaranteed lock-free atomic type is `std::atomic_flag`.
