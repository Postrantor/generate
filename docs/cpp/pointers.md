---
tip: translate by baidu@2023-10-25 08:29:32
---
---

metaTitle: "C++ | Pointers"
description: "Pointer basics, Pointer Operations, Pointer Arithmetic"
---------------------------------------------------------------------

# Pointers

A pointer is an address that refers to a location in memory. They're commonly used to allow functions or data structures to know of and modify memory without having to copy the memory referred to. Pointers are usable with both primitive (built-in) or user-defined types.

> 指针是指内存中某个位置的地址。指针通常用于允许函数或数据结构了解和修改内存，而不必复制引用的内存。指针既可用于基元（内置）类型，也可用于用户定义类型。

Pointers make use of the "dereference" `*` , "address of" `&` , and "arrow" `->` operators. The '*' and '->' operators are used to access the memory being pointed at, and the `&` operator is used to get an address in memory.

> 指针使用“取消引用”`*`、“地址”`&` 和“箭头”`->` 运算符。“*”和“->”运算符用于访问所指向的内存，“&”运算符用于获取内存中的地址。

## Pointer basics

**Note:** in all the following, the existence of the C++11 constant `nullptr` is assumed. For earlier versions, replace `nullptr` with `NULL`, the constant that used to play a similar role.

> **注意：**在以下所有内容中，假定存在 C++11 常量“nullptr”。对于早期版本，请将“nullptr”替换为“NULL”，后者是用于扮演类似角色的常量。

### Creating a pointer variable

A pointer variable can be created using the specific `*` syntax, e.g. `int *pointer_to_int;`.<br />

> 指针变量可以使用特定的“*”语法创建，例如“int*pointer_to_int；”<br/>

When a variable is of a **pointer type** (`int *`), it just contains a memory address. The memory address is the location at which data of the **underlying type** (`int`) is stored.

> 当变量为**指针类型**（“int*”）时，它只包含一个内存地址。存储器地址是存储**基础类型**（“int”）的数据的位置。

The difference is clear when comparing the size of a variable with the size of a pointer to the same type:

> 当将变量的大小与指向同一类型的指针的大小进行比较时，差异显而易见：

```cpp
// Declare a struct type `big_struct` that contains
// three long long ints.
typedef struct {
    long long int foo1;
    long long int foo2;
    long long int foo3;
} big_struct;

// Create a variable `bar` of type `big_struct`
big_struct bar;
// Create a variable `p_bar` of type `pointer to big_struct`.
// Initialize it to `nullptr` (a null pointer).
big_struct *p_bar0 = nullptr;

// Print the size of `bar`
std::cout << "sizeof(bar) = " << sizeof(bar) << std::endl;
// Print the size of `p_bar`.
std::cout << "sizeof(p_bar0) = " << sizeof(p_bar0) << std::endl;

/* Produces:
    sizeof(bar) = 24
    sizeof(p_bar0) = 8
*/

```

### Taking the address of another variable

Pointers can be assigned between each other just as normal variables; in this case, it is the **memory address** that is copied from one pointer to another, **not the actual data** that a pointer points to.<br />

> 指针可以像正常变量一样在彼此之间分配；在这种情况下，从一个指针复制到另一个指针的是**内存地址**，而不是指针指向的实际数据**。<br/>

Moreover, they can take the value `nullptr` which represents a null memory location. A pointer equal to `nullptr` contains an invalid memory location and hence it does not refer to valid data.

> 此外，它们可以取表示空内存位置的值“nullptr”。等于“nullptr”的指针包含无效的内存位置，因此它不引用有效数据。

You can get the memory address of a variable of a given type by prefixing the variable with the **address of** operator `&`. The value returned by `&` is a pointer to the underlying type which contains the memory address of the variable (which is valid data **as long as the variable does not go out of scope**).

> 通过在给定类型的变量前面加上**运算符“&”的**地址，可以获得该变量的内存地址。“&”返回的值是一个指向底层类型的指针，该类型包含变量的内存地址（只要变量不超出范围**，它就是有效数据**）。

