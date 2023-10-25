---
tip: translate by baidu@2023-10-25 08:36:01
---
---

metaTitle: "C++ | Unnamed types"
description: "Unnamed classes, As a type alias, Anonymous members, Anonymous Union"
-----------------------------------------------------------------------------------

# Unnamed types

## Unnamed classes

Unlike a named class or struct, unnamed classes and structs must be instantiated where they are defined, and cannot have constructors or destructors.

> 与命名类或结构不同，未命名类和结构必须在定义它们的地方实例化，并且不能有构造函数或析构函数。

```cpp
struct {
    int foo;
    double bar;
} foobar;

foobar.foo = 5;
foobar.bar = 4.0;

class {
    int baz;
public:
    int buzz;
    
    void setBaz(int v) {
        baz = v;
    }
} barbar;

barbar.setBaz(15);
barbar.buzz = 2;

```

## As a type alias

Unnamed class types may also be used when creating type aliases, i.e. via `typedef` and `using`:

> 创建类型别名时也可以使用未命名的类类型，即通过“typedef”和“using”：

```cpp
using vec2d = struct {
    float x;
    float y;
};

```

```cpp
typedef struct {
    float x;
    float y;
} vec2d;

```

```cpp
vec2d pt;
pt.x = 4.f;
pt.y = 3.f;

```

## Anonymous members

As a non-standard extension to C++, common compilers allow the use of classes as anonymous members.

> 作为 C++ 的非标准扩展，通用编译器允许使用类作为匿名成员。

```cpp
struct Example {
    struct {
        int inner_b;
    };
    
    int outer_b;
    
    //The anonymous struct's members are accessed as if members of the parent struct
    Example() : inner_b(2), outer_b(4) {
        inner_b = outer_b + 2;
    }
};

Example ex;

//The same holds true for external code referencing the struct
ex.inner_b -= ex.outer_b;

```

## Anonymous Union

Member names of an anonymous union belong to the scope of the union declaration an must be distinct to all other names of this scope. The example here has the same construction as example [Anonymous Members](http://stackoverflow.com/documentation/c%2B%2B/2704/unnamed-types/9055/anonymous-members#t=201609090658106572273) using "struct" but is standard conform.

> 匿名联合的成员名称属于联合声明的范围，并且必须与该范围的所有其他名称不同。这里的示例与示例[匿名成员]具有相同的结构([http://stackoverflow.com/documentation/c%2B%2B/2704/unnamed-types/9055/](http://stackoverflow.com/documentation/c%2B%2B/2704/unnamed-types/9055/)匿名成员#t=201609090658106572273），但符合标准。

```cpp
struct Sample {
    union {
        int a;
        int b;
    };
    int c;
};
int main()
{
  Sample sa;
  sa.a =3;
  sa.b =4;
  sa.c =5;
}

```
