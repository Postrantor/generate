---
tip: translate by baidu@2023-10-25 08:25:08
---
---
metaTitle: "C++ | Friend keyword"
description: "Friend function, Friend method, Friend class"
---

# Friend keyword



Well-designed classes encapsulate their functionality, hiding their implementation while providing a clean, documented interface.  This allows redesign or change so long as the interface is unchanged.

> 设计良好的类封装了它们的功能，隐藏了它们的实现，同时提供了一个干净的、文档化的接口。只要界面不变，就可以重新设计或更改。


In a more complex scenario, multiple classes that rely on each others' implementation details may be required.  Friend classes and functions allow these peers access to each others' details, without compromising the encapsulation and information hiding of the documented interface.

> 在更复杂的场景中，可能需要多个依赖于彼此实现细节的类。Friend类和函数允许这些对等方访问彼此的详细信息，而不会影响文档化接口的封装和信息隐藏。



## Friend function



A class or a structure may declare any function it's friend. If a function is a friend of a class, it may access all it's protected and private members:

> 类或结构可以声明它的朋友的任何函数。如果函数是类的朋友，它可以访问所有受保护的私有成员：

```cpp
// Forward declaration of functions.
void friend_function();
void non_friend_function();

class PrivateHolder {
public:
    PrivateHolder(int val) : private_value(val) {}
private:
    int private_value;
    // Declare one of the function as a friend.
    friend void friend_function();
};

void non_friend_function() {
    PrivateHolder ph(10);
    // Compilation error: private_value is private.
    std::cout << ph.private_value << std::endl;
}

void friend_function() {
    // OK: friends may access private values.
    PrivateHolder ph(10);
    std::cout << ph.private_value << std::endl;
}

```


Access modifiers do not alter friend semantics. Public, protected and private declarations of a friend are equivalent.

> 访问修饰符不会改变友元语义。朋友的公开声明、受保护声明和私人声明是等效的。


Friend declarations are not inherited. For example, if we subclass `PrivateHolder`:

> 友元声明不是继承的。例如，如果我们对“PrivateHolder”进行子类化：

```cpp
class PrivateHolderDerived : public PrivateHolder {
public:
    PrivateHolderDerived(int val) : PrivateHolder(val) {}
private:
    int derived_private_value = 0;
};

```

and try to access it's members, we'll get the following:

```cpp
void friend_function() {
    PrivateHolderDerived pd(20);
    // OK.
    std::cout << pd.private_value << std::endl;
    // Compilation error: derived_private_value is private.
    std::cout << pd.derived_private_value << std::endl;
}

```


Note that `PrivateHolderDerived` member function cannot access `PrivateHolder::private_value`, while friend function can do it.

> 请注意，“PrivateHolderDerived”成员函数不能访问“PrivateHolder:：private_value”，而友元函数可以访问。



## Friend method


Methods may declared as friends as well as functions:

```cpp
class Accesser {
public:
    void private_accesser();
};

class PrivateHolder {
public:
    PrivateHolder(int val) : private_value(val) {}
    friend void Accesser::private_accesser();
private:
    int private_value;
};

void Accesser::private_accesser() {
    PrivateHolder ph(10);
    // OK: this method is declares as friend.
    std::cout << ph.private_value << std::endl;
}

```



## Friend class



A whole class may be declared as friend. Friend class declaration means that any member of the friend may access private and protected members of the declaring class:

> 整个类可以被声明为朋友。友元类声明意味着友元的任何成员都可以访问声明类的私有和受保护成员：

```cpp
class Accesser {
public:
    void private_accesser1();
    void private_accesser2();
};

class PrivateHolder {
public:
    PrivateHolder(int val) : private_value(val) {}
    friend class Accesser;
private:
    int private_value;
};

void Accesser::private_accesser1() {
    PrivateHolder ph(10);
    // OK.
    std::cout << ph.private_value << std::endl;
}

void Accesser::private_accesser2() {
    PrivateHolder ph(10);
    // OK.
    std::cout << ph.private_value + 1 << std::endl;
}

```


Friend class declaration is not reflexive. If classes need private access in both directions, both of them need friend declarations.

> 友元类声明不是反射性的。如果类需要双向的私有访问，那么它们都需要友元声明。

```cpp
class Accesser {
public:
    void private_accesser1();
    void private_accesser2();
private:
    int private_value = 0;
};

class PrivateHolder {
public:
    PrivateHolder(int val) : private_value(val) {}
    // Accesser is a friend of PrivateHolder
    friend class Accesser;
    void reverse_accesse() {
        // but PrivateHolder cannot access Accesser's members.
        Accesser a;
        std::cout << a.private_value;
    }
private:
    int private_value;
};

```

