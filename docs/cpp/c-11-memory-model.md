---
tip: translate by baidu@2023-10-25 08:21:21
---
---

metaTitle: "C++ | C++11 Memory Model"
description: "Need for Memory Model, Fence example"
---------------------------------------------------

# C++11 Memory Model

## Need for Memory Model

```cpp
int x, y;
bool ready = false;

void init()
{
  x = 2;
  y = 3;
  ready = true;
}
void use()
{
  if (ready)
    std::cout << x + y;
}

```

One thread calls the `init()` function while another thread (or signal handler) calls the `use()` function. One might expect that the `use()` function will either print `5` or do nothing. This may not always be the case for several reasons:

> 一个线程调用“init（）”函数，而另一个线程（或信号处理程序）调用“use（）”功能。人们可能会期望“use（）”函数要么打印“5”，要么什么也不做。由于以下几个原因，情况可能并不总是如此：

<li>

The CPU may reorder the writes that happen in `init()` so that the code that  actually executes might look like:

> CPU 可能会对“init（）”中发生的写入进行重新排序，以便实际执行的代码看起来像：

```cpp
void init()
{
  ready = true;
  x = 2;
  y = 3;
}

```

</li>
<li>

The CPU may reorder the reads that happen in `use()` so that the actually executed code might become:

> CPU 可以对“use（）”中发生的读取进行重新排序，以便实际执行的代码可能变为：

```cpp
void use()
{
  int local_x = x;
  int local_y = y;
  if (ready)
    std::cout << local_x + local_y;
}

```

</li>
<li>

An optimizing C++ compiler may decide to reorder the program in similar way.

> 优化的 C++ 编译器可以决定以类似的方式重新排序程序。

</li>

Such reordering cannot change the behavior of a program running in single thread because a thread cannot interleave the calls to `init()` and `use()`. On the other hand in a multi-threaded setting one thread may see part of the writes performed by the other thread where it may happen that `use()` may see `ready==true` and garbage in `x` or `y` or both.

> 这种重新排序不能改变在单线程中运行的程序的行为，因为线程不能交错调用“init（）”和“use（）”。另一方面，在多线程设置中，一个线程可能会看到另一个线程执行的写入的一部分，其中“use（）”可能会在“x”或“y”或两者中看到“ready==true”和垃圾。

The C++ Memory Model allows the programmer to specify which reordering operations are permitted and which are not, so that a multi-threaded program would also be able to behave as expected. The example above can be rewritten in thread-safe way like this:

> C++ 内存模型允许程序员指定哪些重新排序操作是允许的，哪些不允许，这样多线程程序也可以按预期运行。上面的例子可以用线程安全的方式重写，如下所示：

```cpp
int x, y;
std::atomic<bool> ready{false};

void init()
{
  x = 2;
  y = 3;
  ready.store(true, std::memory_order_release);
}
void use()
{
  if (ready.load(std::memory_order_acquire))
    std::cout << x + y;
}

```

Here `init()` performs **atomic store-release** operation. This not only stores the value `true` into `ready`, but also tells the compiler that it cannot move this operation before write operations that are **sequenced before** it.

> 这里 `init（）` 执行**原子存储释放**操作。这不仅将值“true”存储到“ready”中，而且还告诉编译器，它不能在**排序在**之前的写操作之前移动此操作。

The `use()` function does an **atomic load-acquire** operation. It reads the current value of `ready` and also forbids the compiler from placing read operations that are **sequenced after** it to **happen before** the **atomic load-acquire**.

> “use（）”函数执行**原子加载获取**操作。它读取“ready”的当前值，并且还禁止编译器将顺序为**的读取操作放在**之后到**发生在**原子负载获取**之前。

These atomic operations also cause the compiler to put whatever hardware instructions are needed to inform the CPU to refrain from the unwanted reorderings.

> 这些原子操作还导致编译器放入所需的任何硬件指令，以通知 CPU 避免不必要的重新排序。

