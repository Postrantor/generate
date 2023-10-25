---
tip: translate by baidu@2023-10-25 08:33:44
---
---
metaTitle: "C++ | Storage class specifiers"
description: "register, extern, mutable, static, auto"
---

# Storage class specifiers



Storage class specifiers are [keywords](http://stackoverflow.com/documentation/c%2b%2b/4891/keywords) that can be used in declarations. They do not affect the type of the declaration, but typically modify the way in which the entity is stored.

> 存储类说明符为[keywords](http://stackoverflow.com/documentation/c%2b%2b/4891/keywords)可以在声明中使用。它们不会影响声明的类型，但通常会修改实体的存储方式。



## register



A storage class specifier that hints to the compiler that a variable will be heavily used. The word "register" is related to the fact that a compiler might choose to store such a variable in a CPU register so that it can be accessed in fewer clock cycles. It was deprecated starting in C++11.

> 一种存储类说明符，它向编译器提示变量将被大量使用。“寄存器”一词与编译器可能选择将这样的变量存储在CPU寄存器中，以便在较少的时钟周期内访问它有关。它从C++11开始就被弃用了。

```cpp
register int i = 0;
while (i < 100) {
    f(i);
    int g = i*i;
    i += h(i, g);
}

```


Both local variables and function parameters may be declared `register`. Unlike C, C++ does not place any restrictions on what can be done with a `register` variable. For example, it is valid to take the address of a `register` variable, but this may prevent the compiler from actually storing such a variable in a register.

> 局部变量和函数参数都可以声明为“register”。与C不同，C++对“register”变量的作用没有任何限制。例如，取“register”变量的地址是有效的，但这可能会阻止编译器将这样的变量实际存储在寄存器中。


The keyword `register` is unused and reserved. A program that uses the keyword `register` is ill-formed.

> 关键字“register”未使用且已保留。使用关键字“register”的程序格式不正确。



## extern



The `extern` storage class specifier can modify a declaration in one of the three following ways, depending on context:

> “extern”存储类说明符可以通过以下三种方式之一修改声明，具体取决于上下文：

<li>

It can be used to declare a variable without defining it. Typically, this is used in a header file for a variable that will be defined in a separate implementation file.

> 它可以用于声明变量而不定义变量。通常，它用于将在单独的实现文件中定义的变量的头文件中。

```cpp
// global scope
int x;             // definition; x will be default-initialized
extern int y;      // declaration; y is defined elsewhere, most likely another TU
extern int z = 42; // definition; "extern" has no effect here (compiler may warn)

```


</li>
<li>

It gives external linkage to a variable at namespace scope even if `const` or `constexpr` would have otherwise caused it to have internal linkage.

> 它在命名空间范围内为变量提供外部链接，即使“const”或“constexpr”会导致它具有内部链接。

```cpp
// global scope
const int w = 42;            // internal linkage in C++; external linkage in C
static const int x = 42;     // internal linkage in both C++ and C
extern const int y = 42;     // external linkage in both C++ and C
namespace {
    extern const int z = 42; // however, this has internal linkage since
                             // it's in an unnamed namespace
}

```


</li>
<li>

It redeclares a variable at block scope if it was previously declared with linkage. Otherwise, it declares a new variable with linkage, which is a member of the nearest enclosing namespace.

> 如果变量以前是用链接声明的，那么它会在块范围内重新声明该变量。否则，它将声明一个带有链接的新变量，该变量是最近封闭命名空间的成员。

```cpp
// global scope
namespace {
    int x = 1;
    struct C {
        int x = 2;
        void f() {
            extern int x;           // redeclares namespace-scope x
            std::cout << x << '\n'; // therefore, this prints 1, not 2
        }
    };
}
void g() {
    extern int y; // y has external linkage; refers to global y defined elsewhere
}

```


</li>


A function can also be declared `extern`, but this has no effect. It is usually used as a hint to the reader that a function declared here is defined in another translation unit. For example:

> 函数也可以声明为“extern”，但这没有任何作用。它通常被用来提示读者，这里声明的函数是在另一个翻译单元中定义的。例如：

```

void f();        // typically a forward declaration; f defined later in this TU
 extern void g(); // typically not a forward declaration; g defined in another TU

```


In the above code, if `f` were changed to `extern` and `g` to non-`extern`, it would not affect the correctness or semantics of the program at all, but would likely confuse the reader of the code.

> 在上述代码中，如果将“f”改为“extern”，将“g”改为非“exten”，则根本不会影响程序的正确性或语义，但可能会混淆代码的读者。



## mutable



A specifier that can be applied to the declaration of a non-static, non-reference data member of a class. A mutable member of a class is not `const` even when the object is `const`.

> 一种说明符，可应用于类的非静态、非引用数据成员的声明。类的可变成员不是const，即使对象是const。

```cpp
class C {
    int x;
    mutable int times_accessed;
  public:
    C(): x(0), times_accessed(0) {
    }
    int get_x() const {
        ++times_accessed; // ok: const member function can modify mutable data member
        return x;
    }
    void set_x(int x) {
        ++times_accessed;
        this->x = x;
    }
};

```


A second meaning for `mutable` was added in C++11. When it follows the parameter list of a lambda, it suppresses the implicit `const` on the lambda's function call operator. Therefore, a mutable lambda can modify the values of entities captured by copy. See [mutable lambdas](http://stackoverflow.com/documentation/c%2b%2b/2705/mutable-keyword/9059/mutable-lambdas) for more details.

> 在C++11中增加了“可变”的第二个含义。当它跟在lambda的参数列表后面时，它会抑制lambda的函数调用运算符上的隐式“const”。因此，可变lambda可以修改复制捕获的实体的值。参见[可变lambdas](http://stackoverflow.com/documentation/c%2b%2b/2705/mutable-关键字/9059/mutablelambdas）。

```cpp
std::vector<int> my_iota(int start, int count) {
    std::vector<int> result(count);
    std::generate(result.begin(), result.end(),
                  [start]() mutable { return start++; });
    return result;
}

```


Note that `mutable` is **not** a storage class specifier when used this way to form a mutable lambda.

> 请注意，当以这种方式用于形成可变lambda时，“mutable”是**而不是**存储类说明符。



## static


The `static` storage class specifier has three different meanings.

<li>

Gives internal linkage to a variable or function declared at namespace scope.

> 为在命名空间范围内声明的变量或函数提供内部链接。

```cpp
// internal function; can't be linked to
static double semiperimeter(double a, double b, double c) {
    return (a + b + c)/2.0;
}
// exported to client
double area(double a, double b, double c) {
    const double s = semiperimeter(a, b, c);
    return sqrt(s*(s-a)*(s-b)*(s-c));
}

```


</li>
<li>

Declares a variable to have static storage duration (unless it is `thread_local`). Namespace-scope variables are implicitly static. A static local variable is initialized only once, the first time control passes through its definition, and is not destroyed every time its scope is exited.

> 声明一个变量具有静态存储持续时间（除非它是“thread_local”）。命名空间作用域变量是隐式静态的。静态局部变量只初始化一次，第一次控件通过其定义时，并且不会在每次退出其作用域时销毁。

```cpp
void f() {
    static int count = 0;
    std::cout << "f has been called " << ++count << " times so far\n";
}

```


</li>
<li>

When applied to the declaration of a class member, declares that member to be a [static member](http://stackoverflow.com/documentation/c%2b%2b/508/classes-structures/15150/static-class-members).

> 当应用于类成员的声明时，将该成员声明为[静态成员](http://stackoverflow.com/documentation/c%2b%2b/508/classes-结构/1550/静态类成员）。

```cpp
struct S {
    static S* create() {
        return new S;
    }
};
int main() {
    S* s = S::create();
}

```


</li>


Note that in the case of a static data member of a class, both 2 and 3 apply simultaneously: the `static` keyword both makes the member into a static data member and makes it into a variable with static storage duration.

> 请注意，在类的静态数据成员的情况下，2和3同时适用：“static”关键字既使该成员成为静态数据成员，又使其成为具有静态存储持续时间的变量。



## auto



Declares a variable to have automatic storage duration. It is redundant, since automatic storage duration is already the default at block scope, and the auto specifier is not allowed at namespace scope.

> 声明一个变量具有自动存储持续时间。它是多余的，因为自动存储持续时间已经是块范围内的默认值，并且在命名空间范围内不允许使用自动说明符。

```cpp
void f() {
    auto int x; // equivalent to: int x;
    auto y;     // illegal in C++; legal in C89
}
auto int z;     // illegal: namespace-scope variable cannot be automatic

```


In C++11, `auto` changed meaning completely, and is no longer a storage class specifier, but is instead used for [type deduction](http://stackoverflow.com/documentation/c%2b%2b/7863/type-deduction/25567/auto-type-deduction).

> 在C++11中，“auto”完全改变了含义，不再是存储类说明符，而是用于[类型推导](http://stackoverflow.com/documentation/c%2b%2b/7863/type-扣除/25567/自动类型扣除）。



#### Remarks



There are six storage class specifiers, although not all in the same version of the language: `auto` (until C++11), `register` (until C++17), `static`, [`thread_local`](http://stackoverflow.com/documentation/c%2b%2b) (since C++11), `extern`, and `mutable`.

> 有六个存储类说明符，尽管并非都在同一版本的语言中：“auto”（直到C++11）、“register”（到C++17）、“static”、[“thread_local”](http://stackoverflow.com/documentation/c%2b%2b)（从C++11开始）、“extern”和“mutable”。

According to the standard,

> 

At most one **storage-class-specifier** shall appear in a given **decl-specifier-seq,** except that `thread_local` may appear with `static` or `extern`.

> 除了“thread_local”可以与“static”或“extern”一起出现之外，在给定的**decl说明符seq中最多应出现一个**存储类说明符**。



A declaration may contain no storage class specifier. In that case, the language specifies a default behaviour. For example, by default, a variable declared at block scope implicitly has automatic storage duration.

> 声明不能包含任何存储类说明符。在这种情况下，语言指定默认行为。例如，默认情况下，在块范围内声明的变量隐式地具有自动存储持续时间。

