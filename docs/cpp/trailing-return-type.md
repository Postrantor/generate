---
tip: translate by baidu@2023-10-25 08:34:55
---
---
metaTitle: "C++ | Trailing return type"
description: "Avoid qualifying a nested type name, Lambda expressions"
---

# Trailing return type



## Avoid qualifying a nested type name


```cpp
class ClassWithAReallyLongName {
  public:
    class Iterator { /* ... */ };
    Iterator end();
};

```

Defining the member `end` with a trailing return type:

```cpp
auto ClassWithAReallyLongName::end() -> Iterator { return Iterator(); }

```

Defining the member `end` without a trailing return type:

```cpp
ClassWithAReallyLongName::Iterator ClassWithAReallyLongName::end() { return Iterator(); }

```


The trailing return type is looked up in the scope of the class, while a leading return type is looked up in the enclosing namespace scope and can therefore require "redundant" qualification.

> 尾部返回类型在类的作用域中查找，而前导返回类型在封闭的命名空间作用域中查询，因此可能需要“冗余”限定。



## Lambda expressions



A lambda can **only** have a trailing return type; the leading return type syntax is not applicable to lambdas. Note that in many cases it is not necessary to specify a return type for a lambda at all.

> lambda只能**具有尾随返回类型；前导返回类型语法不适用于lambdas。请注意，在许多情况下，根本不需要为lambda指定返回类型。

```cpp
struct Base {};
struct Derived1 : Base {};
struct Derived2 : Base {};
auto lambda = [](bool b) -> Base* { if (b) return new Derived1; else return new Derived2; };
// ill-formed: auto lambda = Base* [](bool b) { ... };

```



#### Syntax



- **function_name** ( [**function_args**] ) [**function_attributes**] [**function_qualifiers**] `->` **trailing-return-type** [**requires_clause**]

> -**函数名**（[***function_args**]）[***function_attributes**][***function_qualifiers**]`->`**尾随返回类型**[***requires_clause**]



#### Remarks



The above syntax shows a full function declaration using a trailing type, where square brackets indicate an optional part of the function declaration (like the argument list if a no-arg function).

> 上面的语法显示了一个使用尾随类型的完整函数声明，其中方括号表示函数声明的可选部分（如无arg函数的参数列表）。


Additionally, the syntax of the trailing return type prohibits defining a class, union, or enum type inside a trailing return type (note that this is not allowed in a leading return type either). Other than that, types can be spelled the same way after the `->` as they would be elsewhere.

> 此外，尾部返回类型的语法禁止在尾部返回类型内定义类、并集或枚举类型（请注意，在前导返回类型中也不允许这样做）。除此之外，类型在“->”后面的拼写方式与在其他地方的拼写方式相同。

