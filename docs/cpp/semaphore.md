---
tip: translate by baidu@2023-10-25 08:30:57
---
---

metaTitle: "C++ | Semaphore"
description: "Semaphore C++ 11, Semaphore class in action"
----------------------------------------------------------

# Semaphore

Semaphores are not available in C++ as of now, but can easily be implemented with a  mutex and a condition variable.

> 到目前为止，信号量在 C++ 中还不可用，但可以通过互斥和条件变量轻松实现。

This example was taken from:

[C++0x has no semaphores? How to synchronize threads?](http://stackoverflow.com/questions/4792449/c0x-has-no-semaphores-how-to-synchronize-threads)

> [C++0x 没有信号量？如何同步线程？](http://stackoverflow.com/questions/4792449/c0x-has-no-semaphores-how-to-synchronize-threads)

## Semaphore C++ 11

```cpp
#include <mutex>
#include <condition_variable>
        
class Semaphore {
public:
    Semaphore (int count_ = 0)
    : count(count_) 
    {
    }
    
    inline void notify( int tid ) {
        std::unique_lock<std::mutex> lock(mtx);
        count++;
        cout << "thread " << tid <<  " notify" << endl;
        //notify the waiting thread
        cv.notify_one();
    }
    inline void wait( int tid ) {
        std::unique_lock<std::mutex> lock(mtx);
        while(count == 0) {
            cout << "thread " << tid << " wait" << endl;
            //wait on the mutex until notify is called
            cv.wait(lock);
            cout << "thread " << tid << " run" << endl;
        }
        count--;
    }
private:
    std::mutex mtx;
    std::condition_variable cv;
    int count;
};

```

## Semaphore class in action

The following function adds four threads. Three threads compete for the semaphore, which is set to a count of one. A slower thread calls `notify_one()`, allowing one of the  waiting threads to proceed.

> 以下函数添加了四个线程。三个线程争夺信号量，信号量被设置为计数 1。较慢的线程调用“notify_one（）”，允许其中一个等待的线程继续。

The result is that `s1` immediately starts spinning, causing the Semaphore's usage `count` to remain below 1. The other threads wait in turn on the condition variable until notify() is called.

> 结果是“s1”立即开始旋转，导致 Semaphore 的用法“count”保持在 1 以下。其他线程依次等待条件变量，直到调用 notify（）。

```cpp
int main()
{
    Semaphore sem(1);
    
    thread s1([&]() {
        while(true) {
            this_thread::sleep_for(std::chrono::seconds(5));
            sem.wait( 1 );
        }           
    });
    thread s2([&]() {
        while(true){
            sem.wait( 2 );
        }
    });
    thread s3([&]() {
        while(true) {
            this_thread::sleep_for(std::chrono::milliseconds(600));
            sem.wait( 3 );
        }
    });
    thread s4([&]() {
        while(true) {
            this_thread::sleep_for(std::chrono::seconds(5));
            sem.notify( 4 );
        }
    });
    
    
    s1.join();
    s2.join();
    s3.join();
    s4.join();

    ...
    }

```
