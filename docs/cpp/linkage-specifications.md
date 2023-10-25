---
tip: translate by baidu@2023-10-25 08:27:10
---
---

metaTitle: "C++ | Linkage specifications"
description: "Signal handler for Unix-like operating system, Making a C library header compatible with C++"
-----------------------------------------------------------------------------------------------------------

# Linkage specifications

A linkage specification tells the compiler to compile declarations in a way that allows them to be linked together with declarations written in another language, such as C.

> 链接规范告诉编译器编译声明的方式允许它们与用另一种语言（如 C）编写的声明链接在一起。

## Signal handler for Unix-like operating system

Since a signal handler will be called by the kernel using the C calling convention, we must tell the compiler to use the C calling convention when compiling the function.

> 由于内核将使用 C 调用约定来调用信号处理程序，因此我们必须告诉编译器在编译函数时使用 C 调用惯例。

```cpp
volatile sig_atomic_t death_signal = 0;
extern "C" void cleanup(int signum) {
    death_signal = signum;
}
int main() {
    bind(...);
    listen(...);
    signal(SIGTERM, cleanup);
    while (int fd = accept(...)) {
        if (fd == -1 && errno == EINTR && death_signal) {
            printf("Caught signal %d; shutting down\n", death_signal);
            break;
        }
        // ...
    }
}

```

## Making a C library header compatible with C++

A C library header can usually be included into a C++ program, since most declarations are valid in both C and C++. For example, consider the following `foo.h`:

> C 库头通常可以包含在 C++ 程序中，因为大多数声明在 C 和 C++ 中都是有效的。例如，考虑以下“foo.h”：

```cpp
typedef struct Foo {
    int bar;
} Foo;
Foo make_foo(int);

```

The definition of `make_foo` is separately compiled and distributed with the header in object form.

> “make_foo”的定义是单独编译和分发的，带有对象形式的标头。

A C++ program can `#include <foo.h>`, but the compiler will not know that the `make_foo` function is defined as a C symbol, and will probably try to look for it with a mangled name, and fail to locate it. Even if it can find the definition of `make_foo` in the library, not all platforms use the same calling conventions for C and C++, and the C++ compiler will use the C++ calling convention when calling `make_foo`, which is likely to cause a segmentation fault if `make_foo` is expecting to be called with the C calling convention.

> C++ 程序可以“#include＜foo.h＞”，但编译器不会知道“make_foo”函数被定义为 C 符号，可能会试图用一个错误的名称来查找它，但找不到它。即使它能在库中找到“make_foo'”的定义，也不是所有平台都对 C 和 C++ 使用相同的调用约定，并且 C++ 编译器在调用“make_foo”时将使用 C++ 调用约定，如果期望用 C 调用约定调用“make-foo”，则这可能导致分段错误。

The way to remedy this problem is to wrap almost all the declarations in the header in an `extern "C"` block.

> 解决这个问题的方法是将头中几乎所有的声明都封装在 `extern“C”` 块中。

```cpp
#ifdef __cplusplus
extern "C" {
#endif

typedef struct Foo {
    int bar;
} Foo;
Foo make_foo(int);

#ifdef __cplusplus
}  /* end of "extern C" block */
#endif

```

Now when `foo.h` is included from a C program, it will just appear as ordinary declarations, but when `foo.h` is included from a C++ program, `make_foo` will be inside an `extern "C"` block and the compiler will know to look for an unmangled name and use the C calling convention.

> 现在，当“foo.h”包含在 C 程序中时，它将只显示为普通声明，但当“foo.h”包含在 C++ 程序中时时，“make_foo”将位于“extern”C“”块内，编译器将知道如何查找未处理的名称并使用 C 调用约定。

#### Syntax

- extern **string-literal** { **declaration-seq**(**opt**) }
- extern **string-literal** **declaration**

#### Remarks

The standard requires all compilers to support `extern "C"` in order to allow C++ to be compatible with C, and `extern "C++"`, which may be used to override an enclosing linkage specification and restore the default. Other supported linkage specifications are [implementation-defined](http://stackoverflow.com/documentation/c%2B%2B/1363/implementation-defined-behavior).

> 该标准要求所有编译器都支持 `extern“C”`，以允许 C++ 与 C 兼容，并支持 `exten“C++”`，后者可用于覆盖封闭链接规范并恢复默认值。其他支持的链接规范是[定义的实现]([http://stackoverflow.com/documentation/c%2b%2b/1363/implementation](http://stackoverflow.com/documentation/c%2B%2B/1363/implementation)-定义的行为）。
