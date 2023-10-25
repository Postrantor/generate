---
tip: translate by baidu@2023-10-25 08:32:02
---
---

metaTitle: "C++ | std::array"
description: "Initializing an std::array, Element access, Iterating through the Array, Checking size of the Array, Changing all array elements at once"
-------------------------------------------------------------------------------------------------------------------------------------------------------

# std::array

## Initializing an std::array

**Initializing `std::array<T, N>`, where `T` is a scalar type and `N` is the number of elements of type `T`**

> **正在初始化“std:：array＜T，N＞”，其中“T”是标量类型，“N”是类型为“T”的元素数 `**

If `T` is a scalar type, `std::array` can be initialized in the following ways:

> 如果“T”是标量类型，则可以通过以下方式初始化“std:：array”：

```cpp
// 1) Using aggregate-initialization
std::array<int, 3> a{ 0, 1, 2 };
// or equivalently
std::array<int, 3> a = { 0, 1, 2 };

// 2) Using the copy constructor
std::array<int, 3> a{ 0, 1, 2 };
std::array<int, 3> a2(a);
// or equivalently
std::array<int, 3> a2 = a;

// 3) Using the move constructor
std::array<int, 3> a = std::array<int, 3>{ 0, 1, 2 };

```

<strong>Initializing `std::array<T, N>`, where `T` is a non-scalar type and `N` is the
number of elements of type `T`</strong>

If `T` is a non-scalar type `std::array` can be initialized in the following ways:

> 如果“T”是非标量类型，则可以通过以下方式初始化“std:：array”：

```cpp
struct A { int values[3]; }; // An aggregate type

// 1) Using aggregate initialization with brace elision
// It works only if T is an aggregate type!
std::array<A, 2> a{ 0, 1, 2, 3, 4, 5 };
// or equivalently
std::array<A, 2> a = { 0, 1, 2, 3, 4, 5 };

// 2) Using aggregate initialization with brace initialization of sub-elements
std::array<A, 2> a{ A{ 0, 1, 2 }, A{ 3, 4, 5 } };
// or equivalently
std::array<A, 2> a = { A{ 0, 1, 2 }, A{ 3, 4, 5 } };

// 3)
std::array<A, 2> a{{ { 0, 1, 2 }, { 3, 4, 5 } }};
// or equivalently
std::array<A, 2> a = {{ { 0, 1, 2 }, { 3, 4, 5 } }};

// 4) Using the copy constructor
std::array<A, 2> a{ 1, 2, 3 };
std::array<A, 2> a2(a);
// or equivalently
std::array<A, 2> a2 = a;

// 5) Using the move constructor
std::array<A, 2> a = std::array<A, 2>{ 0, 1, 2, 3, 4, 5 };

```

## Element access

**1. `at(pos)`**

Returns a reference to the element at position `pos` with bounds checking. If `pos` is not within the range of the container, an exception of type `std::out_of_range` is thrown.

> 通过边界检查返回对位置“pos”处元素的引用。如果“pos”不在容器的范围内，则抛出类型为“std:：out_of_range”的异常。

The complexity is constant O(1).

```cpp
#include <array>

int main()
{
    std::array<int, 3> arr;

    // write values
    arr.at(0) = 2;
    arr.at(1) = 4;
    arr.at(2) = 6;
        
    // read values
    int a = arr.at(0); // a is now 2
    int b = arr.at(1); // b is now 4
    int c = arr.at(2); // c is now 6

    return 0;
}

```

**2) `operator[pos]`**

Returns a reference to the element at position `pos` without bounds checking.  If `pos` is not within the range of the container, a runtime **segmentation violation** error can occur. This method provides element access equivalent to classic arrays and thereof more efficient than `at(pos)`.

> 返回对位置“pos”处元素的引用，而不进行边界检查。如果“pos”不在容器的范围内，则可能发生运行时**分段冲突**错误。这种方法提供了相当于经典数组的元素访问，并且比“at（pos）”更有效。

The complexity is constant O(1).

```cpp
#include <array>

int main()
{
    std::array<int, 3> arr;

    // write values
    arr[0] = 2;
    arr[1] = 4;
    arr[2] = 6;
        
    // read values
    int a = arr[0]; // a is now 2
    int b = arr[1]; // b is now 4
    int c = arr[2]; // c is now 6

    return 0;
}

