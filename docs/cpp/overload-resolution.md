---
tip: translate by baidu@2023-10-25 08:29:10
---
---

metaTitle: "C++ | Overload resolution"
description: "Categorization of argument to parameter cost, Exact match, Name lookup and access checking, Overloading on Forwarding Reference, Arithmetic promotions and conversions, Overloading on constness and volatility, Steps of Overload Resolution, Overloading within a class hierarchy"
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# Overload resolution

## Categorization of argument to parameter cost

Overload resolution partitions the cost of passing an argument to a parameter into one of four different categorizes, called "sequences". Each sequence may include zero, one or several conversions

> 重载解析将传递参数到参数的成本划分为四个不同的类别之一，称为“序列”。每个序列可以包括零个、一个或多个转换

<li>
Standard conversion sequence

```cpp
void f(int a); f(42);

```

</li>
<li>
User defined conversion sequence

```cpp
void f(std::string s); f("hello");

```

</li>
<li>
Ellipsis conversion sequence

```cpp
void f(...); f(42);

```

</li>
<li>
List initialization sequence

```cpp
void f(std::vector<int> v); f({1, 2, 3});

```

</li>

The general principle is that Standard conversion sequences are the cheapest, followed by user defined conversion sequences, followed by ellipsis conversion sequences.

> 一般原则是标准转换序列最便宜，其次是用户定义的转换序列，然后是省略号转换序列。

A special case is the list initialization sequence, which does not constitute a conversion (an initializer list is not an expression with a type). Its cost is determined by defining it to be equivalent to one of the other three conversion sequences, depending on the parameter type and form of initializer list.

> 一种特殊情况是列表初始化序列，它不构成转换（初始化器列表不是具有类型的表达式）。它的代价是通过定义它等价于其他三个转换序列中的一个来确定的，这取决于参数类型和初始值设定项列表的形式。

## Exact match

An overload without conversions needed for parameter types or only conversions needed between types that are still considered exact matches is preferred over an overload that requires other conversions in order to call.

> 与需要其他转换才能调用的重载相比，不需要参数类型所需的转换或只需要在仍被视为完全匹配的类型之间进行转换的重载更可取。

```cpp
void f(int x);
void f(double x);
f(42); // calls f(int)

```

When an argument binds to a reference to the same type, the match is considered to not require a conversion even if the reference is more cv-qualified.

> 当一个参数绑定到对同一类型的引用时，即使引用更符合 cv，匹配也被认为不需要转换。

```cpp
void f(int& x);
void f(double x);
int x = 42;
f(x); // argument type is int; exact match with int&

void g(const int& x);
void g(int x);
g(x); // ambiguous; both overloads give exact match

```

For the purposes of overload resolution, the type "array of `T`" is considered to match exactly with the type "pointer to `T`", and the function type `T` is considered to match exactly with the function pointer type `T*`, even though both require conversions.

> 为了解决过载问题，类型“`T` 的数组”被认为与类型“指向 `T'的指针”完全匹配，函数类型“T` 被认为与函数指针类型“T*'完全匹配，即使两者都需要转换。

```cpp
void f(int* p);
void f(void* p);

void g(int* p);
void g(int (&p)[100]);

int a[100];
f(a); // calls f(int*); exact match with array-to-pointer conversion
g(a); // ambiguous; both overloads give exact match

```

## Name lookup and access checking

Overload resolution occurs **after** name lookup. This means that a better-matching function will not be selected by overload resolution if it loses name lookup:

> 重载解析发生在**名称查找之后**。这意味着，如果重载解析丢失了名称查找，则不会选择更好的匹配函数：

```cpp
void f(int x);
struct S {
    void f(double x);
    void g() { f(42); } // calls S::f because global f is not visible here,
                        // even though it would be a better match
};

```

Overload resolution occurs **before** access checking. An inaccessible function might be selected by overload resolution if it is a better match than an accessible function.

> 过载解决发生在**访问检查之前**。如果不可访问函数比可访问函数更匹配，则可以通过重载解析选择不可访问的函数。

```cpp
class C {
  public:
    static void f(double x);
  private:
    static void f(int x);
};
C::f(42); // Error! Calls private C::f(int) even though public C::f(double) is viable.

```

Similarly, overload resolution happens without checking whether the resulting call is well-formed with regards to `explicit`:

> 类似地，重载解析在不检查结果调用是否与“显式”有关的情况下进行：

```cpp
struct X {
    explicit X(int );
    X(char );
};

void foo(X );
foo({4}); // X(int) is better much, but expression is 
          // ill-formed because selected constructor is explicit

```

## Overloading on Forwarding Reference

You must be very careful when providing a forwarding reference overload as it may match too well:

