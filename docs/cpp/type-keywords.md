---
tip: translate by baidu@2023-10-25 08:35:17
---
---

metaTitle: "C++ | Type Keywords"
description: "class, enum, struct, union"
-----------------------------------------

# Type Keywords

## class

<li>

Introduces the definition of a [class](http://stackoverflow.com/documentation/c%2B%2B/508/classes-structures) type.

> 介绍[类]的定义([http://stackoverflow.com/documentation/c%2b%2b/508/classes](http://stackoverflow.com/documentation/c%2B%2B/508/classes)-结构）类型。

```cpp
class foo {
    int x;
  public:
    int get_x();
    void set_x(int new_x);
};

```

</li>
<li>

Introduces an **elaborated type specifier,** which specifies that the following name is the name of a class type. If the class name has been declared already, it can be found even if hidden by another name. If the class name has not been declared already, it is forward-declared.

> 引入一个**详细说明的类型说明符，**指定以下名称为类类型的名称。如果类名已经声明，那么即使被另一个名称隐藏，也可以找到它。如果尚未声明类名，则向前声明该类名。

```cpp
class foo; // elaborated type specifier -> forward declaration
class bar {
  public:
    bar(foo& f);
};
void baz();
class baz; // another elaborated type specifer; another forward declaration
           // note: the class has the same name as the function void baz()
class foo {
    bar b;
    friend class baz; // elaborated type specifier refers to the class,
                      // not the function of the same name
  public:
    foo();
};

```

</li>
<li>

Introduces a type parameter in the declaration of a [template](http://stackoverflow.com/documentation/c%2B%2B/460/templates).

> 在[template]的声明中引入类型参数([http://stackoverflow.com/documentation/c%2b%2b/460/templates](http://stackoverflow.com/documentation/c%2B%2B/460/templates))。

```cpp
template <class T>
const T& min(const T& x, const T& y) {
    return b < a ? b : a;
}

```

</li>
<li>

In the declaration of a [template template parameter](http://stackoverflow.com/documentation/c%2B%2B/460/templates/10838/template-template-parameters), the keyword `class` precedes the name of the parameter. Since the argument for a template template parameter can only be a class template, the use of `class` here is redundant. However, the grammar of C++ requires it.

> 在[template-template 参数]的声明中([http://stackoverflow.com/documentation/c%2b%2b/460/templates/10838/template](http://stackoverflow.com/documentation/c%2B%2B/460/templates/10838/template)-模板参数），关键字“class”位于参数名称之前。由于模板模板参数的参数只能是类模板，因此此处使用“class”是多余的。然而，C++ 的语法需要它。

```cpp
template <template <class T> class U>
//                           ^^^^^ "class" used in this sense here;
//                                 U is a template template parameter
void f() {
    U<int>::do_it();
    U<double>::do_it();
}

```

</li>
<li>

Note that sense 2 and sense 3 may be combined in the same declaration. For example:

> 注意，感测 2 和感测 3 可以组合在同一声明中。例如：

```cpp
template <class T>
class foo {
};

foo<class bar> x; // <- bar does not have to have previously appeared.

```

</li>

<li>

In the declaration or definition of an enum, declares the enum to be a [scoped enum](http://stackoverflow.com/documentation/c%2B%2B/2796/enumeration/13318/scoped-enums#t=201608050832023792595).

> 在枚举的声明或定义中，将枚举声明为[scoped enum]([http://stackoverflow.com/documentation/c%2b%2b/2796/enumeration/13318/scoped-enums#t=201608050832023792595](http://stackoverflow.com/documentation/c%2B%2B/2796/enumeration/13318/scoped-enums#t=201608050832023792595)）。

```cpp
enum class Format {
    TEXT,
    PDF,
    OTHER,
};
Format f = F::TEXT;

```

</li>

## enum

<li>

Introduces the definition of an [enumeration type](http://stackoverflow.com/documentation/c%2B%2B/2796/enumeration).

> 介绍[枚举类型]的定义([http://stackoverflow.com/documentation/c%2b%2b/2796/enumeration](http://stackoverflow.com/documentation/c%2B%2B/2796/enumeration))。

```cpp
enum Direction {
    UP,
    LEFT,
    DOWN,
    RIGHT
};
Direction d = UP;

```

</li>

In C++11, `enum` may optionally be followed by `class` or `struct` to define a [scoped enum](http://stackoverflow.com/documentation/c%2B%2B/2796/enumeration/13318/scoped-enums). Furthermore, both scoped and unscoped enums can have their underlying type explicitly specified by `: T` following the enum name, where `T` refers to an integer type.

> 在 C++11 中，“enum”后面可以有选择地跟“class”或“struct”，以定义[作用域枚举]([http://stackoverflow.com/documentation/c%2b%2b/2796/enumeration/13318/scoped-enums](http://stackoverflow.com/documentation/c%2B%2B/2796/enumeration/13318/scoped-enums)）。此外，作用域枚举和非作用域枚举都可以通过枚举名称后面的“：T”显式指定其基础类型，其中“T”表示整数类型。

```

  enum class Format : char {
       TEXT,
       PDF,
       OTHER
   };
   Format f = Format::TEXT;

   enum Language : int {
       ENGLISH,
       FRENCH,
       OTHER
   };

```

Enumerators in normal `enum` s may also be preceded by the scope operator, although they are still considered to be in the scope the `enum` was defined in.

> 普通“enum”中的枚举器前面也可能有 scope 运算符，尽管它们仍然被认为在定义“enum“的作用域中。

```

  Language l1, l2;

   l1 = ENGLISH;
   l2 = Language::OTHER;

```

<li>

Introduces an **elaborated type specifier,** which specifies that the following name is the name of a previously declared enum type. (An elaborated type specifier cannot be used to forward-declare an enum type.) An enum can be named in this way even if hidden by another name.

> 引入一个**详细说明的类型说明符，**指定以下名称是以前声明的枚举类型的名称。（详细说明的类型说明符不能用于前向声明枚举类型。）即使枚举被其他名称隐藏，也可以用这种方式命名。

```cpp
enum Foo { FOO };
void Foo() {}
Foo foo = FOO;      // ill-formed; Foo refers to the function
enum Foo foo = FOO; // ok; Foo refers to the enum type

```

</li>

<li>

Introduces an **opaque enum declaration,** which declares an enum without defining it. It can either redeclare a previously declared enum, or forward-declare an enum that has not been previously declared.

> 引入**不透明的枚举声明，**声明枚举而不定义它。它可以重新声明以前声明的枚举，也可以正向声明以前未声明的枚举。

An enum first declared as scoped cannot later be declared as unscoped, or **vice versa.** All declarations of an enum must agree in underlying type.

> 首先声明为 scoped 的枚举以后不能声明为 unscoped，**反之亦然。**枚举的所有声明在基础类型中必须一致。

When forward-declaring an unscoped enum, the underlying type must be explicitly specified, since it cannot be inferred until the values of the enumerators are known.

> 当正向声明未受范围限制的枚举时，必须显式指定基础类型，因为在知道枚举器的值之前无法推断它。

```cpp
enum class Format; // underlying type is implicitly int
void f(Format f);
enum class Format {
    TEXT,
    PDF,
    OTHER,
};

enum Direction;    // ill-formed; must specify underlying type

```

</li>

## struct

Interchangeable with [`class`](http://stackoverflow.com/documentation/c%2B%2B/4891/keywords/18504/class), except for the following differences:

> 可与[`class`]互换([http://stackoverflow.com/documentation/c%2b%2b/4891/keywords/18504/class](http://stackoverflow.com/documentation/c%2B%2B/4891/keywords/18504/class))，除了以下差异：

- If a class type is defined using the keyword `struct`, then the default accessibility of bases and members is `public` rather than `private`.

> -如果使用关键字“struct”定义类类型，则基和成员的默认可访问性是“public”而不是“private”。

- `struct` cannot be used to declare a template type parameter or template template parameter; only `class` can.

> -“struct”不能用于声明模板类型参数或模板模板参数；只有“class”可以。

## union

<li>

Introduces the definition of a [union](http://stackoverflow.com/documentation/c%2B%2B/2678/unions) type.

> 介绍[并集]的定义([http://stackoverflow.com/documentation/c%2b%2b/2678/unions](http://stackoverflow.com/documentation/c%2B%2B/2678/unions))类型。

```cpp
// Example is from POSIX
union sigval {
    int     sival_int;
    void   *sival_ptr;
};

```

</li>
<li>

Introduces an **elaborated type specifier,** which specifies that the following name is the name of a union type. If the union name has been declared already, it can be found even if hidden by another name. If the union name has not been declared already, it is forward-declared.

> 引入一个**详细说明的类型说明符，**指定以下名称为并集类型的名称。如果联合名称已经声明，即使被另一个名称隐藏，也可以找到它。如果尚未声明联合名称，则向前声明。

```cpp
union foo; // elaborated type specifier -> forward declaration
class bar {
  public:
    bar(foo& f);
};
void baz();
union baz; // another elaborated type specifer; another forward declaration
           // note: the class has the same name as the function void baz()
union foo {
    long l;
    union baz* b; // elaborated type specifier refers to the class,
                  // not the function of the same name
};

```

</li>
