---
tip: translate by baidu@2023-10-25 08:35:23
---
---

metaTitle: "C++ | Type Traits"
description: "Standard type traits, Type Properties, Type relations with std::is_same<T, T> , Fundamental type traits"
----------------------------------------------------------------------------------------------------------------------

# Type Traits

## Standard type traits

The `type_traits` header contains a set of template classes and helpers to transform and check properties of types at compile-time.

> “type_traits”标头包含一组模板类和助手，用于在编译时转换和检查类型的属性。

These traits are typically used in templates to check for user errors, support generic programming, and allow for optimizations.

> 这些特征通常在模板中用于检查用户错误、支持通用编程并允许优化。

Most type traits are used to check if a type fulfils some criteria. These have the following form:

> 大多数类型特征用于检查一个类型是否满足某些标准。它们具有以下形式：

```cpp
template <class T> struct is_foo;

```

If the template class is instantiated with a type which fulfils some criteria `foo`, then  `is_foo<T>` inherits from `std::integral_constant<bool,true>` (a.k.a. `std::true_type`), otherwise it inherits from `std::integral_constant<bool,false>` (a.k.a. `std::false_type`). This gives the trait the following members:

> 如果模板类是用满足某些条件“foo”的类型实例化的，则“is_foo＜T＞”继承自“std:：integral_content＜bool，true＞”（也称为“std:；true_type”），否则它继承自“std：：integral-content＜bool，false＞”（又称为“std：：false _type”）。这赋予了以下成员特性：

### Constants

`static constexpr bool value`

`true` if `T` fulfils the criteria `foo`, `false` otherwise

### Functions

`operator bool`

Returns `value`

`bool operator()`

Returns `value`

### Types

| Name         | Definition                           |  |  |  |  |  |  |  |  |
| ------------ | ------------------------------------ | - | - | - | - | - | - | - | - |
| `value_type` | `bool`                               |  |  |  |  |  |  |  |  |
| `type`       | `std::integral_constant<bool,value>` |  |  |  |  |  |  |  |  |

