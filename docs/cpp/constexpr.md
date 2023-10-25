---
tip: translate by baidu@2023-10-25 08:23:13
---
---

metaTitle: "C++ | constexpr"
description: "constexpr variables, Static if statement, constexpr functions"
----------------------------------------------------------------------------

# constexpr

`constexpr` is a [keyword](http://stackoverflow.com/documentation/c%2B%2B/4891/keywords) that can be used to mark a variable's value as a constant expression, a function as potentially usable in constant expressions, or (since C++17) an [if statement](http://stackoverflow.com/documentation/c%2B%2B/7837/flow-control/18547/if) as having only one of its branches selected to be compiled.

## constexpr variables

A variable declared `constexpr` is implicitly `const` and its value may be used as a constant expression.

> 声明为“constexpr”的变量隐式为“const”，其值可以用作常量表达式。

**Comparison with `#define`**

A `constexpr` is type-safe replacement for `#define` based compile-time expressions. With `constexpr` the compile-time evaluated expression is replaced with the result. For example:

> “constexpr”是基于“#define”的编译时表达式的类型安全替换。使用“constexpr”，编译时计算的表达式将替换为结果。例如：

```cpp
int main()
{
   constexpr int N = 10 + 2;
   cout << N;
}

```

will produce the following code:

```cpp
cout << 12;

```

A pre-processor based compile-time macro would be different. Consider:

```cpp
#define N 10 + 2

int main()
{
    cout << N;
}

```

will produce:

```cpp
cout << 10 + 2;

```

which will obviously be converted to `cout << 10 + 2;`. However, the compiler would have to do more work. Also, it creates a problem if not used correctly.

> 其显然将被转换为“cout＜＜10+2；”。然而，编译器将不得不做更多的工作。此外，如果使用不当，也会产生问题。

For example (with `#define`):

```cpp
cout << N * 2;

```

forms:

```cpp
cout << 10 + 2 * 2; // 14

```

But a pre-evaluated `constexpr` would correctly give `24`.

**Comparison with `const`**

A `const` variable is a **variable** which needs memory for its storage. A `constexpr` does not. A `constexpr` produces compile time constant, which cannot be changed. You may argue that `const` may also not be changed. But consider:

> const 变量是一个需要内存进行存储的**变量。constexpr 没有。“constexpr”生成编译时常量，该常量不能更改。你可能会争辩说，const 也可能不会改变。但请考虑：

```cpp
int main()
{
   const int size1 = 10;
   const int size2 = abs(10);

   int arr_one[size1]; 
   int arr_two[size2]; 
}

```

With most compilers the second statement will fail (may work with GCC, for example). The size of any array, as you might know, has to be a constant expression (i.e. results in compile-time value). The second variable `size2` is assigned some value that is decided at runtime (even though you know it is `10`, for the compiler it is not compile-time).

> 对于大多数编译器，第二条语句都会失败（例如，可能与 GCC 一起使用）。正如您可能知道的，任何数组的大小都必须是一个常量表达式（即产生编译时值）。第二个变量“size2”被分配了一些在运行时决定的值（尽管您知道它是“10”，但对于编译器来说，它不是编译时）。

This means that a `const` may or may not be a true compile-time constant. You cannot guarantee or enforce that a particular `const` value is absolutely compile-time. You may use `#define` but it has its own pitfalls.

> 这意味着“const”可能是也可能不是真正的编译时常数。不能保证或强制特定的“const”值绝对是编译时的。您可以使用“#define”，但它有自己的陷阱。

Therefore simply use:

```cpp
int main()
{
    constexpr int size = 10;

    int arr[size];
}

```

A `constexpr` expression must evaluate to a compile-time value. Thus, you cannot use:

> “constexpr”表达式的计算结果必须为编译时值。因此，您不能使用：

```cpp
constexpr int size = abs(10);

```

Unless the function (`abs`) is itself returning a `constexpr`.

All basic types can be initialized with `constexpr`.

```cpp
constexpr bool FailFatal = true;
constexpr float PI = 3.14f;
constexpr char* site= "StackOverflow";

```

Interestingly, and conveniently, you may also use `auto`:

```cpp
constexpr auto domain = ".COM";  // const char * const domain = ".COM"
constexpr auto PI = 3.14;        // constexpr double

```

## Static if statement

The `if constexpr` statement can be used to conditionally compile code. The condition must be a constant expression. The branch not selected is **discarded.** A discarded statement inside a template is not instantiated. For example:

> “if constexpr”语句可用于有条件地编译代码。条件必须是常量表达式。未选择的分支**被丢弃。**模板内被丢弃的语句不会被实例化。例如：

```cpp
template<class T, class ... Rest>
void g(T &&p, Rest &&...rs)
{
  // ... handle p
  if constexpr (sizeof...(rs) > 0)
    g(rs...);  // never instantiated with an empty argument list
}

```

In addition, variables and functions that are odr-used only inside discarded statements are not required to be defined, and discarded `return` statements are not used for function return type deduction.

> 此外，只在丢弃的语句中使用 odr 的变量和函数不需要定义，并且丢弃的“return”语句不用于函数返回类型推导。

`if constexpr` is distinct from `#ifdef`. `#ifdef` conditionally compiles code, but only based on conditions that can be evaluated at preprocessing time. For example, `#ifdef` could not be used to conditionally compile code depending on the value of a template parameter. On the other hand, `if constexpr` cannot be used to discard syntactically invalid code, while `#ifdef` can.

```cpp
if constexpr(false) {
    foobar;  // error; foobar has not been declared
    std::vector<int> v("hello, world");  // error; no matching constructor
}

```

## constexpr functions

A function that is declared `constexpr` is implicitly inline and calls to such a function potentially yield constant expressions. For example, the following function, if called with constant expression arguments, yields a constant expression too:

> 声明为“constexpr”的函数是隐式内联的，对此类函数的调用可能会产生常量表达式。例如，如果使用常量表达式参数调用以下函数，也会生成常量表达式：

```cpp
constexpr int Sum(int a, int b)
{
    return a + b;
}

```

Thus, the result of the function call may be used as an array bound or a template argument, or to initialize a `constexpr` variable:

> 因此，函数调用的结果可以用作数组绑定或模板参数，或者初始化“constexpr”变量：

```cpp
int main()
{
    constexpr int S = Sum(10,20);
   
    int Array[S];
    int Array2[Sum(20,30)]; // 50 array size, compile time
}

```

Note that if you remove `constexpr` from function's return type specification, assignment to `S` will not work, as `S` is a `constexpr` variable, and must be assigned a compile-time const. Similarly, size of array will also not be a constant-expression, if function `Sum` is not `constexpr`.

> 请注意，如果从函数的返回类型规范中删除“constexpr”，则对“s”的赋值将不起作用，因为“s”是一个“constexpr”变量，必须为其分配一个编译时常量。类似地，若函数“Sum”不是“constexpr”，数组的大小也将不是常量表达式。

Interesting thing about `constexpr` functions is that you may also use it like ordinary functions:

> “constexpr”函数的有趣之处在于，您也可以像使用普通函数一样使用它：

```cpp
int a = 20;
auto sum = Sum(a, abs(-20));

```

`Sum` will not be a `constexpr` function now, it will be compiled as an ordinary function, taking variable (non-constant) arguments, and returning non-constant value. You need not to write two functions.

It also means that if you try to assign such call to a non-const variable, it won't compile:

> 这也意味着，如果你试图将这样的调用分配给一个非常量变量，它将不会编译：

```cpp
int a = 20;
constexpr auto sum = Sum(a, abs(-20));

```

The reason is simple: `constexpr` must only be assigned a compile-time constant. However, the above function call makes `Sum` a non-`constexpr` (R-value is non-const, but L-value is declaring itself to be `constexpr`).

> 原因很简单：“constexpr”只能被分配一个编译时常数。但是，上面的函数调用使“Sum”成为非“constexpr”（R 值是非 const，但 L 值声明自己为“constexpr”）。

The `constexpr` function **must** also return a compile-time constant.  Following will not compile:

> “constexpr”函数**还必须**返回一个编译时常数。以下内容将不会编译：

```cpp
constexpr int Sum(int a, int b)
{
    int a1 = a;     // ERROR
    return a + b;
}

```

Because `a1` is a non-constexpr **variable**, and prohibits the function from being a true `constexpr` function. Making it `constexpr` and assigning it `a` will also not work - since value of `a` (incoming parameter) is still not yet known:

> 因为“a1”是一个非 constexpr **变量**，并禁止该函数成为真正的“constexpr”函数。将其设为“constexpr”并将其赋值为“a”也将不起作用，因为“a”（传入参数）的值仍然未知：

```cpp
constexpr int Sum(int a, int b)
{
   constexpr int a1 = a;     // ERROR
   ..

```

Furthermore, following will also not compile:

```cpp
constexpr int Sum(int a, int b)
{
   return abs(a) + b; // or abs(a) + abs(b)
}

```

Since `abs(a)` is not a constant expression (even `abs(10)` will not work, since `abs` is not returning a `constexpr int` !

> 由于“abs（a）”不是常量表达式（即使是“abs（10）”也不起作用，因为“abs”不会返回“constexpr int”！

What about this?

```cpp
constexpr int Abs(int v)
{
    return v >= 0 ? v : -v;
}

constexpr int Sum(int a, int b)
{
    return Abs(a) + b;
}

```

We crafted our own `Abs` function which is a `constexpr`, and the body of `Abs` also doesn't break any rule. Also, at the call site (inside `Sum`), the expression evaluates to a `constexpr`. Hence, the call to `Sum(-10, 20)` will be a compile-time constant expression resulting to `30`.

> 我们制作了自己的“Abs”函数，它是一个“constexpr”，“Abs’的主体也没有违反任何规则。此外，在调用位置（位于“Sum”内部），表达式的计算结果为“constexpr”。因此，对“Sum（-10，20）”的调用将是导致“30”的编译时常数表达式。

#### Remarks

The `constexpr` keyword was added in C++11 but for a few years since the C++11 standard was published, not all major compilers supported it.  at the time that the C++11 standard was published. As of the time of publication of C++14, all major compilers support `constexpr`.

> “constexpr”关键字是在 C++11 中添加的，但在 C++11 标准发布后的几年里，并不是所有主要的编译器都支持它。截至 C++14 发布时，所有主要编译器都支持“constexpr”。
