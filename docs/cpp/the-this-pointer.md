---
tip: translate by baidu@2023-10-25 08:34:34
---
---

metaTitle: "C++ | The This Pointer"
description: "this Pointer, Using the this Pointer to Access Member Data, Using the this Pointer to Differentiate Between Member Data and Parameters, this Pointer CV-Qualifiers, this Pointer Ref-Qualifiers"
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# The This Pointer

## this Pointer

All non-static member functions have a hidden parameter, a pointer to an instance of the class, named `this`; this parameter is silently inserted at the beginning of the parameter list, and handled entirely by the compiler.  When a member of the class is accessed inside a member function, it is silently accessed through `this`; this allows the compiler to use a single non-static member function for all instances, and allows a member function to call other member functions polymorphically.

> 所有非静态成员函数都有一个隐藏参数，一个指向类实例的指针，名为“this”；该参数被静默地插入到参数列表的开头，并完全由编译器处理。当在成员函数内部访问类的成员时，它将通过“this”进行静默访问；这允许编译器对所有实例使用一个非静态成员函数，并允许成员函数以多态方式调用其他成员函数。

```cpp
struct ThisPointer {
    int i;

    ThisPointer(int ii);

    virtual void func();

    int  get_i() const;
    void set_i(int ii);
};
ThisPointer::ThisPointer(int ii) : i(ii) {}
// Compiler rewrites as:
ThisPointer::ThisPointer(int ii) : this->i(ii) {}
// Constructor is responsible for turning allocated memory into 'this'.
// As the constructor is responsible for creating the object, 'this' will not be "fully"
// valid until the instance is fully constructed.

/* virtual */ void ThisPointer::func() {
    if (some_external_condition) {
        set_i(182);
    } else {
        i = 218;
    }
}
// Compiler rewrites as:
/* virtual */ void ThisPointer::func(ThisPointer* this) {
    if (some_external_condition) {
        this->set_i(182);
    } else {
        this->i = 218;
    }
}

int  ThisPointer::get_i() const { return i; }
// Compiler rewrites as:
int  ThisPointer::get_i(const ThisPointer* this) { return this->i; }

void ThisPointer::set_i(int ii) { i = ii; }
// Compiler rewrites as:
void ThisPointer::set_i(ThisPointer* this, int ii) { this->i = ii; }

```

In a constructor, `this` can safely be used to (implicitly or explicitly) access any field that has already been initialised, or any field in a parent class; conversely, (implicitly or explicitly) accessing any fields that haven't yet been initialised, or any fields in a derived class, is unsafe (due to the derived class not yet being constructed, and thus its fields neither being initialised nor existing).  It is also unsafe to call virtual member functions through `this` in the constructor, as any derived class functions will not be considered (due to the derived class not yet being constructed, and thus its constructor not yet updating the vtable).

> 在构造函数中，“this”可以安全地用于（隐式或显式）访问任何已经初始化的字段，或父类中的任何字段；相反，（隐式或显式）访问任何尚未初始化的字段或派生类中的任何字段是不安全的（因为派生类尚未构造，因此其字段既没有初始化也不存在）。通过构造函数中的“this”调用虚拟成员函数也是不安全的，因为任何派生类函数都不会被考虑（因为派生类尚未被构造，因此其构造函数尚未更新 vtable）。

Also note that while in a constructor, the type of the object is the type which that constructor constructs.  This holds true even if the object is declared as a derived type.  For example, in the below example, `ctd_good` and `ctd_bad` are type `CtorThisBase` inside `CtorThisBase()`, and type `CtorThis` inside `CtorThis()`, even though their canonical type is `CtorThisDerived`.  As the more-derived classes are constructed around the base class, the instance gradually goes through the class hierarchy until it is a fully-constructed instance of its intended type.

> 还要注意，在构造函数中，对象的类型是该构造函数构造的类型。即使对象被声明为派生类型，这种情况也适用。例如，在下面的示例中，“ctd_good”和“ctd_bad”在“CtorThisBase（）”内部是类型“CtorTheBase'”，在“Ctor This（）”内是类型“CtorThis”，即使它们的规范类型是“Ctor 蓟派生的”。随着更多派生类围绕基类构建，实例逐渐通过类层次结构，直到它成为其预期类型的完全构建实例。

