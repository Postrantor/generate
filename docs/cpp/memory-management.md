---
tip: translate by baidu@2023-10-25 08:27:35
---
---
metaTitle: "C++ | Memory management"
description: "Free Storage (Heap, Dynamic Allocation ...), Placement new, Stack"
---

# Memory management




## Free Storage (Heap, Dynamic Allocation ...)



The term **'heap'** is a general computing term meaning an area of memory from which portions can be allocated and deallocated independently of the memory provided by the **stack**.

> 术语**‘heap’**是一个通用的计算术语，意思是指可以独立于**堆栈**提供的内存来分配和释放部分的内存区域。


In `C++` the **Standard** refers to this area as the **Free Store** which is considered a more accurate term.

> 在“C++”中，**标准**将此区域称为**免费商店**，这被认为是一个更准确的术语。


Areas of memory allocated from the **Free Store** may live longer than the original scope in which it was allocated. Data too large to be stored on the stack may also be allocated from the **Free Store**.

> 从**免费存储**分配的内存区域可能比分配内存的原始作用域寿命更长。太大而无法存储在堆栈上的数据也可能从**免费存储**中分配。


Raw memory can be allocated and deallocated by the **new** and **delete** keywords.

> 原始内存可以通过**new**和**delete**关键字进行分配和释放。

```cpp
float *foo = nullptr;
{
    *foo = new float; // Allocates memory for a float
    float bar;              // Stack allocated 
} // End lifetime of bar, while foo still alive

delete foo;          // Deletes the memory for the float at pF, invalidating the pointer
foo = nullptr;       // Setting the pointer to nullptr after delete is often considered good practice

```


It's also possible to allocate fixed size arrays with **new** and **delete**, with a slightly different syntax. Array allocation is not compatible with non-array allocation, and mixing the two will lead to heap corruption. Allocating an array also allocates memory to track the size of the array for later deletion in an implementation-defined way.

> 也可以使用语法略有不同的**new**和**delete**来分配固定大小的数组。数组分配与非数组分配不兼容，将两者混合会导致堆损坏。分配数组还会分配内存来跟踪数组的大小，以便以后以实现定义的方式删除。

```cpp
// Allocates memory for an array of 256 ints
int *foo = new int[256];
// Deletes an array of 256 ints at foo
delete[] foo;

```


