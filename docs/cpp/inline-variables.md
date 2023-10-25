---
tip: translate by baidu@2023-10-25 08:26:09
---
---

metaTitle: "C++ | Inline variables"
description: "Defining a static data member in the class definition"
--------------------------------------------------------------------

# Inline variables

An inline variable is allowed to be defined in multiple translation units without violating the [One Definition Rule](http://stackoverflow.com/documentation/c%2B%2B/4907/one-definition-rule-odr). If it is multiply defined, the linker will merge all definitions into a single object in the final program.

> 允许在不违反[一个定义规则]的情况下，以多个翻译单元定义内联变量([http://stackoverflow.com/documentation/c%2b%2b/4907/one](http://stackoverflow.com/documentation/c%2B%2B/4907/one)-定义规则 odr）。如果它是多重定义的，链接器将在最终程序中将所有定义合并到一个对象中。

## Defining a static data member in the class definition

A static data member of the class may be fully defined within the class definition if it is declared `inline`. For example, the following class may be defined in a header. Prior to C++17, it would have been necessary to provide a `.cpp` file to contain the definition of `Foo::num_instances` so that it would be defined only once, but in C++17 the multiple definitions of the inline variable `Foo::num_instances` all refer to the same `int` object.

> 如果类的静态数据成员声明为“inline”，则它可以在类定义中完全定义。例如，可以在标头中定义以下类。在 C++17 之前，必须提供一个“.cpp”文件来包含“Foo:：num_instances”的定义，这样它只能定义一次，但在 C++17 中，内联变量“Foo::num_instances”的多个定义都指向同一个“int”对象。

```cpp
// warning: not thread-safe...
class Foo {
  public:
    Foo() { ++num_instances; }
    ~Foo() { --num_instances; }
    inline static int num_instances = 0;
};

```

As a special case, a `constexpr` static data member is implicitly inline.

```cpp
class MyString {
  public:
    MyString() { /* ... */ }
    // ...
    static constexpr int max_size = INT_MAX / 2;
};
// in C++14, this definition was required in a single translation unit:
// constexpr int MyString::max_size;

```