```cpp
// Copy `p_bar0` into `p_bar_1`.
big_struct *p_bar1 = p_bar0;

// Take the address of `bar` into `p_bar_2`
big_struct *p_bar2 = &bar;

// p_bar1 is now nullptr, p_bar2 is &bar.

p_bar0 = p_bar2;

// p_bar0 is now &bar.

p_bar2 = nullptr;

// p_bar0 == &bar
// p_bar1 == nullptr
// p_bar2 == nullptr

```

In contrast with references:

- assigning two pointers does not overwrite the memory that the assigned pointer refers to;
- pointers can be null.
- the **address of** operator is required explicitly.

### Accessing the content of a pointer

As taking an address requires `&`, as well accessing the content requires the usage of the **dereference operator** `*`, as a prefix. When a pointer is dereferenced, it becomes a variable of the underlying type (actually, a reference to it). It can then be read and modified, if not `const`.

> 由于获取地址需要“&”，访问内容也需要使用**解引用运算符***“*”作为前缀。当一个指针被取消引用时，它将成为基础类型的变量（实际上是对它的引用）。如果不是“const”，则可以读取和修改它。

```cpp
(*p_bar0).foo1 = 5;

// `p_bar0` points to `bar`. This prints 5.
std::cout << "bar.foo1 = " << bar.foo1 << std::endl;

// Assign the value pointed to by `p_bar0` to `baz`.
big_struct baz;
baz = *p_bar0;

// Now `baz` contains a copy of the data pointed to by `p_bar0`.
// Indeed, it contains a copy of `bar`.

// Prints 5 as well
std::cout << "baz.foo1 = " << baz.foo1 << std::endl;

```

The combination of `*` and the operator `.` is abbreviated by `->`:

```cpp
std::cout << "bar.foo1 = " << (*p_bar0).foo1 << std::endl; // Prints 5
std::cout << "bar.foo1 = " <<  p_bar0->foo1  << std::endl; // Prints 5

```

### Dereferencing invalid pointers

When dereferencing a pointer, you should make sure it points to valid data. Dereferencing an invalid pointer (or a null pointer) can lead to memory access violation, or to read or write garbage data.

> 取消引用指针时，应确保指针指向有效数据。取消引用无效指针（或空指针）可能导致内存访问冲突，或读取或写入垃圾数据。

```cpp
big_struct *never_do_this() {
   // This is a local variable. Outside `never_do_this` it doesn't exist.
   big_struct retval;
   retval.foo1 = 11;
   // This returns the address of `retval`.
   return &retval;
   // `retval` is destroyed and any code using the value returned
   // by `never_do_this` has a pointer to a memory location that
   // contains garbage data (or is inaccessible).
}

```

In such scenario, `g++` and `clang++` correctly issue the warnings:

```cpp
(Clang) warning: address of stack memory associated with local variable 'retval' returned [-Wreturn-stack-address]
(Gcc)   warning: address of local variable ‘retval’ returned [-Wreturn-local-addr]

```

Hence, care must be taken when pointers are arguments of functions, as they could be null:

> 因此，当指针是函数的参数时必须小心，因为它们可能为 null：

```cpp
void naive_code(big_struct *ptr_big_struct) {
    // ... some code which doesn't check if `ptr_big_struct` is valid.
    ptr_big_struct->foo1 = 12;
}

// Segmentation fault.
naive_code(nullptr);

```

## Pointer Operations

There are two operators for pointers:
Address-of operator (&): Returns the memory address of its operand.

Contents-of (Dereference) operator(*): Returns the value of the variable located at the address specified by its operator.

> （Dereference）运算符（*）的内容：返回位于其运算符指定地址的变量的值。

```cpp
int var = 20;
int *ptr;
ptr = &var;

cout << var << endl;
//Outputs 20 (The value of var)

cout << ptr << endl;
//Outputs 0x234f119 (var's memory location)

cout << *ptr << endl;
//Outputs 20(The value of the variable stored in the pointer ptr

```

The asterisk (*) is used in declaring a pointer for simple purpose of indicating that it is a pointer. Don't confuse this with the **dereference** operator, which is used to obtain the value located at the specified address. They are simply two different things represented with the same sign.

> 星号（*）用于声明指针，目的很简单，表示它是指针。不要将其与**取消引用**运算符混淆，后者用于获取位于指定地址的值。它们只是用同一个符号表示的两个不同的东西。

