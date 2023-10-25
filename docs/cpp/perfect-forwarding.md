---
tip: translate by baidu@2023-10-25 08:29:24
---
---

metaTitle: "C++ | Perfect Forwarding"
description: "Factory functions"
--------------------------------

# Perfect Forwarding

## Factory functions

Suppose we want to write a factory function that accepts an arbitrary list of arguments and passes those arguments unmodified to another function. An example of such a function is `make_unique`, which is used to safely construct a new instance of `T` and return a `unique_ptr<T>` that owns the instance.

> 假设我们想要编写一个工厂函数，该函数接受任意的参数列表，并将这些参数未经修改地传递给另一个函数。这种函数的一个例子是“make_unique”，它用于安全地构造“T”的新实例，并返回拥有该实例的“unique_ptr＜T＞”。

The language rules regarding variadic templates and rvalue references allows us to write such a function.

> 关于可变模板和右值引用的语言规则允许我们编写这样的函数。

```cpp
template<class T, class... A>
unique_ptr<T> make_unique(A&&... args)
{
    return unique_ptr<T>(new T(std::forward<A>(args)...));
}

```

The use of ellipses `...` indicate a parameter pack, which represents an arbitrary number of types. The compiler will expand this parameter pack to the correct number of arguments at the call site. These arguments are then passed to `T`'s constructor using `std::forward`. This function is required to preserve the ref-qualifiers of the arguments.

> 省略号“…”的使用表示一个参数包，它表示任意数量的类型。编译器将在调用站点将此参数包扩展到正确数量的参数。然后使用“std:：forward”将这些参数传递给“T”的构造函数。此函数是保留参数的 ref 限定符所必需的。

```cpp
struct foo
{
    foo() {}
    foo(const foo&) {}                    // copy constructor
    foo(foo&&) {}                         // copy constructor
    foo(int, int, int) {}
};

foo f;
auto p1 = make_unique<foo>(f);            // calls foo::foo(const foo&)
auto p2 = make_unique<foo>(std::move(f)); // calls foo::foo(foo&&)
auto p3 = make_unique<foo>(1, 2, 3); 

```

#### Remarks

Perfect forwarding requires **forwarding references** in order to preserve the ref-qualifiers of the arguments. Such references appear only in a **deduced context**. That is:

> 完美转发需要**转发引用**，以便保留参数的 ref 限定符。此类引用仅出现在**推导的上下文**中。即：

```cpp
template<class T>
void f(T&& x) // x is a forwarding reference, because T is deduced from a call to f()
{
    g(std::forward<T>(x)); // g() will receive an lvalue or an rvalue, depending on x
}

```

The following does not involve perfect forwarding, because `T` is not deduced from the constructor call:

> 以下不涉及完美转发，因为“T”不是从构造函数调用中推导出来的：

```cpp
template<class T>
struct a
{
    a(T&& x); // x is a rvalue reference, not a forwarding reference
};

```

C++17 will allow deduction of class template arguments. The constructor of "a" in the above example will become a user of a forwarding reference

> C++17 将允许类模板参数的推导。上例中“a”的构造函数将成为转发引用的用户

```cpp
a example1(1);
  // same as a<int> example1(1);

int x = 1;
a example2(x);
  // same as a<int&> example2(x);

```
