---
tip: translate by baidu@2023-10-25 08:20:53
---
---
metaTitle: "C++ | Bit fields"
description: "Declaration and Usage"
---

# Bit fields



Bit fields tightly pack C and C++ structures to reduce size.  This appears painless: specify the number of bits for members, and compiler does the work of co-mingling bits.  The restriction is inability to take the address of a bit field member, since it is stored co-mingled.  `sizeof()` is also disallowed.

> 位字段紧密地封装C和C++结构以减小大小。这看起来很简单：指定成员的位数，编译器完成混合位的工作。限制是无法获取位字段成员的地址，因为它是混合存储的`sizeof（）`也是不允许的。


The cost of bit fields is slower access, as memory must be retrieved and bitwise operations applied to extract or modify member values.  These operations also add to executable size.

> 位字段的代价是访问速度较慢，因为必须检索内存并应用逐位操作来提取或修改成员值。这些操作还会增加可执行文件的大小。



## Declaration and Usage


```cpp
struct FileAttributes
{
    unsigned int ReadOnly: 1;    
    unsigned int Hidden: 1;
};

```


Here, each of these two fields will occupy 1 bit in memory. It is specified by **`: 1`** expression after the variable names. Base type of bit field could be any integral type (8-bit int to 64-bit int). Using `unsigned` type is recommended, otherwise surprises may come.

> 这里，这两个字段中的每一个都将占用内存中的1位。它由变量名后面的**`：1`**表达式指定。位字段的基类型可以是任何整数类型（8位int到64位int）。建议使用“unsigned”类型，否则可能会出现意外。


If more bits are required, replace "1" with number of bits required. For example:

> 如果需要更多的位，请将“1”替换为所需的位数。例如：

```cpp
struct Date
{
    unsigned int Year : 13; // 2^13 = 8192, enough for "year" representation for long time
    unsigned int Month: 4;  // 2^4 = 16, enough to represent 1-12 month values.
    unsigned int Day:   5;  // 32
};

```


The whole structure is using just 22 bits, and with normal compiler settings, `sizeof` this structure would be 4 bytes.

> 整个结构只使用了22位，在正常的编译器设置下，这个结构的“sizeof”将是4个字节。


Usage is pretty simple. Just declare the variable, and use it like ordinary structure.

> 用法非常简单。只需声明变量，并像使用普通结构一样使用它。

```cpp
Date d;

d.Year = 2016;
d.Month = 7;
d.Day =  22;

std::cout << "Year:" << d.Year << std::endl <<
        "Month:" << d.Month << std::endl <<
        "Day:" << d.Day << std::endl;

```



#### Remarks



How expensive are the bitwise operations?  Suppose a simple non-bit field structure:

> 按位操作的成本有多高？假设一个简单的非位字段结构：

```cpp
struct foo {
    unsigned x;
    unsigned y;
}
static struct foo my_var;

```

In some later code:

```cpp
my_var.y = 5;

```


If `sizeof (unsigned) == 4`, then x is stored at the start of the structure, and y is stored 4 bytes in.  Assembly code generated may resemble:

> 如果“sizeof（unsigned）==4”，则x存储在结构的开头，y存储在中的4个字节。生成的程序集代码可能类似于：

```cpp
loada register1,#myvar     ; get the address of the structure
storei register1[4],#0x05  ; put the value '5' at offset 4, e.g., set y=5

```


This is straightforward because x is not co-mingled with y.  But imagine redefining the structure with bit fields:

> 这很简单，因为x不与y混在一起。但想象一下用位字段重新定义结构：

```cpp
struct foo {
    unsigned x : 4; /* Range 0-0x0f, or 0 through 15 */
    unsigned y : 4;
}

```


Both `x` and `y` will be allocated 4 bits, sharing a single byte.  The structure thus takes up 1 byte, instead of 8.  Consider the assembly to set `y` now, assuming it ends up in the upper nibble:

> “x”和“y”都将被分配4个比特，共享一个字节。因此，该结构占用了1个字节，而不是8个字节。现在考虑将程序集设置为“y”，假设它最终位于上半字节中：

```cpp
loada  register1,#myvar        ; get the address of the structure
loadb  register2,register1[0]  ; get the byte from memory
andb   register2,#0x0f         ; zero out y in the byte, leaving x alone
orb    register2,#0x50         ; put the 5 into the 'y' portion of the byte
stb    register1[0],register2  ; put the modified byte back into memory

```


This may be a good trade-off if we have thousands or millions of these structures, and it helps keeps memory in cache or prevents swapping—or could bloat the executable to worsen these problems and slow processing.  As with all things, use good judgement.

> 如果我们有成千上万或数百万个这样的结构，这可能是一个很好的权衡，它有助于将内存保持在缓存中或防止交换——或者可能会使可执行文件膨胀，从而加剧这些问题并减缓处理速度。和对待所有事情一样，要有良好的判断力。


**Device driver use:** Avoid bit fields as a clever implementation strategy for device drivers.  Bit field storage layouts are not necessarily consistent between compilers, making such implementations non-portable.  The read-modify-write to set values may not do what devices expect, causing unexpected behaviors.

> **设备驱动程序使用：**避免位字段作为设备驱动程序的巧妙实现策略。编译器之间的位字段存储布局不一定一致，这使得此类实现不可移植。读取-修改-写入-设置值可能无法达到设备预期的效果，从而导致意外行为。

