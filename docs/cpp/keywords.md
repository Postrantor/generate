---
tip: translate by baidu@2023-10-25 08:26:26
---
---

metaTitle: "C++ | Keywords"
description: "asm, explicit, noexcept, typename, sizeof, Different keywords"
----------------------------------------------------------------------------

# Keywords

Keywords have fixed meaning defined by the C++ standard and cannot be used as identifiers. It is illegal to redefine keywords using the preprocessor in any translation unit that includes a standard library header. However, keywords lose their special meaning inside attributes.

> 关键字具有 C++ 标准定义的固定含义，不能用作标识符。在任何包含标准库头的翻译单元中，使用预处理器重新定义关键字都是非法的。然而，关键字在属性中失去了特殊的含义。

## asm

The `asm` keyword takes a single operand, which must be a string literal. It has an implementation-defined meaning, but is typically passed to the implementation's assembler, with the assembler's output being incorporated into the translation unit.

> “asm”关键字接受一个操作数，该操作数必须是字符串文字。它具有实现定义的含义，但通常传递给实现的汇编程序，汇编程序的输出被合并到翻译单元中。

The `asm` statement is a **definition**, not an **expression**, so it may appear either at block scope or namespace scope (including global scope). However, since inline assembly cannot be constrained by the rules of the C++ language, `asm` may not appear inside a `constexpr` function.

> “asm”语句是**定义**，而不是**表达式**，因此它可能出现在块范围或命名空间范围（包括全局范围）。但是，由于内联程序集不能受到 C++ 语言规则的约束，“asm”可能不会出现在“constexpr”函数内部。

Example:

```cpp
[[noreturn]] void halt_system() {
    asm("hlt");
}

```

## explicit

<li>

When applied to a single-argument constructor, prevents that constructor from being used to perform implicit conversions.

> 当应用于单参数构造函数时，会阻止该构造函数用于执行隐式转换。

```cpp
class MyVector {
  public:
    explicit MyVector(uint64_t size);
};
MyVector v1(100);  // ok
uint64_t len1 = 100;
MyVector v2{len1}; // ok, len1 is uint64_t
int len2 = 100;
MyVector v3{len2}; // ill-formed, implicit conversion from int to uint64_t

```

Since C++11 introduced initializer lists, in C++11 and later, `explicit` can be applied to a constructor with any number of arguments, with the same meaning as in the single-argument case.

> 由于 C++11 引入了初始值设定项列表，在 C++11 及更高版本中，“显式”可以应用于具有任意数量参数的构造函数，其含义与单参数情况相同。

```cpp
struct S {
    explicit S(int x, int y);
};
S f() {
    return {12, 34};  // ill-formed
    return S{12, 34}; // ok
}

```

</li>

<li>

When applied to a conversion function, prevents that conversion function from being used to perform implicit conversions.

> 应用于转换函数时，会阻止该转换函数用于执行隐式转换。

```cpp
class C {
    const int x;
  public:
    C(int x) : x(x) {}
    explicit operator int() { return x; }
};
C c(42);
int x = c;                   // ill-formed
int y = static_cast<int>(c); // ok; explicit conversion

```

</li>

## noexcept

<li>

A unary operator that determines whether the evaluation of its operand can propagate an exception. Note that the bodies of called functions are not examined, so `noexcept` can yield false negatives. The operand is not evaluated.

> 一元运算符，用于确定对其操作数的求值是否可以传播异常。请注意，被调用函数的主体没有经过检查，因此“noexcept”可能会产生假阴性。未计算操作数。

```cpp
#include <iostream>
#include <stdexcept>
void foo() { throw std::runtime_error("oops"); }
void bar() {}
struct S {};
int main() {
    std::cout << noexcept(foo()) << '\n'; // prints 0
    std::cout << noexcept(bar()) << '\n'; // prints 0
    std::cout << noexcept(1 + 1) << '\n'; // prints 1
    std::cout << noexcept(S()) << '\n';   // prints 1
}

```

In this example, even though `bar()` can never throw an exception, `noexcept(bar())` is still false because the fact that `bar()` cannot propagate an exception has not been explicitly specified.

> 在本例中，即使“bar（）”永远不能抛出异常，“noexcept（bar（））”仍然为 false，因为“bar（（）”不能传播异常这一事实尚未明确指定。

</li>
<li>

When declaring a function, specifies whether or not the function can propagate an exception. Alone, it declares that the function cannot propagate an exception. With a parenthesized argument, it declares that the function can or cannot propagate an exception depending on the truth value of the argument.

> 在声明函数时，指定该函数是否可以传播异常。单独声明函数不能传播异常。使用带括号的参数，它声明函数可以或不能根据参数的真值传播异常。

