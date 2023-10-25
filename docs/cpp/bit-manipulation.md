---
tip: translate by baidu@2023-10-25 08:20:57
---
---
metaTitle: "C++ | Bit Manipulation"
description: "Toggling a bit, Checking a bit, Set all bits, Remove rightmost set bit, Setting a bit, Clearing a bit, Changing the nth bit to x, Counting bits set, Check if an integer is a power of 2, Bit Manipulation Application: Small to Capital  Letter"
---

# Bit Manipulation



## Toggling a bit


### C-style bit-manipulation

A bit can be toggled using the XOR operator (`^`).

```cpp
// Bit x will be the opposite value of what it is currently
number ^= 1LL << x;

```

### Using std::bitset

```cpp
std::bitset<4> num(std::string("0100"));
num.flip(2); // num is now 0000
num.flip(0); // num is now 0001
num.flip();  // num is now 1110 (flips all bits)

```



## Checking a bit


### C-style bit-manipulation


The value of the bit can be obtained by shifting the number to the right `x` times and then performing bitwise AND (`&`) on it:

> 该位的值可以通过将数字向右移位“x”次，然后对其执行逐位and（“&”）来获得：

```cpp
(number >> x) & 1LL;  // 1 if the 'x'th bit of 'number' is set, 0 otherwise

```


The right-shift operation may be implemented as either an arithmetic (signed) shift or a logical (unsigned) shift. If `number` in the expression `number >> x` has a signed type and a negative value, the resulting value is implementation-defined.

> 右移操作可以实现为算术（有符号）移位或逻辑（无符号）移位。如果表达式“number>>x”中的“number”具有带符号类型和负值，则生成的值是实现定义的。


If we need the value of that bit directly in-place, we could instead left shift the mask:

> 如果我们需要该位的值直接到位，我们可以左移掩码：

```cpp
(number & (1LL << x));  // (1 << x) if the 'x'th bit of 'number' is set, 0 otherwise

```


Either can be used as a conditional, since all non-zero values are considered true.

> 任何一个都可以用作条件，因为所有非零值都被认为是true。

### Using std::bitset

```cpp
std::bitset<4> num(std::string("0010"));
bool bit_val = num.test(1);  // bit_val value is set to true;

```



## Set all bits


### C-style bit-manipulation

```cpp
x = -1; // -1 == 1111 1111 ... 1111b

```

