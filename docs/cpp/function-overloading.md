---
tip: translate by baidu@2023-10-25 08:25:11
---
---

metaTitle: "C++ | Function Overloading"
description: "What is Function Overloading?, Return Type in Function Overloading, Member Function cv-qualifier Overloading"
---------------------------------------------------------------------------------------------------------------------------

# Function Overloading

See also separate topic on [Overload Resolution](http://stackoverflow.com/documentation/c%2B%2B/2021/overload-resolution#t=201705051758072433148)

> 另请参阅[过载解决方案]的单独主题([http://stackoverflow.com/documentation/c%2B%2B/2021/overload](http://stackoverflow.com/documentation/c%2B%2B/2021/overload)-决议#t=201705051758072433148）

## What is Function Overloading?

Function overloading is having multiple functions declared in the same scope with the exact same name exist in the same place (known as **scope**) differing only in their **signature**, meaning the arguments they accept.

> 函数重载是指在同一作用域中声明的多个名称完全相同的函数存在于同一位置（称为**作用域**），其不同之处仅在于它们的**签名**，即它们接受的参数。

Suppose you are writing a series of functions for generalized printing capabilities, beginning with `std::string`:

> 假设您正在为通用打印功能编写一系列函数，以“std:：string”开头：

```cpp
void print(const std::string &str)
{
    std::cout << "This is a string: " << str << std::endl;
}

```

This works fine, but suppose you want a function that also accepts an `int` and prints that too. You could write:

> 这很好，但假设您想要一个同时接受“int”并打印它的函数。你可以写：

```cpp
void print_int(int num)
{
    std::cout << "This is an int:  " << num << std::endl;
}

```

But because the two functions accept different parameters, you can simply write:

> 但是，由于这两个函数接受不同的参数，您可以简单地编写：

```cpp
void print(int num)
{
    std::cout << "This is an int: " << num << std::endl;
}

```

Now you have 2 functions, both named `print`, but with different signatures. One accepts `std::string`, the other one an `int`. Now you can call them without worrying about different names:

> 现在您有两个函数，都命名为“print”，但具有不同的签名。一个接受 std:：string，另一个接受 int。现在，您可以在不担心不同名称的情况下调用它们：

```cpp
print("Hello world!"); //prints "This is a string: Hello world!"
print(1337);           //prints "This is an int: 1337"

```

Instead of:

```cpp
print("Hello world!");
print_int(1337);

```

When you have overloaded functions, the compiler infers which of the functions to call from the parameters you provide it. Care must be taken when writing function overloads. For example, with implicit type conversions:

> 当您有重载函数时，编译器会根据您提供的参数推断要调用哪个函数。编写函数重载时必须小心。例如，对于隐式类型转换：

```cpp
void print(int num)
{
    std::cout << "This is an int: " << num << std::endl;
}
void print(double num)
{
    std::cout << "This is a double: " << num << std::endl;
}

```

Now it's not immediately clear which overload of `print` is called when you write:

> 现在还不清楚在你写作时调用了哪种“print”重载：

```cpp
print(5);

```

And you might need to give your compiler some clues, like:

```cpp
print(static_cast<double>(5));
print(static_cast<int>(5));
print(5.0);

```

Some care also needs to be taken when writing overloads that accept optional parameters:

> 在编写接受可选参数的重载时也需要注意：

```cpp
// WRONG CODE
void print(int num1, int num2 = 0)    //num2 defaults to 0 if not included
{ 
    std::cout << "These are ints: << num1 << " and " << num2 << std::endl;
}
void print(int num)
{
    std::cout << "This is an int: " << num << std::endl;
}

```

Because there's no way for the compiler to tell if a call like `print(17)` is meant for the first or second function because of the optional second parameter, this will fail to compile.

> 由于可选的第二个参数，编译器无法判断像“print（17）”这样的调用是针对第一个函数还是针对第二个函数，因此这将无法编译。

## Return Type in Function Overloading

Note that you cannot overload a function based on its return type. For example:

> 请注意，不能基于函数的返回类型重载函数。例如：

```cpp
// WRONG CODE
std::string getValue()
{
  return "hello";
}

int getValue()
{
  return 0;
}

int x = getValue();

```

This will cause a compilation error as the compiler will not be able to work out which version of `getValue` to call, even though the return type is assigned to an `int`.

> 这将导致编译错误，因为编译器将无法确定要调用哪个版本的“getValue”，即使返回类型已分配给“int”。

## Member Function cv-qualifier Overloading

Functions within a class can be overloaded for when they are accessed through a cv-qualified reference to that class; this is most commonly used to overload for `const`, but can be used to overload for `volatile` and `const volatile`, too.  This is because all non-static member functions take `this` as a hidden parameter, which the cv-qualifiers are applied to.  This is most commonly used to overload for `const`, but can also be used for `volatile` and `const volatile`.

> 当通过对类的 cv 限定引用访问类中的函数时，可以重载该类中的这些函数；这最常用于重载“const”，但也可用于重载“volatile”和“const-vatile”。这是因为所有非静态成员函数都将“This”作为隐藏参数，cv 限定符将应用于该参数。这最常用于重载“const”，但也可用于“volatile”和“const-vatile”。

This is necessary because a member function can only be called if it is at least as cv-qualified as the instance it's called on.  While a non-`const` instance can call both `const` and non-`const` members, a `const` instance can only call `const` members.  This allows a function to have different behaviour depending on the calling instance's cv-qualifiers, and allows the programmer to disallow functions for an undesired cv-qualifier(s) by not providing a version with that qualifier(s).

> 这是必要的，因为只有当成员函数至少与调用它的实例一样具有 cv 限定时，才能调用它。而非 const 实例可以同时调用 const 和非 const 成员，而 const 实例只能调用 const 成员。这允许函数根据调用实例的 cv 限定符具有不同的行为，并允许程序员通过不提供带有该限定符的版本来禁止函数使用不需要的 cv 限制符。

A class with some basic `print` method could be `const` overloaded like so:

> 具有一些基本“print”方法的类可能被“const”重载，如下所示：

```cpp
#include <iostream>

class Integer
{
    public:
        Integer(int i_): i{i_}{}

        void print()
        {
            std::cout << "int: " << i << std::endl;
        }

        void print() const
        {
            std::cout << "const int: " << i << std::endl;
        }

    protected:
        int i;
};

int main()
{
    Integer i{5};
    const Integer &ic = i;
    
    i.print(); // prints "int: 5"
    ic.print(); // prints "const int: 5"
}

```

This is a key tenet of `const` correctness: By marking member functions as `const`, they are allowed to be called on `const` instances, which in turn allows functions to take instances as `const` pointers/references if they don't need to modify them.  This allows code to specify whether it modifies state by taking unmodified parameters as `const` and modified parameters without cv-qualifiers, making code both safer and more readable.

> 这是“常量”正确性的一个关键原则：通过将成员函数标记为“常量”，可以在“常量”实例上调用它们，这反过来又允许函数在不需要修改实例的情况下将其作为“常量”指针/引用。这允许代码通过将未修改的参数作为“const”和不带 cv 限定符的修改参数来指定是否修改状态，从而使代码更加安全和可读。

```cpp
class ConstCorrect 
{
  public:
    void good_func() const 
    {
        std::cout << "I care not whether the instance is const." << std::endl;
    }

    void bad_func() 
    {
        std::cout << "I can only be called on non-const, non-volatile instances." << std::endl;
    }
};

void i_change_no_state(const ConstCorrect& cc) 
{
    std::cout << "I can take either a const or a non-const ConstCorrect." << std::endl;
    cc.good_func(); // Good.  Can be called from const or non-const instance.
    cc.bad_func();  // Error.  Can only be called from non-const instance.
}

void const_incorrect_func(ConstCorrect& cc) 
{
    cc.good_func(); // Good.  Can be called from const or non-const instance.
    cc.bad_func();  // Good.  Can only be called from non-const instance.
}

```

A common usage of this is declaring accessors as `const`, and mutators as non-`const`.

> 它的一个常见用法是将访问器声明为“const”，将赋值器声明为非“const“。

No class members can be modified within a `const` member function. If there is some member that you really need to modify, such as locking a `std::mutex`, you can declare it as [`mutable`](http://stackoverflow.com/documentation/c%2B%2B/2705/mutable-keyword#t=201608042103196324218):

> 在“const”成员函数中不能修改任何类成员。如果确实需要修改某个成员，例如锁定“std:：mutex”，则可以将其声明为[`mutable]([http://stackoverflow.com/documentation/c%2B%2B/2705/mutable](http://stackoverflow.com/documentation/c%2B%2B/2705/mutable)-关键字#t=201608042103196324218）：

```cpp
class Integer
{
    public:
        Integer(int i_): i{i_}{}

        int get() const
        {
            std::lock_guard<std::mutex> lock{mut};
            return i;
        }

        void set(int i_)
        {
            std::lock_guard<std::mutex> lock{mut};
            i = i_;
        }

    protected:
        int i;
        mutable std::mutex mut;
};

```

#### Remarks

Ambiguities can occur when one type can be implicitly converted into more than one type and there is no matching function for that specific type.

> 当一个类型可以隐式转换为多个类型，并且该特定类型没有匹配函数时，可能会出现歧义。

For example:

```cpp
void foo(double, double);
void foo(long, long);

//Call foo with 2 ints
foo(1, 2); //Function call is ambiguous - int can be converted into a double/long at the same time 

```
