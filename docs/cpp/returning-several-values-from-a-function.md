---
tip: translate by baidu@2023-10-25 08:30:43
---
---

metaTitle: "C++ | Returning several values from a function"
description: "Using std::tuple, Structured Bindings, Using struct, Using Output Parameters, Using a Function Object Consumer, Using std::pair, Using std::array, Using Output Iterator, Using std::vector"
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# Returning several values from a function

There are many situations where it is useful to return several values from a function: for example, if you want to input an item and return the price and number in stock, this functionality could be useful. There are many ways to do this in C++, and most involve the STL. However, if you wish to avoid the STL for some reason, there are still several ways to do this, including `structs/classes` and `arrays`.

> 在许多情况下，从函数中返回几个值是有用的：例如，如果您想输入一个项目并返回库存的价格和数量，此功能可能会很有用。在 C++ 中有很多方法可以做到这一点，其中大多数都涉及 STL。然而，如果出于某种原因希望避免 STL，仍然有几种方法可以做到这一点，包括“structs/classes”和“arrays”。

## Using std::tuple

The type [`std::tuple`](http://en.cppreference.com/w/cpp/utility/tuple) can bundle any number of values, potentially including values of different types, into a single return object:

> 类型[`std:：tuple`](http://en.cppreference.com/w/cpp/utility/tuple)可以将任意数量的值（可能包括不同类型的值）绑定到单个返回对象中：

```cpp
std::tuple<int, int, int, int> foo(int a, int b) { // or auto (C++14)
   return std::make_tuple(a + b, a - b, a * b, a / b);
}

```

In C++17, a braced initializer list can be used:

```cpp
std::tuple<int, int, int, int> foo(int a, int b)    {
    return {a + b, a - b, a * b, a / b};
}

```

Retrieving values from the returned `tuple` can be cumbersome, requiring the use of the [`std::get`](http://en.cppreference.com/w/cpp/utility/tuple/get) template function:

> 从返回的“元组”中检索值可能很麻烦，需要使用[`std:：get`](http://en.cppreference.com/w/cpp/utility/tuple/get)模板函数：

```cpp
auto mrvs = foo(5, 12);
auto add = std::get<0>(mrvs);
auto sub = std::get<1>(mrvs);
auto mul = std::get<2>(mrvs);
auto div = std::get<3>(mrvs);

```

If the types can be declared before the function returns, then [`std::tie`](http://en.cppreference.com/w/cpp/utility/tuple/tie) can be employed to unpack a `tuple` into existing variables:

> 如果可以在函数返回之前声明类型，那么[`std:：tie`](http://en.cppreference.com/w/cpp/utility/tuple/tie)可以用来将“元组”解包到现有变量中：

```cpp
int add, sub, mul, div;
std::tie(add, sub, mul, div) = foo(5, 12);

```

If one of the returned values is not needed, [`std::ignore`](http://en.cppreference.com/w/cpp/utility/tuple/ignore) can be used:

> 如果不需要其中一个返回值，则[`std:：igner`](http://en.cppreference.com/w/cpp/utility/tuple/ignore)可以使用：

```cpp
std::tie(add, sub, std::ignore, div) = foo(5, 12);

```

[Structured bindings](http://stackoverflow.com/documentation/c%2B%2B/487/functions-with-multiple-return-values/3384/structured-bindings#t=201607230711000500592) can be used to avoid `std::tie`:

> [结构化绑定]([http://stackoverflow.com/documentation/c%2b%2b/487/functions](http://stackoverflow.com/documentation/c%2B%2B/487/functions)-具有多个返回值/3384/结构化绑定#t=201607230711000050592）可以用于避免“std:：tie”：

```cpp
auto [add, sub, mul, div] = foo(5,12);

```

If you want to return a tuple of lvalue references instead of a tuple of values, use `std::tie` in place of [`std::make_tuple`](http://en.cppreference.com/w/cpp/utility/tuple/make_tuple).

> 如果要返回左值引用的元组而不是值的元组，请使用“std:：tie”代替[`std:：make_tuple`](http://en.cppreference.com/w/cpp/utility/tuple/make_tuple)。

```cpp
std::tuple<int&, int&> minmax( int& a, int& b ) {
  if (b<a)
    return std::tie(b,a);
  else
    return std::tie(a,b);
}

```

which permits

```cpp
void increase_least(int& a, int& b) {
  std::get<0>(minmax(a,b))++;
}

```

In some rare cases you'll use [`std::forward_as_tuple`](http://en.cppreference.com/w/cpp/utility/tuple/forward_as_tuple) instead of `std::tie`; be careful if you do so, as temporaries may not last long enough to be consumed.

> 在极少数情况下，您将使用[`std:：forward_as_tuple`](http://en.cppreference.com/w/cpp/utility/tuple/forward_as_tuple)而不是“std:：tie”；如果你这样做的话要小心，因为临时食品可能不会持续足够长的时间来消费。

## Structured Bindings

C++17 introduces structured bindings, which makes it even easier to deal with multiple return types, as you do not need to rely upon [`std::tie()`](http://en.cppreference.com/w/cpp/utility/tuple/tie) or do any manual tuple unpacking:

> C++17 引入了结构化绑定，这使得处理多个返回类型变得更加容易，因为您不需要依赖[`std:：tie（）`](http://en.cppreference.com/w/cpp/utility/tuple/tie)或者进行任何手动元组拆包：

```cpp
std::map<std::string, int> m;

// insert an element into the map and check if insertion succeeded
auto [iterator, success] = m.insert({"Hello", 42});

if (success) {
    // your code goes here
}

// iterate over all elements without having to use the cryptic 'first' and 'second' names
for (auto const& [key, value] : m) {
    std::cout << "The value for " << key << " is " << value << '\n';
}

```

Structured bindings can be used by default with `std::pair`, `std::tuple`, and any type whose non-static data members are all either public direct members or members of an unambiguous base class:

> 默认情况下，结构化绑定可以与“std:：pair”、“std：：tuple”以及任何非静态数据成员都是公共直接成员或明确基类成员的类型一起使用：

```cpp
struct A { int x; };
struct B : A { int y; };
B foo();

// with structured bindings
const auto [x, y] = foo();

// equivalent code without structured bindings
const auto result = foo();
auto& x = result.x;
auto& y = result.y;

```

If you make your type "tuple-like" it will also automatically work with your type. A tuple-like is a type with appropriate `tuple_size`, `tuple_element` and `get` written:

> 如果您将类型设置为“tuple-like”，它也将自动与您的类型配合使用。类元组是一种写有适当“tuple_size”、“tuple_element”和“get”的类型：

```cpp
namespace my_ns {
    struct my_type {
        int x;
        double d;
        std::string s;
    };
    struct my_type_view {
        my_type* ptr;
    };
}

namespace std {
    template<>
    struct tuple_size<my_ns::my_type_view> : std::integral_constant<std::size_t, 3>
    {};

    template<> struct tuple_element<my_ns::my_type_view, 0>{ using type = int; };
    template<> struct tuple_element<my_ns::my_type_view, 1>{ using type = double; };
    template<> struct tuple_element<my_ns::my_type_view, 2>{ using type = std::string; };
}

namespace my_ns {
    template<std::size_t I>
    decltype(auto) get(my_type_view const& v) {
        if constexpr (I == 0)
            return v.ptr->x;
        else if constexpr (I == 1)
            return v.ptr->d;
        else if constexpr (I == 2)
            return v.ptr->s;
        static_assert(I < 3, "Only 3 elements");
    }
}

```

now this works:

```cpp
my_ns::my_type t{1, 3.14, "hello world"};

my_ns::my_type_view foo() {
    return {&t};
}

int main() {
    auto[x, d, s] = foo();
    std::cout << x << ',' << d << ',' << s << '\n';
}

```

## Using struct

A [`struct`](http://en.cppreference.com/w/cpp/language/class) can be used to bundle multiple return values:

> [`struct`](http://en.cppreference.com/w/cpp/language/class)可以用于绑定多个返回值：

```cpp
struct foo_return_type {
    int add;
    int sub;
    int mul;
    int div;
};

foo_return_type foo(int a, int b) {
    return {a + b, a - b, a * b, a / b};
}

auto calc = foo(5, 12);

```

Instead of assignment to individual fields, a constructor can be used to simplify the constructing of returned values:

> 构造函数可以用来简化返回值的构造，而不是分配给单个字段：

```cpp
struct foo_return_type {
    int add;
    int sub;
    int mul;
    int div;
    foo_return_type(int add, int sub, int mul, int div)
    : add(add), sub(sub), mul(mul), div(div) {}
};

foo_return_type foo(int a, int b) {
     return foo_return_type(a + b, a - b, a * b, a / b);
}

foo_return_type calc = foo(5, 12);

```

The individual results returned by the function `foo()` can be retrieved by accessing the member variables of the `struct` `calc`:

> 函数“foo（）”返回的各个结果可以通过访问“struct”“calc”的成员变量来检索：

```cpp
std::cout << calc.add << ' ' << calc.sub << ' ' << calc.mul << ' ' << calc.div << '\n';

```

**Output:**

>

17 -7 60 0

Note: When using a `struct`, the returned values are grouped together in a single object and accessible using meaningful names. This also helps to reduce the number of extraneous variables created in the scope of the returned values.

> 注意：当使用“struct”时，返回的值被分组在一个对象中，并且可以使用有意义的名称进行访问。这也有助于减少在返回值的范围内创建的无关变量的数量。

In order to unpack a `struct` returned from a function, [structured bindings](https://stackoverflow.com/documentation/c%2B%2B/487/functions-with-multiple-return-values/3384/structured-bindings) can be used. This places the out-parameters on an even footing with the in-parameters:

> 为了解包从函数返回的“struct”，[结构化绑定]([https://stackoverflow.com/documentation/c%2b%2b/487/functions](https://stackoverflow.com/documentation/c%2B%2B/487/functions)-具有多个返回值/3384/结构化绑定）。这将使 out 参数与 in 参数处于均匀的基础上：

```cpp
int a=5, b=12;
auto[add, sub, mul, div] = foo(a, b);
std::cout << add << ' ' << sub << ' ' << mul << ' ' << div << '\n';

```

The output of this code is identical to that above. The `struct` is still used to return the values from the function. This permits you do deal with the fields individually.

> 此代码的输出与上面的相同。“struct”仍然用于返回函数中的值。这允许您单独处理字段。

## Using Output Parameters

Parameters can be used for returning one or more values; those parameters are required to be non-`const` pointers or references.

> 参数可用于返回一个或多个值；那些参数必须是非常量指针或引用。

References:

```cpp
void calculate(int a, int b, int& c, int& d, int& e, int& f) {
    c = a + b;
    d = a - b;
    e = a * b;
    f = a / b;
}

```

Pointers:

```cpp
void calculate(int a, int b, int* c, int* d, int* e, int* f) {
    *c = a + b;
    *d = a - b;
    *e = a * b;
    *f = a / b;
}

```

Some libraries or frameworks use an empty 'OUT' `#define` to make it abundantly obvious which parameters are output parameters in the function signature. This has no functional impact, and will be compiled out, but makes the function signature a bit clearer;

> 一些库或框架使用空的“OUT”“#define”，以使函数签名中的输出参数非常明显。这对函数没有影响，并且会被编译出来，但会使函数签名更加清晰；

```cpp
#define OUT

void calculate(int a, int b, OUT int& c) {
    c = a + b;
}

```

## Using a Function Object Consumer

We can provide a consumer that will be called with the multiple relevant values:

> 我们可以为将被调用的消费者提供多个相关值：

```cpp
template <class F>
void foo(int a, int b, F consumer) {
    consumer(a + b, a - b, a * b, a / b);
}

// use is simple... ignoring some results is possible as well
foo(5, 12, [](int sum, int , int , int ){
    std::cout << "sum is " << sum << '\n';
});

```

This is known as ["continuation passing style"](https://en.wikipedia.org/wiki/Continuation-passing_style).

> 这被称为[“延续传递样式”](https://en.wikipedia.org/wiki/Continuation-passing_style)。

You can adapt a function returning a tuple into a continuation passing style function via:

> 您可以通过以下方式将返回元组的函数调整为延续传递样式的函数：

```cpp
template<class Tuple>
struct continuation {
  Tuple t;
  template<class F>
  decltype(auto) operator->*(F&& f)&&{
    return std::apply( std::forward<F>(f), std::move(t) );
  }
};
std::tuple<int,int,int,int> foo(int a, int b);

continuation(foo(5,12))->*[](int sum, auto&&...) {
  std::cout << "sum is " << sum << '\n';
};

```

with more complex versions being writable in C++14 or C++11.

## Using std::pair

The struct template [`std::pair`](http://en.cppreference.com/w/cpp/utility/pair) can bundle together **exactly** two return values, of any two types:

> 结构模板[`std:：pair`](http://en.cppreference.com/w/cpp/utility/pair)可以将任意两种类型的**恰好**两个返回值捆绑在一起：

```cpp
#include <utility>
std::pair<int, int> foo(int a, int b) {
    return std::make_pair(a+b, a-b);
}

```

With C++11 or later, an initializer list can be used instead of `std::make_pair`:

> 对于 C++11 或更高版本，可以使用初始值设定项列表来代替“std:：make_pair”：

```cpp
#include <utility>
std::pair<int, int> foo(int a, int b) {
    return {a+b, a-b};
}

```

The individual values of the returned `std::pair` can be retrieved by using the pair's `first` and `second` member objects:

> 返回的“std:：pair”的各个值可以通过使用该对的“first”和“second”成员对象来检索：

```cpp
std::pair<int, int> mrvs = foo(5, 12);
std::cout << mrvs.first + mrvs.second << std::endl;

```

Output:

>

10

## Using std::array

The container `std::array` can bundle together a fixed number of return values. This number has to be known at compile-time and all return values have to be of the same type:

> 容器“std:：array”可以将固定数量的返回值捆绑在一起。这个数字必须在编译时已知，并且所有返回值都必须是相同的类型：

```cpp
std::array<int, 4> bar(int a, int b) {
    return { a + b, a - b, a * b, a / b };
}

```

This replaces c style arrays of the form `int bar[4]`. The advantage being that various `c++` std functions can now be used on it. It also provides useful member functions like `at` which is a safe member access function with bound checking, and `size` which allows you to return the size of the array without calculation.

> 这将替换形式为“int bar[4]”的 c 样式数组。优点是现在可以在它上使用各种“c++”std 函数。它还提供了有用的成员函数，如“at”，这是一个带绑定检查的安全成员访问函数，以及“size”，它允许您在不计算的情况下返回数组的大小。

## Using Output Iterator

Several values of the same type can be returned by passing an output iterator to the function. This is particularly common for generic functions (like the algorithms of the standard library).

> 通过将输出迭代器传递给函数，可以返回多个相同类型的值。这对于泛型函数（如标准库的算法）来说尤其常见。

Example:

```cpp
template<typename Incrementable, typename OutputIterator>
void generate_sequence(Incrementable from, Incrementable to, OutputIterator output) {
    for (Incrementable k = from; k != to; ++k)
        *output++ = k;
}

```

Example usage:

```cpp
std::vector<int> digits;
generate_sequence(0, 10, std::back_inserter(digits));
// digits now contains {0, 1, 2, 3, 4, 5, 6, 7, 8, 9}

```

## Using std::vector

A `std::vector` can be useful for returning a dynamic number of variables of the same type. The following example uses `int` as data type, but a `std::vector` can hold any type that is trivially copyable:

> “std:：vector”可用于返回相同类型的动态变量数。以下示例使用“int”作为数据类型，但“std:：vector”可以包含任何可复制的类型：

```cpp
#include <vector>
#include <iostream>

// the following function returns all integers between and including 'a' and 'b' in a vector
// (the function can return up to std::vector::max_size elements with the vector, given that
// the system's main memory can hold that many items)
std::vector<int> fillVectorFrom(int a, int b) {
    std::vector<int> temp;
    for (int i = a; i <= b; i++) {
        temp.push_back(i);
    }
    return temp;
}

int main() {    
    // assigns the filled vector created inside the function to the new vector 'v'
    std::vector<int> v = fillVectorFrom(1, 10);

    // prints "1 2 3 4 5 6 7 8 9 10 "
    for (int i = 0; i < v.size(); i++) {
        std::cout << v[i] << " ";
    }
    std::cout << std::endl;
    return 0;
}

```