<sup>(See [here](http://stackoverflow.com/a/809341/) for an explanation of why this works and is actually the best approach.)</sup>

### Using std::bitset

```cpp
std::bitset<10> x;
x.set(); // Sets all bits to '1'

```



## Remove rightmost set bit


### C-style bit-manipulation

```cpp
template <typename T>
T rightmostSetBitRemoved(T n)
{
    // static_assert(std::is_integral<T>::value && !std::is_signed<T>::value, "type should be unsigned"); // For c++11 and later
    return n & (n - 1);
}

```

**Explanation**

- if `n` is zero, we have `0 & 0xFF..FF` which is zero

- else `n` can be written `0bxxxxxx10..00` and `n - 1` is `0bxxxxxx011..11`, so `n & (n - 1)` is `0bxxxxxx000..00`.

> -否则“n”可以写成“0bxxxxxx10..00”，“n-1”是“0bxxxx011..11”，所以“n&（n-1）”是“0 bxxxxxx0..00”。



## Setting a bit


### C-style bit manipulation

A bit can be set using the bitwise OR operator (`|`).

```cpp
// Bit x will be set
number |= 1LL << x; 

```

### Using std::bitset

`set(x)` or `set(x,true)` - sets bit at position `x` to `1`.

```cpp
std::bitset<5> num(std::string("01100"));
num.set(0);      // num is now 01101
num.set(2);      // num is still 01101
num.set(4,true); // num is now 11110

```



## Clearing a bit


### C-style bit-manipulation

A bit can be cleared using the bitwise AND operator (`&`).

```cpp
// Bit x will be cleared
number &= ~(1LL << x);

```

### Using std::bitset

`reset(x)` or `set(x,false)` - clears the bit at position `x`.

```cpp
std::bitset<5> num(std::string("01100"));
num.reset(2);     // num is now 01000
num.reset(0);     // num is still 01000
num.set(3,false); // num is now 00000

```



## Changing the nth bit to x


### C-style bit-manipulation

```cpp
// Bit n will be set if x is 1 and cleared if x is 0.
number ^= (-x ^ number) & (1LL << n);

```

### Using std::bitset

`set(n,val)` - sets bit `n` to the value `val`.

```cpp
std::bitset<5> num(std::string("00100"));
num.set(0,true);  // num is now 00101
num.set(2,false); // num is now 00001

```



## Counting bits set



The population count of a bitstring is often needed in cryptography and other applications and the problem has been widely studied.

> 比特串的总体计数在密码学和其他应用中经常需要，并且这个问题已经被广泛研究。

The naive way requires one iteration per bit:

```cpp
unsigned value = 1234;
unsigned bits = 0;  // accumulates the total number of bits set in `n`

for (bits = 0; value; value >>= 1)
  bits += value & 1;

```


A nice trick (based on [Remove rightmost set bit](http://stackoverflow.com/documentation/c%2b%2b/3016/bit-manipulation/17299/remove-rightmost-set-bit) ) is:

> 一个不错的技巧（基于[删除最右边的设置位](http://stackoverflow.com/documentation/c%2b%2b/3016/bit-操作/17299/移除最右边的设置位）是：

```cpp
unsigned bits = 0;  // accumulates the total number of bits set in `n`

for (; value; ++bits)
  value &= value - 1;

```


It goes through as many iterations as there are set bits, so it's good when `value` is expected to have few nonzero bits.

> 它会经历与设置位一样多的迭代，所以当“值”预计只有很少的非零位时，这是很好的。


The method was first proposed by Peter Wegner (in [CACM](http://cacm.acm.org/) 3 / 322 - 1960) and it's well known since it appears in **C Programming Language** by Brian W. Kernighan and Dennis M. Ritchie.

> 该方法最早由Peter Wegner（在[CACM](http://cacm.acm.org/)3/322-1960），因为它出现在Brian W.Kernighan和Dennis M.Ritchie的**C编程语言**中而广为人知。

This requires 12 arithmetic operations, one of which is a multication:

```cpp
unsigned popcount(std::uint64_t x)
{
  const std::uint64_t m1  = 0x5555555555555555;  // binary: 0101...
  const std::uint64_t m2  = 0x3333333333333333;  // binary: 00110011..
  const std::uint64_t m4  = 0x0f0f0f0f0f0f0f0f;  // binary: 0000111100001111

  x -= (x >> 1) & m1;             // put count of each 2 bits into those 2 bits
  x = (x & m2) + ((x >> 2) & m2); // put count of each 4 bits into those 4 bits 
  x = (x + (x >> 4)) & m4;        // put count of each 8 bits into those 8 bits 
  return (x * h01) >> 56;  // left 8 bits of x + (x<<8) + (x<<16) + (x<<24) + ... 
}

```


This kind of implementation has the best worst-case behavior (see [Hamming weight](https://en.wikipedia.org/wiki/Hamming_weight) for further details).

> 这种实现具有最佳的最坏情况行为（参见[Haming weight](https://en.wikipedia.org/wiki/Hamming_weight)了解更多细节）。


Many CPUs have a specific instruction (like x86's `popcnt`) and the compiler could offer a specific (**non standard**) built in function. E.g. with g++ there is:

> 许多CPU都有特定的指令（如x86的“popcnt”），编译器可以提供特定的（**非标准**）内置函数。例如，对于g++，存在：

```cpp
int __builtin_popcount (unsigned x);

```



## Check if an integer is a power of 2



The `n & (n - 1)` trick (see [Remove rightmost set bit](http://stackoverflow.com/documentation/c%2b%2b/3016/bit-manipulation/17299/remove-rightmost-set-bit)) is also useful to determine if an integer is a power of 2:

> `n&（n-1）`技巧（请参阅[删除最右边的设置位](http://stackoverflow.com/documentation/c%2b%2b/3016/bit-操作/17299/移除最右边的设置位）对于确定整数是否是2的幂也是有用的：

```cpp
bool power_of_2 = n && !(n & (n - 1));

```


Note that without the first part of the check (`n &&`), `0` is incorrectly considered a power of 2.

> 注意，如果没有校验的第一部分（“n&&”），“0”被错误地认为是2的幂。



## Bit Manipulation Application: Small to Capital  Letter



One of several applications of bit manipulation is converting a letter from small to capital or vice versa by choosing a **mask** and a proper **bit operation**. For example, the **a** letter has this binary representation `01(1)00001` while its capital counterpart has `01(0)00001`. They differ solely in the bit in parenthesis. In this case, converting the **a** letter from small to capital is basically setting the bit in parenthesis to one. To do so, we do the following:

> 位操作的几个应用之一是通过选择**掩码**和适当的**位操作**将字母从小写转换为大写，反之亦然。例如，**a**字母具有二进制表示“01（1）00001”，而其大写对应字母具有“01（0）00001）”。它们的区别仅在于括号中的位。在这种情况下，将**a**字母从小写转换为大写基本上就是将括号中的位设置为1。为此，我们执行以下操作：

```cpp
/****************************************
convert small letter to captial letter.
========================================
     a: 01100001
  mask: 11011111  <-- (0xDF)  11(0)11111
      :---------
a&mask: 01000001  <-- A letter
*****************************************/

```

The code for converting a letter to A letter is

```cpp
#include <cstdio>

int main()
{
    char op1 = 'a';  // "a" letter (i.e. small case)
    int mask = 0xDF; // choosing a proper mask

    printf("a (AND) mask = A\n");
    printf("%c   &   0xDF = %c\n", op1, op1 & mask);
    
    return 0;
}

```

The result is

```cpp
$ g++ main.cpp -o test1
$ ./test1
a (AND) mask = A
a   &   0xDF = A

```



#### Remarks



In order to use [`std::bitset`](http://en.cppreference.com/w/cpp/utility/bitset) you will have to include [`<bitset>` header](http://en.cppreference.com/w/cpp/header/bitset).

> 为了使用[`std:：bitset`](http://en.cppreference.com/w/cpp/utility/bitset)您必须包含[`<bitset>`header](http://en.cppreference.com/w/cpp/header/bitset)。

```cpp
#include <bitset>

```

`std::bitset` overloads all of the operator functions to allow the same usage as the c-style handling of bitsets.

**References**

- [Bit Twiddling Hacks](https://graphics.stanford.edu/%7Eseander/bithacks.html)

