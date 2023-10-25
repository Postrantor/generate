---
tip: translate by baidu@2023-10-25 08:32:00
---
---

metaTitle: "C++ | static_assert"
description: "static_assert"
----------------------------

# static_assert

## static_assert

Assertations mean that a condition should be checked and if it's false, it's an error. For `static_assert()`, this is done compile-time.

> 断言意味着应该检查一个条件，如果它是 false，那就是一个错误。对于“static_assert（）”，这是在编译时完成的。

```cpp
template<typename T>
T mul10(const T t)
{
    static_assert( std::is_integral<T>::value, "mul10() only works for integral types" );
    return (t << 3) + (t << 1);
}

```

A `static_assert()` has a mandatory first parameter, the condition, that is a bool constexpr. It **might** have a second parameter, the message, that is a string literal. From C++17, the second parameter is optional; before that, it's mandatory.

> “static_assert（）”有一个强制性的第一个参数 condition，它是布尔 constexpr。它**可能有第二个参数 message，它是一个字符串文字。在 C++17 中，第二个参数是可选的；在此之前，它是强制性的。

```cpp
template<typename T>
T mul10(const T t)
{
    static_assert(std::is_integral<T>::value);
    return (t << 3) + (t << 1);
}

```

It is used when:

- In general, a verification at compile-time is required on some type on constexpr value
- A template function needs to verify certain properties of a type passed to it

<li>One wants to write test cases for:
<ul>
- template metafunctions
- constexpr functions
- macro metaprogramming

Note that `static_assert()` does not participate in [SFINAE](https://stackoverflow.com/documentation/c%2B%2B/1169/sfinae-substitution-failure-is-not-an-error): thus, when additional overloads / specializations are possible, one should not use it instead of template metaprogramming techniques (like `std::enable_if<>`). It might be used in template code when the expected overload / specialization is already found, but further verifications are required. In such cases, it might provide more concrete error message(s) than relying on SFINAE for this.

> 请注意，“static_assert（）”不参与[SFINAE]([https://stackoverflow.com/documentation/c%2b%2b/1169/sfinae](https://stackoverflow.com/documentation/c%2B%2B/1169/sfinae)-替换失败不是错误）：因此，当可能出现额外的重载/专业化时，不应该使用它来代替模板元编程技术（如“std:：enable_if<>'”）。当已经找到预期的重载/专用化时，它可能会在模板代码中使用，但还需要进一步的验证。在这种情况下，它可能会提供比依赖 SFINAE 更具体的错误消息。

#### Syntax

- static_assert( **bool_constexpr**, **message** )
- static_assert( **bool_constexpr** ) /* Since C++17 */

#### Parameters

| Parameter          | Details                                               |  |  |  |  |  |  |  |  |
| ------------------ | ----------------------------------------------------- | - | - | - | - | - | - | - | - |
| **bool_constexpr** | Expression to check                                   |  |  |  |  |  |  |  |  |
| **message**        | Message to print when **bool_constexpr** is **false** |  |  |  |  |  |  |  |  |

#### Remarks

Unlike [runtime assertions](http://en.cppreference.com/w/cpp/error/assert), static assertions are checked at compile-time and are also enforced when compiling optimized builds.

> 与[运行时断言]不同([http://en.cppreference.com/w/cpp/error/assert](http://en.cppreference.com/w/cpp/error/assert))，静态断言在编译时进行检查，在编译优化的构建时也会强制执行。
