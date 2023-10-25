---
tip: translate by baidu@2023-10-25 08:28:15
---
---

metaTitle: "C++ | Namespaces"
description: "What are namespaces?, Argument Dependent Lookup, Extending namespaces, Using directive, Making namespaces, Unnamed/anonymous namespaces, Compact nested namespaces, Inline namespace, Namespace alias, Aliasing a long namespace, Alias Declaration scope"
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# Namespaces

Used to prevent name collisions when using multiple libraries, a namespace is a declarative prefix for functions, classes, types, etc.

> 命名空间用于在使用多个库时防止名称冲突，它是函数、类、类型等的声明性前缀。

## What are namespaces?

A C++ namespace is a collection of C++ entities (functions, classes, variables), whose names are prefixed by the name of the namespace. When writing code within a namespace, named entities belonging to that namespace need not be prefixed with the namespace name, but entities outside of it must use the fully qualified name.  The fully qualified name has the format `<namespace>::<entity>`. Example:

> C++ 命名空间是 C++ 实体（函数、类、变量）的集合，其名称以命名空间的名称为前缀。在命名空间中编写代码时，属于该命名空间的命名实体不需要以命名空间名称为前缀，但命名空间之外的实体必须使用完全限定的名称。完全限定名称的格式为“＜namespace＞：：＜entity＞”。示例：

```cpp
namespace Example
{
  const int test = 5;

  const int test2 = test + 12; //Works within `Example` namespace
}

const int test3 = test + 3; //Fails; `test` not found outside of namespace.

const int test3 = Example::test + 3; //Works; fully qualified name used.

```

Namespaces are useful for grouping related definitions together. Take the analogy of a shopping mall.  Generally a shopping mall is split up into several stores, each store selling items from a specific category.  One store might sell electronics, while another store might sell shoes.  These logical separations in store types help the shoppers find the items they're looking for.  Namespaces help c++ programmers, like shoppers, find the functions, classes, and variables they're looking for by organizing them in a logical manner.  Example:

> 命名空间对于将相关定义分组在一起很有用。以购物中心为例。一般来说，一个购物中心分为几个商店，每个商店出售特定类别的商品。一家商店可能卖电子产品，而另一家商店则可能卖鞋子。商店类型中的这些逻辑分隔有助于购物者找到他们想要的商品。命名空间帮助 c++ 程序员像购物者一样，通过以逻辑方式组织它们来找到他们想要的函数、类和变量。示例：

```cpp
namespace Electronics
{
    int TotalStock;
    class Headphones
    {
        // Description of a Headphone (color, brand, model number, etc.)
    };
    class Television
    {
        // Description of a Television (color, brand, model number, etc.)
    };
}

namespace Shoes
{
    int TotalStock;
    class Sandal
    {
        // Description of a Sandal (color, brand, model number, etc.)
    };
    class Slipper
    {
        // Description of a Slipper (color, brand, model number, etc.)
    };
}

```

There is a single namespace predefined, which is the global namespace that has no name, but can be denoted by `::`. Example:

> 预定义了一个单独的命名空间，它是没有名称的全局命名空间，但可以用“：：”表示。示例：

```cpp
void bar() {
    // defined in global namespace
}
namespace foo {
    void bar() {
        // defined in namespace foo
    }
    void barbar() {
        bar();   // calls foo::bar()
        ::bar(); // calls bar() defined in global namespace
    }
}

```

## Argument Dependent Lookup

When calling a function without an explicit namespace qualifier, the compiler can choose to call a function within a namespace if one of the parameter types to that function is also in that namespace. This is called "Argument Dependent Lookup", or ADL:

> 当调用没有显式命名空间限定符的函数时，如果函数的参数类型之一也在命名空间中，编译器可以选择调用命名空间中的函数。这被称为“参数相关查找”或 ADL：

```cpp
namespace Test
{
  int call(int i);

  class SomeClass {...};

  int call_too(const SomeClass &data);
}

call(5); //Fails. Not a qualified function name.

Test::SomeClass data;

call_too(data); //Succeeds

```

`call` fails because none of its parameter types come from the `Test` namespace. `call_too` works because `SomeClass` is a member of `Test` and therefore it qualifies for ADL rules.

### When does ADL not occur

ADL does not occur if normal unqualified lookup finds a class member, a function that has been declared at block scope, or something that is not of function type. For example:

> 如果正常的非限定查找找到类成员、已在块范围内声明的函数或不是函数类型的东西，则不会发生 ADL。例如：

```cpp
void foo();
namespace N {
    struct X {};
    void foo(X ) { std::cout << '1'; }
    void qux(X ) { std::cout << '2'; }
}

struct C {
    void foo() {}
    void bar() {
        foo(N::X{}); // error: ADL is disabled and C::foo() takes no arguments
    }
};

void bar() {
    extern void foo(); // redeclares ::foo
    foo(N::X{});       // error: ADL is disabled and ::foo() doesn't take any arguments
}

int qux;

void baz() {
    qux(N::X{}); // error: variable declaration disables ADL for "qux"
}

```

