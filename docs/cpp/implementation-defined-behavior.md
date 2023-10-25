---
tip: translate by baidu@2023-10-25 08:25:48
---
---

metaTitle: "C++ | Implementation-defined behavior"
description: "Size of integral types, Char might be unsigned or signed, Number of bits in a byte, Numeric value of a pointer, Ranges of numeric types, Value representation of floating point types, Overflow when converting from integer to signed integer, Underlying type (and hence size) of an enum"
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# Implementation-defined behavior

## Size of integral types

The following types are defined as **integral types**:

- `char`
- Signed integer types
- Unsigned integer types
- `char16_t` and `char32_t`
- `bool`
- `wchar_t`

With the exception of `sizeof(char)` / `sizeof(signed char)` / `sizeof(unsigned char)`, which is split between § 3.9.1.1 [basic.fundamental/1] and § 5.3.3.1 [expr.sizeof], and `sizeof(bool)`, which is entirely implementation-defined and has no minimum size, the minimum size requirements of these types are given in section § 3.9.1 [basic.fundamental] of the standard, and shall be detailed below.

> 除了“sizeof（char）”/“sizeof”（带符号的 char）“/”sizeof（无符号的 char，并将在下文中详细说明。

### Size of `char`

All versions of the C++ standard specify, in § 5.3.3.1, that `sizeof` yields `1` for `unsigned char`, `signed char`, and `char` (it is implementation defined whether the `char` type is `signed` or `unsigned`).

> 在§5.3.3.1 中，C++ 标准的所有版本都规定，“sizeof”为“unsigned char”、“signed char”和“char”生成“1”（实现定义了“char”类型是“signed”还是“unsigned'”）。

`char` is large enough to represent 256 different values, to be suitable for storing UTF-8 code units.

### Size of signed and unsigned integer types

The standard specifies, in § 3.9.1.2, that in the list of **standard signed integer types**, consisting of `signed char`, `short int`, `int`, `long int`, and `long long int`, each type will provide at least as much storage as those preceding it in the list.  Furthermore, as specified in § 3.9.1.3, each of these types has a corresponding **standard unsigned integer type**, `unsigned char`, `unsigned short int`, `unsigned int`, `unsigned long int`, and `unsigned long long int`, which has the same size and alignment as its corresponding signed type.  Additionally, as specified in § 3.9.1.1, `char` has the same size and alignment requirements as both `signed char` and `unsigned char`.

> 该标准在§3.9.1.2 中规定，在**标准有符号整数类型**的列表中，由“有符号 char”、“short int”、“int”、‘long int’和“long long int”组成，每种类型将提供至少与其列表中前面的类型一样多的存储空间。此外，如§3.9.1.3 所述，这些类型中的每一个都有一个相应的**标准无符号整数类型**、“unsigned char”、“unssigned short int”、“signed int”、‘unsigned long int’和“unsignedlong long int”，其大小和对齐方式与其相应的有符号类型相同。此外，如§3.9.1.1 所述，“char”与“signed char”和“unsigned char”具有相同的大小和对齐要求。

Prior to C++11, `long long` and `unsigned long long` were not officially part of the C++ standard.  However, after their introduction to C, in C99, many compilers supported `long long` as an **extended signed integer type**, and `unsigned long long` as an **extended unsigned integer type**, with the same rules as the C types.

> 在 C++11 之前，“long-long”和“unsigned-long-long”不是 C++ 标准的正式组成部分。然而，在 C99 中引入 C 之后，许多编译器支持将“long-long”作为**扩展的有符号整数类型**，将“unsigned-long-long”用作**扩展的无符号整数类型***，其规则与 C 类型相同。

The standard thus guarantees that:

```cpp
1 == sizeof(char)  == sizeof(signed char) == sizeof(unsigned char)
  <= sizeof(short) == sizeof(unsigned short)
  <= sizeof(int)   == sizeof(unsigned int)
  <= sizeof(long)  == sizeof(unsigned long)

```

```cpp
 <= sizeof(long long) == sizeof(unsigned long long)

```

Specific minimum sizes for each type are not given by the standard.  Instead, each type has a minimum range of values it can support, which is, as specified in § 3.9.1.3, inherited from the C standard, in §5.2.4.2.1.  The minimum size of each type can be roughly inferred from this range, by determining the minimum number of bits required; note that for any given platform, any type's actual supported range may be larger than the minimum.  Note that for signed types, ranges correspond to one's complement, not the more commonly used two's complement; this is to allow a wider range of platforms to comply with the standard.

