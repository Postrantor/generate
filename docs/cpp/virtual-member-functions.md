---
tip: translate by baidu@2023-10-25 08:36:31
---
---
metaTitle: "C++ | Virtual Member Functions"
description: "Using override with virtual in C++11 and later, Virtual vs non-virtual member functions, Final virtual functions, Behaviour of virtual functions in constructors and destructors, Pure virtual functions"
---

# Virtual Member Functions



## Using override with virtual in C++11 and later



The specifier `override` has a special meaning in C++11 onwards, if appended at the end of function signature. This signifies that a function is

> 说明符“override”在C++11中以后具有特殊含义，如果附加在函数签名的末尾。这表示函数

- Overriding the function present in base class &
- The Base class function is `virtual`


There is no `run time` significance of this specifier as is mainly meant as an indication for compilers

> 此说明符没有“运行时”意义，因为它主要用于指示编译器


The example below will demonstrate the change in behaviour with our without using override.

> 下面的示例将演示在不使用覆盖的情况下使用我们的行为的变化。

Without `override`:

```cpp
#include <iostream>

struct X {
    virtual void f() { std::cout << "X::f()\n"; }
};

struct Y : X {
    // Y::f() will not override X::f() because it has a different signature,
    // but the compiler will accept the code (and silently ignore Y::f()).
    virtual void f(int a) { std::cout << a << "\n"; }
};

```

With `override`:

```cpp
#include <iostream>

struct X {
    virtual void f() { std::cout << "X::f()\n"; }
};

struct Y : X {
    // The compiler will alert you to the fact that Y::f() does not
    // actually override anything.
    virtual void f(int a) override { std::cout << a << "\n"; }
};

```


Note that `override` is not a keyword, but a special identifier which only may appear in function signatures. In all other contexts `override`  still may be used as an identifier:

> 请注意，“override”不是关键字，而是一个特殊标识符，只能出现在函数签名中。在所有其他上下文中，“override”仍然可以用作标识符：

```cpp
void foo() {
    int override = 1; // OK.
    int virtual = 2;  // Compilation error: keywords can't be used as identifiers.
}

```



## Virtual vs non-virtual member functions


With virtual member functions:

```cpp
#include <iostream>

struct X {
    virtual void f() { std::cout << "X::f()\n"; }
};

struct Y : X {
    // Specifying virtual again here is optional
    // because it can be inferred from X::f().
    virtual void f() { std::cout << "Y::f()\n"; } 
};

void call(X& a) {
    a.f();
}

int main() {
    X x;
    Y y;
    call(x); // outputs "X::f()"
    call(y); // outputs "Y::f()"
}


```

Without virtual member functions:

```cpp
#include <iostream>

struct X {
   void f() { std::cout << "X::f()\n"; }
};

struct Y : X {
   void f() { std::cout << "Y::f()\n"; }
};

void call(X& a) {
    a.f();
}

int main() {
    X x;
    Y y;
    call(x); // outputs "X::f()"
    call(y); // outputs "X::f()"
}


```



## Final virtual functions



C++11 introduced `final` specifier which forbids method overriding if appeared in method signature:

> C++11引入了“final”说明符，如果出现在方法签名中，则禁止方法重写：

```cpp
class Base {
public:
    virtual void foo() {
        std::cout << "Base::Foo\n";
    }
};

class Derived1 : public Base {
public:
    // Overriding Base::foo
    void foo() final {
        std::cout << "Derived1::Foo\n";
    }
};

class Derived2 : public Derived1 {
public:
    // Compilation error: cannot override final method
    virtual void foo() {
        std::cout << "Derived2::Foo\n";
    }
};

```


The specifier `final` can only be used with `virtual' member function and can't be applied to non-virtual member functions

> 说明符“final”只能与“virtual”成员函数一起使用，不能应用于非虚拟成员函数


Like `final`, there is also an specifier caller 'override' which prevent overriding of `virtual` functions in the derived class.

> 与“final”一样，还有一个说明符调用程序“override”，用于防止重写派生类中的“virtual”函数。


The specifiers `override` and `final` may be combined together to have desired effect:

> 说明符“override”和“final”可以组合在一起以具有所需的效果：

```cpp
class Derived1 : public Base {
public:
    void foo() final override {
        std::cout << "Derived1::Foo\n";
    }
};

```



## Behaviour of virtual functions in constructors and destructors



The behaviour of virtual functions in constructors and destructors is often confusing when first encountered.

> 构造函数和析构函数中虚拟函数的行为在第一次遇到时常常令人困惑。

```cpp
#include <iostream>
using namespace std;

class base { 
public:
    base() { f("base constructor"); }
    ~base() { f("base destructor"); }

    virtual const char* v() { return "base::v()"; }

    void f(const char* caller) { 
        cout << "When called from " << caller << ", "  << v() << " gets called.\n"; 
    }        
};

class derived : public base {
public:
    derived() { f("derived constructor"); }
    ~derived() { f("derived destructor"); }

    const char* v() override { return "derived::v()"; }

};

int main() {
     derived d;
}

```

**Output:**

> 
<p>When called from base constructor, base::v() gets called.<br />
When called from derived constructor, derived::v() gets called.<br />
When called from derived destructor, derived::v() gets called.<br />
When called from base destructor, base::v() gets called.</p>



The reasoning behind this is that the derived class may define additional members which are not yet initialized (in the constructor case) or already destroyed (in the destructor case), and calling its member functions would be unsafe. Therefore during construction and destruction of C++ objects, the **dynamic** type of `*this` is considered to be the constructor's or destructor's class and not a more-derived class.

> 这背后的原因是，派生类可能定义了其他尚未初始化（在构造函数的情况下）或已销毁（在析构函数的情况下下）的成员，并且调用其成员函数是不安全的。因此，在构造和销毁C++对象的过程中，“*this”的**动态**类型被认为是构造函数或析构函数的类，而不是更派生的类。

**Example:**

```cpp
#include <iostream>
#include <memory>

