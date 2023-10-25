---
tip: translate by baidu@2023-10-25 08:34:58
---
---
metaTitle: "C++ | type deduction"
description: "Template parameter deduction for constructors, Auto Type Deduction, Template Type Deduction"
---

# type deduction




## Template parameter deduction for constructors



Prior to C++17, template deduction cannot deduce the class type for you in a constructor. It must be explicitly specified. Sometimes, however, these types can be very cumbersome or (in the case of lambdas) impossible to name, so we got a proliferation of type factories (like `make_pair()`, `make_tuple()`, `back_inserter()`, etc.).

> 在C++17之前，模板推导不能在构造函数中为您推导类类型。必须明确指定。然而，有时这些类型可能非常麻烦，或者（在lambdas的情况下）无法命名，因此我们得到了大量的类型工厂（如“make_pair（）”、“make_tuple（）”和“back_inserter（）”等）。

This is no longer necessary:

```cpp
std::pair p(2, 4.5);     // std::pair<int, double>
std::tuple t(4, 3, 2.5); // std::tuple<int, int, double>
std::copy_n(vi1.begin(), 3,
    std::back_insert_iterator(vi2)); // constructs a back_insert_iterator<std::vector<int>>
std::lock_guard lk(mtx); // std::lock_guard<decltype(mtx)>

```


Constructors are considered to deduce the class template parameters, but in some cases this is insufficient and we can provide explicit deduction guides:

> 构造函数被认为可以推导类模板参数，但在某些情况下这是不够的，我们可以提供明确的推导指南：

```cpp
template <class Iter>
vector(Iter, Iter) -> vector<typename iterator_traits<Iter>::value_type>

int array[] = {1, 2, 3};
std::vector v(std::begin(array), std::end(array)); // deduces std::vector<int>

```



## Auto Type Deduction



