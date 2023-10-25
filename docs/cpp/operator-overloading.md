---
tip: translate by baidu@2023-10-25 08:28:35
---
---
metaTitle: "C++ | Operator Overloading"
description: "Array subscript operator, Arithmetic operators, Conversion operators, Complex Numbers Revisited, Unary operators, Comparison operators, Assignment operator, Named operators, Function call operator, Bitwise NOT operator, Bit shift operators for I/O"
---

# Operator Overloading



In C++, it is possible to define operators such as `+` and `->` for user-defined types. For example, the `<string>` header defines a `+` operator to concatenate strings. This is done by defining an **operator function** using the `operator` [keyword](http://stackoverflow.com/documentation/c%2b%2b/4891/keywords).

> 在C++中，可以为用户定义的类型定义诸如“+”和“->'”之类的运算符。例如，“＜string＞”标头定义了一个“+”运算符来连接字符串。这是通过使用“operator”[关键字]定义**运算符函数**来实现的(http://stackoverflow.com/documentation/c%2b%2b/4891/keywords)。



## Array subscript operator


You can even overload the array subscript operator `[]`.


You should **always** (99.98% of the time) implement 2 versions, a `const` and a not-`const` version, because if the object is `const`, it should not be able to modify the object returned by `[]`.

> 您应该**始终**（99.98%的时间）实现两个版本，一个是“const”版本，另一个是非“const“版本，因为如果对象是“const”，它就不能修改“[]”返回的对象。


The arguments are passed by `const&` instead of by value because passing by reference is faster than by value, and `const` so that the operator doesn't change the index accidentally.

> 参数是按“const&”而不是按值传递的，因为按引用传递比按值传递快，而且是按“常量”传递的，这样运算符就不会意外更改索引。


The operators return by reference, because by design you can modify the object `[]` return, i.e:

> 运算符通过引用返回，因为根据设计，您可以修改对象“[]”返回，即：

```cpp
std::vector<int> v{ 1 };
v[0] = 2; //Changes value of 1 to 2
          //wouldn't be possible if not returned by reference

```

You can **only** overload inside a `class`/`struct`:

```cpp
//I is the index type, normally an int
T& operator[](const I& index)
{
    //Do something
    //return something
}

//I is the index type, normally an int
const T& operator[](const I& index) const
{
    //Do something
    //return something
}

```


Multiple subscript operators, `[][]...`, can be achieved via proxy objects. The following example of a simple row-major matrix class demonstrates this:

> 多个下标运算符，`[][]…`，可以通过代理对象来实现。下面的简单行主矩阵类示例演示了这一点：

```cpp
template<class T>
class matrix {
    // class enabling [][] overload to access matrix elements
    template <class C>
    class proxy_row_vector {
        using reference = decltype(std::declval<C>()[0]);
        using const_reference = decltype(std::declval<C const>()[0]);
    public:
        proxy_row_vector(C& _vec, std::size_t _r_ind, std::size_t _cols)
            : vec(_vec), row_index(_r_ind), cols(_cols) {}
        const_reference operator[](std::size_t _col_index) const {
            return vec[row_index*cols + _col_index];
        }
        reference operator[](std::size_t _col_index) {
            return vec[row_index*cols + _col_index];
        }
    private:
        C& vec;
        std::size_t row_index; // row index to access
        std::size_t cols; // number of columns in matrix
    };

    using const_proxy = proxy_row_vector<const std::vector<T>>;
    using proxy = proxy_row_vector<std::vector<T>>;
public:
    matrix() : mtx(), rows(0), cols(0) {}
    matrix(std::size_t _rows, std::size_t _cols)
        : mtx(_rows*_cols), rows(_rows), cols(_cols) {}

    // call operator[] followed by another [] call to access matrix elements
    const_proxy operator[](std::size_t _row_index) const {
        return const_proxy(mtx, _row_index, cols);
    }

    proxy operator[](std::size_t _row_index) {
        return proxy(mtx, _row_index, cols);
    }
private:
    std::vector<T> mtx;
    std::size_t rows;
    std::size_t cols;
};

```



## Arithmetic operators


You can overload all basic arithmetic operators:

- `+` and `+=`
- `-` and `-=`
- `*` and `*=`
- `/` and `/=`
- `&` and `&=`
- `|` and `|=`
- `^` and `^=`
- `>>` and `>>=`
- `<<` and `<<=`

Overloading for all operators is the same. *Scroll down for explanation*

Overloading outside of `class`/`struct`:

```cpp
//operator+ should be implemented in terms of operator+=
T operator+(T lhs, const T& rhs)
{
    lhs += rhs;
    return lhs;
}

T& operator+=(T& lhs, const T& rhs)
{
    //Perform addition
    return lhs;
}

```

Overloading inside of `class`/`struct`:

```cpp
//operator+ should be implemented in terms of operator+=
T operator+(const T& rhs)
{
    *this += rhs;
    return *this;
}

T& operator+=(const T& rhs)
{
    //Perform addition
    return *this;
}

```


Note: `operator+` should return by non-const value, as returning a reference wouldn't make sense (it returns a *new* object) nor would returning a `const` value (you should generally not return by `const`). The first argument is passed by value, why? Because

> 注意：“operator+”应按非常数值返回，因为返回引用没有意义（它返回一个*new*对象），也不会返回“const”值（通常不应按“const’返回）。第一个参数是按值传递的，为什么？因为


1. You can't modify the original object (`Object foobar = foo + bar;` shouldn't modify `foo` after all, it wouldn't make sense)

> 1.你不能修改原始对象（`object foobar=foo+bar；`毕竟不应该修改`foo`，这没有意义）

1. You can't make it `const`, because you will have to be able to modify the object (because `operator+` is implemented in terms of `operator+=`, which modifies the object)

> 1.你不能把它设为“const”，因为你必须能够修改对象（因为“operator+”是根据“operator+=”实现的，它修改对象）


Passing by `const&` would be an option, but then you will have to make a temporary copy of the passed object. By passing by value, the compiler does it for you.

> 传递“const&”是一种选择，但随后您将不得不对传递的对象进行临时复制。通过传递值，编译器为您执行此操作。

`operator+=` returns a reference to the itself, because it is then possible to chain them (don't use the same variable though, that would be undefined behavior due to sequence points).


The first argument is a reference (we want to modify it), but not `const`, because then you wouldn't be able to modify it. The second argument should not be modified, and so for performance reason is passed by `const&` (passing by const reference is faster than by value).

> 第一个参数是引用（我们想修改它），但不是“const”，因为那样你就无法修改它。第二个参数不应该修改，因此出于性能原因，由“const&”传递（通过const引用传递比通过值传递快）。



## Conversion operators



You can overload type operators, so that your type can be implicitly converted into the specified type.

> 您可以重载类型运算符，这样您的类型就可以隐式转换为指定的类型。

The conversion operator **must** be defined in a `class`/`struct`:

```cpp
operator T() const { /* return something */ }

```

*Note: the operator is `const` to allow `const` objects to be converted.*

Example:

```cpp
struct Text
{
    std::string text;

    // Now Text can be implicitly converted into a const char*
    /*explicit*/ operator const char*() const { return text.data(); }
    // ^^^^^^^
    // to disable implicit conversion
};

Text t;
t.text = "Hello world!";

//Ok
const char* copyoftext = t;

```



## Complex Numbers Revisited



The code below implements a very simple complex number type for which the underlying field is automatically promoted, following the language's type promotion rules, under application of the four basic operators (+, -, *, and /) with a member of a different field (be it another `complex<T>` or some scalar type).

> 下面的代码实现了一个非常简单的复数类型，根据语言的类型提升规则，在应用四个基本运算符（+、-、*和/）和不同字段的成员（无论是另一个“complex＜T＞”还是某种标量类型）的情况下，将自动提升其基础字段。


This is intended to be a holistic example covering operator overloading alongside basic use of templates.

> 这是一个全面的例子，涵盖了运算符重载和模板的基本使用。

```cpp
#include <type_traits>

namespace not_std{

using std::decay_t;

//----------------------------------------------------------------
// complex< value_t >
//----------------------------------------------------------------

template<typename value_t>
struct complex
{
    value_t x;
    value_t y;

    complex &operator += (const value_t &x)
    {
        this->x += x;
        return *this;
    }
    complex &operator += (const complex &other)
    {
        this->x += other.x;
        this->y += other.y;
        return *this;
    }

    complex &operator -= (const value_t &x)
    {
        this->x -= x;
        return *this;
    }
    complex &operator -= (const complex &other)
    {
        this->x -= other.x;
        this->y -= other.y;
        return *this;
    }

    complex &operator *= (const value_t &s)
    {
        this->x *= s;
        this->y *= s;
        return *this;
    }
    complex &operator *= (const complex &other)
    {
        (*this) = (*this) * other;
        return *this;
    }

    complex &operator /= (const value_t &s)
    {
        this->x /= s;
        this->y /= s;
        return *this;
    }
    complex &operator /= (const complex &other)
    {
        (*this) = (*this) / other;
        return *this;
    }

    complex(const value_t &x, const value_t &y)
    : x{x}
    , y{y}
    {}

    template<typename other_value_t>
    explicit complex(const complex<other_value_t> &other)
    : x{static_cast<const value_t &>(other.x)}
    , y{static_cast<const value_t &>(other.y)}
    {}

    complex &operator = (const complex &) = default;
    complex &operator = (complex &&) = default;
    complex(const complex &) = default;
    complex(complex &&) = default;
    complex() = default;
};

// Absolute value squared
template<typename value_t>
value_t absqr(const complex<value_t> &z)
{ return z.x*z.x + z.y*z.y; }

//----------------------------------------------------------------
// operator - (negation)
//----------------------------------------------------------------

template<typename value_t>
complex<value_t> operator - (const complex<value_t> &z)
{ return {-z.x, -z.y}; }

//----------------------------------------------------------------
// operator +
//----------------------------------------------------------------

template<typename left_t,typename right_t>
auto operator + (const complex<left_t> &a, const complex<right_t> &b)
-> complex<decay_t<decltype(a.x + b.x)>>
{ return{a.x + b.x, a.y + b.y}; }

template<typename left_t,typename right_t>
auto operator + (const left_t &a, const complex<right_t> &b)
-> complex<decay_t<decltype(a + b.x)>>
{ return{a + b.x, b.y}; }

template<typename left_t,typename right_t>
auto operator + (const complex<left_t> &a, const right_t &b)
-> complex<decay_t<decltype(a.x + b)>>
{ return{a.x + b, a.y}; }

//----------------------------------------------------------------
// operator -
//----------------------------------------------------------------

template<typename left_t,typename right_t>
auto operator - (const complex<left_t> &a, const complex<right_t> &b)
-> complex<decay_t<decltype(a.x - b.x)>>
{ return{a.x - b.x, a.y - b.y}; }

template<typename left_t,typename right_t>
auto operator - (const left_t &a, const complex<right_t> &b)
-> complex<decay_t<decltype(a - b.x)>>
{ return{a - b.x, - b.y}; }

template<typename left_t,typename right_t>
auto operator - (const complex<left_t> &a, const right_t &b)
-> complex<decay_t<decltype(a.x - b)>>
{ return{a.x - b, a.y}; }

//----------------------------------------------------------------
// operator *
//----------------------------------------------------------------

template<typename left_t, typename right_t>
auto operator * (const complex<left_t> &a, const complex<right_t> &b)
-> complex<decay_t<decltype(a.x * b.x)>>
{
    return {
        a.x*b.x - a.y*b.y,
        a.x*b.y + a.y*b.x
        };
}

template<typename left_t, typename right_t>
auto operator * (const left_t &a, const complex<right_t> &b)
-> complex<decay_t<decltype(a * b.x)>>
{ return {a * b.x, a * b.y}; }

template<typename left_t, typename right_t>
auto operator * (const complex<left_t> &a, const right_t &b)
-> complex<decay_t<decltype(a.x * b)>>
{ return {a.x * b, a.y * b}; }

//----------------------------------------------------------------
// operator /
//----------------------------------------------------------------

template<typename left_t, typename right_t>
auto operator / (const complex<left_t> &a, const complex<right_t> &b)
-> complex<decay_t<decltype(a.x / b.x)>>
{
    const auto r = absqr(b);
    return {
        ( a.x*b.x + a.y*b.y) / r,
        (-a.x*b.y + a.y*b.x) / r
        };
}

template<typename left_t, typename right_t>
auto operator / (const left_t &a, const complex<right_t> &b)
-> complex<decay_t<decltype(a / b.x)>>
{
    const auto s = a/absqr(b);
    return {
         b.x * s,
        -b.y * s
        };
}

template<typename left_t, typename right_t>
auto operator / (const complex<left_t> &a, const right_t &b)
-> complex<decay_t<decltype(a.x / b)>>
{ return {a.x / b, a.y / b}; }

}// namespace not_std


int main(int argc, char **argv)
{
    using namespace not_std;

    complex<float> fz{4.0f, 1.0f};

    // makes a complex<double>
    auto dz = fz * 1.0;

    // still a complex<double>
    auto idz = 1.0f/dz;

    // also a complex<double>
    auto one = dz * idz;

    // a complex<double> again
    auto one_again = fz * idz;

    // Operator tests, just to make sure everything compiles.

    complex<float> a{1.0f, -2.0f};
    complex<double> b{3.0, -4.0};

    // All of these are complex<double>
    auto c0 = a + b;
    auto c1 = a - b;
    auto c2 = a * b;
    auto c3 = a / b;

    // All of these are complex<float>
    auto d0 = a + 1;
    auto d1 = 1 + a;
    auto d2 = a - 1;
    auto d3 = 1 - a;
    auto d4 = a * 1;
    auto d5 = 1 * a;
    auto d6 = a / 1;
    auto d7 = 1 / a;

    // All of these are complex<double>
    auto e0 = b + 1;
    auto e1 = 1 + b;
    auto e2 = b - 1;
    auto e3 = 1 - b;
    auto e4 = b * 1;
    auto e5 = 1 * b;
    auto e6 = b / 1;
    auto e7 = 1 / b;

    return 0;
}

```



## Unary operators


You can overload the 2 unary operators:

- `++foo` and `foo++`
- `--foo` and `foo--`


Overloading is the same for both types (`++` and `--`). *Scroll down for explanation*

> 重载对于两种类型（“++”和“--”）都是相同的*向下滚动查看解释*

Overloading outside of `class`/`struct`:

```cpp
//Prefix operator ++foo
T& operator++(T& lhs)
{
    //Perform addition
    return lhs;
}

//Postfix operator foo++ (int argument is used to separate pre- and postfix) 
//Should be implemented in terms of ++foo (prefix operator)
T operator++(T& lhs, int)
{
    T t(lhs);
    ++lhs;
    return t;
}

```

Overloading inside of `class`/`struct`:

```cpp
//Prefix operator ++foo
T& operator++()
{
    //Perform addition
    return *this;
}

//Postfix operator foo++ (int argument is used to separate pre- and postfix) 
//Should be implemented in terms of ++foo (prefix operator)
T operator++(int)
{
    T t(*this);
    ++(*this);
    return t;
}

```


Note: The prefix operator returns a reference to itself, so that you can continue operations on it. The first argument is a reference, as the prefix operator changes the object, that's also the reason why it isn't `const` (you wouldn't be able to modify it otherwise).

> 注意：前缀运算符返回对其自身的引用，这样您就可以继续对其进行操作。第一个参数是引用，因为前缀运算符更改了对象，这也是它不是“const”的原因（否则您将无法修改它）。


The postfix operator returns by value a temporary (the previous value), and so it cannot be a reference, as it would be a reference to a temporary, which would be garbage value at the end of the function, because the temporary variable goes out of scope). It also cannot be `const`, because you should be able to modify it directly.

> 后缀运算符按值返回一个临时值（前一个值），因此它不能是引用，因为它将是对临时值的引用，临时值在函数末尾将是垃圾值，因为临时变量超出了作用域）。它也不能是“const”，因为您应该能够直接修改它。


The first argument is a non-`const` reference to the "calling" object, because if it were `const`, you wouldn't be able to modify it, and if it weren't a reference, you wouldn't change the original value.

> 第一个参数是对“调用”对象的非const引用，因为如果它是const，你就无法修改它，如果它不是引用，你就不会更改原始值。


It is because of the copying needed in postfix operator overloads that it's better to make it a habit to use prefix ++ instead of postfix ++ in `for` loops. From the `for` loop perspective, they're usually functionally equivalent, but there might be a slight performance advantage to using prefix ++, especially with "fat" classes with a lot of members to copy. Example of using prefix ++ in a for loop:

> 正是因为后缀运算符重载中需要复制，所以最好养成在“for”循环中使用前缀++而不是后缀++的习惯。从“for”循环的角度来看，它们通常在功能上是等效的，但使用前缀++可能会带来一些性能优势，尤其是对于要复制大量成员的“fat”类。在for循环中使用前缀++的示例：

```cpp
for (list<string>::const_iterator it = tokens.begin();
     it != tokens.end();
     ++it) { // Don't use it++
    ...
}

```



## Comparison operators


You can overload all comparison operators:

- `==` and `!=`
- `>` and `<`
- `>=` and `<=`


The recommended way to overload all those operators is by implementing only 2 operators (`==` and `<`) and then using those to define the rest. *Scroll down for explanation*

> 重载所有这些运算符的推荐方法是只实现2个运算符（“==”和“<”），然后使用它们来定义其余运算符。*向下滚动查看解释*

Overloading outside of `class`/`struct`:

```cpp
//Only implement those 2
bool operator==(const T& lhs, const T& rhs) { /* Compare */ }
bool operator<(const T& lhs, const T& rhs) { /* Compare */ }

//Now you can define the rest
bool operator!=(const T& lhs, const T& rhs) { return !(lhs == rhs); }
bool operator>(const T& lhs, const T& rhs) { return rhs < lhs; }
bool operator<=(const T& lhs, const T& rhs) { return !(lhs > rhs); }
bool operator>=(const T& lhs, const T& rhs) { return !(lhs < rhs); }

```

Overloading inside of `class`/`struct`:

```cpp
//Note that the functions are const, because if they are not const, you wouldn't be able
//to call them if the object is const

//Only implement those 2
bool operator==(const T& rhs) const { /* Compare */ }
bool operator<(const T& rhs) const { /* Compare */ }

//Now you can define the rest
bool operator!=(const T& rhs) const { return !(*this == rhs); }
bool operator>(const T& rhs) const { return rhs < *this; }
bool operator<=(const T& rhs) const { return !(*this > rhs); }
bool operator>=(const T& rhs) const { return !(*this < rhs); }

```


The operators obviously return a `bool`, indicating `true` or `false` for the corresponding operation.

> 运算符显然返回一个“bool”，表示相应操作的“true”或“false”。


All of the operators take their arguments by `const&`, because the only thing that does operators do is compare, so they shouldn't modify the objects. Passing by `&` (reference) is faster than by value, and to make sure that the operators don't modify it, it is a `const`-reference.

> 所有的运算符都以“const&”作为参数，因为运算符唯一要做的就是比较，所以不应该修改对象。传递“&”（引用）比传递值快，为了确保运算符不会修改它，它是一个“const”-引用。


Note that the operators inside the `class`/`struct` are defined as `const`, the reason for this is that without the functions being `const`, comparing `const` objects would not be possible, as the compiler doesn't know that the operators don't modify anything.

> 请注意，“class”/“struct”中的运算符被定义为“const”，原因是如果函数不是“const’，则无法比较“const‘对象，因为编译器不知道运算符不会修改任何内容。



## Assignment operator



The assignment operator is one of the most important operators because it allows you to change the status of a variable.

> 赋值运算符是最重要的运算符之一，因为它允许您更改变量的状态。


If you do not overload the assigment operator for your `class`/`struct`, it is automatically generated by the compiler: the automatically-generated assignment operator performs a "memberwise assignment", ie by invoking assignment operators on all members, so that one object is copied to the other, a member at time.

> 如果您没有重载“class”/“struct”的赋值运算符，它将由编译器自动生成：自动生成的赋值运算符执行“成员式赋值”，即通过调用所有成员上的赋值运算符来执行，以便一个对象一次复制到另一个对象，即一个成员。

The assignment operator should be overloaded when the simple memberwise assignment is not suitable for your `class`/`struct`, for example if you need to perform a **deep copy** of an object.

> 当简单的成员式赋值不适合“class”/“struct”时，应重载赋值运算符，例如，如果需要执行对象的**深度复制**。


Overloading the assignment operator `=` is easy, but you should follow some simple steps.

> 重载赋值运算符“=”很容易，但您应该遵循一些简单的步骤。

<li>**Test for self-assignment.** This check is important for two reasons:
<ul>

1. a self-assignment is a needless copy, so it does not make sense to perform it;

> 1.自我分配是不必要的复制品，因此执行它没有意义；
1. the next step will not work in the case of a self-assignment.
</ul>
</li>

1. **Clean the old data.** The old data must be replaced with new ones. Now, you can understand the second reason of the previous step: if the content of the object was destroyed, a self-assignment will fail to perform the copy.

> 1.**清理旧数据。**必须用新数据替换旧数据。现在，您可以理解上一步的第二个原因：如果对象的内容被破坏，则自分配将无法执行复制。

1. **Copy all members.** If you overload the assigment operator for your `class` or your `struct`, it is not automatically generated by the compiler, so you will need to take charge of copying all members from the other object.

> 1.**复制所有成员。**如果重载“class”或“struct”的赋值运算符，它不会由编译器自动生成，因此您需要负责复制其他对象的所有成员。

1. **Return** `*this`. The operator returns by itself by reference, because it allows chaining (i.e. `int b = (a = 6) + 4; //b == 10`).

> 1.**返回**`*this`。运算符本身通过引用返回，因为它允许链接（即`int b=（a=6）+4//b＝＝10`）。

```cpp
//T is some type
T& operator=(const T& other)
{
    //Do something (like copying values)
    return *this;
}

```


**Note:** `other` is passed by `const&`, because the object being assigned should not be changed, and passing by reference is faster than by value, and to make sure than `operator=` doesn't modify it accidentally, it is `const`.

> **注意：**`other`由`const&`传递，因为被分配的对象不应更改，并且通过引用传递比通过值传递快，为了确保`operator=`不会意外修改它，它是`const`。


The assignment operator can **only** to be overloaded in the `class`/`struct`, because the left value of `=` is **always** the `class`/`struct` itself. Defining it as a free function doesn't have this guarantee, and is disallowed because of that.

> 赋值运算符只能****在“class”/“struct”中重载，因为“=”的左值**总是**“class”/“struct’本身。将它定义为一个自由函数没有这个保证，因此是不允许的。


When you declare it in the `class`/`struct`, the left value is implicitly the `class`/`struct` itself, so there is no problem with that.

> 当你在“class”/“struct”中声明它时，左边的值隐式地是“class”或“struct'”本身，所以这没有问题。



## Named operators



You can extend C++ with named operators that are "quoted" by standard C++ operators.

> 您可以使用标准C++运算符“引用”的命名运算符来扩展C++。

First we start with a dozen-line library:

```cpp
namespace named_operator {
  template<class D>struct make_operator{constexpr make_operator(){}};

  template<class T, char, class O> struct half_apply { T&& lhs; };

  template<class Lhs, class Op>
  half_apply<Lhs, '*', Op> operator*( Lhs&& lhs, make_operator<Op> ) {
    return {std::forward<Lhs>(lhs)};
  }

  template<class Lhs, class Op, class Rhs>
  auto operator*( half_apply<Lhs, '*', Op>&& lhs, Rhs&& rhs )
  -> decltype( named_invoke( std::forward<Lhs>(lhs.lhs), Op{}, std::forward<Rhs>(rhs) ) )
  {
    return named_invoke( std::forward<Lhs>(lhs.lhs), Op{}, std::forward<Rhs>(rhs) );
  }
}

```

this doesn't do anything yet.

First, appending vectors

```cpp
namespace my_ns {
  struct append_t : named_operator::make_operator<append_t> {};
  constexpr append_t append{};
  
  template<class T, class A0, class A1>
  std::vector<T, A0> named_invoke( std::vector<T, A0> lhs, append_t, std::vector<T, A1> const& rhs ) {
      lhs.insert( lhs.end(), rhs.begin(), rhs.end() );
      return std::move(lhs);
  }
}
using my_ns::append;

std::vector<int> a {1,2,3};
std::vector<int> b {4,5,6};

auto c = a *append* b;

```


The core here is that we define an `append` object of type `append_t:named_operator::make_operator<append_t>`.

> 这里的核心是我们定义了一个类型为`append_t:named_operator:：make_operator<append_t>`的`append`对象。


We then overload named_invoke( lhs, append_t, rhs ) for the types we want on the right and left.

> 然后，我们重载named_invoke（lhs、append_t、rhs），以便在右侧和左侧使用我们想要的类型。


The library overloads `lhs*append_t`, returning a temporary `half_apply` object.  It also overloads `half_apply*rhs` to call `named_invoke( lhs, append_t, rhs )`.

> 库重载“lhs*append_t”，返回一个临时“half_apply”对象。它还重载了“half_apply*rhs”以调用“named_invoke（lhs，append_t，rhs）”。


We simply have to create the proper `append_t` token and do an ADL-friendly `named_invoke` of the proper signature, and everything hooks up and works.

> 我们只需创建适当的“append_t”令牌，并对适当的签名执行ADL友好的“named_invoke”，一切都会连接起来并正常工作。


For a more complex example, suppose you want to have element-wise multiplication of elements of a std::array:

> 对于一个更复杂的例子，假设您想要对std:：array的元素进行逐元素乘法：

```cpp
template<class=void, std::size_t...Is>
auto indexer( std::index_sequence<Is...> ) {
  return [](auto&& f) {
    return f( std::integral_constant<std::size_t, Is>{}... );
  };
}
template<std::size_t N>
auto indexer() { return indexer( std::make_index_sequence<N>{} ); }

namespace my_ns {
  struct e_times_t : named_operator::make_operator<e_times_t> {};
  constexpr e_times_t e_times{};

  template<class L, class R, std::size_t N,
    class Out=std::decay_t<decltype( std::declval<L const&>()*std::declval<R const&>() )>
  >
  std::array<Out, N> named_invoke( std::array<L, N> const& lhs, e_times_t, std::array<R, N> const& rhs ) {
    using result_type = std::array<Out, N>;
    auto index_over_N = indexer<N>();
    return index_over_N([&](auto...is)->result_type {
      return {{
        (lhs[is] * rhs[is])...
      }};
    });
  }
}

```

[live example](http://coliru.stacked-crooked.com/a/c071a0662834c838).


This element-wise array code can be extended to work on tuples or pairs or C-style arrays, or even variable length containers if you decide what to do if the lengths don't match.

> 如果您决定在长度不匹配的情况下该怎么办，则可以扩展此元素数组代码以处理元组或对或C样式数组，甚至可变长度容器。


You could also an element-wise operator type and get `lhs *element_wise<'+'>* rhs`.

> 您也可以使用元素运算符类型，并获得`lhs*element_wise<'+'>*rhs`。

Writing a `*dot*` and `*cross*` product operators are also obvious uses.


The use of `*` can be extended to support other delimiters, like `+`.  The delimeter precidence determines the precidence of the named operator, which may be important when translating physics equations over to C++ with minimal use of extra `()`s.

> “*”的使用可以扩展为支持其他分隔符，如“+”。delimeter精度决定了命名运算符的精度，这在将物理方程转换为C++时可能很重要，而不需要使用额外的“（）”s。


With a slight change in the library above, we can support `->*then*` operators and extend `std::function` prior to the standard being updated, or write monadic `->*bind*`.  It could also have a stateful named operator, where we carefully pass the `Op` down to the final invoke function, permitting:

> 只要对上面的库做一点更改，我们就可以在更新标准之前支持`->*then*`运算符并扩展`std:：function`，或者编写monadic`->*bind*`。它也可以有一个有状态的命名运算符，在这里我们小心地将“Op”传递给最终的invoke函数，允许：

```cpp
named_operator<'*'> append = [](auto lhs, auto&& rhs) {
  using std::begin; using std::end;
  lhs.insert( end(lhs), begin(rhs), end(rhs) );
  return std::move(lhs);
};

```

generating a named container-appending operator in C++17.



## Function call operator


You can overload the function call operator `()`:

Overloading must be done inside of a `class`/`struct`:

```cpp
//R -> Return type
//Types -> any different type
R operator()(Type name, Type2 name2, ...)
{
    //Do something
    //return something
}

//Use it like this (R is return type, a and b are variables)
R foo = object(a, b, ...);

```

For example:

```cpp
struct Sum
{
    int operator()(int a, int b)
    {
        return a + b;
    }
};

//Create instance of struct
Sum sum;
int result = sum(1, 1); //result == 2

```



## Bitwise NOT operator



Overloading the bitwise NOT (`~`) is fairly simple. *Scroll down for explanation*

> 重载按位NOT（`~`）相当简单*向下滚动查看解释*

Overloading outside of `class`/`struct`:

```cpp
T operator~(T lhs)
{
    //Do operation
    return lhs;
}

```

Overloading inside of `class`/`struct`:

```cpp
T operator~()
{
    T t(*this);
    //Do operation
    return t;
}

```


Note: `operator~` returns by value, because it has to return a new value (the modified value), and not a reference to the value (it would be a reference to the temporary object, which would have garbage value in it as soon as the operator is done). Not `const` either because the calling code should be able to modify it afterwards (i.e. `int a = ~a + 1;` should be possible).

> 注意：“operator ~”按值返回，因为它必须返回一个新值（修改后的值），而不是对该值的引用（它将是对临时对象的引用，一旦运算符完成，临时对象中就会有垃圾值）。也不是“const”，因为调用代码之后应该能够修改它（即“int a=~a+1；”应该是可能的）。


Inside the `class`/`struct` you have to make a temporary object, because you can't modify `this`, as it would modify the original object, which shouldn't be the case.

> 在“class”/“struct”中，你必须创建一个临时对象，因为你不能修改“this”，因为它会修改原始对象，而事实并非如此。



## Bit shift operators for I/O



The operators `<<` and `>>` are commonly used as "write" and "read" operators:

> 运算符`<<`和`>>`通常用作“写入”和“读取”运算符：

- `std::ostream` overloads `<<` to write variables to the underlying stream (example: `std::cout`)

- `std::istream` overloads `>>` to read from the underlying stream to a variable (example: `std::cin`)

> -`std:：istream`overloads`>`从底层流读取变量（例如：`std:；cin`）


The way they do this is similar if you wanted to overload them "normally" outside of the `class`/`struct`, except that specifying the arguments are not of the same type:

> 如果您想在“class”/“struct”之外“正常”重载它们，则它们执行此操作的方式类似，只是指定参数的类型不同：


- Return type is the stream you want to overload from (for example, `std::ostream`) passed by reference, to allow chaining (Chaining: `std::cout << a << b;`). Example: `std::ostream&`

> -返回类型是要从引用传递的流（例如，“std:：ostream”）中重载的流，以允许链接（chaining:`std:：cout<<a<<b；`）。示例：`std:：ostream&`
- `lhs` would be the same as the return type

- `rhs` is the type you want to allow overloading from (i.e. `T`), passed by `const&` instead of value for performance reason (`rhs` shouldn't be changed anyway). Example: `const Vector&`.

> -“rhs”是您希望允许重载的类型（即“T”），由于性能原因，由“const&”而不是值传递（无论如何都不应更改“rhs“）。示例：`const Vector&`。

Example:

```cpp
//Overload std::ostream operator<< to allow output from Vector's
std::ostream& operator<<(std::ostream& lhs, const Vector& rhs)
{
    lhs << "x: " << rhs.x << " y: " << rhs.y << " z: " << rhs.z << '\n';
    return lhs;
}

Vector v = { 1, 2, 3};

//Now you can do
std::cout << v;

```



#### Remarks



The operators for built-in types cannot be changed, operators can only be overloaded for user-defined types. That is, at least one of the operands has to be of a user-defined type.

> 不能更改内置类型的运算符，只能重载用户定义类型的运算符。也就是说，至少有一个操作数必须是用户定义的类型。

The following operators **cannot** be overloaded:

- The member access or "dot" operator `.`
- The pointer to member access operator `.*`
- The scope resolution operator, `::`
- The ternary conditional operator, `?:`

- `dynamic_cast`, `static_cast`, `reinterpret_cast`, `const_cast`, `typeid`, `sizeof`, `alignof`, and `noexcept`

> -`dynamic_cast`、`static_cast`，`interpret_cast `、`const_cast `、`typeid`、`sizeof`、`alignof`和`noexcept`

- The preprocessing directives, `#` and `##`, which are executed before any type information is available.

> -预处理指令“#”和“##”在任何类型信息可用之前执行。


There are some operators that you should **not** (99.98% of the time) overload:

> 有些操作员应该**而不是**（99.98%的时间）过载：

- `&&` and `||` (prefer, instead, to use implicit conversion to `bool`)
- `,`
- The address-of operator (unary `&`)


Why? Because they overload operators that another programmer might never expect, resulting in different behavior than anticipated.

> 为什么？因为它们重载了其他程序员可能从未预料到的运算符，导致了与预期不同的行为。


For example, the user defined `&&` and `||` overloads of these operators [lose their short-circuit evaluation](http://en.cppreference.com/w/cpp/language/operators#Restrictions) and [lose their special sequencing properties (C++17)](http://en.cppreference.com/w/cpp/language/operators#Restrictions), the sequencing issue also applies to `,` operator overloads.

> 例如，用户定义的这些运算符的“&&”和“||”过载[失去短路评估](http://en.cppreference.com/w/cpp/language/operators#Restrictions)和[失去它们特殊的测序特性（C++17）](http://en.cppreference.com/w/cpp/language/operators#Restrictions)，排序问题也适用于“，”运算符重载。

