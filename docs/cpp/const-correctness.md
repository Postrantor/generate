---
tip: translate by baidu@2023-10-25 08:23:00
---
---
metaTitle: "C++ | Const Correctness"
description: "The Basics, Const Correct Class Design, Const Correct Function Parameters, Const Correctness as Documentation"
---

# Const Correctness



## The Basics



**`const` correctness** is the practice of designing code so that only code that **needs** to modify an instance is **able** to modify an instance (i.e. has write access), and conversely, that any code that doesn't need to modify an instance is unable to do so (i.e. only has read access).  This prevents the instance from being modified unintentionally, making code less errorprone, and documents whether the code is intended to change the instance's state or not.  It also allows instances to be treated as `const` whenever they don't need to be modified, or defined as `const` if they don't need to be changed after initialisation, without losing any functionality.

> **`const`corrective**是一种设计代码的做法，这样只有**需要**修改实例的代码**才能**修改实例（即具有写访问权限），相反，任何不需要修改实例的程序都无法修改（即仅具有读访问权限）。这可以防止实例被无意修改，从而减少代码出错的可能性，并记录代码是否旨在更改实例的状态。它还允许实例在不需要修改时被视为“const”，或者在初始化后不需要更改时被定义为“const”，而不会丢失任何功能。


This is done by giving member functions [`const` CV-qualifiers](http://stackoverflow.com/documentation/c%2B%2B/7146/the-this-pointer/24492/this-pointer-cv-qualifiers#t=201610191445574282562), and by making pointer/reference parameters `const`, except in the case that they need write access.

> 这是通过赋予成员函数[`const`CV限定符]来实现的(http://stackoverflow.com/documentation/c%2B%2B/7146/the-this pointer/24492/this pointer cv限定符#t=201610191445574282562），并通过使指针/引用参数为“const”，除非它们需要写访问。

```cpp
class ConstCorrectClass {
    int x;

  public:
    int getX() const { return x; } // Function is const: Doesn't modify instance.
    void setX(int i) { x = i; }    // Not const: Modifies instance.
};

// Parameter is const: Doesn't modify parameter.
int const_correct_reader(const ConstCorrectClass& c) {
    return c.getX();
}

// Parameter isn't const: Modifies parameter.
void const_correct_writer(ConstCorrectClass& c) {
    c.setX(42);
}

const ConstCorrectClass invariant; // Instance is const: Can't be modified.
ConstCorrectClass         variant; // Instance isn't const: Can be modified.

// ...

const_correct_reader(invariant); // Good.   Calling non-modifying function on const instance.
const_correct_reader(variant);   // Good.   Calling non-modifying function on modifiable instance.

const_correct_writer(variant);   // Good.   Calling modifying function on modifiable instance.
const_correct_writer(invariant); // Error.  Calling modifying function on const instance.

```


Due to the nature of const correctness, this starts with the class' member functions, and works its way outwards; if you try to call a non-`const` member function from a `const` instance, or from a non-`const` instance being treated as `const`, the compiler will give you an error about it losing cv-qualifiers.

> 由于常量正确性的性质，它从类的成员函数开始，并向外扩展；如果您试图从“const”实例或从被视为“const’的非“const‘实例调用一个非“const”成员函数，编译器将为您提供一个丢失cv限定符的错误。



## Const Correct Class Design



In a `const`-correct class, all member functions which don't change logical state have `this` cv-qualified as `const`, indicating that they don't modify the object (apart from any [`mutable`](http://stackoverflow.com/documentation/c%2B%2B/2705/mutable-keyword/9058/non-static-class-member-modifier#t=201610041643529340028) fields, which can freely be modified even in `const` instances); if a `const` cv-qualified function returns a reference, that reference should also be `const`.  This allows them to be called on both constant and non-cv-qualified instances, as a `const T*` is capable of binding to either a `T*` or a `const T*`.  This, in turn, allows functions to declare their passed-by-reference parameters as `const` when they don't need to be modified, without losing any functionality.

> 在“const”-correct类中，所有不改变逻辑状态的成员函数都将“this”cv限定为“const“，表示它们不修改对象（除了任何[`mutable`](http://stackoverflow.com/documentation/c%2B%2B/2705/mutable-keyword/9058/非静态类成员修饰符#t=201610041643529340028）字段，即使在“const”实例中也可以自由修改）；如果一个“const”cv限定函数返回一个引用，那么该引用也应该是“const“。这允许在常量和非cv限定实例上调用它们，因为“const T*”能够绑定到“T*”或“const T*”。这反过来又允许函数在不需要修改时将其传递的引用参数声明为“const”，而不会丢失任何功能。


Furthermore, in a `const` correct class, all passed-by-reference function parameters will be `const` correct, as discussed in `Const Correct Function Parameters`, so that they can only be modified when the function explicitly **needs** to modify them.

> 此外，在“const”correct类中，所有通过引用传递的函数参数都将是“const’correct”，如“const correct function parameters”中所述，因此只有当函数明确**需要**修改它们时，才能修改它们。

First, let's look at `this` cv-qualifiers:

```cpp
// Assume class Field, with member function "void insert_value(int);".

class ConstIncorrect {
    Field fld;

  public:
    ConstIncorrect(Field& f); // Modifies.

    Field& getField();        // Might modify.  Also exposes member as non-const reference,
                              //  allowing indirect modification.
    void setField(Field& f);  // Modifies.

    void doSomething(int i);  // Might modify.
    void doNothing();         // Might modify.
};

ConstIncorrect::ConstIncorrect(Field& f) : fld(f) {} // Modifies.
Field& ConstIncorrect::getField() { return fld; }    // Doesn't modify.
void ConstIncorrect::setField(Field& f) { fld = f; } // Modifies.
void ConstIncorrect::doSomething(int i) {            // Modifies.
    fld.insert_value(i);
}
void ConstIncorrect::doNothing() {}                  // Doesn't modify.


class ConstCorrectCVQ {
    Field fld;

  public:
    ConstCorrectCVQ(Field& f);     // Modifies.

    const Field& getField() const; // Doesn't modify.  Exposes member as const reference,
                                   //  preventing indirect modification.
    void setField(Field& f);       // Modifies.

    void doSomething(int i);       // Modifies.
    void doNothing() const;        // Doesn't modify.
};

ConstCorrectCVQ::ConstCorrectCVQ(Field& f) : fld(f) {}
Field& ConstCorrectCVQ::getField() const { return fld; }
void ConstCorrectCVQ::setField(Field& f) { fld = f; }
void ConstCorrectCVQ::doSomething(int i) {
    fld.insert_value(i);
}
void ConstCorrectCVQ::doNothing() const  {}

// This won't work.
// No member functions can be called on const ConstIncorrect instances.
void const_correct_func(const ConstIncorrect& c) {
    Field f = c.getField();
    c.do_nothing();
}

// But this will.
// getField() and doNothing() can be called on const ConstCorrectCVQ instances.
void const_correct_func(const ConstCorrectCVQ& c) {
    Field f = c.getField();
    c.do_nothing();
}

```


We can then combine this with `Const Correct Function Parameters`, causing the class to be fully `const`-correct.

> 然后，我们可以将其与“Const Correct Function Parameters”组合，使类完全“Const”-Correct。

```cpp
class ConstCorrect {
    Field fld;

  public:
    ConstCorrect(const Field& f);  // Modifies instance.  Doesn't modify parameter.

    const Field& getField() const; // Doesn't modify.  Exposes member as const reference,
                                   //  preventing indirect modification.
    void setField(const Field& f); // Modifies instance.  Doesn't modify parameter.

    void doSomething(int i);       // Modifies.  Doesn't modify parameter (passed by value).
    void doNothing() const;        // Doesn't modify.
};

ConstCorrect::ConstCorrect(const Field& f) : fld(f) {}
Field& ConstCorrect::getField() const { return fld; }
void ConstCorrect::setField(const Field& f) { fld = f; }
void ConstCorrect::doSomething(int i) {
    fld.insert_value(i);
}
void ConstCorrect::doNothing() const {}

```


This can also be combined with overloading based on `const`ness, in the case that we want one behaviour if the instance is `const`, and a different behaviour if it isn't; a common use for this is constainers providing accessors that only allow modification if the container itself is non-`const`.

> 这也可以与基于“const”的重载相结合，在这种情况下，如果实例是“const“，我们想要一种行为，如果不是，我们想要另一种行为；这方面的一个常见用途是常量提供访问器，只有当容器本身是非常量时才允许修改。

```cpp
class ConstCorrectContainer {
    int arr[5];

  public:
    // Subscript operator provides read access if instance is const, or read/write access
    // otherwise.    
          int& operator[](size_t index)       { return arr[index]; }
    const int& operator[](size_t index) const { return arr[index]; }

    // ...
};

```


This is commonly used in the standard library, with most containers providing overloads to take `const`ness into account.

> 这通常在标准库中使用，大多数容器都提供重载以考虑“const”。



## Const Correct Function Parameters



In a `const`-correct function, all passed-by-reference parameters are marked as `const` unless the function directly or indirectly modifies them, preventing the programmer from inadvertently changing something they didn't mean to change.  This allows the function to take both `const` and non-cv-qualified instances, and in turn, causes the instance's `this` to be of type `const T*` when a member function is called, where `T` is the class' type.

> 在“const”-correct函数中，所有传递的引用参数都被标记为“const“，除非函数直接或间接地修改它们，以防止程序员无意中更改它们不想更改的内容。这允许函数同时使用“const”和非cv限定的实例，反过来，当调用成员函数时，会导致实例的“This”的类型为“const T*”，其中“T”是类的类型。

```cpp
struct Example {
    void func()       { std::cout << 3 << std::endl; }
    void func() const { std::cout << 5 << std::endl; }
};

void const_incorrect_function(Example& one, Example* two) {
    one.func();
    two->func();
}

void const_correct_function(const Example& one, const Example* two) {
    one.func();
    two->func();
}

int main() {
    Example a, b;
    const_incorrect_function(a, &b);
    const_correct_function(a, &b);
}

// Output:
3
3
5
5

```


While the effects of this are less immediately visible than those of `const` correct class design (in that `const`-correct functions and `const`-incorrect classes will cause compilation errors, while `const`-correct classes and `const`-incorrect functions will compile properly), `const` correct functions will catch a lot of errors that `const` incorrect functions would let slip through, such as the one below.  [Note, however, that a `const`-incorrect function **will** cause compilation errors if passed a `const` instance when it expected a non-`const` one.]

> 虽然这种方法的效果不如“const”correct类设计的效果那么明显（因为“const’-correct函数和“const‘-incorrect类将导致编译错误，而“const′-correct类和“const”-incor正确函数将正确编译），例如下面的一个。[但是，请注意，如果传递了一个“const”实例，而该实例需要一个非“const’实例，那么“const‘-不正确的函数**将导致编译错误。]

```cpp
// Read value from vector, then compute & return a value.
// Caches return values for speed.
template<typename T>
const T& bad_func(std::vector<T>& v, Helper<T>& h) {
    // Cache values, for future use.
    // Once a return value has been calculated, it's cached & its index is registered.
    static std::vector<T> vals = {};

    int v_ind = h.get_index();               // Current working index for v.
    int vals_ind = h.get_cache_index(v_ind); // Will be -1 if cache index isn't registered.

    if (vals.size() && (vals_ind != -1) && (vals_ind < vals.size()) && !(h.needs_recalc())) {
        return vals[h.get_cache_index(v_ind)];
    }

    T temp = v[v_ind];

    temp -= h.poll_device();
    temp *= h.obtain_random();
    temp += h.do_tedious_calculation(temp, v[h.get_last_handled_index()]);

    // We're feeling tired all of a sudden, and this happens.
    if (vals_ind != -1) {
        vals[vals_ind] = temp;
    } else {
        v.push_back(temp);  // Oops.  Should've been accessing vals.
        vals_ind = vals.size() - 1;
        h.register_index(v_ind, vals_ind);
    }

    return vals[vals_ind];
}

// Const correct version.  Is identical to above version, so most of it shall be skipped.
template<typename T>
const T& good_func(const std::vector<T>& v, Helper<T>& h) {
    // ...

    // We're feeling tired all of a sudden, and this happens.
    if (vals_ind != -1) {
        vals[vals_ind] = temp;
    } else {
        v.push_back(temp);  // Error: discards qualifiers.
        vals_ind = vals.size() - 1;
        h.register_index(v_ind, vals_ind);
    }

    return vals[vals_ind];
}

```



## Const Correctness as Documentation



One of the more useful things about `const` correctness is that it serves as a way of documenting code, providing certain guarantees to the programmer and other users.  These guarantees are enforced by the compiler due to `const`ness, with a lack of `const`ness in turn indicating that code doesn't provide them.

> “const”正确性的一个更有用的地方是，它可以作为一种记录代码的方式，为程序员和其他用户提供一定的保证。这些保证是由编译器由于“常量”而强制执行的，而缺少“常量”则表明代码没有提供这些保证。

### `const` CV-Qualified Member Functions:

<li>Any member function which is `const` can be assumed to have intent to read the instance, and:
<ul>

- Shall not modify the logical state of the instance they are called on.  Therefore, they shall not modify any member variables of the instance they are called on, except `mutable` variables.

> -不应修改被调用实例的逻辑状态。因此，除“可变”变量外，它们不应修改所调用实例的任何成员变量。

- Shall not call any **other** functions that would modify any member variables of the instance, except `mutable` variables.

> -不应调用任何**其他**函数来修改实例的任何成员变量，“可变”变量除外。

- May or may not modify logical state.
- May or may not call other functions which modify logical state.


This can be used to make assumptions about the state of the object after any given member function is called, even without seeing the definition of that function:

> 这可以用于在调用任何给定的成员函数后对对象的状态进行假设，即使没有看到该函数的定义：

```cpp
// ConstMemberFunctions.h

class ConstMemberFunctions {
    int val;
    mutable int cache;
    mutable bool state_changed;

  public:
    // Constructor clearly changes logical state.  No assumptions necessary.
    ConstMemberFunctions(int v = 0);

    // We can assume this function doesn't change logical state, and doesn't call
    //  set_val().  It may or may not call squared_calc() or bad_func().
    int calc() const;

    // We can assume this function doesn't change logical state, and doesn't call
    //  set_val().  It may or may not call calc() or bad_func().
    int squared_calc() const;

    // We can assume this function doesn't change logical state, and doesn't call
    //  set_val().  It may or may not call calc() or squared_calc().
    void bad_func() const;

    // We can assume this function changes logical state, and may or may not call
    //  calc(), squared_calc(), or bad_func().
    void set_val(int v);
};

```


Due to `const` rules, these assumptions will in fact be enforced by the compiler.

> 由于“const”规则，这些假设实际上将由编译器强制执行。

```cpp
// ConstMemberFunctions.cpp

ConstMemberFunctions::ConstMemberFunctions(int v /* = 0*/)
  : cache(0), val(v), state_changed(true) {}

// Our assumption was correct.
int ConstMemberFunctions::calc() const {
    if (state_changed) {
        cache = 3 * val;
        state_changed = false;
    }

    return cache;
}

// Our assumption was correct.
int ConstMemberFunctions::squared_calc() const {
    return calc() * calc();
}

// Our assumption was incorrect.
// Function fails to compile, due to `this` losing qualifiers.
void ConstMemberFunctions::bad_func() const {
    set_val(863);
}

// Our assumption was correct.
void ConstMemberFunctions::set_val(int v) {
    if (v != val) {
        val = v;
        state_changed = true;
    }
}

```

### `const` Function Parameters:

<li>Any function with one or more parameters which are `const` can be assumed to have intent to read those parameters, and:
<ul>
- Shall not modify those parameters, or call any member functions that would modify them.

- Shall not pass those parameters to any **other** function which would modify them and/or call any member functions that would modify them.

> -不应将这些参数传递给任何将修改它们的**其他**函数和/或调用任何将修改这些参数的成员函数。

- May or may not modify those parameters, or call any member functions which whould modify them.

- May or may not pass those parameters to other functions which would modify them and/or call any member functions that would modify them.

> -可以将这些参数传递给或不传递给将修改它们的其他函数和/或调用将修改这些参数的任何成员函数。


This can be used to make assumptions about the state of the parameters after being passed to any given function, even without seeing the definition of that function.

> 这可以用来对传递给任何给定函数后的参数状态进行假设，即使没有看到该函数的定义。

```cpp
// function_parameter.h

// We can assume that c isn't modified (and c.set_val() isn't called), and isn't passed
//  to non_qualified_function_parameter().  If passed to one_const_one_not(), it is the first
//  parameter.
void const_function_parameter(const ConstMemberFunctions& c);

// We can assume that c is modified and/or c.set_val() is called, and may or may not be passed
//  to any of these functions.  If passed to one_const_one_not, it may be either parameter.
void non_qualified_function_parameter(ConstMemberFunctions& c);

// We can assume that:
  // l is not modified, and l.set_val() won't be called.
  // l may or may not be passed to const_function_parameter().
  // r is modified, and/or r.set_val() may be called.
  // r may or may not be passed to either of the preceding functions.
void one_const_one_not(const ConstMemberFunctions& l, ConstMemberFunctions& r);

// We can assume that c isn't modified (and c.set_val() isn't called), and isn't passed
//  to non_qualified_function_parameter().  If passed to one_const_one_not(), it is the first
//  parameter.
void bad_parameter(const ConstMemberFunctions& c);

```


Due to `const` rules, these assumptions will in fact be enforced by the compiler.

> 由于“const”规则，这些假设实际上将由编译器强制执行。

```cpp
// function_parameter.cpp

// Our assumption was correct.
void const_function_parameter(const ConstMemberFunctions& c) {
    std::cout << "With the current value, the output is: " << c.calc() << '\n'
              << "If squared, it's: " << c.squared_calc()
              << std::endl;
}

// Our assumption was correct.
void non_qualified_function_parameter(ConstMemberFunctions& c) {
    c.set_val(42);
    std::cout << "For the value 42, the output is: " << c.calc() << '\n'
              << "If squared, it's: " << c.squared_calc()
              << std::endl;
}

// Our assumption was correct, in the ugliest possible way.
// Note that const correctness doesn't prevent encapsulation from intentionally being broken,
//  it merely prevents code from having write access when it doesn't need it.
void one_const_one_not(const ConstMemberFunctions& l, ConstMemberFunctions& r) {
    // Let's just punch access modifiers and common sense in the face here.
    struct Machiavelli {
        int val;
        int unimportant;
        bool state_changed;
    };
    reinterpret_cast<Machiavelli&>(r).val = l.calc();
    reinterpret_cast<Machiavelli&>(r).state_changed = true;

    const_function_parameter(l);
    const_function_parameter(r);
}

// Our assumption was incorrect.
// Function fails to compile, due to `this` losing qualifiers in c.set_val().
void bad_parameter(const ConstMemberFunctions& c) {
    c.set_val(18);
}

```


While it **is** possible to [circumvent `const` correctness](http://stackoverflow.com/documentation/c%2B%2B/4891/keywords/18511/const-cast#t=201610041626457103824), and by extension break these guarantees, this must be done intentionally by the programmer (just like breaking encapsulation with `Machiavelli`, above), and is likely to cause undefined behaviour.

> 虽然**有可能[规避常量正确性](http://stackoverflow.com/documentation/c%2B%2B/4891/keywords/18511/const-cast#t=201610041626457103824），通过扩展破坏这些保证，这必须由程序员有意完成（就像上面用“马基雅维利”破坏封装一样），并且可能导致未定义的行为。

```cpp
class DealBreaker : public ConstMemberFunctions {
  public:
    DealBreaker(int v = 0);

    // A foreboding name, but it's const...
    void no_guarantees() const;
}

DealBreaker::DealBreaker(int v /* = 0 */) : ConstMemberFunctions(v) {}

// Our assumption was incorrect.
// const_cast removes const-ness, making the compiler think we know what we're doing.
void DealBreaker::no_guarantees() const {
    const_cast<DealBreaker*>(this)->set_val(823);
}

// ...

const DealBreaker d(50);
d.no_guarantees(); // Undefined behaviour: d really IS const, it may or may not be modified.

```


However, due to this requiring the programmer to very specifically **tell** the compiler that they intend to ignore `const`ness, and being inconsistent across compilers, it is generally safe to assume that `const` correct code will refrain from doing so unless otherwise specified.

> 然而，由于这需要程序员非常明确地**告诉**编译器他们打算忽略“常量”，并且在编译器之间不一致，因此通常可以安全地假设“常量”正确代码不会这样做，除非另有规定。



#### Syntax


- class ClassOne { public: bool non_modifying_member_function() const { /* ... */ } };
- int ClassTwo::non_modifying_member_function() const { /* ... */ }
- void ClassTwo::modifying_member_function() { /* ... */ }
- char non_param_modding_func(const ClassOne& one, const ClassTwo* two) { /* ... */ }
- float parameter_modifying_function(ClassTwo& one, ClassOne* two) { /* ... */ }
- short ClassThree::non_modding_non_param_modding_f(const ClassOne&) const { /* ... */ }



#### Remarks


`const` correctness is a very useful troubleshooting tool, as it allows the programmer to quickly determine which functions might be inadvertently modifying code.  It also prevents unintentional errors, such as the one shown in `Const Correct Function Parameters`, from compiling properly and going unnoticed.


It is much easier to design a class for `const` correctness, than it is to later add `const` correctness to a pre-existing class.  If possible, design any class that **can** be `const` correct so that it **is** `const` correct, to save yourself and others the hassle of later modifying it.

> 为“const”正确性设计一个类要比以后将“const“正确性添加到预先存在的类中容易得多。如果可能的话，设计任何一个**可以**为“const”正确的类，使其**为“const”正确，这样可以省去您自己和他人以后修改它的麻烦。


Note that this can also be applied to `volatile` correctness if necessary, with the same rules as for `const` correctness, but this is used much less often.

> 请注意，如果必要，这也可以应用于“volatile”正确性，规则与“const”正确性相同，但使用频率要低得多。

Refrences :

[ISO_CPP](https://isocpp.org/wiki/faq/const-correctness)


[Sell me on const correctness](http://stackoverflow.com/questions/136880/sell-me-on-const-correctness)

> [向我推销常量正确性](http://stackoverflow.com/questions/136880/sell-me-on-const-correctness)


[C++ Tutorial](http://www.cprogramming.com/tutorial/const_correctness.html)

> [C++教程](http://www.cprogramming.com/tutorial/const_correctness.html)

