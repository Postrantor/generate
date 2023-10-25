---
tip: translate by baidu@2023-10-25 08:20:47
---
---
metaTitle: "C++ | Basic Type Keywords"
description: "int, bool, char, char16_t, char32_t, float, double, long, short, void, wchar_t"
---

# Basic Type Keywords



## int


Denotes a signed integer type with "the natural size suggested by the

architecture of the execution environment", whose range includes at least -32767 to +32767, inclusive.

> 执行环境的体系结构”，其范围至少包括-32767到+32767（含）。

```cpp
int x = 2;
int y = 3;
int z = x + y;

```


Can be combined with `unsigned`, `short`, `long`, and `long long` (q.v.) in order to yield other integer types.

> 可以与“unsigned”、“short”、“long”和“long-long”（q.v.）组合以生成其他整数类型。



## bool


An integer type whose value can be either `true` or `false`.

```cpp
bool is_even(int x) {
    return x%2 == 0;
}
const bool b = is_even(47); // false

```



## char



An integer type which is "large enough to store any member of the implementation’s basic

> 一个整数类型，它“足够大，可以存储实现的基本

character set". It is implementation-defined whether `char` is signed (and has a range of at least -127 to +127, inclusive) or unsigned (and has a range of at least 0 to 255, inclusive).

> 字符集”。它由实现定义“char”是有符号的（范围至少为-127到+127，包括）还是无符号的（至少为0到255，包括）。

```cpp
const char zero = '0';
const char one = zero + 1;
const char newline = '\n';
std::cout << one << newline; // prints 1 followed by a newline

```



## char16_t



An unsigned integer type with the same size and alignment as `uint_least16_t`, which is therefore large enough to hold a UTF-16 code unit.

> 一种无符号整数类型，其大小和对齐方式与“uint_list16_t”相同，因此其大小足以容纳一个UTF-16代码单元。

```cpp
const char16_t message[] = u"你好，世界\n";            // Chinese for "hello, world\n"
std::cout << sizeof(message)/sizeof(char16_t) << "\n"; // prints 7

```



## char32_t



An unsigned integer type with the same size and alignment as `uint_least32_t`, which is therefore large enough to hold a UTF-32 code unit.

> 一种无符号整数类型，其大小和对齐方式与“uint_list32_t”相同，因此其大小足以容纳一个UTF-32代码单元。

```cpp
const char32_t full_house[] = U"🂣🂳🂨🂸🃈";               // non-BMP characters
std::cout << sizeof(full_house)/sizeof(char32_t) << "\n"; // prints 6

```



## float



A floating point type. Has the narrowest range out of the three floating point types in C++.

> 浮点类型。在C++中的三种浮点类型中，具有最窄的范围。

```cpp
float area(float radius) {
    const float pi = 3.14159f;
    return pi*radius*radius;
}

```



## double



A floating point type. Its range includes that of `float`. When combined with `long`, denotes the `long double` floating point type, whose range includes that of `double`.

> 浮点类型。其范围包括“float”的范围。与“long”组合时，表示“long double”浮点类型，其范围包括“double”的范围。

```cpp
double area(double radius) {
    const double pi = 3.141592653589793;
    return pi*radius*radius;
}

```



## long



Denotes a signed integer type that is at least as long as `int`, and whose range includes at least -2147483647 to +2147483647, inclusive (that is, -(2^31 - 1) to +(2^31 - 1)). This type can also be written as `long int`.

> 表示至少与“int”一样长的带符号整数类型，其范围至少包括-2147483647到+214783647（包括-214743647）（即-（2^31-1）到+（2^31-1））。这种类型也可以写成“long int”。

```cpp
const long approx_seconds_per_year = 60L*60L*24L*365L;

```


The combination `long double` denotes a floating point type, which has the widest range out of the three floating point types.

> 组合“长双”表示浮点类型，在三种浮点类型中具有最宽的范围。

```cpp
long double area(long double radius) {
    const long double pi = 3.1415926535897932385L;
    return pi*radius*radius;
}

```