## Extending namespaces

A useful feature of `namespace` s is that you can expand them (add members to it).

> “namespace”的一个有用功能是可以扩展它们（向其中添加成员）。

```cpp
namespace Foo
{
    void bar() {}
}

//some other stuff

namespace Foo
{
    void bar2() {}
}

```

## Using directive

The keyword ['using'](http://stackoverflow.com/documentation/c%2B%2B/4891/keywords/19096/using#t=20160909070552352037) has three flavors. Combined with keyword 'namespace' you write a 'using directive':

> 关键字['using'](http://stackoverflow.com/documentation/c%2B%2B/4891/keywords/19096/using#t=20160909070552352037)有三种口味。结合关键字“namespace”可以编写“using 指令”：

If you don't want to write `Foo::` in front of every stuff in the namespace `Foo`, you can use `using namespace Foo;` to import every single thing out of `Foo`.

> 如果你不想在名称空间“Foo”中的每一个东西前面写“Foo:：”，你可以使用“using namespace Foo；”从“Foo”中导入每一件东西。

```cpp
namespace Foo
{
    void bar() {}
    void baz() {}
}

//Have to use Foo::bar()
Foo::bar();

//Import Foo
using namespace Foo;
bar(); //OK
baz(); //OK

```

It is also possible to import selected entities in a namespace rather than the whole namespace:

> 也可以在命名空间中导入选定的实体，而不是导入整个命名空间：

```cpp
using Foo::bar;
bar(); //OK, was specifically imported
baz(); // Not OK, was not imported

```

A word of caution: `using namespace` s in header files is seen as bad style in most cases. If this is done, the namespace is imported in **every** file that includes the header. Since there is no way of "un-`using`" a namespace, this can lead to namespace pollution (more or unexpected symbols in the global namespace) or, worse, conflicts. See this example for an illustration of the problem:

> 注意：在大多数情况下，在头文件中使用名称空间被视为不好的风格。如果完成了此操作，则会在包含标头的每个**文件中导入名称空间。由于无法“取消使用”命名空间，这可能会导致命名空间污染（全局命名空间中出现更多或意外的符号），或者更糟的是，还会导致冲突。有关此问题的说明，请参见此示例：

```cpp
/***** foo.h *****/
namespace Foo
{
    class C;
}

/***** bar.h *****/
namespace Bar
{
    class C;
}

/***** baz.h *****/
#include "foo.h"
using namespace Foo;

/***** main.cpp *****/
#include "bar.h"
#include "baz.h"

using namespace Bar;
C c; // error: Ambiguity between Bar::C and Foo::C

```

A **using-directive** cannot occur at class scope.

## Making namespaces

Creating a namespace is really easy:

```cpp
//Creates namespace foo
namespace Foo
{
    //Declares function bar in namespace foo
    void bar() {}
}

```

To call `bar`, you have to specify the namespace first, followed by the scope resolution operator `::`:

> 要调用“bar”，必须首先指定命名空间，然后指定范围解析运算符“：：”：

```cpp
Foo::bar();

```

It is allowed to create one namespace in another, for example:

```cpp
namespace A
{
    namespace B
    {
        namespace C
        {
            void bar() {}
        }
    }
}

```

The above code could be simplified to the following:

```cpp
namespace A::B::C
{
    void bar() {}
}

```

## Unnamed/anonymous namespaces

An unnamed namespace can be used to ensure names have internal linkage (can only be referred to by the current translation unit). Such a namespace is defined in the same way as any other namespace, but without the name:

> 可以使用未命名的命名空间来确保名称具有内部链接（只能由当前翻译单元引用）。这样的名称空间的定义方式与任何其他名称空间相同，但没有名称：

```cpp
namespace {
    int foo = 42;
}

```

`foo` is only visible in the translation unit in which it appears.

It is recommended to never use unnamed namespaces in header files as this gives a version of the content for every translation unit it is included in. This is especially important if you define non-const globals.

> 建议永远不要在头文件中使用未命名的名称空间，因为这会为它所包含的每个翻译单元提供内容的版本。如果定义了非常量全局变量，这一点尤为重要。

```cpp
// foo.h
namespace {
    std::string globalString;
}

// 1.cpp
#include "foo.h" //< Generates unnamed_namespace{1.cpp}::globalString ...

globalString = "Initialize";

// 2.cpp
#include "foo.h" //< Generates unnamed_namespace{2.cpp}::globalString ...

std::cout << globalString; //< Will always print the empty string

```

## Compact nested namespaces

```cpp
namespace a {
  namespace b {
    template<class T>
    struct qualifies : std::false_type {};
  }
}

namespace other {
  struct bob {};
}

namespace a::b {
  template<>
  struct qualifies<::other::bob> : std::true_type {};
}

```

You can enter both the `a` and `b` namespaces in one step with `namespace a::b` starting in C++17.

> 您可以在一个步骤中输入“a”和“b”名称空间，“namespace a：：b”从 C++17 开始。

## Inline namespace

`inline namespace` includes the content of the inlined namespace in the enclosing namespace, so

```cpp
namespace Outer
{
    inline namespace Inner
    {
        void foo();
    }
}

```

is mostly equivalent to

```cpp
namespace Outer
{

    namespace Inner
    {
        void foo();
    }

    using Inner::foo;
}

```

but element from `Outer::Inner::` and those associated into `Outer::` are identical.

> 但是来自“Outer:：Inner:：”的元素和关联到“Outer::：”中的元素是相同的。

So following is equivalent

```cpp
Outer::foo();
Outer::Inner::foo();

```

The alternative `using namespace Inner;` would not be equivalent for some tricky parts as template specialization:

> 备选方案 `using namespace Inner；` 对于一些棘手的部分，将不等同于模板专业化：

For

```cpp
#include <outer.h> // See below

class MyCustomType;
namespace Outer
{
    template <>
    void foo<MyCustomType>() { std::cout << "Specialization"; }
}

```

<li>
The inline namespace allows the specialization of `Outer::foo`

```cpp
// outer.h
// include guard omitted for simplification

namespace Outer
{
    inline namespace Inner
    {
        template <typename T>
        void foo() { std::cout << "Generic"; }
    }
}

```

</li>
<li>

Whereas the `using namespace` doesn't allow the specialization of `Outer::foo`

> 而“using namespace”不允许专门化“Outer:：foo”`

```cpp
// outer.h
// include guard omitted for simplification

namespace Outer
{
    namespace Inner
    {
        template <typename T>
        void foo() { std::cout << "Generic"; }
    }
    using namespace Inner;
    // Specialization of `Outer::foo` is not possible
    // it should be `Outer::Inner::foo`.
}

```

</li>

Inline namespace is a way to allow several version to cohabit and defaulting to the `inline` one

> 内联名称空间是一种允许多个版本共存并默认为“内联”名称空间的方式

```cpp
namespace MyNamespace
{
    // Inline the last version
    inline namespace Version2
    {
        void foo(); // New version
        void bar();
    }

    namespace Version1 // The old one
    {
        void foo();
    }

}

```

And with usage

```cpp
MyNamespace::Version1::foo(); // old version
MyNamespace::Version2::foo(); // new version
MyNamespace::foo();           // default version : MyNamespace::Version1::foo();

```

## Namespace alias

A namespace can be given an alias (**i.e.,** another name for the same namespace) using the `namespace` **identifier** `=` syntax.  Members of the aliased namespace can be accessed by qualifying them with the name of the alias. In the following example, the nested namespace `AReallyLongName::AnotherReallyLongName` is inconvenient to type, so the function `qux` locally declares an alias `N`. Members of that namespace can then be accessed simply using `N::`.

> 可以使用 `namespace` **identifier** `=` 语法为名称空间赋予别名（**，即**同一名称空间的另一个名称）。别名命名空间的成员可以通过使用别名对其进行限定来访问。在以下示例中，嵌套的命名空间“AReallyLongName:：AnotherReallyLongName”不便于键入，因此函数“qux”在本地声明了一个别名“N”。然后，只需使用“N:：”即可访问该命名空间的成员。

```cpp
namespace AReallyLongName {
    namespace AnotherReallyLongName {
        int foo();
        int bar();
        void baz(int x, int y);
    }
}
void qux() {
    namespace N = AReallyLongName::AnotherReallyLongName;
    N::baz(N::foo(), N::bar());
}

```

## Aliasing a long namespace

This is usually used for renaming or shortening long namespace references such referring to components of a library.

> 这通常用于重命名或缩短长名称空间引用，例如引用库的组件。

```cpp
namespace boost
{
    namespace multiprecision
    {
        class Number ...
    }
}

namespace Name1 = boost::multiprecision;


//    Both Type declarations are equivalent
boost::multiprecision::Number X   //    Writing the full namespace path, longer
Name1::Number Y                   //    using the name alias, shorter

```

## Alias Declaration scope

Alias Declaration are affected by preceding **using** statements

```cpp
namespace boost
{
    namespace multiprecision
    {
        class Number ...
    }
}


using namespace boost;

//   Both Namespace are equivalent 
namespace Name1 = boost::multiprecision;
namespace Name2 = multiprecision;

```

However, it is easier to get confused over which namespace you are aliasing when you have something like this:

> 然而，当您有这样的东西时，更容易混淆您正在别名的命名空间：

```cpp
namespace boost
{
    namespace multiprecision
    {
        class Number ...
    }
}

namespace numeric
{
    namespace multiprecision
    {
        class Number ...
    }
}

using namespace numeric;
using namespace boost;

//    Not recommended as 
//    its not explicitly clear whether Name1 refers to
//    numeric::multiprecision or boost::multiprecision
namespace Name1 = multiprecision;

//    For clarity, its recommended to use absolute paths
//    instead
namespace Name2 = numeric::multiprecision;
namespace Name3 = boost::multiprecision;

```

#### Syntax

- namespace **identifier**(**opt**) { **declaration-seq** }
- inline namespace **identifier**(**opt**) { **declaration-seq** } /* since C++11 */
- inline(**opt**) namespace **attribute-specifier-seq** **identifier**(**opt**) { **declaration-seq** } /* since C++17 */

> -内联（**opt**）命名空间**属性说明符 seq***标识符**（**opt***）｛**声明 seq***｝/*自 C++17 以来*/

- namespace **enclosing-namespace-specifier** :: **identifier** { **declaration-seq** } /* since C++17 */

> -命名空间**包含命名空间说明符**：：**标识符**｛**声明 seq***｝/*自 C++17 以来*/

- namespace **identifier** = **qualified-namespace-specifier**;
- using namespace **nested-name-specifier**(**opt**) **namespace-name**;
- **attribute-specifier-seq** using namespace **nested-name-specifier**(**opt**) **namespace-name**; /* since C++11 */

> -**属性说明符 seq** 使用命名空间**嵌套名称说明符**（**opt**）**命名空间名称**；/*自 C++11*/

#### Remarks

The [keyword](http://stackoverflow.com/documentation/c%2B%2B/4891/keywords) `namespace` has three different meanings depending on context:

> [关键字](http://stackoverflow.com/documentation/c%2B%2B/4891/keywords)“namespace”根据上下文有三种不同的含义：

<li>

When followed by an optional name and a brace-enclosed sequence of declarations, it [defines a new namespace](http://stackoverflow.com/documentation/c%2B%2B/495/namespaces) or [extends an existing namespace](http://stackoverflow.com/documentation/c%2B%2B/495/namespaces/1623/extending-namespaces) with those declarations. If the name is omitted, the namespace is an [unnamed namespace](http://stackoverflow.com/documentation/c%2B%2B/495/namespaces/4851/unnamed-anonymous-namespaces).

> 当后面跟着一个可选名称和一个大括号括起来的声明序列时，它[定义了一个新的命名空间](http://stackoverflow.com/documentation/c%2B%2B/495/namespaces)或[扩展现有命名空间]([http://stackoverflow.com/documentation/c%2b%2b/495/namespaces/1623/extending](http://stackoverflow.com/documentation/c%2B%2B/495/namespaces/1623/extending)-命名空间）。如果省略名称，则该名称空间为[未命名名称空间]([http://stackoverflow.com/documentation/c%2b%2b/495/namespaces/4851/unnamed](http://stackoverflow.com/documentation/c%2B%2B/495/namespaces/4851/unnamed)-匿名命名空间）。

</li>
<li>

When followed by a name and an equal sign, it declares a [namespace alias](http://stackoverflow.com/documentation/c%2B%2B/495/namespaces/28691/namespace-alias).

> 当后面跟一个名称和一个等号时，它声明一个[名称空间别名]([http://stackoverflow.com/documentation/c%2b%2b/495/namespaces/28691/namespace](http://stackoverflow.com/documentation/c%2B%2B/495/namespaces/28691/namespace)-别名）。

</li>
<li>

When preceded by `using` and followed by a namespace name, it forms a [**using directive**](http://stackoverflow.com/documentation/c%2B%2B/495/namespaces/1624/using-namespaces#t=201608060458446751916), which allows names in the given namespace to be found by unqualified name lookup (but does not redeclare those names in the current scope). A **using-directive** cannot occur at class scope.

> 当前面是“using”，后面是命名空间名称时，它形成[**using 指令**]([http://stackoverflow.com/documentation/c%2b%2b/495/namespaces/1624/using-namespaces#t=201608060458446751916](http://stackoverflow.com/documentation/c%2B%2B/495/namespaces/1624/using-namespaces#t=201608060458446751916)），这允许通过非限定名称查找来查找给定命名空间中的名称（但不在当前作用域中重新声明这些名称）。使用指令**的**不能出现在类作用域中。

</li>

`using namespace std;` is discouraged. Why? Because `namespace std` is huge! This means that there is a high chance that names will collide:

```cpp
//Really bad!
using namespace std;

//Calculates p^e and outputs it to std::cout
void pow(double p, double e) { /*...*/ }

//Calls pow
pow(5.0, 2.0); //Error! There is already a pow function in namespace std with the same signature,
               //so the call is ambiguous

```
