---
tip: translate by baidu@2023-10-25 08:21:02
---
---

metaTitle: "C++ | Bit Operators"
description: "| - bitwise OR, ^ - bitwise XOR (exclusive OR), & - bitwise AND, << - left shift, >> - right shift, ~ - bitwise NOT (unary complement)"
-----------------------------------------------------------------------------------------------------------------------------------------------------

# Bit Operators

## | - bitwise OR

```cpp
int a = 5;     // 0101b  (0x05)
int b = 12;    // 1100b  (0x0C)
int c = a | b; // 1101b  (0x0D)

std::cout << "a = " << a << ", b = " << b << ", c = " << c << std::endl;

```

**Output**

`a = 5, b = 12, c = 13`

**Why**

A bit wise `OR` operates on the bit level and uses the following Boolean truth table:

> 逐位“OR”在位级进行运算，并使用以下布尔真值表：

```cpp
true OR true = true
true OR false = true
false OR false = false

```

When the binary value for `a` (`0101`) and the binary value for `b` (`1100`) are `OR`'ed together we get the binary value of `1101`:

> 当“a”（“0101”）的二进制值和“b”（“1100”）的二值加在一起“OR”时，我们得到“1101”的二进制值：

```cpp
int a = 0 1 0 1
int b = 1 1 0 0 |
        ---------
int c = 1 1 0 1

```

The bit wise OR does not change the value of the original values unless specifically assigned to using the bit wise assignment compound operator `|=`:

> 除非特别指定使用逐位赋值复合运算符“|=”，否则逐位 OR 不会改变原始值的值：

```cpp
int a = 5;  // 0101b  (0x05)
a |= 12;    // a = 0101b | 1101b

```

## ^ - bitwise XOR (exclusive OR)

```cpp
int a = 5;     // 0101b  (0x05)
int b = 9;     // 1001b  (0x09)
int c = a ^ b; // 1100b  (0x0C)

std::cout << "a = " << a << ", b = " << b << ", c = " << c << std::endl;

```

**Output**

`a = 5, b = 9, c = 12`

**Why**

A bit wise `XOR` (exclusive or) operates on the bit level and uses the following Boolean truth table:

> 逐位“XOR”（异或）在位级进行运算，并使用以下布尔真值表：

```cpp
true OR true = false
true OR false = true
false OR false = false

```

Notice that with an XOR operation `true OR true = false` where as with operations `true AND/OR true = true`, hence the exclusive nature of the XOR operation.

> 请注意，对于 XOR 运算“true OR true=false”，其中与运算“true AND/OR true=true”一样，因此 XOR 运算具有排他性。

Using this, when the binary value for `a` (`0101`) and the binary value for `b` (`1001`) are `XOR`'ed together we get the binary value of `1100`:

> 使用这个，当“a”（“0101”）的二进制值和“b”（“1001”）的二值被“XOR”在一起时，我们得到“1100”的二进制值：

```cpp
int a = 0 1 0 1
int b = 1 0 0 1 ^
        ---------
int c = 1 1 0 0

```

The bit wise XOR does not change the value of the original values unless specifically assigned to using the bit wise assignment compound operator `^=`:

> 除非特别指定使用逐位赋值复合运算符“^=”，否则逐位 XOR 不会更改原始值的值：

```cpp
int a = 5;  // 0101b  (0x05)
a ^= 9;    // a = 0101b ^ 1001b

```

The bit wise XOR can be utilized in many ways and is often utilized in bit mask operations for encryption and compression.

> 逐位 XOR 可以以多种方式使用，并且经常在用于加密和压缩的位掩码操作中使用。

**Note:** The following example is often shown as an example of a nice trick. But should not be used in production code (there are better ways `std::swap()` to achieve the same result).

> **注意：**下面的例子通常是一个很好的技巧的例子。但不应在生产代码中使用（有更好的方法“std:：swap（）”可以获得相同的结果）。

You can also utilize an XOR operation to swap two variables without a temporary:

> 您还可以使用 XOR 操作来交换两个变量，而无需临时：

```cpp
int a = 42;
int b = 64;

// XOR swap
a ^= b;
b ^= a;
a ^= b;

std::cout << "a = " << a << ", b = " << b << "\n";

```

To productionalize this you need to add a check to make sure it can be used.

> 要使其产品化，您需要添加一个检查以确保它可以使用。

```cpp
void doXORSwap(int& a, int& b)
{
    // Need to add a check to make sure you are not swapping the same
    // variable with itself. Otherwise it will zero the value.
    if (&a != &b)
    {
        // XOR swap
        a ^= b;
        b ^= a;
        a ^= b;
    }
}

```

