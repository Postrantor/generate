---
tip: translate by baidu@2023-10-25 08:25:17
---
---

metaTitle: "C++ | Function Template Overloading"
description: "What is a valid function template overloading?"
-------------------------------------------------------------

# Function Template Overloading

## What is a valid function template overloading?

A function template can be overloaded under the rules for non-template function overloading (same name, but different parameter types) and in addition to that, the overloading is valid if

> 函数模板可以根据非模板函数重载规则（名称相同，但参数类型不同）进行重载，除此之外，如果

- The return type is different, or
- The template parameter list is different, except for the naming of parameters and the presence of default arguments (they are not part of the signature)

> -模板参数列表不同，除了参数的命名和默认参数的存在（它们不是签名的一部分）

For a normal function, comparing two parameter types is is easy for the compiler, since it has all informat. But a type within a template may not be determined yet. Therefore, the rule for when two parameter types are equal is approximative here and says that the non-depependend types and values need to match and the spelling of dependent types and expressions needs to be the same (more precisely, they need to conform to the so-called ODR-rules), except that template parameters may be renamed. However, if under such different spellings, two values within the types are deemed different, but will always instantiate to the same values, the overloading is invalid, but no diagnostic is required from the compiler.

> 对于普通函数来说，比较两种参数类型对编译器来说很容易，因为它具有所有信息。但是模板中的类型可能还没有确定。因此，两个参数类型相等时的规则在这里是近似的，并表示非依赖类型和值需要匹配，依赖类型和表达式的拼写需要相同（更准确地说，它们需要符合所谓的 ODR 规则），只是模板参数可以重命名。然而，如果在这种不同的拼写下，类型中的两个值被认为是不同的，但总是实例化为相同的值，则重载是无效的，但不需要编译器进行诊断。

```cpp
template<typename T>
void f(T*) { }

template<typename T>
void f(T) { }

```

This is a valid overload, as "T" and "T*" are different spellings. But the following is invalid, with no diagnostic required

> 这是一个有效的重载，因为“T”和“T*”是不同的拼写。但以下内容无效，无需诊断

```cpp
template<typename T>
void f(T (*x)[sizeof(T) + sizeof(T)]) { }

template<typename T>
void f(T (*x)[2 * sizeof(T)]) { }

```

#### Remarks

- A normal function is never related to a function template, despite same name, same type.
- A normal function call and a generated function template call are different even if they share the same name, same return type and same argument list

> -普通函数调用和生成的函数模板调用是不同的，即使它们共享相同的名称、相同的返回类型和相同的参数列表
