---
tip: translate by baidu@2023-10-25 08:30:41
---
---

metaTitle: "C++ | Return Type Covariance"
description: "2. Covariant result version of the base example, static type checking., 3. Covariant smart pointer result (automated cleanup)., 1. Base example without covariant returns, shows why they're desirable"
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# Return Type Covariance

## 2. Covariant result version of the base example, static type checking.

```cpp
// 2. Covariant result version of the base example, static type checking.

class Top
{
public:
    virtual Top* clone() const = 0;
    virtual ~Top() = default;       // Necessary for `delete` via Top*.
};

class D : public Top
{
public:
    D* /* ← Covariant return */ clone() const override
    { return new D( *this ); }
};

class DD : public D
{
private:
    int answer_ = 42;

public:
    int answer() const
    { return answer_;}

    DD* /* ← Covariant return */ clone() const override
    { return new DD( *this ); }
};

#include <iostream>
using namespace std;

int main()
{
    DD* p1 = new DD();
    DD* p2 = p1->clone();
    // Correct dynamic type DD for *p2 is guaranteed by the static type checking.

    cout << p2->answer() << endl;          // "42"
    delete p2;
    delete p1;
}

```

## 3. Covariant smart pointer result (automated cleanup).

```cpp
// 3. Covariant smart pointer result (automated cleanup).

#include <memory>
using std::unique_ptr;

template< class Type >
auto up( Type* p ) { return unique_ptr<Type>( p ); }

class Top
{
private:
    virtual Top* virtual_clone() const = 0;

public:
    unique_ptr<Top> clone() const
    { return up( virtual_clone() ); }

    virtual ~Top() = default;       // Necessary for `delete` via Top*.
};

class D : public Top
{
private:
    D* /* ← Covariant return */ virtual_clone() const override
    { return new D( *this ); }

public:
    unique_ptr<D> /* ← Apparent covariant return */ clone() const
    { return up( virtual_clone() ); }
};

class DD : public D
{
private:
    int answer_ = 42;

    DD* /* ← Covariant return */ virtual_clone() const override
    { return new DD( *this ); }

public:
    int answer() const
    { return answer_;}

    unique_ptr<DD> /* ← Apparent covariant return */ clone() const
    { return up( virtual_clone() ); }
};

#include <iostream>
using namespace std;

int main()
{
    auto  p1 = unique_ptr<DD>(new DD());
    auto  p2 = p1->clone();
    // Correct dynamic type DD for *p2 is guaranteed by the static type checking.

    cout << p2->answer() << endl;          // "42"
    // Cleanup is automated via unique_ptr.
 }

```

## 1. Base example without covariant returns, shows why they're desirable

```cpp
// 1. Base example not using language support for covariance, dynamic type checking.

class Top
{
public:
    virtual Top* clone() const = 0;
    virtual ~Top() = default;       // Necessary for `delete` via Top*.
};

class D : public Top
{

public:
    Top* clone() const override
    { return new D( *this ); }
};

class DD : public D
{
private:
    int answer_ = 42;

public:
    int answer() const
    { return answer_;}

    Top* clone() const override
    { return new DD( *this ); }
};

#include <assert.h>
#include <iostream>
#include <typeinfo>
using namespace std;

int main()
{
    cout << boolalpha;

    DD* p1 = new DD();
    Top* p2 = p1->clone();
    bool const  correct_dynamic_type = (typeid( *p2 ) == typeid( DD ));
    cout << correct_dynamic_type << endl;               // "true"

    assert( correct_dynamic_type ); // This is essentially dynamic type checking. :(
    auto p2_most_derived = static_cast<DD*>( p2 );
    cout << p2_most_derived->answer() << endl;          // "42"
    delete p2;
    delete p1;
}

```

#### Remarks

**Covariance** of a parameter or a return value for a virtual member function `m` is where its type `T` gets more specific in a derived class' override of `m`. The type `T` then varies (**variance**) in specificity in the same way (**co**) as the classes providing `m`. C++ provides language support for covariant **return types** that are raw pointers or raw references – the covariance is for the pointee or referent type.

> **虚拟成员函数“m”的参数或返回值的协方差**是其类型“T”在派生类“m”重写中变得更具体的地方。然后，类型“T”以与提供“m”的类相同的方式（**co**）在特异性上变化（**方差**）。C++ 为协变**返回类型**提供了语言支持，这些类型是原始指针或原始引用——协变用于指针对象或引用对象类型。

The C++ support is limited to return types because function return values are the only pure **out-arguments** in C++, and covariance is only type safe for a pure out-argument. Otherwise calling code could supply an object of less specific type than the receiving code expects. MIT professor Barbara Liskov investigated this and related variance type safety issues, and it's now known as the Liskov Substitution Principle, or [**LSP**](https://en.wikipedia.org/wiki/Liskov_substitution_principle).

> C++ 支持仅限于返回类型，因为函数返回值是 C++ 中唯一的纯***输出参数**，而协方差对于纯输出参数来说是唯一类型安全的。否则，调用代码可能提供的对象类型不如接收代码所期望的特定。麻省理工学院教授 Barbara Liskov 调查了这一问题和相关的方差型安全问题，现在被称为 Liskov 替代原则，或[**LSP**](https://en.wikipedia.org/wiki/Liskov_substitution_principle)。

The covariance support essentially helps to avoid downcasting and dynamic type checking.

> 协方差支持本质上有助于避免下广播和动态类型检查。

Since smart pointers are of class type one cannot use the built-in support for covariance directly for smart pointer results, but one can define **apparently covariant** non-`virtual` smart pointer result wrapper functions for a covariant `virtual` function that produces raw pointers.

> 由于智能指针是类类型的，因此不能直接为智能指针结果使用对协方差的内置支持，但可以为生成原始指针的协变“虚拟”函数定义**明显协变的**非“虚拟”智能指针结果包装函数。