Because the **atomic store-release** is to the same memory location as the **atomic load-acquire**, the memory model stipulates that if the **load-acquire** operation sees the value written by the **store-release** operation, then all writes performed by `init()`'s thread prior to that **store-release** will be visible to loads that `use()`'s thread executes after its **load-acquire**. That is if `use()` sees `ready==true`, then it is guaranteed to see `x==2` and `y==3`.

> 由于**原子存储释放**与**原子加载获取**位于相同的内存位置，内存模型规定，如果**加载获取**操作看到**存储释放**操作所写的值，则“init（）”线程在**存储释放***之前执行的所有写操作都将对“use（）”的线程在其**加载获取***之后执行的加载可见。也就是说，如果“use（）”看到“ready==true”，则保证看到“x==2”和“y==3”。

Note that the compiler and the CPU are still allowed to write to `y` before writing to `x`, and similarly the reads from these variables in `use()` can happen in any order.

> 请注意，编译器和 CPU 在写入“x”之前仍然可以写入“y”，类似地，从“use（）”中的这些变量读取可以按任何顺序进行。

## Fence example

The example above can also be implemented with fences and relaxed atomic operations:

> 上面的例子也可以用栅栏和放松的原子操作来实现：

```cpp
int x, y;
std::atomic<bool> ready{false};

void init()
{
  x = 2;
  y = 3;
  atomic_thread_fence(std::memory_order_release);
  ready.store(true, std::memory_order_relaxed);
}
void use()
{
  if (ready.load(std::memory_order_relaxed))
  {
    atomic_thread_fence(std::memory_order_acquire);
    std::cout << x + y;
  }
}

```

If the atomic load operation sees the value written by the atomic store then the store happens before the load, and so do the fences: the release fence happens before the acquire fence making the writes to `x` and `y` that precede the release fence to become visible to the `std::cout` statement that follows the acquire fence.

> 如果原子加载操作看到原子存储写入的值，则存储发生在加载之前，栅栏也是如此：释放栅栏发生在获取栅栏之前，使得对释放栅栏之前的“x”和“y”的写入对获取栅栏之后的“std:：cout”语句可见。

A fence might be beneficial if it can reduce the overall number of acquire, release or other synchronization operations. For example:

> 如果围栏可以减少获取、释放或其他同步操作的总数，那么它可能是有益的。例如：

```cpp
void block_and_use()
{
  while (!ready.load(std::memory_order_relaxed))
    ;
  atomic_thread_fence(std::memory_order_acquire);
  std::cout << x + y;
}

```

The `block_and_use()` function spins until the `ready` flag is set with the help of relaxed atomic load. Then a single acquire fence is used to provide the needed memory ordering.

> “block_and_use（）”函数旋转，直到在放松的原子负载的帮助下设置“就绪”标志。然后，使用单个获取围栏来提供所需的内存排序。

#### Remarks

Different threads trying to access the same memory location participate in a **data race** if at least one of the operations is a modification (also known as **store operation**). These **data races** cause **undefined behavior**. To avoid them one needs to prevent these threads from concurrently executing such conflicting operations.

> 如果至少有一个操作是修改操作（也称为**存储操作**），则尝试访问同一内存位置的不同线程将参与**数据竞赛**。这些**数据竞赛**导致**未定义的行为**。为了避免它们，需要防止这些线程同时执行这种冲突的操作。

Synchronization primitives (mutex, critical section and the like) can guard such accesses. The Memory Model introduced in C++11 defines two new portable ways to synchronize access to memory in multi-threaded environment: atomic operations and fences.

> 同步原语（互斥、关键部分等）可以保护此类访问。C++11 中引入的内存模型定义了两种新的可移植方式来同步多线程环境中对内存的访问：原子操作和围栏。

### Atomic Operations

It is now possible to read and write to given memory location by the use of **atomic load** and **atomic store** operations. For convenience these are wrapped in the `std::atomic<t>` template class. This class wraps a value of type `t` but this time **loads** and **stores** to the object are atomic.

> 现在可以通过使用**原子加载**和**原子存储**操作来读取和写入给定的内存位置。为了方便起见，这些都封装在“std:：atomic＜t＞”模板类中。此类包装类型为“t”的值，但这一次**加载**和**存储**到对象是原子的。

