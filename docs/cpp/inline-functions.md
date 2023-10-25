---
tip: translate by baidu@2023-10-25 08:26:04
---
---

metaTitle: "C++ | Inline functions"
description: "Non-member inline function declaration, Non-member inline function definition, Member inline functions, What is function inlining?"
-------------------------------------------------------------------------------------------------------------------------------------------------

# Inline functions

A function defined with the `inline` specifier is an inline function. An inline function can be multiply defined without violating the [One Definition Rule](http://stackoverflow.com/documentation/c%2B%2B/4907/one-definition-rule-odr), and can therefore be defined in a header with external linkage. Declaring a function inline hints to the compiler that the function should be inlined during code generation, but does not provide a guarantee.

> 用“inline”说明符定义的函数是内联函数。内联函数可以在不违反[One Definition Rule]的情况下进行乘法定义([http://stackoverflow.com/documentation/c%2b%2b/4907/one](http://stackoverflow.com/documentation/c%2B%2B/4907/one)-定义规则 odr），因此可以在具有外部链接的头中进行定义。内联声明函数提示编译器在代码生成期间应该内联函数，但不能提供保证。

## Non-member inline function declaration

```cpp
inline int add(int x, int y);

```

## Non-member inline function definition

```cpp
inline int add(int x, int y)
{
    return x + y;
}

```

## Member inline functions

```cpp
// header (.hpp)    
struct A
{
    void i_am_inlined()
    {
    }
};

struct B
{
    void i_am_NOT_inlined();
};

// source (.cpp)    
void B::i_am_NOT_inlined()
{
}

```

## What is function inlining?

```cpp
inline int add(int x, int y)
{
    return x + y;
}

int main()
{
    int a = 1, b = 2;
    int c = add(a, b);
}

```

In the above code, when `add` is inlined, the resulting code would become something like this

> 在上面的代码中，当“add”被内联时，生成的代码将变成这样

```cpp
int main()
{
    int a = 1, b = 2;
    int c = a + b;
}

```

The inline function is nowhere to be seen, its body gets **inlined** into the caller's body. Had `add` not been inlined, a function would be called. The overhead of calling a function -- such as creating a new [stack frame](http://stackoverflow.com/q/10057443/183120), copying arguments, making local variables, jump (losing locality of reference and there by cache misses), etc. -- has to be incurred.

> 内联函数看不到，它的主体被**内联**到调用者的主体中。如果没有内联“add”，就会调用一个函数。调用函数的开销，例如创建一个新的[堆栈帧](http://stackoverflow.com/q/10057443/183120)、复制参数、生成局部变量、跳转（由于缓存未命中而丢失引用的局部性）等等。

#### Syntax

- **`inline`** **function_declaration**
- **`inline`** **function_definition**
- **class { function_definition };**

#### Remarks

Usually if code generated for a function is **sufficiently** small then it's a good candidate to be inlined. Why so? If a function is large and is inlined in a loop, for all the calls made, the large function's code would be duplicated leading to the generated binary size bloat. But, how small is sufficient?

> 通常，如果为函数生成的代码足够小，那么它是内联的好候选者。为什么会这样？如果一个函数很大，并且在循环中内联，那么对于所进行的所有调用，大函数的代码都会重复，从而导致生成的二进制大小膨胀。但是，多小就足够了？

While inline functions seem to be great way to avoid function calling overhead, it's to be noted that not all functions that are marked `inline` are inlined. In other words, when you say `inline`, it is only a hint to the compiler, not an order: the compiler isn't obliged to inline the function, it's free to ignore it - most of them do. Modern compilers are better at making such optimisations that this keyword is now a vestige of the past, when this suggestion of function inlining by the programmer

> 虽然内联函数似乎是避免函数调用开销的好方法，但需要注意的是，并非所有标记为“inline”的函数都是内联的。换句话说，当你说“inline”时，它只是对编译器的一个提示，而不是一个命令：编译器没有义务内联函数，可以忽略它-大多数编译器都这样做了。现代编译器更善于进行优化，以至于这个关键字现在已经是过去的遗迹，当程序员提出函数内联的建议时

was taken seriously by the compilers. Even functions not marked `inline` are inlined by the compiler when it sees benefit in doing so.

> 被编译人员认真对待。即使没有标记为“inline”的函数，当编译器看到这样做的好处时，也会内联。

### Inline as a linkage directive

The more practical use of `inline` in modern C++ comes from using it as a linkage directive. When **defining**, not declaring, a function in a header which is going to be included in multiple sources, then each translation unit will have its own copy of this function leading to a [ODR](http://stackoverflow.com/q/4192170/183120) (One Definition Rule) violation; this rule roughly says that there can be only one definition of a function, variable, etc. To circumvent this violation, marking the function definition `inline` implicitly makes the function linkage internal.

> “inline”在现代 C++ 中更实际的用法来自于将其用作链接指令。当**定义**而不是声明将包含在多个源中的头中的函数时，每个翻译单元将有自己的该函数副本，从而导致 [ODR](http://stackoverflow.com/q/4192170/183120)（一个定义规则）违反；这个规则粗略地说，一个函数、变量等只能有一个定义。为了避免这种违反，将函数定义标记为“inline”将使函数链接成为内部链接。

### FAQs

**When should I write the keyword 'inline' for a function/method in C++?**

> **我应该何时在 C++ 中为函数/方法编写关键字“inline”**

Only when you want the function to be defined in a header. More exactly only when the function's definition can show up in multiple compilation units. It's a good idea to define small (as in one liner) functions in the header file as it gives the compiler more information to work with while optimizing your code. It also increases compilation time.

> 仅当您希望在标头中定义函数时。更确切地说，只有当函数的定义可以显示在多个编译单元中时。在头文件中定义小函数（在一行中）是个好主意，因为它在优化代码时为编译器提供了更多的信息。它还增加了编译时间。

**When should I not write the keyword 'inline' for a function/method in C++?**

> **我什么时候不应该在 C++ 中为函数/方法编写关键字“inline”**

Don't add `inline` when you think your code will run faster if the compiler inlines it.

> 当您认为如果编译器内联代码，代码会运行得更快时，不要添加“inline”。

**When will the the compiler not know when to make a function/method inline?**

> **编译器什么时候不知道何时使函数/方法内联**

Generally, the compiler will be able to do this better than you. However, the compiler doesn't have the option to inline code if it doesn't have the function definition. In maximally optimized code usually all private methods are inlined whether you ask for it or not.

> 一般来说，编译器会比你做得更好。然而，如果编译器没有函数定义，那么它就没有内联代码的选项。在最大限度优化的代码中，通常所有私有方法都是内联的，无论您是否要求。

### See Also

<li>

[When should I write the keyword &#39;inline&#39; for a function/method?](http://stackoverflow.com/q/1759300/183120)

> [我应该何时为函数/方法编写关键字&#39；inline&#39；？](http://stackoverflow.com/q/1759300/183120)

</li>
<li>

[Is there still a use for inline?](http://stackoverflow.com/q/29796264/183120)

> [内联还有用武之地吗？](http://stackoverflow.com/q/29796264/183120)

</li>
