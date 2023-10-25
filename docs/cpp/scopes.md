---
tip: translate by baidu@2023-10-25 08:30:55
---
---
metaTitle: "C++ | Scopes"
description: "Global variables, Simple block scope"
---

# Scopes



## Global variables



To declare a single instance of a variable which is accessible in different source files, it is possible to make it in the global scope with keyword `extern`. This keyword says the compiler that somewhere in the code there is a definition for this variable, so it can be used everywhere and all write/read will be done in one place of memory.

> 要声明可在不同源文件中访问的变量的单个实例，可以使用关键字“extern”将其置于全局范围内。这个关键字告诉编译器，在代码的某个地方有这个变量的定义，所以它可以在任何地方使用，所有的写/读都将在内存的一个地方完成。

```cpp
// File my_globals.h:

#ifndef __MY_GLOBALS_H__
#define __MY_GLOBALS_H__

extern int circle_radius; // Promise to the compiler that circle_radius 
                          // will be defined somewhere

#endif

```

 

```cpp
// File foo1.cpp:

#include "my_globals.h"

int circle_radius = 123; // Defining the extern variable

```

 

```cpp
// File main.cpp:

#include "my_globals.h"
#include <iostream>

int main()
{
    std::cout << "The radius is: " << circle_radius << "\n";'
    return 0;
}

```

Output:

```cpp
The radius is: 123

```



## Simple block scope



The scope of a variable in a block `{ ... }`, begins after declaration and ends at the end of the block. If there is nested block, the inner block can hide the scope of a variable which is declared in the outer block.

> 块“｛…｝”中变量的作用域从声明之后开始，到块的末尾结束。如果存在嵌套块，则内部块可以隐藏在外部块中声明的变量的范围。

```cpp
{
    int x = 100;
    //   ^
    //   Scope of `x` begins here
    //
}   // <- Scope of `x` ends here

```


If a nested block starts within an outer block, a new declared variable with the same name which is before in the outer class, hides the first one.

> 如果嵌套块在外部块中开始，则一个新声明的变量将隐藏第一个变量，该变量的名称与外部类之前的名称相同。

```cpp
{
    int x = 100;

    {
        int x = 200;

        std::cout << x;  // <- Output is 200
    }

    std::cout << x;  // <- Output is 100
}

```

