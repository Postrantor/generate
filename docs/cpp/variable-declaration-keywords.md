---
tip: translate by baidu@2023-10-25 08:36:27
---
---
metaTitle: "C++ | Variable Declaration Keywords"
description: "decltype, const, signed, unsigned, volatile"
---

# Variable Declaration Keywords




## decltype


Yields the type of its operand, which is not evaluated.

<li>

If the operand `e` is a name without any additional parentheses, then `decltype(e)` is the **declared type** of `e`.

> 如果操作数“e”是一个没有任何附加括号的名称，则“decltype（e）”是“e”的**声明类型**。

```cpp
int x = 42;
std::vector<decltype(x)> v(100, x); // v is a vector<int>

```


</li>
<li>

If the operand `e` is a class member access without any additional parentheses, then `decltype(e)` is the **declared type** of the member accessed.

> 如果操作数“e”是没有任何附加括号的类成员访问，则“decltype（e）”是所访问成员的**声明类型**。

```cpp
struct S {
    int x = 42;
};
const S s;
decltype(s.x) y; // y has type int, even though s.x is const

```


</li>
<li>

In all other cases, `decltype(e)` yields both the type and the [value category](http://stackoverflow.com/documentation/c%2b%2b/763/value-categories) of the expression `e`, as follows:

> 在所有其他情况下，“decltype（e）”同时产生类型和[值类别](http://stackoverflow.com/documentation/c%2b%2b/763/value-类别），如下所示：
<ul>
- If `e` is an lvalue of type `T`, then `decltype(e)` is `T&`.
- If `e` is an xvalue of type `T`, then `decltype(e)` is `T&&`.
- If `e` is a prvalue of type `T`, then `decltype(e)` is `T`.

This includes the case with extraneous parentheses.

```cpp
int f() { return 42; }
int& g() { static int x = 42; return x; }
int x = 42;
decltype(f()) a = f(); // a has type int
decltype(g()) b = g(); // b has type int&
decltype((x)) c = x;   // c has type int&, since x is an lvalue

```


The special form `decltype(auto)` deduces the type of a variable from its initializer or the return type of a function from the `return` statements in its definition, using the type deduction rules of `decltype` rather than those of `auto`.

> 特殊形式“decltype（auto）”从其初始值设定项推导变量的类型，或从其定义中的“return”语句推导函数的返回类型，使用“declttype”而不是“auto”的类型推导规则。

```cpp
const int x = 123;
auto y = x;           // y has type int
decltype(auto) z = x; // z has type const int, the declared type of x

```



## const



A type specifier; when applied to a type, produces the const-qualified version of the type. See [const keyword](http://stackoverflow.com/documentation/c%2b%2b/2386/const-keyword) for details on the meaning of `const`.

> 类型说明符；应用于类型时，生成该类型的const限定版本。请参阅[const关键字](http://stackoverflow.com/documentation/c%2b%2b/2386/const-关键字），以获取有关“const”含义的详细信息。

```cpp
const int x = 123;
x = 456;    // error
int& r = x; // error

struct S {
    void f();
    void g() const;
};
const S s;
s.f(); // error
s.g(); // OK

```



## signed


A keyword that is part of certain integer type names.

- When used alone, `int` is implied, so that `signed`, `signed int`, and `int` are the same type.

- When combined with `char`, yields the type `signed char`, which is a different type from `char`, even if `char` is also signed. `signed char` has a range that includes at least -127 to +127, inclusive.

> -当与“char”组合时，会产生类型“signed char”，这与“char'”是不同的类型，即使“char”也有符号`带符号字符“的范围至少包括-127到+127（包括-127）。

- When combined with `short`, `long`, or `long long`, it is redundant, since those types are already signed.

> -当与“short”、“long”或“long-long”组合时，它是多余的，因为这些类型已经签名。
- `signed` cannot be combined with `bool`, `wchar_t`, `char16_t`, or `char32_t`.

Example:

```cpp
signed char celsius_temperature;
std::cin >> celsius_temperature;
if (celsius_temperature < -35) {
    std::cout << "cold day, eh?\n";
}

```



## unsigned


A type specifier that requests the unsigned version of an integer type.

- When used alone, `int` is implied, so `unsigned` is the same type as `unsigned int`.

- The type `unsigned char` is different from the type `char`, even if `char` is unsigned. It can hold integers up to at least 255.

> -类型“unsigned char”与类型“char”不同，即使“char”是无符号的。它可以容纳至少255个整数。

- `unsigned` can also be combined with `short`, `long`, or `long long`. It cannot be combined with `bool`, `wchar_t`, `char16_t`, or `char32_t`.

> -“unsigned”也可以与“short”、“long”或“long-long”组合。它不能与“bool”、“wchar_t”、“char16_t”或“char32_t”组合。

Example:

```cpp
char invert_case_table[256] = { ..., 'a', 'b', 'c', ..., 'A', 'B', 'C', ... };
char invert_case(char c) {
    unsigned char index = c;
    return invert_case_table[index];
    // note: returning invert_case_table[c] directly does the
    // wrong thing on implementations where char is a signed type
}

```



## volatile



A type qualifier; when applied to a type, produces the volatile-qualified version of the type. Volatile qualification plays the same role as `const` qualification in the type system, but `volatile` does not prevent objects from being modified; instead, it forces the compiler to treat all accesses to such objects as side effects.

> 类型限定符；当应用于类型时，生成该类型的volatile限定版本。Volatile限定在类型系统中扮演着与“const”限定相同的角色，但“Volatile”并不阻止对象被修改；相反，它强制编译器将对此类对象的所有访问视为副作用。


In the example below, if `memory_mapped_port` were not volatile, the compiler could optimize the function so that it performs only the final write, which would be incorrect if `sizeof(int)` is greater than 1. The `volatile` qualification forces it to treat all `sizeof(int)` writes as different side effects and hence perform all of them (in order).

> 在下面的例子中，如果“memory_mapped_port”不是易失性的，编译器可以优化函数，使其只执行最终写入，如果“sizeof（int）”大于1，则这是不正确的。“volatile”限定迫使它将所有“sizeof（int）”写入视为不同的副作用，并因此执行所有副作用（按顺序）。

```cpp
extern volatile char memory_mapped_port;
void write_to_device(int x) {
    const char* p = reinterpret_cast<const char*>(&x);
    for (int i = 0; i < sizeof(int); i++) {
        memory_mapped_port = p[i];
    }
}

```