```cpp
class CtorThisBase {
    short s;

  public:
    CtorThisBase() : s(516) {}
};

class CtorThis : public CtorThisBase {
    int i, j, k;

  public:
    // Good constructor.
    CtorThis() : i(s + 42), j(this->i), k(j) {}

    // Bad constructor.
    CtorThis(int ii) : i(ii), j(this->k), k(b ? 51 : -51) {
        virt_func();
    }

    virtual void virt_func() { i += 2; }
};

class CtorThisDerived : public CtorThis {
    bool b;

  public:
    CtorThisDerived()       : b(true) {}
    CtorThisDerived(int ii) : CtorThis(ii), b(false) {}

    void virt_func() override { k += (2 * i); }
};

// ...

CtorThisDerived ctd_good;
CtorThisDerived ctd_bad(3);

```

With these classes and member functions:

<li>In the good constructor, for `ctd_good`:
<ul>

- `CtorThisBase` is fully constructed by the time the `CtorThis` constructor is entered.  Therefore, `s` is in a valid state while initialising `i`, and can thus be accessed.

> -在输入“CtorThis”构造函数时，“CtorTheBase”已完全构造完成。因此，在初始化“i”时，“s”处于有效状态，因此可以访问。

- `i` is initialised before `j(this->i)` is reached.  Therefore, `i` is in a valid state while initialising `j`, and can thus be accessed.

> -在到达“j（this->i）”之前初始化“i”。因此，“i”在初始化“j”时处于有效状态，因此可以访问。

- `j` is initialised before `k(j)` is reached.  Therefore, `j` is in a valid state while initialising `k`, and can thus be accessed.

> -在到达“k（j）”之前对“j”进行初始化。因此，“j”在初始化“k”时处于有效状态，因此可以被访问。

- `k` is initialised after `j(this->k)` is reached.  Therefore, `k` is in an invalid state while initialising `j`, and accessing it causes undefined behaviour.

> -在达到“j（this->k）”之后初始化“k”。因此，“k”在初始化“j”时处于无效状态，访问它会导致未定义的行为。

- `CtorThisDerived` is not constructed until after `CtorThis` is constructed.  Therefore, `b` is in an invalid state while initialising `k`, and accessing it causes undefined behaviour.

> -“CtorThisDerived”直到构造完“CtorThis”之后才构造。因此，“b”在初始化“k”时处于无效状态，访问它会导致未定义的行为。

- The object `ctd_bad` is still a `CtorThis` until it leaves `CtorThis()`, and will not be updated to use `CtorThisDerived`'s vtable until `CtorThisDerived()`.  Therefore, `virt_func()` will call `CtorThis::virt_func()`, regardless of whether it is intended to call that or `CtorThisDerived::virt_func()`.

> -对象“ctd_bad”在离开“CtorThis（）”之前仍然是“CtorThis”，并且在“CtorTheDerived（）”之后才会更新为使用“Ctor ThisDerived”的 vtable。因此，“virt_func（）”将调用“CtorThis:：virt_fun（）”，而不管它是要调用它还是要调用“Ctor ThisDerived:：virt_function（）”。

## Using the this Pointer to Access Member Data

In this context, using the `this` pointer isn't entirely necessary, but it will make your code clearer to the reader, by indicating that a given function or variable is a member of the class. An example in this situation:

> 在这种情况下，使用“this”指针并不是完全必要的，但它会通过指示给定的函数或变量是类的成员，使代码对读者更清晰。这种情况下的一个示例：

```cpp
// Example for this pointer
#include <iostream>
#include <string>

using std::cout;
using std::endl;

class Class
{
  public:
    Class();
    ~Class();
    int getPrivateNumber () const;
  private:
    int private_number = 42;
};

Class::Class(){}
Class::~Class(){}

int Class::getPrivateNumber() const
{
    return this->private_number;
}

int main()
{
    Class class_example;
    cout << class_example.getPrivateNumber() << endl;
}

```

