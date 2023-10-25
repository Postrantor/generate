---
tip: translate by baidu@2023-10-25 08:32:06
---
---
metaTitle: "C++ | std::atomics"
description: "atomic types"
---

# std::atomics



## atomic types



Each instantiation and full specialization of the `std::atomic` template defines an atomic type. If one thread writes to an atomic object while another thread reads from it, the behavior is well-defined (see memory model for details on data races)

> “std:：atomic”模板的每个实例化和完全专业化都定义了一个原子类型。如果一个线程向原子对象写入，而另一个线程从中读取，则行为定义良好（有关数据竞赛的详细信息，请参阅内存模型）


In addition, accesses to atomic objects may establish inter-thread synchronization and order non-atomic memory accesses as specified by `std::memory_order`.

> 此外，对原子对象的访问可以建立线程间同步，并按照“std:：memory_order”指定的顺序对非原子内存访问进行排序。


std::atomic may be instantiated with any `TriviallyCopyable type T. std::atomic` is neither copyable nor movable.

> std:：atomic可以用任何“TriviallyCopyable type T”实例化。std:：原子既不可复制也不可移动。


The standard library provides specializations of the std::atomic template for the following types:

> 标准库为以下类型提供std：：原子模板的专门化：


1. One full specialization for the type `bool` and its typedef name is defined that is treated as a non-specialized `std::atomic<T>` except that it has standard layout, trivial default constructor, trivial destructors, and supports aggregate initialization syntax:

> 1.定义了类型“bool”及其typedef名称的一个完全专门化，该名称被视为非专门化的“std:：atomic＜T＞”，但它具有标准布局、琐碎的默认构造函数、琐碎的析构函数，并支持聚合初始化语法：

|Typedef name|Full specialization
|---|---|---|---|---|---|---|---|---|---
|`std::atomic_bool`|`std::atomic<bool>`

2)Full specializations and typedefs for integral types, as follows:

|Typedef name|Full specialization
|---|---|---|---|---|---|---|---|---|---
|`std::atomic_char`|`std::atomic<char>`
|`std::atomic_char`|`std::atomic<char>`
|`std::atomic_schar`|`std::atomic<signed char>`
|`std::atomic_uchar`|`std::atomic<unsigned char>`
|`std::atomic_short`|`std::atomic<short>`
|`std::atomic_ushort`|`std::atomic<unsigned short>`
|`std::atomic_int`|`std::atomic<int>`
|`std::atomic_uint`|`std::atomic<unsigned int>`
|`std::atomic_long`|`std::atomic<long>`
|`std::atomic_ulong`|`std::atomic<unsigned long>`
|`std::atomic_llong`|`std::atomic<long long>`
|`std::atomic_ullong`|`std::atomic<unsigned long long>`
|`std::atomic_char16_t`|`std::atomic<char16_t>`
|`std::atomic_char32_t`|`std::atomic<char32_t>`
|`std::atomic_wchar_t`|`std::atomic<wchar_t>`
|`std::atomic_int8_t`|`std::atomic<std::int8_t>`
|`std::atomic_uint8_t`|`std::atomic<std::uint8_t>`
|`std::atomic_int16_t`|`std::atomic<std::int16_t>`
|`std::atomic_uint16_t`|`std::atomic<std::uint16_t>`
|`std::atomic_int32_t`|`std::atomic<std::int32_t>`
|`std::atomic_uint32_t`|`std::atomic<std::uint32_t>`
|`std::atomic_int64_t`|`std::atomic<std::int64_t>`
|`std::atomic_uint64_t`|`std::atomic<std::uint64_t>`
|`std::atomic_int_least8_t`|`std::atomic<std::int_least8_t>`
|`std::atomic_uint_least8_t`|`std::atomic<std::uint_least8_t>`
|`std::atomic_int_least16_t`|`std::atomic<std::int_least16_t>`
|`std::atomic_uint_least16_t`|`std::atomic<std::uint_least16_t>`
|`std::atomic_int_least32_t`|`std::atomic<std::int_least32_t>`
|`std::atomic_uint_least32_t`|`std::atomic<std::uint_least32_t>`
|`std::atomic_int_least64_t`|`std::atomic<std::int_least64_t>`
|`std::atomic_uint_least64_t`|`std::atomic<std::uint_least64_t>`
|`std::atomic_int_fast8_t`|`std::atomic<std::int_fast8_t>`
|`std::atomic_uint_fast8_t`|`std::atomic<std::uint_fast8_t>`
|`std::atomic_int_fast16_t`|`std::atomic<std::int_fast16_t>`
|`std::atomic_uint_fast16_t`|`std::atomic<std::uint_fast16_t>`
|`std::atomic_int_fast32_t`|`std::atomic<std::int_fast32_t>`
|`std::atomic_uint_fast32_t`|`std::atomic<std::uint_fast32_t>`
|`std::atomic_int_fast64_t`|`std::atomic<std::int_fast64_t>`
|`std::atomic_uint_fast64_t`|`std::atomic<std::uint_fast64_t>`
|`std::atomic_intptr_t`|`std::atomic<std::intptr_t>`
|`std::atomic_uintptr_t`|`std::atomic<std::uintptr_t>`
|`std::atomic_size_t`|`std::atomic<std::size_t>`
|`std::atomic_ptrdiff_t`|`std::atomic<std::ptrdiff_t>`
|`std::atomic_intmax_t`|`std::atomic<std::intmax_t>`
|`std::atomic_uintmax_t`|`std::atomic<std::uintmax_t>`

Simple example of using std::atomic_int

```cpp
#include <iostream>       // std::cout
#include <atomic>         // std::atomic, std::memory_order_relaxed
#include <thread>         // std::thread

std::atomic_int foo (0);

void set_foo(int x) {
  foo.store(x,std::memory_order_relaxed);     // set value atomically
}

void print_foo() {
  int x;
  do {
    x = foo.load(std::memory_order_relaxed);  // get value atomically
  } while (x==0);
  std::cout << "foo: " << x << '\n';
}

int main ()
{
  std::thread first (print_foo);
  std::thread second (set_foo,10);
  first.join();
  //second.join();
  return 0;
}
//output: foo: 10

```

