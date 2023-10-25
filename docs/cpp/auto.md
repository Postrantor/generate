---
tip: translate by baidu@2023-10-25 08:20:39
---
---

metaTitle: "C++ | auto"
description: "Basic auto sample, auto and Expression Templates, auto, const, and references, Trailing return type, Generic lambda (C++14), auto and proxy objects"
------------------------------------------------------------------------------------------------------------------------------------------------------------------

# auto

## Basic auto sample

The keyword `auto` provides the auto-deduction of type of a variable.

It is especially convenient when dealing with long type names:

```cpp
std::map< std::string, std::shared_ptr< Widget > > table;
// C++98
std::map< std::string, std::shared_ptr< Widget > >::iterator i = table.find( "42" );
// C++11/14/17
auto j = table.find( "42" );

```

with [range-based for loops](http://stackoverflow.com/documentation/c%2B%2B/589/loops/1926/range-based-for):

> 带有[基于范围的循环]([http://stackoverflow.com/documentation/c%2B%2B/589/loops/1926/range](http://stackoverflow.com/documentation/c%2B%2B/589/loops/1926/range)-基于）：

```cpp
vector<int> v = {0, 1, 2, 3, 4, 5};
for(auto n: v)
    std::cout << n << ' ';

```

with [lambdas](http://stackoverflow.com/documentation/c%2B%2B/572/lambdas):

> 与[兰达斯](http://stackoverflow.com/documentation/c%2B%2B/572/lambdas)：

```cpp
auto f = [](){ std::cout << "lambda\n"; };
f();        

```

to avoid the repetition of the type:

```cpp
auto w = std::make_shared< Widget >();

```

to avoid surprising and unnecessary copies:

```cpp
auto myMap = std::map<int,float>();
myMap.emplace(1,3.14);

std::pair<int,float> const& firstPair2 = *myMap.begin();  // copy!
auto const& firstPair = *myMap.begin();  // no copy!

```

The reason for the copy is that the returned type is actually `std::pair<const int,float>`!

> 复制的原因是返回的类型实际上是“std:：pair＜const int，float＞”！

## auto and Expression Templates

`auto` can also cause problems where expression templates come into play:

```cpp
auto mult(int c) {
    return c * std::valarray<int>{1};
}

auto v = mult(3);
std::cout << v[0]; // some value that could be, but almost certainly is not, 3.

```

The reason is that `operator*` on `valarray` gives you a proxy object that refers to the `valarray` as a means of lazy evaluation. By using `auto`, you're creating a dangling reference. Instead of `mult` had returned a `std::valarray<int>`, then the code would definitely print 3.

> 原因是“valarray”上的“operator*”为您提供了一个代理对象，该对象引用“valarray”作为延迟求值的一种方法。通过使用“auto”，您正在创建一个悬空引用。如果“mult”没有返回“std:：valarray＜int＞”，那么代码肯定会打印 3。

## auto, const, and references

The `auto` keyword by itself represents a value type, similar to `int` or `char`. It can be modified with the `const` keyword and the `&` symbol to represent a const type or a reference type, respectively. These modifiers can be combined.

> “auto”关键字本身表示一种值类型，类似于“int”或“char”。可以使用“const”关键字和“&”符号对其进行修改，以分别表示 const 类型或引用类型。这些修改器可以组合使用。

In this example, `s` is a value type (its type will be inferred as `std::string`), so each iteration of the `for` loop **copies** a string from the vector into `s`.

> 在本例中，“s”是一个值类型（其类型将推断为“std:：string”），因此“for”循环**的每次迭代都将**一个字符串从向量复制到“s”中。

```cpp
std::vector<std::string> strings = { "stuff", "things", "misc" };
for(auto s : strings) {
    std::cout << s << std::endl;
}

```

If the body of the loop modifies `s` (such as by calling `s.append(" and stuff")`), only this copy will be modified, not the original member of `strings`.

> 如果循环的主体修改了“s”（例如通过调用“s.append（“and stuff”）”），则只会修改此副本，而不会修改“string”的原始成员。

On the other hand, if `s` is declared with `auto&` it will be a reference type (inferred to be `std::string&`), so on each iteration of the loop it will be assigned a **reference** to a string in the vector:

> 另一方面，如果用“auto&”声明“s”，它将是一个引用类型（推断为“std:：string&”），因此在循环的每次迭代中，它将被分配给向量中的字符串一个**引用**：

```cpp
for(auto& s : strings) {
    std::cout << s << std::endl;
}

```

In the body of this loop, modifications to `s` will directly affect the element of `strings` that it references.

> 在这个循环的主体中，对“s”的修改将直接影响它引用的“string”元素。

Finally, if `s` is declared `const auto&`, it will be a const reference type, meaning that on each iteration of the loop it will be assigned a **const reference** to a string in the vector:

> 最后，如果“s”被声明为“const auto&”，它将是一个 const 引用类型，这意味着在循环的每次迭代中，它将被分配给向量中的字符串一个 **const 引用**：

```cpp
for(const auto& s : strings) {
    std::cout << s << std::endl;
}

```

Within the body of this loop, `s` cannot be modified (i.e. no non-const methods can be called on it).

> 在这个循环的主体中，不能修改“s”（即不能对其调用任何非常数方法）。

When using `auto` with range-based `for` loops, it is generally good practice to use `const auto&` if the loop body will not modify the structure being looped over, since this avoids unnecessary copies.

> 当将“auto”与基于范围的“for”循环一起使用时，如果循环体不会修改正在循环的结构，则通常最好使用“const auto&”，因为这样可以避免不必要的复制。

## Trailing return type

`auto` is used in the syntax for trailing return type:

```cpp
auto main() -> int {}

```

which is equivalent to

```cpp
int main() {}

```

Mostly useful combined with `decltype` to use parameters instead of `std::declval<T>`:

> 与“decltype”组合使用参数而不是“std:：declval＜T＞”非常有用：

```cpp
template <typename T1, typename T2>
auto Add(const T1& lhs, const T2& rhs) -> decltype(lhs + rhs) { return lhs + rhs; }

```

## Generic lambda (C++14)

C++14 allows to use `auto` in lambda argument

```cpp
auto print = [](const auto& arg) { std::cout << arg << std::endl; };

print(42);
print("hello world");

```

That lambda is mostly equivalent to

```cpp
struct lambda {
    template <typename T>
    auto operator ()(const T& arg) const {
        std::cout << arg << std::endl;
    }
};

```

and then

```cpp
lambda print;

print(42);
print("hello world");

```

## auto and proxy objects

Sometimes `auto` may behave not quite as was expected by a programmer.  It type deduces the expression, even when type deduction is not the right thing to do.

> 有时“auto”的行为可能与程序员预期的不太一样。它对表达式进行类型推导，即使类型推导不是正确的做法。

As an example, when proxy objects are used in the code:

```cpp
std::vector<bool> flags{true, true, false};
auto flag = flags[0];
flags.push_back(true);

```

Here `flag` would be not `bool`, but `std::vector<bool>::reference`, since for `bool` specialization of template `vector` the `operator []` returns a proxy object with conversion operator `operator bool` defined.

> 这里的“flag”不是“bool”，而是“std:：vector＜bool＞：reference”，因为对于模板“vector”的“bool’specialization”，“operator[]”返回一个定义了转换运算符“operator bool”的代理对象。

When `flags.push_back(true)` modifies the container, this pseudo-reference could end up dangling, referring to an element that no longer exists.

> 当“flags.push_back（true）”修改容器时，这个伪引用可能会挂起，引用一个不再存在的元素。

It also makes the next situation possible:

```cpp
void foo(bool b);

std::vector<bool> getFlags();

auto flag = getFlags()[5];
foo(flag);

```

The `vector` is discarded immediately, so `flag` is a pseudo-reference to an element that has been discarded.  The call to `foo` invokes undefined behavior.

> “vector”会立即被丢弃，因此“flag”是对已被丢弃的元素的伪引用。对“foo”的调用调用未定义的行为。

In cases like this you can declare a variable with `auto` and initialize it by casting to the type you want to be deduced:

> 在这种情况下，您可以用“auto”声明一个变量，并通过强制转换为要推导的类型来初始化它：

```cpp
auto flag = static_cast<bool>(getFlags()[5]);

```

but at that point, simply replacing `auto` with `bool` makes more sense.

Another case where proxy objects can cause problems is  [expression templates](https://stackoverflow.com/documentation/c%2B%2B/3404/expression-templates#t=201608151452546656817).  In that case, the templates are sometimes not designed to last beyond the current full-expression for efficiency sake, and using the proxy object on the next causes undefined behavior.

> 代理对象可能导致问题的另一种情况是[表达式模板]([https://stackoverflow.com/documentation/c%2b%2b/3404/expression](https://stackoverflow.com/documentation/c%2B%2B/3404/expression)-模板#t=2016081514524656817）。在这种情况下，为了提高效率，模板有时不会设计为超过当前的完整表达式，并且在下一个表达式上使用代理对象会导致未定义的行为。

#### Remarks

The keyword `auto` is a typename that represents an automatically-deduced type.

> 关键字“auto”是一个表示自动推导出的类型的类型名。

It was already a reserved keyword in C++98, inherited from C. In old versions of C++, it could be used to explicitly state that a variable has automatic storage duration:

> 它已经是 C++98 中的一个保留关键字，从 C 继承而来。在 C++ 的旧版本中，它可以用于显式声明变量具有自动存储持续时间：

```cpp
int main()
{
  auto int i = 5; // removing auto has no effect
}

```

That old meaning is now removed.
