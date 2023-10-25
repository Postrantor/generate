---
tip: translate by baidu@2023-10-25 08:27:59
---
---

metaTitle: "C++ | Move Semantics"
description: "Using std::move to reduce complexity from O(n²) to O(n), Move semantics, Move constructor, Move assignment, Re-use a moved object, Using move semantics on containers"
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# Move Semantics

## Using std::move to reduce complexity from O(n²) to O(n)

C++11 introduced core language and standard library support for **moving** an object. The idea is that when an object **o** is a temporary and one wants a logical copy, then its safe to just pilfer **o**'s resources, such as a dynamically allocated buffer, leaving **o** logically empty but still destructible and copyable.

> C++11 引入了核心语言和标准库，支持**移动**对象。这个想法是，当一个对象 **o** 是临时的，并且想要一个逻辑副本时，那么窃取 **o** 的资源是安全的，比如动态分配的缓冲区，使 **o** 在逻辑上是空的，但仍然是可破坏和可复制的。

The core language support is mainly

<li>

the **rvalue reference** type builder `&&`, e.g., `std::string&&` is an rvalue reference to a `std::string`, indicating that that referred to object is a temporary whose resources can just be pilfered (i.e. moved)

> **右值引用**类型构造器“&&”，例如“std:：string&&”是对“std：：string”的右值引用，表示被引用的对象是临时对象，其资源只能被窃取（即移动）

</li>
<li>

special support for a **move constructor** `T( T&& )`, which is supposed to efficiently move resources from the specified other object, instead of actually copying those resources, and

> 对 **move 构造函数** `T（T&&）` 的特殊支持，该构造函数应该有效地从指定的其他对象中移动资源，而不是实际复制这些资源，以及

</li>
<li>

special support for a **move assignment operator** `auto operator=(T&&) -> T&`, which also is supposed to move from the source.

> 对**移动赋值运算符** `自动运算符=（T&&）->T&` 的特殊支持，该运算符也应该从源移动。

</li>

The standard library support is mainly the `std::move` function template from the `<utility>` header. This function produces an rvalue reference to the specified object, indicating that it can be moved from, just as if it were a temporary.

> 标准库支持主要是 `<utility>` 头中的 `std:：move` 函数模板。此函数生成对指定对象的右值引用，表示可以将其从中移动，就像它是临时对象一样。

For a container actual copying is typically of O(**n**) complexity, where **n** is the number of items in the container, while moving is O(1), constant time. And for an algorithm that logically copies that container **n** times, this can reduce the complexity from the usually impractical O(**n**²) to just linear O(**n**).

> 对于容器，实际复制通常具有 O（**n**）复杂性，其中 **n** 是容器中的项目数，而移动是 O（1），恒定时间。对于逻辑复制该容器 **n** 次的算法，这可以将复杂性从通常不切实际的 O（**n***²）降低到线性 O（**n**）。

