---
tip: translate by baidu@2023-10-25 08:29:08
---
---

metaTitle: "C++ | Optimization"
description: "Inline Expansion/Inlining, Empty base optimization"
-----------------------------------------------------------------

# Optimization

When compiling, the compiler will often modify the program to increase performance. This is permitted by the [as-if rule](http://en.cppreference.com/w/cpp/language/as_if), which allows any and all transformations that do not change observable behavior.

> 编译时，编译器经常会修改程序以提高性能。这是[好像规则]允许的([http://en.cppreference.com/w/cpp/language/as_if](http://en.cppreference.com/w/cpp/language/as_if))，它允许任何和所有不改变可观察行为的转换。

## Inline Expansion/Inlining

Inline expansion (also known as inlining) is compiler optimisation that replaces a call to a function with the body of that function. This saves the function call overhead, but at the cost of space, since the function may be duplicated several times.

> 内联扩展（也称为内联）是一种编译器优化，它用函数体替换对函数的调用。这节省了函数调用开销，但以空间为代价，因为函数可能会重复多次。

```cpp
// source:

int process(int value)
{
    return 2 * value;
}

int foo(int a)
{
    return process(a);
}

// program, after inlining:

int foo(int a)
{
    return 2 * a; // the body of process() is copied into foo()
}

```

Inlining is most commonly done for small functions, where the function call overhead is significant compared to the size of the function body.

> 内联最常见于小函数，与函数体的大小相比，函数调用开销很大。

## Empty base optimization

The size of any object or member subobject is required to be at least 1 even if the type is an empty `class` type (that is, a `class` or `struct` that has no non-static data members), in order to be able to guarantee that the addresses of distinct objects of the same type are always distinct.

> 任何对象或成员子对象的大小都必须至少为 1，即使该类型是空的“class”类型（即没有非静态数据成员的“类”或“结构”），以便能够保证同一类型的不同对象的地址总是不同的。

However, base `class` subobjects are not so constrained, and can be completely optimized out from the object layout:

> 但是，基本的“类”子对象不受约束，可以从对象布局中完全优化：

```cpp
#include <cassert>

struct Base {}; // empty class

struct Derived1 : Base {
    int i;
};

int main() {
    // the size of any object of empty class type is at least 1
    assert(sizeof(Base) == 1);

    // empty base optimization applies
    assert(sizeof(Derived1) == sizeof(int));
}

```

Empty base optimization is commonly used by allocator-aware standard library classes (`std::vector`, `std::function`, `std::shared_ptr`, etc) to avoid occupying any additional storage for its allocator member if the allocator is stateless. This is achieved by storing one of the required data members (e.g., `begin`, `end`, or `capacity` pointer for the `vector`).

> 空基优化通常由具有分配器意识的标准库类（“std:：vector”、“std：：function”、“std：：shared_ptr”等）使用，以避免在分配器是无状态的情况下占用其分配器成员的任何额外存储。这是通过存储一个所需的数据成员（例如，“开始”、“结束”或“向量”的“容量”指针）来实现的。

Reference: [cppreference](http://en.cppreference.com/w/cpp/language/ebo)