When using **new** and **delete** instead [**malloc** and **free**](http://stackoverflow.com/documentation/c/4726/memory-management), the constructor and destructor will get executed (Similar to stack based objects). This is why **new** and **delete** are prefered over **malloc** and **free**.

> 当使用**new**和**delete**代替[**malloc**和**free**]时(http://stackoverflow.com/documentation/c/4726/memory-management)，构造函数和析构函数将被执行（类似于基于堆栈的对象）。这就是为什么**new**和**delete**比**malloc**和**free**更受欢迎的原因。

```cpp
struct ComplexType {
    int a = 0;

    ComplexType() { std::cout << "Ctor" << std::endl; }
    ~ComplexType() { std::cout << "Dtor" << std::endl; }
};

// Allocates memory for a ComplexType, and calls its constructor
ComplexType *foo = new ComplexType();
//Calls the destructor for ComplexType() and deletes memory for a Complextype at pC
delete foo;

```


From C++11 on, the use of [smart pointers](http://stackoverflow.com/documentation/c%2b%2b/509/smart-pointers) is recommended for indicating ownership.

> 从C++11开始，[智能指针]的使用(http://stackoverflow.com/documentation/c%2b%2b/509/smart-指针）来指示所有权。


C++14 added `std::make_unique` to the STL, changing the recommendation to favor `std::make_unique` or `std::make_shared` instead of using naked **new** and **delete**.

> C++14在STL中添加了“std:：make_unique”，将建议改为支持“std::：make_unique”或“std：：make_shared”，而不是使用裸**新**和**删除**。



## Placement new



There are situations when we don't want to rely upon Free Store for allocating memory and we want to use custom memory allocations using `new`.

> 有些情况下，我们不想依赖Free Store来分配内存，而是希望使用“new”来使用自定义内存分配。


For these situations we can use `Placement New`, where we can tell `new' operator to allocate memory from a pre-allocated memory location

> 对于这些情况，我们可以使用“Placement New”，其中我们可以告诉“New”运算符从预先分配的内存位置分配内存

For example

```cpp
int a4byteInteger;

char *a4byteChar = new (&a4byteInteger) char[4];

```


In this example, the memory pointed by `a4byteChar` is 4 byte allocated to 'stack' via integer variable `a4byteInteger`.

> 在本例中，“a4byteChar”所指向的内存是通过整数变量“a4beterInteger”分配给“stack”的4字节。


The benefit of this kind of memory allocation is the fact that programmers control the allocation. In the example above, since `a4byteInteger` is allocated on stack, we don't need to make an explicit call to 'delete a4byteChar`.

> 这种内存分配的好处是程序员可以控制分配。在上面的例子中，由于`a4byteInteger`是在堆栈上分配的，所以我们不需要显式调用`delete a4byteChar`。


Same behavior can be achieved for dynamic allocated memory also. For example

> 对于动态分配的内存也可以实现相同的行为。例如

```cpp
int *a8byteDynamicInteger = new int[2];

char *a8byteChar = new (a8byteDynamicInteger) char[8];

```


In this case, the memory pointer by `a8byteChar` will be referring to dynamic memory allocated by `a8byteDynamicInteger`. In this case however, we need to explicitly call`delete a8byteDynamicInteger` to release the memory

> 在这种情况下，“a8byteChar”的内存指针将指向“a8byteDynamicInteger”分配的动态内存。然而，在这种情况下，我们需要显式调用“delete a8bytDynamicInteger”来释放内存

Another example for C++ Class

```cpp
struct ComplexType {
    int a;

    ComplexType() : a(0) {}
    ~ComplexType() {}
};

int main() {
    char* dynArray = new char[256];

    //Calls ComplexType's constructor to initialize memory as a ComplexType
    new((void*)dynArray) ComplexType();

    //Clean up memory once we're done
    reinterpret_cast<ComplexType*>(dynArray)->~ComplexType();
    delete[] dynArray;

    //Stack memory can also be used with placement new
    alignas(ComplexType) char localArray[256]; //alignas() available since C++11

    new((void*)localArray) ComplexType();

    //Only need to call the destructor for stack memory
    reinterpret_cast<ComplexType*>(localArray)->~ComplexType();

    return 0;
}

```



## Stack



The stack is a small region of memory into which temporary values are placed during execution. Allocating data into the stack is very fast compared to heap allocation, as all the memory has already been assigned for this purpose.

> 堆栈是内存中的一个小区域，在执行过程中会将临时值放入其中。与堆分配相比，将数据分配到堆栈中的速度非常快，因为所有内存都已为此目的分配。

```cpp
int main() {
    int a = 0; //Stored on the stack
    return a;
}

```


The stack is named because chains of function calls will have their temporary memory 'stacked' on top of each other, each one using a separate small section of memory.

> 之所以命名堆栈，是因为函数调用链的临时内存会“堆叠”在一起，每个调用都使用一小段单独的内存。

```cpp
float bar() {
    //f will be placed on the stack after anything else
    float f = 2;
    return f;
}

double foo() {
    //d will be placed just after anything within main()
    double d = bar();
    return d;
}

int main() {
    //The stack has no user variables stored in it until foo() is called
    return (int)foo();
}

```


Data stored on the stack is only valid so long as the scope that allocated the variable is still active.

> 存储在堆栈上的数据只有在分配变量的作用域仍然处于活动状态时才有效。

```cpp
int* pA = nullptr;

void foo() {
    int b = *pA;
    pA = &b;
}

int main() {
    int a = 5;
    pA = &a;
    foo();
    //Undefined behavior, the value pointed to by pA is no longer in scope
    a = *pA;
}

```



#### Syntax


- ::(**opt**) new (**expression-list**)(**opt**) **new-type-id** **new-initializer**(**opt**)
- ::(**opt**) new (**expression-list**)(**opt**) (**type-id**) **new-initializer**(**opt**)
- ::(**opt**) delete **cast-expression**
- ::(**opt**) delete [] **cast-expression**
- std::unique_ptr<**type-id**> var_name(new **type-id**(**opt**)); //C++11
- std::shared_ptr<**type-id**> var_name(new **type-id**(**opt**)); //C++11
- std::shared_ptr<**type-id**> var_name = std::make_shared<**type-id**>(**opt**); //C++11
- std::unique_ptr<**type-id**> var_name = std::make_unique<**type-id**>(**opt**); //C++14



#### Remarks



A leading `::` forces the new or delete operator to be looked up in global scope, overriding any overloaded class-specific new or delete operators.

> 前导的“：：”强制在全局范围内查找new或delete运算符，从而覆盖任何重载的类特定的new或delete操作符。


The optional arguments following the `new` keyword are usually used to call [placement new](http://stackoverflow.com/documentation/c%2b%2b/2873/memory-management/9740/placement-new), but can also be used to pass additional information to the allocator, such as a tag requesting that memory be allocated from a chosen pool.

> 关键字“new”后面的可选参数通常用于调用[placement-new](http://stackoverflow.com/documentation/c%2b%2b/2873/memory-management/9740/placement-new），但是也可以用于向分配器传递附加信息，例如请求从所选池分配内存的标签。


The type allocated is usually explicitly specified, **e.g.,** `new Foo`, but can also be written as `auto` (since C++11) or `decltype(auto)` (since C++14) to deduce it from the initializer.

> 分配的类型通常是显式指定的，**例如，**“new Foo”，但也可以写成“auto”（自C++11以来）或“decltype（auto）”（自C++14以来），以从初始值设定项推导它。


Initialization of the object allocated occurs according to the same rules as initialization of local variables. In particular, the object will be default-initialized if the initializer iso omitted, and when dynamically allocating a scalar type or an array of scalar type, there is no guarantee that the memory will be zeroed out.

> 所分配对象的初始化按照与局部变量初始化相同的规则进行。特别是，如果省略了初始化器iso，则对象将默认初始化，并且在动态分配标量类型或标量类型的数组时，无法保证内存将被清零。


An array object created by a **new-expression** must be destroyed using `delete[]`, regardless of whether the **new-expression** was written with `[]` or not. For example:

> 无论**新表达式**是否使用“[]”写入，都必须使用“delete[]”销毁由**新表达式***创建的数组对象。例如：

```cpp
using IA = int[4];
int* pIA = new IA;
delete[] pIA;  // right
// delete pIA;  // wrong

```