In his article [“Containers That Never Change” in Dr. Dobbs Journal in September 19 2013](http://www.drdobbs.com/cpp/containters-that-never-change/240161543), Andrew Koenig presented an interesting example of algorithmic inefficiency when using a style of programming where variables are immutable after initialization. With this style loops are generally expressed using recursion. And for some algorithms such as generating a Collatz sequence, the recursion requires logically copying a container:

> 在他的文章【2013 年 9 月 19 日多布斯博士期刊上的“永不改变的容器”】([http://www.drdobbs.com/cpp/containters-that-never-change/240161543](http://www.drdobbs.com/cpp/containters-that-never-change/240161543))，Andrew Koenig 提出了一个有趣的例子，说明在使用初始化后变量不可变的编程风格时，算法效率低下。使用这种风格，循环通常使用递归来表示。对于某些算法，如生成 Collatz 序列，递归需要从逻辑上复制容器：

```cpp
// Based on an example by Andrew Koenig in his Dr. Dobbs Journal article
// “Containers That Never Change” September 19, 2013, available at
// <url: http://www.drdobbs.com/cpp/containters-that-never-change/240161543>

// Includes here, e.g. <vector>

namespace my {
    template< class Item >
    using Vector_ = /* E.g. std::vector<Item> */;

    auto concat( Vector_<int> const& v, int const x )
        -> Vector_<int>
    {
        auto result{ v };
        result.push_back( x );
        return result;
    }

    auto collatz_aux( int const n, Vector_<int> const& result )
        -> Vector_<int>
    {
        if( n == 1 )
        {
            return result;
        }
        auto const new_result = concat( result, n );
        if( n % 2 == 0 )
        {
            return collatz_aux( n/2, new_result );
        }
        else
        {
            return collatz_aux( 3*n + 1, new_result );
        }
    }

    auto collatz( int const n )
        -> Vector_<int>
    {
        assert( n != 0 );
        return collatz_aux( n, Vector_<int>() );
    }
}  // namespace my

#include <iostream>
using namespace std;
auto main() -> int
{
    for( int const x : my::collatz( 42 ) )
    {
        cout << x << ' ';
    }
    cout << '\n';
}

```

Output:

```cpp
using std::move;

auto concat( Vector_<int> v, int const x )
    -> Vector_<int>
{
    v.push_back( x );
    // warning: moving a local object in a return statement prevents copy elision [-Wpessimizing-move]
    // See https://stackoverflow.com/documentation/c%2b%2b/2489/copy-elision
    // return move( v );
    return v;
}

auto collatz_aux( int const n, Vector_<int> result )
    -> Vector_<int>
{
    if( n == 1 )
    {
        return result;
    }
    auto new_result = concat( move( result ), n );
    struct result;      // Make absolutely sure no use of `result` after this.
    if( n % 2 == 0 )
    {
        return collatz_aux( n/2, move( new_result ) );
    }
    else
    {
        return collatz_aux( 3*n + 1, move( new_result ) );
    }
}

auto collatz( int const n )
    -> Vector_<int>
{
    assert( n != 0 );
    return collatz_aux( n, Vector_<int>() );
}

```

Here, with g++ and Visual C++ compilers, the number of item copy operations due to vector copy constructor invocations, was exactly 0.

> 在这里，对于 g++ 和 Visual C++ 编译器，由于向量复制构造函数调用而导致的项复制操作的数量正好为 0。

The algorithm is necessarily still O(**n**) in the length of the Collatz sequence produced, but this is a quite dramatic improvement: O(**n**²) → O(**n**).

> 该算法在产生的 Collatz 序列的长度上必然仍然是 O（**n**），但这是一个相当显著的改进：O（**n***²）→ O（**n**）。

With some language support one could perhaps use moving and still express and enforce the immutability of a variable **between its initialization and final move**, after which any use of that variable should be an error. Alas, as of C++14 C++ does not support that. For loop-free code the no use after move can be enforced via a re-declaration of the relevant name as an incomplete `struct`, as with `struct result;` above, but this is ugly and not likely to be understood by other programmers; also the diagnostics can be quite misleading.

> 有了一些语言支持，也许可以使用移动，并且仍然表示和强制变量**在初始化和最终移动之间的不变性，之后对该变量的任何使用都应该是错误的。遗憾的是，从 C++14 开始，C++ 就不支持这一点。对于无循环代码，可以通过将相关名称重新声明为不完整的“struct”（与“struct result；”以上，但这很难看，其他程序员不太可能理解；此外，诊断可能会产生很大的误导。

Summing up, the C++ language and library support for moving allows drastic improvements in algorithm complexity, but due the support's incompleteness, at the cost of forsaking the code correctness guarantees and code clarity that `const` can provide.

> 总之，C++ 语言和库对移动的支持可以极大地提高算法的复杂性，但由于支持的不完整性，代价是放弃了“const”所能提供的代码正确性保证和代码清晰度。

```cpp
template< class Item >
class Copy_tracking_vector
{
private:
    static auto n_copy_ops()
        -> int&
    {
        static int value;
        return value;
    }
    
    vector<Item>    items_;
    
public:
    static auto n() -> int { return n_copy_ops(); }

    void push_back( Item const& o ) { items_.push_back( o ); }
    auto begin() const { return items_.begin(); }
    auto end() const { return items_.end(); }

    Copy_tracking_vector(){}
    
    Copy_tracking_vector( Copy_tracking_vector const& other )
        : items_( other.items_ )
    { n_copy_ops() += items_.size(); }

    Copy_tracking_vector( Copy_tracking_vector&& other )
        : items_( move( other.items_ ) )
    {}
};

```

## Move semantics

Move semantics are a way of moving one object to another in C++. For this, we empty the old object and place everything it had in the new object.

> 移动语义是在 C++ 中将一个对象移动到另一个对象的一种方式。为此，我们清空旧对象，并将它所拥有的一切都放在新对象中。

For this, we must understand what an rvalue reference is. An rvalue reference (`T&&` where T is the object type) is not much different than a normal reference (`T&`, now called lvalue references). But they act as 2 different types, and so, we can make constructors or functions that take one type or the other, which will be necessary when dealing with move semantics.

> 为此，我们必须理解什么是右值引用。右值引用（“T&&”，其中 T 是对象类型）与正常引用（“T&”，现在称为左值引用）没有太大区别。但它们充当两种不同的类型，因此，我们可以生成采用一种或另一种类型的构造函数或函数，这在处理移动语义时是必要的。

The reason why we need two different types is to specify two different behaviors. Lvalue reference constructors are related to copying, while rvalue reference constructors are related to moving.

> 我们需要两种不同类型的原因是指定两种不同的行为。Lvalue 引用构造函数与复制有关，而右值引用构造函数与移动有关。

To move an object, we will use `std::move(obj)`. This function returns an rvalue reference to the object, so that we can steal the data from that object into a new one. There are several ways of doing this which are discussed below.

> 要移动对象，我们将使用“std:：move（obj）”。这个函数返回一个对对象的右值引用，这样我们就可以将该对象的数据窃取到一个新对象中。有几种方法可以做到这一点，下面将进行讨论。

Important to note is that the use of `std::move` creates just an rvalue reference.

> 需要注意的是，使用“std:：move”只会创建一个右值引用。

In other words the statement `std::move(obj)` does not change the content of obj, while `auto obj2 = std::move(obj)` (possibly) does.

> 换句话说，语句“std:：move（obj）”不会更改 obj 的内容，而“auto obj2=std:：movie（obg）”（可能）会更改。

## Move constructor

Say we have this code snippet.

```cpp
class A {
public:
    int a;
    int b;
       
    A(const A &other) {
        this->a = other.a;
        this->b = other.b;
    }
};

```

To create a copy constructor, that is, to make a function that copies an object and creates a new one, we normally would choose the syntax shown above, we would have a constructor for A that takes an reference to another object of type A, and we would copy the object manually inside the method.

> 要创建复制构造函数，也就是说，要创建一个复制对象并创建新对象的函数，我们通常会选择上面显示的语法，我们会为 a 创建一个引用另一个类型为 a 的对象的构造函数，我们会在方法内部手动复制对象。

Alternatively, we could have written `A(const A &) = default;` which automatically copies over all members, making use of its copy constructor.

> 或者，我们可以写 `A（const A&）=default；` 它利用其复制构造函数自动复制所有成员。

To create a move constructor, however, we will be taking an rvalue reference instead of an lvalue reference, like here.

> 然而，要创建一个 move 构造函数，我们将使用一个右值引用，而不是像这里那样的左值引用。

```cpp
class Wallet {
public:
    int nrOfDollars;
    
    Wallet() = default; //default ctor

    Wallet(Wallet &&other) {
        this->nrOfDollars = other.nrOfDollars;
        other.nrOfDollars = 0;
    }
};

```

Please notice that we set the old values to `zero`. The default move constructor (`Wallet(Wallet&&) = default;`) copies the value of `nrOfDollars`, as it is a POD.

> 请注意，我们将旧值设置为“零”。默认的移动构造函数（`Wallet（Wallet&&）=default；`）复制“nrOfDollars”的值，因为它是 POD。

As move semantics are designed to allow 'stealing' state from the original instance, it is important to consider how the original instance should look like after this stealing. In this case, if we would not change the value to zero we would have doubled the amount of dollars into play.

> 由于移动语义是为了允许从原始实例“窃取”状态而设计的，因此考虑在这种窃取之后原始实例应该是什么样子是很重要的。在这种情况下，如果我们不将价值更改为零，我们将使投入的美元数量翻倍。

```cpp
Wallet a;
a.nrOfDollars = 1;
Wallet b (std::move(a)); //calling B(B&& other);
std::cout << a.nrOfDollars << std::endl; //0
std::cout << b.nrOfDollars << std::endl; //1

```

Thus we have move constructed an object from an old one.

While the above is a simple example, it shows what the move constructor is intended to do.  It becomes more useful in more complex cases, such as when resource management is involved.

> 虽然上面是一个简单的例子，但它展示了 move 构造函数的意图。在更复杂的情况下，例如涉及资源管理时，它会变得更有用。

```

   // Manages operations involving a specified type.
    // Owns a helper on the heap, and one in its memory (presumably on the stack).
    // Both helpers are DefaultConstructible, CopyConstructible, and MoveConstructible.
    template<typename T,
             template<typename> typename HeapHelper,
             template<typename> typename StackHelper>
    class OperationsManager {
        using MyType = OperationsManager<T, HeapHelper, StackHelper>;

        HeapHelper<T>* h_helper;
        StackHelper<T> s_helper;
        // ...

      public:
        // Default constructor & Rule of Five.
        OperationsManager() : h_helper(new HeapHelper<T>) {}
        OperationsManager(const MyType& other)
          : h_helper(new HeapHelper<T>(*other.h_helper)), s_helper(other.s_helper) {}
        MyType& operator=(MyType copy) {
            swap(*this, copy);
            return *this;
        }
        ~OperationsManager() {
            if (h_helper) { delete h_helper; }
        }

        // Move constructor (without swap()).
        // Takes other's HeapHelper<T>*.
        // Takes other's StackHelper<T>, by forcing the use of StackHelper<T>'s move constructor.
        // Replaces other's HeapHelper<T>* with nullptr, to keep other from deleting our shiny
        //  new helper when it's destroyed.
        OperationsManager(MyType&& other) noexcept
          : h_helper(other.h_helper),
            s_helper(std::move(other.s_helper)) {
            other.h_helper = nullptr;
        }

        // Move constructor (with swap()).
        // Places our members in the condition we want other's to be in, then switches members
        //  with other.
        // OperationsManager(MyType&& other) noexcept : h_helper(nullptr) {
        //     swap(*this, other);
        // }

        // Copy/move helper.
        friend void swap(MyType& left, MyType& right) noexcept {
            std::swap(left.h_helper, right.h_helper);
            std::swap(left.s_helper, right.s_helper);
        }
    };

```

## Move assignment

Similarly to how we can assign a value to an object with an lvalue reference, copying it, we can also move the values from an object to another without constructing a new one. We call this move assignment. We move the values from one object to another existing object.

> 类似于我们如何将值分配给具有左值引用的对象并复制它，我们也可以将值从一个对象移动到另一个对象，而无需构建新的值。我们称之为搬家任务。我们将值从一个对象移动到另一个现有对象。

For this, we will have to overload `operator =`, not so that it takes an lvalue reference, like in copy assignment, but so that it takes an rvalue reference.

> 为此，我们必须重载“operator=”，而不是像复制赋值那样，让它接受左值引用，而是让它接受右值引用。

```cpp
class A {
    int a;
    A& operator= (A&& other) {
        this->a = other.a;
        other.a = 0;
        return *this;
    }
};

```

This is the typical syntax to define move assignment. We overload `operator =` so that we can feed it an rvalue reference and it can assign it to another object.

> 这是定义移动分配的典型语法。我们重载“operator=”，这样我们就可以给它一个右值引用，它也可以把它分配给另一个对象。

```cpp
A a;
a.a = 1;
A b;
b = std::move(a); //calling A& operator= (A&& other)
std::cout << a.a << std::endl; //0
std::cout << b.a << std::endl; //1

```

Thus, we can move assign an object to another one.

## Re-use a moved object

You can re-use a moved object:

```cpp
void consumingFunction(std::vector<int> vec) {
    // Some operations
}

int main() {
    // initialize vec with 1, 2, 3, 4
    std::vector<int> vec{1, 2, 3, 4};

    // Send the vector by move
    consumingFunction(std::move(vec));

    // Here the vec object is in an indeterminate state.
    // Since the object is not destroyed, we can assign it a new content.
    // We will, in this case, assign an empty value to the vector,
    // making it effectively empty
    vec = {};

    // Since the vector as gained a determinate value, we can use it normally.
    vec.push_back(42);

    // Send the vector by move again.
    consumingFunction(std::move(vec));
}

```

## Using move semantics on containers

You can move a container instead of copying it:

```cpp
void print(const std::vector<int>& vec) {
    for (auto&& val : vec) {
        std::cout << val << ", ";
    }
    std::cout << std::endl;
}

int main() {
    // initialize vec1 with 1, 2, 3, 4 and vec2 as an empty vector
    std::vector<int> vec1{1, 2, 3, 4};
    std::vector<int> vec2;

    // The following line will print 1, 2, 3, 4
    print(vec1);

    // The following line will print a new line
    print(vec2);

    // The vector vec2 is assigned with move assingment.
    // This will "steal" the value of vec1 without copying it.
    vec2 = std::move(vec1);

    // Here the vec1 object is in an indeterminate state, but still valid.
    // The object vec1 is not destroyed,
    // but there's is no guarantees about what it contains.

    // The following line will print 1, 2, 3, 4
    print(vec2);
}

```