```

**3) `std::get<pos>`**

This **non-member** function returns a reference to the element at **compile-time constant** position `pos` without bounds checking. If `pos` is not within the range of the container, a runtime **segmentation violation** error can occur.

> 这个**非成员**函数返回对**编译时常数**位置“pos”处元素的引用，而不进行边界检查。如果“pos”不在容器的范围内，则可能发生运行时**分段冲突**错误。

The complexity is constant O(1).

```cpp
#include <array>

int main()
{
    std::array<int, 3> arr;

    // write values
    std::get<0>(arr) = 2;
    std::get<1>(arr) = 4;
    std::get<2>(arr) = 6;
        
    // read values
    int a = std::get<0>(arr); // a is now 2
    int b = std::get<1>(arr); // b is now 4
    int c = std::get<2>(arr); // c is now 6

    return 0;
}

```

**4) `front()`**

Returns a reference to the first element in container. Calling `front()` on an empty container is undefined.

> 返回对容器中第一个元素的引用。对空容器调用“front（）”是未定义的。

The complexity is constant O(1).

**Note:** For a container c, the expression `c.front()` is equivalent to `*c.begin()`.

> **注意：**对于容器 c，表达式“c.front（）”等效于“*.c.begin（）”。

```cpp
#include <array>

int main()
{
    std::array<int, 3> arr{ 2, 4, 6 };

    int a = arr.front(); // a is now 2

    return 0;
}

```

**5) `back()`**

Returns reference to the last element in the container.
Calling `back()` on an empty container is undefined.

The complexity is constant O(1).

```cpp
#include <array>

int main()
{
    std::array<int, 3> arr{ 2, 4, 6 };

    int a = arr.back(); // a is now 6

    return 0;
}

```

**6) `data()`**

Returns pointer to the underlying array serving as element storage. The pointer is such that `range [data(); data() + size())` is always a valid range, even if the container is empty (`data()` is not dereferenceable in that case).

> 返回指向用作元素存储的基础数组的指针。指针使得“range[data（）；data（）+size（））”始终是一个有效的范围，即使容器是空的（在这种情况下，“data（）”是不可取消引用的）。

The complexity is constant O(1).

```cpp
#include <iostream>
#include <cstring>
#include <array>

int main ()
{
    const char* cstr = "Test string";
    std::array<char, 12> arr;
    
    std::memcpy(arr.data(), cstr, 12); // copy cstr to arr
    
    std::cout << arr.data(); // outputs: Test string
    
    return 0;
}

```

## Iterating through the Array

`std::array` being a STL container, can use range-based for loop similar to other containers like `vector`

```cpp
int main() {
     std::array<int, 3> arr = { 1, 2, 3 };
     for (auto i : arr)
         cout << i << '\n';
}

```

## Checking size of the Array

One of the main advantage of `std::array` as compared to `C` style array is that we can check the size of the array using `size()` member function

> 与“C”风格的数组相比，“std:：array”的主要优点之一是，我们可以使用“size（）”成员函数检查数组的大小

```cpp
int main() {
    std::array<int, 3> arr = { 1, 2, 3 };
    cout << arr.size() << endl;
}


```

## Changing all array elements at once

The member function `fill()` can be used on `std::array` for changing the values at once post initialization

> 成员函数“fill（）”可用于“std:：array”，用于在初始化后立即更改值

```cpp
int main() {
    
    std::array<int, 3> arr = { 1, 2, 3 };
    // change all elements of the array to 100
    arr.fill(100);
    
}

```

#### Parameters

| Parameter       | Definition                                   |  |  |  |  |  |  |  |  |
| --------------- | -------------------------------------------- | - | - | - | - | - | - | - | - |
| `class T`       | Specifies the data type of array members     |  |  |  |  |  |  |  |  |
| `std::size_t N` | Specifies the number of members in the array |  |  |  |  |  |  |  |  |

#### Remarks

Use of a `std::array` requires the inclusion of the `<array>` header using `#include <array>`.

> 使用“std:：array”需要使用“#include＜array＞”包含“＜array>”标头。