```cpp
void f1() { throw std::runtime_error("oops"); }
void f2() noexcept(false) { throw std::runtime_error("oops"); }
void f3() {}
void f4() noexcept {}
void f5() noexcept(true) {}
void f6() noexcept {
    try {
        f1();
    } catch (const std::runtime_error&) {}
}

```

In this example, we have declared that `f4`, `f5`, and `f6` cannot propagate exceptions. (Although an exception can be thrown during execution of `f6`, it is caught and not allowed to propagate out of the function.) We have declared that `f2` may propagate an exception. When the `noexcept` specifier is omitted, it is equivalent to `noexcept(false)`, so we have implicitly declared that `f1` and `f3` may propagate exceptions, even though exceptions cannot actually be thrown during the execution of `f3`.

> 在这个例子中，我们已经声明“f4”、“f5”和“f6”不能传播异常。（尽管在执行“f6”期间可以抛出异常，但它会被捕获，并且不允许传播到函数之外。）我们已经声明“f2”可以传播异常。当省略“noexcept”说明符时，它相当于“noexception（false）”，因此我们隐式声明“f1”和“f3”可以传播异常，即使在执行“f3’期间实际上不能抛出异常。

</li>

Whether or not a function is `noexcept` is part of the function's type: that is, in the example above, `f1`, `f2`, and `f3` have different types from `f4`, `f5`, and `f6`. Therefore, `noexcept` is also significant in function pointers, template arguments, and so on.

> 函数是否为“noexcept”是函数类型的一部分：也就是说，在上面的例子中，“f1”、“f2”和“f3”具有与“f4”、“f5”和“f6”不同的类型。因此，“noexcept”在函数指针、模板参数等中也很重要。

```cpp
void g1() {}
void g2() noexcept {}
void (*p1)() noexcept = &g1; // ill-formed, since g1 is not noexcept
void (*p2)() noexcept = &g2; // ok; types match
void (*p3)() = &g1;          // ok; types match
void (*p4)() = &g2;          // ok; implicit conversion

```

## typename

<li>

