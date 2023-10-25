---
tip: translate by baidu@2023-10-25 08:29:00
---
---
metaTitle: "C++ | Optimization in C++"
description: "Introduction to performance, Empty Base Class Optimization, Optimizing by executing less code, Using efficient containers, Small Object Optimization"
---

# Optimization in C++



## Introduction to performance



C and C++ are well known as high-performance languages - largely due to the heavy amount of code customization, allowing a user to specify performance by choice of structure.

> C和C++是众所周知的高性能语言，这主要是由于大量的代码自定义，允许用户通过选择结构来指定性能。


When optimizing it is important to benchmark relevant code and completely understand how the code will be used.

> 在进行优化时，对相关代码进行基准测试并完全了解如何使用代码是很重要的。

Common optimization mistakes include:


- **Premature optimization:** Complex code may perform **worse** after optimization, wasting time and effort. First priority should be to write **correct** and **maintainable** code, rather than optimized code.

> -**过早优化：**复杂的代码在优化后可能会执行**更差**，浪费时间和精力。首要任务应该是编写**正确的**和**可维护的**代码，而不是优化的代码。

- **Optimization for the wrong use case:** Adding overhead for the 1% might not be worth the slowdown for the other 99%

> -**针对错误用例的优化：**增加1%的开销可能不值得其他99%的开销降低

- **Micro-optimization:** Compilers do this very efficiently and micro-optimization can even hurt the compilers ability to further optimize the code

> -**微观优化：**编译器可以非常有效地执行此操作，微观优化甚至会损害编译器进一步优化代码的能力

Typical optimization goals are:

- To do less work
- To use more efficient algorithms/structures
- To make better use of hardware

Optimized code can have negative side effects, including:

- Higher memory usage
- Complex code -being difficult to read or maintain
- Compromised API and code design



## Empty Base Class Optimization



An object cannot occupy less than 1 byte, as then the members of an array of this type would have the same address. Thus `sizeof(T)>=1` always holds. It's also true that a derived class cannot be smaller than **any of** its base classes. However, when the base class is empty, its size is not necessarily added to the derived class:

> 一个对象不能占用少于1个字节，因为这样该类型数组的成员将具有相同的地址。因此，“sizeof（T）>=1”始终成立。派生类不能小于**其任何基类，这也是事实。但是，当基类为空时，其大小不一定会添加到派生类中：

```cpp
class Base {};

class Derived : public Base
{
public:
    int i;
};

```


In this case, it's not required to allocate a byte for `Base` within `Derived` to have a distinct address per type per object. If empty base class optimization is performed (and no padding is required), then `sizeof(Derived) == sizeof(int)`, that is, no additional allocation is done for the empty base. This is possible with multiple base classes as well (in C++, multiple bases cannot have the same type, so no issues arise from that).

> 在这种情况下，不需要为“Derived”中的“Base”分配一个字节，以便每个对象的每个类型都有一个不同的地址。如果执行空基类优化（并且不需要填充），则“sizeof（Derived）==sizeof（int）”，也就是说，不对空基类进行额外分配。这在多个基类中也是可能的（在C++中，多个基类不能具有相同的类型，因此不会产生任何问题）。


Note that this can only be performed if the first member of `Derived` differs in type from any of the base classes. This includes any direct or indirect common bases. If it's the same type as one of the bases (or there's a common base), at least allocating a single byte is required to ensure that no two distinct objects of the same type have the same address.

> 请注意，只有当“Derived”的第一个成员的类型与任何基类不同时，才能执行此操作。这包括任何直接或间接的共同基础。如果它与其中一个基的类型相同（或者有一个公共基），则至少需要分配一个字节，以确保同一类型的两个不同对象没有相同的地址。



## Optimizing by executing less code



The most straightforward approach to optimizing is by executing less code. This approach usually gives a fixed speed-up without changing the time complexity of the code.

> 最直接的优化方法是执行更少的代码。这种方法通常在不改变代码的时间复杂性的情况下提供固定的加速。


Even though this approach gives you a clear speedup, this will only give noticable improvements when the code is called a lot.

> 尽管这种方法可以明显加快速度，但当代码被大量调用时，这只会带来显著的改进。

### Removing useless code

```cpp
void func(const A *a); // Some random function

// useless memory allocation + deallocation for the instance
auto a1 = std::make_unique<A>();
func(a1.get()); 

// making use of a stack object prevents 
auto a2 = A{};
func(&a2);

```


From C++14, compilers are allowed to optimize this code to remove the allocation and matching deallocation.

