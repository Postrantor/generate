---
tip: translate by baidu@2023-10-25 08:35:29
---
---
metaTitle: "C++ | Typedef and type aliases"
description: "Basic typedef syntax, More complex uses of typedef, Declaring multiple types with typedef, Alias declaration with using"
---

# Typedef and type aliases



The `typedef` and (since C++11) `using` [keywords](http://stackoverflow.com/documentation/c%2b%2b/4891/keywords) can be used to give a new name to an existing type.

> “typedef”和（自C++11以来）“using”[keywords](http://stackoverflow.com/documentation/c%2b%2b/4891/keywords)可以用于为现有类型赋予新名称。



## Basic typedef syntax



A `typedef` declaration has the same syntax as a variable or function declaration, but it contains the word `typedef`. The presence of `typedef` causes the declaration to declare a type instead of a variable or function.

> “typedef”声明与变量或函数声明具有相同的语法，但它包含单词“typedef”。“typedef”的存在会导致声明声明类型，而不是变量或函数。

```cpp
int T;         // T has type int
typedef int T; // T is an alias for int

int A[100];         // A has type "array of 100 ints"
typedef int A[100]; // A is an alias for the type "array of 100 ints"

```


Once a type alias has been defined, it can be used interchangeably with the original name of the type.

> 一旦定义了类型别名，它就可以与类型的原始名称互换使用。

```cpp
typedef int A[100];
// S is a struct containing an array of 100 ints
struct S {
    A data;
};

```

`typedef` never creates a distinct type. It only gives another way of referring to an existing type.

```cpp
struct S {
    int f(int);
};
typedef int I;
// ok: defines int S::f(int)
I S::f(I x) { return x; }

```



## More complex uses of typedef



The rule that `typedef` declarations have the same syntax as ordinary variable and function declarations can be used to read and write more complex declarations.

> “typedef”声明与普通变量和函数声明具有相同语法的规则可用于读取和写入更复杂的声明。

```cpp
void (*f)(int);         // f has type "pointer to function of int returning void"
typedef void (*f)(int); // f is an alias for "pointer to function of int returning void"

```


This is especially useful for constructs with confusing syntax, such as pointers to non-static members.

> 这对于语法混乱的构造尤其有用，例如指向非静态成员的指针。

```cpp
void (Foo::*pmf)(int);         // pmf has type "pointer to member function of Foo taking int
                               // and returning void"
typedef void (Foo::*pmf)(int); // pmf is an alias for "pointer to member function of Foo
                               // taking int and returning void"

```


It is hard to remember the syntax of the following function declarations, even for experienced programmers:

> 即使对于经验丰富的程序员来说，也很难记住以下函数声明的语法：

```cpp
void (Foo::*Foo::f(const char*))(int);
int (&g())[100];

```

`typedef` can be used to make them easier to read and write:

```cpp
typedef void (Foo::pmf)(int);  // pmf is a pointer to member function type
pmf Foo::f(const char*);       // f is a member function of Foo

typedef int (&ra)[100];        // ra means "reference to array of 100 ints"
ra g();                        // g returns reference to array of 100 ints

```



## Declaring multiple types with typedef



The `typedef` keyword is a specifier, so it applies separately to each declarator. Therefore, each name declared refers to the type that that name would have in the absence of `typedef`.

> “typedef”关键字是一个说明符，因此它分别应用于每个声明符。因此，声明的每个名称都引用了该名称在没有“typedef”的情况下所具有的类型。

```cpp
int *x, (*p)();         // x has type int*, and p has type int(*)()
typedef int *x, (*p)(); // x is an alias for int*, while p is an alias for int(*)()

```



## Alias declaration with "using"



The syntax of `using` is very simple: the name to be defined goes on the left hand side, and the definition goes on the right hand side. No need to scan to see where the name is.

> “using”的语法非常简单：要定义的名称在左手边，定义在右手边。无需扫描以查看名称的位置。

```cpp
using I = int;
using A = int[100];             // array of 100 ints
using FP = void(*)(int);        // pointer to function of int returning void
using MP = void (Foo::*)(int);  // pointer to member function of Foo of int returning void

```


Creating a type alias with `using` has exactly the same effect as creating a type alias with `typedef`. It is simply an alternative syntax for accomplishing the same thing.

> 使用“using”创建类型别名与使用“typedef”创建类型化名的效果完全相同。它只是完成相同任务的一种替代语法。


Unlike `typedef`, `using` can be templated. A "template typedef" created with `using` is called an [alias template](http://stackoverflow.com/documentation/c%2b%2b/460/templates/10144/alias-template).

> 与“typedef”不同，“using”可以是模板化的。使用“using”创建的“template typedef”称为[别名模板](http://stackoverflow.com/documentation/c%2b%2b/460/templates/10144/alias-模板）。



#### Syntax


- typedef **type-specifier-seq** **init-declarator-list**;

- **attribute-specifier-seq** typedef **decl-specifier-seq** **init-declarator-list**; // since C++11

> -**属性说明符seq**typedef**decl说明符seq***init声明符列表**；//自C++11
- using **identifier** **attribute-specifier-seq**(**opt**) = **type-id**; // since C++11

