---
tip: translate by baidu@2023-10-25 08:20:32
---
---
metaTitle: "C++ | Attributes"
description: "[[fallthrough]], [[deprecated]] and [[deprecated(reason)]], [[nodiscard]], [[noreturn]], [[maybe_unused]]"
---

# Attributes




## [[fallthrough]]



Whenever a `case` is ended in a `switch`, the code of the next case will get executed. This last one can be prevented by using the ´break` statement.

> 每当“case”以“switch”结束时，就会执行下一个case的代码。最后一个可以通过使用“break”语句来防止。

As this so-called fallthrough behavior can introduce bugs when not intended, several compilers and static analyzers give a warning on this.

> 由于这种所谓的失败行为可能会在无意中引入错误，因此一些编译器和静态分析器会对此发出警告。


From C++17 on, a standard attribute was introduced to indicate that the warning is not needed when the code is meant to fall through.

> 从C++17开始，引入了一个标准属性来指示当代码即将失败时不需要警告。

Compilers can safely give warnings when a case is ended without `break` or `[[fallthrough]]` and has at least one statement.

> 当一个事例结束时不带“break”或“[[fallsthrough]]”并且至少有一条语句时，编译器可以安全地发出警告。

```cpp
switch(input) {
    case 2011:
    case 2014:
    case 2017:
        std::cout << "Using modern C++" << std::endl;
        [[fallthrough]]; // > No warning
    case 1998:
    case 2003:
        standard = input;
}

```


See [the proposal](https://isocpp.org/files/papers/P0188R0.pdf) for more detailed examples on how `[[fallthrough]]` can be used.

> 参见[建议书](https://isocpp.org/files/papers/P0188R0.pdf)有关如何使用“[[fallsthrough]]”的更详细示例。



## [[deprecated]] and [[deprecated("reason")]]



C++14 introduced a standard way of deprecating functions via attributes. `[[deprecated]]` can be used to indicate that a function is deprecated. `[[deprecated("reason")]]` allows adding a specific reason which can be shown by the compiler.

> C++14引入了一种通过属性来弃用函数的标准方法`[[deprecated]]`可用于表示函数已被弃用`[[deprecated（“reason”）]]`允许添加编译器可以显示的特定原因。

```cpp
void function(std::unique_ptr<A> &&a);

// Provides specific message which helps other programmers fixing there code
[[deprecated("Use the variant with unique_ptr instead, this function will be removed in the next release")]]
void function(std::auto_ptr<A> a);

// No message, will result in generic warning if called.
[[deprecated]]
void function(A *a);

```

This attribute may be applied to:

- the declaration of a class
- a typedef-name
- a variable
- a non-static data member
- a function
- an enumeration
- a template specialization


(ref. [c++14 standard draft](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2013/n3797.pdf): 7.6.5  Deprecated attribute)

> （参考[c++14标准草案](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2013/n3797.pdf)：7.6.5不推荐的属性）



## [[nodiscard]]



The `[[nodiscard]]` attribute can be used to indicate that the return value of a function shouldn't be ignored when you do a function call. If the return value is ignored, the compiler should give a warning on this. The attribute can be added to:

> “[[nodiscard]]”属性可用于指示在执行函数调用时不应忽略函数的返回值。如果忽略返回值，编译器应该对此发出警告。该属性可以添加到：

- A function definition
- A type


Adding the attribute to a type has the same behaviour as adding the attribute to every single function which returns this type.

> 将属性添加到类型的行为与将该属性添加到每个返回该类型的函数的行为相同。

```cpp
template<typename Function>
[[nodiscard]] Finally<std::decay_t<Function>> onExit(Function &&f);

void f(int &i) {
    assert(i == 0);                    // Just to make comments clear!
    ++i;                               // i == 1
    auto exit1 = onExit([&i]{ --i; }); // Reduce by 1 on exiting f()
    ++i;                               // i == 2
    onExit([&i]{ --i; });              // BUG: Reducing by 1 directly
                                       //      Compiler warning expected
    std::cout << i << std::end;        // Expected: 2, Real: 1
}

```


See [the proposal](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2015/p0068r0.pdf) for more detailed examples on how `[[nodiscard]]` can be used.

> 参见[建议书](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2015/p0068r0.pdf)有关如何使用“[[nodiscard]]”的更详细示例。


**Note:** The implementation details of `Finally`/`onExit` are omitted in the example, see [Finally/ScopeExit](http://stackoverflow.com/documentation/c%2b%2b/1320/raii-resource-acquisition-is-initialization/4551/finally-scopeexit).

> **注意：**示例中省略了`Finally `/`onExit`的实现细节，请参见[Finally/ScopeExit](http://stackoverflow.com/documentation/c%2b%2b/1320/raii-资源获取是初始化/4551/最终scopeexit）。



## [[noreturn]]


C++11 introduced the `[[noreturn]]` attribute.

It can be used for a function to indicate that the function does not return to the caller by either executing a **return** statement, or by reaching the end if it's body (it is important to note that this does not apply to `void` functions, since they do return to the caller, they just do not return any value). Such a function may end by calling `std::terminate` or `std::exit`, or by throwing an exception. It is also worth noting that such a function can return by executing `longjmp`.

> 它可以用于函数，通过执行**return**语句，或者如果函数是body，则通过到达末尾来指示函数不会返回给调用者（需要注意的是，这不适用于“void”函数，因为它们确实返回给调用者，只是不返回任何值）。这样的函数可以通过调用“std:：terminate”或“std::：exit”或抛出异常来结束。同样值得注意的是，这样的函数可以通过执行“longjmp”返回。


For instance, the function below will always either throw an exception or call `std::terminate`, so it is a good candidate for `[[noreturn]]`:

> 例如，下面的函数总是抛出异常或调用“std:：terminate”，因此它是“[[noreturn]]”的一个很好的候选者：

```cpp
[[noreturn]] void ownAssertFailureHandler(std::string message) {
    std::cerr << message << std::endl;
    if (THROW_EXCEPTION_ON_ASSERT)
        throw AssertException(std::move(message));
    std::terminate();
}

