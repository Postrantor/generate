---
tip: translate by baidu@2023-10-25 08:27:03
---
---

metaTitle: "C++ | Layout of object types"
description: "Class types, Arithmetic types, Arrays"
----------------------------------------------------

# Layout of object types

## Class types

By "class", we mean a type that was defined using the `class` or `struct` keyword (but not `enum struct` or `enum class`).

> 所谓“类”，我们指的是使用“class”或“struct”关键字（而不是“enum-struct”或“enum-class”）定义的类型。

<li>

Even an empty class still occupies at least one byte of storage; it will therefore consist purely of padding. This ensures that if `p` points to an object of an empty class, then `p + 1` is a distinct address and points to a distinct object. However, it is possible for an empty class to have a size of 0 when used as a base class. See [empty base optimisation](http://en.cppreference.com/w/cpp/language/ebo).

> 即使是一个空类仍然占用至少一个字节的存储空间；因此，它将纯粹由填充组成。这确保了如果“p”指向一个空类的对象，那么“p+1”是一个不同的地址并指向一个不同对象。但是，当用作基类时，空类的大小可能为 0。参见[空基优化](http://en.cppreference.com/w/cpp/language/ebo)。

```cpp
class Empty_1 {};                               // sizeof(Empty_1)       == 1
class Empty_2 {};                               // sizeof(Empty_2)       == 1
class Derived : Empty_1 {};                     // sizeof(Derived)       == 1
class DoubleDerived : Empty_1, Empty_2 {};      // sizeof(DoubleDerived) == 1
class Holder { Empty_1 e; };                    // sizeof(Holder)        == 1
class DoubleHolder { Empty_1 e1; Empty_2 e2; }; // sizeof(DoubleHolder)  == 2
class DerivedHolder : Empty_1 { Empty_1 e; };   // sizeof(DerivedHolder) == 2

```

</li>
<li>

The object representation of a class type contains the object representations of the base class and non-static member types. Therefore, for example, in the following class:

> 类类型的对象表示包含基类和非静态成员类型的对象表达。因此，例如，在以下类中：

```cpp
struct S {
    int x;
    char* y;
};

```

there is a consecutive sequence of `sizeof(int)` bytes within an `S` object, called a **subobject,** that contain the value of `x`, and another subobject with `sizeof(char*)` bytes that contains the value of `y`. The two cannot be interleaved.

> 在一个“S”对象内有一个连续的“sizeof（int）”字节序列，称为包含“x”值的**子对象，以及另一个包含“y”值的具有“size of（char*）”字节的子对象。两者不能交错。

</li>
<li>

If a class type has members and/or base classes with types `t1, t2,...tN`, the size must be at least `sizeof(t1) + sizeof(t2) + ... + sizeof(tN)` given the preceding points. However, depending on the [alignment](http://stackoverflow.com/documentation/c%2B%2B/9249/alignment) requirements of the members and base classes, the compiler may be forced to insert padding between subobjects, or at the beginning or end of the complete object.

> 如果类类型具有类型为“t1，t2，…”的成员和/或基类，。。。tN `，大小必须至少为` sizeof（t1）+sizeof（t2）+…+sizeof（tN）`。但是，取决于[对齐](http://stackoverflow.com/documentation/c%2B%2B/9249/alignment)根据成员和基类的要求，编译器可能会被迫在子对象之间插入填充，或者在整个对象的开头或结尾插入填充。

```cpp
struct AnInt      { int i; };
  // sizeof(AnInt)        == sizeof(int)
  // Assuming a typical 32- or 64-bit system, sizeof(AnInt)        == 4 (4).
struct TwoInts    { int i, j; };
  // sizeof(TwoInts)      >= 2 * sizeof(int)
  // Assuming a typical 32- or 64-bit system, sizeof(TwoInts)      == 8 (4 + 4).
struct IntAndChar { int i; char c; };
  // sizeof(IntAndChar)   >= sizeof(int) + sizeof(char)
  // Assuming a typical 32- or 64-bit system, sizeof(IntAndChar)   == 8 (4 + 1 + padding).
struct AnIntDerived : AnInt { long long l; };
  // sizeof(AnIntDerived) >= sizeof(AnInt) + sizeof(long long)
  // Assuming a typical 32- or 64-bit system, sizeof(AnIntDerived) == 16 (4 + padding + 8).

```

</li>
<li>
<p>If padding is inserted in an object due to alignment requirements, the size will be greater than the sum of the sizes of the members and base classes.  With `n`-byte alignment, size will typically be the smallest multiple of `n` which is larger than the size of all members & base classes.  Each member `memN` will typically be placed at an address which is a multiple of `alignof(memN)`, and `n` will typically be the largest `alignof` out of all members' `alignof`s.  Due to this, if a member with a smaller `alignof` is followed by a member with a larger `alignof`, there is a possibility that the latter member will not be aligned properly if placed immediately after the former.  In this case, padding (also known as an **alignment member** ) will be placed between the two members, such that the latter member can have its desired alignment.  Conversely, if a member with a larger `alignof` is followed by a member with a smaller `alignof`, no padding will usually be necessary.  This process is also known as "packing".<br />

Due to classes typically sharing the `alignof` of their member with the largest `alignof`, classes will typically be aligned to the `alignof` of the largest built-in type they directly or indirectly contain.</p>

> 由于类通常与最大的“alignof”共享其成员的“alionof”，因此类通常会与它们直接或间接包含的最大内置类型的“alinof”对齐</p>

```cpp
// Assume sizeof(short) == 2, sizeof(int) == 4, and sizeof(long long) == 8.
// Assume 4-byte alignment is specified to the compiler.
struct Char { char c; };
  // sizeof(Char)                == 1 (sizeof(char))
struct Int  { int i; };
  // sizeof(Int)                 == 4 (sizeof(int))
struct CharInt { char c; int i; };
  // sizeof(CharInt)             == 8 (1 (char) + 3 (padding) + 4 (int))
struct ShortIntCharInt { short s; int i; char c; int j; };
  // sizeof(ShortIntCharInt)     == 16 (2 (short) + 2 (padding) + 4 (int) + 1 (char) +
  //                                    3 (padding) + 4 (int))
struct ShortIntCharCharInt { short s; int i; char c; char d; int j; };
  // sizeof(ShortIntCharCharInt) == 16 (2 (short) + 2 (padding) + 4 (int) + 1 (char) +
  //                                    1 (char) + 2 (padding) + 4 (int))
struct ShortCharShortInt { short s; char c; short t; int i; };
  // sizeof(ShortCharShortInt)   == 12 (2 (short) + 1 (char) + 1 (padding) + 2 (short) +
  //                                    2 (padding) + 4 (int))
struct IntLLInt { int i; long long l; int j; };
  // sizeof(IntLLInt)            == 16 (4 (int) + 8 (long long) + 4 (int))
  // If packing isn't explicitly specified, most compilers will pack this as
  //   8-byte alignment, such that:
  // sizeof(IntLLInt)            == 24 (4 (int) + 4 (padding) + 8 (long long) +
  //                                    4 (int) + 4 (padding))

// Assume sizeof(bool) == 1, sizeof(ShortIntCharInt) == 16, and sizeof(IntLLInt) == 24.
// Assume default alignment: alignof(ShortIntCharInt) == 4, alignof(IntLLInt) == 8.
struct ShortChar3ArrShortInt {
    short s;
    char c3[3];
    short t;
    int i;
};
  // ShortChar3ArrShortInt has 4-byte alignment: alignof(int) >= alignof(char) &&
  //                                             alignof(int) >= alignof(short)
  // sizeof(ShortChar3ArrShortInt) == 12 (2 (short) + 3 (char[3]) + 1 (padding) +
  //                                      2 (short) + 4 (int))
  // Note that t is placed at alignment of 2, not 4.  alignof(short) == 2.

struct Large_1 {
    ShortIntCharInt sici;
    bool b;
    ShortIntCharInt tjdj;
};
  // Large_1 has 4-byte alignment.
    // alignof(ShortIntCharInt) == alignof(int) == 4
    // alignof(b) == 1
    // Therefore, alignof(Large_1) == 4.
  // sizeof(Large_1) == 36 (16 (ShortIntCharInt) + 1 (bool) + 3 (padding) +
  //                        16 (ShortIntCharInt))
struct Large_2 {
    IntLLInt illi;
    float f;
    IntLLInt jmmj;
};
  // Large_2 has 8-byte alignment.
    // alignof(IntLLInt) == alignof(long long) == 8
    // alignof(float) == 4
    // Therefore, alignof(Large_2) == 8.
  // sizeof(Large_2) == 56 (24 (IntLLInt) + 4 (float) + 4 (padding) + 24 (IntLLInt))

```

</li>

<li>

If strict alignment is forced with `alignas`, padding will be used to force the type to meet the specified alignment, even when it would otherwise be smaller. For example, with the definition below, `Chars<5>` will have three (or possibly more) padding bytes inserted at the end so that its total size is 8. It is not possible for a class with an alignment of 4 to have a size of 5 because it would be impossible to make an array of that class, so the size must be "rounded up" to a multiple of 4 by inserting padding bytes.

> 如果强制使用“alignas”进行严格对齐，则将使用填充来强制类型符合指定的对齐方式，即使在其他情况下会更小。例如，对于下面的定义，“Chars<5>”将在末尾插入三个（或可能更多）填充字节，因此其总大小为 8。对齐方式为 4 的类不可能具有 5 的大小，因为不可能生成该类的数组，因此必须通过插入填充字节将大小“四舍五入”为 4 的倍数。

```cpp
// This type shall always be aligned to a multiple of 4.  Padding shall be inserted as
// needed.
// Chars<1>..Chars<4> are 4 bytes, Chars<5>..Chars<8> are 8 bytes, etc.
template<size_t SZ>
struct alignas(4) Chars { char arr[SZ]; };

static_assert(sizeof(Chars<1>) == sizeof(Chars<4>), "Alignment is strict.\n");

```

</li>

- If two non-static members of a class have the same [access specifier](http://stackoverflow.com/documentation/c%2B%2B/508/classes-structures/1668/access-specifiers), then the one that comes later in declaration order is guaranteed to come later in the object representation. But if two non-static members have different access specifiers, their relative order within the object is unspecified.

> -如果一个类的两个非静态成员具有相同的[access 说明符]([http://stackoverflow.com/documentation/c%2b%2b/508/classes-structures/1668/access](http://stackoverflow.com/documentation/c%2B%2B/508/classes-structures/1668/access) 说明符），那么在声明顺序中较晚出现的那个就保证在对象表示中较晚出现。但是，如果两个非静态成员具有不同的访问说明符，则它们在对象中的相对顺序是未指定的。

- It is unspecified what order the base class subobjects appear in within an object, whether they occur consecutively, and whether they appear before, after, or between member subobjects.

> -未指定基类子对象在对象中的显示顺序，是否连续出现，以及是否在成员子对象之前、之后或之间出现。

## Arithmetic types

### Narrow character types

The `unsigned char` type uses all bits to represent a binary number. Therefore, for example, if `unsigned char` is 8 bits long, then the 256 possible bit patterns of a `char` object represent the 256 different values {0, 1, ..., 255}. The number 42 is guaranteed to be represented by the bit pattern `00101010`.

> “unsigned char”类型使用所有位来表示二进制数。因此，例如，如果“unsigned char”是 8 比特长，则“char”对象的 256 个可能的比特模式表示 256 个不同的值{0，1，…，255}。数字 42 保证由比特模式“00101010”表示。

The `signed char` type has no padding bits, **i.e.,** if `signed char` is 8 bits long, then it has 8 bits of capacity to represent a number.

> “有符号字符”类型没有填充位，**，即**如果“有符号的字符”是 8 位长，则它有 8 位容量来表示一个数字。

Note that these guarantees do not apply to types other than narrow character types.

> 请注意，这些保证不适用于窄字符类型以外的类型。

### Integer types

The unsigned integer types use a pure binary system, but may contain padding bits. For example, it is possible (though unlikely) for `unsigned int` to be 64 bits long but only  be capable of storing integers between 0 and 2<sup>32</sup> - 1, inclusive. The other 32 bits would be padding bits, which should not be written to directly.

> 无符号整数类型使用纯二进制系统，但可能包含填充位。例如，“unsigned int”有可能（尽管不太可能）是 64 位长，但只能存储 0 到 2<sup>32</sup>-1 之间的整数，包括 0 和 2<sup>。其他 32 位将是填充位，不应直接写入。

The signed integer types use a binary system with a sign bit and possibly padding bits. Values that belong to the common range of a signed integer type and the corresponding unsigned integer type have the same representation. For example, if the bit pattern `0001010010101011` of an `unsigned short` object represents the value `5291`, then it also represents the value `5291` when interpreted as a `short` object.

> 带符号整数类型使用带有符号位和可能的填充位的二进制系统。属于有符号整数类型的公共范围的值和相应的无符号整数类型具有相同的表示形式。例如，如果“无符号短”对象的位模式“0001010010101011”表示值“5291”，那么当被解释为“短”对象时，它也表示值“5231”。

It is implementation-defined whether a two's complement, one's complement, or sign-magnitude representation is used, since all three systems satisfy the requirement in the previous paragraph.

> 由于所有三个系统都满足上一段中的要求，因此实现定义了是否使用二补码、一补码或符号幅度表示。

### Floating point types

The value representation of floating point types is implementation-defined. Most commonly, the `float` and `double` types conform to IEEE 754 and are 32 and 64 bits long (so, for example, `float` would have 23 bits of precision which would follow 8 exponent bits and 1 sign bit). However, the standard does not guarantee anything. Floating point types often have "trap representations", which cause errors when they are used in calculations.

> 浮点类型的值表示是由实现定义的。最常见的是，“float”和“double”类型符合 IEEE 754，并且是 32 和 64 位长（因此，例如，“floast”将具有 23 位精度，其将跟随 8 个指数位和 1 个符号位）。然而，该标准并不能保证任何东西。浮点类型通常具有“陷阱表示法”，在计算中使用时会导致错误。

## Arrays

An array type has no padding in between its elements. Therefore, an array with element type `T` is just a sequence of `T` objects laid out in memory, in order.

> 数组类型的元素之间没有填充。因此，元素类型为“T”的数组只是按顺序排列在内存中的“T”对象序列。

A multidimensional array is an array of arrays, and the above applies recursively. For example, if we have the declaration

> 多维数组是数组的数组，上面的内容递归应用。例如，如果我们有声明

```cpp
int a[5][3];

```

then `a` is an array of 5 arrays of 3 `int` s. Therefore, `a[0]`, which consists of the three elements `a[0][0]`, `a[0][1]`, `a[0][2]`, is laid out in memory before `a[1]`, which consists of `a[1][0]`, `a[1][1]`, and `a[1][2]`. This is called **row major** order.

> 则‘a’是 3 个‘int’s 的 5 个数组的数组。因此，由三个元素“a[0][0]”、“a[0][1]”、“a[0][2]”组成的“a[0]”在存储器中被布置在由“a[1][0]”、‘a[1][1]’和‘a[1][2]’组成的‘a[1]’之前。这被称为**行主**顺序。

#### Remarks

See also [Size of integral types](http://stackoverflow.com/documentation/c%2B%2B/1363/implementation-defined-behavior/4450/size-of-integral-types).

> 另请参见[积分类型的大小]([http://stackoverflow.com/documentation/c%2b%2b/1363/implementation](http://stackoverflow.com/documentation/c%2B%2B/1363/implementation)-定义的行为/450/积分类型的大小）。
