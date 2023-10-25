---
tip: translate by baidu@2023-10-25 08:31:10
---
---

metaTitle: "C++ | Side by Side Comparisons of classic C++ examples solved via C++ vs C++11 vs C++14 vs C++17"

> metaTitle:“C++|通过C++与C++11与C++14与C++17解决的经典C++示例的并排比较”
description: "Looping through a container"
---

# Side by Side Comparisons of classic C++ examples solved via C++ vs C++11 vs C++14 vs C++17



## Looping through a container



In C++, looping through a sequence container `c` can be done using indexes as follows:

> 在C++中，可以使用如下索引在序列容器“C”中进行循环：

```cpp
for(size_t i = 0; i < c.size(); ++i) c[i] = 0;

```


While simple, such writings are subject to common semantic errors, like wrong comparison operator, or wrong indexing variable:

> 虽然很简单，但这样的写作会遇到常见的语义错误，比如错误的比较运算符或错误的索引变量：

```cpp
for(size_t i = 0; i <= c.size(); ++j) c[i] = 0;
                     ^~~~~~~~~~~~~~^

```


Looping can also be achieved for all containers using iterators, with similar drawbacks:

> 使用迭代器也可以为所有容器实现循环，但有类似的缺点：

```cpp
for(iterator it = c.begin(); it != c.end(); ++it) (*it) = 0;

```


C++11 introduced range-based for loops and `auto` keyword, allowing the code to become:

> C++11引入了基于范围的循环和“auto”关键字，使代码变成：

```cpp
for(auto& x : c) x = 0;

```


Here the only parameters are the container `c`, and a variable `x` to hold the current value. This prevents the semantics errors previously pointed.

> 这里唯一的参数是容器“c”和保存当前值的变量“x”。这样可以防止前面指出的语义错误。


According to the C++11 standard, the underlying implementation is equivalent to:

> 根据C++11标准，底层实现等效于：

```cpp
for(auto begin = c.begin(), end = c.end(); begin != end; ++begin)
{
    // ...
}

```


In such implementation, the expression `auto begin = c.begin(), end = c.end();` forces `begin` and `end` to be of the same type, while `end` is never incremented, nor dereferenced. So the range-based for loop only works for containers defined by a pair iterator/iterator. The C++17 standard relaxes this constraint by changing the implementation to:

> 在这样的实现中，表达式`auto begin=c.begin（），end=c.end（）；`强制“begin”和“end”为同一类型，而“end”从不递增或取消引用。因此，基于范围的for循环只适用于由对迭代器/迭代器定义的容器。C++17标准通过将实现更改为：

```cpp
auto begin = c.begin();
auto end = c.end();
for(; begin != end; ++begin)
{
    // ...
}

```


Here `begin` and `end` are allowed to be of different types, as long as they can be compared for inequality. This allows to loop through more containers, e.g. a container defined by a pair iterator/sentinel.

> 在这里，“begin”和“end”可以是不同的类型，只要它们可以进行不平等的比较。这允许循环通过更多的容器，例如由迭代器/哨兵对定义的容器。