```


This kind of functionality allows the compiler to end a function without a return statement if it knows the code will never be executed. Here, because the call to `ownAssertFailureHandler` (defined above) in the code below will never return, the compiler does not need to add code below that call:

> 这种功能允许编译器在知道代码永远不会被执行的情况下，在没有返回语句的情况下结束函数。这里，因为下面代码中对“ownAssertFailureHandler”（上面定义的）的调用永远不会返回，所以编译器不需要在该调用下面添加代码：

```cpp
std::vector<int> createSequence(int end) {
    if (end > 0) {
        std::vector<int> sequence;
        sequence.reserve(end+1);
        for (int i = 0; i <= end; ++i)
            sequence.push_back(i);
        return sequence;
    }
    ownAssertFailureHandler("Negative number passed to createSequence()"s);
    // return std::vector<int>{}; //< Not needed because of [[noreturn]]
}

```


It is undefined behavior if the function will actually return, so the following is not allowed:

> 如果函数实际返回，则这是未定义的行为，因此不允许出现以下情况：

```cpp
[[noreturn]] void assertPositive(int number) {
    if (number >= 0)
        return;
    else
        ownAssertFailureHandler("Positive number expected"s); //< [[noreturn]]
}

```


Note that the `[[noreturn]]` is mostly used in void functions. However, this is not a requirement, allowing the functions to be used in generic programming:

> 请注意，“[[noreturn]]”主要用于void函数中。然而，这不是一项要求，允许在通用编程中使用函数：

```cpp
template<class InconsistencyHandler>
double fortyTwoDivideBy(int i) {
    if (i == 0)
         i = InconsistencyHandler::correct(i);
    return 42. / i;
}

struct InconsistencyThrower {
    static [[noreturn]] int correct(int i) { ownAssertFailureHandler("Unknown inconsistency"s); }
}

struct InconsistencyChangeToOne {
    static int correct(int i) { return 1; }
}

double fortyTwo = fortyTwoDivideBy<InconsistencyChangeToOne>(0);
double unreachable = fortyTwoDivideBy<InconsistencyThrower>(0);

```

The following standard library functions have this attribute:

- std::abort
- std::exit
- std::quick_exit
- std::unexpected
- std::terminate
- std::rethrow_exception
- std::throw_with_nested
- std::nested_exception::rethrow_nested



## [[maybe_unused]]



The `[[maybe_unused]]` attribute is created for indicating in code that certain logic might not be used. This if often linked to preprocessor conditions where this might be used or might not be used. As compilers can give warnings on unused variables, this is a way of suppressing them by indicating intent.

> 创建“[[maybe_unused]]”属性是为了在代码中指示可能不使用某些逻辑。这通常与预处理器条件有关，在预处理器条件下可能使用或不使用。由于编译器可以对未使用的变量发出警告，因此这是一种通过指示意图来抑制它们的方法。


A typical example of variables which are needed in debug builds while unneeded in production are return values indicating success. In the debug builds, the condition should be asserted, though in production these asserts have been removed.

> 调试构建中需要而生产中不需要的变量的一个典型示例是指示成功的返回值。在调试构建中，应该断言条件，尽管在生产中这些断言已被删除。

```cpp
[[maybe_unused]] auto mapInsertResult = configuration.emplace("LicenseInfo", stringifiedLicenseInfo);
assert(mapInsertResult.second); // We only get called during startup, so we can't be in the map

```


A more complex example are different kind of helper functions which are in an unnamed namespace. If these functions aren't used during compilation, a compiler might give a warning on them. Ideally you would like to guard them with the same preprocessor tags as the caller, though as this might become complex the `[[maybe_unused]]` attribute is a more maintainable alternative.

> 一个更复杂的例子是位于未命名名称空间中的不同类型的辅助函数。如果在编译过程中没有使用这些函数，编译器可能会对它们发出警告。理想情况下，您希望使用与调用方相同的预处理器标记来保护它们，尽管这可能会变得复杂，但“[[maybe_unused]]”属性是一个更易于维护的替代选项。

```cpp
namespace {
    [[maybe_unused]] std::string createWindowsConfigFilePath(const std::string &relativePath);
    // TODO: Reuse this on BSD, MAC ...
    [[maybe_unused]] std::string createLinuxConfigFilePath(const std::string &relativePath);
}

std::string createConfigFilePath(const std::string &relativePath) {
#if OS == "WINDOWS"
      return createWindowsConfigFilePath(relativePath);
#elif OS == "LINUX"
      return createLinuxConfigFilePath(relativePath);
#else
#error "OS is not yet supported"
#endif
}

```


See [the proposal](https://isocpp.org/files/papers/P0212R0.pdf) for more detailed examples on how `[[maybe_unused]]` can be used.

> 参见[建议书](https://isocpp.org/files/papers/P0212R0.pdf)有关如何使用“[[maybe_unused]]”的更详细示例。



#### Syntax


<li>
[[details]]: Simple no-argument attribute
</li>
<li>
[[details(arguments)]]: Attribute with arguments
</li>
<li>
__attribute(details): Non-standard GCC/Clang/IBM specific
</li>
<li>
__declspec(details): Non-standard MSVC specific
</li>