> 从C++14开始，编译器可以优化此代码，以删除分配和匹配的解除分配。

### Doing code only once

```cpp
std::map<std::string, std::unique_ptr<A>> lookup;
// Slow insertion/lookup
// Within this function, we will traverse twice through the map lookup an element
// and even a thirth time when it wasn't in
const A *lazyLookupSlow(const std::string &key) {
    if (lookup.find(key) != lookup.cend())
        lookup.emplace_back(key, std::make_unique<A>());
    return lookup[key].get();
}

// Within this function, we will have the same noticeable effect as the slow variant while going at double speed as we only traverse once through the code
const A *lazyLookupSlow(const std::string &key) {
    auto &value = lookup[key];
    if (!value)
        value = std::make_unique<A>();
    return value.get();
}

```


A similar approach to this optimization can be used to implement a stable version of `unique`

> 可以使用与此优化类似的方法来实现“唯一”的稳定版本`

```cpp
std::vector<std::string> stableUnique(const std::vector<std::string> &v) {
    std::vector<std::string> result;
    std::set<std::string> checkUnique;
    for (const auto &s : v) {
        // As insert returns if the insertion was successful, we can deduce if the element was already in or not
        // This prevents an insertion, which will traverse through the map for every unique element
        // As a result we can almost gain 50% if v would not contain any duplicates
        if (checkUnique.insert(s).second)
            result.push_back(s);
    }
    return result;
}

```

### Preventing useless reallocating and copying/moving


In the previous example, we already prevented lookups in the std::set, however the `std::vector` still contains a growing algorithm, in which it will have to realloc

> 在前面的例子中，我们已经阻止了在std:：集中查找，但是“std:：vector”仍然包含一个增长算法，在该算法中，它必须realloc
its storage. This can be prevented by first reserving for the right size.

```cpp
std::vector<std::string> stableUnique(const std::vector<std::string> &v) {
    std::vector<std::string> result;
    // By reserving 'result', we can ensure that no copying or moving will be done in the vector
    // as it will have capacity for the maximum number of elements we will be inserting
    // If we make the assumption that no allocation occurs for size zero
    // and allocating a large block of memory takes the same time as a small block of memory
    // this will never slow down the program
    // Side note: Compilers can even predict this and remove the checks the growing from the generated code
    result.reserve(v.size());
    std::set<std::string> checkUnique;
    for (const auto &s : v) {
        // See example above
        if (checkUnique.insert(s).second)
            result.push_back(s);
    }
    return result;
}

```



## Using efficient containers



Optimizing by using the right data structures at the right time can change the time-complexity of the code.

> 在适当的时间使用适当的数据结构进行优化可以改变代码的时间复杂性。

```cpp
// This variant of stableUnique contains a complexity of N log(N)
// N > number of elements in v
// log(N) > insert complexity of std::set
std::vector<std::string> stableUnique(const std::vector<std::string> &v) {
    std::vector<std::string> result;
    std::set<std::string> checkUnique;
    for (const auto &s : v) {
        // See Optimizing by executing less code
        if (checkUnique.insert(s).second)
            result.push_back(s);
    }
    return result;
}

```


By using a container which uses a different implementation for storing its elements (hash container instead of tree), we can transform our implementation to complexity N.

> 通过使用一个使用不同实现来存储其元素的容器（哈希容器而不是树），我们可以将我们的实现转换为复杂性N。

As a side effect, we will call the comparison operator for std::string less, as it only has to be called when the inserted string should end up in the same bucket.

> 作为副作用，我们将为std:：string less调用比较运算符，因为只有当插入的字符串最终应该在同一个bucket中时才需要调用它。

```cpp
// This variant of stableUnique contains a complexity of N
// N > number of elements in v
// 1 > insert complexity of std::unordered_set
std::vector<std::string> stableUnique(const std::vector<std::string> &v) {
    std::vector<std::string> result;
    std::unordered_set<std::string> checkUnique;
    for (const auto &s : v) {
        // See Optimizing by executing less code
        if (checkUnique.insert(s).second)
            result.push_back(s);
    }
    return result;
}

