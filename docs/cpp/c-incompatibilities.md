---
tip: translate by baidu@2023-10-25 08:21:50
---
---
metaTitle: "C++ | C incompatibilities"
description: "Reserved Keywords, Weakly typed pointers, goto or switch"
---

# C incompatibilities


This describes what C code will break in a C++ compiler.



## Reserved Keywords



The first example are keywords that have a special purpose in C++: the following is legal in C, but not C++.

> 第一个例子是在C++中有特殊用途的关键字：以下在C中是合法的，但在C++中不是。

```cpp
int class = 5

```

These errors are easy to fix: just rename the variable.



## Weakly typed pointers



In C, pointers can be cast to a `void*`, which needs an explicit cast in C++. The following is illegal in C++, but legal in C:

> 在C中，指针可以强制转换为“void*”，这需要在C++中进行显式强制转换。以下内容在C++中是非法的，但在C中是合法的：

```cpp
void* ptr;
int* intptr = ptr;

```

Adding an explicit cast makes this work, but can cause further issues.



## goto or switch



In C++, you may not skip initializations with `goto` or `switch`. The following is valid in C, but not C++:

> 在C++中，不能跳过带有“goto”或“switch”的初始化。以下内容在C中有效，但在C++中无效：

```cpp
goto foo;
int skipped = 1;
foo;

```

These bugs may require redesign.

