---
tip: translate by baidu@2023-10-25 08:22:55
---
---

metaTitle: "C++ | Concurrency With OpenMP"
description: "OpenMP: Parallel Sections, OpenMP: Parallel Sections, OpenMP: Parallel For Loop, OpenMP: Parallel Gathering / Reduction"
--------------------------------------------------------------------------------------------------------------------------------------

# Concurrency With OpenMP

This topic covers the basics of concurrency in C++ using OpenMP. OpenMP is documented in more detail in the [OpenMP tag](https://stackoverflow.com/documentation/openmp).

> 本主题介绍使用 OpenMP 在 C++ 中实现并发的基础知识。OpenMP 在[OpenMP 标签]中有更详细的记录([https://stackoverflow.com/documentation/openmp](https://stackoverflow.com/documentation/openmp))。

Parallelism or concurrency implies the execution of code at the same time.

> 并行性或并发性意味着同时执行代码。

## OpenMP: Parallel Sections

This example illustrates the basics of executing sections of code in parallel.

> 此示例说明了并行执行代码部分的基本原理。

As OpenMP is a built-in compiler feature, it works on any supported compilers without including any libraries. You may wish to include `omp.h` if you want to use any of the openMP API features.

> 由于 OpenMP 是一个内置的编译器功能，它可以在任何受支持的编译器上工作，而不包括任何库。如果您想使用任何 openMP API 功能，您可能希望包含“omp.h”。

**Sample Code**

```cpp
std::cout << "begin ";
//    This pragma statement hints the compiler that the
//    contents within the { } are to be executed in as
//    parallel sections using openMP, the compiler will
//    generate this chunk of code for parallel execution
#pragma omp parallel sections
{
    //    This pragma statement hints the compiler that
    //    this is a section that can be executed in parallel
    //    with other section, a single section will be executed
    //    by a single thread.
    //    Note that it is "section" as opposed to "sections" above
    #pragma omp section
    {
        std::cout << "hello " << std::endl;
        /** Do something **/
    }
    #pragma omp section
    {
        std::cout << "world " << std::endl;
        /** Do something **/
    }
}
//    This line will not be executed until all the
//    sections defined above terminates
std::cout << "end" << std::endl;

```

**Outputs**

This example produces 2 possible outputs and is dependent on the operating system and hardware. The output also illustrates a **race condition** problem that would occur from such an implementation.

> 此示例产生 2 种可能的输出，并且取决于操作系统和硬件。输出还说明了这种实现可能出现的**竞争条件**问题。

| OUTPUT A              | OUTPUT B              |  |  |  |  |  |  |  |  |
| --------------------- | --------------------- | - | - | - | - | - | - | - | - |
| begin hello world end | begin world hello end |  |  |  |  |  |  |  |  |

## OpenMP: Parallel Sections

This example shows how to execute chunks of code in parallel

```cpp
std::cout << "begin ";
//    Start of parallel sections
#pragma omp parallel sections
{
    //    Execute these sections in parallel
    #pragma omp section
    {
        ... do something ...
        std::cout << "hello ";
    }
    #pragma omp section
    {
        ... do something ...
        std::cout << "world ";
    }
    #pragma omp section
    {
        ... do something ...
        std::cout << "forever ";
    }
}
//    end of parallel sections
std::cout << "end";

```

**Output**

- begin hello world forever end
- begin world hello forever end
- begin hello forever world end
- begin forever hello world end

As execution order is not guaranteed, you may observe any of the above output.

> 由于不能保证执行顺序，您可能会观察到上面的任何输出。

## OpenMP: Parallel For Loop

This example shows how to divide a loop into equal parts and execute them in parallel.

> 这个例子展示了如何将循环分成相等的部分并并行执行。

```cpp
//    Splits element vector into element.size() / Thread Qty
//    and allocate that range for each thread.
#pragma omp parallel for
for    (size_t i = 0; i < element.size(); ++i)
    element[i] = ...

//    Example Allocation (100 element per thread)
//    Thread 1 : 0 ~ 99
//    Thread 2 : 100 ~ 199
//    Thread 2 : 200 ~ 299
//    ...

//    Continue process
//    Only when all threads completed their allocated
//    loop job
...

```

*Please take extra care to not modify the size of the vector used in parallel for loops as **allocated range indices doesn't update automatically**.

> *请格外小心，不要修改循环并行使用的向量的大小，因为**分配的范围索引不会自动更新**。

## OpenMP: Parallel Gathering / Reduction

This example illustrates a concept to perform reduction or gathering using `std::vector` and OpenMP.

> 此示例说明了使用“std:：vector”和 OpenMP 执行减少或收集的概念。

Supposed we have a scenario where we want multiple threads to help us generate a bunch of stuff, `int` is used here for simplicity and can be replaced with other data types.

> 假设我们有一个场景，需要多个线程来帮助我们生成一堆东西，为了简单起见，这里使用了“int”，并且可以用其他数据类型替换。

This is particularly useful when you need to merge results from slaves to avoid segement faults or memory access violations and do not wish to use libraries or custom sync container libraries.

> 当您需要合并从属对象的结果以避免分段错误或内存访问冲突，并且不希望使用库或自定义同步容器库时，这一点尤其有用。

```cpp
//    The Master vector
//    We want a vector of results gathered from slave threads
std::vector<int> Master;    

//    Hint the compiler to parallelize this { } of code
//    with all available threads (usually the same as logical processor qty)
#pragma omp parallel
{
    //    In this area, you can write any code you want for each
    //    slave thread, in this case a vector to hold each of their results
    //    We don't have to worry about how many threads were spawn or if we need
    //    to repeat this declaration or not.
    std::vector<int> Slave;

    //    Tell the compiler to use all threads allocated for this parallel region
    //    to perform this loop in parts. Actual load appx = 1000000 / Thread Qty
    //    The nowait keyword tells the compiler that the slave threads don't
    //    have to wait for all other slaves to finish this for loop job
    #pragma omp for nowait
    for (size_t i = 0; i < 1000000; ++i
    {
        /* Do something */
        ....
        Slave.push_back(...);
    }

    //    Slaves that finished their part of the job
    //    will perform this thread by thread one at a time
    //    critical section ensures that only 0 or 1 thread performs
    //    the { } at any time
    #pragma omp critical
    {
        //    Merge slave into master
        //    use move iterators instead, avoid copy unless
        //    you want to use it for something else after this section
        Master.insert(Master.end(), 
                      std::make_move_iterator(Slave.begin()), 
                      std::make_move_iterator(Slave.end()));
    }
}

//    Have fun with Master vector
...

```

#### Remarks

OpenMP does not require any special headers or libraries as it is a built-in compiler feature. However, if you use any OpenMP API functions such as `omp_get_thread_num()`, you will need to include `omp.h` and its library.

> OpenMP 不需要任何特殊的头或库，因为它是一个内置的编译器功能。但是，如果使用任何 OpenMP API 函数，如“omp_get_thread_num（）”，则需要包含“omp.h”及其库。

OpenMP `pragma` statements are ignored when the OpenMP option is not enabled during compilation. You may want to refer to the compiler option in your compiler's manual.

> 编译期间未启用 OpenMP 选项时，将忽略 OpenMP“pragma”语句。您可能需要参考编译器手册中的编译器选项。

- GCC uses   `-fopenmp`
- Clang uses `-fopenmp`
- MSVC uses  `/openmp`
