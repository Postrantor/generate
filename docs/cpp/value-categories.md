---
tip: translate by baidu@2023-10-25 08:36:20
---
---

metaTitle: "C++ | Value Categories"
description: "Value Category Meanings, rvalue, xvalue, prvalue, lvalue, glvalue"
--------------------------------------------------------------------------------

# Value Categories

## Value Category Meanings

Expressions in C++ are assigned a particular value category, based on the result of those expressions. Value categories for expressions can affect C++ function overload resolution.

> C++ 中的表达式会根据这些表达式的结果指定一个特定的值类别。表达式的值类别可能会影响 C++ 函数重载的分辨率。

Value categories determines two important-but-separate properties about an expression. One property is whether the expression has identity. An expression has identity if it refers to an object that has a variable name. The variable name may not be involved in the expression, but the object can still have one.

> 值类别确定表达式的两个重要但独立的属性。一个属性是表达式是否具有标识。如果表达式引用的对象具有变量名，则该表达式具有标识。变量名可能不包含在表达式中，但对象仍然可以包含一个变量名。

The other property is whether it is legal to implicitly move from the expression's value. Or more specifically, whether the expression, when used as a function parameter, will bind to r-value parameter types or not.

> 另一个属性是隐含地偏离表达式的值是否合法。或者更具体地说，当表达式用作函数参数时，它是否会绑定到 r 值参数类型。

C++ defines 3 value categories which represent the useful combination of these properties: lvalue (expressions with identity but not movable from), xvalue (expressions with identity that are moveable from), and prvalue (expressions without identity that are moveable from). C++ does not have expressions which have no identity and cannot be moved from.

> C++ 定义了 3 个值类别，它们表示这些属性的有用组合：lvalue（具有标识但不能从中移动的表达式）、xvalue（具有标识的可从中移动表达式）和 prvalue（没有标识的可移动表达式）。C++ 没有没有没有标识且无法从中移动的表达式。

C++ defines two other value categories, each based solely on one of these properties: glvalue (expressions with identity) and rvalue (expressions that can be moved from). These act as useful groupings of the prior categories.

> C++ 定义了另外两个值类别，每个类别都只基于其中一个属性：glvalue（具有标识的表达式）和 rvalue（可以从中移动的表达式）。这些作为先前类别的有用分组。

This graph serves as an illustration:

