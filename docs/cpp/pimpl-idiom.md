---
tip: translate by baidu@2023-10-25 08:29:26
---
---
metaTitle: "C++ | Pimpl Idiom"
description: "Basic Pimpl idiom"
---

# Pimpl Idiom



## Basic Pimpl idiom


In the header file:

```cpp
// widget.h

#include <memory>  // std::unique_ptr
#include <experimental/propagate_const>

class Widget
{
    public:
        Widget();
        ~Widget();
        void DoSomething();

    private:
        // the pImpl idiom is named after the typical variable name used
        // ie, pImpl:
        struct Impl;                    // forward declaration
        std::experimental::propagate_const<std::unique_ptr< Impl >> pImpl;  // ptr to actual implementation
};

```

In the implementation file:

```cpp
// widget.cpp

#include "widget.h"
#include "reallycomplextype.h" // no need to include this header inside widget.h

struct Widget::Impl
{
    // the attributes needed from Widget go here
    ReallyComplexType rct;
};

Widget::Widget() :
    pImpl(std::make_unique<Impl>())
{}

Widget::~Widget() = default;

void Widget::DoSomething()
{
    // do the stuff here with pImpl
}

```


The `pImpl` contains the `Widget` state (or some/most of it).  Instead of the `Widget` description of state being exposed in the header file, it can be only exposed within the implementation.

> “pImpl”包含“Widget”状态（或部分/大部分）。与其在头文件中公开状态的“Widget”描述，它只能在实现中公开。

`pImpl` stands for "pointer to implementation".  The "real" implementation of `Widget` is in the `pImpl`.


Danger: Note that for this to work with `unique_ptr`, `~Widget()` must be implemented at a point in a file where the `Impl` is fully visible.  You can `=default` it there, but if `=default` where `Impl` is undefined, the program may easily become ill-formed, no diagnostic required.

> 危险：请注意，要使用“unique_ptr”，必须在文件中“Impl”完全可见的位置实现“~Widget（）”。你可以在那里“=默认”它，但如果“=默认“，其中“Impl”未定义，程序可能很容易变得格式错误，不需要诊断。



#### Remarks



The **pimpl idiom** (**p**ointer to **impl**ementation, sometimes referred to as  **opaque pointer** or **cheshire cat technique**), reduces the compilation times of a class by moving all its private data members into a struct defined in the .cpp file.

> **pimpl习惯用法**（**p**pointer to**impl**注释，有时被称为**不透明指针**或**cheshire cat技术**）通过将类的所有私有数据成员移动到.cpp文件中定义的结构中来减少类的编译时间。


The class owns a pointer to the implementation. This way, it can be forward declared, so that the header file does not need to `#include` classes that are used in private member variables.

> 该类拥有指向实现的指针。通过这种方式，它可以被前向声明，这样头文件就不需要“#include”在私有成员变量中使用的类。


When using the pimpl idiom, changing a private data member does not require recompiling classes that depend on it.

> 当使用pimpl习惯用法时，更改私有数据成员不需要重新编译依赖它的类。