The template is not available for all types. Which types are available is implementation specific, but this usually includes most (or all) available integral types as well as pointer types. So that `std::atomic<unsigned>` and `std::atomic<std::vector<foo> *>` should be available, while `std::atomic<std::pair<bool,char>>` most probably wont be.

> 该模板并非适用于所有类型。可用的类型是特定于实现的，但这通常包括大多数（或所有）可用的积分类型以及指针类型。因此，“std:：atomic＜unsigned＞”和“std::：atonic＜std:：vector＜foo＞*＞”应该可用，而“std:∶atomic<std:：pair＜bool，char＞”很可能不会可用。

Atomic operations have the following properties:

- All atomic operations can be performed concurrently from multiple threads without causing undefined behavior.

> -所有原子操作都可以从多个线程并发执行，而不会导致未定义的行为。

- An **atomic load** will see either the initial value which the atomic object was constructed with, or the value written to it via some **atomic store** operation.

> -**原子加载**将看到构造原子对象时使用的初始值，或者通过某个**原子存储**操作写入的值。

- **Atomic stores** to the same atomic object are ordered the same in all threads. If a thread has already seen the value of some **atomic store** operation, subsequent **atomic load** operations will see either the same value, or the value stored by subsequent **atomic store** operation.

> -**同一原子对象的原子存储**在所有线程中的顺序相同。如果线程已经看到某个**原子存储**操作的值，那么后续的**原子加载**操作将看到相同的值，或者后续的**核存储**操作存储的值。

- **Atomic read-modify-write** operations allow **atomic load** and **atomic store** to happen without other **atomic store** in between. For example one can atomically increment a counter from multiple threads, and no increment will be lost regardless of the contention between the threads.

> -**原子读取-修改-写入**操作允许**原子加载**和**原子存储**发生，而其间没有其他**原子存储***。例如，可以从多个线程中原子地递增计数器，并且无论线程之间的争用如何，都不会丢失任何增量。

- Atomic operations receive an optional `std::memory_order` parameter which defines what additional properties the operation has regarding other memory locations.

> -原子操作接收一个可选的“std:：memory_order”参数，该参数定义了操作对其他内存位置的附加属性。

| std::memory_order                                          | Meaning                                                                                                                                                                  |  |  |  |  |  |  |  |  |
| ---------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | - | - | - | - | - | - | - | - |
| `std::memory_order_relaxed`                                | no additional restrictions                                                                                                                                               |  |  |  |  |  |  |  |  |
| `std::memory_order_release` → `std::memory_order_acquire` | if `load-acquire` sees the value stored by `store-release` then stores **sequenced before** the `store-release` happen before loads sequenced after the **load acquire** |  |  |  |  |  |  |  |  |
| `std::memory_order_consume`                                | like `memory_order_acquire` but only for dependent loads                                                                                                                 |  |  |  |  |  |  |  |  |
| `std::memory_order_acq_rel`                                | combines `load-acquire` and `store-release`                                                                                                                              |  |  |  |  |  |  |  |  |
| `std::memory_order_seq_cst`                                | sequential consistency                                                                                                                                                   |  |  |  |  |  |  |  |  |

These memory order tags allow three different memory ordering disciplines: **sequential consistency**, **relaxed**, and **release-acquire** with its sibling **release-consume**.

> 这些内存顺序标签允许三种不同的内存顺序规则：**顺序一致性**、**放松**和**释放获取**及其兄弟**释放消耗**。

### Sequential Consistency

If no memory order is specified for an atomic operation, the order defaults to **sequential consistency**. This mode can also be explicitly selected by tagging the operation with `std::memory_order_seq_cst`.

> 如果没有为原子操作指定内存顺序，则顺序默认为**顺序一致性**。也可以通过用“std:：memory_order_seq_cst”标记操作来显式选择此模式。

With this order no memory operation can cross the atomic operation. All memory operations sequenced before the atomic operation happen before the atomic operation and the atomic operation happens before all memory operations that are sequenced after it. This mode is probably the easiest one to reason about but it also leads to the greatest penalty to performance. It also prevents all compiler optimizations that might otherwise try to reorder operations past the atomic operation.

