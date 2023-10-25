---
tip: translate by baidu@2023-10-25 08:30:23
---
---
metaTitle: "C++ | Recursive Mutex"
description: "std::recursive_mutex"
---

# Recursive Mutex




## std::recursive_mutex



Recursive mutex allows the same thread to recursively lock a resource - up to an unspecified limit.

> 递归互斥允许同一个线程递归地锁定一个资源，最多锁定一个未指定的限制。


There are very few real-word justifications for this. Certain complex implementations might need to call an overloaded copy of a function without releasing the lock.

> 对此，几乎没有什么真正的理由。某些复杂的实现可能需要在不释放锁的情况下调用函数的重载副本。

```

   std::atomic_int temp{0};
    std::recursive_mutex _mutex;
    
    //launch_deferred launches asynchronous tasks on the same thread id

    auto future1 = std::async(
                std::launch::deferred,
                [&]()
                {
                    std::cout << std::this_thread::get_id() << std::endl;

                    std::this_thread::sleep_for(std::chrono::seconds(3));
                    std::unique_lock<std::recursive_mutex> lock( _mutex);
                    temp=0;
                    
                });

    auto future2 = std::async(
                std::launch::deferred,
                [&]()
                {
                    std::cout << std::this_thread::get_id() << std::endl;
                    while ( true )
                    {
                        std::this_thread::sleep_for(std::chrono::milliseconds(1));
                        std::unique_lock<std::recursive_mutex> lock( _mutex, std::try_to_lock);
                        if ( temp < INT_MAX )
                             temp++;

                        cout << temp << endl;
                        
                    }
                });
    future1.get();
    future2.get();

```

