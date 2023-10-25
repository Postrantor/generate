---
tip: translate by baidu@2023-10-25 08:34:26
---
---

metaTitle: "C++ | The Rule of Three, Five, And Zero"
description: "Rule of Zero, Rule of Five, Rule of Three, Self-assignment Protection"
------------------------------------------------------------------------------------

# The Rule of Three, Five, And Zero

## Rule of Zero

We can combine the principles of the Rule of Five and [RAII](http://stackoverflow.com/documentation/c%2B%2B/1320/raii-resource-acquisition-is-initialization#t=201607230254049480454) to get a much leaner interface: the Rule of Zero: any resource that needs to be managed should be in its own type. That type would have to follow the Rule of Five, but all users of that resource do not need to write **any** of the five special member functions and can simply `default` all of them.

> 我们可以将“五条规则”和[RAI]的原则结合起来([http://stackoverflow.com/documentation/c%2B%2B/1320/raii](http://stackoverflow.com/documentation/c%2B%2B/1320/raii)-资源获取是初始化#t=201607230254049480454）以获得更精简的接口：零规则：任何需要管理的资源都应该是自己的类型。这种类型必须遵循“五条规则”，但该资源的所有用户不需要编写五个特别成员函数中的任何一个，只需“默认”所有这些函数即可。

Using the `Person` class introduced in the [Rule of Three example](http://stackoverflow.com/documentation/c%2B%2B/1206/the-rule-of-three-five-and-zero/9867/rule-of-three#t=201607281743050894887), we can create a resource-managing object for `cstrings`:

> 使用[三规则示例]中介绍的“Person”类([http://stackoverflow.com/documentation/c%2b%2b/1206/the](http://stackoverflow.com/documentation/c%2B%2B/1206/the)-三五规则和零/9867/三规则#t=201607281743050894887），我们可以为“cstrings”创建一个资源管理对象：

```cpp
class cstring {
private:
    char* p;

public:
    ~cstring() { delete [] p; }
    cstring(cstring const& );
    cstring(cstring&& );
    cstring& operator=(cstring const& );
    cstring& operator=(cstring&& );

    /* other members as appropriate */
};

```

And once this is separate, our `Person` class becomes far simpler:

```cpp
class Person {
    cstring name;
    int arg;

public:
    ~Person() = default;
    Person(Person const& ) = default;
    Person(Person&& ) = default;
    Person& operator=(Person const& ) = default;
    Person& operator=(Person&& ) = default;

    /* other members as appropriate */
};

```

The special members in `Person` do not even need to be declared explicitly; the compiler will default or delete them appropriately, based on the contents of `Person`. Therefore, the following is also an example of the rule of zero.

> “人”中的特殊成员甚至不需要明确声明；编译器将根据“Person”的内容适当地默认或删除它们。因此，以下也是零规则的一个例子。

```cpp
struct Person {
    cstring name;
    int arg;
};

```

If `cstring` were to be a move-only type, with a `delete` d copy constructor/assignment operator, then `Person` would automatically be move-only as well.

> 如果“cstring”是一个只移动类型，带有“delete”d 复制构造函数/赋值运算符，那么“Person”也将自动成为只移动类型。

The term rule of zero was [introduced by R. Martinho Fernandes](https://rmf.io/cxx11/rule-of-zero)

> 零的术语规则是[由 R.Martinho Fernandes 提出的](https://rmf.io/cxx11/rule-of-zero)

## Rule of Five

C++11 introduces two new special member functions: the move constructor and the move assignment operator. For all the same reasons that you want to follow the [Rule of Three](http://stackoverflow.com/documentation/c%2B%2B/1206/the-rule-of-three-five-and-zero/9867/rule-of-three#t=201607251426414241612) in C++03, you usually want to follow the Rule of Five in C++11: If a class requires ONE of five special member functions, and if move semantics are desired, then it most likely requires ALL FIVE of them.

> C++11 引入了两个新的特殊成员函数：move 构造函数和 move 赋值运算符。出于同样的原因，你想遵循[三条规则]([http://stackoverflow.com/documentation/c%2b%2b/1206/the-rule](http://stackoverflow.com/documentation/c%2B%2B/1206/the-rule) of three five and zero/9867/rule of three#t=2016072514614241612），您通常希望遵循 C++11 中的 rule of five：如果一个类需要五个特殊成员函数中的一个，并且需要移动语义，那么它很可能需要全部五个。

Note, however, that failing to follow the Rule of Five is usually not considered an error, but a missed optimisation opportunity, as long as the Rule of Three is still followed.  If no move constructor or move assignment operator is available when the compiler would normally use one, it will instead use copy semantics if possible, resulting in a less efficient operation due to unnecessary copy operations.  If move semantics aren't desired for a class, then it has no need to declare a move constructor or assignment operator.

> 然而，请注意，只要仍然遵守“三条规则”，不遵守“五条规则”通常不被视为错误，而是错过了优化机会。如果编译器通常使用移动构造函数或移动赋值运算符时没有可用的移动构造函数或移赋值运算符，则如果可能的话，它将使用复制语义，由于不必要的复制操作，导致操作效率较低。如果类不需要移动语义，那么它就不需要声明移动构造函数或赋值运算符。

Same example as for the Rule of Three:

```cpp
class Person
{
    char* name;
    int age;

public:
    // Destructor 
    ~Person() { delete [] name; }

    // Implement Copy Semantics
    Person(Person const& other)
        : name(new char[std::strlen(other.name) + 1])
        , age(other.age)
    {
        std::strcpy(name, other.name);
    }
    
    Person &operator=(Person const& other) 
    {
        // Use copy and swap idiom to implement assignment.
        Person copy(other);
        swap(*this, copy);
        return *this;
    }

    // Implement Move Semantics
    // Note: It is usually best to mark move operators as noexcept
    //       This allows certain optimizations in the standard library
    //       when the class is used in a container.

    Person(Person&& that) noexcept
        : name(nullptr)               // Set the state so we know it is undefined
        , age(0)
    {
        swap(*this, that);
    }

    Person& operator=(Person&& that) noexcept
    {
        swap(*this, that);
        return *this;
    }

    friend void swap(Person& lhs, Person& rhs) noexcept
    {
        std::swap(lhs.name, rhs.name);
        std::swap(lhs.age, rhs.age);
    }
};

```

Alternatively, both the copy and move assignment operator can be replaced with a single assignment operator, which takes an instance by value instead of reference or rvalue reference to facilitate using the copy-and-swap idiom.

> 或者，复制和移动赋值运算符都可以替换为单个赋值运算符，该运算符按值而不是引用或重新值引用来获取实例，以便于使用复制和交换习惯用法。

```cpp
Person& operator=(Person copy)
{
    swap(*this, copy);
    return *this;
}

```

Extending from the Rule of Three to the Rule of Five is important for performance reasons, but is not strictly necessary in most cases. Adding the copy constructor and assignment operator ensures that moving the type will not leak memory (move-constructing will simply fall back to copying in that case), but will be performing copies that the caller probably did not anticipate.

> 从“三条规则”扩展到“五条规则”对于性能原因很重要，但在大多数情况下并不是绝对必要的。添加复制构造函数和赋值运算符可以确保移动类型不会泄漏内存（在这种情况下，移动构造只会返回到复制），但会执行调用方可能没有预料到的复制。

## Rule of Three

The Rule of Three states that if a type ever needs to have a user-defined copy constructor, copy assignment operator, or destructor, then it must have **all three**.

> 三规则规定，如果一个类型需要有一个用户定义的复制构造函数、复制赋值运算符或析构函数，那么它必须有**这三个**。

The reason for the rule is that a class which needs any of the three manages some resource (file handles, dynamically allocated memory, etc), and all three are needed to manage that resource consistently. The copy functions deal with how the resource gets copied between objects, and the destructor would destroy the resource, in accord with [RAII principles](http://stackoverflow.com/documentation/c%2B%2B/1320/raii-resource-acquisition-is-initialization#t=201607230301520057618).

> 该规则的原因是，需要这三者中任何一个的类都会管理一些资源（文件句柄、动态分配的内存等），并且需要这三个类来一致地管理该资源。复制函数处理如何在对象之间复制资源，析构函数将根据[RAII 原则销毁资源([http://stackoverflow.com/documentation/c%2B%2B/1320/raii](http://stackoverflow.com/documentation/c%2B%2B/1320/raii)-资源获取是初始化#t=201607230301520057618）。

Consider a type that manages a string resource:

```cpp
class Person
{
    char* name;
    int age;

public:
    Person(char const* new_name, int new_age)
        : name(new char[std::strlen(new_name) + 1])
        , age(new_age)
    {
       std::strcpy(name, new_name);
    }

    ~Person() {
        delete [] name;
    }
};

```

Since `name` was allocated in the constructor, the destructor deallocates it to avoid leaking memory. But what happens if such an object is copied?

> 由于“name”是在构造函数中分配的，析构函数会将其释放以避免内存泄漏。但是如果复制这样的对象会发生什么呢？

```cpp
int main()
{
    Person p1("foo", 11);
    Person p2 = p1;
}

```

First, `p1` will be constructed. Then `p2` will be copied from `p1`. However, the C++-generated copy constructor will copy each component of the type as-is. Which means that `p1.name` and `p2.name` both point to the **same** string.

> 首先，将构造“p1”。然后将从“p1”复制“p2”。但是，C++ 生成的复制构造函数将按原样复制该类型的每个组件。这意味着“p1.name”和“p2.name”都指向相同的**字符串。

When `main` ends, destructors will be called. First `p2`'s destructor will be called; it will delete the string. Then `p1`'s destructor will be called. However, the string is **already deleted**. Calling `delete` on memory that was already deleted yields undefined behavior.

> 当“main”结束时，将调用析构函数。将调用第一个“p2”的析构函数；它将删除该字符串。然后将调用“p1”的析构函数。但是，字符串**已被删除**。对已删除的内存调用“delete”会产生未定义的行为。

To avoid this, it is necessary to provide a suitable copy constructor. One approach is to implement a reference counted system, where different `Person` instances share the same string data. Each time a copy is performed, the shared reference count is incremented. The destructor then decrements the reference count, only releasing the memory if the count is zero.

> 为了避免这种情况，有必要提供一个合适的复制构造函数。一种方法是实现引用计数系统，其中不同的“Person”实例共享相同的字符串数据。每次执行复制时，共享引用计数都会递增。然后析构函数递减引用计数，只有当计数为零时才释放内存。

Or we could implement [value semantics and deep copying behavior](http://stackoverflow.com/documentation/c%2B%2B/1955/value-and-reference-semantics#t=201607230312543894418):

> 或者我们可以实现[值语义和深度复制行为]([http://stackoverflow.com/documentation/c%2B%2B/1955/value](http://stackoverflow.com/documentation/c%2B%2B/1955/value)-以及引用语义#t=201607230312543894418）：

```cpp
Person(Person const& other)
    : name(new char[std::strlen(other.name) + 1])
    , age(other.age)
{
    std::strcpy(name, other.name);
}

Person &operator=(Person const& other) 
{
    // Use copy and swap idiom to implement assignment
    Person copy(other);
    swap(copy);            //  assume swap() exchanges contents of *this and copy
    return *this;
}

```

Implementation of the copy assignment operator is complicated by the need to release an existing buffer.  The copy and swap technique creates a temporary object which holds a new buffer.  Swapping the contents of `*this` and `copy` gives ownership to `copy` of the original buffer.  Destruction of `copy`, as the function returns, releases the buffer previously owned by `*this`.

> 由于需要释放现有缓冲区，拷贝分配运算符的实现变得复杂。复制和交换技术创建一个临时对象，该对象包含一个新的缓冲区。将“*this”和“copy”的内容交换为原始缓冲区的“copy”赋予所有权。当函数返回时，“副本”的销毁会释放“*this”以前拥有的缓冲区。

## Self-assignment Protection

When writing a copy assignment operator, it is **very** important that it be able to work in the event of self-assignment. That is, it has to allow this:

> 在编写副本分配运算符时，它能够在自我分配的情况下工作是非常重要的。也就是说，它必须允许：

```cpp
SomeType t = ...;
t = t;

```

Self-assignment usually doesn't happen in such an obvious way. It typically happens via a circuitous route through various code systems, where the location of the assignment simply has two `Person` pointers or references and has no idea that they are the same object.

> 自我分配通常不会以如此明显的方式发生。它通常通过各种代码系统中的迂回路径发生，其中赋值的位置只有两个“Person”指针或引用，并且不知道它们是同一对象。

Any copy assignment operator you write must be able to take this into account.

> 您编写的任何副本分配运算符都必须能够考虑到这一点。

The typical way to do so is to wrap all of the assignment logic in a condition like this:

> 这样做的典型方法是将所有赋值逻辑封装在这样的条件中：

```cpp
SomeType &operator=(const SomeType &other)
{
    if(this != &other)
    {
        //Do assignment logic.
    }
    return *this;
}

```

**Note:** It is important to think about self-assignment and ensure that your code behaves correctly when it happens. However, self-assignment is a very rare occurrence and optimizing to prevent it may actually pessimize the normal case. Since the normal case is much more common, pessimizing for self-assignment may well reduce your code efficiency (so be careful using it).

> **注意：**重要的是要考虑自分配，并确保代码在发生这种情况时正确运行。然而，自分配是一种非常罕见的情况，为了防止它而进行的优化实际上可能会使正常情况变得悲观。由于正常情况更为常见，因此对自分配的悲观情绪很可能会降低代码效率（因此使用时要小心）。

As an example, the normal technique for implementing the assignment operator is the `copy and swap idiom`. The normal implementation of this technique does not bother to test for self-assignment (even though self-assignment is expensive because a copy is made). The reason is that pessimization of the normal case has been shown to be much more costly (as it happens more often).

> 例如，实现赋值运算符的常用技术是“复制和交换习惯用法”。该技术的正常实现不需要测试自分配（即使由于复制而导致自分配成本高昂）。原因是，对正常情况的悲观化已经被证明代价要高得多（因为这种情况发生得更频繁）。

Move assignment operators must also be protected against self-assignment. However, the logic for many such operators is based on `std::swap`, which can handle swapping from/to the same memory just fine. So if your move assignment logic is nothing more than a series of swap operations, then you do not need self-assignment protection.

> 移动赋值运算符也必须受到保护，防止自赋值。然而，许多此类运算符的逻辑是基于“std:：swap”的，它可以很好地处理同一内存之间的交换。因此，如果您的移动分配逻辑只不过是一系列交换操作，那么您就不需要自分配保护。

If this is not the case, you **must** take similar measures as above.
