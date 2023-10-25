---
tip: translate by baidu@2023-10-25 08:21:54
---
---

metaTitle: "C++ | Classes/Structures"
description: "Class basics, Final classes and structs, Access specifiers, Inheritance, Friendship, Virtual Inheritance, Accessing class members, Private inheritance: restricting base class interface, Nested Classes/Structures, Member Types and Aliases, Multiple Inheritance, Static class members, Non-static member functions, Unnamed struct/class"
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# Classes/Structures

## Class basics

A **class** is a user-defined type. A class is introduced with the `class`, `struct` or `union` keyword. In colloquial usage, the term "class" usually refers only to non-union classes.

> **类**是用户定义的类型。类是用“class”、“struct”或“union”关键字引入的。在口语用法中，“类”一词通常只指非联合类。

A class is a collection of **class members**, which can be:

- member variables (also called "fields"),
- member functions (also called "methods"),
- member types or typedefs (e.g. "nested classes"),
- member templates (of any kind: variable, function, class or alias template)

The `class` and `struct` keywords, called **class keys**, are largely interchangeable, except that the default access specifier for members and bases is "private" for a class declared with the `class` key and "public" for a class declared with the `struct` or `union` key (cf. [Access modifiers](http://stackoverflow.com/documentation/c%2B%2B/508/classes-structures/1668/access-modifiers)).

> 被称为**类键**的“class”和“struct”关键字在很大程度上是可互换的，除了成员和基的默认访问说明符对于用“class”键声明的类是“private”，而对于用“struct'或“union”键声明（参见[access modifiers]([http://stackoverflow.com/documentation/c%2b%2b/508/classes](http://stackoverflow.com/documentation/c%2B%2B/508/classes)-结构/1668/访问修饰符）。

For example, the following code snippets are identical:

```cpp
struct Vector
{
    int x;
    int y;
    int z;
};
// are equivalent to
class Vector
{
public:
    int x;
    int y;
    int z;
};

```

By declaring a class` a new type is added to your program, and it is possible to instantiate objects of that class by

> 通过声明一个类“一个新类型被添加到程序中，并且可以通过

```cpp
Vector my_vector;

```

Members of a class are accessed using dot-syntax.

```cpp
my_vector.x = 10;
my_vector.y = my_vector.x + 1; // my_vector.y = 11;
my_vector.z = my_vector.y - 4; // my:vector.z = 7;

```

## Final classes and structs

Deriving a class may be forbidden with `final` specifier. Let's declare a final class:

> 可以禁止使用“final”说明符派生类。让我们宣布最后一个类：

```cpp
class A final {
};

```

Now any attempt to subclass it will cause a compilation error:

```cpp
// Compilation error: cannot derive from final class:
class B : public A {
};

```

Final class may appear anywhere in class hierarchy:

```cpp
class A {
};

// OK.
class B final : public A {
};

// Compilation error: cannot derive from final class B.
class C : public B {
};

```

## Access specifiers

There are three [keywords](http://stackoverflow.com/documentation/c%2B%2B/4891/keywords) that act as **access specifiers**. These limit the access to class members  following the specifier, until another specifier changes the access level again:

> 有三个[关键字](http://stackoverflow.com/documentation/c%2B%2B/4891/keywords)充当**访问说明符**。这些限制了对说明符后面的类成员的访问，直到另一个说明符再次更改访问级别：

| Keyword     | Description                                                    |  |  |  |  |  |  |  |  |
| ----------- | -------------------------------------------------------------- | - | - | - | - | - | - | - | - |
| `public`    | Everyone has access                                            |  |  |  |  |  |  |  |  |
| `protected` | Only the class itself, derived classes and friends have access |  |  |  |  |  |  |  |  |
| `private`   | Only the class itself and friends have access                  |  |  |  |  |  |  |  |  |

When the type is defined using the `class` keyword, the default access specifier is `private`, but if the type is defined using the `struct` keyword, the default access specifier is `public`:

> 当使用“class”关键字定义类型时，默认访问说明符为“private”，但如果使用“struct”关键字定义该类型，则默认访问说明符是“public”：

```cpp
struct MyStruct { int x; };
class MyClass { int x; };

MyStruct s;
s.x = 9; // well formed, because x is public

MyClass c;
c.x = 9; // ill-formed, because x is private

```

Access specifiers are mostly used to limit access to internal fields and methods, and force the programmer to use a specific interface, for example to force use of getters and setters instead of referencing a variable directly:

> 访问说明符主要用于限制对内部字段和方法的访问，并强制程序员使用特定接口，例如强制使用 getter 和 setter，而不是直接引用变量：

```cpp
class MyClass {

public: /* Methods: */

    int x() const noexcept { return m_x; }
    void setX(int const x) noexcept { m_x = x; }

private: /* Fields: */

    int m_x;

};

```

Using `protected` is useful for allowing certain functionality of the type to be only accessible to the derived classes, for example, in the following code, the method `calculateValue()` is only accessible to classes deriving from the base class `Plus2Base`, such as `FortyTwo`:

> 使用“protected”有助于允许该类型的某些功能仅可由派生类访问，例如，在以下代码中，方法“calculateValue（）”仅可由从基类“Plus2Base”派生的类访问，如“FortyTwo”：

```cpp
struct Plus2Base {
    int value() noexcept { return calculateValue() + 2; }
protected: /* Methods: */
    virtual int calculateValue() noexcept = 0;
};
struct FortyTwo: Plus2Base {
protected: /* Methods: */
    int calculateValue() noexcept final override { return 40; }
};

```

Note that the `friend` keyword can be used to add access exceptions to functions or types for accessing protected and private members.

> 请注意，“friend”关键字可用于为访问受保护成员和私有成员的函数或类型添加访问例外。

The `public`, `protected`, and `private` keywords can also be used to grant or limit access to base class subobjects. See the [Inheritance](http://stackoverflow.com/documentation/c%2B%2B/508/classes-structures/1669/inheritance) example.

> “public”、“protected”和“private”关键字也可用于授予或限制对基类子对象的访问权限。参见[继承]([http://stackoverflow.com/documentation/c%2b%2b/508/classes](http://stackoverflow.com/documentation/c%2B%2B/508/classes)-结构/1669/继承）示例。

## Inheritance

Classes/structs can have inheritance relations.

If a class/struct `B` inherits from a class/struct `A`, this means that `B` has as a parent `A`. We say that `B` is a derived class/struct from `A`, and `A` is the base class/struct.

> 如果类/结构“B”继承自类/结构的“a”，这意味着“B”具有父级“a”。我们说“B”是从“a”派生的类/结构，而“a”是基类/结构。

```cpp
struct A
{
public:
    int p1;
protected:
    int p2;
private:
    int p3;
};

//Make B inherit publicly (default) from A
struct B : A
{
};

```

There are 3 forms of inheritance for a class/struct:

- `public`
- `private`
- `protected`

Note that the default inheritance is the same as the default visibility of members: `public` if you use the `struct` keyword, and `private` for the `class` keyword.

> 请注意，默认继承与成员的默认可见性相同：如果使用“struct”关键字，则为“public”，而“class”关键字为“private”。

It's even possible to have a `class` derive from a `struct` (or vice versa). In this case, the default inheritance is controlled by the child, so a `struct` that derives from a `class` will default to public inheritance, and a `class` that derives from a `struct` will have private inheritance by default.

> 甚至可以从“struct”派生“class”（反之亦然）。在这种情况下，默认继承由子代控制，因此从“class”派生的“struct”将默认为公共继承，而从“struct'派生的“class”将默认具有私有继承。

`public` inheritance:

```cpp
struct B : public A // or just `struct B : A`
{
    void foo()
    {
        p1 = 0; //well formed, p1 is public in B
        p2 = 0; //well formed, p2 is protected in B
        p3 = 0; //ill formed, p3 is private in A
    }
};

B b;
b.p1 = 1; //well formed, p1 is public
b.p2 = 1; //ill formed, p2 is protected
b.p3 = 1; //ill formed, p3 is inaccessible

```

`private` inheritance:

```cpp
struct B : private A
{
    void foo()
    {
        p1 = 0; //well formed, p1 is private in B
        p2 = 0; //well formed, p2 is private in B
        p3 = 0; //ill formed, p3 is private in A
    }
};

B b;
b.p1 = 1; //ill formed, p1 is private
b.p2 = 1; //ill formed, p2 is private
b.p3 = 1; //ill formed, p3 is inaccessible

```

`protected` inheritance:

```cpp
struct B : protected A
{
    void foo()
    {
        p1 = 0; //well formed, p1 is protected in B
        p2 = 0; //well formed, p2 is protected in B
        p3 = 0; //ill formed, p3 is private in A
    }
};

B b;
b.p1 = 1; //ill formed, p1 is protected
b.p2 = 1; //ill formed, p2 is protected
b.p3 = 1; //ill formed, p3 is inaccessible

```

Note that although `protected` inheritance is allowed, the actual use of it is rare. One instance of how `protected` inheritance is used in application is in partial base class specialization (usually referred to as "controlled polymorphism").

> 请注意，尽管允许“受保护”继承，但实际使用它的情况很少。在应用程序中如何使用“受保护”继承的一个例子是部分基类专门化（通常称为“受控多态性”）。

When OOP was relatively new, (public) inheritance was frequently said to model an "IS-A" relationship. That is, public inheritance is correct only if an instance of the derived class **is also an** instance of the base class.

> 当 OOP 相对较新时，（公共）继承经常被认为是“IS-A”关系的模型。也就是说，只有当派生类**的实例也是基类的**实例时，公共继承才是正确的。

This was later refined into the [Liskov Substitution Principle](https://en.wikipedia.org/wiki/Liskov_substitution_principle): public inheritance should only be used when/if an instance of the derived class can be substituted for an instance of the base class under any possible circumstance (and still make sense).

> 这后来被完善为 [Liskov 替代原则](https://en.wikipedia.org/wiki/Liskov_substitution_principle)：只有当派生类的实例在任何可能的情况下都可以替换基类的实例时（并且仍然有意义），才应使用公共继承。

Private inheritance is typically said to embody a completely different relationship: "is implemented in terms of" (sometimes called a "HAS-A" relationship). For example, a `Stack` class could inherit privately from a `Vector` class. Private inheritance bears a much greater similarity to aggregation than to public inheritance.

> 私人继承通常被认为体现了一种完全不同的关系：“根据”（有时被称为“HAS-a”关系）来实现。例如，“Stack”类可以从“Vector”类私有继承。私有继承与聚合的相似性远大于公共继承。

Protected inheritance is almost never used, and there's no general agreement on what sort of relationship it embodies.

> 受保护的继承几乎从未被使用过，而且对于它所体现的关系也没有达成一致。

## Friendship

The `friend` [keyword](http://stackoverflow.com/documentation/c%2B%2B/4891/keyword) is used to give other classes and functions access to private and protected members of the class, even through they are defined outside the class`s scope.

> `friend` [关键字](http://stackoverflow.com/documentation/c%2B%2B/4891/keyword)用于授予其他类和函数访问该类的私有和受保护成员的权限，即使它们是在类的范围之外定义的。

```cpp
class Animal{
private:
    double weight;
    double height;
public:
    friend void printWeight(Animal animal);
    friend class AnimalPrinter;
    // A common use for a friend function is to overload the operator<< for streaming. 
    friend std::ostream& operator<<(std::ostream& os, Animal animal);
};

void printWeight(Animal animal)
{
    std::cout << animal.weight << "\n";
}

class AnimalPrinter
{
public:
    void print(const Animal& animal)
    {
        // Because of the `friend class AnimalPrinter;" declaration, we are
        // allowed to access private members here.
        std::cout << animal.weight << ", " << animal.height << std::endl;
    }
}

std::ostream& operator<<(std::ostream& os, Animal animal)
{
    os << "Animal height: " << animal.height << "\n";
    return os;
}

int main() {
    Animal animal = {10, 5};
    printWeight(animal);

    AnimalPrinter aPrinter;
    aPrinter.print(animal);

    std::cout << animal;
}

```

```cpp
10
10, 5
Animal height: 5

```

## Virtual Inheritance

When using inheritance, you can specify the `virtual` keyword:

```cpp
struct A{};
struct B: public virtual A{};

```

When class `B` has virtual base `A` it means that `A` **will reside in most derived class** of inheritance tree, and thus that most derived class is also responsible for initializing that virtual base:

> 当类“B”具有虚拟基“A”时，这意味着“A”**将驻留在继承树的最派生类**中，因此该最派生类也负责初始化该虚拟基：

```cpp
struct A
{
    int member;
    A(int param)
    {
        member = param;
    }
};

struct B: virtual A
{
    B(): A(5){}
};

struct C: B
{
    C(): /*A(88)*/ {}
};

void f()
{
    C object; //error since C is not initializing it's indirect virtual base `A`
}

```

If we un-comment `/*A(88)*/` we won't get any error since `C` is now initializing it's indirect virtual base `A`.

> 如果我们取消注释“/*A（88）*/”，我们不会得到任何错误，因为“C”现在正在初始化它的间接虚拟基“A”。

Also note that when we're creating variable `object`, most derived class is `C`, so `C` is responsible for creating(calling constructor of) `A` and thus value of `A::member` is `88`, not `5` (as it would be if we were creating object of type `B`).

> 还要注意，当我们创建变量“object”时，大多数派生类都是“C”，所以“C”负责创建（调用的构造函数）“A”，因此“A:：member”的值是“88”，而不是“5”（如果我们创建类型为“B”的对象，则会是这样）。

It is useful when solving the [diamond problem](https://en.wikipedia.org/wiki/Multiple_inheritance#The_diamond_problem).:

> 它在解决[钻石问题]时很有用([https://en.wikipedia.org/wiki/Multiple_inheritance#The_diamond_problem)](https://en.wikipedia.org/wiki/Multiple_inheritance#The_diamond_problem)).：

```

 A                                        A   A
 / \                                       |   |
B   C                                      B   C
 \ /                                        \ /
  D                                          D
virtual inheritance                   normal inheritance

```

`B` and `C` both inherit from `A`, and `D` inherits from `B` and `C`, so **there are 2 instances of `A` in `D`!** This results in ambiguity when you're accessing member of `A` through `D`, as the compiler has no way of knowing from which class do you want to access that member (the one which `B` inherits, or the one that is inherited by `C`?).

Virtual inheritance solves this problem: Since virtual base resides only in most derived object, there will be only one instance of `A` in `D`.

> 虚拟继承解决了这个问题：由于虚拟基只存在于大多数派生对象中，因此“D”中只有一个“A”实例。

```cpp
struct A
{
    void foo() {}
};

struct B : public /*virtual*/ A {};
struct C : public /*virtual*/ A {};

struct D : public B, public C
{
    void bar()
    {
        foo(); //Error, which foo? B::foo() or C::foo()? - Ambiguous
    }
};

```

Removing the comments resolves the ambiguity.

## Accessing class members

To access member variables and member functions of an object of a class, the `.` operator is used:

> 要访问类的对象的成员变量和成员函数，“.”使用运算符：

```cpp
struct SomeStruct {
  int a;
  int b;
  void foo() {}
};

SomeStruct var;
// Accessing member variable a in var.
std::cout << var.a << std::endl;
// Assigning member variable b in var.
var.b = 1;
// Calling a member function.
var.foo();

```

When accessing the members of a class via a pointer, the `->` operator is commonly used.  Alternatively, the instance can be dereferenced and the `.` operator used, although this is less common:

> 当通过指针访问类的成员时，通常使用 `->` 运算符。或者，可以取消引用实例，并使用“.”使用了运算符，尽管这种情况不太常见：

```cpp
struct SomeStruct {
  int a;
  int b;
  void foo() {}
};

SomeStruct var;
SomeStruct *p = &var;
// Accessing member variable a in var via pointer.
std::cout << p->a << std::endl;
std::cout << (*p).a << std::endl;
// Assigning member variable b in var via pointer.
p->b = 1;
(*p).b = 1;
// Calling a member function via a pointer.
p->foo();
(*p).foo();

```

When accessing static class members, the `::` operator is used, but on the name of the class instead of an instance of it.  Alternatively, the static member can be accessed from an instance or a pointer to an instance using the `.` or `->` operator, respectively, with the same syntax as accessing non-static members.

> 访问静态类成员时，会使用“：：”运算符，但它位于类的名称上，而不是它的实例上。或者，可以使用“.”从实例或指向实例的指针访问静态成员或 `->` 运算符，语法与访问非静态成员相同。

```cpp
struct SomeStruct {
  int a;
  int b;
  void foo() {}

  static int c;
  static void bar() {}
};
int SomeStruct::c;

SomeStruct var;
SomeStruct* p = &var;
// Assigning static member variable c in struct SomeStruct.
SomeStruct::c = 5;
// Accessing static member variable c in struct SomeStruct, through var and p.
var.a = var.c;
var.b = p->c;
// Calling a static member function.
SomeStruct::bar();
var.bar();
p->bar();

```

### Background

The `->` operator is needed because the member access operator `.` has precedence over the dereferencing operator `*`.

> 需要 `->` 运算符，因为成员访问运算符为 `.` 优先于取消引用运算符“*”。

One would expect that `*p.a` would dereference `p` (resulting in a reference to the object `p` is pointing to) and then accessing its member `a`. But in fact, it tries to access the member `a` of `p` and then dereference it. I.e. `*p.a` is equivalent to `*(p.a)`. In the example above, this would result in a compiler error because of two facts: First, `p` is a pointer and does not have a member `a`. Second, `a` is an integer and, thus, can't be dereferenced.

> 可以预期，“*p.a”将取消引用“p”（导致对“p”所指向的对象的引用），然后访问其成员“a”。但事实上，它试图访问“p”的成员“a”，然后取消对它的引用。例如，“*p.a”相当于“*（p.a）”。在上面的例子中，由于两个事实，这将导致编译器错误：首先，“p”是一个指针，没有成员“a’。其次，“a”是一个整数，因此不能取消引用。

The uncommonly used solution to this problem would be to explicitly control the precedence: `(*p).a`

> 这个问题的罕见解决方案是显式控制优先级：`（*p）.a`

Instead, the `->` operator is almost always used. It is a short-hand for first dereferencing the pointer and then accessing it. I.e. `(*p).a` is exactly the same as `p->a`.

> 相反，几乎总是使用 `->` 运算符。它是先取消引用指针，然后访问指针的捷径。例如，“（*p）.a”与“p->a”完全相同。

The `::` operator is the scope operator, used in the same manner as accessing a member of a namespace.  This is because a static class member is considered to be in that class' scope, but isn't considered a member of instances of that class.  The use of normal `.` and `->` is also allowed for static members, despite them not being instance members, for historical reasons; this is of use for writing generic code in templates, as the caller doesn't need to be concerned with whether a given member function is static or non-static.

> “：：”运算符是作用域运算符，使用方式与访问命名空间的成员相同。这是因为静态类成员被认为在该类的作用域中，但不被认为是该类实例的成员。正常 `.` 的用法由于历史原因，静态成员也允许使用 `->`，尽管它们不是实例成员；这对于在模板中编写泛型代码很有用，因为调用者不需要关心给定的成员函数是静态的还是非静态的。

## Private inheritance: restricting base class interface

Private inheritance is useful when it is required to restrict the public interface of the class:

> 当需要限制类的公共接口时，私有继承非常有用：

```cpp
class A {
public:
    int move();
    int turn();
};

class B : private A {
public:
    using A::turn; 
};

B b;
b.move();  // compile error
b.turn();  // OK

```

This approach efficiently prevents an access to the A public methods by casting to the A pointer or reference:

> 这种方法通过强制转换为 A 指针或引用，有效地防止了对 A 公共方法的访问：

```cpp
B b; 
A& a = static_cast<A&>(b); // compile error

```

In the case of public inheritance such casting will provide access to all the A public methods despite on alternative ways to prevent this in derived B, like hiding:

> 在公共继承的情况下，这种强制转换将提供对所有 A 公共方法的访问，尽管在派生的 B 中有其他方法可以防止这种情况，比如隐藏：

```cpp
class B : public A {
private:
    int move();  
};

```

or private using:

```cpp
class B : public A {
private:
    using A::move;  
};

```

then for both cases it is possible:

```cpp
B b;
A& a = static_cast<A&>(b); // OK for public inheritance
a.move(); // OK

```

## Nested Classes/Structures

A `class` or `struct` can also contain another `class`/`struct` definition inside itself, which is called a "nested class"; in this situation, the containing class is referred to as the "enclosing class".  The nested class definition is considered to be a member of the enclosing class, but is otherwise separate.

> “class”或“struct”也可以在其内部包含另一个“class”/“struct'定义，称为“嵌套类”；在这种情况下，包含类被称为“封闭类”。嵌套类定义被认为是封闭类的一个成员，但在其他方面是独立的。

```cpp
struct Outer {
    struct Inner { };
};

```

From outside of the enclosing class, nested classes are accessed using the scope operator.  From inside the enclosing class, however, nested classes can be used without qualifiers:

> 从封闭类的外部，使用作用域运算符访问嵌套类。但是，在封闭类内部，嵌套类可以在没有限定符的情况下使用：

```cpp
struct Outer {
    struct Inner { };

    Inner in;
};

// ...

Outer o;
Outer::Inner i = o.in;

```

As with a non-nested `class`/`struct`, member functions and static variables can be defined either within a nested class, or in the enclosing namespace.  However, they cannot be defined within the enclosing class, due to it being considered to be a different class than the nested class.

> 与非嵌套的“class”/“struct”一样，成员函数和静态变量可以在嵌套类中定义，也可以在封闭的命名空间中定义。但是，它们不能在封闭类中定义，因为它被认为是与嵌套类不同的类。

```cpp
// Bad.
struct Outer {
    struct Inner {
        void do_something();
    };

    void Inner::do_something() {}
};


// Good.
struct Outer {
    struct Inner {
        void do_something();
    };

};

void Outer::Inner::do_something() {}

```

As with non-nested classes, nested classes can be forward declared and defined later, provided they are defined before being used directly.

> 与非嵌套类一样，嵌套类可以在以后进行前向声明和定义，前提是它们在直接使用之前进行了定义。

```cpp
class Outer {
    class Inner1;
    class Inner2;

    class Inner1 {};

    Inner1 in1;
    Inner2* in2p;

  public:
    Outer();
    ~Outer();
};

class Outer::Inner2 {};

Outer::Outer() : in1(Inner1()), in2p(new Inner2) {}
Outer::~Outer() {
    if (in2p) { delete in2p; }
}

```

Prior to C++11, nested classes only had access to type names, `static` members, and enumerators from the enclosing class; all other members defined in the enclosing class were off-limits.

> 在 C++11 之前，嵌套类只能访问封闭类中的类型名、“静态”成员和枚举器；封闭类中定义的所有其他成员都是禁止使用的。

As of C++11, nested classes, and members thereof, are treated as if they were `friend` s of the enclosing class, and can access all of its members, according to the usual access rules; if members of the nested class require the ability to evaluate one or more non-static members of the enclosing class, they must therefore be passed an instance:

> 从 C++11 开始，嵌套类及其成员被视为封闭类的“朋友”，并且可以根据通常的访问规则访问其所有成员；如果嵌套类的成员需要计算封闭类的一个或多个非静态成员的能力，则必须向它们传递一个实例：

```cpp
class Outer {
    struct Inner {
        int get_sizeof_x() {
            return sizeof(x); // Legal (C++11): x is unevaluated, so no instance is required.
        }

        int get_x() {
            return x; // Illegal: Can't access non-static member without an instance.
        }

        int get_x(Outer& o) {
            return o.x; // Legal (C++11): As a member of Outer, Inner can access private members.
        }
    };

    int x;
};

```

Conversely, the enclosing class is **not** treated as a friend of the nested class, and thus cannot access its private members without explicitly being granted permission.

> 相反，封闭类**不被视为嵌套类的朋友，因此在没有明确授予权限的情况下无法访问其私有成员。

```cpp
class Outer {
    class Inner {
        // friend class Outer;

        int x;
    };

    Inner in;

  public:
    int get_x() {
        return in.x; // Error: int Outer::Inner::x is private.
        // Uncomment "friend" line above to fix.
    }
};

```

Friends of a nested class are not automatically considered friends of the enclosing class; if they need to be friends of the enclosing class as well, this must be declared separately.  Conversely, as the enclosing class is not automatically considered a friend of the nested class, neither will friends of the enclosing class be considered friends of the nested class.

> 嵌套类的朋友不会自动被视为封闭类的朋友；如果它们也需要是封闭类的朋友，则必须单独声明。相反，由于封闭类不会自动被视为嵌套类的朋友，因此封闭类的朋友也不会被视为嵌入类的朋友。

```cpp
class Outer {
    friend void barge_out(Outer& out, Inner& in);

    class Inner {
        friend void barge_in(Outer& out, Inner& in);

        int i;
    };

    int o;
};

void barge_in(Outer& out, Outer::Inner& in) {
    int i = in.i;  // Good.
    int o = out.o; // Error: int Outer::o is private.
}

void barge_out(Outer& out, Outer::Inner& in) {
    int i = in.i;  // Error: int Outer::Inner::i is private.
    int o = out.o; // Good.
}

```

As with all other class members, nested classes can only be named from outside the class if they have public access.  However, you are allowed to access them regardless of access modifier, as long as you don't explicitly name them.

> 与所有其他类成员一样，只有具有公共访问权限的嵌套类才能从类外部命名。但是，只要不显式命名它们，就可以访问它们，而不管访问修饰符是什么。

```cpp
class Outer {
    struct Inner {
        void func() { std::cout << "I have no private taboo.\n"; }
    };

  public:
    static Inner make_Inner() { return Inner(); }
};

// ...

Outer::Inner oi; // Error: Outer::Inner is private.

auto oi = Outer::make_Inner(); // Good.
oi.func();                     // Good.
Outer::make_Inner().func();    // Good.

```

You can also create a type alias for a nested class.  If a type alias is contained in the enclosing class, the nested type and the type alias can have different access modifiers.  If the type alias is outside the enclosing class, it requires that either the nested class, or a `typedef` thereof, be public.

> 也可以为嵌套类创建类型别名。如果封闭类中包含类型别名，则嵌套类型和类型别名可以具有不同的访问修饰符。如果类型别名在封闭类之外，则要求嵌套类或其“typedef”是公共的。

```cpp
class Outer {
    class Inner_ {};

  public:
    typedef Inner_ Inner;
};

typedef Outer::Inner  ImOut; // Good.
typedef Outer::Inner_ ImBad; // Error.

// ...

Outer::Inner  oi; // Good.
Outer::Inner_ oi; // Error.
ImOut         oi; // Good.

```

As with other classes, nested classes can both derive from or be derived from by other classes.

> 与其他类一样，嵌套类既可以派生自其他类，也可以由其他类派生。

```cpp
struct Base {};

struct Outer {
    struct Inner : Base {};
};

struct Derived : Outer::Inner {};

```

This can be useful in situations where the enclosing class is derived from by another class, by allowing the programmer to update the nested class as necessary.  This can be combined with a typedef to provide a consistent name for each enclosing class' nested class:

> 这在封闭类由另一个类派生的情况下非常有用，因为它允许程序员根据需要更新嵌套类。这可以与 typedef 相结合，为每个封闭类的嵌套类提供一致的名称：

```cpp
class BaseOuter {
    struct BaseInner_ {
        virtual void do_something() {}
        virtual void do_something_else();
    } b_in;

  public:
    typedef BaseInner_ Inner;

    virtual ~BaseOuter() = default;

    virtual Inner& getInner() { return b_in; }
};

void BaseOuter::BaseInner_::do_something_else() {}

// ---

class DerivedOuter : public BaseOuter {
    // Note the use of the qualified typedef; BaseOuter::BaseInner_ is private.
    struct DerivedInner_ : BaseOuter::Inner {
        void do_something() override {}
        void do_something_else() override;
    } d_in;

  public:
    typedef DerivedInner_ Inner;

    BaseOuter::Inner& getInner() override { return d_in; }
};

void DerivedOuter::DerivedInner_::do_something_else() {}

// ...

// Calls BaseOuter::BaseInner_::do_something();
BaseOuter* b = new BaseOuter;
BaseOuter::Inner& bin = b->getInner();
bin.do_something();
b->getInner().do_something();

// Calls DerivedOuter::DerivedInner_::do_something();
BaseOuter* d = new DerivedOuter;
BaseOuter::Inner& din = d->getInner();
din.do_something();
d->getInner().do_something();

```

In the above case, both `BaseOuter` and `DerivedOuter` supply the member type `Inner`, as `BaseInner_` and `DerivedInner_`, respectively.  This allows nested types to be derived without breaking the enclosing class' interface, and allows the nested type to be used polymorphically.

> 在上述情况下，“BaseOuter”和“DerivedOuter”都提供成员类型“Inner”，分别为“BaseInner_”和“DerivedInner_。这允许在不破坏封闭类接口的情况下派生嵌套类型，并允许嵌套类型以多形式使用。

## Member Types and Aliases

A `class` or `struct` can also define member type aliases, which are type aliases contained within, and treated as members of, the class itself.

> “class”或“struct”也可以定义成员类型别名，这些别名是包含在类本身中并被视为其成员的类型别名。

```cpp
struct IHaveATypedef {
    typedef int MyTypedef;
};

struct IHaveATemplateTypedef {
    template<typename T>
    using MyTemplateTypedef = std::vector<T>;
};

```

Like static members, these typedefs are accessed using the scope operator, `::`.

> 与静态成员一样，这些 typedef 是使用作用域运算符“：：”访问的。

```cpp
IHaveATypedef::MyTypedef i = 5; // i is an int.

IHaveATemplateTypedef::MyTemplateTypedef<int> v; // v is a std::vector<int>.

```

As with normal type aliases, each member type alias is allowed to refer to any type defined or aliased before, but not after, its definition.  Likewise, a typedef outside the class definition can refer to any accessible typedefs within the class definition, provided it comes after the class definition.

> 与普通类型别名一样，每个成员类型别名都可以引用在其定义之前（而不是之后）定义或别名的任何类型。同样，类定义之外的 typedef 可以引用类定义中任何可访问的 typedefs，只要它在类定义之后。

```cpp
template<typename T>
struct Helper {
    T get() const { return static_cast<T>(42); }
};

struct IHaveTypedefs {
//    typedef MyTypedef NonLinearTypedef; // Error if uncommented.
    typedef int MyTypedef;
    typedef Helper<MyTypedef> MyTypedefHelper;
};

IHaveTypedefs::MyTypedef        i; // x_i is an int.
IHaveTypedefs::MyTypedefHelper hi; // x_hi is a Helper<int>.

typedef IHaveTypedefs::MyTypedef TypedefBeFree;
TypedefBeFree ii;                  // ii is an int.

```

Member type aliases can be declared with any access level, and will respect the appropriate access modifier.

> 成员类型别名可以用任何访问级别声明，并将尊重适当的访问修饰符。

```cpp
class TypedefAccessLevels {
    typedef int PrvInt;

  protected:
    typedef int ProInt;

  public:
    typedef int PubInt;
};

TypedefAccessLevels::PrvInt prv_i; // Error: TypedefAccessLevels::PrvInt is private.
TypedefAccessLevels::ProInt pro_i; // Error: TypedefAccessLevels::ProInt is protected.
TypedefAccessLevels::PubInt pub_i; // Good.

class Derived : public TypedefAccessLevels {
    PrvInt prv_i; // Error: TypedefAccessLevels::PrvInt is private.
    ProInt pro_i; // Good.
    PubInt pub_i; // Good.
};

```

This can be used to provide a level of abstraction, allowing a class' designer to change its internal workings without breaking code that relies on it.

> 这可以用来提供一个抽象级别，允许类的设计器在不破坏依赖它的代码的情况下更改其内部工作。

```cpp
class Something {
    friend class SomeComplexType;

    short s;
    // ...

  public:
    typedef SomeComplexType MyHelper;

    MyHelper get_helper() const { return MyHelper(8, s, 19.5, "shoe", false); }

    // ...
};

// ...

Something s;
Something::MyHelper hlp = s.get_helper();

```

In this situation, if the helper class is changed from `SomeComplexType` to some other type, only the `typedef` and the `friend` declaration would need to be modified; as long as the helper class provides the same functionality, any code that uses it as `Something::MyHelper` instead of specifying it by name will usually still work without any modifications.  In this manner, we minimise the amount of code that needs to be modified when the underlying implementation is changed, such that the type name only needs to be changed in one location.

> 在这种情况下，如果 helper 类从“SomeComplexType”更改为其他类型，则只需要修改“typedef”和“friend”声明；只要 helper 类提供相同的功能，任何使用它作为“Something:：MyHelper”而不是按名称指定它的代码通常都可以在没有任何修改的情况下工作。通过这种方式，我们可以最大限度地减少在更改底层实现时需要修改的代码量，从而只需要在一个位置更改类型名称。

This can also be combined with `decltype`, if one so desires.

```cpp
class SomethingElse {
    AnotherComplexType<bool, int, SomeThirdClass> helper;

  public:
    typedef decltype(helper) MyHelper;

  private:
    InternalVariable<MyHelper> ivh;

    // ...

  public:
    MyHelper& get_helper() const { return helper; }

    // ...
};

```

In this situation, changing the implementation of `SomethingElse::helper` will automatically change the typedef for us, due to `decltype`.  This minimises the number of modifications necessary when we want to change `helper`, which minimises the risk of human error.

> 在这种情况下，由于“decltype”，更改“SomethingElse:：helper”的实现将自动更改我们的 typedef。当我们想要更改“helper”时，这样可以最大限度地减少必要的修改次数，从而最大限度地降低人为错误的风险。

As with everything, however, this can be taken too far.  If the typename is only used once or twice internally and zero times externally, for example, there's no need to provide an alias for it.  If it's used hundreds or thousands of times throughout a project, or if it has a long enough name, then it can be useful to provide it as a typedef instead of always using it in absolute terms.  One must balance forwards compatibility and convenience with the amount of unnecessary noise created.

> 然而，与所有事情一样，这可能太过分了。例如，如果 typename 只在内部使用一到两次，在外部使用零次，则无需为其提供别名。如果它在整个项目中使用了数百或数千次，或者它的名称足够长，则将其作为 typedef 提供，而不是总是以绝对值使用它，这将非常有用。必须在前向兼容性和便利性与所产生的不必要的噪声量之间取得平衡。

This can also be used with template classes, to provide access to the template parameters from outside the class.

> 这也可以与模板类一起使用，以提供从类外部对模板参数的访问。

```cpp
template<typename T>
class SomeClass {
    // ...

  public:
    typedef T MyParam;
    MyParam getParam() { return static_cast<T>(42); }
};

template<typename T>
typename T::MyParam some_func(T& t) {
    return t.getParam();
}

SomeClass<int> si;
int i = some_func(si);

```

This is commonly used with containers, which will usually provide their element type, and other helper types, as member type aliases.  Most of the containers in the C++ standard library, for example, provide the following 12 helper types, along with any other special types they might need.

> 这通常与容器一起使用，容器通常会将其元素类型和其他辅助类型作为成员类型别名提供。例如，C++ 标准库中的大多数容器都提供以下 12 种辅助类型，以及它们可能需要的任何其他特殊类型。

```cpp
template<typename T>
class SomeContainer {
    // ...

  public:
    // Let's provide the same helper types as most standard containers.
    typedef T                                     value_type;
    typedef std::allocator<value_type>            allocator_type;
    typedef value_type&                           reference;
    typedef const value_type&                     const_reference;
    typedef value_type*                           pointer;
    typedef const value_type*                     const_pointer;
    typedef MyIterator<value_type>                iterator;
    typedef MyConstIterator<value_type>           const_iterator;
    typedef std::reverse_iterator<iterator>       reverse_iterator;
    typedef std::reverse_iterator<const_iterator> const_reverse_iterator;
    typedef size_t                                size_type;
    typedef ptrdiff_t                             difference_type;
};

```

Prior to C++11, it was also commonly used to provide a "template `typedef`" of sorts, as the feature wasn't yet available; these have become a bit less common with the introduction of alias templates, but are still useful in some situations (and are combined with alias templates in other situations, which can be very useful for obtaining individual components of a complex type such as a function pointer).  They commonly use the name `type` for their type alias.

> 在 C++11 之前，它也通常用于提供某种“模板 `typedef`”，因为该功能还不可用；随着别名模板的引入，这些模板变得不那么常见了，但在某些情况下仍然有用（在其他情况下与别名模板结合使用，这对于获得复杂类型的单个组件（如函数指针）非常有用）。它们通常使用名称“type”作为类型别名。

```cpp
template<typename T>
struct TemplateTypedef {
    typedef T type;
}

TemplateTypedef<int>::type i; // i is an int.

```

This was often used with types with multiple template parameters, to provide an alias that defines one or more of the parameters.

> 这通常用于具有多个模板参数的类型，以提供定义一个或多个参数的别名。

```cpp
template<typename T, size_t SZ, size_t D>
class Array { /* ... */ };

template<typename T, size_t SZ>
struct OneDArray {
    typedef Array<T, SZ, 1> type;
};

template<typename T, size_t SZ>
struct TwoDArray {
    typedef Array<T, SZ, 2> type;
};

template<typename T>
struct MonoDisplayLine {
    typedef Array<T, 80, 1> type;
};

OneDArray<int, 3>::type     arr1i; // arr1i is an Array<int, 3, 1>.
TwoDArray<short, 5>::type   arr2s; // arr2s is an Array<short, 5, 2>.
MonoDisplayLine<char>::type arr3c; // arr3c is an Array<char, 80, 1>.

```

## Multiple Inheritance

Aside from single inheritance:

```cpp
class A {};
class B : public A {};

```

You can also have multiple inheritance:

```cpp
class A {};
class B {};
class C : public A, public B {};

```

`C` will now have inherit from `A` and `B` at the same time.

***Note: this can lead to ambiguity if the same names are used in multiple inherited `class` s or `struct` s. Be careful!***

> ***注意：如果在多个继承的“class”或“struct”中使用相同的名称，这可能会导致歧义。小心***

*Ambiguity in Multiple Inheritance *

Multiple inheritance may be helpful in certain cases but, sometimes odd sort of problem encounters while using multiple inheritance.

> 多重继承在某些情况下可能会有所帮助，但有时在使用多重继承时会遇到一些奇怪的问题。

For example: Two base classes have functions with same name which is not overridden in derived class and if you write code to access that function using object of derived class, compiler shows error because, it cannot determine which function to call. Here is a code for this type of ambiguity in multiple inheritance.

> 例如：两个基类具有相同名称的函数，这些函数在派生类中不被重写，如果您编写代码使用派生类的对象访问该函数，编译器会显示错误，因为它无法确定要调用哪个函数。以下是针对多重继承中这种类型的模糊性的代码。

```cpp
class base1
{
  public:
     void funtion( )
     { //code for base1 function }  
};
class base2
{
    void function( )
     { // code for base2 function } 
};

class derived : public base1, public base2
{
    
};

int main()
{
    derived obj;
    
  // Error because compiler can't figure out which function to call 
  //either function( ) of base1 or base2 .   
    obj.function( )  
}

```

But, this problem can be solved using scope resolution function to specify which function to class either base1 or base2:

> 但是，这个问题可以通过使用范围解析函数来指定要对 base1 或 base2 进行分类的函数来解决：

```cpp
int main()
{
    obj.base1::function( );  // Function of class base1 is called. 
    obj.base2::function( );  // Function of class base2 is called.
}

```

## Static class members

A class is also allowed to have `static` members, which can be either variables or functions.  These are considered to be in the class' scope, but aren't treated as normal members; they have static storage duration (they exist from the start of the program to the end), aren't tied to a particular instance of the class, and only one copy exists for the entire class.

> 类还允许具有“静态”成员，这些成员可以是变量或函数。这些成员被认为在类的范围内，但不被视为正常成员；它们具有静态存储持续时间（从程序开始到结束都存在），不绑定到类的特定实例，并且整个类只存在一个副本。

```cpp
class Example {
    static int num_instances;      // Static data member (static member variable).
    int i;                         // Non-static member variable.

  public:
    static std::string static_str; // Static data member (static member variable).
    static int static_func();      // Static member function.

    // Non-static member functions can modify static member variables.
    Example() { ++num_instances; }
    void set_str(const std::string& str);
};

int         Example::num_instances;
std::string Example::static_str = "Hello.";

// ...

Example one, two, three;
// Each Example has its own "i", such that:
//  (&one.i != &two.i)
//  (&one.i != &three.i)
//  (&two.i != &three.i).
// All three Examples share "num_instances", such that:
//  (&one.num_instances == &two.num_instances)
//  (&one.num_instances == &three.num_instances)
//  (&two.num_instances == &three.num_instances)

```

Static member variables are not considered to be defined inside the class, only declared, and thus have their definition outside the class definition; the programmer is allowed, but not required, to initialise static variables in their definition.  When defining the member variables, the keyword `static` is omitted.

> 静态成员变量不被认为是在类内部定义的，只被声明，因此它们的定义在类定义之外；允许但不要求程序员在其定义中初始化静态变量。在定义成员变量时，省略了关键字“static”。

```cpp
class Example {
    static int num_instances;               // Declaration.

  public:
    static std::string static_str;          // Declaration.

    // ...
};

int         Example::num_instances;         // Definition.  Zero-initialised.
std::string Example::static_str = "Hello."; // Definition.

```

Due to this, static variables can be incomplete types (apart from `void`), as long as they're later defined as a complete type.

> 因此，静态变量可以是不完整类型（除了“void”），只要它们后来被定义为完整类型。

```cpp
struct ForwardDeclared;

class ExIncomplete {
    static ForwardDeclared fd;
    static ExIncomplete    i_contain_myself;
    static int             an_array[];
};

struct ForwardDeclared {};

ForwardDeclared ExIncomplete::fd;
ExIncomplete    ExIncomplete::i_contain_myself;
int             ExIncomplete::an_array[5];

```

Static member functions can be defined inside or outside the class definition, as with normal member functions.  As with static member variables, the keyword `static` is omitted when defining static member functions outside the class definition.

> 与普通成员函数一样，静态成员函数可以在类定义内部或外部定义。与静态成员变量一样，在类定义之外定义静态成员函数时，会省略关键字“static”。

```cpp
// For Example above, either...
class Example {
    // ...

  public:
    static int static_func() { return num_instances; }

    // ...

    void set_str(const std::string& str) { static_str = str; }
};

// Or...

class Example { /* ... */ };

int  Example::static_func() { return num_instances; }
void Example::set_str(const std::string& str) { static_str = str; }

```

If a static member variable is declared `const` but not `volatile`, and is of an integral or enumeration type, it can be initialised at declaration, inside the class definition.

> 如果静态成员变量被声明为“const”而不是“volatile”，并且是整型或枚举类型，则可以在类定义内的声明时对其进行初始化。

```cpp
enum E { VAL = 5 };

struct ExConst {
    const static int ci = 5;              // Good.
    static const E ce = VAL;              // Good.
    const static double cd = 5;           // Error.
    static const volatile int cvi = 5;    // Error.

    const static double good_cd;
    static const volatile int good_cvi;
};

const double ExConst::good_cd = 5;        // Good.
const volatile int ExConst::good_cvi = 5; // Good.

```

As of C++11, static member variables of `LiteralType` types (types that can be constructed at compile time, according to `constexpr` rules) can also be declared as `constexpr`; if so, they must be initialised within the class definition.

> 从 C++11 开始，“LiteralType”类型的静态成员变量（根据“constexpr”规则，可以在编译时构造的类型）也可以声明为“constexpr”；如果是这样，则必须在类定义中对它们进行初始化。

```cpp
struct ExConstexpr {
    constexpr static int ci = 5;                      // Good.
    static constexpr double cd = 5;                   // Good.
    constexpr static int carr[] = { 1, 1, 2 };        // Good.
    static constexpr ConstexprConstructibleClass c{}; // Good.
    constexpr static int bad_ci;                      // Error.
};

constexpr int ExConstexpr::bad_ci = 5;                // Still an error.

```

If a `const` or `constexpr` static member variable is **odr-used** (informally, if it has its address taken or is assigned to a reference), then it must still have a separate definition, outside the class definition.  This definition is not allowed to contain an initialiser.

> 如果“const”或“constexpr”静态成员变量被 **odr 使用**（非正式地，如果它的地址被占用或被分配给引用），那么它仍然必须在类定义之外有一个单独的定义。此定义不允许包含初始化程序。

```cpp
struct ExODR {
    static const int odr_used = 5;
};

// const int ExODR::odr_used;

const int* odr_user = & ExODR::odr_used; // Error; uncomment above line to resolve.

```

As static members aren't tied to a given instance, they can be accessed using the scope operator, `::`.

> 由于静态成员没有绑定到给定的实例，因此可以使用作用域运算符“：：”来访问它们。

```cpp
std::string str = Example::static_str;

```

They can also be accessed as if they were normal, non-static members.  This is of historical significance, but is used less commonly than the scope operator to prevent confusion over whether a member is static or non-static.

> 它们也可以被访问，就好像它们是普通的非静态成员一样。这具有历史意义，但与作用域运算符相比，它的使用频率较低，以防止混淆成员是静态的还是非静态的。

```cpp
Example ex;
std::string rts = ex.static_str;

```

Class members are able to access static members without qualifying their scope, as with non-static class members.

> 与非静态类成员一样，类成员可以在不限定其作用域的情况下访问静态成员。

```cpp
class ExTwo {
    static int num_instances;
    int my_num;

  public:
    ExTwo() : my_num(num_instances++) {}

    static int get_total_instances() { return num_instances; }
    int get_instance_number() const { return my_num; }
};

int ExTwo::num_instances;

```

They cannot be `mutable`, nor would they need to be; as they aren't tied to any given instance, whether an instance is or isn't const doesn't affect static members.

> 它们不可能是“可变的”，也不需要是；由于它们不绑定到任何给定的实例，因此实例是否为 const 不会影响静态成员。

```cpp
struct ExDontNeedMutable {
    int immuta;
    mutable int muta;

    static int i;

    ExDontNeedMutable() : immuta(-5), muta(-5) {}
};
int ExDontNeedMutable::i;

// ...

const ExDontNeedMutable dnm;
dnm.immuta = 5; // Error: Can't modify read-only object.
dnm.muta = 5;   // Good.  Mutable fields of const objects can be written.
dnm.i = 5;      // Good.  Static members can be written regardless of an instance's const-ness.

```

Static members respect access modifiers, just like non-static members.

```cpp
class ExAccess {
    static int prv_int;

  protected:
    static int pro_int;

  public:
    static int pub_int;
};

int ExAccess::prv_int;
int ExAccess::pro_int;
int ExAccess::pub_int;

// ...

int x1 = ExAccess::prv_int; // Error: int ExAccess::prv_int is private.
int x2 = ExAccess::pro_int; // Error: int ExAccess::pro_int is protected.
int x3 = ExAccess::pub_int; // Good.

```

As they aren't tied to a given instance, static member functions have no `this` pointer; due to this, they can't access non-static member variables unless passed an instance.

> 由于它们不绑定到给定的实例，静态成员函数没有“this”指针；因此，除非传递实例，否则它们不能访问非静态成员变量。

```cpp
class ExInstanceRequired {
    int i;

  public:
    ExInstanceRequired() : i(0) {}

    static void bad_mutate() { ++i *= 5; }                         // Error.
    static void good_mutate(ExInstanceRequired& e) { ++e.i *= 5; } // Good.
};

```

Due to not having a `this` pointer, their addresses can't be stored in pointers-to-member-functions, and are instead stored in normal pointers-to-functions.

> 由于没有“this”指针，它们的地址不能存储在指向成员函数的指针中，而是存储在指向函数的普通指针中。

```cpp
struct ExPointer {
           void nsfunc() {}
    static void  sfunc() {}
};

typedef void (ExPointer::* mem_f_ptr)();
typedef void (*f_ptr)();

mem_f_ptr p_sf = &ExPointer::sfunc; // Error.
    f_ptr p_sf = &ExPointer::sfunc; // Good.

```

Due to not having a `this` pointer, they also cannot be `const` or `volatile`, nor can they have ref-qualifiers.  They also cannot be virtual.

> 由于没有“this”指针，它们也不能是“const”或“volatile”，也不能有 ref 限定符。它们也不能是虚拟的。

```cpp
struct ExCVQualifiersAndVirtual {
    static void   func()                {} // Good.
    static void  cfunc() const          {} // Error.
    static void  vfunc() volatile       {} // Error.
    static void cvfunc() const volatile {} // Error.
    static void  rfunc() &              {} // Error.
    static void rvfunc() &&             {} // Error.

    virtual static void vsfunc()        {} // Error.
    static virtual void svfunc()        {} // Error.
};

```

As they aren't tied to a given instance, static member variables are effectively treated as special global variables; they're created when the program starts, and destroyed when it exits, regardless of whether any instances of the class actually exist.  Only a single copy of each static member variable exists (unless the variable is declared `thread_local` (C++11 or later), in which case there's one copy per thread).

> 由于静态成员变量不与给定实例绑定，因此它们被有效地视为特殊的全局变量；它们在程序启动时创建，在程序退出时销毁，而不管类的实例是否真的存在。每个静态成员变量只存在一个副本（除非变量被声明为“thread_local”（C++11 或更高版本），在这种情况下，每个线程有一个副本）。

Static member variables have the same linkage as the class, whether the class has external or internal linkage.  Local classes and unnamed classes aren't allowed to have static members.

> 静态成员变量与类具有相同的链接，无论类具有外部链接还是内部链接。本地类和未命名类不允许具有静态成员。

## Non-static member functions

A class can have [non-static member functions](http://stackoverflow.com/documentation/c%2B%2B/5661/non-static-member-functions), which operate on individual instances of the class.

> 类可以具有[非静态成员函数]([http://stackoverflow.com/documentation/c%2B%2B/5661/non](http://stackoverflow.com/documentation/c%2B%2B/5661/non)-静态成员函数），它们对类的各个实例进行操作。

```cpp
class CL {
  public:
    void member_function() {}
};

```

These functions are called on an instance of the class, like so:

```cpp
CL instance;
instance.member_function();

```

They can be defined either inside or outside the class definition; if defined outside, they are specified as being in the class' scope.

> 它们可以在类定义内部或外部进行定义；如果在外部定义，则将它们指定为在类的作用域中。

```cpp
struct ST {
    void  defined_inside() {}
    void defined_outside();
};
void ST::defined_outside() {}

```

They can be [CV-qualified](http://stackoverflow.com/documentation/c%2B%2B/7146/the-this-pointer/24492/this-pointer-cv-qualifiers#t=201610150101175922343) and/or [ref-qualified](http://stackoverflow.com/documentation/c%2B%2B/7146/the-this-pointer/24493/this-pointer-ref-qualifiers#t=201610150101354472159), affecting how they see the instance they're called upon; the function will see the instance as having the specified cv-qualifier(s), if any.  Which version is called will be based on the instance's cv-qualifiers.  If there is no version with the same cv-qualifiers as the instance, then a more-cv-qualified version will be called if available.

> 他们可以[简历合格]([http://stackoverflow.com/documentation/c%2B%2B/7146/the-this](http://stackoverflow.com/documentation/c%2B%2B/7146/the-this) pointer/24492/this pointer cv 限定符#t=201610150101175922343）和/或[ref qualified]([http://stackoverflow.com/documentation/c%2B%2B/7146/the-this](http://stackoverflow.com/documentation/c%2B%2B/7146/the-this) pointer/24493/this pointer ref 限定符#t=201610150101354472159），影响它们如何查看被调用的实例；函数会将实例视为具有指定的 cv 限定符（如果有的话）。调用哪个版本将基于实例的 cv 限定符。如果没有与实例具有相同 cv 限定符的版本，那么将调用一个更符合 cv 限定的版本（如果可用）。

```cpp
struct CVQualifiers {
    void func()                   {} // 1: Instance is non-cv-qualified.
    void func() const             {} // 2: Instance is const.

    void cv_only() const volatile {}
};

CVQualifiers       non_cv_instance;
const CVQualifiers      c_instance;

non_cv_instance.func(); // Calls #1.
c_instance.func();      // Calls #2.

non_cv_instance.cv_only(); // Calls const volatile version.
c_instance.cv_only();      // Calls const volatile version.

```

Member function ref-qualifiers indicate whether or not the function is intended to be called on rvalue instances, and use the same syntax as function cv-qualifiers.

> 成员函数 ref 限定符指示是否要在右值实例上调用该函数，并使用与函数 cv 限定符相同的语法。

```cpp
struct RefQualifiers {
    void func() &  {} // 1: Called on normal instances.
    void func() && {} // 2: Called on rvalue (temporary) instances.
};

RefQualifiers rf;
rf.func();              // Calls #1.
RefQualifiers{}.func(); // Calls #2.

```

CV-qualifiers and ref-qualifiers can also be combined if necessary.

```cpp
struct BothCVAndRef {
    void func() const& {} // Called on normal instances.  Sees instance as const.
    void func() &&     {} // Called on temporary instances.
};

```

They can also be [virtual](http://stackoverflow.com/documentation/c%2B%2B/1752/virtual-member-functions); this is fundamental to polymorphism, and allows a child class(es) to provide the same interface as the parent class, while supplying their own functionality.

> 它们也可以是[虚拟的]([http://stackoverflow.com/documentation/c%2B%2B/1752/virtual](http://stackoverflow.com/documentation/c%2B%2B/1752/virtual)-成员职能）；这是多态性的基础，并且允许子类提供与父类相同的接口，同时提供它们自己的功能。

```cpp
struct Base {
    virtual void func() {}
};
struct Derived {
    virtual void func() {}
};

Base* bp = new Base;
Base* dp = new Derived;
bp.func(); // Calls Base::func().
dp.func(); // Calls Derived::func().

```

For more information, see [here](http://stackoverflow.com/documentation/c%2B%2B/5661/non-static-member-functions).

> 有关更多信息，请参阅[此处]([http://stackoverflow.com/documentation/c%2B%2B/5661/non](http://stackoverflow.com/documentation/c%2B%2B/5661/non)-静态成员函数）。

## Unnamed struct/class

**Unnamed `struct`** is allowed (type has no name)

```cpp
void foo()
{
    struct /* No name */ {
        float x;
        float y;
    } point;
    
    point.x = 42;
}

```

or

```cpp
struct Circle
{
    struct /* No name */ {
        float x;
        float y;
    } center; // but a member name
    float radius;
};

```

and later

```cpp
Circle circle;
circle.center.x = 42.f;

```

but NOT **anonymous `struct`** (unnamed type and unnamed object)

```cpp
struct InvalidCircle
{
    struct /* No name */ {
        float centerX;
        float centerY;
    }; // No member either.
    float radius;
};

```

Note: Some compilers allow **anonymous `struct`** as **extension**.

<li>
**lamdba** can be seen as a special **unnamed `struct`**.
</li>
<li>
`decltype` allows to retrieve the type of **unnamed `struct`**:

```cpp
decltype(circle.point) otherPoint;

```

</li>
<li>
**unnamed `struct`** instance can be parameter of template method:

```cpp
void print_square_coordinates()
{
    const struct {float x; float y;} points[] = {
        {-1, -1}, {-1, 1}, {1, -1}, {1, 1}
    };

    // for range relies on `template <class T, std::size_t N> std::begin(T (&)[N])`
    for (const auto& point : points) { 
        std::cout << "{" << point.x << ", " << point.y << "}\n";
    }

    decltype(points[0]) topRightCorner{1, 1};
    auto it = std::find(points, points + 4, topRightCorner);
    std::cout << "top right corner is the "
              << 1 + std::distance(points, it) << "th\n";
}

```

</li>

#### Syntax

- variable.member_var = constant;
- variable.member_function();
- variable_pointer->member_var = constant;
- variable_pointer->member_function();

#### Remarks

Note that the ****only**** difference between the `struct` and `class` keywords is that by default, the member variables, member functions, and base classes of a `struct` are `public`, while in a `class` they are `private`. C++ programmers tend to call it a class if it has constructors and destructors, and the ability to enforce its own invariants; or a struct if it's just a simple collection of values, but the C++ language itself makes no distinction.

> 请注意，“struct”和“class”关键字之间仅有***的区别是，默认情况下，“struct'”的成员变量、成员函数和基类是“public”，而在“class”中，它们是“private”。C++ 程序员倾向于将其称为类，如果它有构造函数和析构函数，并且能够强制执行自己的不变量；或者是一个结构，如果它只是一个简单的值集合，但 C++ 语言本身没有区别。