When the `long` specifier occurs twice, as in `long long`, it denotes a signed integer type that is at least as long as `long`, and whose range includes at least -9223372036854775807 to +9223372036854775807, inclusive (that is, -(2^63 - 1) to +(2^63 - 1)).

> 当“long”说明符出现两次时，如在“long-long”中，它表示一个有符号整数类型，该类型的长度至少与“long“一样长，并且其范围至少包括9223372036854775807到9223372036864775807（包括在内）（即-（2^63-1）到+（2^63/1））。

```cpp
// support files up to 2 TiB
const long long max_file_size = 2LL << 40;

```



## short



Denotes a signed integer type that is at least as long as `char`, and whose range includes at least -32767 to +32767, inclusive. This type can also be written as `short int`.

> 表示长度至少与“char”一样长的带符号整数类型，其范围至少包括-32767到+32767（含）。这种类型也可以写成“short int”。

```cpp
// (during the last year)
short hours_worked(short days_worked) {
    return 8*days_worked;
}

```



## void



An incomplete type; it is not possible for an object to have type `void`, nor are there arrays of `void` or references to `void`. It is used as the return type of functions that do not return anything.

> 不完整的类型；对象不可能具有“void”类型，也不存在“void”数组或对“void”的引用。它被用作不返回任何内容的函数的返回类型。


Moreover, a function may redundantly be declared with a single parameter of type `void`; this is equivalent to declaring a function with no parameters (e.g. `int main()` and `int main(void)` declare the same function). This syntax is allowed for compatibility with C (where function declarations have a different meaning than in C++).

> 此外，函数可以用“void”类型的单个参数冗余地声明；这相当于声明一个没有参数的函数（例如，“int main（）”和“int main（void）”声明相同的函数）。为了与C兼容，允许使用此语法（其中函数声明与C++中的含义不同）。


The type `void*` ("pointer to `void`") has the property that any object pointer can be converted to it and back and result in the same pointer. This feature makes the type `void*` suitable for certain kinds of (type-unsafe) type-erasing interfaces, for example for generic contexts in C-style APIs (e.g. `qsort`, `pthread_create`).

> 类型“void*”（“指向“void”的指针”）具有这样的属性，即任何对象指针都可以转换为它并返回，从而产生相同的指针。此功能使类型“void*”适用于某些类型（类型不安全）的类型擦除接口，例如C风格API中的通用上下文（例如“qsort”、“pthread_create”）。


Any expression may be converted to an expression of type `void`; this is called a **discarded-value expression**:

> 任何表达式都可以转换为“void”类型的表达式；这被称为**丢弃值表达式**：

```cpp
static_cast<void>(std::printf("Hello, %s!\n", name));  // discard return value

```


This may be useful to signal explicitly that the value of an expression is not of interest and that the expression is to be evaluated for its side effects only.

> 这可能有助于明确表示表达式的值不感兴趣，并且只评估该表达式的副作用。



## wchar_t



An integer type large enough to represent all characters of the largest supported extended character set, also known as the wide-character set. (It is not portable to make the assumption that `wchar_t` uses any particular encoding, such as UTF-16.)

> 一种整数类型，其大小足以表示支持的最大扩展字符集（也称为宽字符集）的所有字符。（假设“wchar_t”使用任何特定编码（如UTF-16）是不可移植的。）


It is normally used when you need to store characters over ASCII 255 , as it has a greater size than the character type `char`.

> 当您需要存储ASCII 255以上的字符时，通常会使用它，因为它的大小大于字符类型“char”。

```cpp
const wchar_t message_ahmaric[] = L"ሰላም ልዑል\n"; //Ahmaric for "hello, world\n"
const wchar_t message_chinese[] = L"你好，世界\n";// Chinese for "hello, world\n"
const wchar_t message_hebrew[]  = L"שלום עולם\n"; //Hebrew for "hello, world\n"
const wchar_t message_russian[] = L"Привет мир\n";  //Russian for "hello, world\n"
const wchar_t message_tamil[]   = L"ஹலோ உலகம்\n"; //Tamil for "hello, world\n"

```