The trait can then be used in constructs such as [`static_assert`](http://stackoverflow.com/documentation/c%2B%2B/3822/static-assert#t=201607281007230765701) or [`std::enable_if`](http://stackoverflow.com/documentation/c%2B%2B/1169/sfinae-substitution-failure-is-not-an-error/3777/enable-if#t=201607281006475141537). An example with `std::is_pointer`:

> 然后，该特性可以用于构造，如[`static_assert`]([http://stackoverflow.com/documentation/c%2b%2b/3822/static-assert#t=2016072810077230765701](http://stackoverflow.com/documentation/c%2B%2B/3822/static-assert#t=2016072810077230765701)）或[`std:：enable_if`]([http://stackoverflow.com/documentation/c%2b%2b/1169/sfinae](http://stackoverflow.com/documentation/c%2B%2B/1169/sfinae)-如果#t=20160728100647141537，则替换失败不是错误/3777/启用）。带有“std:：is_pointer”的示例：

```cpp
template <typename T>
void i_require_a_pointer (T t) {
    static_assert(std::is_pointer<T>::value, "T must be a pointer type");
}

//Overload for when T is not a pointer type
template <typename T>
typename std::enable_if<!std::is_pointer<T>::value>::type
does_something_special_with_pointer (T t) {
    //Do something boring
}

//Overload for when T is a pointer type
template <typename T>
typename std::enable_if<std::is_pointer<T>::value>::type 
does_something_special_with_pointer (T t) {
    //Do something special
}

```

There are also various traits which transform types, such as `std::add_pointer` and `std::underlying_type`. These traits generally expose a single `type` member type which contains the transformed type. For example, `std::add_pointer<int>::type` is `int*`.

> 还有各种特性可以转换类型，例如“std:：add_pointer”和“std::：underlying_type”。这些特征通常暴露出包含转换类型的单个“类型”成员类型。例如，“std:：add_pointer＜int＞：：type”是“int*”。

## Type Properties

Type properties compare the modifiers that can be placed upon different variables. The usefulness of these type traits is not always obvious.

> 类型属性比较可以放置在不同变量上的修饰符。这些类型特征的有用性并不总是显而易见的。

**Note:** The example below would only offer an improvement on a non-optimizing compiler. It is a simple a proof of concept, rather than complex example.

> **注意：**下面的示例只会对非优化编译器进行改进。这是一个简单的概念证明，而不是复杂的例子。

e.g. Fast divide by four.

```cpp
template<typename T>
inline T FastDivideByFour(cont T &var) {
  // Will give an error if the inputted type is not an unsigned integral type.    
  static_assert(std::is_unsigned<T>::value && std::is_integral<T>::value,
    "This function is only designed for unsigned integral types.");
  return (var >> 2);
}

```

**Is Constant:**

This will evaluate as true when type is constant.

```cpp
std::cout << std::is_const<const int>::value << "\n"; // Prints true.
std::cout << std::is_const<int>::value << "\n"; // Prints false.

```

**Is Volatile:**

This will evaluate as true when the type is volatile.

```cpp
std::cout << std::is_volatile<static volatile int>::value << "\n"; // Prints true.
std::cout << std::is_const<const int>::value << "\n"; // Prints false.

```

**Is signed:**

This will evaluate as true for all signed types.

```cpp
std::cout << std::is_signed<int>::value << "\n"; // Prints true.
std::cout << std::is_signed<float>::value << "\n"; // Prints true.
std::cout << std::is_signed<unsigned int>::value << "\n"; // Prints false.
std::cout << std::is_signed<uint8_t>::value << "\n"; // Prints false.

```

**Is Unsigned:**

Will evaluate as true for all unsigned types.

```cpp
std::cout << std::is_unsigned<unsigned int>::value << "\n"; // Prints true.
std::cout << std::is_signed<uint8_t>::value << "\n"; // Prints true.
std::cout << std::is_unsigned<int>::value << "\n"; // Prints false.
std::cout << std::is_signed<float>::value << "\n"; // Prints false.

```

## Type relations with std::is_same<T, T>

The `std::is_same<T, T>` type relation is used to compare two types. It will evaluate as boolean, true if the types are the same and false if otherwise.

> “std:：is_same＜T，T＞”类型关系用于比较两种类型。它将计算为布尔值，如果类型相同则为 true，否则为 false。

e.g.

```cpp
// Prints true on most x86 and x86_64 compilers.
std::cout << std::is_same<int, int32_t>::value << "\n";
// Prints false on all compilers.
std::cout << std::is_same<float, int>::value << "\n";
// Prints false on all compilers.
std::cout  << std::is_same<unsigned int, int>::value << "\n";

```

The `std::is_same` type relation will also work regardless of typedefs. This is actually demonstrated in the first example when comparing `int == int32_t` however this is not entirely clear.

> “std:：is_same”类型关系也将工作，而与 typedef 无关。在第一个例子中，当比较“int==int 32_t”时，这一点实际上得到了证明，但这并不完全清楚。

e.g.

```cpp
// Prints true on all compilers.
typedef int MyType
std::cout << std::is_same<int, MyType>::value <<  "\n";

```

**Using `std::is_same`  to warn when improperly using a templated class or function.**

> **当不正确使用模板化的类或函数时，使用“std:：is_same”发出警告**

When combined with a static assert the `std::is_same` template can be valuable tool in enforcing proper usage of templated classes and functions.

> 当与静态断言相结合时，“std:：is_same”模板可以成为强制正确使用模板化类和函数的宝贵工具。

e.g. A function that only allows input from an `int` and a choice of two structs.

> 例如，一个只允许从“int”输入和选择两个结构的函数。

```cpp
#include <type_traits>
struct foo {
  int member;
  // Other variables
};

struct bar {
  char member;
};

template<typename T>
int AddStructMember(T var1, int var2) {
  // If type T != foo || T != bar then show error message.
  static_assert(std::is_same<T, foo>::value || 
    std::is_same<T, bar>::value,
    "This function does not support the specified type.");
  return var1.member + var2;
}

```

## Fundamental type traits

There are a number of different type traits that compare more general types.

> 有许多不同的类型特征可以比较更一般的类型。

**Is Integral:**

Evaluates as true for all integer types `int`, `char`, `long`, `unsigned int` etc.

> 对所有整数类型“int”、“char”、“long”、“unsigned int”等求值为 true。

```cpp
std::cout << std::is_integral<int>::value << "\n"; // Prints true.
std::cout << std::is_integral<char>::value << "\n"; // Prints true.
std::cout << std::is_integral<float>::value << "\n"; // Prints false.

```

**Is Floating Point:**

Evaluates as true for all floating point types. `float`,`double`, `long double` etc.

> 对所有浮点类型计算为 true`float、double、long double 等。

```cpp
std::cout << std::is_floating_point<float>::value << "\n"; // Prints true.
std::cout << std::is_floating_point<double>::value << "\n"; // Prints true.
std::cout << std::is_floating_point<char>::value << "\n"; // Prints false.

```

**Is Enum:**

Evaluates as true for all enumerated types, including `enum class`.

```cpp
enum fruit {apple, pair, banana};
enum class vegetable {carrot, spinach, leek};
std::cout << std::is_enum<fruit>::value << "\n"; // Prints true.
std::cout << std::is_enum<vegetable>::value << "\n"; // Prints true.
std::cout << std::is_enum<int>::value << "\n"; // Prints false.

```

**Is Pointer:**

Evaluates as true for all pointers.

```cpp
std::cout << std::is_pointer<int *>::value << "\n"; // Prints true.
typedef int* MyPTR;
std::cout << std::is_pointer<MyPTR>::value << "\n"; // Prints true.
std::cout << std::is_pointer<int>::value << "\n"; // Prints false.

```

**Is Class:**

Evaluates as true for all classes and struct, with the exception of `enum class`.

> 除“enum class”外，所有类和结构的计算结果均为 true。

```cpp
struct FOO {int x, y;};
class BAR {
 public:
  int x, y;
};
enum class fruit {apple, pair, banana};
std::cout << std::is_class<FOO>::value << "\n"; // Prints true.
std::cout << std::is_class<BAR>::value << "\n"; // Prints true.
std::cout << std::is_class<fruit>::value << "\n"; // Prints false.
std::cout << std::is_class<int>::value << "\n"; // Prints false.

```

#### Remarks

Type traits are templated constructs used to compare and test the properties of different types at compile time. They can be used to provide conditional logic at compile time that can limit or extend the functionality of your code in a specific manner. The type traits library was brought in with the `c++11` standard which provides a number different functionalities. It is also possible to create your own type trait comparison templates.

> 类型特征是用于在编译时比较和测试不同类型的属性的模板化构造。它们可以用于在编译时提供条件逻辑，以特定的方式限制或扩展代码的功能。类型特征库是根据“c++11”标准引入的，该标准提供了许多不同的功能。也可以创建自己的类型特征比较模板。
