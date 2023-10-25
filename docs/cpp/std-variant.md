---
tip: translate by baidu@2023-10-25 08:33:08
---
---
metaTitle: "C++ | std::variant"
description: "Create pseudo-method pointers, Basic std::variant use, Constructing a `std::variant`"
---

# std::variant



## Create pseudo-method pointers


This is an advanced example.

You can use variant for light weight type erasure.

```cpp
template<class F>
struct pseudo_method {
  F f;
  // enable C++17 class type deduction:
  pseudo_method( F&& fin ):f(std::move(fin)) {}

  // Koenig lookup operator->*, as this is a pseudo-method it is appropriate:
  template<class Variant> // maybe add SFINAE test that LHS is actually a variant.
  friend decltype(auto) operator->*( Variant&& var, pseudo_method const& method ) {
    // var->*method returns a lambda that perfect forwards a function call,
    // behaving like a method pointer basically:
    return [&](auto&&...args)->decltype(auto) {
      // use visit to get the type of the variant:
      return std::visit(
        [&](auto&& self)->decltype(auto) {
          // decltype(x)(x) is perfect forwarding in a lambda:
          return method.f( decltype(self)(self), decltype(args)(args)... );
        },
        std::forward<Var>(var)
      );
    };
  }
};

```


this creates a type that overloads `operator->*` with a `Variant` on the left hand side.

> 这创建了一个重载“operator->*”的类型，左侧有一个“Variant”。

```cpp
// C++17 class type deduction to find template argument of `print` here.
// a pseudo-method lambda should take `self` as its first argument, then
// the rest of the arguments afterwards, and invoke the action:
pseudo_method print = [](auto&& self, auto&&...args)->decltype(auto) {
  return decltype(self)(self).print( decltype(args)(args)... );
};

```

Now if we have 2 types each with a `print` method:

```cpp
struct A {
  void print( std::ostream& os ) const {
    os << "A";
  }
};
struct B {
  void print( std::ostream& os ) const {
    os << "B";
  }
};

```

note that they are unrelated types.  We can:

```cpp
std::variant<A,B> var = A{};

(var->*print)(std::cout);

```


and it will dispatch the call directly to `A::print(std::cout)` for us.  If we instead initialized the `var` with `B{}`, it would dispatch to `B::print(std::cout)`.

> 并且它将为我们直接将调用分派到“A:：print（std:：cout）”。如果我们用“B｛｝”初始化“var”，它将分派到“B:：print（std：：cout”）”。

If we created a new type C:

```cpp
struct C {};

```

then:

```cpp
std::variant<A,B,C> var = A{};
(var->*print)(std::cout);

```

will fail to compile, because there is no `C.print(std::cout)` method.


Extending the above would permit free function `print`s to be detected and used, possibly with use of `if constexpr` within the `print` pseudo-method.

> 扩展以上内容将允许检测和使用自由函数“print”，可能在“print”伪方法中使用“if constexpr”。


[live example](http://coliru.stacked-crooked.com/a/57f29d8406ad7b51) currently using `boost::variant` in place of `std::variant`.

> [实例](http://coliru.stacked-crooked.com/a/57f29d8406ad7b51)当前使用“boost:：variant”代替“std:：variation”。



## Basic std::variant use



This creates a variant (a tagged union) that can store either an `int` or a `string`.

> 这将创建一个变量（带标签的并集），该变量可以存储“int”或“string”。

```cpp
std::variant< int, std::string > var;

```

We can store one of either type in it:

```cpp
var = "hello"s;

```

And we can access the contents via `std::visit`:

```cpp
// Prints "hello\n":
visit( [](auto&& e) {
  std::cout << e << '\n';
}, var );

```

by passing in a polymorphic lambda or similar function object.

If we are certain we know what type it is, we can get it:

```cpp
auto str = std::get<std::string>(var);

```

but this will throw if we get it wrong.  `get_if`:

```cpp
auto* str  = std::get_if<std::string>(&var);

```

returns `nullptr` if you guess wrong.


Variants guarantee no dynamic memory allocation (other than which is allocated by their contained types).  Only one of the types in a variant is stored there, and in rare cases (involving exceptions while assigning and no safe way to back out) the variant can become empty.

> 变体保证没有动态内存分配（除了由其包含的类型分配）。变体中只有一个类型存储在那里，在极少数情况下（涉及分配时的异常，并且没有安全的退出方式），变体可能会变空。


Variants let you store multiple value types in one variable safely and efficiently.  They are basically smart, type-safe `union`s.

> 变量使您能够安全有效地将多个值类型存储在一个变量中。他们基本上是聪明的，类型安全的“联合”。



## Constructing a `std::variant`


This does not cover allocators.

```cpp
struct A {};
struct B { B()=default; B(B const&)=default; B(int){}; };
struct C { C()=delete; C(int) {}; C(C const&)=default; };
struct D { D( std::initializer_list<int> ) {}; D(D const&)=default; D()=default; };

std::variant<A,B> var_ab0; // contains a A()
std::variant<A,B> var_ab1 = 7; // contains a B(7)
std::variant<A,B> var_ab2 = var_ab1; // contains a B(7)
std::variant<A,B,C> var_abc0{ std::in_place_type<C>, 7 }; // contains a C(7)
std::variant<C> var_c0; // illegal, no default ctor for C
std::variant<A,D> var_ad0( std::in_place_type<D>, {1,3,3,4} ); // contains D{1,3,3,4}
std::variant<A,D> var_ad1( std::in_place_index<0> ); // contains A{}
std::variant<A,D> var_ad2( std::in_place_index<1>, {1,3,3,4} ); // contains D{1,3,3,4}

```



#### Remarks



Variant is a replacement for raw `union` use.  It is type-safe and knows what type it is, and it carefully constructs and destroys the objects within it when it should.

> 变体是原始“联合”使用的替代品。它是类型安全的，知道它是什么类型，并且在应该的时候小心地构造和销毁其中的对象。


It is almost never empty: only in corner cases where replacing its content throws and it cannot back out safely does it end up being in an empty state.

> 它几乎从来都不是空的：只有在替换内容抛出并且无法安全退出的情况下，它才会最终处于空状态。


It behaves somewhat like a `std::tuple`, and somewhat like an `std::optional`.

> 它的行为有点像“std:：tuple”，也有点像一个“std：：optional”。


Using `std::get` and `std::get_if` is usually a bad idea.  The right answer is usually `std::visit`, which lets you deal with every possibility right there.  `if constexpr` can be used within the `visit` if you need to branch your behavior, rather than doing a sequence of runtime checks that duplicate what `visit` will do more efficiently.

> 使用“std:：get”和“std::：get_if”通常是个坏主意。正确的答案通常是“std:：visit”，它可以让你处理所有的可能性`如果您需要对行为进行分支，则可以在“visit”中使用if constexpr，而不是执行一系列重复“viste”将更有效地执行的运行时检查。