Type deduction using the `auto` [keyword](http://stackoverflow.com/documentation/c%2b%2b/4891/keywords) works almost the same as Template Type Deduction. Below are a few examples:

> 使用`auto`[关键字]键入推导(http://stackoverflow.com/documentation/c%2b%2b/4891/keywords)工作原理与模板类型推导几乎相同。以下是几个例子：

```cpp
auto x = 27;           // (x is neither a pointer nor a reference), x's type is int
const auto cx = x;     // (cx is neither a pointer nor a reference), cs's type is const int
const auto& rx = x;    // (rx is a non-universal reference), rx's type is a reference to a const int

auto&& uref1 = x;      // x is int and lvalue, so uref1's type is int&
auto&& uref2 = cx;     // cx is const int and lvalue, so uref2's type is const int &
auto&& uref3 = 27;     // 27 is an int and rvalue, so uref3's type is int&&

```

The differences are outlined below:

```cpp
auto x1 = 27;          // type is int, value is 27
auto x2(27);           // type is int, value is 27
auto x3 = { 27 };      // type is std::initializer_list<int>, value is { 27 }
auto x4{ 27 };         // type is std::initializer_list<int>, value is { 27 }
                       // in some compilers type may be deduced as an int with a 
                       // value of 27. See remarks for more information.
auto x5 = { 1, 2.0 }   // error! can't deduce T for std::initializer_list<t>

```


As you can see if you use braced initializers, auto is forced into creating a variable of type `std::initializer_list<T>`. If it can't deduce the of `T`, the code is rejected.

> 正如您所看到的，如果您使用braked初始化器，auto将被迫创建一个类型为“std:：initializer_list＜T＞”的变量。如果它不能推导出“t”的值，则该代码被拒绝。


When `auto` is used as the return type of a function, it specifies that the function has a [trailing return type](http://stackoverflow.com/documentation/c%2b%2b/4142/trailing-return-type).

> 当“auto”用作函数的返回类型时，它指定函数具有[尾随返回类型](http://stackoverflow.com/documentation/c%2b%2b/4142/trailing-返回类型）。

```cpp
auto f() -> int {
    return 42;
}

```


C++14 allows, in addition to the usages of auto allowed in C++11, the following:

> C++14除了允许在C++11中使用auto外，还允许以下内容：

<li>

When used as the return type of a function without a trailing return type, specifies that the function's return type should be deduced from the return statements in the function's body, if any.

> 当用作不带尾部返回类型的函数的返回类型时，指定函数的返回型应该从函数体中的返回语句推导出来（如果有的话）。

```cpp
// f returns int:
auto f() { return 42; }
// g returns void:
auto g() { std::cout << "hello, world!\n"; }

```


</li>
<li>

When used in the parameter type of a lambda, defines the lambda to be a [generic lambda](http://stackoverflow.com/documentation/c%2b%2b/572/lambdas/1969/generic-lambdas).

> 在lambda的参数类型中使用时，将lambda定义为[泛型lambda](http://stackoverflow.com/documentation/c%2b%2b/572/lambdas/1969/generic-lambdas）。

```cpp
auto triple = [](auto x) { return 3*x; };
const auto x = triple(42); // x is a const int with value 126

```


</li>


The special form `decltype(auto)` deduces a type using the type deduction rules of `decltype` rather than those of `auto`.

> 特殊形式“decltype（auto）”使用“declttype”的类型推导规则而不是“auto”的类型推断规则来推导类型。

```cpp
int* p = new int(42);
auto x = *p;           // x has type int
decltype(auto) y = *p; // y is a reference to *p

```


In C++03 and earlier, the `auto` keyword had a completely different meaning as a [storage class specifier](http://stackoverflow.com/documentation/c%2b%2b/9225/storage-class-specifiers/28629/auto) that was inherited from C.

> 在C++03及更早版本中，“auto”关键字作为[存储类说明符]具有完全不同的含义(http://stackoverflow.com/documentation/c%2b%2b/9225/storage-类说明符/228629/auto）。



## Template Type Deduction


Template Generic Syntax

```cpp
template<typename T>
void f(ParamType param);

f(expr);

```


Case 1: `ParamType` is a Reference or Pointer, but not a Universal or Forward Reference. In this case type deduction works this way. The compiler ignores the reference part if it exists in `expr`. The compiler then pattern-matches `expr`'s type against `ParamType` to determing `T`.

> 情况1：“ParamType”是引用或指针，但不是通用引用或正向引用。在这种情况下，类型推导是这样工作的。如果引用部分存在于“expr”中，编译器将忽略该引用部分。然后编译器将“expr”的类型与“ParamType”进行模式匹配，以确定“T”。

```cpp
template<typename T>
void f(T& param);      //param is a reference

int x = 27;            // x is an int
const int cx = x;      // cx is a const int
const int& rx = x;     // rx is a reference to x as a const int

f(x);                  // T is int, param's type is int&
f(cx);                 // T is const int, param's type is const int&
f(rx);                 // T is const int, param's type is const int&

```


Case 2: `ParamType` is a Universal Reference or Forward Reference. In this case type deduction is the same as in case 1 if the `expr` is an rvalue. If `expr` is an lvalue, both `T` and `ParamType` are deduced to be lvalue references.

> 情况2：“ParamType”是通用引用或正向引用。在这种情况下，如果“expr”是右值，则类型推导与情况1相同。如果“expr”是左值，则“T”和“ParamType”都被推导为左值引用。

```cpp
template<typename T>
void f(T&& param);     // param is a universal reference

int x = 27;            // x is an int
const int cx = x;      // cx is a const int
const int& rx = x;     // rx is a reference to x as a const int

f(x);                  // x is lvalue, so T is int&, param's type is also int&
f(cx);                 // cx is lvalue, so T is const int&, param's type is also const int&
f(rx);                 // rx is lvalue, so T is const int&, param's type is also const int&
f(27);                 // 27 is rvalue, so T is int, param's type is therefore int&&

```


Case 3: `ParamType` is Neither a Pointer nor a Reference. If `expr` is a reference the reference part is ignored. If `expr` is const that is ignored as well. If it is volatile that is also ignored when deducing T's type.

> 情况3：“ParamType”既不是指针也不是引用。如果“expr”是引用，则忽略引用部分。如果“expr”是const，也会被忽略。如果它是易失性的，那么在推导T的类型时也会忽略它。

```cpp
template<typename T>
void f(T param);       // param is now passed by value

int x = 27;            // x is an int
const int cx = x;      // cx is a const int
const int& rx = x;     // rx is a reference to x as a const int

f(x);                  // T's and param's types are both int
f(cx);                 // T's and param's types are again both int
f(rx);                 // T's and param's types are still both int

```



#### Remarks



In November 2014, the C++ Standardization Committee adopted proposal N3922, which eliminates the special type deduction rule for auto and braced initializers using direct initialization syntax. This is not part of the C++ standard but has been implemented by some compilers.

> 2014年11月，C++标准化委员会通过了N3922提案，该提案取消了使用直接初始化语法的自动和支撑初始化程序的特殊类型推导规则。这不是C++标准的一部分，但已经由一些编译器实现了。