See it in action [here](http://cpp.sh/9flka).

## Using the this Pointer to Differentiate Between Member Data and Parameters

This is an actual useful strategy to differentiate member data from parameters... Lets take this example :

> 这是区分成员数据和参数的一种实际有用的策略。。。举个例子：

```cpp
// Dog Class Example
#include <iostream>
#include <string>

using std::cout;
using std::endl;

/*
* @class Dog
*   @member name
*       Dog's name
*   @function bark
*       Dog Barks!
*   @function getName
*       To Get Private
*       Name Variable
*/
class Dog
{
 public:
    Dog(std::string name);
    ~Dog();
    void  bark() const;
    std::string  getName() const;
 private:
    std::string name;
};

Dog::Dog(std::string name)
{
    /*
    *  this->name is the
    *  name variable from 
    *  the class dog . and
    *  name is from the 
    *  parameter of the function
    */
    this->name = name; 
}

Dog::~Dog(){}

void Dog::bark() const
{
  cout << "BARK" << endl;   
}

std::string  Dog::getName() const
{
    return this->name;
}


int main()
{
    Dog dog("Max");
    cout << dog.getName() << endl;
    dog.bark();
}

```

You can see here in the constructor we execute the following:

```cpp
this->name = name; 

```

Here , you can see we are assinging the parameter name to the name of the private variable from the class Dog(this->name) .

> 在这里，您可以看到我们正在将参数名称与类 Dog 中的私有变量的名称进行分类（this->name）。

To see the output of above code : [http://cpp.sh/75r7](http://cpp.sh/75r7)

> 要查看以上代码的输出：[http://cpp.sh/75r7](http://cpp.sh/75r7)

## this Pointer CV-Qualifiers

`this` can also be cv-qualified, the same as any other pointer.  However, due to the `this` parameter not being listed in the parameter list, special syntax is required for this; the cv-qualifiers are listed after the parameter list, but before the function's body.

```cpp
struct ThisCVQ {
    void no_qualifier()                {} // "this" is: ThisCVQ*
    void  c_qualifier() const          {} // "this" is: const ThisCVQ*
    void  v_qualifier() volatile       {} // "this" is: volatile ThisCVQ*
    void cv_qualifier() const volatile {} // "this" is: const volatile ThisCVQ*
};

```

As `this` is a parameter, a [function can be overloaded based on its `this` cv-qualifier(s)](http://stackoverflow.com/documentation/c%2B%2B/510/function-overloading/11242/member-function-cv-qualifier-overloading#t=201610191448370301667).

> 由于“this”是一个参数，[函数可以基于其“this”cv 限定符重载]([http://stackoverflow.com/documentation/c%2B%2B/510/function](http://stackoverflow.com/documentation/c%2B%2B/510/function)-重载/11242/成员函数 cv 限定符重载#t=201610191448370301667）。

```cpp
struct CVOverload {
    int func()                { return    3; }
    int func() const          { return   33; }
    int func() volatile       { return  333; }
    int func() const volatile { return 3333; }
};

```

When `this` is `const` (including `const volatile`), the function is unable to write to member variables through it, whether implicitly or explicitly.  The sole exception to this is [`mutable` member variables](http://stackoverflow.com/documentation/c%2B%2B/2705/mutable-keyword/9058/non-static-class-member-modifier#t=201610111819273885699), which can be written regardless of const-ness.  Due to this, `const` is used to indicate that the member function doesn't change the object's logical state (the way the object appears to the outside world), even if it does modify the physical state (the way the object looks under the hood).

> 当“this”是“const”（包括“const volatile”）时，函数无法通过它写入成员变量，无论是隐式还是显式。唯一的例外是[`mutable` 成员变量]([http://stackoverflow.com/documentation/c%2B%2B/2705/mutable](http://stackoverflow.com/documentation/c%2B%2B/2705/mutable)-关键字/9058/非静态类成员修饰符#t=201610111819273885699），无论常量如何都可以写入。因此，“const”用于表示成员函数不会改变对象的逻辑状态（对象在外部世界中的外观），即使它确实修改了物理状态（对象看起来像是在引擎盖下）。

>

Logical state is the way the object appears to outside observers.  It isn't directly tied to physical state, and indeed, might not even be stored as physical state.  As long as outside observers can't see any changes, the logical state is constant, even if you flip every single bit in the object.

> 逻辑状态是对象在外部观察者看来的方式。它与物理状态没有直接联系，事实上，甚至可能不会作为物理状态存储。只要外部观察者看不到任何变化，即使翻转对象中的每一位，逻辑状态也是不变的。

>

Physical state, also known as bitwise state, is how the object is stored in memory.  This is the object's nitty-gritty, the raw 1s and 0s that make up its data.  An object is only physically constant if its representation in memory never changes.

> 物理状态，也称为位状态，是对象在内存中的存储方式。这就是对象的本质，构成其数据的原始 1 和 0。只有当对象在内存中的表示从未改变时，对象在物理上才是恒定的。

Note that C++ bases `const` ness on logical state, not physical state.

```cpp
class DoSomethingComplexAndOrExpensive {
    mutable ResultType cached_result;
    mutable bool state_changed;

    ResultType calculate_result();
    void modify_somehow(const Param& p);

    // ...

  public:
    DoSomethingComplexAndOrExpensive(Param p) : state_changed(true) {
        modify_somehow(p);
    }

    void change_state(Param p) {
        modify_somehow(p);
        state_changed = true;
    }

    // Return some complex and/or expensive-to-calculate result.
    // As this has no reason to modify logical state, it is marked as "const".
    ResultType get_result() const;
};
ResultType DoSomethingComplexAndOrExpensive::get_result() const {
    // cached_result and state_changed can be modified, even with a const "this" pointer.
    // Even though the function doesn't modify logical state, it does modify physical state
    //  by caching the result, so it doesn't need to be recalculated every time the function
    //  is called.  This is indicated by cached_result and state_changed being mutable.

    if (state_changed) {
        cached_result = calculate_result();
        state_changed = false;
    }

    return cached_result;
}

```

Note that while you technically **could** use `const_cast` on `this` to make it non-cv-qualified, you really, ****REALLY**** shouldn't, and should use `mutable` instead.  A `const_cast` is liable to invoke undefined behaviour when used on an object that actually **is** `const`, while `mutable` is designed to be safe to use.  It is, however, possible that you may run into this in extremely old code.

> 请注意，虽然从技术上讲，**可以在“this”上使用“const_cast”来使其不符合 cv 条件，但实际上，***实际上***不应该，应该使用“mutable”。当“const_cast”用于一个实际上**是“const”的对象时，它很容易调用未定义的行为，而“mutable”则是为了安全使用而设计的。然而，您可能会在非常旧的代码中遇到这种情况。

[An exception to this rule is defining non-cv-qualified accessors in terms of `const` accessors; as the object is guaranteed to not be `const` if the non-cv-qualified version is called, there's no risk of UB.](http://stackoverflow.com/documentation/c%2B%2B/2386/const-keyword/16974/avoiding-duplication-of-code-in-const-and-non-const-getter-methods)

> [此规则的一个例外是根据“const”访问器定义非 cv 限定的访问器；由于如果调用非 cv 限制的版本，则对象保证不是“const’，因此不存在 UB 风险。]([http://stackoverflow.com/documentation/c%2B%2B/2386/const](http://stackoverflow.com/documentation/c%2B%2B/2386/const)-关键字/16974/避免常量和非常量 getter 方法中的代码重复）

```cpp
class CVAccessor {
    int arr[5];

  public:
    const int& get_arr_element(size_t i) const { return arr[i]; }

    int& get_arr_element(size_t i) {
        return const_cast<int&>(const_cast<const CVAccessor*>(this)->get_arr_element(i));
    }
};

```

This prevents unnecessary duplication of code.

As with regular pointers, if `this` is `volatile` (including `const volatile`), it is loaded from memory each time it is accessed, instead of being cached.  This has the same effects on optimisation as declaring any other pointer `volatile` would, so care should be taken.

> 与常规指针一样，如果“this”是“volatile”（包括“const-volatile”），则每次访问时都会从内存加载，而不是缓存。这对优化的影响与声明任何其他指针“volatile”的效果相同，因此应该小心。

Note that if an instance is cv-qualified, the only member functions it is allowed to access are member functions whose `this` pointer is at least as cv-qualified as the instance itself:

> 请注意，如果实例是 cv 限定的，那么它被允许访问的唯一成员函数是其“this”指针至少与实例本身一样具有 cv 限定性的成员函数：

- Non-cv instances can access any member functions.
- `const` instances can access `const` and `const volatile` functions.
- `volatile` instances can access `volatile` and `const volatile` functions.
- `const volatile` instances can access `const volatile` functions.

This is one of the key tenets of [`const` correctness](http://stackoverflow.com/documentation/c%2B%2B/7217/const-correctness#t=201610111815341169566).

> 这是[`const` corrective]的关键原则之一([http://stackoverflow.com/documentation/c%2B%2B/7217/const](http://stackoverflow.com/documentation/c%2B%2B/7217/const)-正确性#t=201610111815341169566）。

```cpp
struct CVAccess {
    void    func()                {}
    void  func_c() const          {}
    void  func_v() volatile       {}
    void func_cv() const volatile {}
};

CVAccess cva;
cva.func();    // Good.
cva.func_c();  // Good.
cva.func_v();  // Good.
cva.func_cv(); // Good.

const CVAccess c_cva;
c_cva.func();    // Error.
c_cva.func_c();  // Good.
c_cva.func_v();  // Error.
c_cva.func_cv(); // Good.

volatile CVAccess v_cva;
v_cva.func();    // Error.
v_cva.func_c();  // Error.
v_cva.func_v();  // Good.
v_cva.func_cv(); // Good.

const volatile CVAccess cv_cva;
cv_cva.func();    // Error.
cv_cva.func_c();  // Error.
cv_cva.func_v();  // Error.
cv_cva.func_cv(); // Good.

```

## this Pointer Ref-Qualifiers

Similarly to `this` cv-qualifiers, we can also apply **ref-qualifiers** to `*this`.  Ref-qualifiers are used to choose between normal and rvalue reference semantics, allowing the compiler to use either copy or move semantics depending on which are more appropriate, and are applied to `*this` instead of `this`.

> 类似于“this”cv 限定符，我们也可以将 **ref 限定符**应用于“*this”。引用限定符用于在正常引用语义和右值引用语义之间进行选择，允许编译器根据更合适的语义使用复制或移动语义，并应用于“*this”而不是“this”。

Note that despite ref-qualifiers using reference syntax, `this` itself is still a pointer.  Also note that ref-qualifiers don't actually change the type of `*this`; it's just easier to describe and understand their effects by looking at them as if they did.

> 请注意，尽管 ref 限定符使用引用语法，但“this”本身仍然是一个指针。还要注意，ref 限定符实际上并不会更改“*this”的类型；通过观察它们的效果，更容易描述和理解它们的效果。

```cpp
struct RefQualifiers {
    std::string s;

    RefQualifiers(const std::string& ss = "The nameless one.") : s(ss) {}

    // Normal version.
    void func() &  { std::cout << "Accessed on normal instance "    << s << std::endl; }
    // Rvalue version.
    void func() && { std::cout << "Accessed on temporary instance " << s << std::endl; }

    const std::string& still_a_pointer() &  { return this->s; }
    const std::string& still_a_pointer() && { this->s = "Bob"; return this->s; }
};

// ...

RefQualifiers rf("Fred");
rf.func();              // Output:  Accessed on normal instance Fred
RefQualifiers{}.func(); // Output:  Accessed on temporary instance The nameless one

```

A member function cannot have overloads both with and without ref-qualifiers; the programmer has to choose between one or the other.  Thankfully, cv-qualifiers can be used in conjunction with ref-qualifiers, allowing [`const` correctness](http://stackoverflow.com/documentation/c%2B%2B/7217/const-correctness#t=201610111809376589726) rules to be followed.

> 一个成员函数不能同时具有带有和不带有 ref 限定符的重载；程序员必须在两者之间做出选择。值得庆幸的是，cv 限定符可以与 ref 限定符一起使用，允许[`const` corrective]([http://stackoverflow.com/documentation/c%2B%2B/7217/const](http://stackoverflow.com/documentation/c%2B%2B/7217/const)-正确性#t=201610111809376589726）规则。

```cpp
struct RefCV {
    void func() &                {}
    void func() &&               {}
    void func() const&           {}
    void func() const&&          {}
    void func() volatile&        {}
    void func() volatile&&       {}
    void func() const volatile&  {}
    void func() const volatile&& {}
};

```

#### Remarks

The `this` pointer is a keyword for C++ therfore there is no library needed to implement this. And do not forget `this` is a pointer! So you cannot do:

> “this”指针是 C++ 的关键字，因此不需要库来实现它。别忘了“这”是一个指针！所以你不能做：

```

this.someMember();

```

As you access member functions or member variables from pointers using the arrow symbol    `->` :

> 当您使用箭头符号 `->` 从指针访问成员函数或成员变量时：

```cpp
this->someMember();

```

Other helpful links to the better understanding of the `this` pointer :

[What is the &#39;this&#39; pointer?](http://stackoverflow.com/questions/16492736/what-is-the-this-pointer)

> [这个指针是什么？](http://stackoverflow.com/questions/16492736/what-is-the-this-pointer)

[http://www.geeksforgeeks.org/this-pointer-in-c/](http://www.geeksforgeeks.org/this-pointer-in-c/)

> [http://www.geeksforgeeks.org/this-pointer-in-c/](http://www.geeksforgeeks.org/this-pointer-in-c/)

[https://www.tutorialspoint.com/cplusplus/cpp_this_pointer.htm](https://www.tutorialspoint.com/cplusplus/cpp_this_pointer.htm)

> [https://www.tutorialspoint.com/cplusplus/cpp_this_pointer.htm](https://www.tutorialspoint.com/cplusplus/cpp_this_pointer.htm)
