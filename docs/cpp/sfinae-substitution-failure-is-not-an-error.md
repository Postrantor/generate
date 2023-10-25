---
tip: translate by baidu@2023-10-25 08:30:58
---
---
metaTitle: "C++ | SFINAE (Substitution Failure Is Not An Error)"
description: "What is SFINAE, void_t, enable_if, is_detected, Overload resolution with a large number of options, trailing decltype in function templates, enable_if_all / enable_if_any"
---

# SFINAE (Substitution Failure Is Not An Error)




## What is SFINAE



SFINAE stands for **S**ubstitution **F**ailure **I**s **N**ot **A**n **E**rror. Ill-formed code that results from substituting types (or values) to instantiate a function template or a class template is **not** a hard compile error, it is only treated as a deduction failure.

> SFINAE代表**S**取代**F**失败**I**S**N**ot**A**N**E**错误。用类型（或值）来实例化函数模板或类模板所导致的格式错误代码**不是**硬编译错误，它只被视为推导失败。


Deduction failures on instantiating function templates or class template specializations remove that candidate from the set of consideration - as if that failed candidate did not exist to begin with.

> 实例化函数模板或类模板专业化时的推导失败会将该候选项从考虑的集合中删除，就好像失败的候选项一开始就不存在一样。

```cpp
template <class T>
auto begin(T& c) -> decltype(c.begin()) { return c.begin(); }

template <class T, size_t N>
T* begin(T (&arr)[N]) { return arr; }

int vals[10];
begin(vals); // OK. The first function template substitution fails because
             // vals.begin() is ill-formed. This is not an error! That function
             // is just removed from consideration as a viable overload candidate,
             // leaving us with the array overload. 

```


Only substitution failures in the **immediate context** are considered deduction failures, all others are considered hard errors.

> 只有**直接上下文**中的替换失败才被视为推导失败，所有其他都被视为硬错误。

```cpp
template <class T>
void add_one(T& val) { val += 1; }

int i = 4;
add_one(i); // ok

std::string msg = "Hello";
add_one(msg); // error. msg += 1 is ill-formed for std::string, but this
              // failure is NOT in the immediate context of substituting T

```



## void_t


`void_t` is a meta-function that maps any (number of) types to type `void`.
The primary purpose of `void_t` is to facilitate writing of type traits.