```



## Small Object Optimization



Small object optimization is a technique which is used within low level data structures, for instance the `std::string` (Sometimes referred to as Short/Small String Optimization). It's meant to use stack space as a buffer instead of some allocated memory in case the content is small enough to fit within the reserved space.

> 小对象优化是一种用于低级别数据结构的技术，例如“std:：string”（有时称为短/小字符串优化）。它的目的是使用堆栈空间作为缓冲区，而不是一些分配的内存，以防内容小到可以容纳在保留空间内。


By adding extra memory overhead and extra calculations, it tries to prevent an expensive heap allocation. The benefits of this technique are dependent on the usage and can even hurt performance if incorrectly used.

> 通过添加额外的内存开销和额外的计算，它试图防止昂贵的堆分配。这种技术的好处取决于使用情况，如果使用不当，甚至会影响性能。

### Example


A very naive way of implementing a string with this optimization would the following:

> 用这种优化实现字符串的一种非常天真的方法是：

```cpp
#include <cstring>

class string final
{
    constexpr static auto SMALL_BUFFER_SIZE = 16;

    bool _isAllocated{false};                       ///< Remember if we allocated memory
    char *_buffer{nullptr};                         ///< Pointer to the buffer we are using
    char _smallBuffer[SMALL_BUFFER_SIZE]= {'\0'};   ///< Stack space used for SMALL OBJECT OPTIMIZATION

public:
    ~string()
    {
        if (_isAllocated)
            delete [] _buffer;
    }        

    explicit string(const char *cStyleString)
    {
        auto stringSize = std::strlen(cStyleString);
        _isAllocated = (stringSize > SMALL_BUFFER_SIZE);
        if (_isAllocated)
            _buffer = new char[stringSize];
        else
            _buffer = &_smallBuffer[0];
        std::strcpy(_buffer, &cStyleString[0]);
    }

    string(string &&rhs)
       : _isAllocated(rhs._isAllocated)
       , _buffer(rhs._buffer)
       , _smallBuffer(rhs._smallBuffer) //< Not needed if allocated
    {
        if (_isAllocated)
        {
           // Prevent double deletion of the memory
           rhs._buffer = nullptr;
        }
        else
        {
            // Copy over data
            std::strcpy(_smallBuffer, rhs._smallBuffer);
            _buffer = &_smallBuffer[0];
        }
    }
    // Other methods, including other constructors, copy constructor,
    // assignment operators have been omitted for readability
};

```


As you can see in the code above, some extra complexity has been added in order to prevent some `new` and `delete` operations. On top of this, the class has a larger memory footprint which might not be used except in a couple of cases.

> 正如您在上面的代码中看到的，为了防止一些“新建”和“删除”操作，添加了一些额外的复杂性。除此之外，该类具有较大的内存占用空间，除非在少数情况下，否则可能无法使用。


Often it is tried to encode the bool value `_isAllocated`, within the pointer `_buffer` with [bit manipulation](https://stackoverflow.com/documentation/c%2b%2b/3016/bit-manipulation) to reduce the size of a single instance (intel 64 bit: Could reduce size by 8 byte). An optimization which is only possible when its known what the alignment rules of the platform is.

> 通常尝试使用[位操作]对指针“_buffer”内的布尔值“_isAllocated”进行编码(https://stackoverflow.com/documentation/c%2b%2b/3016/bit-操作）来减少单个实例的大小（intel 64位：可以将大小减少8字节）。只有当知道平台的对齐规则是什么时，才有可能进行优化。

### When to use?


As this optimization adds a lot of complexity, it is not recommended to use this optimization on every single class. It will often be encountered in commonly used, low-level data structures. In common C++11 `standard library` implementations one can find usages in [`std::basic_string<>`](https://stackoverflow.com/documentation/c%2b%2b/488/stdstring) and [`std::function<>`](https://stackoverflow.com/documentation/c%2b%2b/2294/stdfunction-to-wrap-any-element-that-is-callable).

> 由于这种优化增加了很多复杂性，因此不建议在每个类上使用这种优化。它经常出现在常用的低级数据结构中。在常见的C++11“标准库”实现中，可以在[`std:：basic_string<>`]中找到用法(https://stackoverflow.com/documentation/c%2b%2b/488/stdstring)和[`std:：function<>`](https://stackoverflow.com/documentation/c%2b%2b/2294/stdfunction-包装任何可调用的元素）。


As this optimization only prevents memory allocations when the stored data is smaller than the buffer, it will only give benefits if the class is often used with small data.

> 由于这种优化只会在存储的数据小于缓冲区时阻止内存分配，因此只有当类经常与小数据一起使用时，它才会带来好处。


A final drawback of this optimization is that extra effort is required when moving the buffer, making the move-operation more expensive than when the buffer would not be used. This is especially true when the buffer contains a non-POD type.

> 这种优化的最后一个缺点是，移动缓冲区时需要付出额外的努力，这使得移动操作比不使用缓冲区时更昂贵。当缓冲区包含非POD类型时尤其如此。