> 标准中没有给出每种类型的具体最小尺寸。相反，每种类型都有其可以支持的最小值范围，如§3.9.1.3 中所述，该范围继承自§5.2.4.2.1 中的 C 标准。通过确定所需的最小比特数，可以从该范围大致推断出每种类型的最小大小；注意，对于任何给定的平台，任何类型的实际支持范围都可能大于最小值。注意，对于有符号类型，范围对应于一个补码，而不是更常用的两个补码；这是为了允许更广泛的平台符合该标准。

<th align="left">Type|Minimum range</th><th align="right">Minimum bits required</th>
|---|---|---|---|---|---|---|---|---|---
<td align="left">`signed char`|-127 to 127 (-(2<sup>7</sup> - 1) to (2<sup>7</sup> - 1))</td><td align="right">8</td>
<td align="left">`unsigned char`|0 to 255 (0 to 2<sup>8</sup> - 1)</td><td align="right">8</td>
<td align="left">`signed short`|-32,767 to 32,767 (-(2<sup>15</sup> - 1) to (2<sup>15</sup> - 1))</td><td align="right">16</td>
<td align="left">`unsigned short`|0 to 65,535 (0 to 2<sup>16</sup> - 1)</td><td align="right">16</td>
<td align="left">`signed int`|-32,767 to 32,767 (-(2<sup>15</sup> - 1) to (2<sup>15</sup> - 1))</td><td align="right">16</td>
<td align="left">`unsigned int`|0 to 65,535 (0 to 2<sup>16</sup> - 1)</td><td align="right">16</td>
<td align="left">`signed long`|-2,147,483,647 to 2,147,483,647 (-(2<sup>31</sup> - 1) to (2<sup>31</sup> - 1))</td><td align="right">32</td>
<td align="left">`unsigned long`|0 to 4,294,967,295 (0 to 2<sup>32</sup> - 1)</td><td align="right">32</td>

<th align="left">Type|Minimum range</th><th align="right">Minimum bits required</th>
|---|---|---|---|---|---|---|---|---|---
<td align="left">`signed long long`|-9,223,372,036,854,775,807 to 9,223,372,036,854,775,807 (-(2<sup>63</sup> - 1) to (2<sup>63</sup> - 1))</td><td align="right">64</td>
<td align="left">`unsigned long long`|0 to 18,446,744,073,709,551,615 (0 to 2<sup>64</sup> - 1)</td><td align="right">64</td>

As each type is allowed to be greater than its minimum size requirement, types may differ in size between implementations.  The most notable example of this is with the 64-bit data models LP64 and LLP64, where LLP64 systems (such as 64-bit Windows) have 32-bit `ints` and `long` s, and LP64 systems (such as 64-bit Linux) have 32-bit `int` s and 64-bit `long` s.  Due to this, integer types cannot be assumed to have a fixed width across all platforms.

> 由于每个类型都允许大于其最小大小要求，因此不同实现之间的类型大小可能不同。这方面最显著的例子是 64 位数据模型 LP64 和 LLP64，其中 LLP64 系统（如 64 位 Windows）具有 32 位的“int”和“long”，而 LP64 系统（如 64bit Linux）具有 32 比特的“int’s”和 64 比特的“long’s”。因此，不能假设整数类型在所有平台上都具有固定的宽度。

