---
tip: translate by baidu@2023-10-25 08:23:08
---
---

metaTitle: "C++ | const keyword"
description: "Const local variables, Const pointers, Const member functions, Avoiding duplication of code in const and non-const getter methods."
-------------------------------------------------------------------------------------------------------------------------------------------------

# const keyword

## Const local variables

Declaration and usage.

```cpp
// a is const int, so it can't be changed
const int a = 15;  
a = 12;           // Error: can't assign new value to const variable
a += 1;           // Error: can't assign new value to const variable

```

Binding of references and pointers

```cpp
int &b = a;       // Error: can't bind non-const reference to const variable
const int &c = a; // OK; c is a const reference

int *d = &a;      // Error: can't bind pointer-to-non-const to const variable
const int *e = &a // OK; e is a pointer-to-const

int f = 0;
e = &f;           // OK; e is a non-const pointer-to-const,
                  // which means that it can be rebound to new int* or const int*

*e = 1            // Error: e is a pointer-to-const which means that
                  // the value it points to can't be changed through dereferencing e

int *g = &f;
*g = 1;           // OK; this value still can be changed through dereferencing
                  // a pointer-not-to-const

```

## Const pointers

```cpp
int a = 0, b = 2;

const int* pA = &a; // pointer-to-const. `a` can't be changed through this
int* const pB = &a; // const pointer. `a` can be changed, but this pointer can't.
const int* const pC = &a; // const pointer-to-const.

//Error: Cannot assign to a const reference
*pA = b;

pA = &b;

*pB = b;

//Error: Cannot assign to const pointer
pB = &b;

//Error: Cannot assign to a const reference
*pC = b;

//Error: Cannot assign to const pointer
pC = &b;

```

## Const member functions

Member functions of a class can be declared `const`, which tells the compiler and future readers that this function will not modify the object:

> 类的成员函数可以声明为“const”，这告诉编译器和未来的读者该函数不会修改对象：

```cpp
class MyClass
{
private:
    int myInt_;
public:
    int myInt() const { return myInt_; }
    void setMyInt(int myInt) { myInt_ = myInt; }
};

```

In a `const` member function, the `this` pointer is effectively a `const MyClass *` instead of a `MyClass *`. This means that you cannot change any member variables within the function; the compiler will emit a warning. So `setMyInt` could not be declared `const`.

> 在“const”成员函数中，“this”指针实际上是一个“const MyClass*”，而不是“MyClass*”。这意味着您不能更改函数中的任何成员变量；编译器将发出警告。因此，“setMyInt”不能声明为“const”。

You should almost always mark member functions as `const` when possible. Only `const` member functions can be called on a `const MyClass`.

> 在可能的情况下，您几乎应该总是将成员函数标记为“const”。在“const MyClass”上只能调用“const”成员函数。

`static` methods cannot be declared as `const`. This is because a static method belongs to a class and is not called on object; therefore it can never modify object's internal variables. So declaring `static` methods as `const` would be redundant.

## Avoiding duplication of code in const and non-const getter methods.

In C++ methods that differs only by `const` qualifier can be overloaded. Sometimes there may be a need of two versions of getter that return a reference to some member.

> 在 C++ 中，只有“const”限定符不同的方法可以重载。有时可能需要返回对某个成员的引用的 getter 的两个版本。

Let `Foo` be a class, that has two methods that perform identical operations and returns a reference to an object of type `Bar`:

> 设“Foo”是一个类，它有两个执行相同操作的方法，并返回对“Bar”类型对象的引用：

```cpp
class Foo
{
public:
    Bar& GetBar(/* some arguments */)
    {
        /* some calculations */
        return bar;
    }
    
    const Bar& GetBar(/* some arguments */) const
    {
        /* some calculations */
        return bar;
    }

    // ...
};

```

The only difference between them is that one method is non-const and return a non-const reference (that can be use to modify object) and the second is const and returns const reference.

> 它们之间唯一的区别是，一个方法是非常数并返回非常数引用（可用于修改对象），第二个方法是常量并返回常量引用。

To avoid the code duplication, there is a temptation to call one method from another. However, we can not call non-const method from the const one.

> 为了避免代码重复，很容易从另一个方法调用一个方法。但是，我们不能从 const 方法调用非常量方法。

But we can call const method from non-const one. That will require as to use 'const_cast' to remove the const qualifier.

> 但我们可以从非 const 方法调用 const 方法。这将需要 as 使用“const_cast”来删除 const 限定符。

The solution is:

```cpp
struct Foo
{
    Bar& GetBar(/*arguments*/)
    {
        return const_cast<Bar&>(const_cast<const Foo*>(this)->GetBar(/*arguments*/));
    }
    
    const Bar& GetBar(/*arguments*/) const
    {
        /* some calculations */
        return foo;
    }
};

```

In code above, we call const version of `GetBar` from the non-const `GetBar` by casting this to const type: `const_cast<const Foo*>(this)`. Since we call const method from non-const, the object itself is non-const, and casting away the const is allowed.

> 在上面的代码中，我们从非常量“GetBar”调用“GetBar’的常量版本，方法是将其强制转换为常量类型：“const_cast＜const Foo*＞（this）”。由于我们从非常数调用 const 方法，因此对象本身是非常数，并且允许强制转换 const。

Examine the following more complete example:

```cpp
#include <iostream>

class Student
{
public:
    char& GetScore(bool midterm)
    {
        return const_cast<char&>(const_cast<const Student*>(this)->GetScore(midterm));
    }
    
    const char& GetScore(bool midterm) const
    {
        if (midterm)
        {
            return midtermScore;
        }
        else
        {
            return finalScore;
        }
    }
    
private:
    char midtermScore;
    char finalScore;
};

int main()
{
    // non-const object
    Student a; 
    // We can assign to the reference. Non-const version of GetScore is called
    a.GetScore(true) = 'B';
    a.GetScore(false) = 'A';
    
    // const object
    const Student b(a); 
    // We still can call GetScore method of const object,
    // because we have overloaded const version of GetScore
    std::cout << b.GetScore(true) << b.GetScore(false) << '\n'; 
}

```

#### Syntax

- const Type myVariable = initial; // Declares a const variable; cannot be changed
- const Type &myReference = myVariable; // Declares a reference to a const variable
- const Type *myPointer = &myVariable; // Declares a pointer-to-const. The pointer can change, but the underlying data member cannot be changed through the pointer

> -const Type*myPointer=&myVariable；//声明指向常量的指针。指针可以更改，但不能通过指针更改基础数据成员

- Type * const myPointer = &myVariable; // Declares a const pointer. The pointer cannot be reassigned to point to something else, but the underlying data member can be changed

> -类型*const myPointer=&myVariable；//声明一个常量指针。不能重新分配指针以指向其他对象，但可以更改基础数据成员

- const Type * const myPointer = &myVariable; // Declares a const pointer-to-const.

#### Remarks

A variable marked as `const` cannot<sup>1</sup> be changed. Attempting to call any non-const operations on it will result in a compiler error.

> 标记为“const”的变量不能被更改。试图对其调用任何非常量操作都会导致编译器错误。

<sub>1: Well, it can be changed through `const_cast`, but you should almost never use that</sub>
