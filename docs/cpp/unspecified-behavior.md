---
tip: translate by baidu@2023-10-25 08:36:03
---
---

metaTitle: "C++ | Unspecified behavior"
description: "Value of an out-of-range enum, Evaluation order of function arguments, Result of some reinterpret_cast conversions, Space occupied by a reference, Moved-from state of most standard library classes, Order of initialization of globals across TU, Static cast from bogus void* value, Result of some pointer comparisons"
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# Unspecified behavior

## Value of an out-of-range enum

If a scoped enum is converted to an integral type that is too small to hold its value, the resulting value is unspecified. Example:

> 如果作用域枚举转换为太小而无法容纳其值的整数类型，则未指定结果值。示例：

```cpp
enum class E {
    X = 1,
    Y = 1000,
};
// assume 1000 does not fit into a char
char c1 = static_cast<char>(E::X); // c1 is 1
char c2 = static_cast<char>(E::Y); // c2 has an unspecified value

```

Also, if an integer is converted to an enum and the integer's value is outside the range of the enum's values, the resulting value is unspecified. Example:

> 此外，如果一个整数被转换为枚举，并且该整数的值在枚举值的范围之外，则结果值是未指定的。示例：

```cpp
enum Color {
    RED = 1,
    GREEN = 2,
    BLUE = 3,
};
Color c = static_cast<Color>(4);

```

However, in the next example, the behavior is **not** unspecified, since the source value is within the **range** of the enum, although it is unequal to all enumerators:

> 但是，在下一个示例中，行为**不是**未指定，因为源值在枚举的**范围**内，尽管它不等于所有枚举器：

```cpp
enum Scale {
    ONE = 1,
    TWO = 2,
    FOUR = 4,
};
Scale s = static_cast<Scale>(3);

```

Here `s` will have the value 3, and be unequal to `ONE`, `TWO`, and `FOUR`.

> 这里的“s”将具有值 3，并且不等于“一”、“二”和“四”。

## Evaluation order of function arguments

If a function has multiple arguments, it is unspecified what order they are evaluated in. The following code could print `x = 1, y = 2` or `x = 2, y = 1` but it is unspecified which.

> 如果一个函数有多个参数，则不指定它们的求值顺序。下面的代码可以打印“x=1，y=2”或“x=2，y=1”，但不指定是哪一个。

```cpp
int f(int x, int y) {
    printf("x = %d, y = %d\n", x, y);
}
int get_val() {
    static int x = 0;
    return ++x;
}
int main() {
    f(get_val(), get_val());
}

```

In C++17, the order of evaluation of function arguments remains unspecified.

> 在 C++17 中，函数参数的求值顺序仍然未指定。

However, each function argument is completely evaluated, and the calling object is guaranteed evaluated before any function arguments are.

> 但是，每个函数参数都是完全求值的，并且保证在对任何函数参数求值之前对调用对象进行求值。

```cpp
struct from_int {
  from_int(int x) { std::cout << "from_int (" << x << ")\n"; }
};
int make_int(int x){ std::cout << "make_int (" << x << ")\n"; return x; }


void foo(from_int a, from_int b) {
}
void bar(from_int a, from_int b) {
}

auto which_func(bool b){
  std::cout << b?"foo":"bar" << "\n";
  return b?foo:bar;
}

int main(int argc, char const*const* argv) {
  which_func( true )( make_int(1), make_int(2) );
}

```

this must print:

```cpp
bar
make_int(1)
from_int(1)
make_int(2)
from_int(2)

```

or

```cpp
bar
make_int(2)
from_int(2)
make_int(1)
from_int(1)

```

it may **not** print `bar` after any of the `make` or `from`'s, and it may not print:

> 它可以**不**在“make”或“from”的任何一个后面打印“bar”，也可以不打印：

```cpp
bar
make_int(2)
make_int(1)
from_int(2)
from_int(1)

```

or similar.  Prior to C++17 printing `bar` after `make_int` s was legal, as was doing both `make_int` s prior to doing any `from_int` s.

## Result of some reinterpret_cast conversions

The result of a `reinterpret_cast` from one function pointer type to another, or one function reference type to another, is unspecified. Example:

