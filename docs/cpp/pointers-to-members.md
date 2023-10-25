---
tip: translate by baidu@2023-10-25 08:29:28
---
---

metaTitle: "C++ | Pointers to members"
description: "Pointers to static member functions, Pointers to member functions, Pointers to member variables, Pointers to static member variables"
---------------------------------------------------------------------------------------------------------------------------------------------------

# Pointers to members

## Pointers to static member functions

A `static` member function is just like an ordinary C/C++ function, except with scope:

> “static”成员函数与普通的 C/C++ 函数一样，除了作用域：

- It is inside a `class`, so it needs its name decorated with the class name;
- It has accessibility, with `public`, `protected` or `private`.

So, if you have access to the `static` member function and decorate it correctly, then you can point to the function like any normal function outside a `class`:

> 因此，如果您可以访问“static”成员函数并正确地对其进行修饰，那么您就可以像“class”之外的任何正常函数一样指向该函数：

```cpp
typedef int Fn(int); // Fn is a type-of function that accepts an int and returns an int

// Note that MyFn() is of type 'Fn'
int MyFn(int i) { return 2*i; }

class Class {
public:
    // Note that Static() is of type 'Fn'
    static int Static(int i) { return 3*i; }
}; // Class

int main() {
    Fn *fn;    // fn is a pointer to a type-of Fn

    fn = &MyFn;          // Point to one function
    fn(3);               // Call it
    fn = &Class::Static; // Point to the other function
    fn(4);               // Call it
 } // main()

```

## Pointers to member functions

To access a member function of a class, you need to have a "handle" to the particular instance, as either the instance itself, or a pointer or reference to it. Given a class instance, you can point to various of its members with a pointer-to-member, IF you get the syntax correct! Of course, the pointer has to be declared to be of the same type as what you are pointing to...

> 要访问类的成员函数，你需要有一个特定实例的“句柄”，作为实例本身，或者指向它的指针或引用。给定一个类实例，如果语法正确，你可以用指向成员的指针指向它的各个成员！当然，指针必须声明为与您所指向的类型相同。。。

```cpp
typedef int Fn(int); // Fn is a type-of function that accepts an int and returns an int

class Class {
public:
    // Note that A() is of type 'Fn'
    int A(int a) { return 2*a; }
    // Note that B() is of type 'Fn'
    int B(int b) { return 3*b; }
}; // Class

int main() {
    Class c;          // Need a Class instance to play with
    Class *p = &c;    // Need a Class pointer to play with

    Fn Class::*fn;    // fn is a pointer to a type-of Fn within Class

    fn = &Class::A;   // fn now points to A within any Class
    (c.*fn)(5);       // Pass 5 to c's function A (via fn)
    fn = &Class::B;   // fn now points to B within any Class
    (p->*fn)(6);      // Pass 6 to c's (via p) function B (via fn)
} // main()

```

Unlike pointers to member variables (in the previous example), the association between the class instance and the member pointer need to be bound tightly together with parentheses, which looks a little strange (as though the `.*` and `->*` aren't strange enough!)

> 与指向成员变量的指针（在上一个示例中）不同，类实例和成员指针之间的关联需要用括号紧密绑定在一起，这看起来有点奇怪（好像 `.*` 和 `->*` 还不够奇怪！）

## Pointers to member variables

To access a member of a `class`, you need to have a "handle" to the particular instance, as either the instance itself, or a pointer or reference to it. Given a `class` instance, you can point to various of its members with a pointer-to-member, IF you get the syntax correct! Of course, the pointer has to be declared to be of the same type as what you are pointing to...

> 要访问“class”的成员，你需要有一个特定实例的“句柄”，作为实例本身，或者指向它的指针或引用。给定一个“class”实例，如果语法正确，你可以用指向成员的指针指向它的各个成员！当然，指针必须声明为与您所指向的类型相同。。。

```cpp
class Class {
public:
    int x, y, z;
    char m, n, o;
}; // Class

int x;  // Global variable

int main() {
    Class c;        // Need a Class instance to play with
    Class *p = &c;  // Need a Class pointer to play with

    int *p_i;       // Pointer to an int

    p_i = &x;       // Now pointing to x
    p_i = &c.x;     // Now pointing to c's x

    int Class::*p_C_i; // Pointer to an int within Class

    p_C_i = &Class::x; // Point to x within any Class
    int i = c.*p_C_i;  // Use p_c_i to fetch x from c's instance
    p_C_i = &Class::y; // Point to y within any Class
    i = c.*p_C_i;      // Use p_c_i to fetch y from c's instance

    p_C_i = &Class::m; // ERROR! m is a char, not an int!

    char Class::*p_C_c = &Class::m; // That's better...
} // main()

```

The syntax of pointer-to-member requires some extra syntactic elements:

- To define the type of the pointer, you need to mention the base type, as well as the fact that it is inside a class: `int Class::*ptr;`.

> -要定义指针的类型，您需要提到基类型，以及它位于类内部的事实：`int class:：*ptr；`。

- If you have a class or reference and want to use it with a pointer-to-member, you need to use the `.*` operator (akin to the `.` operator).

> -如果您有一个类或引用，并且希望将其与指向成员的指针一起使用，则需要使用 `.*` 运算符（类似于 `.` 运算符）。

- If you have a pointer to a class and want to use it with a pointer-to-member, you need to use the `->*` operator (akin to the `->` operator).

> -如果您有一个指向类的指针，并希望将其与指向成员的指针一起使用，则需要使用 `->*` 运算符（类似于 `->` 运算符）。

## Pointers to static member variables

A `static` member variable is just like an ordinary C/C++ variable, except with scope:

> “static”成员变量与普通的 C/C++ 变量一样，除了作用域：

- It is inside a `class`, so it needs its name decorated with the class name;
- It has accessibility, with `public`, `protected` or `private`.

So, if you have access to the `static` member variable and decorate it correctly, then you can point to the variable like any normal variable outside a `class`:

> 因此，如果您可以访问“static”成员变量并正确地对其进行修饰，那么您就可以像“class”之外的任何普通变量一样指向该变量：

```cpp
class Class {
public:
    static int i;
}; // Class

int Class::i = 1; // Define the value of i (and where it's stored!)

int j = 2;   // Just another global variable

int main() {
    int k = 3; // Local variable

    int *p;

    p = &k;   // Point to k
    *p = 2;   // Modify it
    p = &j;   // Point to j
    *p = 3;   // Modify it
    p = &Class::i; // Point to Class::i
    *p = 4;   // Modify it
} // main()

```

#### Syntax

<li>
Assuming a class named Class...
<ul>
- type *ptr = &Class::member;        // Point to static members only
- type Class::*ptr = &Class::member; // Point to non-static Class members

For pointers to non-static class members, given the following two definitions:

> 对于指向非静态类成员的指针，给定以下两个定义：

- Class instance;
- Class *p = &instance;

Pointers to Class member variables

- ptr = &Class::i;   // Point to variable i within every Class
- instance.*ptr = 1; // Access instance's i
- p->*ptr = 1;       // Access p's i

Pointers to Class member functions

- ptr = &Class::F;    // Point to function 'F' within every Class
- (instance.*ptr)(5); // Call instance's F
- (p->*ptr)(6);       // Call p's F
