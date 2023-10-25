---
tip: translate by baidu@2023-10-25 08:31:45
---
---
metaTitle: "C++ | Special Member Functions"
description: "Default Constructor, Copy and swap, Destructor, Implicit Move and Copy, Virtual and Protected Destructors"
---

# Special Member Functions



## Default Constructor



A **default constructor** is a type of constructor that requires no parameters when called. It is named after the type it constructs and is a member function of it (as all constructors are).

> **默认构造函数**是一种在调用时不需要参数的构造函数类型。它以它构造的类型命名，并且是它的成员函数（就像所有构造函数一样）。

```cpp
class C{
    int i;
public:
    // the default constructor definition
    C()
    : i(0){ // member initializer list -- initialize i to 0
        // constructor function body -- can do more complex things here
    }
};

C c1; // calls default constructor of C to create object c1
C c2 = C(); // calls default constructor explicitly
C c3(); // ERROR: this intuitive version is not possible due to "most vexing parse"
C c4{}; // but in C++11 {} CAN be used in a similar way

C c5[2]; // calls default constructor for both array elements
C* c6 = new C[2]; // calls default constructor for both array elements

```


Another way to satisfy the "no parameters" requirement is for the developer to provide default values for all parameters:

> 满足“无参数”要求的另一种方法是开发人员为所有参数提供默认值：

```cpp
class D{
    int i;
    int j;
public:
    // also a default constructor (can be called with no parameters)
    D( int i = 0, int j = 42 ) 
    : i(i), j(j){
    }
};


D d; // calls constructor of D with the provided default values for the parameters

```


Under some circumstances (i.e., the developer provides no constructors and there are no other disqualifying conditions), the compiler implicitly provides an empty default constructor:

> 在某些情况下（即，开发人员不提供构造函数，也不存在其他不合格条件），编译器隐式提供一个空的默认构造函数：

```cpp
class C{
    std::string s; // note: members need to be default constructible themselves
};

C c1; // will succeed -- C has an implicitly defined default constructor

```


Having some other type of constructor is one of the disqualifying conditions mentioned earlier:

> 拥有其他类型的构造函数是前面提到的不合格条件之一：

```cpp
class C{
    int i;
public:
    C( int i ) : i(i){}
};

C c1; // Compile ERROR: C has no (implicitly defined) default constructor

```


To prevent implicit default constructor creation, a common technique is to declare it as `private` (with no definition). The intention is to cause a compile error when someone tries to use the constructor (this either results in an **Access to private** error or a linker error, depending on the compiler).

> 为了防止隐式默认构造函数的创建，一种常见的技术是将其声明为“private”（没有定义）。其目的是在有人尝试使用构造函数时导致编译错误（这可能会导致**访问私有**错误或链接器错误，具体取决于编译器）。


To be sure a default constructor (functionally similar to the implicit one) is defined, a developer could write an empty one explicitly.

> 为了确保定义了默认构造函数（功能类似于隐式构造函数），开发人员可以显式编写一个空构造函数。


In C++11, a developer can also use the `delete` keyword to prevent the compiler from providing a default constructor.

> 在C++11中，开发人员还可以使用“delete”关键字来防止编译器提供默认构造函数。

```cpp
class C{
    int i;
public:
    // default constructor is explicitly deleted
    C() = delete;
};

C c1; // Compile ERROR: C has its default constructor deleted

```


Furthermore, a developer may also be explicit about wanting the compiler to provide a default constructor.

> 此外，开发人员也可能明确表示希望编译器提供默认构造函数。

```cpp
class C{
    int i;
public:
    // does have automatically generated default constructor (same as implicit one)
    C() = default;

    C( int i ) : i(i){}
};

C c1; // default constructed
C c2( 1 ); // constructed with the int taking constructor 

```


You can determine whether a type has a default constructor (or is a primitive type) using `std::is_default_constructible` from `<type_traits>`:

