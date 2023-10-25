---
tip: translate by baidu@2023-10-25 08:34:46
---
---

metaTitle: "C++ | Threading"
description: "Creating a std::thread, Passing a reference to a thread, Using std::async instead of std::thread, Ensuring a thread is always joined, Basic Synchronization, Create a simple thread pool, Operations on the current thread, Using Condition Variables, Thread operations, Thread-local storage, Reassigning thread objects"
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# Threading

## Creating a std::thread

In C++, threads are created using the std::thread class. A thread is a separate flow of execution; it is analogous to having a helper perform one task while you simultaneously perform another. When all the code in the thread is executed, it **terminates**.  When creating a thread, you need to pass something to be executed on it. A few things that you can pass to a thread:

> 在 C++ 中，线程是使用 std:：thread 类创建的。线程是一个单独的执行流；这类似于让一个助手执行一项任务，同时执行另一项任务。当线程中的所有代码都被执行时，它**终止**。创建线程时，需要传递一些要在其上执行的内容。可以传递给线程的一些内容：

- Free functions
- Member functions
- Functor objects
- Lambda expressions

Free function example - executes a function on a separate thread ([Live Example](http://ideone.com/hX1Ygn)):

> 自由函数示例-在单独的线程上执行函数（[Live example](http://ideone.com/hX1Ygn))：

```cpp
#include <iostream>
#include <thread>
 
void foo(int a)
{
    std::cout << a << '\n';
}
 
int main()
{
    // Create and execute the thread
    std::thread thread(foo, 10); // foo is the function to execute, 10 is the
                                 // argument to pass to it
 
    // Keep going; the thread is executed separately
 
    // Wait for the thread to finish; we stay here until it is done
    thread.join();
 
    return 0;
}

```

Member function example - executes a member function on a separate thread ([Live Example](http://ideone.com/4QeG4E)):

> 成员函数示例-在单独的线程上执行成员函数（[Live example](http://ideone.com/4QeG4E))：

```cpp
#include <iostream>
#include <thread>
 
class Bar
{
public:
    void foo(int a)
    {
        std::cout << a << '\n';
    }
};
 
int main()
{
    Bar bar;
    
    // Create and execute the thread
    std::thread thread(&Bar::foo, &bar, 10); // Pass 10 to member function
 
    // The member function will be executed in a separate thread
 
    // Wait for the thread to finish, this is a blocking operation
    thread.join();
 
    return 0;
}

```

Functor object example ([Live Example](http://ideone.com/h2EepE)):

```cpp
#include <iostream>
#include <thread>
 
class Bar
{
public:
    void operator()(int a)
    {
        std::cout << a << '\n';
    }
};
 
int main()
{
    Bar bar;
    
    // Create and execute the thread
    std::thread thread(bar, 10); // Pass 10 to functor object
 
    // The functor object will be executed in a separate thread
 
    // Wait for the thread to finish, this is a blocking operation
    thread.join();
 
    return 0;
}

```

Lambda expression example ([Live Example](http://ideone.com/UacLRf)):

```cpp
#include <iostream>
#include <thread>
 
int main()
{
    auto lambda = [](int a) { std::cout << a << '\n'; };

    // Create and execute the thread
    std::thread thread(lambda, 10); // Pass 10 to the lambda expression
 
    // The lambda expression will be executed in a separate thread
 
    // Wait for the thread to finish, this is a blocking operation
    thread.join();
 
    return 0;
}

```

## Passing a reference to a thread

You cannot pass a reference (or `const` reference) directly to a thread because `std::thread` will copy/move them. Instead, use `std::reference_wrapper`:

> 不能将引用（或“const”引用）直接传递给线程，因为“std:：thread”将复制/移动它们。相反，请使用“std:：reference_wrapper”：

```cpp
void foo(int& b)
{
    b = 10;
}

int a = 1;
std::thread thread{ foo, std::ref(a) }; //'a' is now really passed as reference

thread.join();
std::cout << a << '\n'; //Outputs 10

```

```cpp
void bar(const ComplexObject& co)
{
    co.doCalculations();
}

ComplexObject object;
std::thread thread{ bar, std::cref(object) }; //'object' is passed as const&

thread.join();
std::cout << object.getResult() << '\n'; //Outputs the result

```

## Using std::async instead of std::thread

`std::async` is also able to make threads. Compared to `std::thread` it is considered less powerful but easier to use when you just want to run a function asynchronously.

### Asynchronously calling a function

```cpp
#include <future>
#include <iostream>

unsigned int square(unsigned int i){
    return i*i;
}

int main() {
    auto f = std::async(std::launch::async, square, 8);
    std::cout << "square currently running\n"; //do something while square is running
    std::cout << "result is " << f.get() << '\n'; //getting the result from square
}

```

### Common Pitfalls

<li>
`std::async` returns a `std::future` that holds the return value that will be calculated by the function. When that `future` gets destroyed it waits until the thread completes, making your code effectively single threaded. This is easily overlooked when you don't need the return value:

```cpp
std::async(std::launch::async, square, 5);
//thread already completed at this point, because the returning future got destroyed

```

</li>
<li>
`std::async` works without a launch policy, so `std::async(square, 5);` compiles. When you do that the system gets to decide if it wants to create a thread or not. The idea was that the system chooses to make a thread unless it is already running more threads than it can run efficiently. Unfortunately implementations commonly just choose not to create a thread in that situation, ever, so you need to override that behavior with `std::launch::async` which forces the system to create a thread.
</li>
<li>
Beware of race conditions.
</li>

More on async on [Futures and Promises](http://stackoverflow.com/documentation/c%2B%2B/9840/futures-and-promises#t=201704281532312052997)

> 更多关于〔期货和承诺〕异步([http://stackoverflow.com/documentation/c%2B%2B/9840/futures](http://stackoverflow.com/documentation/c%2B%2B/9840/futures)-并承诺#t=201704281532312052997）

## Ensuring a thread is always joined

When the destructor for `std::thread` is invoked, a call to either `join()` or `detach()` **must** have been made. If a thread has not been joined or detached, then by default `std::terminate` will be called. Using [RAII](http://en.cppreference.com/w/cpp/language/raii), this is generally simple enough to accomplish:

> 当调用“std:：thread”的析构函数时，必须对“join（）”或“detachment（）”**进行调用。如果线程尚未联接或分离，则默认情况下将调用“std:：terminate”。使用 [RAI](http://en.cppreference.com/w/cpp/language/raii)，这通常足够简单，可以实现：

```cpp
class thread_joiner
{
public:

    thread_joiner(std::thread t)
        : t_(std::move(t))
    { }

    ~thread_joiner()
    {
        if(t_.joinable()) {
            t_.join();
        }
    }

private:

    std::thread t_;
}

```

This is then used like so:

```

void perform_work()
 {
     // Perform some work
 }

 void t()
 {
     thread_joiner j{std::thread(perform_work)};
     // Do some other calculations while thread is running
 } // Thread is automatically joined here

```

This also provides exception safety; if we had created our thread normally and the work done in `t()` performing other calculations had thrown an exception, `join()` would never have been called on our thread and our process would have been terminated.

> 这也提供了异常安全性；如果我们正常地创建了线程，并且在执行其他计算的“t（）”中所做的工作引发了异常，那么“join（）”将永远不会在线程上调用，我们的进程也将终止。

## Basic Synchronization

Thread synchronization can be accomplished using mutexes, among other synchronization primitives. There are several mutex types provided by the standard library, but the simplest is `std::mutex`. To lock a mutex, you construct a lock on it. The simplest lock type is `std::lock_guard`:

> 线程同步可以使用互斥以及其他同步原语来实现。标准库提供了几种互斥类型，但最简单的是“std:：互斥”。要锁定互斥锁，可以在它上构造一个锁。最简单的锁类型是“std:：lock_guard”：

```cpp
std::mutex m;
void worker() {
    std::lock_guard<std::mutex> guard(m); // Acquires a lock on the mutex
    // Synchronized code here
} // the mutex is automatically released when guard goes out of scope

```

With `std::lock_guard` the mutex is locked for the whole lifetime of the lock object. In cases where you need to manually control the regions for locking, use `std::unique_lock` instead:

> 使用“std:：lock_guard”，互斥对象将在锁定对象的整个生命周期内被锁定。如果需要手动控制区域进行锁定，请改用“std:：unique_lock”：

```cpp
std::mutex m;
void worker() {
    // by default, constructing a unique_lock from a mutex will lock the mutex
    // by passing the std::defer_lock as a second argument, we
    // can construct the guard in an unlocked state instead and
    // manually lock later.
    std::unique_lock<std::mutex> guard(m, std::defer_lock);
    // the mutex is not locked yet!
    guard.lock();
    // critical section
    guard.unlock();
    // mutex is again released
}

```

More [Thread synchronization structures](http://stackoverflow.com/documentation/c%2B%2B/9794/thread-synchronization-structures#t=201704240523319231466)

> 更多[线程同步结构]([http://stackoverflow.com/documentation/c%2B%2B/9794/thread](http://stackoverflow.com/documentation/c%2B%2B/9794/thread)-同步结构#t=201704240523319231466）

## Create a simple thread pool

C++11 threading primitives are still relatively low level.  They can be used to write a higher level construct, like a thread pool:

> C++11 线程原语仍然是相对较低的级别。它们可以用于编写更高级别的构造，如线程池：

```cpp
struct tasks {
  // the mutex, condition variable and deque form a single
  // thread-safe triggered queue of tasks:
  std::mutex m;
  std::condition_variable v;
  // note that a packaged_task<void> can store a packaged_task<R>:
  std::deque<std::packaged_task<void()>> work;

  // this holds futures representing the worker threads being done:
  std::vector<std::future<void>> finished;

  // queue( lambda ) will enqueue the lambda into the tasks for the threads
  // to use.  A future of the type the lambda returns is given to let you get
  // the result out.
  template<class F, class R=std::result_of_t<F&()>>
  std::future<R> queue(F&& f) {
    // wrap the function object into a packaged task, splitting
    // execution from the return value:
    std::packaged_task<R()> p(std::forward<F>(f));

    auto r=p.get_future(); // get the return value before we hand off the task
    {
      std::unique_lock<std::mutex> l(m);
      work.emplace_back(std::move(p)); // store the task<R()> as a task<void()>
    }
    v.notify_one(); // wake a thread to work on the task

    return r; // return the future result of the task
  }

  // start N threads in the thread pool.
  void start(std::size_t N=1){
    for (std::size_t i = 0; i < N; ++i)
    {
      // each thread is a std::async running this->thread_task():
      finished.push_back(
        std::async(
          std::launch::async,
          [this]{ thread_task(); }
        )
      );
    }
  }
  // abort() cancels all non-started tasks, and tells every working thread
  // stop running, and waits for them to finish up.
  void abort() {
    cancel_pending();
    finish();
  }
  // cancel_pending() merely cancels all non-started tasks:
  void cancel_pending() {
    std::unique_lock<std::mutex> l(m);
    work.clear();
  }
  // finish enques a "stop the thread" message for every thread, then waits for them:
  void finish() {
    {
      std::unique_lock<std::mutex> l(m);
      for(auto&&unused:finished){
        work.push_back({});
      }
    }
    v.notify_all();
    finished.clear();
  }
  ~tasks() {
    finish();
  }
private:
  // the work that a worker thread does:
  void thread_task() {
    while(true){
      // pop a task off the queue:
      std::packaged_task<void()> f;
      {
        // usual thread-safe queue code:
        std::unique_lock<std::mutex> l(m);
        if (work.empty()){
          v.wait(l,[&]{return !work.empty();});
        }
        f = std::move(work.front());
        work.pop_front();
      }
      // if the task is invalid, it means we are asked to abort:
      if (!f.valid()) return;
      // otherwise, run the task:
      f();
    }
  }
};

```

`tasks.queue( []{ return "hello world"s; } )` returns a `std::future<std::string>`, which when the tasks object gets around to running it is populated with `hello world`.

You create threads by running `tasks.start(10)` (which starts 10 threads).

> 您可以通过运行“tasks.start（10）”（启动 10 个线程）来创建线程。

The use of `packaged_task<void()>` is merely because there is no type-erased `std::function` equivalent that stores move-only types.  Writing a custom one of those would probably be faster than using `packaged_task<void()>`.

> 使用“packaged_task＜void（）＞”仅仅是因为没有等价于已擦除类型的“std:：function”来存储仅移动类型。编写其中一个自定义任务可能比使用 `packaged_task<void（）>` 更快。

[Live example](http://coliru.stacked-crooked.com/).

In C++11, replace `result_of_t<blah>` with `typename result_of<blah>::type`.

> 在 C++11 中，将“result_of_t<blah>'”替换为“typename result_of<blah>：type”。

More on [Mutexes](http://stackoverflow.com/documentation/c%2B%2B/9895/mutexes#t=201705101230556157064).

> 有关[Mutexes]的更多信息([http://stackoverflow.com/documentation/c%2B%2B/9895/mutexes#t=201705101230556157064](http://stackoverflow.com/documentation/c%2B%2B/9895/mutexes#t=201705101230556157064))。

## Operations on the current thread

`std::this_thread` is a `namespace` which has functions to do interesting things on the current thread from function it is called from.

| Function      | Description                                               |  |  |  |  |  |  |  |  |
| ------------- | --------------------------------------------------------- | - | - | - | - | - | - | - | - |
| `get_id`      | Returns the id of the thread                              |  |  |  |  |  |  |  |  |
| `sleep_for`   | Sleeps for a specified amount of time                     |  |  |  |  |  |  |  |  |
| `sleep_until` | Sleeps until a specific time                              |  |  |  |  |  |  |  |  |
| `yield`       | Reschedule running threads, giving other threads priority |  |  |  |  |  |  |  |  |

Getting the current threads id using `std::this_thread::get_id`:

```cpp
void foo()
{
    //Print this threads id
    std::cout << std::this_thread::get_id() << '\n';
}

std::thread thread{ foo };
thread.join(); //'threads' id has now been printed, should be something like 12556

foo(); //The id of the main thread is printed, should be something like 2420

```

Sleeping for 3 seconds using `std::this_thread::sleep_for`:

```cpp
void foo()
{
    std::this_thread::sleep_for(std::chrono::seconds(3));
}

std::thread thread{ foo };
foo.join();

std::cout << "Waited for 3 seconds!\n";

```

Sleeping until 3 hours in the future using  `std::this_thread::sleep_until`:

> 使用“std:：this_thread:：sleep_until”睡眠到未来 3 小时：

```cpp
void foo()
{
    std::this_thread::sleep_until(std::chrono::system_clock::now() + std::chrono::hours(3));
}

std::thread thread{ foo };
thread.join();

std::cout << "We are now located 3 hours after the thread has been called\n";

```

Letting other threads take priority using `std::this_thread::yield`:

```cpp
void foo(int a)
{
    for (int i = 0; i < al ++i)
        std::this_thread::yield(); //Now other threads take priority, because this thread
                                   //isn't doing anything important

    std::cout << "Hello World!\n";
}

std::thread thread{ foo, 10 };
thread.join();

```

## Using Condition Variables

A condition variable is a primitive used in conjunction with a mutex to orchestrate communication between threads. While it is neither the exclusive or most efficient way to accomplish this, it can be among the simplest to those familiar with the pattern.

> 条件变量是一个基元，与互斥体一起使用，以协调线程之间的通信。虽然它既不是实现这一点的唯一方法，也不是最有效的方法，但对于熟悉该模式的人来说，它可能是最简单的方法之一。

One waits on a `std::condition_variable` with a `std::unique_lock<std::mutex>`. This allows the code to safely examine shared state before deciding whether or not to proceed with acquisition.

> 一个等待具有“std:：unique_lock＜std:：mutex＞”的“std：：condition_variable”。这允许代码在决定是否继续获取之前安全地检查共享状态。

Below is a producer-consumer sketch that uses `std::thread`, `std::condition_variable`, `std::mutex`, and a few others to make things interesting.

> 下面是生产者-消费者的示意图，它使用“std:：thread”、“std：：condition_variable”、“std：：mutex”和其他一些方法来让事情变得有趣。

```cpp
#include <condition_variable>
#include <cstddef>
#include <iostream>
#include <mutex>
#include <queue>
#include <random>
#include <thread>


int main()
{
    std::condition_variable cond;
    std::mutex mtx;
    std::queue<int> intq;
    bool stopped = false;

    std::thread producer{[&]()
    {
        // Prepare a random number generator.
        // Our producer will simply push random numbers to intq.
        //
        std::default_random_engine gen{};
        std::uniform_int_distribution<int> dist{};

        std::size_t count = 4006;    
        while(count--)
        {    
            // Always lock before changing
            // state guarded by a mutex and
            // condition_variable (a.k.a. "condvar").
            std::lock_guard<std::mutex> L{mtx};

            // Push a random int into the queue
            intq.push(dist(gen));

            // Tell the consumer it has an int
            cond.notify_one();
        }

        // All done.
        // Acquire the lock, set the stopped flag,
        // then inform the consumer.
        std::lock_guard<std::mutex> L{mtx};

        std::cout << "Producer is done!" << std::endl;

        stopped = true;
        cond.notify_one();
    }};

    std::thread consumer{[&]()
    {
        do{
            std::unique_lock<std::mutex> L{mtx};
            cond.wait(L,[&]()
            {
                // Acquire the lock only if
                // we've stopped or the queue
                // isn't empty
                return stopped || ! intq.empty();
            });

            // We own the mutex here; pop the queue
            // until it empties out.

            while( ! intq.empty())
            {
                const auto val = intq.front();
                intq.pop();

                std::cout << "Consumer popped: " << val << std::endl;
            }

            if(stopped){
                // producer has signaled a stop
                std::cout << "Consumer is done!" << std::endl;
                break;
            }

        }while(true);
    }};

    consumer.join();
    producer.join();
    
    std::cout << "Example Completed!" << std::endl;

    return 0;
}

```

## Thread operations

When you start a thread, it will execute until it is finished.

Often, at some point, you need to (possibly - the thread may already be done) wait for the thread to finish, because you want to use the result for example.

> 通常，在某个时刻，您需要（可能-线程可能已经完成）等待线程完成，因为您想使用结果作为示例。

```cpp
int n;
std::thread thread{ calculateSomething, std::ref(n) };

//Doing some other stuff

//We need 'n' now!
//Wait for the thread to finish - if it is not already done
thread.join();

//Now 'n' has the result of the calculation done in the seperate thread
std::cout << n << '\n';

```

You can also `detach` the thread, letting it execute freely:

```cpp
std::thread thread{ doSomething };

//Detaching the thread, we don't need it anymore (for whatever reason)
thread.detach();

//The thread will terminate when it is done, or when the main thread returns

```

## Thread-local storage

Thread-local storage can be created using the `thread_local` [keyword](http://stackoverflow.com/documentation/c%2B%2B/4891/keywords). A variable declared with the `thread_local` specifier is said to have **thread storage duration.**

> 可以使用“Thread_local”[关键字]创建线程本地存储([http://stackoverflow.com/documentation/c%2b%2b/4891/keywords](http://stackoverflow.com/documentation/c%2B%2B/4891/keywords))。用“thread_local”说明符声明的变量称为具有**线程存储持续时间**

- Each thread in a program has its own copy of each thread-local variable.
- A thread-local variable with function (local) scope will be initialized the first time control passes through its definition. Such a variable is implicitly static, unless declared `extern`.

> -具有函数（局部）作用域的线程局部变量将在控件第一次通过其定义时初始化。除非声明为“extern”，否则这样的变量是隐式静态的。

- A thread-local variable with namespace or class (non-local) scope will be initialized as part of thread startup.

> -具有命名空间或类（非本地）作用域的线程本地变量将作为线程启动的一部分进行初始化。

- Thread-local variables are destroyed upon thread termination.
- A member of a class can only be thread-local if it is static. There will therefore be one copy of that variable per thread, rather than one copy per (thread, instance) pair.

> -如果类的成员是静态的，那么它只能是线程本地的。因此，每个线程将有一个该变量的副本，而不是每个（线程，实例）对一个副本。

Example:

```cpp
void debug_counter() {
    thread_local int count = 0;
    Logger::log("This function has been called %d times by this thread", ++count);
}

```

## Reassigning thread objects

We can create empty thread objects and assign work to them later.

If we assign a thread object to another active, `joinable` thread,  `std::terminate` will automatically be called before the thread is replaced.

> 如果我们将一个线程对象分配给另一个活动的“可连接”线程，则在替换该线程之前将自动调用“std:：terminate”。

```cpp
#include <thread>

void foo()
{
    std::this_thread::sleep_for(std::chrono::seconds(3));
}
//create 100 thread objects that do nothing
std::thread executors[100];

// Some code

// I want to create some threads now

for (int i = 0;i < 100;i++)
{
    // If this object doesn't have a thread assigned
    if (!executors[i].joinable())
         executors[i] = std::thread(foo);
}

```

#### Syntax

- thread()
- thread(thread&& other)
- explicit thread(Function&& func, Args&&... args)

#### Parameters

| Parameter | Details                                                            |  |  |  |  |  |  |  |  |
| --------- | ------------------------------------------------------------------ | - | - | - | - | - | - | - | - |
| `other`   | Takes ownership of `other`, `other` doesn't own the thread anymore |  |  |  |  |  |  |  |  |
| `func`    | Function to call in a seperate thread                              |  |  |  |  |  |  |  |  |
| `args`    | Arguments for `func`                                               |  |  |  |  |  |  |  |  |

#### Remarks

Some notes:

- Two `std::thread` objects can **never** represent the same thread.
- A `std::thread` object can be in a state where it doesn't represent **any** thread (i.e. after a move, after calling `join`, etc.).

> -“std:：thread”对象可能处于不表示**任何**线程的状态（即在移动之后、调用“join”之后等）。
