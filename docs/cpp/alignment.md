---
tip: translate by baidu@2023-10-25 08:16:30
metaTitle: "C++ | Alignment"
description: "Controlling alignment, Querying the alignment of a type"
---

# Alignment

All types in C++ have an alignment. This is a restriction on the memory address that objects of that type can be created within. A memory address is valid for an object's creation if dividing that address by the object's alignment is a whole number.

> C++中的所有类型都有对齐方式。这是对可以在其中创建该类型对象的内存地址的限制。如果内存地址除以对象的对齐方式是一个整数，则该地址对对象的创建有效。

Type alignments are always a power of two (including 1).

## Controlling alignment

The `alignas` [keyword](http://stackoverflow.com/documentation/c%2b%2b/4891/keywords) can be used to force a variable, class data member, declaration or definition of a class, or declaration or definition of an enum, to have a particular alignment, if supported. It comes in two forms:

> `alignas`[关键字](http://stackoverflow.com/documentation/c%2b%2b/4891/keywords)可以用于强制变量、类数据成员、类的声明或定义，或枚举的声明或限定具有特定的对齐方式（如果支持的话）。它有两种形式：

- `alignas(x)`, where `x` is a constant expression, gives the entity the alignment `x`, if supported.

> -“alignas（x）”，其中“x”是一个常量表达式，如果支持，则为实体提供对齐方式“x”。

- `alignas(T)`, where `T` is a type, gives the entity an alignment equal to the alignment requirement of `T`, that is, `alignof(T)`, if supported.

> -“alignas（T）”，其中“T”是一种类型，如果支持，则为实体提供与“T”的对齐要求相等的对齐，即“alignof（T）'。

If multiple `alignas` specifiers are applied to the same entity, the strictest one applies.

> 如果多个“alignas”说明符应用于同一实体，则应用最严格的一个。

In this example, the buffer `buf` is guaranteed to be appropriately aligned to hold an `int` object, even though its element type is `unsigned char`, which may have a weaker alignment requirement.

> 在这个例子中，缓冲区“buf”被保证适当地对齐以容纳“int”对象，即使它的元素类型是“unsigned char”，这可能具有较弱的对齐要求。

```cpp
alignas(int) unsigned char buf[sizeof(int)];
new (buf) int(42);

```

`alignas` cannot be used to give a type a smaller alignment than the type would have without this declaration:

```cpp
alignas(1) int i; //Il-formed, unless `int` on this platform is aligned to 1 byte.
alignas(char) int j; //Il-formed, unless `int` has the same or smaller alignment than `char`.

```

`alignas`, when given an integer constant expression, must be given a valid alignment. Valid alignments are always powers of two, and must be greater than zero. Compilers are required to support all valid alignments up to the alignment of the type `std::max_align_t`. They **may** support larger alignments than this, but support for allocating memory for such objects is limited. The upper limit on alignments is implementation dependent.

C++17 features direct support in `operator new` for allocating memory for over-aligned types.

> C++17 在“operator new”中直接支持为过对齐类型分配内存。

## Querying the alignment of a type

The alignment requirement of a type can be queried using the `alignof` [keyword](http://stackoverflow.com/documentation/c%2b%2b/4891/keywords) as a unary operator. The result is a constant expression of type `std::size_t`, **i.e.,** it can be evaluated at compile time.

> 可以使用`alignof`[关键字]查询类型的对齐要求(http://stackoverflow.com/documentation/c%2b%2b/4891/keywords)作为一元运算符。结果是一个类型为“std:：size_t”的常量表达式，**即**可以在编译时对其求值。

```cpp
#include <iostream>
int main() {
    std::cout << "The alignment requirement of int is: " << alignof(int) << '\n';
}

```

Possible output

> The alignment requirement of int is: 4

If applied to an array, it yields the alignment requirement of the element type. If applied to a reference type, it yields the alignment requirement of the referenced type. (References themselves have no alignment, since they are not objects.)

> 如果应用于数组，则会产生元素类型的对齐要求。如果应用于引用类型，则会产生引用类型的对齐要求。（引用本身没有对齐，因为它们不是对象。）

#### Remarks

The standard guarantees the following:

- The alignment requirement of a type is a divisor of its size. For example, a class with size 16 bytes could have an alignment of 1, 2, 4, 8, or 16, but not 32. (If a class's members only total 14 bytes in size, but the class needs to have an alignment requirement of 8, the compiler will insert 2 padding bytes to make the class's size equal to 16.)

> -类型的对齐要求是其大小的除数。例如，大小为 16 字节的类可以具有 1、2、4、8 或 16 的对齐，但不能具有 32。（如果一个类的成员总共只有 14 个字节，但该类需要有 8 个对齐要求，则编译器将插入 2 个填充字节，使该类的大小等于 16。）

- The signed and unsigned versions of an integer type have the same alignment requirement.
- A pointer to `void` has the same alignment requirement as a pointer to `char`.
- The cv-qualified and cv-unqualified versions of a type have the same alignment requirement.

Note that while alignment exists in C++03, it was not until C++11 that it became possible to query alignment (using `alignof`) and control alignment (using `alignas`).

> 注意，虽然对齐存在于 C++03 中，但直到 C++11，才有可能查询对齐（使用“alignof”）和控制对齐（使用”alignas“）。