When followed by a qualified name, `typename` specifies that it is the name of a type. This is often required in templates, in particular, when the nested name specifier is a dependent type other than the current instantiation. In this example, `std::decay<T>` depends on the template parameter `T`, so in order to name the nested type `type`, we need to prefix the entire qualified name with `typename`. For more deatils, see [Where and why do I have to put the "template" and "typename" keywords?](http://stackoverflow.com/questions/610245/where-and-why-do-i-have-to-put-the-template-and-typename-keywords)

> 当后面跟一个限定名称时，“typename”指定它是一个类型的名称。这在模板中经常是必需的，尤其是当嵌套的名称说明符是当前实例化之外的依赖类型时。在本例中，“std:：decay＜T＞”取决于模板参数“T”，因此为了将嵌套类型命名为“type”，我们需要在整个限定名称前面加上“typename”。有关更多 deatils，请参阅[我必须在哪里以及为什么要放置“template”和“typename”关键字？](http://stackoverflow.com/questions/610245/where-and-why-do-i-have-to-put-the-template-and-typename-keywords)

```cpp
template <class T>
auto decay_copy(T&& r) -> typename std::decay<T>::type;

```

</li>
<li>

Introduces a type parameter in the declaration of a [template](http://stackoverflow.com/documentation/c%2B%2B/460/templates). In this context, it is interchangeable with `class`.

> 在[template]的声明中引入类型参数([http://stackoverflow.com/documentation/c%2b%2b/460/templates](http://stackoverflow.com/documentation/c%2B%2B/460/templates))。在这种情况下，它可以与“class”互换。

```cpp
template <typename T>
const T& min(const T& x, const T& y) {
    return b < a ? b : a;
} 

```

</li>

<li>
`typename` can also be used when declaring a [template template parameter](http://stackoverflow.com/documentation/c%2b%2b/460/templates/10838/template-template-parameters), preceding the name of the parameter, just like `class`.

```cpp
template <template <class T> typename U>
void f() {
    U<int>::do_it();
    U<double>::do_it();
}

```

</li>

## sizeof

A unary operator that yields the size in bytes of its operand, which may be either an expression or a type. If the operand is an expression, it is not evaluated. The size is a constant expression of type `std::size_t`.

> 一种一元运算符，用于产生操作数的字节大小，操作数可以是表达式，也可以是类型。如果操作数是表达式，则不会对其求值。size 是一个类型为“std:：size_t”的常量表达式。

If the operand is a type, it must be parenthesized.

- It is illegal to apply `sizeof` to a function type.
- It is illegal to apply `sizeof` to an incomplete type, including `void`.
- If sizeof is applied to a reference type `T&` or `T&&`, it is equivalent to `sizeof(T)`.
- When `sizeof` is applied to a class type, it yields the number of bytes in a complete object of that type, including any padding bytes in the middle or at the end. Therefore, a `sizeof` expression can never have a value of 0. See [layout of object types](http://stackoverflow.com/documentation/c%2B%2B/9329/layout-of-object-types) for more details.

> -当“sizeof”应用于类类型时，它会产生该类型的完整对象中的字节数，包括中间或结尾的任何填充字节。因此，“sizeof”表达式的值永远不能为 0。参见[对象类型布局]([http://stackoverflow.com/documentation/c%2b%2b/9329/layout](http://stackoverflow.com/documentation/c%2B%2B/9329/layout)-对象类型的）以获取更多细节。

- The `char`, `signed char`, and `unsigned char` types have a size of 1. Conversely, a byte is defined to be the amount of memory required to store a `char` object. It does not necessarily mean 8 bits, as some systems have `char` objects longer than 8 bits.

> -“char”、“signed char”和“unsigned char”类型的大小为 1。相反，字节被定义为存储“char”对象所需的内存量。它不一定意味着 8 位，因为有些系统的“char”对象比 8 位长。

If **expr** is an expression, `sizeof(` **expr** `)` is equivalent to `sizeof(T)` where `T` is the type of **expr.**

> 如果 **expr** 是一个表达式，则“sizeof（`**expr**`）”等效于“sizeof”（T），其中“T”是 **expr 的类型**

```cpp
int a[100];
std::cout << "The number of bytes in `a` is: " << sizeof a;
memset(a, 0, sizeof a); // zeroes out the array

```

The `sizeof...` operator yields the number of elements in a parameter pack.

> `sizeof…` 运算符生成参数包中的元素数。

```cpp
template <class... T>
void f(T&&...) {
    std::cout << "f was called with " << sizeof...(T) << " arguments\n";
}

```

## Different keywords

>

**void C++**

<li>

When used as a function return type, the void keyword specifies that the function does not return a value. When used for a function's parameter list, void specifies that the function takes no parameters. When used in the declaration of a pointer, void specifies that the pointer is "universal."

> 当用作函数返回类型时，void 关键字指定函数不返回值。当用于函数的参数列表时，void 指定该函数不接受任何参数。当在指针的声明中使用时，void 指定指针是“通用的”

</li>
<li>

If a pointer's type is void *, the pointer can point to any variable that is not declared with the const or volatile keyword. A void pointer cannot be dereferenced unless it is cast to another type. A void pointer can be converted into any other type of data pointer.

> 如果指针的类型为 void*，则指针可以指向任何未使用 const 或 volatile 关键字声明的变量。除非将 void 指针强制转换为其他类型，否则无法取消引用该指针。void 指针可以转换为任何其他类型的数据指针。

</li>
<li>
A void pointer can point to a function, but not to a class member in C++.

```cpp
void vobject;   // C2182  
void *pv;   // okay  
int *pint; int i;  
int main() {  
pv = &i;  
   // Cast optional in C required in C++  
pint = (int *)pv;  

```

</li>

>

**Volatile C++**

<li>

A type qualifier that you can use to declare that an object can be modified in the program by the hardware.

> 一种类型限定符，可以用来声明硬件可以在程序中修改对象。

```cpp
volatile declarator ;

```

</li>

>

**virtual C++**

<li>
The virtual keyword declares a virtual function or a virtual base class.

```cpp
virtual [type-specifiers] member-function-declarator  
virtual [access-specifier] base-class-name 

```

</li>

**Parameters**

<li>

**type-specifiers**   Specifies the return type of the virtual member function.

> **类型说明符**指定虚拟成员函数的返回类型。

</li>
<li>
**member-function-declarator**   Declares a member function.
</li>
<li>

**access-specifier**   Defines the level of access to the base class, public, protected or private. Can appear before or after the virtual keyword.

> **访问说明符**定义对基类（public、protected 或 private）的访问级别。可以出现在虚拟关键字之前或之后。

</li>
<li>
**base-class-name**  Identifies a previously declared class type
</li>

>

**this pointer**

<li>

The this pointer is a pointer accessible only within the nonstatic member functions of a class, struct, or union type. It points to the object for which the member function is called. Static member functions do not have a this pointer.

> this 指针是一个只能在类、结构或联合类型的非静态成员函数中访问的指针。它指向调用成员函数的对象。静态成员函数没有 this 指针。

```cpp
this->member-identifier  

```

</li>

An object's this pointer is not part of the object itself; it is not reflected in the result of a sizeof statement on the object. Instead, when a nonstatic member function is called for an object, the address of the object is passed by the compiler as a hidden argument to the function. For example, the following function call:

> 对象的 this 指针不是对象本身的一部分；它不会反映在对象的 sizeof 语句的结果中。相反，当为对象调用非静态成员函数时，编译器会将对象的地址作为隐藏参数传递给该函数。例如，以下函数调用：

```cpp
myDate.setMonth( 3 );  

can be interpreted this way:


setMonth( &myDate, 3 );  

The object's address is available from within the member function as the this pointer. Most uses of this are implicit. It is legal, though unnecessary, to explicitly use this when referring to members of the class. For example:


void Date::setMonth( int mn )  
{  
   month = mn;            // These three statements  
   this->month = mn;      // are equivalent  
   (*this).month = mn;  
}  

The expression *this is commonly used to return the current object from a member function:


return *this;  

The this pointer is also used to guard against self-reference:


if (&Object != this) {  
// do not execute in cases of self-reference 

```

>

**try, throw, and catch Statements (C++)**

1. To implement exception handling in C++, you use try, throw, and catch expressions.

> 1.要在 C++ 中实现异常处理，可以使用 try、throw 和 catch 表达式。

1. First, use a try block to enclose one or more statements that might throw an exception.

> 1.首先，使用 try 块来封装一个或多个可能引发异常的语句。

1. A throw expression signals that an exceptional condition—often, an error—has occurred in a try block. You can use an object of any type as the operand of a throw expression. Typically, this object is used to communicate information about the error. In most cases, we recommend that you use the std::exception class or one of the derived classes that are defined in the standard library. If one of those is not appropriate, we recommend that you derive your own exception class from std::exception.

> 1.抛出表达式表示在 try 块中发生了异常情况，通常是错误。可以使用任何类型的对象作为 throw 表达式的操作数。通常，此对象用于传递有关错误的信息。在大多数情况下，我们建议您使用 std:：exception 类或标准库中定义的派生类之一。如果其中一个不合适，我们建议您从 std:：exception 派生自己的异常类。

1. To handle exceptions that may be thrown, implement one or more catch blocks immediately following a try block. Each catch block specifies the type of exception it can handle.

> 1.要处理可能引发的异常，请在 try 块之后立即实现一个或多个 catch 块。每个 catch 块指定它可以处理的异常类型。

```

   MyData md;  
try {  
   // Code that could throw an exception  
   md = GetNetworkResource();  
}  
catch (const networkIOException& e) {  
   // Code that executes when an exception of type  
   // networkIOException is thrown in the try block  
   // ...  
   // Log error message in the exception object  
   cerr << e.what();  
}  
catch (const myDataFormatException& e) {  
   // Code that handles another exception type  
   // ...  
   cerr << e.what();  
}  
  
// The following syntax shows a throw expression  
MyData GetNetworkResource()  
{  
   // ...  
   if (IOSuccess == false)  
      throw networkIOException("Unable to connect");  
   // ...  
   if (readError)  
      throw myDataFormatException("Format error");   
   // ...  
}

```

>

The code after the try clause is the guarded section of code. The throw expression throws—that is, raises—an exception. The code block after the catch clause is the exception handler. This is the handler that catches the exception that's thrown if the types in the throw and catch expressions are compatible.

> try 子句之后的代码是代码中受保护的部分。throw 表达式抛出——也就是引发——一个异常。catch 子句后面的代码块是异常处理程序。如果 throw 和 catch 表达式中的类型兼容，则该处理程序将捕获抛出的异常。

```

   try {  
   throw CSomeOtherException();  
}  
catch(...) {  
   // Catch all exceptions – dangerous!!!  
   // Respond (perhaps only partially) to the exception, then  
   // re-throw to pass the exception to some other handler  
   // ...  
   throw;  
}

```

>

**friend (C++)**

<li>

In some circumstances, it is more convenient to grant member-level access to functions that are not members of a class or to all members in a separate class. Only the class implementer can declare who its friends are. A function or class cannot declare itself as a friend of any class. In a class definition, use the friend keyword and the name of a non-member function or other class to grant it access to the private and protected members of your class. In a template definition, a type parameter can be declared as a friend.

> 在某些情况下，将成员级别的访问权限授予不是类成员的函数或单独类中的所有成员更方便。只有类实现者才能声明它的朋友是谁。函数或类不能将自己声明为任何类的友元。在类定义中，使用 friend 关键字和非成员函数或其他类的名称，授予它访问类的私有成员和受保护成员的权限。在模板定义中，类型参数可以声明为友元。

</li>
<li>

If you declare a friend function that was not previously declared, that function is exported to the enclosing nonclass scope.

> 如果您声明了以前未声明的友元函数，则该函数将导出到封闭的非类作用域。

```cpp
class friend F  
friend F;
class ForwardDeclared;// Class name is known.  
class HasFriends  
{  
   friend int ForwardDeclared::IsAFriend();// C2039 error expected  
};  

```

</li>

>

**friend functions**

<li>

A friend function is a function that is not a member of a class but has access to the class's private and protected members.Friend functions are not considered class members; they are normal external functions that are given special access privileges.

> 友元函数不是类的成员，但可以访问类的私有成员和受保护成员。友元函数不被视为类成员；它们是被赋予特殊访问权限的正常外部功能。

</li>
<li>

Friends are not in the class's scope, and they are not called using the member-selection operators (. and –>) unless they are members of another class.

> 朋友不在类的作用域内，并且不会使用成员选择运算符（.和–>）调用他们，除非他们是另一个类的成员。

</li>
<li>

A friend function is declared by the class that is granting access. The friend declaration can be placed anywhere in the class declaration. It is not affected by the access control keywords.

> 友元函数是由授予访问权限的类声明的。友元声明可以放在类声明中的任何位置。它不受访问控制关键字的影响。

```cpp
#include <iostream>  

using namespace std;  
class Point  
{  
    friend void ChangePrivate( Point & );  
public:  
    Point( void ) : m_i(0) {}  
    void PrintPrivate( void ){cout << m_i << endl; }  

private:  
int m_i;  
};  

void ChangePrivate ( Point &i ) { i.m_i++; }  

int main()  
{  
   Point sPoint;  
   sPoint.PrintPrivate();  
   ChangePrivate(sPoint);  
   sPoint.PrintPrivate();  
    // Output: 0  
           1  
}  

```

</li>

**Class members as friends**

```cpp
class B;  

class A {  
public:  
   int Func1( B& b );  

private:  
   int Func2( B& b );  
};  

class B {  
private:  
int _b;  

   // A::Func1 is a friend function to class B  
   // so A::Func1 has access to all members of B  
   friend int A::Func1( B& );  
};  

int A::Func1( B& b ) { return b._b; }   // OK  
int A::Func2( B& b ) { return b._b; }   // C2248  

```

#### Syntax

- asm (**string-literal**);
- noexcept(**expression**) // meaning 1
- noexcept(**constant-expression**) // meaning 2
- noexcept // meaning 2
- sizeof **unary-expression**
- sizeof(**type-id**)
- sizeof...(**identifier**) // since C++11
- typename **nested-name-specifier** **identifier** // meaning 1
- typename **nested-name-specifier** template(**opt**) **simple-template-id** // meaning 1
- typename **identifier**(**opt**) // meaning 2
- typename... **identifier**(**opt**) // meaning 2; since C++11
- typename **identifier**(**opt**) = **type-id** // meaning 2
- template <**template-parameter-list**> typename ...(**opt**) **identifier**(**opt**) // meaning 3

> -template＜ **模板参数列表** ＞typename。。。（**opt**）**标识符**（**opt***）//含义 3

- template <**template-parameter-list**> typename **identifier**(**opt**) = **id-expression** // meaning 3

> -template<**template parameter list***>typename**identifier**（**opt**）=**id 表达式**//含义 3

#### Remarks

The full list of keywords is as follows:

- [`alignas`](http://stackoverflow.com/documentation/c%2B%2B/9249/alignment/17909/controlling-alignment) (since C++11)
- [`alignof`](http://stackoverflow.com/documentation/c%2B%2B/9249/alignment/17475/querying-the-alignment-of-a-type) (since C++11)
- [`asm`](http://stackoverflow.com/documentation/c%2B%2B/4891/keywords/18214/asm)
- `auto`: [since C++11](http://stackoverflow.com/documentation/c%2B%2B/7863/type-deduction/25567/auto-type-deduction), [before C++11](http://stackoverflow.com/documentation/c%2B%2B/9225/storage-class-specifiers/28629/auto)

> -`auto`：[自 C++11]([http://stackoverflow.com/documentation/c%2b%2b/7863/type](http://stackoverflow.com/documentation/c%2B%2B/7863/type)-扣除/25567/自动类型扣除），[在 C++11 之前]([http://stackoverflow.com/documentation/c%2b%2b/9225/storage](http://stackoverflow.com/documentation/c%2B%2B/9225/storage)-类说明符/28629/auto）

- [`bool`](http://stackoverflow.com/documentation/c%2B%2B/7839/basic-type-keywords/18416/bool)
- [`break`](http://stackoverflow.com/documentation/c%2B%2B/7841/iteration/18476/break)
- [`case`](http://stackoverflow.com/documentation/c%2B%2B/7837/flow-control/18489/case)
- [`catch`](http://stackoverflow.com/documentation/c%2B%2B/7837/flow-control/18492/catch)
- [`char`](http://stackoverflow.com/documentation/c%2B%2B/7839/basic-type-keywords/18494/char)
- [`char16_t`](http://stackoverflow.com/documentation/c%2B%2B/7839/basic-type-keywords/18501/char16-t) (since C++11)
- [`char32_t`](http://stackoverflow.com/documentation/c%2B%2B/7839/basic-type-keywords/18502/char32-t) (since C++11)
- [`class`](http://stackoverflow.com/documentation/c%2B%2B/7838/type-keywords/18504/class)
- [`const`](http://stackoverflow.com/documentation/c%2B%2B/7840/variable-declaration-keywords/18509/const)
- [`constexpr`](http://stackoverflow.com/documentation/c%2B%2B/3899/constexpr) (since C++11)
- [`const_cast`](http://stackoverflow.com/documentation/c%2B%2B/3090/explicit-type-conversions/11225/casting-away-constness)
- [`continue`](http://stackoverflow.com/documentation/c%2B%2B/7841/iteration/18512/continue)
- [`decltype`](http://stackoverflow.com/documentation/c%2B%2B/7840/variable-declaration-keywords/18513/decltype) (since C++11)
- [`default`](http://stackoverflow.com/documentation/c%2B%2B/7837/flow-control/18514/default)
- `delete` [for memory management](http://stackoverflow.com/documentation/c%2B%2B/2873/memory-management), [for functions](http://stackoverflow.com/documentation/c%2B%2B/206/getting-started-with-c-language/25460/function) (since C++11)

> -`delete `[用于内存管理]([http://stackoverflow.com/documentation/c%2b%2b/2873/memory](http://stackoverflow.com/documentation/c%2B%2B/2873/memory)-管理），[用于功能]([http://stackoverflow.com/documentation/c%2b%2b/206/getting](http://stackoverflow.com/documentation/c%2B%2B/206/getting)-用 c 语言/25460/函数启动）（从 c++11 开始）

- [`do`](http://stackoverflow.com/documentation/c%2B%2B/7841/iteration/18544/do)
- [`double`](http://stackoverflow.com/documentation/c%2B%2B/7839/basic-type-keywords/18640/double)
- [`dynamic_cast`](http://stackoverflow.com/documentation/c%2B%2B/3090/explicit-type-conversions/10518/base-to-derived-conversion)
- [`else`](http://stackoverflow.com/documentation/c%2B%2B/7837/flow-control/18548/else)
- [`enum`](http://stackoverflow.com/documentation/c%2B%2B/7838/type-keywords/18566/enum)
- [`explicit`](http://stackoverflow.com/documentation/c%2B%2B/4891/keywords/18568/explicit)
- [`export`](http://stackoverflow.com/documentation/c%2B%2B/460/templates)
- `extern` [as declaration specifier](http://stackoverflow.com/documentation/c%2B%2B/9225/storage-class-specifiers), [in linkage specification](http://stackoverflow.com/documentation/c%2B%2B/9268/linkage-specifications), [for templates](http://stackoverflow.com/documentation/c%2B%2B/460/templates/28734/explicit-instantiation)

> -`extern `[作为声明说明符]([http://stackoverflow.com/documentation/c%2b%2b/9225/storage](http://stackoverflow.com/documentation/c%2B%2B/9225/storage)-类说明符），[在链接规范中]([http://stackoverflow.com/documentation/c%2b%2b/9268/linkage](http://stackoverflow.com/documentation/c%2B%2B/9268/linkage)-规格），[用于模板]([http://stackoverflow.com/documentation/c%2b%2b/460/templates/28734/explicit](http://stackoverflow.com/documentation/c%2B%2B/460/templates/28734/explicit)-实例化）

- [`false`](http://stackoverflow.com/documentation/c%2B%2B/7836/literal-keywords/18638/false)
- [`float`](http://stackoverflow.com/documentation/c%2B%2B/7839/basic-type-keywords/18639/float)
- [`for`](http://stackoverflow.com/documentation/c%2B%2B/7841/iteration/18641/for)
- [`friend`](http://stackoverflow.com/documentation/c%2B%2B/508/classes-structures/7872/friendship)
- [`goto`](http://stackoverflow.com/documentation/c%2B%2B/7837/flow-control/18643/goto)
- [`if`](http://stackoverflow.com/documentation/c%2B%2B/7837/flow-control/18547/if)
- `inline` [for functions](http://stackoverflow.com/documentation/c%2B%2B/7150/inline-functions), [for namespaces](http://stackoverflow.com/documentation/c%2B%2B/495/namespaces/4556/inline-namespace) (since C++11), [for variables](http://stackoverflow.com/documentation/c%2B%2B/9265/inline-variables) (since C++17)

> -`inline`[用于函数]([http://stackoverflow.com/documentation/c%2b%2b/7150/inline](http://stackoverflow.com/documentation/c%2B%2B/7150/inline)-函数），[用于名称空间]([http://stackoverflow.com/documentation/c%2b%2b/495/namespaces/4556/inline](http://stackoverflow.com/documentation/c%2B%2B/495/namespaces/4556/inline)-命名空间）（自 C++11 以来），[用于变量]([http://stackoverflow.com/documentation/c%2b%2b/9265/inline](http://stackoverflow.com/documentation/c%2B%2B/9265/inline)-变量）（自 C++17 以来）

- [`int`](http://stackoverflow.com/documentation/c%2B%2B/7839/basic-type-keywords/17264/int)
- [`long`](http://stackoverflow.com/documentation/c%2B%2B/7839/basic-type-keywords/18645/long)
- [`mutable`](http://stackoverflow.com/documentation/c%2B%2B/9225/storage-class-specifiers/18647/mutable)
- [`namespace`](http://stackoverflow.com/documentation/c%2B%2B/4891/keywords/18653/namespace)
- [`new`](http://stackoverflow.com/documentation/c%2B%2B/2873/memory-management)
- [`noexcept`](http://stackoverflow.com/documentation/c%2B%2B/4891/keywords/18664/noexcept) (since C++11)
- [`nullptr`](http://stackoverflow.com/documentation/c%2B%2B/7836/literal-keywords/18669/nullptr) (since C++11)
- [`operator`](http://stackoverflow.com/documentation/c%2B%2B/562/operator-overloading)
- [`private`](http://stackoverflow.com/documentation/c%2B%2B/508/classes-structures/1668/access-specifiers)
- [`protected`](http://stackoverflow.com/documentation/c%2B%2B/508/classes-structures/1668/access-specifiers)
- [`public`](http://stackoverflow.com/documentation/c%2B%2B/508/classes-structures/1668/access-specifiers)
- [`register`](http://stackoverflow.com/documentation/c%2B%2B/9225/storage-class-specifiers/18681/register)
- [`reinterpret_cast`](http://stackoverflow.com/documentation/c%2B%2B/3090/explicit-type-conversions)
- [`return`](http://stackoverflow.com/documentation/c%2B%2B/7837/flow-control/18683/return)
- [`short`](http://stackoverflow.com/documentation/c%2B%2B/7839/basic-type-keywords/18646/short)
- [`signed`](http://stackoverflow.com/documentation/c%2B%2B/7840/variable-declaration-keywords/18685/signed)
- [`sizeof`](http://stackoverflow.com/documentation/c%2B%2B/4891/keywords/18687/sizeof)
- [`static`](http://stackoverflow.com/documentation/c%2B%2B/9225/storage-class-specifiers/18689/static)
- [`static_assert`](http://stackoverflow.com/documentation/c%2B%2B/3822/static-assert) (since C++11)
- [`static_cast`](http://stackoverflow.com/documentation/c%2B%2B/3090/explicit-type-conversions)
- [`struct`](http://stackoverflow.com/documentation/c%2B%2B/7838/type-keywords/18505/struct)
- [`switch`](http://stackoverflow.com/documentation/c%2B%2B/7837/flow-control/18490/switch)
- [`template`](http://stackoverflow.com/documentation/c%2B%2B/460/templates)
- [`this`](http://stackoverflow.com/documentation/c%2B%2B/7836/literal-keywords/18758/this)
- [`thread_local`](http://stackoverflow.com/documentation/c%2B%2B/699/threading/18759/thread-local-storage) (since C++11)
- [`throw`](http://stackoverflow.com/documentation/c%2B%2B/7837/flow-control/18765/throw)
- [`true`](http://stackoverflow.com/documentation/c%2B%2B/7836/literal-keywords/18637/true)
- [`try`](http://stackoverflow.com/documentation/c%2B%2B/7837/flow-control/18781/try)
- [`typedef`](http://stackoverflow.com/documentation/c%2B%2B/9328/typedef-and-type-aliases)
- [`typeid`](http://stackoverflow.com/documentation/c%2B%2B/3129/rtti-run-time-type-information)
- [`typename`](http://stackoverflow.com/documentation/c%2B%2B/4891/keywords/19075/typename)
- [`union`](http://stackoverflow.com/documentation/c%2B%2B/7838/type-keywords/19092/union)
- [`unsigned`](http://stackoverflow.com/documentation/c%2B%2B/7840/variable-declaration-keywords/18686/unsigned)
- `using` [to redeclare a name](http://stackoverflow.com/documentation/c%2B%2B/9301/using-declaration), [to alias a namespace](http://stackoverflow.com/documentation/c%2B%2B/495/namespaces/28691/namespace-alias), [to alias a type](http://stackoverflow.com/documentation/c%2B%2B/9328/typedef-and-type-aliases)

> -`using`[重新声明名称]([http://stackoverflow.com/documentation/c%2b%2b/9301/using](http://stackoverflow.com/documentation/c%2B%2B/9301/using)-声明），[别名名称空间]([http://stackoverflow.com/documentation/c%2b%2b/495/namespaces/28691/namespace-alias](http://stackoverflow.com/documentation/c%2B%2B/495/namespaces/28691/namespace-alias)），[对类型进行别名]([http://stackoverflow.com/documentation/c%2b%2b/9328/typedef](http://stackoverflow.com/documentation/c%2B%2B/9328/typedef)-和类型别名）

- `virtual` [for functions](http://stackoverflow.com/documentation/c%2B%2B/4891/keywords/19097/virtual), [for base classes](http://stackoverflow.com/documentation/c%2B%2B/508/classes-structures/1670/virtual-inheritance)

> -`virtual` [用于函数](http://stackoverflow.com/documentation/c%2B%2B/4891/keywords/19097/virtual)，[对于基类]([http://stackoverflow.com/documentation/c%2b%2b/508/classes](http://stackoverflow.com/documentation/c%2B%2B/508/classes)-结构/1670/虚拟继承）

- [`void`](http://stackoverflow.com/documentation/c%2B%2B/7839/basic-type-keywords/19098/void)
- [`volatile`](http://stackoverflow.com/documentation/c%2B%2B/7840/variable-declaration-keywords/19102/volatile)
- [`wchar_t`](http://stackoverflow.com/documentation/c%2B%2B/7839/basic-type-keywords/19113/wchar-t)
- [`while`](http://stackoverflow.com/documentation/c%2B%2B/7841/iteration/25514/while)

The tokens `final` and `override` are not keywords. They may be used as identifiers and have special meaning only in certain contexts.

> 标记“final”和“override”不是关键字。它们可以用作标识符，并且仅在特定上下文中具有特殊含义。

The tokens `and`, `and_eq`, `bitand`, `bitor`, `compl`, `not`, `not_eq`, `or`, `or_eq`, `xor`, and `xor_eq` are alternative spellings of `&&`, `&=`, `&`, `|`, `~`, `!`, `!=`, `||`, `|=`, `^`, and `^=`, respectively. The standard does not treat them as keywords, but they are keywords for all intents and purposes, since it is impossible to redefine them or use them to mean anything other than the operators they represent.

> 标记 `and`，`and_eq`，`bitand`，` bitor`，`compl`，`not`，`or`，` or_eq`、`xor` 和 `xor_eq` 是 `&&`、`&&`，`|`、`~`、`！`、`！=`、`||`、` |=`、` ^` 的替代拼写，和“^=”。该标准不将它们视为关键字，但它们是所有意图和目的的关键字，因为不可能重新定义它们或使用它们来表示它们所代表的运算符之外的任何其他含义。

The following topics contain detailed explanations of many of the keywords in C++, which serve fundamental purposes such as naming basic types or controlling the flow of execution.

> 以下主题包含对 C++ 中许多关键字的详细解释，这些关键字具有命名基本类型或控制执行流等基本目的。

<li>

[Basic Type Keywords](http://stackoverflow.com/documentation/c%2B%2B/7839/basic-type-keywords)

> [基本类型关键字]([http://stackoverflow.com/documentation/c%2B%2B/7839/basic](http://stackoverflow.com/documentation/c%2B%2B/7839/basic)-键入关键字）

</li>
<li>

[Flow Control](http://stackoverflow.com/documentation/c%2B%2B/7837/flow-control)

> [流量控制]([http://stackoverflow.com/documentation/c%2B%2B/7837/flow](http://stackoverflow.com/documentation/c%2B%2B/7837/flow)-控制）

</li>
<li>

[Iteration](http://stackoverflow.com/documentation/c%2B%2B/7841/iteration)

> [迭代](http://stackoverflow.com/documentation/c%2B%2B/7841/iteration)

</li>
<li>

[Literal Keywords](http://stackoverflow.com/documentation/c%2B%2B/7836/literal-keywords)

> [文字关键字]([http://stackoverflow.com/documentation/c%2B%2B/7836/literal](http://stackoverflow.com/documentation/c%2B%2B/7836/literal)-关键字）

</li>
<li>

[Type Keywords](http://stackoverflow.com/documentation/c%2B%2B/7838/type-keywords)

> [键入关键字]([http://stackoverflow.com/documentation/c%2B%2B/7838/type](http://stackoverflow.com/documentation/c%2B%2B/7838/type)-关键字）

</li>
<li>

[Variable Declaration Keywords](http://stackoverflow.com/documentation/c%2B%2B/7840/variable-declaration-keywords)

> [变量声明关键字]([http://stackoverflow.com/documentation/c%2B%2B/7840/variable](http://stackoverflow.com/documentation/c%2B%2B/7840/variable)-声明关键字）

</li>
<li>

[Classes/Structures](http://stackoverflow.com/documentation/c%2B%2B/508/classes-structures)

> [类别/结构]([http://stackoverflow.com/documentation/c%2b%2b/508/classes](http://stackoverflow.com/documentation/c%2B%2B/508/classes)-结构）

</li>
<li>

[Storage class specifiers](http://stackoverflow.com/documentation/c%2B%2B/9225/storage-class-specifiers)

> [存储类说明符]([http://stackoverflow.com/documentation/c%2b%2b/9225/storage](http://stackoverflow.com/documentation/c%2B%2B/9225/storage)-类说明符）

</li>
