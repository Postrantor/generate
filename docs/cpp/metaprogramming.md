---
tip: translate by baidu@2023-10-25 08:27:42
---
---

metaTitle: "C++ | Metaprogramming"
description: "Calculating Factorials, Iterating over a parameter pack, Iterating with std::integer_sequence, Tag Dispatching, Detect Whether Expression is Valid, If-then-else, Calculating power with C++11 (and higher), Manual distinction of types when given any type T, Generic Min/Max with variable argument count"
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# Metaprogramming

In C++ Metaprogramming refers to the use of macros or templates to generate code at compile-time.

> 在 C++ 中，元编程是指在编译时使用宏或模板生成代码。

In general, macros are frowned upon in this role and templates are preferred, although they are not as generic.

> 一般来说，宏在这个角色中是不受欢迎的，模板是首选，尽管它们不是通用的。

Template metaprogramming often makes use of compile-time computations, whether via templates or `constexpr` functions, to achieve its goals of generating code, however compile-time computations are not metaprogramming per se.

> 模板元编程通常利用编译时计算，无论是通过模板还是“constexpr”函数，来实现生成代码的目标，然而编译时计算本身并不是元编程。

## Calculating Factorials

Factorials can be computed at compile-time using template metaprogramming techniques.

> 可以使用模板元编程技术在编译时计算阶乘。

```cpp
#include <iostream>

template<unsigned int n>
struct factorial
{
    enum
    {
        value = n * factorial<n - 1>::value
    };
};

template<>
struct factorial<0>
{
    enum { value = 1 };
};

int main()
{
    std::cout << factorial<7>::value << std::endl;    // prints "5040"
}

```

`factorial` is a struct, but in template metaprogramming it is treated as a template metafunction. By convention, template metafunctions are evaluated by checking a particular member, either `::type` for metafunctions that result in types, or `::value` for metafunctions that generate values.

In the above code, we evaluate the `factorial` metafunction by instantiating the template with the parameters we want to pass, and using `::value` to get the result of the evaluation.

> 在上面的代码中，我们通过用我们想要传递的参数实例化模板来评估“factorial”元函数，并使用“：：value”来获得评估结果。