So though it looks like a nice trick in isolation it is not useful in real code.

> 因此，尽管它看起来是一个很好的孤立技巧，但在实际代码中却没有用处。
> xor is not a base logical operation,but a combination of others:
> a^c=~(a&c)&(a|c)

also in 2015+ compilers variables may be assigned as binary:

```cpp
int cn=0b0111;

```

## & - bitwise AND

```cpp
int a = 6;     // 0110b  (0x06)
int b = 10;    // 1010b  (0x0A)
int c = a & b; // 0010b  (0x02)

std::cout << "a = " << a << ", b = " << b << ", c = " << c << std::endl;

```

**Output**

`a = 6, b = 10, c = 2`

**Why**

A bit wise `AND` operates on the bit level and uses the following Boolean truth table:

> 逐位“AND”在位级别上操作，并使用以下布尔真值表：

```cpp
TRUE  AND TRUE  = TRUE
TRUE  AND FALSE = FALSE
FALSE AND FALSE = FALSE

```

When the binary value for `a` (`0110`) and the binary value for `b` (`1010`) are `AND`'ed together we get the binary value of `0010`:

> 当“a”（“0110”）的二进制值和“b”（“1010”）的二值一起“与”时，我们得到二进制值“0010”：

```cpp
int a = 0 1 1 0
int b = 1 0 1 0 &
        ---------
int c = 0 0 1 0

```

The bit wise AND does not change the value of the original values unless specifically assigned to using the bit wise assignment compound operator `&=`:

> 逐位 AND 不会改变原始值的值，除非特别指定使用逐位赋值复合运算符“&=”：

```cpp
int a = 5;  // 0101b  (0x05)
a &= 10;    // a = 0101b & 1010b

```

## << - left shift

```cpp
int a = 1;      // 0001b
int b = a << 1; // 0010b

std::cout << "a = " << a << ", b = " << b << std::endl;

```

**Output**

`a = 1, b = 2`

**Why**

The left bit wise shift will shift the bits of the left hand value (`a`) the number specified on the right (`1`), essentially padding the least significant bits with 0's, so shifting the value of `5` (binary `0000 0101`) to the left 4 times (e.g. `5 << 4`) will yield the value of `80` (binary `0101 0000`).  You might note that shifting a value to the left 1 time is also the same as multiplying the value by 2, example:

> 左逐位移位将使左边值（‘a’）的位移位到右边指定的数字（‘1’），基本上用 0 填充最低有效位，因此将‘5’（二进制‘0000 0101’）的值向左移位 4 次（例如，‘5＜＜4’）将产生‘80’（二进制“0101 0000”）的值。您可能会注意到，将值向左移动 1 次也与将值乘以 2 相同，例如：

```cpp
int a = 7;
while (a < 200) {
    std::cout << "a = " << a << std::endl;
    a <<= 1;
}

a = 7;
while (a < 200) {
    std::cout << "a = " << a << std::endl;
    a *= 2;
}

```

But it should be noted that the left shift operation will shift *all* bits to the left, including the sign bit, example:

> 但应该注意的是，左移位操作将把*所有*位向左移位，包括符号位，例如：

```cpp
int a = 2147483647; // 0111 1111 1111 1111 1111 1111 1111 1111
int b = a << 1;     // 1111 1111 1111 1111 1111 1111 1111 1110

std::cout << "a = " << a << ", b = " << b << std::endl;

```

Possible output: `a = 2147483647, b = -2`

While some compilers will yield results that seem expected, it should be noted that if you left shift a signed number so that the sign bit is affected, the result is **undefined**. It is also **undefined** if the number of bits you wish to shift by is a negative number or is larger than the number of bits the type on the left can hold, example:

> 虽然有些编译器会产生预期的结果，但需要注意的是，如果对有符号数字进行左移，从而影响符号位，则结果为**未定义**。如果您希望移位的位数是负数或大于左侧类型可以容纳的位数，则也是**未定义**，例如：

```cpp
int a = 1;
int b = a << -1;  // undefined behavior
char c = a << 20; // undefined behavior

```

The bit wise left shift does not change the value of the original values unless specifically assigned to using the bit wise assignment compound operator `<<=`:

> 逐位左移不会改变原始值的值，除非特别指定使用逐位赋值复合运算符“<<=”：

```cpp
int a = 5;  // 0101b
a <<= 1;    // a = a << 1;

```

## >> - right shift

```cpp
int a = 2;      // 0010b
int b = a >> 1; // 0001b

std::cout << "a = " << a << ", b = " << b << std::endl;

```

