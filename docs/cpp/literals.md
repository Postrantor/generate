---
tip: translate by baidu@2023-10-25 08:27:13
---
---
metaTitle: "C++ | Literals"
description: "true, false, nullptr, this, Integer literal"
---

# Literals



Traditionally, a literal is an expression denoting a constant whose type and value are evident from its spelling. For example, `42` is a literal, while `x` is not since one must see its declaration to know its type and read previous lines of code to know its value.

> 传统上，字面是表示常量的表达式，其类型和值从拼写中可以明显看出。例如，“42”是一个文字，而“x”不是，因为必须查看其声明才能知道其类型，并读取前几行代码才能知道其值。


However, C++11 also added [user-defined literals](http://stackoverflow.com/documentation/c%2b%2b/2745/user-defined-literals), which are not literals in the traditional sense but can be used as a shorthand for function calls.

> 然而，C++11也添加了[用户定义的文字](http://stackoverflow.com/documentation/c%2b%2b/2745/user-定义的文字），它们不是传统意义上的文字，但可以用作函数调用的简写。



## true



A [keyword](http://stackoverflow.com/documentation/c%2b%2b/4891/keywords) denoting one of the two possible values of type `bool`.

> A[关键字](http://stackoverflow.com/documentation/c%2b%2b/4891/keywords)表示“bool”类型的两个可能值之一。

```cpp
bool ok = true;
if (!f()) {
    ok = false;
    goto end;
}

```



## false



A [keyword](http://stackoverflow.com/documentation/c%2b%2b/4891/keywords) denoting one of the two possible values of type `bool`.

> A[关键字](http://stackoverflow.com/documentation/c%2b%2b/4891/keywords)表示“bool”类型的两个可能值之一。

```cpp
bool ok = true;
if (!f()) {
    ok = false;
    goto end;
}

```



## nullptr



A [keyword](http://stackoverflow.com/documentation/c%2b%2b/4891/keywords) denoting a null pointer constant. It can be converted to any pointer or pointer-to-member type, yielding a null pointer of the resulting type.

> A[关键字](http://stackoverflow.com/documentation/c%2b%2b/4891/keywords)表示空指针常量。它可以转换为任何指针或指向成员类型的指针，从而生成结果类型的空指针。

```cpp
Widget* p = new Widget();
delete p;
p = nullptr; // set the pointer to null after deletion

```


Note that `nullptr` is not itself a pointer. The type of `nullptr` is a fundamental type known as `std::nullptr_t`.

> 请注意，“nullptr”本身并不是指针。“nullptr”的类型是一种称为“std:：nullptr_t”的基本类型。

```cpp
void f(int* p);

template <class T>
void g(T* p);

void h(std::nullptr_t p);

int main() {
    f(nullptr); // ok
    g(nullptr); // error
    h(nullptr); // ok
}

```



## this



Within a member function of a class, the [keyword](http://stackoverflow.com/documentation/c%2b%2b/4891/keywords) `this` is a pointer to the instance of the class on which the function was called. `this` cannot be used in a static member function.

> 在类的成员函数中，[关键字](http://stackoverflow.com/documentation/c%2b%2b/4891/keywords)“this”是指向调用函数的类实例的指针`this不能在静态成员函数中使用。

```cpp
struct S {
    int x;
    S& operator=(const S& other) {
        x = other.x;
        // return a reference to the object being assigned to
        return *this;
    }
};

```


The type of `this` depends on the cv-qualification of the member function: if `X::f` is `const`, then the type of `this` within `f` is `const X*`, so `this` cannot be used to modify non-static data members from within a `const` member function. Likewise, `this` inherits `volatile` qualification from the function it appears in.

> “this”的类型取决于成员函数的cv限定：如果“X:：f”是“const”，则“f”内的“this”类型为“const X*”，因此“this”不能用于从“const“成员函数中修改非静态数据成员。同样，“this”从其出现的函数继承了“volatile”限定。

`this` can also be used in a **brace-or-equal-initializer** for a non-static data member.

```cpp
struct S;
struct T {
    T(const S* s);
    // ...
};
struct S {
    // ...
    T t{this};
};

```

`this` is an rvalue, so it cannot be assigned to.



## Integer literal


An integer literal is a primary expression of the form

- decimal-literal


It is a non-zero decimal digit (1, 2, 3, 4, 5, 6, 7, 8, 9), followed by zero or more decimal digits (0, 1, 2, 3, 4, 5, 6, 7, 8, 9)

> 它是一个非零十进制数字（1、2、3、4、5、6、7、8、9），后面跟零个或多个十进制数字（0、1、2，3、4，5、6，7、8，9）

`int d = 42;`

- octal-literal


It is the digit zero (0) followed by zero or more octal digits (0, 1, 2, 3, 4, 5, 6, 7)

> 它是数字零（0）后接零个或多个八进制数字（0、1、2、3、4、5、6、7）

`int o = 052`

- hex-literal


It is the character sequence 0x or the character sequence 0X followed by one or more hexadecimal digits (0, 1, 2, 3, 4, 5, 6, 7, 8, 9, a, A, b, B, c, C, d, D, e, E, f, F)

> 它是字符序列0x或字符序列0x，后跟一个或多个十六进制数字（0、1、2、3、4、5、6、7、8、9、a、a、b、b、c、c、d、d、e、e、f、f）

`int x = 0x2a;`
`int X = 0X2A;`

- binary-literal     (since C++14)


It is the character sequence 0b or the character sequence 0B followed by one or more binary digits (0, 1)

> 它是字符序列0b或后面跟着一个或多个二进制数字（0，1）的字符序列0b

`int b = 0b101010; // C++14`


Integer-suffix, if provided, may contain one or both of the following (if both are provided, they may appear in any order:

> 整数后缀（如果提供）可能包含以下内容之一或两者（如果两者都提供），它们可能以任何顺序出现：

- unsigned-suffix (the character u or the character U)

`unsigned int u_1 = 42u;`


- long-suffix (the character l or the character L) or the long-long-suffix (the character sequence ll or the character sequence LL) (since C++11)

> -长后缀（字符l或字符l）或长后缀（字母序列ll或字母序列ll）（由于C++11）

The following variables are also initialized to the same value:

```cpp
unsigned long long l1 = 18446744073709550592ull; // C++11
unsigned long long l2 = 18'446'744'073'709'550'592llu; // C++14
unsigned long long l3 = 1844'6744'0737'0955'0592uLL; // C++14
unsigned long long l4 = 184467'440737'0'95505'92LLU; // C++14

```

**Notes**


Letters in the integer literals are case-insensitive: 0xDeAdBaBeU and 0XdeadBABEu represent the same number (one exception is the long-long-suffix, which is either ll or LL, never lL or Ll)

> 整数文本中的字母不区分大小写：0xDeAdBaBeU和0xDeAdBaBeU表示相同的数字（一个例外是长后缀，它是ll或ll，而不是ll或ll）


There are no negative integer literals. Expressions such as -1 apply the unary minus operator to the value represented by the literal, which may involve implicit type conversions.

> 不存在负整数文字。诸如-1之类的表达式将一元减号运算符应用于文本表示的值，这可能涉及隐式类型转换。


In C prior to C99 (but not in C++), unsuffixed decimal values that do not fit in long int are allowed to have the type unsigned long int.

> 在C99之前的C中（但不在C++中），不适合长整型的未填充十进制值可以具有无符号长整型。


When used in a controlling expression of #if or #elif, all signed integer constants act as if they have type std::intmax_t and all unsigned integer constants act as if they have type std::uintmax_t.

> 当在#if或#elif的控制表达式中使用时，所有有符号整数常量的作用就像它们的类型为std:：intmax_t，而所有无符号整数常数的作用就像他们的类型为std：：uintmax_t。