The metafunction itself relies on recursively instantiating the same metafunction with smaller values. The `factorial<0>` specialization represents the terminating condition. Template metaprogramming has most of the restrictions of a [functional programming language](https://en.wikipedia.org/wiki/Functional_programming), so recursion is the primary "looping" construct.

> 元函数本身依赖于用较小的值递归实例化相同的元函数。“factorial＜0>”特殊化表示终止条件。模板元编程具有[函数式编程语言]的大部分限制([https://en.wikipedia.org/wiki/Functional_programming](https://en.wikipedia.org/wiki/Functional_programming))，所以递归是主要的“循环”构造。

Since template metafunctions execute at compile time, their results can be used in contexts that require compile-time values. For example:

> 由于模板元函数在编译时执行，因此它们的结果可以在需要编译时值的上下文中使用。例如：

```cpp
int my_array[factorial<5>::value];

```

Automatic arrays must have a compile-time defined size. And the result of a metafunction is a compile-time constant, so it can be used here.

> 自动数组必须具有编译时定义的大小。元函数的结果是一个编译时间常数，所以它可以在这里使用。

**Limitation**: Most of the compilers won't allow recursion depth beyond a limit. For example, `g++` compiler by default limits recursion depeth to 256 levels. In case of `g++`, programmer can set recursion depth using `-ftemplate-depth-X` option.

> **限制**：大多数编译器都不允许递归深度超过限制。例如，“g++”编译器默认情况下将递归限制为 256 级。在“g++”的情况下，程序员可以使用“-ftemplate-depth-X”选项设置递归深度。

Since C++11, the `std::integral_constant` template can be used for this kind of template computation:

> 由于 C++11，“std:：integral_content”模板可用于此类模板计算：

```cpp
#include <iostream>
#include <type_traits>

template<long long n>
struct factorial :
  std::integral_constant<long long, n * factorial<n - 1>::value> {};

template<>
struct factorial<0> :
  std::integral_constant<long long, 1> {};

int main()
{
    std::cout << factorial<7>::value << std::endl;    // prints "5040"
}

```

Additionally, `constexpr` functions become a cleaner alternative.

```cpp
#include <iostream>

constexpr long long factorial(long long n)
{
  return (n == 0) ? 1 : n * factorial(n - 1);
}

int main()
{
  char test[factorial(3)];
  std::cout << factorial(7) << '\n';
}

```

The body of `factorial()` is written as a single statement because in C++11 `constexpr` functions can only use a quite limited subset of the language.

> “factorial（）”的主体被写成一个单独的语句，因为在 C++11 中，“constexpr”函数只能使用该语言的一个非常有限的子集。

Since C++14, many restrictions for `constexpr` functions have been dropped and they can now be written much more conveniently:

> 自从 C++14 以来，“constexpr”函数的许多限制已经被取消，现在可以更方便地编写它们：

```cpp
constexpr long long factorial(long long n)
{
  if (n == 0)
    return 1;
  else
    return n * factorial(n - 1);
}

```

Or even:

```cpp
constexpr long long factorial(int n)
{
  long long result = 1;
  for (int i = 1; i <= n; ++i) {
    result *= i;
  }
  return result;
}

```

Since c++17 one can use fold expression to calculate factorial:

```cpp
#include <iostream>
#include <utility>

template <class T, T N, class I = std::make_integer_sequence<T, N>>
struct factorial;

template <class T, T N, T... Is>
struct factorial<T,N,std::index_sequence<T, Is...>> {
   static constexpr T value = (static_cast<T>(1) * ... * (Is + 1));
};

int main() {
   std::cout << factorial<int, 5>::value << std::endl;
}

```

## Iterating over a parameter pack

Often, we need to perform an operation over every element in a variadic template parameter pack. There are many ways to do this, and the solutions get easier to read and write with C++17. Suppose we simply want to print every element in a pack. The simplest solution is to recurse:

> 通常，我们需要对可变模板参数包中的每个元素执行操作。有很多方法可以做到这一点，并且使用 C++17 可以更容易地读取和编写解决方案。假设我们只是想打印一个包中的每个元素。最简单的解决方案是递归：

```cpp
void print_all(std::ostream& os) {
    // base case
}

template <class T, class... Ts>
void print_all(std::ostream& os, T const& first, Ts const&... rest) {
    os << first;
    
    print_all(os, rest...);
}

```

We could instead use the expander trick, to perform all the streaming in a single function. This has the advantage of not needing a second overload, but has the disadvantage of less than stellar readability:

> 相反，我们可以使用扩展器技巧，在一个函数中执行所有流。这样做的优点是不需要第二次过载，但缺点是可读性较差：

```cpp
template <class... Ts>
void print_all(std::ostream& os, Ts const&... args) {
    using expander = int[];
    (void)expander{0,
        (void(os << args), 0)...
    };
}

```

For an explanation of how this works, see [T.C's excellent answer](http://stackoverflow.com/a/25683817/2069064).

> 有关其工作原理的解释，请参阅 [T.C 的精彩回答](http://stackoverflow.com/a/25683817/2069064)。

With C++17, we get two powerful new tools in our arsenal for solving this problem. The first is a fold-expression:

> 有了 C++17，我们的武器库中有两个强大的新工具来解决这个问题。第一个是 fold 表达式：

```cpp
template <class... Ts>
void print_all(std::ostream& os, Ts const&... args) {
    ((os << args), ...);
}

```

And the second is `if constexpr`, which allows us to write our original recursive solution in a single function:

> 第二个是“if constexpr”，它允许我们在单个函数中编写原始递归解决方案：

```cpp
template <class T, class... Ts>
void print_all(std::ostream& os, T const& first, Ts const&... rest) {
    os << first;

    if constexpr (sizeof...(rest) > 0) {        
        // this line will only be instantiated if there are further
        // arguments. if rest... is empty, there will be no call to
        // print_all(os). 
        print_all(os, rest...);
    }
}

```

## Iterating with std::integer_sequence

Since C++14, the standard provides the class template

```cpp
template <class T, T... Ints>
class integer_sequence;

template <std::size_t... Ints>
using index_sequence = std::integer_sequence<std::size_t, Ints...>;

```

and a generating metafunction for it:

```cpp
template <class T, T N>
using make_integer_sequence = std::integer_sequence<T, /* a sequence 0, 1, 2, ..., N-1 */ >;

template<std::size_t N>
using make_index_sequence = make_integer_sequence<std::size_t, N>;

```

While this comes standard in C++14, this can be implemented using C++11 tools.

> 虽然这是 C++14 中的标准配置，但可以使用 C++11 工具来实现。

We can use this tool to call a function with a `std::tuple` of arguments (standardized in C++17 as `std::apply`):

> 我们可以使用此工具来调用具有参数“std:：tuple”的函数（在 C++17 中标准化为“std：：apply”）：

```cpp
namespace detail {
    template <class F, class Tuple, std::size_t... Is>
    decltype(auto) apply_impl(F&& f, Tuple&& tpl, std::index_sequence<Is...> ) {
        return std::forward<F>(f)(std::get<Is>(std::forward<Tuple>(tpl))...);
    }
}

template <class F, class Tuple>
decltype(auto) apply(F&& f, Tuple&& tpl) {
    return detail::apply_impl(std::forward<F>(f),
        std::forward<Tuple>(tpl),
        std::make_index_sequence<std::tuple_size<std::decay_t<Tuple>>::value>{});
}


// this will print 3
int f(int, char, double);

auto some_args = std::make_tuple(42, 'x', 3.14);
int r = apply(f, some_args); // calls f(42, 'x', 3.14)

```

## Tag Dispatching

A simple way of selecting between functions at compile time is to dispatch a function to an overloaded pair of functions that take a tag as one (usually the last) argument. For example, to implement `std::advance()`, we can dispatch on the iterator category:

> 在编译时在函数之间进行选择的一种简单方法是将一个函数分派给一对重载的函数，这些函数将一个标记作为一个（通常是最后一个）参数。例如，要实现“std:：advanced（）”，我们可以对迭代器类别进行分派：

```cpp
namespace details {
    template <class RAIter, class Distance>
    void advance(RAIter& it, Distance n, std::random_access_iterator_tag) {
        it += n;
    }

    template <class BidirIter, class Distance>
    void advance(BidirIter& it, Distance n, std::bidirectional_iterator_tag) {
        if (n > 0) {
            while (n--) ++it;
        }
        else {
            while (n++) --it;
        }
    }

    template <class InputIter, class Distance>
    void advance(InputIter& it, Distance n, std::input_iterator_tag) {
        while (n--) {
            ++it;
        }
    }    
}

template <class Iter, class Distance>
void advance(Iter& it, Distance n) {
    details::advance(it, n, 
            typename std::iterator_traits<Iter>::iterator_category{} );
}

```

The `std::XY_iterator_tag` arguments of the overloaded `details::advance` functions are unused function parameters. The actual implementation does not matter (actually it is completely empty). Their only purpose is to allow the compiler to select an overload based on which tag class `details::advance` is called with.

> 重载的“details:：advance”函数的“std:：XY_iterator_tag”参数是未使用的函数参数。实际的实现并不重要（实际上它是完全空的）。它们的唯一目的是允许编译器根据使用哪个标记类“details:：advance”调用来选择重载。

In this example, `advance` uses the `iterator_traits<T>::iterator_category` metafunction which returns one of the `iterator_tag` classes, depending on the actual type of `Iter`. A default-constructed object of the `iterator_category<Iter>::type` then lets the compiler select one of the different overloads of `details::advance`.

> 在本例中，“advance”使用“iterator_traits＜T＞：iterator_category”元函数，该函数根据“Iter”的实际类型返回“迭代器标记”类之一。“iterator_category＜Iter＞：：type”的默认构造对象允许编译器选择“details:：advanced”的不同重载之一。

(This function parameter is likely to be completely optimized away, as it is a default-constructed object of an empty `struct` and never used.)

> （此函数参数可能会被完全优化掉，因为它是空“struct”的默认构造对象，从未使用过。）

Tag dispatching can give you code that's much easier to read than the equivalents using SFINAE and `enable_if`.

> 标记调度可以为您提供比使用 SFINAE 和“enable_if”的等效代码更容易阅读的代码。

**Note: while C++17's `if constexpr` may simplify the implementation of `advance` in particular, it is not suitable for open implementations unlike tag dispatching.**

> **注意：虽然 C++17 的“if constexpr”可以特别简化“advance”的实现，但它不适合于与标记调度不同的开放实现**

## Detect Whether Expression is Valid

It is possible to detect whether an operator or function can be called on a type. To test if a class has an overload of `std::hash`, one can do this:

> 可以检测是否可以对类型调用运算符或函数。要测试类是否具有“std:：hash”重载，可以执行以下操作：

```cpp
#include <functional> // for std::hash
#include <type_traits> // for std::false_type and std::true_type
#include <utility> // for std::declval

template<class, class = void>
struct has_hash
    : std::false_type
{};

template<class T>
struct has_hash<T, decltype(std::hash<T>()(std::declval<T>()), void())>
    : std::true_type
{};

```

Since C++17, `std::void_t` can be used to simplify this type of construct

```cpp
#include <functional> // for std::hash
#include <type_traits> // for std::false_type, std::true_type, std::void_t
#include <utility> // for std::declval

template<class, class = std::void_t<> >
struct has_hash
    : std::false_type
{};

template<class T>
struct has_hash<T, std::void_t< decltype(std::hash<T>()(std::declval<T>())) > >
    : std::true_type
{};

```

where `std::void_t` is defined as:

```cpp
template< class... > using void_t = void;

```

For detecting if an operator, such as `operator<` is defined, the syntax is almost the same:

> 为了检测是否定义了运算符，如“operator<”，语法几乎相同：

```cpp
template<class, class = void>
struct has_less_than
    : std::false_type
{};

template<class T>
struct has_less_than<T, decltype(std::declval<T>() < std::declval<T>(), void())>
    : std::true_type
{};

```

These can be used to use a `std::unordered_map<T>` if `T` has an overload for `std::hash`, but otherwise attempt to use a `std::map<T>`:

> 如果“T”具有“std:：hash”的重载，则可以使用这些函数来使用“std::：unordereded_map＜T＞”，但在其他情况下尝试使用“std：：map＜T＞：

```cpp
template <class K, class V>
using hash_invariant_map = std::conditional_t<
    has_hash<K>::value,
    std::unordered_map<K, V>,
    std::map<K,V>>;    

```

## If-then-else

The type `std::conditional` in the standard library header `<type_traits>` can select one type or the other, based on a compile-time boolean value:

> 标准库标头“＜type_traits＞”中的类型“std:：conditional”可以根据编译时布尔值选择一种或另一种类型：

```cpp
template<typename T>
struct ValueOrPointer
{
    typename std::conditional<(sizeof(T) > sizeof(void*)), T*, T>::type vop;
};

```

This struct contains a pointer to `T` if `T` is larger than the size of a pointer, or `T` itself if it is smaller or equal to a pointer's size. Therefore `sizeof(ValueOrPointer)` will always be <= `sizeof(void*)`.

> 如果“T”大于指针的大小，则此结构包含指向“T”的指针；如果“T”小于或等于指针的大小则包含指向“T”本身的指针。因此，“sizeof（ValueOrPointer）”将始终 <=“sizeof”（void*）“。

## Calculating power with C++11 (and higher)

With C++11 and higher calculations at compile time can be much easier. For example calculating the power of a given number at compile time will be following:

> 使用 C++11 和更高版本，编译时的计算会容易得多。例如，在编译时计算给定数字的幂如下：

```cpp
template <typename T>
constexpr T calculatePower(T value, unsigned power) {
    return power == 0 ? 1 : value * calculatePower(value, power-1);
}

```

Keyword `constexpr` is responsible for calculating function in compilation time, then and only then, when all the requirements for this will be met (see more at constexpr keyword reference) for example all the arguments must be known at compile time.

> 关键字“constexpr”负责在编译时计算函数，只有在满足所有要求时（请参阅 constexpr 关键字引用中的更多信息），例如，所有参数都必须在编译时已知。

Note: In C++11 `constexpr` function must compose only from one return statement.

> 注意：在 C++11 中，“constexpr”函数只能由一条返回语句组成。

Advantages: Comparing this to the standard way of compile time calculation, this method is also useful for runtime calculations. It means, that if the arguments of the function are not known at the compilation time (e.g. value and power are given as input via user), then function is run in a compilation time, so there's no need to duplicate a code (as we would be forced in older standards of C++).

> 优点：与编译时间计算的标准方法相比，这种方法对运行时计算也很有用。这意味着，如果函数的参数在编译时是未知的（例如，值和幂是通过用户输入的），那么函数是在编译时运行的，因此不需要复制代码（因为我们在旧的 C++ 标准中会被迫这样做）。

E.g.

```cpp
void useExample() {
    constexpr int compileTimeCalculated = calculatePower(3, 3); // computes at compile time,
                               // as both arguments are known at compilation time
                               // and used for a constant expression.
    int value;
    std::cin >> value;
    int runtimeCalculated = calculatePower(value, 3);  // runtime calculated,
                                    // because value is known only at runtime.
}

```

Another way to calculate power at compile time can make use of fold expression as follows:

> 编译时计算幂的另一种方法可以使用 fold 表达式，如下所示：

```cpp
#include <iostream>
#include <utility>

template <class T, T V, T N, class I = std::make_integer_sequence<T, N>>
struct power;

template <class T, T V, T N, T... Is>
struct power<T, V, N, std::integer_sequence<T, Is...>> {
   static constexpr T value = (static_cast<T>(1) * ... * (V * static_cast<bool>(Is + 1)));
};

int main() {
   std::cout << power<int, 4, 2>::value << std::endl;
}

```

## Manual distinction of types when given any type T

When implementing [SFINAE](http://stackoverflow.com/documentation/c%2B%2B/1169/sfinae-substitution-failure-is-not-an-error#t=201607250635342308267) using [`std::enable_if`](http://stackoverflow.com/documentation/c%2B%2B/1169/sfinae-substitution-failure-is-not-an-error/3777/enable-if), it is often useful to have access to helper templates that determines if a given type `T` matches a set of criteria.

> 执行[SFINAE]时([http://stackoverflow.com/documentation/c%2b%2b/1169/sfinae](http://stackoverflow.com/documentation/c%2B%2B/1169/sfinae)-替换失败不是使用[`std:：enable_if`]的错误#t=201607250635342308267）([http://stackoverflow.com/documentation/c%2b%2b/1169/sfinae](http://stackoverflow.com/documentation/c%2B%2B/1169/sfinae)-替换失败不是错误/33777/enable-if），访问帮助模板来确定给定类型“T”是否匹配一组标准通常是有用的。

To help us with that, the standard already provides two types analog to `true` and `false` which are `std::true_type` and `std::false_type`.

> 为了帮助我们做到这一点，该标准已经提供了两种类似于“true”和“false”的类型，即“std:：true_type”和“std::：false _type”。

The following example show how to detect if a type `T` is a pointer or not, the `is_pointer` template mimic the behavior of the standard `std::is_pointer` helper:

> 以下示例显示了如何检测类型“T”是否是指针，“is_pointer”模板模仿标准“std:：is_pointer'助手的行为：

```cpp
template <typename T>
struct is_pointer_: std::false_type {};

template <typename T>
struct is_pointer_<T*>: std::true_type {};

template <typename T>
struct is_pointer: is_pointer_<typename std::remove_cv<T>::type> { }

```

There are three steps in the above code (sometimes you only need two):

<li>

The first declaration of `is_pointer_` is the **default case**, and inherits from `std::false_type`. The **default** case should always inherit from `std::false_type` since it is analogous to a "`false` condition".

> “is_pointer_”的第一个声明是**默认大小写**，它继承自“std:：false _type”。**默认**大小写应始终继承自“std:：false _type”，因为它类似于“false”条件。

</li>
<li>

The second declaration specialize the `is_pointer_` template for pointer `T*` without caring about what `T` is really. This version inherits from `std::true_type`.

> 第二个声明专门化指针“T*”的“is_pointer_”模板，而不关心“T”到底是什么。此版本继承自“std:：true_type”。

</li>
<li>

The third declaration (the real one) simply remove any unnecessary information from `T` (in this case we remove `const` and `volatile` qualifiers) and then fall backs to one of the two previous declarations.

> 第三个声明（真正的声明）只是从“T”中删除任何不必要的信息（在这种情况下，我们删除了“const”和“volatile”限定符），然后返回到前两个声明之一。

</li>

Since `is_pointer<T>` is a class, to access its value you need to either:

- Use `::value`, e.g. `is_pointer<int>::value` – `value` is a static class member of type `bool` inherited from `std::true_type` or `std::false_type`;

> -使用 `:：value`，例如 `is_pointer<int>：：value`–`value` 是继承自 `std:：true_type` 或 `std::false _type` 的类型为 `bool` 的静态类成员；

- Construct an object of this type, e.g. `is_pointer<int>{}` – This works because `std::is_pointer` inherits its default constructor from `std::true_type` or `std::false_type` (which have `constexpr` constructors) and both `std::true_type` and `std::false_type` have `constexpr` conversion operators to `bool`.

> -构造此类型的对象，例如“is_pointer＜int＞｛｝”–这之所以有效，是因为“std:：is_pointer'”从“std：：true_type”或“std::：false_type”（具有“constexpr”构造函数）继承了其默认构造函数，并且“std:”true_type“和“std:。

It is a good habit to provides "helper helper templates" that let you directly access the value:

> 提供“帮助者-帮助者模板”是一个好习惯，可以让您直接访问值：

```cpp
template <typename T>
constexpr bool is_pointer_v = is_pointer<T>::value;

```

In C++17 and above, most helper templates already provide a `_v` version, e.g.:

> 在 C++17 及以上版本中，大多数帮助模板已经提供了“_v”版本，例如：

```cpp
template< class T > constexpr bool is_pointer_v = is_pointer<T>::value;
template< class T > constexpr bool is_reference_v = is_reference<T>::value;

```

## Generic Min/Max with variable argument count

It's possible to write a generic function (for example `min`) which accepts various numerical types and arbitrary argument count by template meta-programming. This function declares a `min` for two arguments and recursively for more.

> 可以编写一个通用函数（例如“min”），通过模板元编程接受各种数字类型和任意参数计数。此函数为两个参数声明“min”，为更多参数声明递归。

```cpp
template <typename T1, typename T2>
auto min(const T1 &a, const T2 &b) 
-> typename std::common_type<const T1&, const T2&>::type
{
    return a < b ? a : b;
}

template <typename T1, typename T2, typename ... Args>
auto min(const T1 &a, const T2 &b, const Args& ... args)
-> typename std::common_type<const T1&, const T2&, const Args& ...>::type
{
    return min(min(a, b), args...);
}

auto minimum = min(4, 5.8f, 3, 1.8, 3, 1.1, 9);

```

#### Remarks

Metaprogramming (or more specifically, Template Metaprogramming) is the practice of using [templates](http://stackoverflow.com/documentation/c%2B%2B/460/templates/3999/basic-class-template#t=201607281319383275025) to create constants, functions, or data structures at compile-time. This allows computations to be performed once at compile time rather than at each run time.

> 元编程（或者更具体地说，模板元编程）是使用[模板]的实践([http://stackoverflow.com/documentation/c%2B%2B/460/templates/3999/basic](http://stackoverflow.com/documentation/c%2B%2B/460/templates/3999/basic)-类模板#t=201607281319383275025），以在编译时创建常量、函数或数据结构。这允许在编译时执行一次计算，而不是在每次运行时执行。
