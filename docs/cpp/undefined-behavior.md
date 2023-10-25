---
tip: translate by baidu@2023-10-25 08:35:32
---
---

metaTitle: "C++ | Undefined Behavior"
description: "Reading or writing through a null pointer, Using an uninitialized local variable, Accessing an out-of-bounds index, Deleting a derived object via a pointer to a base class that doesn't have a virtual destructor., Extending the `std` or `posix` Namespace, No return statement for a function with a non-void return type, Integer division by zero, Accessing a dangling reference, Invalid pointer arithmetic, Signed Integer Overflow, Incorrect pairing of memory allocation and deallocation, Shifting by an invalid number of positions, Multiple non-identical definitions (the One Definition Rule), Modifying a const object, Modifying a string literal, Accessing an object as the wrong type, Overflow during conversion to or from floating point type, Returning from a [[noreturn]] function, Infinite template recursion, Floating point overflow, Calling (Pure) Virtual Members From Constructor Or Destructor, Invalid base-to-derived static cast, Function call through mismatched function pointer type, Access to nonexistent member through pointer to member, Invalid derived-to-base conversion for pointers to members, Destroying an object that has already been destroyed"
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# Undefined Behavior

What is undefined behavior (UB)? According to the ISO C++ Standard (§1.3.24, N4296), it is "behavior for which this International Standard imposes no requirements."

> 什么是未定义行为（UB）？根据 ISO C++ 标准（§1.3.24，N4296），这是“本国际标准没有要求的行为”

