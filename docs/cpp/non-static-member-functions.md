---
tip: translate by baidu@2023-10-25 08:28:24
---
---

metaTitle: "C++ | Non-Static Member Functions"
description: "Non-static Member Functions, Encapsulation, Name Hiding & Importing, Virtual Member Functions, Const Correctness"
-------------------------------------------------------------------------------------------------------------------------------

# Non-Static Member Functions

## Non-static Member Functions

A `class` or `struct` can have member functions as well as member variables.  These functions have syntax mostly similar to standalone functions, and can be defined either inside or outside the class definition; if defined outside the class definition, the function's name is prefixed with the class' name and the scope (`::`) operator.

> “class”或“struct”既可以具有成员函数，也可以具有成员变量。这些函数的语法大多类似于独立函数，可以在类定义内部或外部定义；如果在类定义之外定义，函数的名称将以类名和 scope（`：：`）运算符为前缀。

```cpp
class CL {
  public:
    void  definedInside() {}
    void definedOutside();
};
void CL::definedOutside() {}

```

These functions are called on an instance (or reference to an instance) of the class with the dot (`.`) operator, or a pointer to an instance with the arrow (`->`) operator, and each call is tied to the instance the function was called on; when a member function is called on an instance, it has access to all of that instance's fields (through the [`this` pointer](http://stackoverflow.com/documentation/c%2B%2B/7146/the-this-pointer#t=201610191424041635584)), but can only access other instances' fields if those instances are supplied as parameters.