> 从一个函数指针类型到另一个，或从一个功能引用类型到另另一个函数引用类型的“interpret_cast”的结果是未指定的。示例：

```cpp
int f();
auto fp = reinterpret_cast<int(*)(int)>(&f); // fp has unspecified value

```

The result of a `reinterpret_cast` from one object pointer type to another, or one object reference type to another, is unspecified. Example:

> 从一个对象指针类型到另一个对象的“interpret_cast”，或从一种对象引用类型到另另一种对象的结果是未指定的。示例：

```cpp
int x = 42;
char* p = reinterpret_cast<char*>(&x); // p has unspecified value

```

However, with most compilers, this was equivalent to `static_cast<char*>(static_cast<void*>(&x))` so the resulting pointer `p` pointed to the first byte of `x`. This was made the standard behavior in C++11. See [type punning conversion](http://stackoverflow.com/documentation/c%2B%2B/3090/explicit-type-conversions/12169/type-punning-conversion) for more details.

> 然而，对于大多数编译器，这相当于“static_cast＜char*＞（static_cast＞void*＞（&x））”，因此结果指针“p”指向“x”的第一个字节。这已成为 C++11 中的标准行为。参见[类型双关语转换]([http://stackoverflow.com/documentation/c%2b%2b/3090/explicit](http://stackoverflow.com/documentation/c%2B%2B/3090/explicit)-类型转换/12169/类型双关转换）。

## Space occupied by a reference

A reference is not an object, and unlike an object, it is not guaranteed to occupy some contiguous bytes of memory. The standard leaves it unspecified whether a reference requires any storage at all. A number of features of the language conspire to make it impossible to portably examine any storage the reference might occupy:

> 引用不是对象，与对象不同，它不能保证占用一些连续的内存字节。该标准未指明引用是否需要任何存储。该语言的许多功能共同导致无法对引用可能占用的任何存储进行便携式检查：

- If `sizeof` is applied to a reference, it returns the size of the referenced type, thereby giving no information about whether the reference occupies any storage.

> -如果将“sizeof”应用于引用，它将返回被引用类型的大小，从而不提供有关引用是否占用任何存储的信息。

- Arrays of references are illegal, so it is not possible to examine the addresses of two consecutive elements of a hypothetical reference of arrays in order to determine the size of a reference.

> -引用数组是非法的，因此不可能通过检查假设引用数组的两个连续元素的地址来确定引用的大小。

- If the address of a reference is taken, the result is the address of the referent, so we cannot get a pointer to the reference itself.

> -如果取引用的地址，结果就是引用对象的地址，因此我们无法获得指向引用本身的指针。

- If a class has a reference member, attempting to extract the address of that member using `offsetof` yields undefined behavior since such a class is not a standard-layout class.

> -如果一个类有引用成员，则尝试使用“offsetof”提取该成员的地址会产生未定义的行为，因为这样的类不是标准布局类。

- If a class has a reference member, the class is no longer standard layout, so attempts to access any data used to store the reference results in undefined or unspecified behavior.

> -如果类具有引用成员，则该类不再是标准布局，因此尝试访问用于存储引用的任何数据会导致未定义或未指定的行为。

In practice, in some cases a reference variable may be implemented similarly to a pointer variable and hence occupy the same amount of storage as a pointer, while in other cases a reference may occupy no space at all since it can be optimized out. For example, in:

> 在实践中，在某些情况下，引用变量可以类似于指针变量来实现，因此占用与指针相同的存储量，而在其他情况下，由于引用可以被优化，因此引用可能根本不占用空间。例如，在：

```cpp
void f() {
    int x;
    int& r = x;
    // do something with r
}

```

the compiler is free to simply treat `r` as an alias for `x` and replace all occurrences of `r` in the rest of the function `f` with `x`, and not allocate any storage to hold `r`.

> 编译器可以简单地将“r”视为“x”的别名，并用“x”替换函数“f”其余部分中出现的所有“r”，而不分配任何存储来保存“r”。

## Moved-from state of most standard library classes

All standard library containers are left in a **valid but unspecified** state after being moved from. For example, in the following code, `v2` will contain `{1, 2, 3, 4}` after the move, but `v1` is not guaranteed to be empty.

> 所有标准库容器在从中移出后都处于**有效但未指定的**状态。例如，在下面的代码中，“v2”在移动后将包含“{1,2,3,4}”，但“v1”不保证为空。

```cpp
int main() {
    std::vector<int> v1{1, 2, 3, 4};
    std::vector<int> v2 = std::move(v1);
}

```

Some classes do have a precisely defined moved-from state. The most important case is that of `std::unique_ptr<T>`, which is guaranteed to be null after being moved from.

> 有些类确实具有精确定义的移出状态。最重要的情况是“std:：unique_ptr＜T＞”，它在从中移动后保证为 null。

## Order of initialization of globals across TU

Whereas inside a Translation Unit, order of initialization of global variables is specified, order of initialization across Translation Units is unspecified.

> 而在转换单元内部，全局变量的初始化顺序是指定的，而跨转换单元的初始化顺序则是未指定的。

So program with following files

<li>
foo.cpp

```cpp
#include <iostream>

int dummyFoo = ((std::cout << "foo"), 0);

```

</li>
<li>
bar.cpp

```cpp
#include <iostream>

int dummyBar = ((std::cout << "bar"), 0);

```

</li>
<li>
main.cpp

```cpp
int main() {}

```

</li>

might produce as output:

```cpp
foobar

```

or

```cpp
barfoo

```

That may lead to **Static Initialization Order Fiasco**.

## Static cast from bogus void* value

If a `void*` value is converted to a pointer to object type, `T*`, but is not properly aligned for `T`, the resulting pointer value is unspecified. Example:

> 如果“void*”值被转换为指向对象类型“T*”的指针，但没有正确对齐“T”，则生成的指针值是未指定的。示例：

```cpp
// Suppose that alignof(int) is 4
int x = 42;
void* p1 = &x;
// Do some pointer arithmetic...
void* p2 = static_cast<char*>(p1) + 2;
int* p3 = static_cast<int*>(p2);

```

The value of `p3` is unspecified because `p2` cannot point to an object of type `int`; its value is not a properly aligned address.

> “p3”的值未指定，因为“p2”不能指向“int”类型的对象；其值不是正确对齐的地址。

## Result of some pointer comparisons

If two pointers are compared using `<`, `>`, `<=`, or `>=`, the result is unspecified in the following cases:

> 如果使用“＜”、“＞”、“＜＝”或“＞＝”比较两个指针，则在以下情况下，结果是未指定的：

<li>

The pointers point into different arrays. (A non-array object is considered an array of size 1.)

> 指针指向不同的数组。（非数组对象被视为大小为 1 的数组。）

```cpp
int x;
int y;
const bool b1 = &x < &y;            // unspecified
int a[10];
const bool b2 = &a[0] < &a[1];      // true
const bool b3 = &a[0] < &x;         // unspecified
const bool b4 = (a + 9) < (a + 10); // true
                                    // note: a+10 points past the end of the array

```

</li>
<li>

The pointers point into the same object, but to members with different access control.

> 指针指向同一对象，但指向具有不同访问控制的成员。

```cpp
class A {
  public:
    int x;
    int y;
    bool f1() { return &x < &y; } // true; x comes before y
    bool f2() { return &x < &z; } // unspecified
  private:
    int z;
};

```

</li>

#### Remarks

If the behavior of a construct is unspecified, then the standard places some constraints on the behavior, but leaves some freedom to the implementation, which is **not** required to document what happens in a given situation. It contrasts with [implementation-defined behavior](http://stackoverflow.com/documentation/c%2B%2B/1363/implementation-defined-behavior), in which the implementation **is** required to document what happens, and undefined behavior, in which anything can happen.

> 如果构造的行为是未指定的，那么标准会对该行为施加一些约束，但会给实现留下一些自由，这不是记录给定情况下发生的事情所必需的。它与[实现定义的行为]形成对比([http://stackoverflow.com/documentation/c%2b%2b/1363/implementation](http://stackoverflow.com/documentation/c%2B%2B/1363/implementation)-定义的行为），其中需要实现**来记录所发生的事情，以及未定义的行为，其中任何事情都可能发生。
