---
tip: translate by baidu@2023-10-25 08:23:41
---
---

metaTitle: "C++ | Curiously Recurring Template Pattern (CRTP)"
description: "The Curiously Recurring Template Pattern (CRTP), CRTP to avoid code duplication"
----------------------------------------------------------------------------------------------

# Curiously Recurring Template Pattern (CRTP)

A pattern in which a class inherits from a class template with itself as one of its template parameters. CRTP is usually used to provide **static polymorphism** in C++.

> 一种模式，其中类从类模板继承，并将其自身作为模板参数之一。CRTP 通常用于在 C++ 中提供**静态多态性**。

## The Curiously Recurring Template Pattern (CRTP)

CRTP is a powerful, static alternative to virtual functions and traditional inheritance that can be used to give types properties at compile time. It works by having a base class template which takes, as one of its template parameters, the derived class. This permits it to legally perform a `static_cast` of its `this` pointer to the derived class.

> CRTP 是虚拟函数和传统继承的强大的静态替代方案，可用于在编译时赋予类型属性。它的工作原理是有一个基类模板，该模板将派生类作为其模板参数之一。这允许它合法地执行指向派生类的“This”指针的“static_cast”。

Of course, this also means that a CRTP class must **always** be used as the base class of some other class. And the derived class must pass itself to the base class.

> 当然，这也意味着 CRTP 类必须**始终**用作其他类的基类。派生类必须将自身传递给基类。

Let's say you have a set of containers that all support the functions `begin()` and `end()`. The standard library's requirements for containers require more functionality. We can design a CRTP base class that provides that functionality, based solely on `begin()` and `end()`:

> 假设您有一组容器，它们都支持函数“begin（）”和“end（）”。标准库对容器的要求需要更多的功能。我们可以仅基于“begin（）”和“end（）”来设计一个提供该功能的 CRTP 基类：

```cpp
#include <iterator>
template <typename Sub>
class Container {
  private:
    // self() yields a reference to the derived type
    Sub& self() { return *static_cast<Sub*>(this); }
    Sub const& self() const { return *static_cast<Sub const*>(this); }

  public:
    decltype(auto) front() {
      return *self().begin();
    }

    decltype(auto) back() {
      return *std::prev(self().end());
    }

    decltype(auto) size() const {
      return std::distance(self().begin(), self().end());
    }

    decltype(auto) operator[](std::size_t i) {
      return *std::next(self().begin(), i);
    }
};

```

The above class provides the functions `front()`, `back()`, `size()`, and `operator[]` for any subclass which provides `begin()` and `end()`. An example subclass is a simple dynamically allocated array:

> 上面的类为任何提供“begin（）”和“end（）”的子类提供了函数“front（）”、“back（）”，“size（）”以及“operator[]”。一个示例子类是一个简单的动态分配数组：

```cpp
#include <memory>
// A dynamically allocated array
template <typename T>
class DynArray : public Container<DynArray<T>> {
  public:
    using Base = Container<DynArray<T>>;

    DynArray(std::size_t size)
      : size_{size},
      data_{std::make_unique<T[]>(size_)}
    { }

    T* begin() { return data_.get(); }
    const T* begin() const { return data_.get(); }
    T* end() { return data_.get() + size_; }
    const T* end() const { return data_.get() + size_; }

  private:
    std::size_t size_;
    std::unique_ptr<T[]> data_;
};

```

Users of the `DynArray` class can use the interfaces provided by the CRTP base class easily as follows:

> “DynArray”类的用户可以很容易地使用 CRTP 基类提供的接口，如下所示：

```cpp
DynArray<int> arr(10);
arr.front() = 2;
arr[2] = 5;
assert(arr.size() == 10);

```

**Usefulness:**

This pattern particularly avoids virtual function calls at run-time which occur to traverse down the inheritance hierarchy and simply relies on static casts:

> 这种模式特别避免了在运行时进行虚拟函数调用，这种调用是为了遍历继承层次结构而发生的，并且只依赖于静态强制转换：

```cpp
DynArray<int> arr(10);
DynArray<int>::Base & base = arr;
base.begin(); // no virtual calls

```

The only static cast inside the function `begin()` in the base class `Container<DynArray<int>>` allows the compiler to drastically optimize the code and no virtual table look up happens at runtime.

> 基类“Container＜DynArray＜int＞”中函数“begin（）”内部唯一的静态类型转换允许编译器大幅优化代码，并且在运行时不会查找虚拟表。

**Limitations:**

Because the base class is templated and different for two different `DynArray` s

> 因为基类是模板化的，并且对于两个不同的“DynArray”是不同的

it is not possible to store pointers to their base classes in an type-homogenous array as one could generally do with normal inheritance where the base class is not dependent on the derived type:

> 不可能将指向其基类的指针存储在类型同构数组中，这通常与基类不依赖于派生类型的正常继承一样：

```cpp
class A {};
class B: public A{};

A* a = new B;

```

## CRTP to avoid code duplication

The example in [Visitor Pattern](http://stackoverflow.com/documentation/c%2B%2B/4335/design-pattern-implementation-in-c/15127/visitor-pattern#t=201607291405391540231) provides a compelling use-case for CRTP:

> [访问者模式]中的示例([http://stackoverflow.com/documentation/c%2b%2b/4335/design-pattern-imimplementation-in-c/15127/viewer](http://stackoverflow.com/documentation/c%2B%2B/4335/design-pattern-imimplementation-in-c/15127/viewer) pattern#t=201607291405391540231）为 CRTP 提供了一个引人注目的用例：

```cpp
struct IShape
{
    virtual ~IShape() = default;

    virtual void accept(IShapeVisitor&) const = 0;
};

struct Circle : IShape
{
    // ...        
    // Each shape has to implement this method the same way
    void accept(IShapeVisitor& visitor) const override { visitor.visit(*this); }
    // ...
};

struct Square : IShape
{
    // ...    
    // Each shape has to implement this method the same way
    void accept(IShapeVisitor& visitor) const override { visitor.visit(*this); }
    // ...
};

```

Each child type of `IShape` needs to implement the same function the same way. That's a lot of extra typing. Instead, we can introduce a new type in the hierarchy that does this for us:

> “IShape”的每个子类型都需要以相同的方式实现相同的函数。这是很多额外的打字。相反，我们可以在层次结构中引入一种新的类型来为我们做到这一点：

```cpp
template <class Derived>
struct IShapeAcceptor : IShape {
    void accept(IShapeVisitor& visitor) const override {
        // visit with our exact type
        visitor.visit(*static_cast<Derived const*>(this));
    }
};

```

And now, each shape simply needs to inherit from the acceptor:

```cpp
struct Circle : IShapeAcceptor<Circle>
{
    Circle(const Point& center, double radius) : center(center), radius(radius) {}
    Point center;
    double radius;
};

struct Square : IShapeAcceptor<Square>
{
    Square(const Point& topLeft, double sideLength) : topLeft(topLeft), sideLength(sideLength) {}    
    Point topLeft;
    double sideLength;
};

```

No duplicate code necessary.
