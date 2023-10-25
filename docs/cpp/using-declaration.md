---
tip: translate by baidu@2023-10-25 08:36:14
---
---

metaTitle: "C++ | Using declaration"
description: "Importing names individually from a namespace, Redeclaring members from a base class to avoid name hiding, Inheriting constructors"
-------------------------------------------------------------------------------------------------------------------------------------------------

# Using declaration

A `using` declaration introduces a single name into the current scope that was previously declared elsewhere.

> “using”声明将以前在其他地方声明的单个名称引入当前作用域。

## Importing names individually from a namespace

Once `using` is used to introduce the name `cout` from the namespace `std` into the scope of the `main` function, the `std::cout` object can be referred to as `cout` alone.

> 一旦使用“using”将名称空间“std”中的名称“cout”引入“main”函数的范围，“std:：cout”对象就可以单独称为“cout’”。

```cpp
#include <iostream>
int main() {
    using std::cout;
    cout << "Hello, world!\n";
}

```

## Redeclaring members from a base class to avoid name hiding

If a **using-declaration** occurs at class scope, it is only allowed to redeclare a member of a base class. For example, `using std::cout` is not allowed at class scope.

> 如果在类作用域中出现使用声明**，则只允许重新声明基类的成员。例如，在类作用域中不允许使用“using std:：cout”。

Often, the name redeclared is one that would otherwise be hidden. For example, in the below code, `d1.foo` only refers to `Derived1::foo(const char*)` and a compilation error will occur. The function `Base::foo(int)` is hidden not considered at all. However, `d2.foo(42)` is fine because the **using-declaration** brings `Base::foo(int)` into the set of entities named `foo` in `Derived2`. Name lookup then finds both `foo` s and overload resolution selects `Base::foo`.

> 通常，重新声明的名称会被隐藏。例如，在下面的代码中，“d1.foo”仅引用“Derived1:：foo（const-char*）”，并且将发生编译错误。函数“Base:：foo（int）”是隐藏的，根本不考虑。但是，“d2.foo（42）”是好的，因为**使用声明**将“Base:：foo（int）”带入“Derived2”中名为“foo”的实体集中。然后，名称查找查找“foo”，重载解析选择“Base:：foo”。

```cpp
struct Base {
    void foo(int);
};
struct Derived1 : Base {
    void foo(const char*);
};
struct Derived2 : Base {
    using Base::foo;
    void foo(const char*);
};
int main() {
    Derived1 d1;
    d1.foo(42);  // error
    Derived2 d2;
    d2.foo(42);  // OK
}

```

## Inheriting constructors

As a special case, a **using-declaration** at class scope can refer to the constructors of a direct base class. Those constructors are then **inherited** by the derived class and can be used to initialize the derived class.

> 作为一种特殊情况，在类范围内使用声明**的**可以引用直接基类的构造函数。然后，这些构造函数由派生类**继承**，并可用于初始化派生类。

```cpp
struct Base {
    Base(int x, const char* s);
};
struct Derived1 : Base {
    Derived1(int x, const char* s) : Base(x, s) {}
};
struct Derived2 : Base {
    using Base::Base;
};
int main() {
    Derived1 d1(42, "Hello, world");
    Derived2 d2(42, "Hello, world");
}

```

In the above code, both `Derived1` and `Derived2` have constructors that forward the arguments directly to the corresponding constructor of `Base`. `Derived1` performs the forwarding explicitly, while `Derived2`, using the C++11 feature of inheriting constructors, does so implicitly.

> 在上面的代码中，“Derived1”和“Derived2”都有构造函数，它们将参数直接转发给“Base”的相应构造函数 `Derived1'显式执行转发，而使用继承构造函数的 C++11 特性的 Derived2'则隐式执行转发。

#### Syntax

- using typename(**opt**) **nested-name-specifier** **unqualified-id**;
- using :: **unqualified-id**;

#### Remarks

A **using-declaration** is distinct from a [using directive](http://stackoverflow.com/documentation/c%2B%2B/495/namespaces/1624/using-directive), which tells the compiler to look in a particular namespace when looking up **any** name. A **using-directive** begins with `using namespace`.

> 使用声明**与[using 指令]不同([http://stackoverflow.com/documentation/c%2b%2b/495/namespaces/1624/using](http://stackoverflow.com/documentation/c%2B%2B/495/namespaces/1624/using)-指令），它告诉编译器在查找**任何**名称时查找特定的名称空间。一个** using 指令**以“using namespace”开头。

A **using-declaration** is also distinct from an alias declaration, which gives a new name to an existing type in the same manner as [`typedef`](http://stackoverflow.com/documentation/c%2B%2B/7840/variable-declaration-keywords/18785/typedef). An alias declaration contains an equals sign.

> 使用**的声明**也不同于别名声明，后者以与[`typedef`]相同的方式为现有类型赋予新名称([http://stackoverflow.com/documentation/c%2b%2b/7840/variable](http://stackoverflow.com/documentation/c%2B%2B/7840/variable)-声明关键字/18785/typedef）。别名声明包含等号。