If integer types with fixed width are required, use types from the [`<cstdint>`](http://en.cppreference.com/w/cpp/header/cstdint) header, but note that the standard makes it optional for implementations to support the exact-width types `int8_t`, `int16_t`, `int32_t`, `int64_t`, `intptr_t`, `uint8_t`, `uint16_t`, `uint32_t`, `uint64_t` and `uintptr_t`.

> 如果需要具有固定宽度的整数类型，请使用[`<cstilt>`]中的类型([http://en.cppreference.com/w/cpp/header/cstdint](http://en.cppreference.com/w/cpp/header/cstdint))头，但请注意，该标准使实现可以选择支持精确宽度类型“int8_t”、“int16_t”，“int32_t”和“int64_t”。

### Size of `char16_t` and `char32_t`

The sizes of `char16_t` and `char32_t` are implementation-defined, as specified in § 5.3.3.1, with the stipulations given in § 3.9.1.5:

> “char16_t”和“char32_t”的大小由§5.3.3.1 中规定的实现定义，并符合§3.9.1.5 中的规定：

<li>
`char16_t` is large enough to represent any UTF-16 code unit, and has the same size, signedness, and alignment as [`uint_least16_t`](http://en.cppreference.com/w/cpp/header/cstdint); it is thus required to be at least 16 bits in size.
</li>
<li>
`char32_t` is large enough to represent any UTF-32 code unit, and has the same size, signedness, and alignment as [`uint_least32_t`](http://en.cppreference.com/w/cpp/header/cstdint); it is thus required to be at least 32 bits in size.
</li>

### Size of `bool`

The size of `bool` is implementation defined, and may or may not be `1`.

### Size of wchar_t

`wchar_t`, as specified in § 3.9.1.5, is a distinct type, whose range of values can represent every distinct code unit of the largest extended character set among the supported locales.  It has the same size, signedness, and alignment as one of the other integral types, which is known as its **underlying type**.  This type's size is implementation-defined, as specified in § 5.3.3.1, and may be, for example, at least 8, 16, or 32 bits; if a system supports Unicode, for example, `wchar_t` is required to be at least 32 bits (an exception to this rule is Windows, where `wchar_t` is 16 bits for compatibility purposes).  It is inherited from the C90 standard, ISO 9899:1990 § 4.1.5, with only minor rewording.

Depending on the implementation, the size of `wchar_t` is often, but not always, 8, 16, or 32 bits.  The most common examples of these are:

> 根据实现方式的不同，“wchar_t”的大小通常但不总是 8、16 或 32 位。最常见的例子有：

- In Unix and Unix-like systems, `wchar_t` is 32-bit, and is usually used for UTF-32.
- In Windows, `wchar_t` is 16-bit, and is used for UTF-16.
- On a system which only has 8-bit support, `wchar_t` is 8 bit.

If Unicode support is desired, it is recommended to use `char` for UTF-8, `char16_t` for UTF-16, or `char32_t` for UTF-32, instead of using `wchar_t`.

> 如果需要 Unicode 支持，建议对 UTF-8 使用“char”，对 UTF-16 使用“char16_t”，或对 UTF-32 使用“char32_t”，而不是使用“wchar_t”。

### Data Models

As mentioned above, the widths of integer types can differ between platforms.  The most common models are as follows, with sizes specified in bits:

> 如上所述，不同平台之间整数类型的宽度可能不同。最常见的型号如下，以位为单位指定大小：

| Model         | `int` | `long` | pointer |  |  |  |  |  |  |
| ------------- | ----- | ------ | ------- | - | - | - | - | - | - |
| LP32 (2/4/4)  | 16    | 32     | 32      |  |  |  |  |  |  |
| ILP32 (4/4/4) | 32    | 32     | 32      |  |  |  |  |  |  |
| LLP64 (4/4/8) | 32    | 32     | 64      |  |  |  |  |  |  |
| LP64 (4/8/8)  | 32    | 64     | 64      |  |  |  |  |  |  |

Out of these models:

- 16-bit Windows used LP32.
- 32-bit *nix systems (Unix, Linux, Mac OSX, and other Unix-like OSes) and Windows use ILP32.
- 64-bit Windows uses LLP64.
- 64-bit *nix systems use LP64.

Note, however, that these models aren't specifically mentioned in the standard itself.

> 但是，请注意，这些模型在标准本身中没有特别提及。

## Char might be unsigned or signed

The standard doesn't specify if `char` should be signed or unsigned. Different compilers implement it differently, or might allow to change it using a command line switch.

> 该标准没有指定“char”是有符号的还是无符号的。不同的编译器实现它的方式不同，或者可能允许使用命令行开关进行更改。

## Number of bits in a byte

In C++, a **byte** is the space occupied by a `char` object. The number of bits in a byte is given by `CHAR_BIT`, which is defined in `climits` and required to be at least 8. While most modern systems have 8-bit bytes, and POSIX requires `CHAR_BIT` to be exactly 8, there are some systems where `CHAR_BIT` is greater than 8 i.e a single byte may be comprised of 8, 16, 32 or 64 bits.

> 在 C++ 中，**字节**是“char”对象所占用的空间。字节中的位数由“CHAR_BIT”给出，该值在“climits”中定义，要求至少为 8。虽然大多数现代系统具有 8 位字节，并且 POSIX 要求“CHAR_bit”恰好为 8，但也有一些系统的“CHAR_bit”大于 8，即单个字节可以由 8、16、32 或 64 位组成。

## Numeric value of a pointer

The result of casting a pointer to an integer using `reinterpret_cast` is implementation-defined, but "... is intended to be unsurprising to those who know the addressing structure of the underlying machine."

> 使用“interpret_cast”将指针强制转换为整数的结果是由实现定义的，但“…对于那些了解底层机器寻址结构的人来说并不奇怪。”

```cpp
int x = 42;
int* p = &x;
long addr = reinterpret_cast<long>(p);
std::cout << addr << "\n"; // prints some numeric address,
                           // probably in the architecture's native address format

```

Likewise, the pointer obtained by conversion from an integer is also implementation-defined.

> 同样，通过从整数转换获得的指针也是实现定义的。

The right way to store a pointer as an integer is using the `uintptr_t` or `intptr_t` types:

> 将指针存储为整数的正确方法是使用“uintptrtr_t”或“intptr_t”类型：

```cpp
// `uintptr_t` was not in C++03. It's in C99, in <stdint.h>, as an optional type
#include <stdint.h>

uintptr_t uip;

```

```cpp
// There is an optional `std::uintptr_t` in C++11
#include <cstdint>

std::uintptr_t uip;

```

C++11 refers to C99 for the definition `uintptr_t` (C99 standard, 6.3.2.3):

> C++11 在定义“uintptr_t”中引用 C99（C99 标准，6.3.2.3）：

>

an unsigned integer type with the property that any valid pointer to `void` can be converted to this type, then converted back to pointer to `void`, and the result will compare equal to the original pointer.

> 一个无符号整数类型，其属性是任何指向“void”的有效指针都可以转换为此类型，然后再转换回指向“void’”的指针，结果将与原始指针进行比较。

While, for the majority of modern platforms, you can assume a flat address space and that arithmetic on `uintptr_t` is equivalent to arithmetic on `char *`, it's entirely possible for an implementation to perform any transformation when casting `void *` to `uintptr_t` as long the transformation can be reversed when casting back from `uintptr_t` to `void *`.

> 虽然对于大多数现代平台，您可以假设一个平面地址空间，并且“uintptr_t”上的算术等同于“char*”上的运算，但在将“void*”强制转换为“uintpter_t”时，实现完全可以执行任何转换，只要从“uinttr_t”强制转换回“void*’时可以反转转换即可。

**Technicalities**

<li>

On XSI-conformant (X/Open System Interfaces) systems, `intptr_t` and `uintptr_t` types are required, otherwise they are **optional**.

> 在符合 XSI（X/Open System Interfaces）的系统上，“intptr_t”和“uintptr_t“类型是必需的，否则它们是**可选的**。

</li>
<li>

Within the meaning of the C standard, functions aren't objects; it isn't guaranteed by the C standard that `uintptr_t` can hold a function pointer. Anyway POSIX (2.12.3) conformance requires that:

> 在 C 标准的意义上，函数不是对象；C 标准不能保证“uintptr_t”可以保存函数指针。无论如何，POSIX（2.12.3）合规性要求：

<blockquote>

All function pointer types shall have the same representation as the type pointer to void. Conversion of a function pointer to void * shall not alter the representation. A void * value resulting from such a conversion can be converted back to the original function pointer type, using an explicit cast, without loss of information.

> 所有函数指针类型应具有与指向 void 的类型指针相同的表示形式。将函数指针转换为 void *不应改变表示。通过使用显式强制转换，可以将这种转换产生的 void* 值转换回原始函数指针类型，而不会丢失信息。

</blockquote>
</li>
<li>
C99 §7.18.1:
<blockquote>

When typedef names differing only in the absence or presence of the initial u are defined, they shall denote corresponding signed and unsigned types as described in 6.2.5; an implementation providing one of these corresponding types shall also provide the other.

> 当定义了仅在不存在或存在首字母 u 的情况下不同的 typedef 名称时，它们应表示 6.2.5 中所述的相应的有符号和无符号类型；提供这些对应类型中的一个的实现也应提供另一个。

</blockquote>
`uintptr_t` might make sense if you want to do things to the bits of the pointer that you can't do as sensibly with a signed integer.
</li>

>

When typedef names differing only in the absence or presence of the initial u are defined, they shall denote corresponding signed and unsigned types as described in 6.2.5; an implementation providing one of these corresponding types shall also provide the other.

> 当定义了仅在不存在或存在首字母 u 的情况下不同的 typedef 名称时，它们应表示 6.2.5 中所述的相应的有符号和无符号类型；提供这些对应类型中的一个的实现也应提供另一个。

## Ranges of numeric types

The ranges of the integer types are implementation-defined. The header `<limits>` provides the `std::numeric_limits<T>` template which provides the minimum and maximum values of all fundamental types. The values satisfy guarantees provided by the C standard through the `<climits>` and (>= C++11) `<cinttypes>` headers.

> 整数类型的范围是由实现定义的。标头“＜limits＞”提供了“std:：numeric_limits＜T＞”模板，该模板提供了所有基本类型的最小值和最大值。这些值满足 C 标准通过“＜climits＞”和（>=C++11）“＜cinttypes＞”标头提供的保证。

- `std::numeric_limits<signed char>::min()` equals `SCHAR_MIN`, which is less than or equal to -127.

> -`std:：numeric_limits<signed char>：：min（）` 等于 `SCHAR_min`，小于或等于-127。

- `std::numeric_limits<signed char>::max()` equals `SCHAR_MAX`, which is greater than or equal to 127.

> -`std:：numeric_limits<signed char>：：max（）` 等于 `SCHAR_max`，大于或等于 127。

- `std::numeric_limits<unsigned char>::max()` equals `UCHAR_MAX`, which is greater than or equal to 255.

> -`std:：numeric_limits<unsigned char>：：max（）` 等于 `UCHAR_max`，大于或等于 255。

- `std::numeric_limits<short>::min()` equals `SHRT_MIN`, which is less than or equal to -32767.
- `std::numeric_limits<short>::max()` equals `SHRT_MAX`, which is greater than or equal to 32767.
- `std::numeric_limits<unsigned short>::max()` equals `USHRT_MAX`, which is greater than or equal to 65535.

> -`std:：numeric_limits<unsigned short>：：max（）` 等于 `USHRT_max`，大于或等于 65535。

- `std::numeric_limits<int>::min()` equals `INT_MIN`, which is less than or equal to -32767.
- `std::numeric_limits<int>::max()` equals `INT_MAX`, which is greater than or equal to 32767.
- `std::numeric_limits<unsigned int>::max()` equals `UINT_MAX`, which is greater than or equal to 65535.

> -`std:：numeric_limits<unsigned int>：：max（）` 等于 `UINT_max`，大于或等于 65535。

- `std::numeric_limits<long>::min()` equals `LONG_MIN`, which is less than or equal to -2147483647.

> -“std:：numeric_limits＜long＞：：min（）”等于“long_min”，小于或等于-2147483647。

- `std::numeric_limits<long>::max()` equals `LONG_MAX`, which is greater than or equal to 2147483647.

> -“std:：numeric_limits＜long＞：：max（）”等于“long_max”，大于或等于 2147483647。

- `std::numeric_limits<unsigned long>::max()` equals `ULONG_MAX`, which is greater than or equal to 4294967295.

> -`std:：numeric_limits<unsigned long>：：max（）` 等于 `ULONG_max`，大于或等于 4294967295。

- `std::numeric_limits<long long>::min()` equals `LLONG_MIN`, which is less than or equal to -9223372036854775807.

> -`std:：numeric_limits<long-long>：：min（）` 等于 `LLONG_min`，小于或等于 9223372036854775807。

- `std::numeric_limits<long long>::max()` equals `LLONG_MAX`, which is greater than or equal to 9223372036854775807.

> -`std:：numeric_limits<long-long>：：max（）` 等于 `LLONG_max`，大于或等于 9223372036854775807。

- `std::numeric_limits<unsigned long long>::max()` equals `ULLONG_MAX`, which is greater than or equal to 18446744073709551615.

> -`std:：numeric_limits<unsigned long-long>：：max（）` 等于 `ULLONG_max`，大于或等于 18446744073709551615。

For floating-point types `T`, `max()` is the maximum finite value while `min()` is the minimum positive normalized value. Additional members are provided for floating-point types, which are also implementation-defined but satisfy certain guarantees provided by the C standard through the `<cfloat>` header.

> 对于浮点类型“T”，“max（）”是最大有限值，而“min（）”则是最小正归一化值。为浮点类型提供了额外的成员，浮点类型也是实现定义的，但满足 C 标准通过 `<cfloat>` 标头提供的某些保证。

<li>The member `digits10` gives the number of decimal digits of precision.
<ul>
- `std::numeric_limits<float>::digits10` equals `FLT_DIG`, which is at least 6.
- `std::numeric_limits<double>::digits10` equals `DBL_DIG`, which is at least 10.
- `std::numeric_limits<long double>::digits10` equals `LDBL_DIG`, which is at least 10.

- `std::numeric_limits<float>::min_exponent10` equals `FLT_MIN_10_EXP`, which is at most -37.

<li>`std::numeric_limits<double>::min_exponent10` equals `DBL_MIN_10_EXP`, which is at most -37.
`std::numeric_limits<long double>::min_exponent10` equals `LDBL_MIN_10_EXP`, which is at most -37.</li>

- `std::numeric_limits<float>::max_exponent10` equals `FLT_MIN_10_EXP`, which is at least 37.
- `std::numeric_limits<double>::max_exponent10` equals `DBL_MIN_10_EXP`, which is at least 37.
- `std::numeric_limits<long double>::max_exponent10` equals `LDBL_MIN_10_EXP`, which is at least 37.

> -`std:：numeric_limits＜long double＞：：max_exponent10` 等于 `LDBL_MIN_10_EXP`，至少为 37。

## Value representation of floating point types

The standard requires that `long double` provides at least as much precision as `double`, which provides at least as much precision as `float`; and that a `long double` can represent any value that a `double` can represent, while a `double` can represent any value that a `float` can represent. The details of the representation are, however, implementation-defined.

> 该标准要求“长双精度”提供的精度至少与“双精度”相同，后者提供的精度也至少与“浮点”相同；“长双精度”可以表示“双精度”所能表示的任何值，而“双精度’可以表示“浮点”所能代表的任何值。然而，表示的细节是由实现定义的。

For a floating point type `T`, `std::numeric_limits<T>::radix` specifies the radix used by the representation of `T`.

> 对于浮点类型“T”，“std:：numeric_limits＜T＞：：radix”指定“T”表示所使用的基数。

If `std::numeric_limits<T>::is_iec559` is true, then the representation of `T` matches one of the formats defined by IEC 559 / IEEE 754.

> 如果“std:：numeric_limits<T>：：is_iec559”为真，则“T”的表示与 IEC 559/IEEE 754 定义的格式之一匹配。

## Overflow when converting from integer to signed integer

When either a signed or unsigned integer is converted to a signed integer type, and its value is not representable in the destination type, the value produced is implementation-defined. Example:

> 当有符号或无符号整数转换为有符号整数类型，并且其值在目标类型中不可表示时，生成的值是实现定义的。示例：

```cpp
// Suppose that on this implementation, the range of signed char is -128 to +127 and
// the range of unsigned char is 0 to 255
int x = 12345;
signed char sc = x;   // sc has an implementation-defined value
unsigned char uc = x; // uc is initialized to 57 (i.e., 12345 modulo 256)

```

## Underlying type (and hence size) of an enum

If the underlying type is not explicitly specified for an unscoped enumeration type, it is determined in an implementation-defined manner.

> 如果未显式为未分级的枚举类型指定基础类型，则以实现定义的方式确定。

```cpp
enum E {
    RED,
    GREEN,
    BLUE,
};
using T = std::underlying_type<E>::type; // implementation-defined

```

However, the standard does require the underlying type of an enumeration to be no larger than `int` unless both `int` and `unsigned int` are unable to represent all the values of the enumeration. Therefore, in the above code, `T` could be `int`, `unsigned int`, or `short`, but not `long long`, to give a few examples.

> 然而，该标准确实要求枚举的基本类型不大于“int”，除非“int”和“unsigned int”都不能表示枚举的所有值。因此，在上面的代码中，“T”可以是“int”、“unsigned int”或“short”，但不能是“long-long”，举几个例子。

Note that an enum has the same size (as returned by `sizeof`) as its underlying type.

> 请注意，枚举的大小（由“sizeof”返回）与其基础类型相同。