> 有了这个顺序，任何内存操作都不能跨越原子操作。原子操作之前排序的所有内存操作都发生在原子操作之前，原子操作发生在其后排序的所有存储器操作之前。这种模式可能是最容易推理的模式，但它也会对性能造成最大的损失。它还可以防止所有编译器优化，否则这些优化可能会试图重新排序原子操作之后的操作。

### Relaxed Ordering

The opposite to **sequential consistency** is the **relaxed** memory ordering. It is selected with the `std::memory_order_relaxed` tag. Relaxed atomic operation will impose no restrictions on other memory operations. The only effect that remains, is that the operation is itself still atomic.

> 与**顺序一致性**相反的是**放松的**记忆顺序。它是用“std:：memory_order_relaxed”标记选中的。宽松的原子操作不会对其他内存操作施加任何限制。剩下的唯一效果是，操作本身仍然是原子的。

### Release-Acquire Ordering

An **atomic store** operation can be tagged with `std::memory_order_release` and an **atomic load** operation can be tagged with `std::memory_order_acquire`. The first operation is called **(atomic) store-release** while the second is called **(atomic) load-acquire**.

> **原子存储**操作可以标记为“std:：memory_order_release”，**原子加载**操作可以添加为“std：：memory_ord_acquire”。第一个操作被称为**（原子）存储释放**，而第二个操作则被称为***（原子）加载获取**。

When **load-acquire** sees the value written by a **store-release** the following happens: all store operations sequenced before the **store-release** become visible to (**happen before**) load operations that are sequenced after the **load-acquire**.

> 当**加载获取**看到**存储版本**写入的值时，会发生以下情况：**存储版本***之前排序的所有存储操作对**加载获取***之后排序的加载操作可见（**发生在**之前）。

Atomic read-modify-write operations can also receive the cumulative tag `std::memory_order_acq_rel`. This makes the **atomic load** portion of the operation an **atomic load-acquire** while the **atomic store** portion becomes **atomic store-release**.

> 原子读取-修改-写入操作还可以接收累积标记“std:：memory_order_aq_rel”。这使得操作的**原子加载**部分成为**原子加载获取**，而**原子存储**部分变成**原子存储释放**。

The compiler is not allowed to move store operations after an **atomic store-release** operation. It is also not allowed to move load operations before **atomic load-acquire** (or **load-consume**).

> 不允许编译器在**原子存储释放**操作之后移动存储操作。在**原子负载获取**（或**负载消耗**）之前，也不允许移动负载操作。

Also note that there is no **atomic load-release** or **atomic store-acquire**. Attempting to create such operations makes them **relaxed** operations.

> 还要注意，没有**原子加载释放**或**原子存储获取**。试图创建这样的操作会使它们变得**轻松**操作。

### Release-Consume Ordering

This combination is similar to **release-acquire**, but this time the **atomic load** is tagged with `std::memory_order_consume` and becomes **(atomic) load-consume** operation. This mode is the same as **release-acquire** with the only difference that among the load operations sequenced after the **load-consume** only these depending on the value loaded by the **load-consume** are ordered.

> 这种组合类似于**发布获取**，但这一次**原子加载**被标记为“std:：memory_order_sume”，并成为**（原子）加载消耗**操作。该模式与**释放获取**相同，唯一的区别是在**负载消耗**之后排序的加载操作中，仅根据**负载消耗***加载的值对这些操作进行排序。

### Fences

Fences also allow memory operations to be ordered between threads. A fence is either a release fence or acquire fence.

> 围栏还允许在线程之间对内存操作进行排序。围栏是释放围栏或获取围栏。

If a release fence happens before an acquire fence, then stores sequenced before the release fence are visible to loads sequenced after the acquire fence. To guarantee that the release fence happens before the acquire fence one may use other synchronization primitives including relaxed atomic operations.

> 如果释放围栏发生在获取围栏之前，则在释放围栏之前排序的存储对于在获取围栏之后排序的负载可见。为了保证释放围栏发生在获取围栏之前，可以使用包括放松原子操作的其他同步原语。
