---
tip: translate by baidu@2023-10-25 08:28:32
---
---
metaTitle: "C++ | One Definition Rule (ODR)"
description: "Multiply defined function, Inline functions, ODR violation via overload resolution"
---

# One Definition Rule (ODR)



## Multiply defined function



The most important consequence of the One Definition Rule is that non-inline functions with external linkage should only be defined once in a program, although they can be declared multiple times. Therefore, such functions should not be defined in headers, since a header can be included multiple times from different translation units.

> 一个定义规则最重要的结果是，具有外部链接的非内联函数在程序中只能定义一次，尽管它们可以声明多次。因此，不应在标头中定义此类函数，因为标头可以从不同的翻译单元多次包含。

`foo.h`:

```cpp
#ifndef FOO_H
#define FOO_H
#include <iostream>
void foo() { std::cout << "foo"; }
void bar();
#endif

```

`foo.cpp`:

```cpp
#include "foo.h"
void bar() { std:: cout << "bar"; }

```

`main.cpp`:

```cpp
#include "foo.h"
int main() {
    foo();
    bar();
}

```


In this program, the function `foo` is defined in the header `foo.h`, which is included twice: once from `foo.cpp` and once from `main.cpp`. Each translation unit therefore contains its own definition of `foo`. Note that the include guards in `foo.h` do not prevent this from happening, since `foo.cpp` and `main.cpp` both **separately** include `foo.h`. The most likely result of trying to build this program is a link-time error identifying `foo` as having been multiply defined.

> 在这个程序中，函数“foo”是在头“foo.h”中定义的，它包含两次：一次来自“foo.cpp”，一次来自于“main.cpp”。因此，每个翻译单元都包含自己的“foo”定义。请注意，“foo.h”中的include保护并不能阻止这种情况的发生，因为“foo.cpp”和“main.cpp”都**分别**包含“foo.h”。尝试构建此程序的最可能结果是链接时错误，将“foo”标识为已被多重定义。


To avoid such errors, one should **declare** functions in headers and **define** them in the corresponding `.cpp` files, with some exceptions (see other examples).

> 为了避免此类错误，应在标头中**声明**函数，并在相应的“.cpp”文件中**定义**函数，但有一些例外（请参阅其他示例）。



## Inline functions



A function declared `inline` may be defined in multiple translation units, provided that all definitions are identical. It also must be defined in every translation unit in which it is used. Therefore, inline functions **should** be defined in headers and there is no need to mention them in the implementation file.

> 声明为“inline”的函数可以在多个翻译单元中定义，前提是所有定义都相同。它还必须在使用它的每个翻译单元中进行定义。因此，内联函数**应该在头中定义，而无需在实现文件中提及它们。


The program will behave as though there is a single definition of the function.

> 该程序将表现为函数只有一个定义。

`foo.h`:

```cpp
#ifndef FOO_H
#define FOO_H
#include <iostream>
inline void foo() { std::cout << "foo"; }
void bar();
#endif

```

`foo.cpp`:

```cpp
#include "foo.h"
void bar() {
    // more complicated definition
}

```

`main.cpp`:

```cpp
#include "foo.h"
int main() {
    foo();
    bar();
}

```


In this example, the simpler function `foo` is defined inline in the header file while the more complicated function `bar` is not inline and is defined in the implementation file. Both the `foo.cpp` and `main.cpp` translation units contain definitions of `foo`, but this program is well-formed since `foo` is inline.

> 在这个例子中，更简单的函数“foo”是在头文件中内联定义的，而更复杂的函数“bar”不是内联的，而是在实现文件中定义的。“foo.cpp”和“main.cpp”翻译单元都包含“foo”的定义，但这个程序的格式很好，因为“foo’是内联的。


A function defined within a class definition (which may be a member function or a friend function) is **implicitly** inline. Therefore, if a class is defined in a header, member functions of the class may be defined within the class definition, even though the definitions may be included in multiple translation units:

> 类定义中定义的函数（可以是成员函数或友元函数）是**隐式**内联的。因此，如果在头中定义了一个类，则该类的成员函数可以在类定义中定义，即使这些定义可以包括在多个翻译单元中：

```cpp
// in foo.h
class Foo {
    void bar() { std::cout << "bar"; }
    void baz();
};

// in foo.cpp
void Foo::baz() {
   // definition
}

```


The function `Foo::baz` is defined out-of-line, so it is **not** an inline function, and must not be defined in the header.

> 函数“Foo:：baz”是越界定义的，因此它不是**内联函数，并且不能在标头中定义。



## ODR violation via overload resolution



Even with identical tokens for inline functions, ODR can be violated if lookup of names doesn't refer to the same entity. let's consider `func` in following:

> 即使对于内联函数使用相同的令牌，如果名称查找不引用同一实体，也可能违反ODR。让我们考虑下面的“func”：

<li>
header.h

```cpp
void overloaded(int);
inline void func() { overloaded('*'); }

```


</li>
<li>
foo.cpp

```cpp
#include "header.h"

void foo()
{
    func(); // `overloaded` refers to `void overloaded(int)`
}

```


</li>
<li>
bar.cpp

```cpp
void overloaded(char); // can come from other include
#include "header.h"

void bar()
{
    func(); // `overloaded` refers to `void overloaded(char)`
}

```


</li>


We have an ODR violation as `overloaded` refers to different entities depending of the translation unit.

> 我们有一个ODR违规，因为“重载”指的是不同的实体，具体取决于翻译单元。