> 这些函数是用点（`.`）运算符在类的实例（或引用实例）上调用的，或是用箭头（`->`）运算符对实例的指针调用的，每次调用都与调用函数的实例绑定；当在实例上调用成员函数时，它可以访问该实例的所有字段（通过[`this'pointer]([http://stackoverflow.com/documentation/c%2B%2B/7146/the](http://stackoverflow.com/documentation/c%2B%2B/7146/the)-此指针#t=201610191424041635584），但只有当其他实例作为参数提供时，才能访问这些实例的字段。

```cpp
struct ST {
    ST(const std::string& ss = "Wolf", int ii = 359) : s(ss), i(ii) { }

    int get_i() const { return i; }
    bool compare_i(const ST& other) const { return (i == other.i); }

  private:
    std::string s;
    int i;
};
ST st1;
ST st2("Species", 8472);

int  i = st1.get_i(); // Can access st1.i, but not st2.i.
bool b = st1.compare_i(st2); // Can access st1 & st2.

```

These functions are allowed to access member variables and/or other member functions, regardless of either the variable or function's access modifiers.  They can also be written out-of-order, accessing member variables and/or calling member functions declared before them, as the entire class definition must be parsed before the compiler can begin to compile a class.

> 允许这些函数访问成员变量和/或其他成员函数，而不考虑变量或函数的访问修饰符。它们也可能被乱写，访问成员变量和/或调用在它们之前声明的成员函数，因为在编译器开始编译类之前，必须解析整个类定义。

```cpp
class Access {
  public:
    Access(int i_ = 8088, int j_ = 8086, int k_ = 6502) : i(i_), j(j_), k(k_) {}

    int i;
    int get_k() const { return k; }
    bool private_no_more() const { return i_be_private(); }
  protected:
    int j;
    int get_i() const { return i; }
  private:
    int k;
    int get_j() const { return j; }
    bool i_be_private() const { return ((i > j) && (k < j)); }
};

```

## Encapsulation

A common use of member functions is for encapsulation, using an **accessor** (commonly known as a getter) and a **mutator** (commonly known as a setter) instead of accessing fields directly.

> 成员函数的一个常见用途是用于封装，使用**访问器**（通常称为 getter）和**赋值器**（一般称为 setter），而不是直接访问字段。

```cpp
class Encapsulator {
    int encapsulated;

  public:
    int  get_encapsulated() const { return encapsulated; }
    void set_encapsulated(int e)  { encapsulated = e; }

    void some_func() {
        do_something_with(encapsulated);
    }
};

```

Inside the class, `encapsulated` can be freely accessed by any non-static member function; outside the class, access to it is regulated by member functions, using `get_encapsulated()` to read it and `set_encapsulated()` to modify it.  This prevents unintentional modifications to the variable, as separate functions are used to read and write it.  [There are many discussions on whether getters and setters provide or break encapsulation, with good arguments for both claims; such heated debate is outside the scope of this example.]

> 在类内部，任何非静态成员函数都可以自由访问“encapsulated”；在类之外，对它的访问由成员函数控制，使用“get_encapsulated（）”读取它，使用“set_encapsurated（））”修改它。这可以防止对变量的意外修改，因为单独的函数用于读写它。[关于 getter 和 setter 是提供封装还是破坏封装，有很多讨论，这两种说法都有很好的论据；如此激烈的争论超出了本例的范围。]

## Name Hiding & Importing

When a base class provides a set of overloaded functions, and a derived class adds another overload to the set, this hides all of the overloads provided by the base class.

> 当基类提供一组重载函数，而派生类向该集合添加另一个重载时，这将隐藏基类提供的所有重载。

```cpp
struct HiddenBase {
    void f(int) { std::cout << "int" << std::endl; }
    void f(bool) { std::cout << "bool" << std::endl; }
    void f(std::string) { std::cout << "std::string" << std::endl; }
};

struct HidingDerived : HiddenBase {
    void f(float) { std::cout << "float" << std::endl; }
};

// ...

HiddenBase hb;
HidingDerived hd;
std::string s;

hb.f(1);    // Output:  int
hb.f(true); // Output:  bool
hb.f(s);    // Output:  std::string;

hd.f(1.f);  // Output:  float
hd.f(3);    // Output:  float
hd.f(true); // Output:  float
hd.f(s);    // Error: Can't convert from std::string to float.

```

This is due to name resolution rules: During name lookup, once the correct name is found, we stop looking, even if we clearly haven't found the correct **version** of the entity with that name (such as with `hd.f(s)`); due to this, overloading the function in the derived class prevents name lookup from discovering the overloads in the base class.  To avoid this, a using-declaration can be used to "import" names from the base class into the derived class, so that they will be available during name lookup.

> 这是由于名称解析规则：在名称查找过程中，一旦找到正确的名称，我们就会停止查找，即使我们显然没有找到具有该名称的实体的正确**版本**（例如带有“hd.f（s）”）；因此，在派生类中重载函数可以防止名称查找发现基类中的重载。为了避免这种情况，可以使用 using 声明将基类中的名称“导入”到派生类中，以便在名称查找过程中可以使用这些名称。

```cpp
struct HidingDerived : HiddenBase {
     // All members named HiddenBase::f shall be considered members of HidingDerived for lookup.
    using HiddenBase::f;

    void f(float) { std::cout << "float" << std::endl; }
};

// ...

HidingDerived hd;

hd.f(1.f);  // Output:  float
hd.f(3);    // Output:  int
hd.f(true); // Output:  bool
hd.f(s);    // Output:  std::string

```

If a derived class imports names with a using-declaration, but also declares functions with the same signature as functions in the base class, the base class functions will silently be overridden or hidden.

> 如果派生类使用 using 声明导入名称，但同时声明与基类中的函数具有相同签名的函数，则基类函数将被静默地重写或隐藏。

```cpp
struct NamesHidden {
    virtual void hide_me()      {}
    virtual void hide_me(float) {}
    void hide_me(int)           {}
    void hide_me(bool)          {}
};

struct NameHider : NamesHidden {
    using NamesHidden::hide_me;

    void hide_me()    {} // Overrides NamesHidden::hide_me().
    void hide_me(int) {} // Hides NamesHidden::hide_me(int).
};

```

A using-declaration can also be used to change access modifiers, provided the imported entity was `public` or `protected` in the base class.

> using 声明也可以用于更改访问修饰符，前提是导入的实体在基类中是“public”或“protected”。

```cpp
struct ProMem {
  protected:
    void func() {}
};

struct BecomesPub : ProMem {
    using ProMem::func;
};

// ...

ProMem pm;
BecomesPub bp;

pm.func(); // Error: protected.
bp.func(); // Good.

```

Similarly, if we explicitly want to call a member function from a specific class in the inheritance hierarchy, we can qualify the function name when calling the function, specifying that class by name.

> 类似地，如果我们明确希望从继承层次结构中的特定类调用成员函数，我们可以在调用函数时限定函数名称，并按名称指定该类。

```cpp
struct One {
    virtual void f() { std::cout << "One." << std::endl; }
};

struct Two : One {
    void f() override {
        One::f(); // this->One::f();
        std::cout << "Two." << std::endl;
    }
};

struct Three : Two {
    void f() override {
        Two::f(); // this->Two::f();
        std::cout << "Three." << std::endl;
    }
};

// ...

Three t;

t.f();      // Normal syntax.
t.Two::f(); // Calls version of f() defined in Two.
t.One::f(); // Calls version of f() defined in One.

```

## Virtual Member Functions

[Member functions can also be declared `virtual`.](http://stackoverflow.com/documentation/c%2B%2B/1752/virtual-member-functions)  In this case, if called on a pointer or reference to an instance, they will not be accessed directly; rather, they will look up the function in the virtual function table (a list of pointers-to-member-functions for virtual functions, more commonly known as the `vtable` or `vftable`), and use that to call the version appropriate for the instance's dynamic (actual) type.  If the function is called directly, from a variable of a class, no lookup is performed.

> [成员函数也可以声明为“虚拟”。]([http://stackoverflow.com/documentation/c%2B%2B/1752/virtual](http://stackoverflow.com/documentation/c%2B%2B/1752/virtual)-成员函数）在这种情况下，如果在指向实例的指针或引用上调用，则不会直接访问它们；相反，他们将在虚拟函数表（指向虚拟函数的成员函数的指针列表，通常称为“vtable”或“vftable”）中查找函数，并使用它来调用适合实例的动态（实际）类型的版本。如果从类的变量直接调用函数，则不执行查找。

```cpp
struct Base {
    virtual void func() { std::cout << "In Base." << std::endl; }
};

struct Derived : Base {
    void func() override { std::cout << "In Derived." << std::endl; }
};

void slicer(Base x) { x.func(); }

// ...

Base b;
Derived d;

Base *pb = &b, *pd = &d; // Pointers.
Base &rb = b, &rd = d;   // References.

b.func();   // Output:  In Base.
d.func();   // Output:  In Derived.

pb->func(); // Output:  In Base.
pd->func(); // Output:  In Derived.

rb.func();  // Output:  In Base.
rd.func();  // Output:  In Derived.

slicer(b);  // Output:  In Base.
slicer(d);  // Output:  In Base.

```

Note that while `pd` is `Base*`, and `rd` is a `Base&`, calling `func()` on either of the two calls `Derived::func()` instead of `Base::func()`; this is because the `vtable` for `Derived` updates the `Base::func()` entry to instead point to `Derived::func()`.  Conversely, note how passing an instance to `slicer()` always results in `Base::func()` being called, even when the passed instance is a `Derived`; this is because of something known as **data slicing**, where passing a `Derived` instance into a `Base` parameter by value renders the portion of the `Derived` instance that isn't a `Base` instance inaccessible.

> 请注意，当“pd”是“Base*”，“rd”是“Base&”时，在两个调用中的任何一个调用“Derived:：func（）”而不是“Base:：func”（）；这是因为“Derived”的“vtable”更新了“Base:：func（）”条目，改为指向“Derived:：func）”。相反，请注意，将实例传递给“slicer（）”总是会导致调用“Base:：func（）”，即使传递的实例是“Derived”；这是因为被称为**数据切片**的东西，其中按值将“Derived”实例传递到“Base”参数会使“Deriveed”实例中非“Base”实例的部分不可访问。

When a member function is defined as virtual, all derived class member functions with the same signature override it, regardless of whether the overriding function is specified as `virtual` or not.  This can make derived classes harder for programmers to parse, however, as there's no indication as to which function(s) is/are `virtual`.

> 当一个成员函数被定义为 virtual 时，所有具有相同签名的派生类成员函数都会覆盖它，而不管覆盖函数是否被指定为“virtual”。然而，这可能会使程序员更难解析派生类，因为没有指示哪些函数是“虚拟的”。

```cpp
struct B {
    virtual void f() {}
};

struct D : B {
    void f() {} // Implicitly virtual, overrides B::f.
                //  You'd have to check B to know that, though.
};

```

Note, however, that a derived function only overrides a base function if their signatures match; even if a derived function is explicitly declared `virtual`, it will instead create a new virtual function if the signatures are mismatched.

> 但是，请注意，派生函数只有在签名匹配的情况下才会覆盖基函数；即使派生函数被明确声明为“虚拟”，如果签名不匹配，它也会创建一个新的虚拟函数。

```cpp
struct BadB {
    virtual void f() {}
};

struct BadD : BadB {
    virtual void f(int i) {} // Does NOT override BadB::f.
};

```

As of C++11, intent to override can be made explicit with the context-sensitive keyword `override`.  This tells the compiler that the programmer expects it to override a base class function, which causes the compiler to omit an error if it **doesn't** override anything.

> 从 C++11 开始，覆盖的意图可以用上下文相关的关键字“override”来明确表示。这告诉编译器程序员希望它重写基类函数，这会导致编译器在**不重写任何内容的情况下忽略错误。

```cpp
struct CPP11B {
    virtual void f() {}
};

struct CPP11D : CPP11B {
    void f() override {}
    void f(int i) override {} // Error: Doesn't actually override anything.
};

```

This also has the benefit of telling programmers that the function is both virtual, and also declared in at least one base class, which can make complex classes easier to parse.

> 这还有一个好处，告诉程序员该函数既是虚拟的，也是在至少一个基类中声明的，这可以使复杂的类更容易解析。

When a function is declared `virtual`, and defined outside the class definition, the `virtual` specifier must be included in the function declaration, and not repeated in the definition.

> 当函数被声明为“virtual”，并且在类定义之外定义时，“virtual“说明符必须包含在函数声明中，而不能在定义中重复。

This also holds true for `override`.

```cpp
struct VB {
    virtual void f(); // "virtual" goes here.
    void g();
};
/* virtual */ void VB::f() {} // Not here.
virtual void VB::g() {} // Error.

```

If a base class overloads a `virtual` function, only overloads that are explicitly specified as `virtual` will be virtual.

> 如果基类重载“virtual”函数，则只有明确指定为“virtual“的重载才会是 virtual。

```cpp
struct BOverload {
    virtual void func() {}
    void func(int) {}
};

struct DOverload : BOverload {
    void func() override {}
    void func(int) {}
};

// ...

BOverload* bo = new DOverload;
bo->func(); // Calls DOverload::func().
bo->func(1); // Calls BOverload::func(int).

```

For more information, see [the relevant topic](http://stackoverflow.com/documentation/c%2B%2B/1752/virtual-member-functions).

> 有关更多信息，请参阅[相关主题]([http://stackoverflow.com/documentation/c%2B%2B/1752/virtual](http://stackoverflow.com/documentation/c%2B%2B/1752/virtual)-成员功能）。

## Const Correctness

One of the primary uses for `this` cv-qualifiers is [**`const` correctness**](http://stackoverflow.com/documentation/c%2B%2B/7217/const-correctness#t=201610191431578985982).  This is the practice of guaranteeing that only accesses that **need** to modify an object are **able** to modify the object, and that any (member or non-member) function that doesn't need to modify an object doesn't have write access to that object (whether directly or indirectly).  This prevents unintentional modifications, making code less errorprone.  It also allows any function that doesn't need to modify state to be able to take either a `const` or non-`const` object, without needing to rewrite or overload the function.

> “this”cv 限定符的主要用途之一是[**`const` corrective**]([http://stackoverflow.com/documentation/c%2B%2B/7217/const](http://stackoverflow.com/documentation/c%2B%2B/7217/const)-正确性#t=201610191431578985982）。这是一种实践，保证只有**需要**修改对象的访问才**能够**修改对象，并且任何不需要修改对象的（成员或非成员）函数都没有对该对象的写访问权限（无论是直接还是间接）。这样可以防止无意的修改，减少代码出错的可能性。它还允许任何不需要修改状态的函数接受“const”或非“const’对象，而无需重写或重载函数。

`const` correctness, due to its nature, starts at the bottom up: Any class member function that doesn't need to change state is [declared as `const`](http://stackoverflow.com/documentation/c%2B%2B/7146/the-this-pointer/24492/this-pointer-cv-qualifiers#t=201610191432572968878), so that it can be called on `const` instances.  This, in turn, allows passed-by-reference parameters to be declared `const` when they don't need to be modified, which allows functions to take either `const` or non-`const` objects without complaining, and `const`-ness can propagate outwards in this manner.  Due to this, getters are frequently `const`, as are any other functions that don't need to modify logical state.

```cpp
class ConstIncorrect {
    Field fld;

  public:
    ConstIncorrect(const Field& f) : fld(f) {}     // Modifies.

    const Field& get_field()       { return fld; } // Doesn't modify; should be const.
    void set_field(const Field& f) { fld = f; }    // Modifies.

    void do_something(int i) {                     // Modifies.
        fld.insert_value(i);
    }
    void do_nothing()        { }                   // Doesn't modify; should be const.
};

class ConstCorrect {
    Field fld;

  public:
    ConstCorrect(const Field& f) : fld(f) {}       // Not const: Modifies.

    const Field& get_field() const { return fld; } // const: Doesn't modify.
    void set_field(const Field& f) { fld = f; }    // Not const: Modifies.

    void do_something(int i) {                     // Not const: Modifies.
        fld.insert_value(i);
    }
    void do_nothing() const  { }                   // const: Doesn't modify.
};

// ...

const ConstIncorrect i_cant_do_anything(make_me_a_field());
// Now, let's read it...
Field f = i_cant_do_anything.get_field();
  // Error: Loses cv-qualifiers, get_field() isn't const.
i_cant_do_anything.do_nothing();
  // Error: Same as above.
// Oops.

const ConstCorrect but_i_can(make_me_a_field());
// Now, let's read it...
Field f = but_i_can.get_field(); // Good.
but_i_can.do_nothing();          // Good.

```

As illustrated by the comments on `ConstIncorrect` and `ConstCorrect`, properly cv-qualifying functions also serves as documentation.  If a class is `const` correct, any function that isn't `const` can safely be assumed to change state, and any function that is `const` can safely be assumed not to change state.

> 如“ConstIncorrect”和“ConstCorrect”的注释所示，适当的 cv 限定函数也可作为文档。如果一个类是“const”正确的，则可以安全地假设任何不是“const“的函数更改状态，并且可以安全地假定任何是“const”的函数不更改状态。

#### Syntax

<li>
// Calling:
<ul>
- variable.member_function();
- variable_pointer->member_function();

// Definition:

<li>ret_type class_name::member_function() cv-qualifiers {
<ul>
- body;

- }

// Prototype:

<li>class class_name {
<ul>
- virt-specifier ret_type member_function() cv-qualifiers virt-specifier-seq;
- // virt-specifier: "virtual", if applicable.
- // cv-qualifiers: "const" and/or "volatile", if applicable.
- // virt-specifier-seq: "override" and/or "final", if applicable.

- }

#### Remarks

A non-`static` member function is a [`class`/`struct`/`union`](http://stackoverflow.com/documentation/c%2B%2B/508/classes-structures) member function, which is called on a particular instance, and operates on said instance.  Unlike `static` member functions, it cannot be called without specifying an instance.

> 非“static”成员函数是[`class`/`struct`/`union`]([http://stackoverflow.com/documentation/c%2B%2B/508/classes-structures](http://stackoverflow.com/documentation/c%2B%2B/508/classes-structures)）成员函数，该函数在特定实例上被调用，并在所述实例上操作。与“static”成员函数不同，不能在不指定实例的情况下调用它。

For information on classes, structures, and unions, please see [the parent topic](http://stackoverflow.com/documentation/c%2B%2B/508/classes-structures).

> 有关类、结构和联合的信息，请参阅[父主题]([http://stackoverflow.com/documentation/c%2B%2B/508/classes](http://stackoverflow.com/documentation/c%2B%2B/508/classes)-结构）。