[<img src="http://i.stack.imgur.com/C09fH.png" alt="Graph of value categories in the C++ language" />](http://i.stack.imgur.com/C09fH.png)

> [<img src=“http://i.stack.imgur.com/C09fH.png“alt=”C++ 语言中的值类别图“/>](http://i.stack.imgur.com/C09fH.png)

## rvalue

An rvalue expression is any expression which can be implicitly moved from, regardless of whether it has identity.

> 右值表达式是任何可以隐式移动的表达式，无论它是否具有同一性。

More precisely, rvalue expressions may be used as the argument to a function that takes a parameter of type `T &&` (where `T` is the type of `expr`). **Only** rvalue expressions may be given as arguments to such function parameters; if a non-rvalue expression is used, then overload resolution will pick any function that does not use an rvalue reference parameter. And if none exist, then you get an error.

> 更准确地说，右值表达式可以用作函数的自变量，该函数接受类型为“T&&”的参数（其中“T”是“expr”的类型）**只能将**右值表达式作为此类函数参数的自变量；如果使用非右值表达式，则重载解析将选择任何不使用右值引用参数的函数。如果不存在，则会出现错误。

The category of rvalue expressions includes all xvalue and prvalue expressions, and only those expressions.

> 右值表达式的类别包括所有 xvalue 和 prvalue 表达式，并且仅包括这些表达式。

The standard library function `std::move` exists to explicitly transform a non-rvalue expression into an rvalue. More specifically, it turns the expression into an xvalue, since even if it was an identity-less prvalue expression before, by passing it as a parameter to `std::move`, it gains identity (the function's parameter name) and becomes an xvalue.

> 标准库函数“std:：move”用于将非右值表达式显式转换为右值。更具体地说，它将表达式转换为 xvalue，因为即使它以前是一个无标识的 prvalue 表达式，通过将其作为参数传递给“std:：move”，它也会获得标识（函数的参数名称）并变为 xvalue。

Consider the following:

```cpp
std::string str("init");                       //1
std::string test1(str);                        //2
std::string test2(std::move(str));             //3

str = std::string("new value");                //4 
std::string &&str_ref = std::move(str);        //5
std::string test3(str_ref);                    //6

```

`std::string` has a constructor which takes a single parameter of type `std::string&&`, commonly called a "move constructor". However, the value category of the expression `str` is not an rvalue (specifically it is an lvalue), so it cannot call that constructor overload. Instead, it calls the `const std::string&` overload, the copy constructor.

Line 3 changes things. The return value of `std::move` is a `T&&`, where `T` is the base type of the parameter passed in. So `std::move(str)` returns `std::string&&`. A function call who's return value is an rvalue reference is an rvalue expression (specifically an xvalue), so it may call the move constructor of `std::string`. After line 3, `str` has been moved from (who's contents are now undefined).

> 第 3 行改变了一切。“std:：move”的返回值是一个“T&&”，其中“T”是传入参数的基类型。因此“std：：move（str）”返回“std::string&&”。返回值为右值的函数调用是右值表达式（特别是 xvalue），因此它可以调用“std:：string”的 move 构造函数。在第 3 行之后，“str”已从中移出（who 的内容现在未定义）。

Line 4 passes a temporary to the assignment operator of `std::string`. This has an overload which takes a `std::string&&`. The expression `std::string("new value")` is an rvalue expression (specifically a prvalue), so it may call that overload. Thus, the temporary is moved into `str`, replacing the undefined contents with specific contents.

> 第 4 行将一个临时值传递给“std:：string”的赋值运算符。这有一个重载，它接受一个“std:：string&&”。表达式“std:：string（“new value”）”是一个右值表达式（特别是一个 prvalue），因此它可能会调用该重载。因此，临时内容被移到“str”中，用特定内容替换未定义的内容。

Line 5 creates a named rvalue reference called `str_ref` that refers to `str`. This is where value categories get confusing.

> 第 5 行创建了一个名为“str_ref”的命名右值引用，该引用引用了“str”。这就是价值类别令人困惑的地方。

See, while `str_ref` is an rvalue reference to `std::string`, the value category of the expression `str_ref` **is not an rvalue**. It is an lvalue expression. Yes, really. Because of this, one cannot call the move constructor of `std::string` with the expression `str_ref`. Line 6 therefore **copies** the value of `str` into `test3`.

> 请参阅，虽然“str_ref”是对“std:：string”的右值引用，但表达式“str_ref`**”的值类别不是右值**。它是一个左值表达式。是的，真的。因此，不能用表达式“str_ref”调用“std:：string”的 move 构造函数。因此，第 6 行**将“str”的值复制到“test3”中。

To move it, we would have to employ `std::move` again.

## xvalue

An xvalue (eXpiring value) expression is an expression which has identity and represents an object which can be implicitly moved from. The general idea with xvalue expressions is that the object they represent is going to be destroyed soon (hence the "eXpiring" part), and therefore implicitly moving from them is fine.

> xvalue（eXpiring value）表达式是一个具有同一性的表达式，表示可以隐式移动的对象。xvalue 表达式的总体思想是，它们所代表的对象很快就会被破坏（因此是“eXpiring”部分），因此隐含地从它们中移动是可以的。

Given:

```cpp
struct X { int n; };
extern X x;

4;                   // prvalue: does not have an identity
x;                   // lvalue
x.n;                 // lvalue
std::move(x);        // xvalue
std::forward<X&>(x); // lvalue
X{4};                // prvalue: does not have an identity
X{4}.n;              // xvalue: does have an identity and denotes resources
                     // that can be reused

```

## prvalue

A prvalue (pure-rvalue) expression is an expression which lacks identity, whose evaluation is typically used to initialize an object, and which can be implicitly moved from. These include, but are not limited to:

> prvalue（纯右值）表达式是一个缺乏同一性的表达式，其求值通常用于初始化对象，并且可以从中隐式移动。其中包括但不限于：

- Expressions that represent temporary objects, such as `std::string("123")`.
- A function call expression that does not return a reference
- A literal (**except** a string literal - those are lvalues), such has `1`, `true`, `0.5f`, or `'a'`

> -文字（**除外**字符串文字-这些都是左值），例如有“1”、“true”、“0.5f”或“A”`

- A lambda expression

The built-in addressof operator (`&`) cannot be applied on these expressions.

> 内置的 addressof 运算符（`&`）不能应用于这些表达式。

## lvalue

An lvalue expression is an expression which has identity, but cannot be implicitly moved from. Among these are expressions that consist of a variable name, function name, expressions that are built-in dereference operator uses and expressions that refer to lvalue references.

> 左值表达式是一个具有标识的表达式，但不能从中隐式移动。其中包括由变量名、函数名、内置解引用运算符使用的表达式以及引用左值引用的表达式组成的表达式。

The typical lvalue is simply a name, but lvalues can come in other flavors as well:

> 典型的左值只是一个名字，但左值也可以有其他风格：

```cpp
struct X { ... };

X x;         // x is an lvalue
X* px = &x;  // px is an lvalue
*px = X{};   // *px is also an lvalue, X{} is a prvalue

X* foo_ptr();  // foo_ptr() is a prvalue
X& foo_ref();  // foo_ref() is an lvalue

```

Additionally, while most literals (e.g. `4`, `'x'`, etc.) are prvalues, string literals are lvalues.

> 此外，虽然大多数文字（例如“4”、“x”等）是 prvalue，但字符串文字是 lvalue。

## glvalue

A glvalue (a "generalized lvalue") expression is any expression which has identity, regardless of whether it can be moved from or not. This category includes lvalues (expressions that have identity but can't be moved from) and xvalues (expressions that have identity, and can be moved from), but excludes prvalues (expressions without identity).

> glvalue（“广义左值”）表达式是任何具有同一性的表达式，无论它是否可以从中移动。该类别包括 lvalues（具有标识但不能从中移动的表达式）和 xvalues（具有标识符且可以从中移动），但不包括 prvalues（没有标识的表达式）。

If an expression has a **name**, it's a glvalue:

```cpp
struct X { int n; };
X foo();

X x;
x; // has a name, so it's a glvalue
std::move(x); // has a name (we're moving from "x"), so it's a glvalue
              // can be moved from, so it's an xvalue not an lvalue

foo(); // has no name, so is a prvalue, not a glvalue
X{};   // temporary has no name, so is a prvalue, not a glvalue
X{}.n; // HAS a name, so is a glvalue. can be moved from, so it's an xvalue

```