## Pointer Arithmetic

### Increment / Decrement

A pointer can be incremented or decremented (prefix and postfix). Incrementing a pointer advances the pointer value to the element in the array one element past the currently pointed to element. Decrementing a pointer moves it to the previous element in the array.

> 指针可以递增或递减（前缀和后缀）。递增指针会使指向数组中元素的指针值超过当前指向的元素一个元素。指针递减会将其移动到数组中的上一个元素。

Pointer arithmetic is not permitted if the type that the pointer points to is not complete. `void` is always an incomplete type.

> 如果指针指向的类型不完整，则不允许使用指针算术 `void 总是一个不完整的类型。

```cpp
char* str = new char[10]; // str = 0x010
++str;                    // str = 0x011  in this case sizeof(char) = 1 byte

int* arr = new int[10];   // arr = 0x00100
++arr;                    // arr = 0x00104 if sizeof(int) = 4 bytes

void* ptr = (void*)new char[10];
++ptr;    // void is incomplete.

```

If a pointer to the end element is incremented, then the pointer points to one element past the end of the array. Such a pointer cannot be dereferenced, but it can be decremented.

> 如果指向结束元素的指针递增，则指针指向数组结束后的一个元素。这样的指针不能取消引用，但可以递减。

Incrementing a pointer to the one-past-the-end element in the array, or decrementing a pointer to the first element in an array yields undefined behavior.

> 将指向数组中经过结束元素的指针的指针递增，或将指向数组的第一个元素的指针递减，会产生未定义的行为。

A pointer to a non-array object can be treated, for the purposes of pointer arithmetic, as though it were an array of size 1.

> 为了指针算术的目的，可以将指向非数组对象的指针视为大小为 1 的数组。

### Addition / Subtraction

Integer values can be added to pointers; they act as incrementing, but by a specific number rather than by 1. Integer values can be subtracted from pointers as well, acting as pointer decrementing. As with incrementing/decrementing, the pointer must point to a complete type.

> 可以将整数值添加到指针中；它们的作用是递增，但递增一个特定的数字，而不是 1。整数值也可以从指针中减去，起到指针递减的作用。与递增/递减一样，指针必须指向一个完整的类型。

```cpp
char* str = new char[10];  // str = 0x010
str += 2;                  // str = 0x010 + 2 * sizeof(char) = 0x012

int* arr = new int[10];    // arr = 0x100
arr += 2;                  // arr = 0x100 + 2 * sizeof(int) = 0x108, assuming sizeof(int) == 4.

```

### Pointer Differencing

The difference between two pointers to the same type can be computed. The two pointers must be within the same array object; otherwise undefined behavior results.

> 可以计算指向相同类型的两个指针之间的差异。这两个指针必须位于同一数组对象内；否则会产生未定义的行为。

Given two pointers `P` and `Q` in the same array, if `P` is the `i` th element in the array, and `Q` is the `j` th element, then `P - Q` shall be `i - j`. The type of the result is `std::ptrdiff_t`, from `<cstddef>`.

> 给定同一数组中的两个指针“P”和“Q”，如果“P”是数组中的第 i 个元素，而“Q”是第 j 个元素，则“P-Q”应为“i-j”。结果的类型是“std:：ptrdiff_t”，来自“<cstdedf>'”。

```cpp
char* start = new char[10];  // str = 0x010
char* test = &start[5];
std::ptrdiff_t diff = test - start; //Equal to 5.
std::ptrdiff_t diff = start - test; //Equal to -5; ptrdiff_t is signed.

```

#### Syntax

- <Data type> *<Variable name>;
- <Data type> *<Variable name> = &<Variable name of same Data type>;
- <Data type> *<Variable name> = <Value of same Data type>;
- int *foo; //A pointer which would points to an integer value
- int *bar = &myIntVar;
- long *bar[2];
- long *bar[] = {&myLongVar1, &myLongVar2}; //Equals to: long *bar[2]

#### Remarks

Be aware of problems when declaring multiple pointers on the same line.

```cpp
int* a, b, c; //Only a is a pointer, the others are regular ints.

int* a, *b, *c; //These are three pointers!

int *foo[2]; //Both *foo[0] and *foo[1] are pointers.

```