**Output**

`a = 2, b = 1`

**Why**

The right bit wise shift will shift the bits of the left hand value (`a`) the number specified on the right (`1`); it should be noted that while the operation of a right shift is standard, what happens to the bits of a right shift on a *signed negative* number is *implementation defined* and thus cannot be guaranteed to be portable, example:

> 右移将把左边值（‘a’）的位移到右边指定的数字（‘1’）；应该注意的是，虽然右移的操作是标准的，但在*有符号负数*上发生的右移位是*实现定义的*，因此不能保证是可移植的，例如：

```cpp
int a = -2;    
int b = a >> 1; // the value of b will be depend on the compiler

```

It is also undefined if the number of bits you wish to shift by is a negative number, example:

> 如果您希望移位的位数是负数，则也未定义，例如：

```cpp
int a = 1;
int b = a >> -1;  // undefined behavior

```

The bit wise right shift does not change the value of the original values unless specifically assigned to using the bit wise assignment compound operator `>>=`:

> 逐位右移不会改变原始值的值，除非特别指定使用逐位赋值复合运算符“>>=”：

```cpp
int a = 2;  // 0010b
a >>= 1;    // a = a >> 1;

```

## ~ - bitwise NOT (unary complement)

```cpp
unsigned char a = 234;  // 1110 1010b  (0xEA)
unsigned char b = ~a;   // 0001 0101b  (0x15)

std::cout << "a = " << static_cast<int>(a) <<
             ", b = " << static_cast<int>(b) << std::endl;

```

**Output**

`a = 234, b = 21`

**Why**

A bit wise `NOT` (unary complement) operates on the bit level and simply flips each bit. If it's a `1`, it's changed to a `0`, if it's a `0`, it's changed to a `1`. The bit wise NOT has the same effect as XOR'ing a value against the max value for a specific type:

> 按位“NOT”（一元补码）在位级别上操作，并简单地翻转每个位。如果是“1”，它将变为“0”；如果是“0”，它会变为“1”。对于特定类型，逐位 NOT 具有与将值与最大值进行异或相同的效果：

```cpp
unsigned char a = 234;  // 1110 1010b  (0xEA)
unsigned char b = ~a;   // 0001 0101b  (0x15)
unsigned char c = a ^ ~0;

```

The bit wise NOT can also be a convenient way to check the maximum value for a specific integral type:

> 逐位 NOT 也是检查特定积分类型的最大值的一种方便方法：

```cpp
unsigned int i = ~0;
unsigned char c = ~0;

std::cout << "max uint = " << i << std::endl <<
             "max uchar = " << static_cast<short>(c) << std::endl;

```

The bit wise NOT does not change the value of the original value and does not have a compound assignment operator, so you can not do `a ~= 10` for example.

> 按位 NOT 不会更改原始值的值，也没有复合赋值运算符，因此不能执行“a~=10”等操作。

The *bit wise* NOT (`~`) should not be confused with the *logical* NOT (`!`); where a bit wise NOT will flip each bit, a logical NOT will use the whole value to do its operation on, in other words `(!1) != (~1)`

> 不应将*位* NOT（`~`）与*逻辑* NOT（`！`）混淆；其中按位 NOT 将翻转每个位，逻辑 NOT 将使用整个值对其进行运算，换句话说 `（！1）！=（~1）`

#### Remarks

Bit shift operations are not portable across all processor architectures, different processors can have different bit-widths. In other words, if you wrote

> 并非所有处理器架构都可以移植位移操作，不同的处理器可能具有不同的位宽。换句话说，如果你写

```cpp
int a = ~0;
int b = a << 1;

```

This value would be different on a 64 bit machine vs. on a 32 bit machine, or from an x86 based processor to a PIC based processor.

> 该值在 64 位机器上与在 32 位机器上不同，或者在基于 x86 的处理器与基于 PIC 的处理器之间不同。

Endian-ness does not need to be taken into account for the bit wise operations themselves, that is, the right shift (`>>`) will shift the bits towards the least significant bit and an XOR will perform an exclusive or on the bits. Endian-ness only needs to be taken into account with the data itself, that is, if endian-ness is a concern for your application, it's a concern regardless of bit wise operations.

> 逐位运算本身不需要考虑 Endianness，也就是说，右移（“>>`）将把位移向最低有效位，XOR 将对位执行异或。Endian-ness 只需要在数据本身中考虑，也就是说，如果 Endian-ness 是应用程序的一个问题，那么无论位操作如何，它都是一个问题。