> 提供转发引用重载时必须非常小心，因为它可能匹配得太好：

```cpp
struct A {
    A() = default;           // #1
    A(A const& ) = default;  // #2

    template <class T>
    A(T&& );                 // #3
};

```

The intent here was that `A` is copyable, and that we have this other constructor that might initialize some other member. However:

> 这里的意图是“A”是可复制的，并且我们有另一个构造函数，它可能会初始化其他成员。但是：

```cpp
A a;     // calls #1
A b(a);  // calls #3!

```

There are two viable matches for the construction call:

```cpp
A(A const& ); // #2
A(A& );       // #3, with T = A&

```

Both are Exact Matches, but `#3` takes a reference to a less **cv**-qualified object than `#2` does, so it has the better standard conversion sequence and is the best viable function.

> 两者都是精确匹配，但“#3”引用了一个比“#2”更少的 **cv***限定对象，因此它具有更好的标准转换序列，是最好的可行函数。

The solution here is to always constrain these constructors (e.g. using SFINAE):

> 这里的解决方案是始终约束这些构造函数（例如，使用 SFINAE）：

```cpp
template <class T,
    class = std::enable_if_t<!std::is_convertible<std::decay_t<T>*, A*>::value>
    >
A(T&& );

```

The type trait here is to exclude any `A` or class publicly and unambiguously derived from `A` from consideration, which would make this constructor ill-formed in the example described earlier (and hence removed from the overload set). As a result, the copy constructor is invoked - which is what we wanted.

> 这里的类型特征是排除任何公开且明确地从“A”派生的“A”或类，这将使该构造函数在前面描述的示例中格式不正确（因此从重载集中删除）。结果，复制构造函数被调用了——这正是我们想要的。

## Arithmetic promotions and conversions

Converting an integer type to the corresponding promoted type is better than converting it to some other integer type.

> 将整数类型转换为相应的提升类型比将其转换为其他整数类型要好。

```cpp
void f(int x);
void f(short x);
signed char c = 42;
f(c); // calls f(int); promotion to int is better than conversion to short
short s = 42;
f(s); // calls f(short); exact match is better than promotion to int

```

Promoting a `float` to `double` is better than converting it to some other floating point type.

> 将“float”提升为“double”比将其转换为其他浮点类型要好。

```cpp
void f(double x);
void f(long double x);
f(3.14f); // calls f(double); promotion to double is better than conversion to long double

```

Arithmetic conversions other than promotions are neither better nor worse than each other.

> 除了促销之外，算术转换既不好也不差。

```cpp
void f(float x);
void f(long double x);
f(3.14); // ambiguous

void g(long x);
void g(long double x);
g(42); // ambiguous
g(3.14); // ambiguous

```

Therefore, in order to ensure that there will be no ambiguity when calling a function `f` with either integral or floating-point arguments of any standard type, a total of eight overloads are needed, so that for each possible argument type, either an overload matches exactly or the unique overload with the promoted argument type will be selected.

> 因此，为了确保在使用任何标准类型的整数或浮点参数调用函数“f”时不会出现歧义，总共需要八个重载，以便对于每个可能的参数类型，要么选择一个完全匹配的重载，要么选择具有提升参数类型的唯一重载。

```cpp
void f(int x);
void f(unsigned int x);
void f(long x);
void f(unsigned long x);
void f(long long x);
void f(unsigned long long x);
void f(double x);
void f(long double x);

```

## Overloading on constness and volatility

Passing a pointer argument to a `T*` parameter, if possible, is better than passing it to a `const T*` parameter.

> 如果可能的话，将指针参数传递给“T*”参数比将其传递给“const T*”的参数要好。

```cpp
struct Base {};
struct Derived : Base {};
void f(Base* pb);
void f(const Base* pb);
void f(const Derived* pd);
void f(bool b);

Base b;
f(&b); // f(Base*) is better than f(const Base*)
Derived d;
f(&d); // f(const Derived*) is better than f(Base*) though;
       // constness is only a "tie-breaker" rule

```

Likewise, passing an argument to a `T&` parameter, if possible, is better than passing it to a `const T&` parameter, even if both have exact match rank.

> 同样，如果可能的话，将一个参数传递给“T&”参数比将其传递给“const T&”参数要好，即使两者的秩都完全匹配。

```cpp
void f(int& r);
void f(const int& r);
int x;
f(x); // both overloads match exactly, but f(int&) is still better
const int y = 42;
f(y); // only f(const int&) is viable

```

This rule applies to const-qualified member functions as well, where it is important for allowing mutable access to non-const objects and immutable access to const objects.