> 您可以使用`<type_traits>`中的`std:：is_default_constructible`来确定类型是否具有默认构造函数（或是基元类型）：

```cpp
class C1{ };
class C2{ public: C2(){} };
class C3{ public: C3(int){} };

using std::cout; using std::boolalpha; using std::endl;
using std::is_default_constructible;
cout << boolalpha << is_default_constructible<int>() << endl; // prints true
cout << boolalpha << is_default_constructible<C1>() << endl; // prints true
cout << boolalpha << is_default_constructible<C2>() << endl; // prints true
cout << boolalpha << is_default_constructible<C3>() << endl; // prints false

```


In C++11, it is still possible to use the non-functor version of `std::is_default_constructible`:

> 在C++11中，仍然可以使用“std:：is_default_constructible”的非函子版本：

```cpp
cout << boolalpha << is_default_constructible<C1>::value << endl; // prints true

```



## Copy and swap



If you're writing a class that manages resources, you need to implement all the special member functions (see [Rule of Three/Five/Zero](http://stackoverflow.com/documentation/c%2b%2b/1206/the-rule-of-three-five-and-zero#t=201607231819366817285)). The most direct approach to writing the copy constructor and assignment operator would be:

> 如果您正在编写一个管理资源的类，则需要实现所有特殊的成员函数（请参见[Rule of Three/Five/Zero](http://stackoverflow.com/documentation/c%2b%2b/1206/the-三五和零的规则#t=201607231819366817285）。编写复制构造函数和赋值运算符的最直接方法是：

```cpp
person(const person &other)
    : name(new char[std::strlen(other.name) + 1])
    , age(other.age)
{
    std::strcpy(name, other.name);
}

person& operator=(person const& rhs) {
    if (this != &other) {
        delete [] name;
        name = new char[std::strlen(other.name) + 1];
        std::strcpy(name, other.name);
        age = other.age;
    }

    return *this;
}

```


But this approach has some problems. It fails the strong exception guarantee - if `new[]` throws, we've already cleared the resources owned by `this` and cannot recover. We're duplicating a lot of the logic of copy construction in copy assignment. And we have to remember the self-assignment check, which usually just adds overhead to the copy operation, but is still critical.

> 但这种方法也存在一些问题。它没有通过强有力的异常保证——如果“new[]”抛出，我们已经清除了“this”拥有的资源，无法恢复。我们在复制作业中复制了很多复制构造的逻辑。我们必须记住自分配检查，它通常只会增加复制操作的开销，但仍然很关键。


To satisfy the strong exception guarantee and avoid code duplication (double so with the subsequent move assignment operator), we can use the copy-and-swap idiom:

> 为了满足强大的异常保证并避免代码重复（使用后续的移动分配运算符加倍），我们可以使用复制和交换习惯用法：

```cpp
class person {
    char* name;
    int age;
public:
    /* all the other functions ... */

    friend void swap(person& lhs, person& rhs) {
        using std::swap; // enable ADL

        swap(lhs.name, rhs.name);
        swap(lhs.age, rhs.age);
    }

    person& operator=(person rhs) {
        swap(*this, rhs);
        return *this;
    }
};

```

Why does this work? Consider what happens when we have

```cpp
person p1 = ...;
person p2 = ...;
p1 = p2;

```


First, we copy-construct `rhs` from `p2` (which we didn't have to duplicate here). If that operation throws, we don't do anything in `operator=` and `p1` remains untouched. Next, we swap the members between `*this` and `rhs`, and then `rhs` goes out of scope. When `operator=`, that implicitly cleans the original resources of `this` (via the destructor, which we didn't have to duplicate). Self-assignment works too - it's less efficient with copy-and-swap (involves an extra allocation and deallocation), but if that's the unlikely scenario, we don't slow down the typical use case to account for it.

> 首先，我们从“p2”复制构造“rhs”（此处不必重复）。如果该操作抛出，则在“operator=”中不执行任何操作，并且“p1”保持不变。接下来，我们在“*this”和“rhs”之间交换成员，然后“rhs’超出范围。当“operator=”时，它隐式地清除了“this”的原始资源（通过析构函数，我们不必复制它）。自分配也起作用——复制和交换（涉及额外的分配和释放）的效率较低，但如果这是不太可能的情况，我们不会放慢典型用例的速度来解释它。

The above formulation works as-is already for move assignment.

```cpp
p1 = std::move(p2);

```


Here, we move-construct `rhs` from `p2`, and all the rest is just as valid. If a class is movable but not copyable, there is no need to delete the copy-assignment, since this assignment operator will simply be ill-formed due to the deleted copy constructor.

> 在这里，我们将构造“rhs”从“p2”移走，其余的都同样有效。如果一个类是可移动的但不可复制的，那么就没有必要删除复制赋值，因为这个赋值运算符只会因为删除了复制构造函数而格式不正确。



## Destructor



A **destructor** is a function without arguments that is called when a user-defined object is about to be destroyed. It is named after the type it destructs with a `~` prefix.

> **析构函数**是一个没有参数的函数，在用户定义的对象即将被销毁时调用。它是以其销毁的类型命名的，前缀为“~”。

```cpp
class C{
    int* is;
    string s;
public:
    C()
    : is( new int[10] ){
    }

    ~C(){  // destructor definition
        delete[] is;
    }
};

class C_child : public C{
    string s_ch;
public:
    C_child(){}
    ~C_child(){} // child destructor
};

void f(){
    C c1; // calls default constructor
    C c2[2]; // calls default constructor for both elements
    C* c3 = new C[2]; // calls default constructor for both array elements

    C_child c_ch;  // when destructed calls destructor of s_ch and of C base (and in turn s)

    delete[] c3; // calls destructors on c3[0] and c3[1]
} // automatic variables are destroyed here -- i.e. c1, c2 and c_ch

```


Under most circumstances (i.e., a user provides no destructor, and there are no other disqualifying conditions), the compiler provides a default destructor implicitly:

> 在大多数情况下（即，用户不提供析构函数，并且不存在其他不合格条件），编译器隐式提供默认析构函数：

```cpp
class C{
    int i;
    string s;
};

void f(){
    C* c1 = new C;
    delete c1; // C has a destructor
}

```

```cpp
class C{
    int m;
private:
    ~C(){} // not public destructor!
};

class C_container{
    C c;
};

void f(){
    C_container* c_cont = new C_container;
    delete c_cont; // Compile ERROR: C has no accessible destructor
}

```


In C++11, a developer can override this behavior by preventing the compiler from providing a default destructor.

> 在C++11中，开发人员可以通过阻止编译器提供默认的析构函数来覆盖这种行为。

```cpp
class C{
    int m;
public:
    ~C() = delete; // does NOT have implicit destructor
};

void f{
    C c1; 
} // Compile ERROR: C has no destructor

```


Furthermore, a developer may also be explicit about wanting the compiler to provide a default destructor.

> 此外，开发人员也可能明确表示希望编译器提供默认的析构函数。

```cpp
class C{
    int m;
public:
    ~C() = default; // saying explicitly it does have implicit/empty destructor
};

void f(){
    C c1;
} // C has a destructor -- c1 properly destroyed

```


You can determine whether a type has a destructor (or is a primitive type) using `std::is_destructible` from `<type_traits>`:

> 您可以使用`<type_traits>`中的`std:：is_destructible`来确定类型是否具有析构函数（或是基元类型）：

```cpp
class C1{ };
class C2{ public: ~C2() = delete };
class C3 : public C2{ };

using std::cout; using std::boolalpha; using std::endl;
using std::is_destructible;
cout << boolalpha << is_destructible<int>() << endl; // prints true
cout << boolalpha << is_destructible<C1>() << endl; // prints true
cout << boolalpha << is_destructible<C2>() << endl; // prints false
cout << boolalpha << is_destructible<C3>() << endl; // prints false

```



## Implicit Move and Copy



Bear in mind that declaring a destructor inhibits the compiler from generating implicit move constructors and move assignment operators. If you declare a destructor, remember to also add appropriate definitions for the move operations.

> 请记住，声明析构函数会禁止编译器生成隐式移动构造函数和移动赋值运算符。如果您声明了析构函数，请记住还要为移动操作添加适当的定义。


Furthermore, declaring move operations will suppress the generation of copy operations, so these should also be added (if the objects of this class are required to have copy semantics).

> 此外，声明移动操作将抑制复制操作的生成，因此还应该添加这些操作（如果该类的对象需要具有复制语义）。

```cpp
class Movable {
public:
    virtual ~Movable() noexcept = default;

    //    compiler won't generate these unless we tell it to
    //    because we declared a destructor
    Movable(Movable&&) noexcept = default;
    Movable& operator=(Movable&&) noexcept = default;

    //    declaring move operations will suppress generation
    //    of copy operations unless we explicitly re-enable them
    Movable(const Movable&) = default;
    Movable& operator=(const Movable&) = default;
};

```



## Virtual and Protected Destructors



A class designed to be inherited-from is called a Base class. Care should be taken with the special member functions of such a class.

> 设计为从继承的类称为基类。应注意此类的特殊成员功能。


A class designed to be used polymorphically at run-time (through a pointer to the base class) should declare the destructor `virtual`. This allows the derived parts of the object to be properly destroyed, even when the object is destroyed through a pointer to the base class.

> 设计为在运行时以多态方式使用的类（通过指向基类的指针）应该声明析构函数“virtual”。这允许正确地销毁对象的派生部分，即使通过指向基类的指针销毁对象也是如此。

```cpp
class Base {
public:
    virtual ~Base() = default;

private:
    //    data members etc.
};

class Derived : public Base { //  models Is-A relationship
public:
    //    some methods

private:
    //    more data members
};

//    virtual destructor in Base ensures that derived destructors
//    are also called when the object is destroyed
std::unique_ptr<Base> base = std::make_unique<Derived>();
base = nullptr;  //    safe, doesn't leak Derived's members

```


If the class does not need to be polymorphic, but still needs to allow its interface to be inherited, use a non-virtual `protected` destructor.

> 如果类不需要是多态的，但仍然需要允许继承其接口，请使用非虚拟的“受保护”析构函数。

```cpp
class NonPolymorphicBase {
public:
    //    some methods

protected:
    ~NonPolymorphicBase() = default; //    note: non-virtual

private:
    //    etc.
};

```


Such a class can never be destroyed through a pointer, avoiding silent leaks due to slicing.

> 这样的类永远不能通过指针销毁，避免了由于切片而导致的静默泄漏。


This technique especially applies to classes designed to be `private` base classes. Such a class might be used to encapsulate some common implementation details, while providing `virtual` methods as customisation points. This kind of class should never be used polymorphically, and a `protected` destructor helps to document this requirement directly in the code.

> 这种技术特别适用于设计为“私有”基类的类。这样的类可以用来封装一些常见的实现细节，同时提供“虚拟”方法作为定制点。这类类永远不应该以多态方式使用，而“受保护”的析构函数有助于在代码中直接记录这一需求。


Finally, some classes may require that they are **never** used as a base class. In this case, the class can be marked `final`. A normal non-virtual public destructor is fine in this case.

> 最后，有些类可能要求它们**永远不要**用作基类。在这种情况下，类可以标记为“final”。在这种情况下，普通的非虚拟公共析构函数是可以的。

```cpp
class FinalClass final {  //    marked final here
public:
    ~FinalClass() = default;

private:
    //    etc.
};

```

