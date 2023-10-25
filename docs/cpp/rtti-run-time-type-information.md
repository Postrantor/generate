---
tip: translate by baidu@2023-10-25 08:30:52
---
---
metaTitle: "C++ | RTTI: Run-Time Type Information"
description: "Name of a type, dynamic_cast, The typeid keyword, When to use which cast in c++"
---

# RTTI: Run-Time Type Information




## Name of a type



You can retrieve the implementation defined name of a type in runtime by using the `.name()` member function of the `std::type_info` object returned by `typeid`.

> 通过使用“typeid”返回的“std:：type_info”对象的“.name（）”成员函数，可以在运行时检索类型的实现定义名称。

```cpp
#include <iostream>
#include <typeinfo>

int main()
{
    int speed = 110;

    std::cout << typeid(speed).name() << '\n';
}

```

Output (implementation-defined):

```cpp
int

```



## dynamic_cast



Use `dynamic_cast<>()` as a function, which helps you to cast down through an inheritance hierarchy ([main description](http://stackoverflow.com/documentation/c%2B%2B/5660/casts/20075/dynamic-cast-for-safely-casting-within-a-class-hierarchy#t=201611081029314553455)).

> 使用“dynamic_cast<>（）”作为函数，它可以帮助您通过继承层次结构向下进行强制转换（[main-description](http://stackoverflow.com/documentation/c%2B%2B/5660/casts/20075/dynamic-铸造-分级铸造-分级内铸造#t=201611081029314553455）。


If you must do some non-polymorphic work on some derived classes `B` and `C`, but received the base `class A`,  then write like this:

> 如果您必须对某些派生类“B”和“C”执行一些非多态性操作，但收到了基“class A”，则如下所示：

```cpp
class A { public: virtual ~A(){} };

class B: public A
{ public: void work4B(){} };

class C: public A
{ public: void work4C(){} };

void non_polymorphic_work(A* ap)
{
  if (B* bp =dynamic_cast<B*>(ap))
    bp->work4B(); 
  if (C* cp =dynamic_cast<C*>(ap))
    cp->work4C(); 
}

```



## The typeid keyword



The `typeid` [keyword](http://stackoverflow.com/documentation/c%2b%2b/4891/keywords) is a unary operator that yields run-time type information about its operand if the operand's type is a polymorphic class type. It returns an lvalue of type `const std::type_info`. Top-level cv-qualification are ignored.

> `typeid`[关键字](http://stackoverflow.com/documentation/c%2b%2b/4891/keywords)是一个一元运算符，如果操作数的类型是多态类类型，它将生成有关其操作数的运行时类型信息。它返回一个类型为`conststd:：type_info`的左值。忽略高级简历资格。

```cpp
struct Base {
    virtual ~Base() = default;
};
struct Derived : Base {};
Base* b = new Derived;
assert(typeid(*b) == typeid(Derived{})); // OK

```

`typeid` can also be applied to a type directly. In this case, first top-level references are stripped, then top-level cv-qualification is ignored. Thus, the above example could have been written with `typeid(Derived)` instead of `typeid(Derived{})`:

```cpp
assert(typeid(*b) == typeid(Derived{})); // OK

```


If `typeid` is applied to any expression that is **not** of polymorphic class type, the operand is not evaluated, and the type info returned is for the static type.

> 如果将“typeid”应用于任何不是多态类类型**的表达式，则不计算操作数，并且返回的类型信息用于静态类型。

```cpp
struct Base {
    // note: no virtual destructor
};
struct Derived : Base {};
Derived d;
Base& b = d;
assert(typeid(b) == typeid(Base)); // not Derived
assert(typeid(std::declval<Base>()) == typeid(Base)); // OK because unevaluated

```



## When to use which cast in c++



Use **dynamic_cast** for converting pointers/references within an inheritance hierarchy.

> 使用**dynamic_cast**在继承层次结构中转换指针/引用。

Use **static_cast** for ordinary type conversions.


Use **reinterpret_cast** for low-level reinterpreting of bit patterns. Use with extreme caution.

> 使用**reinterpret_cast**对位模式进行低级重新解释。使用时要格外小心。


Use **const_cast** for casting away const/volatile. Avoid this unless you are stuck using a const-incorrect API.

> 使用**const_cast**来强制转换const/vatile。除非您一直在使用const-incorrect API，否则请避免这种情况。