> 此规则也适用于 const 限定的成员函数，在这些函数中，允许对非 const 对象进行可变访问和对 const 对象的不可变访问非常重要。

```cpp
class IntVector {
  public:
    // ...
    int* data() { return m_data; }
    const int* data() const { return m_data; }
  private:
    // ...
    int* m_data;
};
IntVector v1;
int* data1 = v1.data();       // Vector::data() is better than Vector::data() const;
                              // data1 can be used to modify the vector's data
const IntVector v2;
const int* data2 = v2.data(); // only Vector::data() const is viable;
                              // data2 can't be used to modify the vector's data

```

In the same way, a volatile overload will be less preferred than a non-volatile overload.

> 以同样的方式，易失性过载将不如非易失性超载更可取。

```cpp
class AtomicInt {
  public:
    // ...
    int load();
    int load() volatile;
  private:
    // ...
};
AtomicInt a1;
a1.load(); // non-volatile overload preferred; no side effect
volatile AtomicInt a2;
a2.load(); // only volatile overload is viable; side effect
static_cast<volatile AtomicInt&>(a1).load(); // force volatile semantics for a1

```

## Steps of Overload Resolution

The steps of overload resolution are:

<li>

Find candidate functions via name lookup. Unqualified calls will perform both regular unqualified lookup as well as argument-dependent lookup (if applicable).

> 通过名称查找查找候选函数。非限定调用将执行常规非限定查找以及参数相关查找（如果适用）。

</li>
<li>

Filter the set of candidate functions to a set of **viable** functions. A viable function for which there exists an implicit conversion sequence between the arguments the function is called with and the parameters the function takes.

> 将一组候选函数过滤为一组**可行**函数。一种可行的函数，在调用该函数的参数和函数所采用的参数之间存在隐式转换序列。

```cpp
void f(char);          // (1)
void f(int ) = delete; // (2)
void f();              // (3)
void f(int& );         // (4)

f(4); // 1,2 are viable (even though 2 is deleted!) 
      // 3 is not viable because the argument lists don't match
      // 4 is not viable because we cannot bind a temporary to 
      //     a non-const lvalue reference

```

</li>
<li>

Pick the best viable candidate. A viable function `F1` is a better function than another viable function `F2` if the implicit conversion sequence for each argument in `F1` is not worse than the corresponding implicit conversion sequence in `F2`, and...:

> 选择最可行的候选人。如果“F1”中每个自变量的隐式转换序列不比“F2”中相应的隐式转化序列差，则可行函数“F1”是比另一个可行函数“F2”更好的函数，并且…：

3.1. For some argument, the implicit conversion sequence for that argument in `F1` is a better conversion sequence than for that argument in `F2`, or

> 3.1.对于某个自变量，“F1”中该自变量的隐式转换序列比“F2”中的该自变量的转换序列更好，或者

```cpp
void f(int );  // (1)
void f(char ); // (2)

f(4);  // call (1), better conversion sequence

```

3.2. In a user-defined conversion, the standard conversion sequence from the return of `F1` to the destination type is a better conversion sequence than that of the return type of `F2`, or

> 3.2 在用户定义的转换中，从“F1”的返回到目的类型的标准转换序列是比“F2”的返回类型更好的转换序列，或者

```cpp
struct A 
{
    operator int();
    operator double();
} a;

int i = a; // a.operator int() is better than a.operator double() and a conversion
float f = a; // ambiguous

```

3.3. In a direct reference binding, `F1` has the same kind of reference by `F2` is not, or

> 3.3.在直接引用绑定中，“F1”具有与“F2”不同的引用类型，或者

```cpp
struct A 
{
    operator X&();  // #1
    operator X&&(); // #2
};
A a;
X& lx = a;  // calls #1
X&& rx = a; // calls #2

```

3.4. `F1` is not a function template specialization, but `F2` is, or

```cpp
template <class T> void f(T ); // #1
void f(int );                  // #2

f(42); // calls #2, the non-template

```

3.5. `F1` and `F2` are both function template specializations, but `F1` is more specialized than `F2`.

> 3.5.“F1”和“F2”都是函数模板专业化，但“F1”比“F2”更专业。

```cpp
template <class T> void f(T );  // #1
template <class T> void f(T* ); // #2

int* p;
f(p); // calls #2, more specialized

```

</li>

The ordering here is significant. The better conversion sequence check happens before the template vs non-template check. This leads to a common error with overloading on forwarding reference:

> 这里的订单意义重大。与非模板检查相比，更好的转换序列检查发生在模板检查之前。这导致了一个常见的转发引用重载错误：