using namespace std;
class base {
public:
    base()
    {
        std::cout << "foo is " << foo() << std::endl;
    }
    virtual int foo() { return 42; }
};

class derived : public base {
    unique_ptr<int> ptr_;
public:
    derived(int i) : ptr_(new int(i*i)) { }
    // The following cannot be called before derived::derived due to how C++ behaves, 
    // if it was possible... Kaboom!
    int foo() override   { return *ptr_; } 
};

int main() {
    derived d(4);
}

```



## Pure virtual functions



We can also specify that a `virtual` function is **pure virtual** (abstract), by appending `= 0` to the declaration.  Classes with one or more pure virtual functions are considered to be abstract, and cannot be instantiated; only derived classes which define, or inherit definitions for, all pure virtual functions can be instantiated.

> 我们还可以通过在声明中附加“=0”来指定“virtual”函数是**纯虚拟**（抽象）。具有一个或多个纯虚拟函数的类被认为是抽象的，不能实例化；只有定义或继承所有纯虚拟函数定义的派生类才能实例化。

```cpp
struct Abstract {
    virtual void f() = 0;
};

struct Concrete {
    void f() override {}
};

Abstract a; // Error.
Concrete c; // Good.

```


Even if a function is specified as pure virtual, it can be given a default implementation.  Despite this, the function will still be considered abstract, and derived classes will have to define it before they can be instantiated.  In this case, the derived class' version of the function is even allowed to call the base class' version.

> 即使函数被指定为纯虚拟函数，也可以为其提供默认实现。尽管如此，函数仍然被认为是抽象的，派生类在实例化之前必须定义它。在这种情况下，函数的派生类版本甚至可以调用基类版本。

```cpp
struct DefaultAbstract {
    virtual void f() = 0;
};
void DefaultAbstract::f() {}

struct WhyWouldWeDoThis : DefaultAbstract {
    void f() override { DefaultAbstract::f(); }
};

```

There are a couple of reasons why we might want to do this:

<li>

If we want to create a class that can't itself be instantiated, but doesn't prevent its derived classes from being instantiated, we can declare the destructor as pure virtual.  Being the destructor, it must be defined anyways, if we want to be able to deallocate the instance.  And [as the destructor is most likely already virtual to prevent memory leaks during polymorphic use](http://stackoverflow.com/documentation/c%2B%2B/1717/polymorphism/20178/polymorphism-destructors), we won't incur an unnecessary performance hit from declaring another function `virtual`.  This can be useful when making interfaces.

> 如果我们想创建一个本身不能实例化但又不阻止其派生类被实例化的类，我们可以将析构函数声明为纯虚拟的。作为析构函数，如果我们希望能够释放实例，那么无论如何都必须定义它。并且[因为析构函数很可能已经是虚拟的，以防止多态使用期间的内存泄漏](http://stackoverflow.com/documentation/c%2B%2B/1717/polymorphism/20178/polymorphism-析构函数），我们不会因为声明另一个函数“virtual”而导致不必要的性能损失。这在制作界面时非常有用。

```cpp
  struct Interface {
      virtual ~Interface() = 0;
  };
  Interface::~Interface() = default;

  struct Implementation : Interface {};
  // ~Implementation() is automatically defined by the compiler if not explicitly
  //  specified, meeting the "must be defined before instantiation" requirement.

```


</li>
<li>

If most or all implementations of the pure virtual function will contain duplicate code, that code can instead be moved to the base class version, making the code easier to maintain.

> 如果纯虚拟函数的大多数或所有实现都包含重复的代码，则可以将该代码移到基类版本，从而使代码更易于维护。

```cpp
  class SharedBase {
      State my_state;
      std::unique_ptr<Helper> my_helper;
      // ...

    public:
      virtual void config(const Context& cont) = 0;
      // ...
  };
  /* virtual */ void SharedBase::config(const Context& cont) {
      my_helper = new Helper(my_state, cont.relevant_field);
      do_this();
      and_that();
  }

  class OneImplementation : public SharedBase {
      int i;
      // ...

    public:
      void config(const Context& cont) override;
      // ...
  };
  void OneImplementation::config(const Context& cont) /* override */ {
      my_state = { cont.some_field, cont.another_field, i };
      SharedBase::config(cont);
      my_unique_setup();
  };

  // And so on, for other classes derived from SharedBase.

```


</li>



#### Syntax


<li>
virtual void f();
</li>
<li>
virtual void g() = 0;
</li>
<li>
// C++11 or later:
<ul>
- virtual void h() override;
- void i() override;
- virtual void j() final;
- void k() final;



#### Remarks


- Only non-static, non-template member functions can be `virtual`.

- If you are using C++11 or later, it is recommended to use `override` when overriding a virtual member function from a base class.

> -如果使用C++11或更高版本，建议在重写基类中的虚拟成员函数时使用“override”。
- [Polymorphic base classes often have virtual destructors to allow a derived object to be deleted through a pointer to the base class](http://stackoverflow.com/documentation/c%2B%2B/1717/polymorphism/20178/polymorphism-destructors). If the destructor were not virtual, such an operation leads to [undefined behavior](http://stackoverflow.com/documentation/c%2b%2b/1812/undefined-behavior)<sup>[expr.delete] §5.3.5/3 </sup>.

