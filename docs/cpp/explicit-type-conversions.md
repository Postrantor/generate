---
tip: translate by baidu@2023-10-25 08:24:11
---
---

metaTitle: "C++ | Explicit type conversions"
description: "Casting away constness, C-style casting, Base to derived conversion, Conversion between pointer and integer, Type punning conversion, Conversion by explicit constructor or explicit conversion function, Implicit conversion, Enum conversions, Derived to base conversion for pointers to members, void* to T*"
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# Explicit type conversions

An expression can be **explicitly converted** or **cast** to type `T` using `dynamic_cast<T>`, `static_cast<T>`, `reinterpret_cast<T>`, or `const_cast<T>`, depending on what type of cast is intended.

> 表达式可以使用“dynamic_cast＜T＞”、“static_cast＜T>”、“interpret_cast＜T＞’”或“const_cast＜T＞”**显式转换**或**强制转换**为类型“T”，具体取决于所要转换的类型。

C++ also supports function-style cast notation, `T(expr)`, and C-style cast notation, `(T)expr`.

> C++ 还支持函数样式的转换表示法“T（expr）”和 C 样式的转换表达法“（T）expr”。

## Casting away constness

A pointer to a const object can be converted to a pointer to non-const object using the `const_cast` [keyword](http://stackoverflow.com/documentation/c%2B%2B/4891/keywords). Here we use `const_cast` to call a function that is not const-correct. It only accepts a non-const `char*` argument even though it never writes through the pointer:

> 可以使用 `const_cast`[关键字]将指向常量对象的指针转换为指向非常量对象的指针([http://stackoverflow.com/documentation/c%2b%2b/4891/keywords](http://stackoverflow.com/documentation/c%2B%2B/4891/keywords))。在这里，我们使用“const_cast”来调用一个常量不正确的函数。它只接受非常量的“char*”参数，即使它从不通过指针写入：

```cpp
void bad_strlen(char*);
const char* s = "hello, world!";
bad_strlen(s);                    // compile error
bad_strlen(const_cast<char*>(s)); // OK, but it's better to make bad_strlen accept const char*

```

`const_cast` to reference type can be used to convert a const-qualified lvalue into a non-const-qualified value.

`const_cast` is dangerous because it makes it impossible for the C++ type system to prevent you from trying to modify a const object. Doing so results in undefined behavior.

```cpp
const int x = 123;
int& mutable_x = const_cast<int&>(x);
mutable_x = 456; // may compile, but produces *undefined behavior*

```

## C-style casting

C-Style casting can be considered 'Best effort' casting and is named so as it is the only cast which could be used in C. The syntax for this cast is `(NewType)variable`.

> C 样式强制转换可以被视为“尽力而为”强制转换，并且其名称是因为它是 C 中唯一可以使用的强制转换。此强制转换的语法为“（NewType）variable”。

Whenever this cast is used, it uses one of the following c++ casts (in order):

> 每当使用此强制转换时，它都会使用以下 c++ 强制转换之一（按顺序）：

- `const_cast<NewType>(variable)`
- `static_cast<NewType>(variable)`
- `const_cast<NewType>(static_cast<const NewType>(variable))`
- `reinterpret_cast<const NewType>(variable)`
- `const_cast<NewType>(reinterpret_cast<const NewType>(variable))`

Functional casting is very similar, though as a few restrictions as the result of its syntax: `NewType(expression)`. As a result, only types without spaces can be cast to.

> 函数型转换非常相似，但由于其语法的原因有一些限制：`NewType（expression）`。因此，只有不带空格的类型才能强制转换为。

It's better to use new c++ cast, because s more readable and can be spotted easily anywhere inside a C++ source code and errors will be detected in compile-time, instead in run-time.

> 最好使用新的 c++ 转换，因为它更可读，并且可以很容易地在 c++ 源代码中的任何地方发现，并且错误将在编译时检测到，而不是在运行时检测到。

As this cast can result in unintended `reinterpret_cast`, it is often considered dangerous.

> 由于此强制转换可能会导致意外的“relpret_cast”，因此通常被认为是危险的。

## Base to derived conversion

A pointer to base class can be converted to a pointer to derived class using `static_cast`. `static_cast` does not do any run-time checking and can lead to undefined behaviour when the pointer does not actually point to the desired type.

> 可以使用“static_cast”将指向基类的指针转换为指向派生类的指针 `static_cast 不执行任何运行时检查，并且当指针实际上没有指向所需类型时，可能会导致未定义的行为。

```cpp
struct Base {};
struct Derived : Base {};
Derived d;
Base* p1 = &d;
Derived* p2 = p1;                        // error; cast required
Derived* p3 = static_cast<Derived*>(p1); // OK; p2 now points to Derived object
Base b;
Base* p4 = &b;
Derived* p5 = static_cast<Derived*>(p4); // undefined behaviour since p4 does not
                                         // point to a Derived object

```

Likewise, a reference to base class can be converted to a reference to derived class using `static_cast`.

> 同样，可以使用“static_cast”将对基类的引用转换为对派生类的引用。

```cpp
struct Base {};
struct Derived : Base {};
Derived d;
Base& r1 = d;
Derived& r2 = r1;                        // error; cast required
Derived& r3 = static_cast<Derived&>(r1); // OK; r3 now refers to Derived object

```

If the source type is polymorphic, `dynamic_cast` can be used to perform a base to derived conversion. It performs a run-time check and failure is recoverable instead of producing undefined behaviour. In the pointer case, a null pointer is returned upon failure. In the reference case, an exception is thrown upon failure of type `std::bad_cast` (or a class derived from `std::bad_cast`).

> 如果源类型是多态的，则可以使用“dynamic_cast”执行从基到派生的转换。它执行运行时检查，故障是可恢复的，而不是产生未定义的行为。在指针的情况下，失败时返回一个空指针。在引用的情况下，当类型为“std:：bad_cast”（或从“std：：bad_ast”派生的类）失败时会引发异常。

```cpp
struct Base { virtual ~Base(); }; // Base is polymorphic
struct Derived : Base {};
Base* b1 = new Derived;
Derived* d1 = dynamic_cast<Derived*>(b1); // OK; d1 points to Derived object
Base* b2 = new Base;
Derived* d2 = dynamic_cast<Derived*>(b2); // d2 is a null pointer

```

## Conversion between pointer and integer

An object pointer (including `void*`) or function pointer can be converted to an integer type using `reinterpret_cast`. This will only compile if the destination type is long enough. The result is implementation-defined and typically yields the numeric address of the byte in memory that the pointer pointers to.

> 对象指针（包括“void*”）或函数指针可以使用“interpret_cast”转换为整数类型。只有当目标类型足够长时，才会进行编译。结果是实现定义的，通常会产生指针指向的内存中字节的数字地址。

Typically, `long` or `unsigned long` is long enough to hold any pointer value, but this is not guaranteed by the standard.

> 通常，“long”或“unsigned long”的长度足以容纳任何指针值，但标准并不能保证这一点。

If the types `std::intptr_t` and `std::uintptr_t` exist, they are guaranteed to be long enough to hold a `void*` (and hence any pointer to object type). However, they are not guaranteed to be long enough to hold a function pointer.

> 如果类型“std:：intptr_t”和“std:”uintptr_t“存在，则保证它们足够长，可以容纳“void*”（因此可以容纳指向对象类型的任何指针）。但是，不能保证它们的长度足以容纳函数指针。

Similarly, `reinterpret_cast` can be used to convert an integer type into a pointer type. Again the result is implementation-defined, but a pointer value is guaranteed to be unchanged by a round trip through an integer type. The standard does not guarantee that the value zero is converted to a null pointer.

> 类似地，“interpret_cast”可以用于将整数类型转换为指针类型。同样，结果是实现定义的，但指针值保证通过整数类型的往返而保持不变。该标准不保证将值零转换为空指针。

```cpp
void register_callback(void (*fp)(void*), void* arg); // probably a C API
void my_callback(void* x) {
    std::cout << "the value is: " << reinterpret_cast<long>(x); // will probably compile
}
long x;
std::cin >> x;
register_callback(my_callback,
                  reinterpret_cast<void*>(x)); // hopefully this doesn't lose information...

```

## Type punning conversion

A pointer (resp. reference) to an object type can be converted to a pointer (resp. reference) to any other object type using `reinterpret_cast`. This does not call any constructors or conversion functions.

> 一个指向对象类型的指针（resp.reference）可以使用“interpret_cast”转换为指向任何其他对象类型的指示器（resp.reference）。这不会调用任何构造函数或转换函数。

```cpp
int x = 42;
char* p = static_cast<char*>(&x);      // error: static_cast cannot perform this conversion
char* p = reinterpret_cast<char*>(&x); // OK
*p = 'z';                              // maybe this modifies x (see below)

```

The result of `reinterpret_cast` represents the same address as the operand, provided that the address is appropriately aligned for the destination type. Otherwise, the result is unspecified.

> “interpret_cast”的结果表示与操作数相同的地址，前提是地址与目标类型适当对齐。否则，结果是未指定的。

```cpp
int x = 42;
char& r = reinterpret_cast<char&>(x);
const void* px = &x;
const void* pr = &r;
assert(px == pr); // should never fire

```

The result of `reinterpret_cast` is unspecified, except that a pointer (resp. reference) will survive a round trip from the source type to the destination type and back, as long as the destination type's alignment requirement is not stricter than that of the source type.

> “reinterpret_cast”的结果是未指定的，除了指针（resp.reference）将在从源类型到目标类型的往返过程中幸存下来，只要目标类型的对齐要求不比源类型的更严格。

```cpp
int x = 123;
unsigned int& r1 = reinterpret_cast<unsigned int&>(x);
int& r2 = reinterpret_cast<int&>(r1);
r2 = 456; // sets x to 456

```

On most implementations, `reinterpret_cast` does not change the address, but this requirement was not standardized until C++11.

> 在大多数实现中，“interpret_cast”不会更改地址，但这一要求直到 C++11 才标准化。

`reinterpret_cast` can also be used to convert from one pointer-to-data-member type to another, or one pointer-to-member-function type to another.

Use of `reinterpret_cast` is considered dangerous because reading or writing through a pointer or reference obtained using `reinterpret_cast` may trigger undefined behaviour when the source and destination types are unrelated.

> 使用“interpret_cast”被认为是危险的，因为当源类型和目的地类型不相关时，通过使用“interret_cast”获得的指针或引用进行读取或写入可能会触发未定义的行为。

## Conversion by explicit constructor or explicit conversion function

A conversion that involves calling an [explicit](http://stackoverflow.com/documentation/c%2B%2B/4891/keywords/18568/explicit) constructor or conversion function can't be done implicitly. We can request that the conversion be done explicitly using `static_cast`. The meaning is the same as that of a direct initialization, except that the result is a temporary.

> 涉及调用〔explicit〕的转换([http://stackoverflow.com/documentation/c%2b%2b/4891/keywords/18568/explicit](http://stackoverflow.com/documentation/c%2B%2B/4891/keywords/18568/explicit))构造函数或转换函数不能隐式完成。我们可以请求使用“static_cast”显式完成转换。其含义与直接初始化的含义相同，只是结果是临时的。

```cpp
class C {
    std::unique_ptr<int> p;
  public:
    explicit C(int* p) : p(p) {}
};
void f(C c);
void g(int* p) {
    f(p);                 // error: C::C(int*) is explicit
    f(static_cast<C>(p)); // ok
    f(C(p));              // equivalent to previous line
    C c(p); f(c);         // error: C is not copyable
}

```

## Implicit conversion

`static_cast` can perform any implicit conversion. This use of `static_cast` can occasionally be useful, such as in the following examples:

<li>

When passing arguments to an ellipsis, the "expected" argument type is not statically known, so no implicit conversion will occur.

> 将参数传递给省略号时，“预期”的参数类型不是静态已知的，因此不会发生隐式转换。

```cpp
const double x = 3.14;
printf("%d\n", static_cast<int>(x)); // prints 3
// printf("%d\n", x); // undefined behaviour; printf is expecting an int here
// alternative:
// const int y = x; printf("%d\n", y);

```

Without the explicit type conversion, a `double` object would be passed to the ellipsis, and undefined behaviour would occur.

> 如果没有显式的类型转换，“double”对象将被传递给省略号，并且将发生未定义的行为。

</li>
<li>

A derived class assignment operator can call a base class assignment operator like so:

> 派生类赋值运算符可以调用基类赋值运算符，如下所示：

```cpp
struct Base { /* ... */ };
struct Derived : Base {
    Derived& operator=(const Derived& other) {
        static_cast<Base&>(*this) = other;
        // alternative:
        // Base& this_base_ref = *this; this_base_ref = other;
    }
};

```

</li>

## Enum conversions

`static_cast` can convert from an integer or floating point type to an enumeration type (whether scoped or unscoped), and **vice versa.** It can also convert between enumeration types.

- The conversion from an unscoped enumeration type to an arithmetic type is an implicit conversion; it is possible, but not necessary, to use `static_cast`.

> -从无范围枚举类型到算术类型的转换是隐式转换；使用“static_cast”是可能的，但不是必须的。

<li>
When a scoped enumeration type is converted to an arithmetic type:
<ul>

- If the enum's value can be represented exactly in the destination type, the result is that value.

> -如果枚举的值可以在目标类型中精确表示，则结果就是该值。

- Otherwise, if the destination type is an integer type, the result is unspecified.
- Otherwise, if the destination type is a floating point type, the result is the same as that of converting to the underlying type and then to the floating point type.

> -否则，如果目标类型是浮点类型，则结果与转换为基础类型然后转换为浮点类型的结果相同。

Example:

```cpp
enum class Format {
    TEXT = 0,
    PDF = 1000,
    OTHER = 2000,
};
Format f = Format::PDF;
int a = f;                         // error
int b = static_cast<int>(f);       // ok; b is 1000
char c = static_cast<char>(f);     // unspecified, if 1000 doesn't fit into char
double d = static_cast<double>(f); // d is 1000.0... probably

```

<li>
When an integer or enumeration type is converted to an enumeration type:
<ul>

- If the original value is within the destination enum's range, the result is that value. Note that this value might be unequal to all enumerators.

> -如果原始值在目标枚举的范围内，则结果就是该值。请注意，此值可能不等于所有枚举器。

- Otherwise, the result is unspecified (<= C++14) or undefined (>= C++17).

Example:

```cpp
enum Scale {
    SINGLE = 1,
    DOUBLE = 2,
    QUAD = 4
};
Scale s1 = 1;                     // error
Scale s2 = static_cast<Scale>(2); // s2 is DOUBLE
Scale s3 = static_cast<Scale>(3); // s3 has value 3, and is not equal to any enumerator
Scale s9 = static_cast<Scale>(9); // unspecified value in C++14; UB in C++17

```

<li>

When a floating point type is converted to an enumeration type, the result is the same as converting to the enum's underlying type and then to the enum type.

> 将浮点类型转换为枚举类型时，结果与转换为枚举的基础类型，然后再转换为枚举型相同。

```cpp
enum Direction {
    UP = 0,
    LEFT = 1,
    DOWN = 2,
    RIGHT = 3,
};
Direction d = static_cast<Direction>(3.14); // d is RIGHT

```

</li>

## Derived to base conversion for pointers to members

A pointer to member of derived class can be converted to a pointer to member of base class using `static_cast`. The types pointed to must match.

> 可以使用“static_cast”将指向派生类成员的指针转换为指向基类成员的指针。指向的类型必须匹配。

If the operand is a null pointer to member value, the result is also a null pointer to member value.

> 如果操作数是指向成员值的空指针，则结果也是指向成员值。

Otherwise, the conversion is only valid if the member pointed to by the operand actually exists in the destination class, or if the destination class is a base or derived class of the class containing the member pointed to by the operand. `static_cast` does not check for validity. If the conversion is not valid, the behaviour is undefined.

> 否则，只有当操作数指向的成员实际存在于目标类中，或者目标类是包含操作数指向成员的类的基类或派生类时，转换才有效 `static_cast` 不检查有效性。如果转换无效，则行为未定义。

```cpp
struct A {};
struct B { int x; };
struct C : A, B { int y; double z; };
int B::*p1 = &B::x;
int C::*p2 = p1;                              // ok; implicit conversion
int B::*p3 = p2;                              // error
int B::*p4 = static_cast<int B::*>(p2);       // ok; p4 is equal to p1
int A::*p5 = static_cast<int A::*>(p2);       // undefined; p2 points to x, which is a member
                                              // of the unrelated class B
double C::*p6 = &C::z;
double A::*p7 = static_cast<double A::*>(p6); // ok, even though A doesn't contain z
int A::*p8 = static_cast<int A::*>(p6);       // error: types don't match

```

## void* to T*

In C++, `void*` cannot be implicitly converted to `T*` where `T` is an object type. Instead, `static_cast` should be used to perform the conversion explicitly. If the operand actually points to a `T` object, the result points to that object. Otherwise, the result is unspecified.

> 在 C++ 中，“void*”不能隐式转换为“T*”，其中“T”是对象类型。相反，应使用“static_cast”显式执行转换。如果操作数实际指向“T”对象，则结果指向该对象。否则，结果是未指定的。

Even if the operand does not point to a `T` object, as long as the operand points to a byte whose address is properly aligned for the type `T`, the result of the conversion points to the same byte.

> 即使操作数不指向“T”对象，只要操作数指向一个地址与类型“T”正确对齐的字节，转换结果也指向同一字节。

```cpp
// allocating an array of 100 ints, the hard way
int* a = malloc(100*sizeof(*a));                    // error; malloc returns void*
int* a = static_cast<int*>(malloc(100*sizeof(*a))); // ok
// int* a = new int[100];                           // no cast needed
// std::vector<int> a(100);                         // better

const char c = '!';
const void* p1 = &c;
const char* p2 = p1;                           // error
const char* p3 = static_cast<const char*>(p1); // ok; p3 points to c
const int* p4 = static_cast<const int*>(p1);   // unspecified in C++03;
                                               // possibly unspecified in C++11 if
                                               // alignof(int) > alignof(char)
char* p5 = static_cast<char*>(p1);             // error: casting away constness

```

#### Syntax

- **simple-type-specifier** `(` `)`
- **simple-type-specifier** `(` **expression-list**  `)`
- **simple-type-specifier** **braced-init-list**
- **typename-specifier** `(` `)`
- **typename-specifier** `(` **expression-list** `)`
- **typename-specifier** **braced-init-list**
- `dynamic_cast` `<` **type-id** `>` `(` **expression** `)`
- `static_cast` `<` **type-id** `>` `(` **expression** `)`
- `reinterpret_cast` `<` **type-id** `>` `(` **expression** `)`
- `const_cast` `<` **type-id** `>` `(` **expression** `)`
- `(` **type-id** `)` **cast-expression**

#### Remarks

All six cast notations have one thing in common:

- Casting to an lvalue reference type, as in `dynamic_cast<Derived&>(base)`, yields an lvalue. Therefore, when you want to do something with the same object but treat it as a different type, you would cast to an lvalue reference type.

> -强制转换为左值引用类型，如“dynamic_cast＜Derived&＞（base）”，会产生左值。因此，当您想对同一个对象执行某些操作，但将其视为不同的类型时，您将强制转换为左值引用类型。

- Casting to an rvalue reference type, as in `static_cast<string&&>(s)`, yields an rvalue.
- Casting to a non-reference type, as in `(int)x`, yields a prvalue, which may be thought of as a **copy** of the value being cast, but with a different type from the original.

> -强制转换为非引用类型，如“（int）x”，会产生一个 prvalue，它可以被认为是要强制转换的值的**副本**，但类型与原始类型不同。

The `reinterpret_cast` keyword is responsible for performing two different kinds of "unsafe" conversions:

> “interpret_cast”关键字负责执行两种不同类型的“不安全”转换：

- The ["type punning"](http://stackoverflow.com/documentation/c%2B%2B/3090/explicit-type-conversions/12169/type-punning-conversion) conversions, which can be used to access memory of one type as though it is of a different type.

> -[“类型双关语”]([http://stackoverflow.com/documentation/c%2b%2b/3090/explicit](http://stackoverflow.com/documentation/c%2B%2B/3090/explicit)-类型转换/12169/类型 punning 转换）转换，其可以被用来访问一种类型的存储器，就好像它是不同类型的一样。

- Conversions [between integer types and pointer types](http://stackoverflow.com/documentation/c%2B%2B/3090/explicit-type-conversions/13555/conversion-between-pointer-and-integer), in either direction.

> -转换[在整数类型和指针类型之间]([http://stackoverflow.com/documentation/c%2b%2b/3090/explicit](http://stackoverflow.com/documentation/c%2B%2B/3090/explicit)-类型转换/13555/指针和整数之间的转换）。

The `static_cast` keyword can perform a variety of different conversions:

<li>

[Base to derived](http://stackoverflow.com/documentation/c%2B%2B/3090/explicit-type-conversions/10518/base-to-derived-conversion) conversions

> [从基础到派生]([http://stackoverflow.com/documentation/c%2b%2b/3090/explicit](http://stackoverflow.com/documentation/c%2B%2B/3090/explicit)-类型转换/10518/基本到派生转换）转换

</li>
<li>

Any conversion that can be done by a direct initialization, including both implicit conversions and conversions that call an explicit constructor or conversion function. See [here](http://stackoverflow.com/documentation/c%2B%2B/3090/explicit-type-conversions/18732/implicit-conversion) and [here](http://stackoverflow.com/documentation/c%2B%2B/3090/explicit-type-conversions/18731/conversion-by-explicit-constructor-or-explicit-conversion-function) for more details.

> 可以通过直接初始化完成的任何转换，包括隐式转换和调用显式构造函数或转换函数的转换。参见[此处]([http://stackoverflow.com/documentation/c%2b%2b/3090/explicit](http://stackoverflow.com/documentation/c%2B%2B/3090/explicit)-类型转换/18732/隐式转换）和[此处]([http://stackoverflow.com/documentation/c%2b%2b/3090/explicit-type](http://stackoverflow.com/documentation/c%2B%2B/3090/explicit-type) conversions/18731/通过显式构造函数或显式转换函数进行的转换）。

</li>
<li>
To `void`, which discards the value of the expression.

```cpp
// on some compilers, suppresses warning about x being unused
static_cast<void>(x);

```

</li>
<li>

Between arithmetic and enumeration types, and between different enumeration types. See [enum conversions](http://stackoverflow.com/documentation/c%2B%2B/3090/explicit-type-conversions/18751/enum-conversions)

> 算术和枚举类型之间，以及不同枚举类型之间。请参阅[枚举转换]([http://stackoverflow.com/documentation/c%2b%2b/3090/explicit](http://stackoverflow.com/documentation/c%2B%2B/3090/explicit)-类型转换/18551/枚举转换）

</li>
<li>

From pointer to member of derived class, to pointer to member of base class. The types pointed to must match. See [derived to base conversion for pointers to members](http://stackoverflow.com/documentation/c%2B%2B/3090/explicit-type-conversions/18752/derived-to-base-conversion-for-pointers-to-members)

> 从指向派生类成员的指针，到指向基类成员的指针。指向的类型必须匹配。请参阅[指向成员的指针的派生到基的转换]([http://stackoverflow.com/documentation/c%2b%2b/3090/explicit](http://stackoverflow.com/documentation/c%2B%2B/3090/explicit)-类型转换/18752/指向成员的指针的派生到基的转换）

</li>
<li>

[`void*` to `T*`](http://stackoverflow.com/documentation/c%2B%2B/3090/explicit-type-conversions/18753/void-to-t).

> [`void*` 到 `T*`]([http://stackoverflow.com/documentation/c%2b%2b/3090/explicit](http://stackoverflow.com/documentation/c%2B%2B/3090/explicit)-类型转换/18753/void-to-t）。

</li>

- From an lvalue to an xvalue, as in `std::move`. See [move semantics](http://stackoverflow.com/documentation/c%2B%2B/2129/move-semantics).

> -从左值到 x 值，如“std:：move”。请参阅[移动语义]([http://stackoverflow.com/documentation/c%2b%2b/2129/move](http://stackoverflow.com/documentation/c%2B%2B/2129/move)-语义）。