This means that when a program encounters UB, it is allowed to do whatever it wants. This often means a crash, but it may simply do nothing, [make demons fly out of your nose](http://catb.org/jargon/html/N/nasal-demons.html), or even **appear** to work properly!

> 这意味着当程序遇到 UB 时，它可以做任何它想做的事情。这通常意味着崩溃，但它可能根本没有任何作用，[让恶魔从你的鼻子里飞出来](http://catb.org/jargon/html/N/nasal-demons.html)，甚至**出现**以正常工作！

Needless to say, you should avoid writing code that invokes UB.

## Reading or writing through a null pointer

```cpp
int *ptr = nullptr;
*ptr = 1; // Undefined behavior

```

This is **undefined behavior**, because a null pointer does not point to any valid object, so there is no object at `*ptr` to write to.

> 这是**未定义的行为**，因为空指针不指向任何有效对象，因此在“*ptr”处没有要写入的对象。

Although this most often causes a segmentation fault, it is undefined and anything can happen.

> 尽管这通常会导致分割错误，但它是未定义的，任何事情都可能发生。

## Using an uninitialized local variable

```cpp
int a;
std::cout << a; // Undefined behavior!

```

This results in **undefined behavior**, because `a` is uninitialised.

It is often, incorrectly, claimed that this is because the value is "indeterminate", or "whatever value was in that memory location before".  However, it is the act of accessing the value of `a` in the above example that gives undefined behaviour.  In practice, printing a "garbage value" is a common symptom in this case, but that is only one possible form of undefined behaviour.

> 人们经常错误地声称这是因为该值“不确定”，或者“以前该内存位置中的任何值”。然而，正是访问上述示例中“a”的值的行为给出了未定义的行为。在实践中，打印“垃圾值”是这种情况下的常见症状，但这只是未定义行为的一种可能形式。

Although highly unlikely in practice (since it is reliant on specific hardware support) the compiler could equally well electrocute the programmer when compiling the code sample above.   With such a compiler and hardware support, such a response to undefined behaviour would markedly increase average (living) programmer understanding of the true meaning of undefined behaviour - which is that the standard places no constraint on the resultant behaviour.

> 尽管在实践中极不可能（因为它依赖于特定的硬件支持），但在编译上面的代码示例时，编译器同样可以电死程序员。有了这样的编译器和硬件支持，对未定义行为的这种响应将显著提高普通（活着的）程序员对未定义的行为的真正含义的理解——也就是说，标准对由此产生的行为没有限制。

Using an indeterminate value of `unsigned char` type does not produce undefined behavior if the value is used as:

> 如果将“unsigned char”类型的不确定值用作以下值，则使用该值不会产生未定义的行为：

- the second or third operand of the ternary conditional operator;
- the right operand of the built-in comma operator;
- the operand of a conversion to `unsigned char`;
- the right operand of the assignment operator, if the left operand is also of type `unsigned char`;

> -赋值运算符的右操作数，如果左操作数也是“unsigned char”类型；

- the initializer for an `unsigned char` object;

or if the value is discarded. In such cases, the indeterminate value simply propagates to the result of the expression, if applicable.

Note that a `static` variable is **always** zero-initialized (if possible):

> 请注意，“static”变量**始终**初始化为零（如果可能）：

```cpp
static int a;
std::cout << a; // Defined behavior, 'a' is 0

```

## Accessing an out-of-bounds index

It is **undefined behavior** to access an index that is out of bounds for an array (or standard library container for that matter, as they are all implemented using a **raw** array):

> 访问超出数组（或标准库容器，因为它们都是使用 **raw** 数组实现的）界限的索引是**未定义的行为**：

```

int array[] = {1, 2, 3, 4, 5};
 array[5] = 0;  // Undefined behavior

```

It is **allowed** to have a pointer pointing to the end of the array (in this case `array + 5`), you just can't dereference it, as it is not a valid element.

> **允许**有一个指向数组末尾的指针（在本例中为“array+5”），但不能取消引用它，因为它不是有效元素。

```

const int *end = array + 5;  // Pointer to one past the last index
 for (int *p = array; p != end; ++p)
   // Do something with `p`

```

In general, you're not allowed to create an out-of-bounds pointer. A pointer must point to an element within the array, or one past the end.

> 通常，不允许创建越界指针。指针必须指向数组中的某个元素，或者指向数组末尾之后的一个元素。

## Deleting a derived object via a pointer to a base class that doesn't have a virtual destructor.

```cpp
class base { };
class derived: public base { }; 

int main() {
    base* p = new derived();
    delete p; // The is undefined behavior!
}

```

In section [expr.delete] §5.3.5/3 the standard says that if `delete` is called on an object whose static type does not have a `virtual` destructor:

> 在第[expr.delete]§5.3.5/3 节中，标准规定，如果对静态类型没有“虚拟”析构函数的对象调用“delete”：

>

<p>if the static type of the object to be deleted is different from its

dynamic type, the static type shall be a base class of the dynamic type of the object to be deleted and the

> 动态类型，静态类型应为要删除的对象的动态类型的基类

static type shall have a virtual destructor or the behavior is undefined.</p>

> 静态类型应具有虚拟析构函数，或者行为未定义</p>

This is the case regardless of the question whether the derived class added any data members to the base class.

> 无论派生类是否向基类添加了任何数据成员，都会出现这种情况。

## Extending the `std` or `posix` Namespace

[The standard (17.6.4.2.1/1)](https://isocpp.org/files/papers/N3690.pdf) generally forbids extending the `std` namespace:

> [标准（17.6.4.2.1/1）](https://isocpp.org/files/papers/N3690.pdf)通常禁止扩展“std”命名空间：

>

The behavior of a C++ program is undefined if it adds declarations or definitions to namespace std or to a namespace within namespace std unless otherwise specified.

> 如果 C++ 程序向命名空间 std 或命名空间 std 中的命名空间添加声明或定义，则其行为是未定义的，除非另有指定。

The same goes for `posix` (17.6.4.2.2/1):

>

The behavior of a C++ program is undefined if it adds declarations or definitions to namespace posix or to a namespace within namespace posix unless otherwise specified.

> 如果 C++ 程序向命名空间 posix 或命名空间 posix 中的命名空间添加声明或定义，则其行为是未定义的，除非另有指定。

Consider the following:

```cpp
#include <algorithm>

namespace std
{
    int foo(){}
}

```

Nothing in the standard forbids `algorithm` (or one of the headers it includes) defining the same definition, and so this code would violate the [One Definition Rule](https://en.wikipedia.org/wiki/One_Definition_Rule).

> 标准中没有任何内容禁止定义相同定义的“算法”（或其包含的一个标头），因此此代码将违反[一个定义规则](https://en.wikipedia.org/wiki/One_Definition_Rule)。

So, in general, this is forbidden. There are [specific exceptions allowed](http://en.cppreference.com/w/cpp/language/extending_std), though. Perhaps most usefully, it is allowed to add specializations for user defined types. So, for example, suppose your code has

> 所以，总的来说，这是被禁止的。有[允许的特定例外](http://en.cppreference.com/w/cpp/language/extending_std)不过。也许最有用的是，允许为用户定义的类型添加专门化。因此，例如，假设您的代码

```cpp
class foo
{
    // Stuff
};

```

Then the following is fine

```cpp
namespace std
{
    template<>
    struct hash<foo>
    {
    public:
        size_t operator()(const foo &f) const;
    };
}

```

## No return statement for a function with a non-void return type

Omitting the `return` statement in a function which is has a return type that is not `void` is **undefined behavior**.

> 在返回类型不是“void”的函数中省略“return”语句是**未定义的行为**。

```cpp
int function() {  
    // Missing return statement
} 

int main() {
    function(); //Undefined Behavior
}

```

Most modern day compilers emit a warning at compile time for this kind of undefined behavior.

> 大多数现代编译器在编译时都会针对这种未定义的行为发出警告。

**Note:** `main` is the only exception to the rule. If `main` doesn't have a `return` statement, the compiler automatically inserts `return 0;` for you, so it can be safely left out.

> **注意：**“main”是该规则的唯一例外。如果“main”没有“return”语句，编译器会自动插入“return 0；”对你来说，这样它就可以安全地被排除在外。

## Integer division by zero

```cpp
int x = 5 / 0;    // Undefined behavior

```

Division by `0` is mathematically undefined, and as such it makes sense that this is undefined behavior.

> 除以“0”在数学上是未定义的，因此这是未定义行为是有道理的。

However:

```cpp
float x = 5.0f / 0.0f;   // x is +infinity

```

Most implementation implement IEEE-754, which defines floating point division by zero to return `NaN` (if numerator is `0.0f`), `infinity` (if numerator is positive) or `-infinity` (if numerator is negative).

> 大多数实现实现实现 IEEE-754，它定义浮点除以零以返回“NaN”（如果分子为“0.0f”）、“无穷大”（如果原子为正）或“-无穷大”。

## Accessing a dangling reference

It is illegal to access a reference to an object that has gone out of scope or been otherwise destroyed. Such a reference is said to be **dangling** since it no longer refers to a valid object.

> 访问对超出范围或以其他方式销毁的对象的引用是非法的。这样的引用被称为“悬空”，因为它不再引用有效对象。

```cpp
#include <iostream>
int& getX() {
    int x = 42;
    return x;
}
int main() {
    int& r = getX();
    std::cout << r << "\n";
}

```

In this example, the local variable `x` goes out of scope when `getX` returns. (Note that **lifetime extension** cannot extend the lifetime of a local variable past the scope of the block in which it is defined.) Therefore `r` is a dangling reference. This program has undefined behavior, although it may appear to work and print `42` in some cases.

> 在本例中，当“getX”返回时，局部变量“x”超出范围。（请注意，**生存期扩展**不能将局部变量的生存期扩展到定义它的块的范围之外。）因此，“r”是一个悬空引用。这个程序有未定义的行为，尽管在某些情况下它可能看起来可以工作并打印“42”。

## Invalid pointer arithmetic

The following uses of pointer arithmetic cause undefined behavior:

<li>

Addition or subtraction of an integer, if the result does not belong to the same array object as the pointer operand. (Here, the element one past the end is considered to still belong to the array.)

> 整数的加法或减法，如果结果与指针操作数不属于同一数组对象。（这里，超过末尾的元素被认为仍然属于数组。）

```cpp
int a[10];
int* p1 = &a[5];
int* p2 = p1 + 4; // ok; p2 points to a[9]
int* p3 = p1 + 5; // ok; p2 points to one past the end of a
int* p4 = p1 + 6; // UB
int* p5 = p1 - 5; // ok; p2 points to a[0]
int* p6 = p1 - 6; // UB
int* p7 = p3 - 5; // ok; p7 points to a[5]

```

</li>
<li>

Subtraction of two pointers if they do not both belong to the same array object. (Again, the element one past the end is considered to belong to the array.) The exception is that two null pointers may be subtracted, yielding 0.

> 如果两个指针不同时属于同一数组对象，则进行减法运算。（同样，超过末尾的元素被认为属于数组。）例外情况是，可以减去两个空指针，得到 0。

```cpp
int a[10];
int b[10];
int *p1 = &a[8], *p2 = &a[3];
int d1 = p1 - p2; // yields 5
int *p3 = p1 + 2; // ok; p3 points to one past the end of a
int d2 = p3 - p2; // yields 7
int *p4 = &b[0];
int d3 = p4 - p1; // UB

```

</li>
<li>
Subtraction of two pointers if the result overflows `std::ptrdiff_t`.
</li>
<li>
<p>Any pointer arithmetic where either operand's pointee type does not match the dynamic type of the object pointed to (ignoring cv-qualification). According to the standard, "[in] particular, a pointer to a base

class cannot be used for pointer arithmetic when the array contains objects of a derived class type."</p>

> 当数组包含派生类类型的对象时，类不能用于指针算术。“</p>

```cpp
struct Base { int x; };
struct Derived : Base { int y; };
Derived a[10];
Base* p1 = &a[1];           // ok
Base* p2 = p1 + 1;          // UB; p1 points to Derived
Base* p3 = p1 - 1;          // likewise
Base* p4 = &a[2];           // ok
auto p5 = p4 - p1;          // UB; p4 and p1 point to Derived
const Derived* p6 = &a[1];
const Derived* p7 = p6 + 1; // ok; cv-qualifiers don't matter

```

</li>

## Signed Integer Overflow

```cpp
int x = INT_MAX + 1;

// x can be anything -> Undefined behavior

```

>

If during the evaluation of an expression, the result is not mathematically defined or not in the range of representable values for its type, the behavior is undefined.

> 如果在表达式的求值过程中，结果没有在数学上定义，或者不在其类型的可表示值范围内，则行为是未定义的。

<sup>(C++11 Standard paragraph 5/4)</sup>

This is one of the more nasty ones, as it usually yields reproducible, non-crashing behavior so developers may be tempted to rely heavily on the observed behavior.

> 这是一种更令人讨厌的行为，因为它通常会产生可复制的、不崩溃的行为，因此开发人员可能会严重依赖观察到的行为。

On the other hand:

```cpp
unsigned int x = UINT_MAX + 1;

// x is 0

```

is well defined since:

>

Unsigned integers, declared unsigned, shall obey the laws of arithmetic modulo `2^n` where `n` is the number of bits in the value representation of that particular size of integer.

> 声明为无符号的无符号整数应遵循算术模“2^n”定律，其中“n”是该特定大小整数的值表示中的位数。

<sup>(C++11 Standard paragraph 3.9.1/4)</sup>

Sometimes compilers may exploit an undefined behavior and optimize

```cpp
signed int x ;
if(x > x + 1)
{
    //do something
}

```

Here since a signed integer overflow is not defined, compiler is free to assume

> 在这里，由于没有定义有符号整数溢出，编译器可以自由地假设
> that it may never happen and hence it can optimize away the "if" block

## Incorrect pairing of memory allocation and deallocation

An object can only be deallocated by `delete` if it was allocated by `new` and is not an array. If the argument to `delete` was not returned by `new` or is an array, the behavior is undefined.

> 如果对象是由“new”分配的并且不是数组，则只能由“delete”解除分配。如果“delete”的参数不是由“new”返回的，或者是一个数组，则行为是未定义的。

An object can only be deallocated by `delete[]` if it was allocated by `new` and is an array. If the argument to `delete[]` was not returned by `new` or is not an array, the behavior is undefined.

> 如果对象是由“new”分配的并且是一个数组，则只能由“delete[]”解除分配。如果“delete[]”的参数不是由“new”返回的，或者不是数组，则行为是未定义的。

If the argument to `free` was not returned by `malloc`, the behavior is undefined.

> 如果“malloc”没有返回“free”的参数，则行为是未定义的。

```cpp
int* p1 = new int;
delete p1;      // correct
// delete[] p1; // undefined
// free(p1);    // undefined

int* p2 = new int[10];
delete[] p2;    // correct
// delete p2;   // undefined
// free(p2);    // undefined

int* p3 = static_cast<int*>(malloc(sizeof(int)));
free(p3);       // correct
// delete p3;   // undefined
// delete[] p3; // undefined

```

Such issues can be avoided by completely avoiding `malloc` and `free` in C++ programs, preferring the standard library smart pointers over raw `new` and `delete`, and preferring `std::vector` and `std::string` over raw `new` and `delete[]`.

> 这样的问题可以通过完全避免 C++ 程序中的“malloc”和“free”来避免，更喜欢标准库智能指针而不是原始的“new”和“delete”，更喜欢“std:：vector”和“std：：string”而非原始的“new”和“delete[]”。

## Shifting by an invalid number of positions

For the built-in shift operator, the right operand must be nonnegative and strictly less than the bit width of the promoted left operand. Otherwise, the behavior is undefined.

> 对于内置移位运算符，右操作数必须是非负的，并且严格小于提升的左操作数的位宽。否则，行为是未定义的。

```cpp
const int a = 42;
const int b = a << -1; // UB
const int c = a << 0;  // ok
const int d = a << 32; // UB if int is 32 bits or less
const int e = a >> 32; // also UB if int is 32 bits or less
const signed char f = 'x';
const int g = f << 10; // ok even if signed char is 10 bits or less;
                       // int must be at least 16 bits

```

## Multiple non-identical definitions (the One Definition Rule)

If a class, enum, inline function, template, or member of a template has external linkage and is defined in multiple translation units, all definitions must be identical or the behavior is undefined according to the [One Definition Rule (ODR)](https://en.wikipedia.org/wiki/One_Definition_Rule).

> 如果类、枚举、内联函数、模板或模板的成员具有外部链接，并且在多个转换单元中定义，则所有定义必须相同，或者根据[一个定义规则（ODR）]未定义行为([https://en.wikipedia.org/wiki/One_Definition_Rule](https://en.wikipedia.org/wiki/One_Definition_Rule))。

`foo.h`:

```cpp
class Foo {
  public:
    double x;
  private:
    int y;
};

Foo get_foo();

```

`foo.cpp`:

```cpp
#include "foo.h"
Foo get_foo() { /* implementation */ }

```

`main.cpp`:

```cpp
// I want access to the private member, so I am going to replace Foo with my own type
class Foo {
  public:
    double x;
    int y;
};
Foo get_foo(); // declare this function ourselves since we aren't including foo.h
int main() {
    Foo foo = get_foo();
    // do something with foo.y
}

```

The above program exhibits undefined behavior because it contains two definitions of the class `::Foo`, which has external linkage, in different translation units, but the two definitions are not identical. Unlike redefinition of a class within the **same** translation unit, this problem is not required to be diagnosed by the compiler.

> 上面的程序显示了未定义的行为，因为它在不同的翻译单元中包含了类“：：Foo”的两个定义，这两个定义具有外部链接，但这两种定义并不相同。与在相同的**翻译单元中重新定义类不同，编译器不需要诊断这个问题。

## Modifying a const object

Any attempt to modify a `const` object results in undefined behavior. This applies to `const` variables, members of `const` objects, and class members declared `const`. (However, a `mutable` member of a `const` object is not `const`.)

> 任何修改“const”对象的尝试都会导致未定义的行为。这适用于 `const` 变量、`const` 对象的成员以及声明为 `const` 的类成员。（但是，“const”对象的“mutable”成员不是“const“。）

Such an attempt can be made through `const_cast`:

```cpp
const int x = 123;
const_cast<int&>(x) = 456;
std::cout << x << '\n';

```

A compiler will usually inline the value of a `const int` object, so it's possible that this code compiles and prints `123`. Compilers can also place `const` objects' values in read-only memory, so a segmentation fault may occur. In any case, the behavior is undefined and the program might do anything.

> 编译器通常会内联“const int”对象的值，因此此代码可能编译并打印“123”。编译器还可以将“const”对象的值放在只读内存中，因此可能会出现分段错误。在任何情况下，行为都是未定义的，程序可能会执行任何操作。

The following program conceals a far more subtle error:

```cpp
#include <iostream>

class Foo* instance;

class Foo {
  public:
    int get_x() const { return m_x; }
    void set_x(int x) { m_x = x; }
  private:
    Foo(int x, Foo*& this_ref): m_x(x) {
        this_ref = this;
    }
    int m_x;
    friend const Foo& getFoo();
};

const Foo& getFoo() {
    static const Foo foo(123, instance);
    return foo;
}

void do_evil(int x) {
    instance->set_x(x);
}

int main() {
    const Foo& foo = getFoo();
    do_evil(456);
    std::cout << foo.get_x() << '\n';
}

```

In this code, `getFoo` creates a singleton of type `const Foo` and its member `m_x` is initialized to `123`. Then `do_evil` is called and the value of `foo.m_x` is apparently changed to 456. What went wrong?

> 在该代码中，“getFoo”创建类型为“const Foo”的单例，其成员“m_x”初始化为“123”。然后调用“do_evil”，“foo.m_x”的值显然变为 456。出了什么问题？

Despite its name, `do_evil` does nothing particularly evil; all it does is call a setter through a `Foo*`. But that pointer points to a `const Foo` object even though `const_cast` was not used. This pointer was obtained through `Foo`'s constructor. A `const` object does not become `const` until its initialization is complete, so `this` has type `Foo*`, not `const Foo*`, within the constructor.

> 尽管名字叫“do_evil”，但它并没有什么特别邪恶的事；它所做的只是通过“Foo*”调用 setter。但该指针指向一个“const Foo”对象，即使没有使用“const_cast”。此指针是通过“Foo”的构造函数获得的。“const”对象在初始化完成之前不会变成“const“，因此“this”在构造函数中具有类型“Foo*”，而不是“const Foo*”。

Therefore, undefined behavior occurs even though there are no obviously dangerous constructs in this program.

> 因此，即使这个程序中没有明显危险的构造，也会发生未定义的行为。

## Modifying a string literal

```cpp
char *str = "hello world";
str[0] = 'H';

```

`"hello world"` is a string literal, so modifying it gives undefined behaviour.

The initialisation of `str` in the above example was formally deprecated (scheduled for removal from a future version of the standard) in C++03.  A number of compilers before 2003 might issue a warning about this (e.g. a suspicious conversion).  After 2003, compilers typically warn about a deprecated conversion.

> 上述示例中“str”的初始化在 C++03 中被正式否决（计划从标准的未来版本中删除）。2003 年之前的一些编译器可能会对此发出警告（例如可疑的转换）。2003 年之后，编译器通常会对不推荐使用的转换发出警告。

The above example is illegal, and results in a compiler diagnostic, in C++11 and later.   A similar example may be constructed to exhibit undefined behaviour by explicitly permitting the type conversion, such as:

> 在 C++11 及更高版本中，上述示例是非法的，并导致编译器诊断。类似的例子可以通过明确地允许类型转换来表现出未定义的行为，例如：

```cpp
char *str = const_cast<char *>("hello world");
str[0] = 'H'; 

```

## Accessing an object as the wrong type

In most cases, it is illegal to access an object of one type as though it were a different type (disregarding cv-qualifiers). Example:

> 在大多数情况下，访问一个类型的对象就像访问另一个类型一样是非法的（忽略 cv 限定符）。示例：

```cpp
float x = 42;
int y = reinterpret_cast<int&>(x);

```

The result is undefined behavior.

There are some exceptions to  this **strict aliasing** rule:

- An object of class type can be accessed as though it were of a type that is a base class of the actual class type.

> -类类型的对象可以被访问，就好像它的类型是实际类类型的基类一样。

- Any type can be accessed as a `char` or `unsigned char`, but the reverse is not true: a char array cannot be accessed as though it were an arbitrary type.

> -任何类型都可以作为“char”或“unsigned char”访问，但相反：不能将 char 数组作为任意类型访问。

- A signed integer type can be accessed as the corresponding unsigned type and **vice versa**.

A related rule is that if a non-static member function is called on an object that does not actually have the same type as the defining class of the function, or a derived class, then undefined behavior occurs. This is true even if the function does not access the object.

> 一个相关的规则是，如果在一个与函数的定义类或派生类实际上不具有相同类型的对象上调用非静态成员函数，则会发生未定义的行为。即使函数不访问对象，也是如此。

```cpp
struct Base {
};
struct Derived : Base {
    void f() {}
};
struct Unrelated {};
Unrelated u;
Derived& r1 = reinterpret_cast<Derived&>(u); // ok
r1.f();                                      // UB
Base b;
Derived& r2 = reinterpret_cast<Derived&>(b); // ok
r2.f();                                      // UB

```

## Overflow during conversion to or from floating point type

If, during the conversion of:

- an integer type to a floating point type,
- a floating point type to an integer type, or
- a floating point type to a shorter floating point type,

the source value is outside the range of values that can be represented in the destination type, the result is undefined behavior. Example:

> 源值在目标类型中可以表示的值范围之外，结果是未定义的行为。示例：

```cpp
double x = 1e100;
int y = x; // int probably cannot hold numbers that large, so this is UB

```

## Returning from a [[noreturn]] function

Example from the Standard, [dcl.attr.noreturn]:

```cpp
[[ noreturn ]] void f() {
  throw "error"; // OK
}
[[ noreturn ]] void q(int i) { // behavior is undefined if called with an argument <= 0
  if (i > 0)
    throw "positive";
}

```

## Infinite template recursion

Example from the Standard, [temp.inst]/17:

```cpp
template<class T> class X {
    X<T>* p; // OK
    X<T*> a; // implicit generation of X<T> requires
             // the implicit instantiation of X<T*> which requires
             // the implicit instantiation of X<T**> which ...
};

```

## Floating point overflow

If an arithmetic operation that yields a floating point type produces a value that is not in the range of representable values of the result type, the behavior is undefined according to the C++ standard, but may be defined by other standards the machine might conform to, such as IEEE 754.

> 如果产生浮点类型的算术运算产生的值不在结果类型的可表示值的范围内，则根据 C++ 标准，行为是未定义的，但可以由机器可能遵循的其他标准来定义，例如 IEEE 754。

```cpp
float x = 1.0;
for (int i = 0; i < 10000; i++) {
    x *= 10.0; // will probably overflow eventually; undefined behavior
}

```

## Calling (Pure) Virtual Members From Constructor Or Destructor

[The Standard (10.4)](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2015/n4527.pdf) states:

> 【标准（10.4）】([http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2015/n4527.pdf](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2015/n4527.pdf))状态：

>

Member functions can be called from a constructor (or destructor) of an abstract class; the effect of making a virtual call (10.3) to a pure virtual function directly or indirectly for the object being created (or destroyed) from such a constructor (or destructor) is undefined.

> 成员函数可以从抽象类的构造函数（或析构函数）中调用；从这样的构造函数（或析构函数）直接或间接地为正在创建（或销毁）的对象对纯虚拟函数进行虚拟调用（10.3）的效果是未定义的。

More generally, some C++ authorities, e.g. Scott Meyers, [suggest](http://www.artima.com/cppsource/nevercall.html) never calling virtual functions (even non-pure ones) from constructors and dstructors.

> 更普遍地说，一些 C++ 权威机构，例如 Scott Meyers，[建议](http://www.artima.com/cppsource/nevercall.html)从不从构造函数和 dstructor 调用虚拟函数（甚至是非纯函数）。

Consider the following example, modified from the above link:

```cpp
class transaction
{
public:
    transaction(){ log_it(); }
    virtual void log_it() const = 0;
};

class sell_transaction : public transaction
{
public:
    virtual void log_it() const { /* Do something */ }
};

```

Suppose we create a `sell_transaction` object:

```cpp
sell_transaction s;

```

This implicitly calls the constructor of `sell_transaction`, which first calls the constructor of `transaction`. When the constructor of `transaction` is called though, the object is not yet of the type `sell_transaction`, but rather only of the type `transaction`.

> 这会隐式调用“sell_transaction”的构造函数，该构造函数首先调用“transaction”。然而，当调用“transaction”的构造函数时，该对象还不是“sell_transaction”类型，而是仅属于“transaction'”类型。

Consequently, the call in `transaction::transaction()` to `log_it`, won't do what might seem to be the intuitive thing - namely call `sell_transaction::log_it`.

> 因此，在“transaction:：transaction（）”中对“log_it”的调用不会做看似直观的事情，即调用“sell_transaction:”log_it“。

<li>

If `log_it` is pure virtual, as in this example, the behaviour is undefined.

> 如果“log_it”是纯虚拟的，如本例所示，则行为是未定义的。

</li>
<li>
If `log_it` is non-pure virtual, `transaction::log_it` will be called.
</li>

## Invalid base-to-derived static cast

If `static_cast` is used to convert a pointer (resp. reference) to base class to a pointer (resp. reference) to derived class, but the operand does not point (resp. refer) to an object of the derived class type, the behavior is undefined. See [Base to derived conversion](http://stackoverflow.com/documentation/c%2B%2B/3090/explicit-type-conversions/10518/base-to-derived-conversion#t=201608021020103102096).

> 如果使用“static_cast”将基类的指针（resp.reference）转换为派生类的指针（resp.reference。请参见[基础到派生的转换]([http://stackoverflow.com/documentation/c%2b%2b/3090/explicit](http://stackoverflow.com/documentation/c%2B%2B/3090/explicit)-类型转换/10518/基础到派生转换#t=201608021020103102096）。

## Function call through mismatched function pointer type

In order to call a function through a function pointer, the function pointer's type must exactly match the function's type. Otherwise, the behaviour is undefined. Example:

> 为了通过函数指针调用函数，函数指针的类型必须与函数的类型完全匹配。否则，行为是未定义的。示例：

```cpp
int f();
void (*p)() = reinterpret_cast<void(*)()>(f);
p(); // undefined

```

## Access to nonexistent member through pointer to member

When accessing a non-static member of an object through a pointer to member, if the object does not actually contain the member denoted by the pointer, the behavior is undefined. (Such a pointer to member can be obtained through `static_cast`.)

> 当通过指向成员的指针访问对象的非静态成员时，如果对象实际上不包含指针所表示的成员，则行为是未定义的。（这样一个指向成员的指针可以通过“static_cast”获得。）

```cpp
struct Base { int x; };
struct Derived : Base { int y; };
int Derived::*pdy = &Derived::y;
int Base::*pby = static_cast<int Base::*>(pdy);

Base* b1 = new Derived;
b1->*pby = 42; // ok; sets y in Derived object to 42
Base* b2 = new Base;
b2->*pby = 42; // undefined; there is no y member in Base

```

## Invalid derived-to-base conversion for pointers to members

When `static_cast` is used to convert `T D::*` to `T B::*`, the member pointed to must belong to a class that is a base class or derived class of `B`. Otherwise the behavior is undefined.     See [Derived to base conversion for pointers to members](http://stackoverflow.com/documentation/c%2B%2B/3090/explicit-type-conversions/18752/derived-to-base-conversion-for-pointers-to-members)

> 当使用“static_cast”将“T D:：*'”转换为“T B:：*` 时，所指向的成员必须属于作为“B”的基类或派生类的类。否则行为是未定义的。请参阅[指向成员的指针的派生到基的转换]([http://stackoverflow.com/documentation/c%2b%2b/3090/explicit](http://stackoverflow.com/documentation/c%2B%2B/3090/explicit)-类型转换/18752/指向成员的指针的派生到基的转换）

## Destroying an object that has already been destroyed

In this example, a destructor is explicitly invoked for an object that will later be automatically destroyed.

> 在本例中，为稍后将自动销毁的对象显式调用析构函数。

```cpp
struct S {
    ~S() { std::cout << "destroying S\n"; }
};
int main() {
    S s;
    s.~S();
} // UB: s destroyed a second time here

```

A similar issue occurs when a `std::unique_ptr<T>` is made to point at a `T` with automatic or static storage duration.

> 当使“std:：unique_ptr＜T＞”指向具有自动或静态存储持续时间的“T”时，也会出现类似的问题。

```cpp
void f(std::unique_ptr<S> p);
int main() {
    S s;
    std::unique_ptr<S> p(&s);
    f(std::move(p)); // s destroyed upon return from f
}                    // UB: s destroyed

```

Another way to destroy an object twice is by having two `shared_ptr` s both manage the object without sharing ownership with each other.

> 另一种两次销毁对象的方法是让两个“shared_ptr”在不共享所有权的情况下管理对象。

```cpp
void f(std::shared_ptr<S> p1, std::shared_ptr<S> p2);
int main() {
    S* p = new S;
    // I want to pass the same object twice...
    std::shared_ptr<S> sp1(p);
    std::shared_ptr<S> sp2(p);
    f(sp1, sp2);
} // UB: both sp1 and sp2 will destroy s separately
// NB: this is correct:
// std::shared_ptr<S> sp(p);
// f(sp, sp);

```

#### Remarks

If a program contains undefined behavior, the C++ standard places no constraints on its behavior.

> 如果程序包含未定义的行为，则 C++ 标准不会对其行为进行约束。

- It may appear to work as the developer intended, but it may also crash or produce strange results.

> -它可能看起来像开发人员想要的那样工作，但也可能崩溃或产生奇怪的结果。

- The behavior may vary between runs of the same program.
- Any part of the program may malfunction, including lines that come before the line that contains undefined behavior.

> -程序的任何部分都可能发生故障，包括包含未定义行为的行之前的行。

- The implementation is not required to document the result of undefined behavior.

An implementation **may** document the result of an operation that produces undefined behavior according to the standard, but a program that depends on such documented behavior is not portable.

> 实现**可以**记录根据标准产生未定义行为的操作的结果，但依赖于这种记录行为的程序是不可移植的。

**Why undefined behavior exists**

Intuitively, undefined behavior is considered a bad thing as such errors can't be handled graciously through, say, exception handlers.

> 直觉上，未定义的行为被认为是一件坏事，因为这样的错误不能通过异常处理程序优雅地处理。

But leaving some behavior undefined is actually an integral part of C++'s promise "you don't pay for what you don't use".

> 但是，让一些行为不被定义实际上是 C++ 承诺的一个组成部分“你不会为你不使用的东西付费”。

Undefined behavior allows a compiler to assume the developer knows what he's doing and not introduce code to check for the mistakes highlighted in the above examples.

> 未定义的行为允许编译器假设开发人员知道他在做什么，而不引入代码来检查上面示例中突出显示的错误。

**Finding and avoiding undefined behavior**

Some tools can be used to discover undefined behavior during development:

- Most compilers have warning flags to warn about some cases of undefined behavior at compile time.

> -大多数编译器都有警告标志，以警告在编译时出现未定义行为的某些情况。

- Newer versions of gcc and clang include a so-called "Undefined Behavior Sanitizer" flag (`-fsanitize=undefined`) that will check for undefined behavior at runtime, at a performance cost.

> -gcc 和 clang 的较新版本包括一个所谓的“未定义行为 Sanitizer”标志（`-fsanitize=Undefined`），它将在运行时检查未定义的行为，但会以性能为代价。

- `lint`-like tools may perform more thorough undefined behavior analysis.

**Undefined, unspecified and [implementation-defined](http://stackoverflow.com/documentation/c%2B%2B/1363/implementation-defined-behavior) behavior**

> **未定义、未指定和[已定义实现]([http://stackoverflow.com/documentation/c%2B%2B/1363/implementation](http://stackoverflow.com/documentation/c%2B%2B/1363/implementation)-定义的行为）行为**

From C++14 standard (ISO/IEC 14882:2014) section 1.9 (Program Execution):

>

<ol>
<li>

The semantic descriptions in this International Standard define a parameterized nondeterministic abstract machine. [CUT]

> 本国际标准中的语义描述定义了一个参数化的不确定性抽象机。[剪切]

</li>
<li>

Certain aspects and operations of the abstract machine are described in this International Standard as **implementation-defined** (for example, `sizeof(int)`). These constitute **the parameters of the abstract machine**. Each implementation shall include documentation describing its characteristics and behavior in these respects. [CUT]

> 抽象机的某些方面和操作在本国际标准中被描述为**实现定义的**（例如，`sizeof（int）`）。这些构成了**抽象机器的参数**。每个实施应包括描述其在这些方面的特征和行为的文件。[剪切]

</li>
<li>

Certain other aspects and operations of the abstract machine are described in this International Standard as **unspecified** (for example, evaluation of expressions in a **new-initializer** if the allocation function fails to allocate memory). Where possible, this International Standard defines a set of allowable behaviors. These define the nondeterministic aspects of the abstract machine. An instance of the abstract machine can thus have more than one possible execution for a given program and a given input.

> 抽象机的某些其他方面和操作在本国际标准中被描述为**未指定**（例如，如果分配函数未能分配内存，则对**新初始化器**中的表达式进行求值）。在可能的情况下，本国际标准定义了一组允许的行为。这些定义了抽象机器的不确定性方面。因此，抽象机器的实例对于给定的程序和给定的输入可以具有不止一个可能的执行。

</li>
<li>

Certain other operations are described in this International Standard as **undefined** (or example, the effect of attempting to modify a `const` object). [ **Note**: this International Standard imposes no requirements on the behavior of programs that contain undefined behavior. —**end note** ]

> 某些其他操作在本国际标准中被描述为**未定义**（例如，试图修改“const”对象的效果）。[**注**：本国际标准对包含未定义行为的程序的行为没有任何要求。--**尾注**]

</li>
</ol>
