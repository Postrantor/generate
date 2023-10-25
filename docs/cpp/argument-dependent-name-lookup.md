---
tip: translate by baidu@2023-10-25 08:20:19
---
---
metaTitle: "C++ | Argument Dependent Name Lookup"
description: "What functions are found"
---

# Argument Dependent Name Lookup



## What functions are found



Functions are found by first collecting a set of **"associated classes"** and **"associated namespaces"** that include one ore more of the following, depending on the argument type `T`. First, let us show the rules for classes, enumeration and class template specialization names.

> 函数是通过首先收集一组**“关联类”**和**“关联命名空间”**来找到的，这些类包括以下一个或多个，具体取决于参数类型“T”。首先，让我们展示类、枚举和类模板专用化名称的规则。

- If `T` is a nested class, member enumeration, then the surrounding class of it.
- If `T` is an enumeration (it may **also** be a class member!), the innermost namespace of it.

- If `T` is a class (it may **also** be nested!), all its base classes and the class itself. The innermost namespace of all associated classes.

> -如果“T”是一个类（它也可以**嵌套！），那么它的所有基类和类本身。所有关联类的最内部命名空间。

- If `T` is a `ClassTemplate<TemplateArguments>` (this is **also** a class!), the classes and namespaces associated with the template type arguments, the namespace of any template template argument and the surrounding class of any template template argument, if a template argument is a member template.

> -如果“T”是“ClassTemplate＜TemplateArguments＞”（这也是**类！），则与模板类型参数相关联的类和命名空间、任何模板模板参数的命名空间以及任何模板参数的周围类（如果模板参数是成员模板）。

Now there are a few rules for builtin types as well


- If `T` is a pointer to `U` or array of `U`, the classes and namespaces associated with `U`. Example: `void (*fptr)(A); f(fptr);`, includes the namespaces and classes associated with `void(A)` (see next rule).

> -如果“T”是指向“U”或“U”的数组的指针，则与“U”相关联的类和命名空间。示例：`void（*fptr）（A）；f（fptr）；`，包括与“void（A）”相关联的命名空间和类（请参见下一条规则）。

- If `T` is a function type, the classes and namespaces associated with parameter and return types. Example: `void(A)` would includes the namespaces and classes associated with `A`.

> -如果“T”是函数类型，则与参数和返回类型相关联的类和命名空间。示例：“void（A）”将包括与“A”关联的命名空间和类。

- If `T` is a pointer to member, the classes and namespaces associated with the member type (may apply to both pointer to member functions and pointer to data member!). Example: `B A::*p; void (A::*pf)(B); f(p); f(pf);` includes the namespaces and classes associated with `A`, `B`, `void(B)` (which applies bullet above for function types).

> -如果“T”是指向成员的指针，则与成员类型相关联的类和命名空间（可以同时应用于指向成员函数的指针和指向数据成员的指针！）。示例：`B A:：*p；无效（A:：*pf）（B）；f（p）；f（pf）；`包括与“A”、“B”、“void（B）”相关联的名称空间和类（将上面的项目符号应用于函数类型）。


**All functions and templates within all associated namespaces are found by argument dependent lookup.** In addition, **namespace-scope friend functions declared in associated classes are found**, which are normally not visible. Using directives are ignored, however.

> **所有关联命名空间中的所有函数和模板都是通过依赖于参数的查找找到的。**此外，还找到了**在关联类中声明的命名空间作用域友元函数，这些函数通常不可见。但是，会忽略使用指令。


All of the following example calls are valid, without qualifying `f` by the namespace name in the call.

> 以下所有示例调用都是有效的，不需要通过调用中的命名空间名称限定“f”。

```cpp
namespace A {
   struct Z { };
   namespace I { void g(Z); }
   using namespace I;

   struct X { struct Y { }; friend void f(Y) { } };
   void f(X p) { }
   void f(std::shared_ptr<X> p) { }
}

// example calls
f(A::X());
f(A::X::Y());
f(std::make_shared<A::X>());

g(A::Z()); // invalid: "using namespace I;" is ignored!

```