```cpp
struct A {
    A(A const& ); // #1
    
    template <class T>
    A(T&& );      // #2, not constrained
};

A a;
A b(a); // calls #2!
        // #1 is not a template but #2 resolves to
        // A(A& ), which is a less cv-qualified reference than #1
        // which makes it a better implicit conversion sequence

```

If there's no single best viable candidate at the end, the call is ambiguous:

> 如果最后没有一个最可行的候选人，那么这个决定是模棱两可的：

```cpp
void f(double ) { }
void f(float ) { }

f(42); // error: ambiguous

```

## Overloading within a class hierarchy

The following examples will use this class hierarchy:

```cpp
struct A { int m; };
struct B : A {};
struct C : B {};

```

The conversion from derived class type to base class type is preferred to user-defined conversions. This applies when passing by value or by reference, as well as when converting pointer-to-derived to pointer-to-base.

> 从派生类类型到基类类型的转换比用户定义的转换更可取。这适用于通过值或引用传递，以及将指向派生的指针转换为指向基的指针时。

```cpp
struct Unrelated {
    Unrelated(B b);
};
void f(A a);
void f(Unrelated u);
B b;
f(b); // calls f(A)

```

A pointer conversion from derived class to base class is also better than conversion to `void*`.

> 从派生类到基类的指针转换也比转换到“void*”要好。

```cpp
void f(A* p);
void f(void* p);
B b;
f(&b); // calls f(A*)

```

If there are multiple overloads within the same chain of inheritance, the most derived base class overload is preferred. This is based on a similar principle as virtual dispatch: the "most specialized" implementation is chosen. However, overload resolution always occurs at compile time and will never implicitly down-cast.

> 如果在同一继承链中有多个重载，则首选派生次数最多的基类重载。这是基于与虚拟调度类似的原理：选择“最专业”的实现。但是，重载解析总是在编译时发生，并且永远不会隐式降频。

```cpp
void f(const A& a);
void f(const B& b);
C c;
f(c); // calls f(const B&)
B b;
A& r = b;
f(r); // calls f(const A&); the f(const B&) overload is not viable

```

For pointers to members, which are contravariant with respect to the class, a similar rule applies in the opposite direction: the least derived derived class is preferred.

> 对于指向与类相反的成员的指针，类似的规则适用于相反的方向：首选派生最少的类。

```cpp
void f(int B::*p);
void f(int C::*p);
int A::*p = &A::m;
f(p); // calls f(int B::*)

```

#### Remarks

Overload resolution happens in several different situations

- Calls to named overloaded functions. The candidates are all the functions found by name lookup.
- Calls to class object. The candidates are usually all the overloaded function call operators of the class.

> -调用类对象。候选者通常是类的所有重载函数调用运算符。

- Use of an operator. The candidates are the overloaded operator functions at namespace scope, the overloaded operator functions in the left class object (if any) and the built-in operators.

> -运算符的使用。候选是命名空间范围内的重载运算符函数、左类对象中的重载运算符功能（如果有的话）和内置运算符。

<li>Overload resolution to find the correct conversion operator function or constructor to invoke for an initialization
<ul>

- For non-list direct initialization (`Class c(value)`), the candidates are constructors of `Class`.

> -对于非列表直接初始化（“Class c（value）”），候选为“Class”的构造函数。

- For non-list copy initialization (`Class c = value`) and for finding the user defined conversion function to invoke in a user defined conversion sequence. The candidates are the constructors of `Class` and if the source is a class object, its conversion operator functions.

> -用于非列表复制初始化（“Class c=value”），以及用于查找要在用户定义的转换序列中调用的用户定义转换函数。候选者是“Class”的构造函数，如果源是类对象，则其转换运算符将起作用。

- For initialization of a non-class from a class object (`Nonclass c = classObject`). The candidates are the conversion operator functions of the initializer object.

> -用于从类对象初始化非类（`Nonclass c=classObject`）。候选项是初始值设定项对象的转换运算符函数。

- For initializing a reference with a class object (`R &r = classObject`), when the class has conversion operator functions that yield values that can be bound directly to `r`. The candidates are such conversion operator functions.

> -用于初始化具有类对象（“R&R=classObject”）的引用，当类具有转换运算符函数时，该函数产生可以直接绑定到“R”的值。候选者就是这样的转换运算符函数。

- For list-initialization of a non-aggregate class object (`Class c{1, 2, 3}`), the candidates are the initializer list constructors for a first pass through overload resolution. If this doesn't find a viable candidate, a second pass through overload resolution is done, with the constructors of `Class` as candidates.

> -对于非聚合类对象（“class c｛1，2，3｝”）的列表初始化，候选是第一次通过重载解析的初始化器列表构造函数。如果找不到可行的候选者，则执行第二次通过重载解析，将“Class”的构造函数作为候选者。