[`std::void_t`](http://en.cppreference.com/w/cpp/types/void_t) will be part of C++17, but until then, it is extremely straightforward to implement:

> [`std:：void_t`](http://en.cppreference.com/w/cpp/types/void_t)将是C++17的一部分，但在此之前，实现起来非常简单：

```cpp
template <class...> using void_t = void;

```


Some compilers [require](http://open-std.org/JTC1/SC22/WG21/docs/cwg_defects.html#1558) a slightly different implementation:

> 某些编译器[需要](http://open-std.org/JTC1/SC22/WG21/docs/cwg_defects.html#1558)略有不同的实现方式：

```cpp
template <class...>
struct make_void { using type = void; };

template <typename... T>
using void_t = typename make_void<T...>::type;

```


The primary application of `void_t` is writing type traits that check validity of a statement. For example, let's check if a type has a member function `foo()` that takes no arguments:

> “void_t”的主要应用是编写检查语句有效性的类型特征。例如，让我们检查一个类型是否有一个不带参数的成员函数“foo（）”：

```cpp
template <class T, class=void>
struct has_foo : std::false_type {};

template <class T>
struct has_foo<T, void_t<decltype(std::declval<T&>().foo())>> : std::true_type {};

```


How does this work? When I try to instantiate `has_foo<T>::value`, that will cause the compiler to try to look for the best specialization for `has_foo<T, void>`. We have two options: the primary, and this secondary one which involves having to instantiate that underlying expression:

> 这是怎么回事？当我尝试实例化“has_foo＜T＞：：value”时，这将导致编译器尝试为“has_foo＜T，void＞”寻找最佳专业化。我们有两种选择：一种是主选项，另一种是第二选项，它涉及到必须实例化底层表达式：


- If `T` **does** have a member function `foo()`, then whatever type that returns gets converted to `void`, and the specialization is preferred to the primary  based on the template partial ordering rules. So `has_foo<T>::value` will be `true`

> -如果“T”**确实有一个成员函数“foo（）”，则返回的任何类型都将转换为“void”，并且根据模板部分排序规则，专用化比主类更可取。因此`has_foo<T>：：value`将为`true`

- If `T` **doesn't** have such a member function (or it requires more than one argument), then substitution fails for the specialization and we only have the primary template to fallback on. Hence, `has_foo<T>::value` is `false`.

> -如果`T`**没有**这样的成员函数（或者它需要多个参数），则专用化的替换失败，并且我们只有主模板可供回退。因此，`has_foo<T>：value`为`false`。

A simpler case:

```cpp
template<class T, class=void>
struct can_reference : std::false_type {};

template<class T>
struct can_reference<T, std::void_t<T&>> : std::true_type {};

```

this doesn't use `std::declval` or `decltype`.


You may notice a common pattern of a void argument.  We can factor this out:

> 您可能会注意到一个常见的void参数模式。我们可以考虑一下：

```cpp
struct details {
  template<template<class...>class Z, class=void, class...Ts>
  struct can_apply:
    std::false_type
  {};
  template<template<class...>class Z, class...Ts>
  struct can_apply<Z, std::void_t<Z<Ts...>>, Ts...>:
    std::true_type
  {};
};

template<template<class...>class Z, class...Ts>
using can_apply = details::can_apply<Z, void, Ts...>;

```


which hides the use of `std::void_t` and makes `can_apply` act like an indicator whether the type supplied as the first template argument is well-formed after substituting the other types into it. The previous examples may now be rewritten using `can_apply` as:

> 它隐藏了“std:：void_t”的使用，并使“can_apply”充当一个指示符，在将其他类型代入后，作为第一个模板参数提供的类型是否形成良好

```cpp
template<class T>
using ref_t = T&;

template<class T>
using can_reference = can_apply<ref_t, T>;    // Is T& well formed for T?

```

and:

```cpp
template<class T>
using dot_foo_r = decltype(std::declval<T&>().foo());

template<class T>
using can_dot_foo = can_apply< dot_foo_r, T >;    // Is T.foo() well formed for T?

```

which seems simpler than the original versions.

There are post-C++17 proposals for `std` traits similar to `can_apply`.


The utility of `void_t` was discovered by Walter Brown. He gave a wonderful [presentation](https://youtu.be/a0FliKwcwXE?t=1747) on it at CppCon 2016.

> “void_t”的效用是由沃尔特·布朗发现的。他做了精彩的[介绍](https://youtu.be/a0FliKwcwXE?t=1747)在2016年CppCon上。



## enable_if


`std::enable_if` is a convenient utility to use boolean conditions to trigger SFINAE. It is defined as:

```cpp
template <bool Cond, typename Result=void>
struct enable_if { };

template <typename Result>
struct enable_if<true, Result> {
    using type = Result;
};

```


That is, `enable_if<true, R>::type`  is an alias for `R`, whereas `enable_if<false, T>::type` is ill-formed as that specialization of `enable_if` does not have a `type` member type.

> 也就是说，`enable_if<true，R>：：type`是`R`的别名，而`enable_if<false，T>：：type `格式不正确，因为`enable_idf`的专门化没有`type`成员类型。

`std::enable_if` can be used to constrain templates:

```cpp
int negate(int i) { return -i; }

template <class F>
auto negate(F f) { return -f(); }

```


Here, a call to `negate(1)` would fail due to ambiguity. But the second overload is not intended to be used for integral types, so we can add:

> 在这里，对“否定（1）”的调用将由于歧义而失败。但第二个重载并不用于积分类型，因此我们可以添加：

```cpp
int negate(int i) { return -i; }

template <class F, class = typename std::enable_if<!std::is_arithmetic<F>::value>::type>
auto negate(F f) { return -f(); }

```


Now, instantiating `negate<int>` would result in a substitution failure since `!std::is_arithmetic<int>::value` is `false`. Due to SFINAE, this is not a hard error, this candidate is simply removed from the overload set. As a result, `negate(1)` only has one single viable candidate - which is then called.

> 现在，实例化“negate＜int＞”将导致替换失败，因为“！std:：is_athmetric<int>：：value`为`false`。由于SFINAE，这不是一个硬错误，这个候选者只是从过载集中删除。因此，“否定（1）”只有一个可行的候选者，然后称之为。

### When to use it


It's worth keeping in mind that `std::enable_if` is a helper **on top** of SFINAE, but it's not what makes SFINAE work in the first place. Let's consider these two alternatives for implementing functionality similar to `std::size`, i.e. an overload set `size(arg)` that produces the size of a container or array:

> 值得记住的是，“std:：enable_if”是SFINAE之上的一个辅助**，但它并不是SFINAE最初工作的原因。让我们考虑这两种替代方案来实现类似于“std:：size”的功能，即生成容器或数组大小的重载集“size（arg）”：

```cpp
// for containers
template<typename Cont>
auto size1(Cont const& cont) -> decltype( cont.size() );

// for arrays
template<typename Elt, std::size_t Size>
std::size_t size1(Elt const(&arr)[Size]);

// implementation omitted
template<typename Cont>
struct is_sizeable;

// for containers
template<typename Cont, std::enable_if_t<std::is_sizeable<Cont>::value, int> = 0>
auto size2(Cont const& cont);

// for arrays
template<typename Elt, std::size_t Size>
std::size_t size2(Elt const(&arr)[Size]);

```


Assuming that `is_sizeable` is written appropriately, these two declarations should be exactly equivalent with respect to SFINAE. Which is the easiest to write, and which is the easiest to review and understand at a glance?

> 假设“is_sizeable”写得恰当，那么这两个声明对于SFINAE应该是完全等价的。哪个最容易写，哪个最容易复习和一目了然？


Now let's consider how we might want to implement arithmetic helpers that avoid signed integer overflow in favour of wrap around or modular behaviour. Which is to say that e.g. `incr(i, 3)` would be the same as `i += 3` save for the fact that the result would always be defined even if `i` is an `int` with value `INT_MAX`. These are two possible alternatives:

> 现在，让我们考虑如何实现算术助手，以避免有符号整数溢出，而支持环绕或模块化行为。也就是说，例如“incr（i，3）”将与“i+=3”相同，除了即使“i”是值为“int_MAX”的“int”，结果也将始终被定义这一事实。这是两种可能的替代方案：

```cpp
// handle signed types
template<typename Int>
auto incr1(Int& target, Int amount)
-> std::void_t<int[static_cast<Int>(-1) < static_cast<Int>(0)]>;

// handle unsigned types by just doing target += amount
// since unsigned arithmetic already behaves as intended
template<typename Int>
auto incr1(Int& target, Int amount)
-> std::void_t<int[static_cast<Int>(0) < static_cast<Int>(-1)]>;
 
template<typename Int, std::enable_if_t<std::is_signed<Int>::value, int> = 0>
void incr2(Int& target, Int amount);
 
template<typename Int, std::enable_if_t<std::is_unsigned<Int>::value, int> = 0>
void incr2(Int& target, Int amount);

```


Once again which is the easiest to write, and which is the easiest to review and understand at a glance?

> 再说一遍，哪个最容易写，哪个最易于复习和理解？


A strength of `std::enable_if` is how it plays with refactoring and API design. If `is_sizeable<Cont>::value` is meant to reflect whether `cont.size()` is valid then just using the expression as it appears for `size1` can be more concise, although that could depend on whether `is_sizeable` would be used in several places or not. Contrast that with `std::is_signed` which reflects its intention much more clearly than when its implementation leaks into the declaration of `incr1`.

> “std:：enable_if”的优势在于它如何处理重构和API设计。如果“is_sizeable＜Cont＞：：value”旨在反映“Cont.size（）”是否有效，那么只使用“size1”的表达式会更简洁，尽管这可能取决于“is_size”是否会在多个地方使用。与“std:：is_signed”形成对比，后者比其实现泄漏到“incr1”的声明中更清楚地反映了其意图。



## is_detected


To generalize type_trait creation:based on SFINAE
there are experimental traits `detected_or`, `detected_t`, `is_detected`.


With template parameters `typename Default`, `template <typename...> Op` and `typename ... Args`:

> 带有模板参数`typenameDefault`，`template<typename…>操作`and`typename。。。参数`:


- `is_detected`: alias of `std::true_type` or `std::false_type` depending of the validity of `Op<Args...>`

> -`is_detected`：`std:：true_type`或` std:：false_type`的别名，具体取决于`Op<Args…>的有效性`
- `detected_t`: alias of `Op<Args...>` or `nonesuch` depending of validity of `Op<Args...>`.

- `detected_or`: alias of a struct with `value_t` which is `is_detected`, and `type` which is `Op<Args...>` or `Default` depending of validity of `Op<Args...>`

> -“detected_or”：具有“value_t”和“type”的结构的别名，前者为“is_detected”，后者为“Op<Args…>`或“默认”，取决于“Op＜Args…＞”的有效性`

which can be implemented using `std::void_t` for SFINAE as following:

```cpp
namespace detail {
    template <class Default, class AlwaysVoid,
              template<class...> class Op, class... Args>
    struct detector
    {
        using value_t = std::false_type;
        using type = Default;
    };

    template <class Default, template<class...> class Op, class... Args>
    struct detector<Default, std::void_t<Op<Args...>>, Op, Args...>
    {
        using value_t = std::true_type;
        using type = Op<Args...>;
    };

} // namespace detail

// special type to indicate detection failure
struct nonesuch {
    nonesuch() = delete;
    ~nonesuch() = delete;
    nonesuch(nonesuch const&) = delete;
    void operator=(nonesuch const&) = delete;
};

template <template<class...> class Op, class... Args>
using is_detected =
    typename detail::detector<nonesuch, void, Op, Args...>::value_t;

template <template<class...> class Op, class... Args>
using detected_t = typename detail::detector<nonesuch, void, Op, Args...>::type;

template <class Default, template<class...> class Op, class... Args>
using detected_or = detail::detector<Default, void, Op, Args...>;

```

Traits to detect presence of method can then be simply implemented:

```cpp
typename <typename T, typename ...Ts>
using foo_type = decltype(std::declval<T>().foo(std::declval<Ts>()...));

struct C1 {};

struct C2 {
    int foo(char) const;
};

template <typename T>
using has_foo_char = is_detected<foo_type, T, char>;

static_assert(!has_foo_char<C1>::value, "Unexpected");
static_assert(has_foo_char<C2>::value, "Unexpected");

static_assert(std::is_same<int, detected_t<foo_type, C2, char>>::value,
              "Unexpected");

static_assert(std::is_same<void, // Default
                           detected_or<void, foo_type, C1, char>>::value,
              "Unexpected");
static_assert(std::is_same<int, detected_or<void, foo_type, C2, char>>::value,
              "Unexpected");

```



## Overload resolution with a large number of options


If you need to select between several options,
enabling just one via `enable_if<>` can be quite cumbersome,
since several conditions needs to be negated too.

The ordering between overloads can instead be selected using
inheritance, i.e. tag dispatch.

Instead of testing for the thing that needs to be well-formed, and also

testing the negation of all the other versions conditions, we instead test

> 测试所有其他版本条件的否定，我们测试

just for what we need, preferably in a `decltype` in a trailing return. <br>

> 只是为了满足我们的需要，最好是在尾部返回中的“decltype”中<br>

This might leave several option well formed, we differentiate between those

> 这可能会留下几个良好的选择，我们区分
using 'tags', similar to iterator-trait tags (`random_access_tag` et al).

This works because a direct match is better that a base class, which is better

> 这是因为直接匹配比基类更好，基类更好
that a base class of a base class, etc.

```cpp
#include <algorithm>
#include <iterator>

namespace detail
{
    // this gives us infinite types, that inherit from each other
    template<std::size_t N>
    struct pick : pick<N-1> {};
    template<>
    struct pick<0> {};

    // the overload we want to be preferred have a higher N in pick<N>
    // this is the first helper template function
    template<typename T>
    auto stable_sort(T& t, pick<2>)
        -> decltype( t.stable_sort(), void() )
    {
        // if the container have a member stable_sort, use that
        t.stable_sort();
    }

    // this helper will be second best match
    template<typename T>
    auto stable_sort(T& t, pick<1>)
        -> decltype( t.sort(), void() )
    {
        // if the container have a member sort, but no member stable_sort
        // it's customary that the sort member is stable
        t.sort();
    }

    // this helper will be picked last
    template<typename T>
    auto stable_sort(T& t, pick<0>)
        -> decltype( std::stable_sort(std::begin(t), std::end(t)), void() )
    {
        // the container have neither a member sort, nor member stable_sort
        std::stable_sort(std::begin(t), std::end(t));
    }

}

// this is the function the user calls. it will dispatch the call
// to the correct implementation with the help of 'tags'.
template<typename T>
void stable_sort(T& t)
{
    // use an N that is higher that any used above.
    // this will pick the highest overload that is well formed.
    detail::stable_sort(t, detail::pick<10>{});
}

```

There are other methods commonly used to differentiate between overloads,

such as exact match being better than conversion, being better than ellipsis.

> 例如精确匹配优于转换、优于省略号。


However, tag-dispatch can extend to any number of choices, and is a bit more clear in intent.

> 然而，标记调度可以扩展到任意数量的选择，并且在意图上更清晰一些。



## trailing decltype in function templates



One of constraining function is to use trailing `decltype` to specify the return type:

> 约束函数之一是使用尾随的“decltype”来指定返回类型：

```cpp
namespace details {
   using std::to_string;

   // this one is constrained on being able to call to_string(T)
   template <class T>
   auto convert_to_string(T const& val, int )
       -> decltype(to_string(val))
   {
       return to_string(val);
   }

   // this one is unconstrained, but less preferred due to the ellipsis argument
   template <class T>
   std::string convert_to_string(T const& val, ... )
   {
       std::ostringstream oss;
       oss << val;
       return oss.str();
   }
}

template <class T>
std::string convert_to_string(T const& val)
{
    return details::convert_to_string(val, 0);
}

```


If I call `convert_to_string()` with an argument with which I can invoke `to_string()`, then I have two viable functions for `details::convert_to_string()`. The first is preferred since the conversion from `0` to `int` is a better implicit conversion sequence than the conversion from `0` to `...`

> 如果我用一个可以调用“to_string（）”的参数调用“convert_to_string（）”，那么我有两个可行的“details”函数：：convert_to_string（）。第一个是优选的，因为从“0”到“int”的转换是比从“0’到“…”的转换更好的隐式转换序列`


If I call `convert_to_string()` with an argument from which I cannot invoke `to_string()`, then the first function template instantiation leads to substitution failure (there is no `decltype(to_string(val))`). As a result, that candidate is removed from the overload set. The second function template is unconstrained, so it is selected and we instead go through `operator<<(std::ostream&, T)`. If that one is undefined, then we have a hard compile error with a template stack on the line `oss << val`.

> 如果我用一个不能调用“to_string（）”的参数调用“convert_to_string（）”，那么第一个函数模板实例化会导致替换失败（没有“decltype（to_string）”）。因此，该候选项将从重载集中删除。第二个函数模板是不受约束的，因此它被选中，我们改为执行“operator<<（std:：ostream&，T）”。如果这个是未定义的，那么我们在`oss<<val`行上的模板堆栈会出现硬编译错误。



## enable_if_all / enable_if_any


**Motivational example**


When you have a variadic template pack in the template parameters list, like in the following code snippet:

> 当您在模板参数列表中有一个可变模板包时，如以下代码片段所示：

```cpp
template<typename ...Args> void func(Args &&...args) { //... };

```


The standard library (prior to C++17) offers no direct way to write **enable_if** to impose SFINAE constraints on ****all of the parameters**** in `Args` or ****any of the parameters**** in `Args`. C++17 offers [`std::conjunction`](http://en.cppreference.com/w/cpp/types/conjunction) and [`std::disjunction`](http://en.cppreference.com/w/cpp/types/disjunction) which solve this problem. For example:

> 标准库（在C++17之前）没有提供直接的方式来编写**enable_if**，以对“Args”中的***所有参数或“Args’中的***任何参数施加SFINAE约束。C++17提供[`std:：连词`](http://en.cppreference.com/w/cpp/types/conjunction)和[`std:：析取`](http://en.cppreference.com/w/cpp/types/disjunction)解决了这个问题。例如：

```cpp
/// C++17: SFINAE constraints on all of the parameters in Args.
template<typename ...Args,
         std::enable_if_t<std::conjunction_v<custom_conditions_v<Args>...>>* = nullptr>
void func(Args &&...args) { //... };

/// C++17: SFINAE constraints on any of the parameters in Args.
template<typename ...Args,
         std::enable_if_t<std::disjunction_v<custom_conditions_v<Args>...>>* = nullptr>
void func(Args &&...args) { //... };

```


If you do not have C++17 available, there are several solutions to achieve these. One of them is to use a base-case class and **partial specializations**, as demonstrated in answers of this [question](http://stackoverflow.com/questions/26421104/how-do-i-enable-if-a-class-with-variadic-template-arguments).

> 如果您没有可用的C++17，有几种解决方案可以实现这些功能。其中之一是使用一个基类和**部分专业化**，如这个[问题]的答案所示(http://stackoverflow.com/questions/26421104/how-do-i-enable-if-a-class-with-variadic-template-arguments)。


Alternatively, one may also implement by hand the behavior of `std::conjunction` and `std::disjunction` in a rather straight-forward way. In the following example I'll demonstrate the implementations and combine them with `std::enable_if` to produce two alias: `enable_if_all` and `enable_if_any`, which do exactly what they are supposed to semantically. This may provide a more scalable solution.

> 或者，也可以用一种相当直接的方式手动实现“std:：连词”和“std：：析取”的行为。在下面的示例中，我将演示这些实现，并将它们与“std:：enable_if”组合以生成两个别名：“enable_if_all”和“enable_if _any”，这两个别名在语义上完全符合它们的要求。这可以提供更具可扩展性的解决方案。

**Implementation of** `enable_if_all` **and** `enable_if_any`


First let's emulate `std::conjunction` and `std::disjunction` using customized `seq_and` and `seq_or` respectively:

> 首先，让我们分别使用自定义的“seq_and”和“seq_or”来模拟“std:：concurrent”和“std：：disconnection”：

```cpp
/// Helper for prior to C++14.
template<bool B, class T, class F >
using conditional_t = typename std::conditional<B,T,F>::type;

/// Emulate C++17 std::conjunction.
template<bool...> struct seq_or: std::false_type {};
template<bool...> struct seq_and: std::true_type {};

template<bool B1, bool... Bs>
struct seq_or<B1,Bs...>: 
  conditional_t<B1,std::true_type,seq_or<Bs...>> {};

template<bool B1, bool... Bs>
struct seq_and<B1,Bs...>:
  conditional_t<B1,seq_and<Bs...>,std::false_type> {};  

```

Then the implementation is quite straight-forward:

```cpp
template<bool... Bs>
using enable_if_any = std::enable_if<seq_or<Bs...>::value>;

template<bool... Bs>
using enable_if_all = std::enable_if<seq_and<Bs...>::value>;

```

Eventually some helpers:

```cpp
template<bool... Bs>
using enable_if_any_t = typename enable_if_any<Bs...>::type;

template<bool... Bs>
using enable_if_all_t = typename enable_if_all<Bs...>::type;

```

**Usage**

The usage is also straight-forward:

```

   /// SFINAE constraints on all of the parameters in Args.
    template<typename ...Args,
             enable_if_all_t<custom_conditions_v<Args>...>* = nullptr>
    void func(Args &&...args) { //... };

    /// SFINAE constraints on any of the parameters in Args.
    template<typename ...Args,
             enable_if_any_t<custom_conditions_v<Args>...>* = nullptr>
    void func(Args &&...args) { //... };

```

