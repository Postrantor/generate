---
tip: translate by baidu@2023-10-25 08:20:23
---
---
metaTitle: "C++ | Arrays"
description: "Array initialization, A fixed size raw array matrix (that is, a 2D raw array)., Dynamically sized raw array, Array size: type safe at compile time., Expanding dynamic size array by using std::vector., A dynamic size matrix using std::vector for storage."
---

# Arrays



Arrays are elements of the same type placed in adjoining memory locations. The elements can be individually referenced by a unique identifier with an added index.

> 数组是放置在相邻内存位置的相同类型的元素。元素可以由添加了索引的唯一标识符单独引用。


This allows you to declare multiple variable values of a specific type and access them individually without needing to declare a variable for each value.

> 这允许您声明特定类型的多个变量值并单独访问它们，而无需为每个值声明一个变量。



## Array initialization



An array is just a block of sequential memory locations for a specific type of variable. Arrays are allocated the same way as normal variables, but with square brackets appended to its name `[]` that contain the number of elements that fit into the array memory.

> 数组只是一个特定类型变量的顺序存储位置块。数组的分配方式与普通变量相同，但在其名称“[]”后加上方括号，方括号包含适合数组内存的元素数量。


The following example of an array uses the typ `int`, the variable name `arrayOfInts`, and the number of elements `[5]` that the array has space for:

> 以下数组示例使用类型“int”、变量名“arrayOfInts”以及数组有空间容纳的元素数“[5]”：

```cpp
int arrayOfInts[5];

```

An array can be declared and initialized at the same time like this

```cpp
int arrayOfInts[5] = {10, 20, 30, 40, 50};

```


When initializing an array by listing all of its members, it is not necessary to include the number of elements inside the square brackets. It will be automatically calculated by the compiler. In the following example, it's 5:

> 当通过列出数组的所有成员来初始化数组时，不需要在方括号内包含元素的数量。它将由编译器自动计算。在以下示例中，它是5：

```cpp
int arrayOfInts[] = {10, 20, 30, 40, 50};

```


It is also possible to initialize only the first elements while allocating more space. In this case, defining the length in brackets is mandatory. The following will allocate an array of length 5 with partial initialization, the compiler initializes all remaining elements with the standard value of the element type, in this case zero.

> 还可以在分配更多空间的同时仅初始化第一元素。在这种情况下，必须在括号中定义长度。下面将分配一个长度为5的数组，并进行部分初始化，编译器使用元素类型的标准值初始化所有剩余元素，在这种情况下为零。

```cpp
int arrayOfInts[5] = {10,20}; // means 10, 20, 0, 0, 0

```

Arrays of other basic data types may be initialized in the same way.

```cpp
char arrayOfChars[5]; // declare the array and allocate the memory, don't initialize

char arrayOfChars[5] = { 'a', 'b', 'c', 'd', 'e' } ; //declare and initialize

double arrayOfDoubles[5] = {1.14159, 2.14159, 3.14159, 4.14159, 5.14159};

string arrayOfStrings[5] = { "C++", "is", "super", "duper", "great!"};

```


It is also important to take note that when accessing array elements, the array's element index(or position) starts from 0.

> 同样重要的是要注意，在访问数组元素时，数组的元素索引（或位置）从0开始。

```cpp
int array[5] = { 10/*Element no.0*/, 20/*Element no.1*/, 30, 40, 50/*Element no.4*/};
std::cout << array[4]; //outputs 50
std::cout << array[0]; //outputs 10

```



## A fixed size raw array matrix (that is, a 2D raw array).


```cpp
// A fixed size raw array matrix (that is, a 2D raw array).
#include <iostream>
#include <iomanip>
using namespace std;

auto main() -> int
{
    int const   n_rows  = 3;
    int const   n_cols  = 7;
    int const   m[n_rows][n_cols] =             // A raw array matrix.
    {
        {  1,  2,  3,  4,  5,  6,  7 },
        {  8,  9, 10, 11, 12, 13, 14 },
        { 15, 16, 17, 18, 19, 20, 21 }
    };
    
    for( int y = 0; y < n_rows; ++y )
    {
        for( int x = 0; x < n_cols; ++x )
        {
            cout << setw( 4 ) << m[y][x];       // Note: do NOT use m[y,x]!
        }
        cout << '\n';
    }
}

```

Output:


C++ doesn't support special syntax for indexing a multi-dimensional array. Instead such an array is viewed as an array of arrays (possibly of arrays, and so on), and the ordinary single index notation `[`**`i`**`]` is used for each level. In the example above `m[y]` refers to row `y` of `m`, where `y` is a zero-based index. Then this row can be indexed in turn, e.g. `m[y][x]`, which refers to the `x`<sup>th</sup> item – or column – of row `y`.

> C++不支持对多维数组进行索引的特殊语法。相反，这样的数组被视为数组的数组（可能是数组的数组，等等），并且普通的单索引表示法“[`**`i`**`]”用于每个级别。在上面的例子中，“m[y]”指的是“m”的行“y”，其中“y”是一个从零开始的索引。然后可以依次对该行进行索引，例如“m[y][x]'，它指的是行“y”的第x'<sup>个</sup>项或列。


I.e. the last index varies fastest, and in the declaration the range of this index, which here is the number of columns per row, is the last and “innermost” size specified.

> 也就是说，最后一个索引变化最快，在声明中，这个索引的范围（这里是每行的列数）是指定的最后一个也是“最里面”的大小。


Since C++ doesn't provide built-in support for dynamic size arrays, other than dynamic allocation, a dynamic size matrix is often implemented as a class. Then the raw array matrix indexing notation `m[y][x]` has some cost, either by exposing the implementation (so that e.g. a view of a transposed matrix becomes practically impossible) or by adding some overhead and slight inconvenience when it's done by returning a proxy object from `operator[]`. And so the indexing notation for such an abstraction can and will usually be different, both in look-and-feel and in the order of indices, e.g. `m(x,y)` or `m.at(x,y)` or `m.item(x,y)`.

> 由于除了动态分配之外，C++不提供对动态大小数组的内置支持，因此动态大小矩阵通常作为类来实现。然后，原始数组矩阵索引表示法“m[y][x]”具有一定的成本，要么是通过公开实现（使得转置矩阵的视图实际上变得不可能），要么是在通过从“operator[]”返回代理对象来完成时添加一些开销和轻微的不便。因此，这种抽象的索引表示法在外观和索引顺序上可以而且通常都是不同的，例如“m（x，y）”或“m.at（x，y）”或”m.item（x，y）“。



## Dynamically sized raw array


```cpp
// Example of raw dynamic size array. It's generally better to use std::vector.
#include <algorithm>            // std::sort
#include <iostream>
using namespace std;

auto int_from( istream& in ) -> int { int x; in >> x; return x; }

auto main()
    -> int
{
    cout << "Sorting n integers provided by you.\n";
    cout << "n? ";
    int const   n   = int_from( cin );
    int*        a   = new int[n];       // ← Allocation of array of n items.
    
    for( int i = 1; i <= n; ++i )
    {
        cout << "The #" << i << " number, please: ";
        a[i-1] = int_from( cin );
    }

    sort( a, a + n );
    for( int i = 0; i < n; ++i ) { cout << a[i] << ' '; }
    cout << '\n';
    
    delete[] a;
}

```


A program that declares an array `T a[n];` where `n` is determined a run-time, can compile with certain compilers that support C99 **variadic length arrays** (VLAs) as a language extension. But VLAs are not supported by standard C++. This example shows how to manually allocate a dynamic size array via a `new[]`-expression,

> 一种声明数组`T A[n]；`的程序其中“n”被确定为运行时，可以使用支持C99**可变长度数组**（VLA）作为语言扩展的某些编译器进行编译。但是VLA不受标准C++的支持。此示例显示了如何通过“new[]”表达式手动分配动态大小数组，

```cpp
int*        a   = new int[n];       // ← Allocation of array of n items.

```

… then use it, and finally deallocate it via a `delete[]`-expression:

```cpp
delete[] a;

```


The array allocated here has indeterminate values, but it can be zero-initialized by just adding an empty parenthesis `()`, like this: `new int[n]()`. More generally, for arbitrary item type, this performs a **value-initialization**.

> 这里分配的数组有不确定的值，但只需添加一个空括号“（）”，就可以对其进行零初始化，比如：“new int[n]（）”。更一般地说，对于任意项目类型，这将执行**值初始化**。


As part of a function down in a call hierarchy this code would not be exception safe, since an exception before the `delete[]` expression (and after the `new[]`) would cause a memory leak. One way to address that issue is to automate the cleanup via e.g. a `std::unique_ptr` smart pointer. But a generally better way to address it is to just use a `std::vector`: that's what `std::vector` is there for.

> 作为调用层次结构中函数的一部分，此代码不会是异常安全的，因为在“delete[]”表达式之前（和“new[]”之后）发生异常会导致内存泄漏。解决该问题的一种方法是通过“std:：unique_ptr”智能指针自动进行清理。但解决这个问题的一个更好的方法是只使用“std:：vector”：这就是“std::vector”的作用。



## Array size: type safe at compile time.


```cpp
#include <stddef.h>     // size_t, ptrdiff_t

//----------------------------------- Machinery:

using Size = ptrdiff_t;

template< class Item, size_t n >
constexpr auto n_items( Item (&)[n] ) noexcept
    -> Size
{ return n; }


//----------------------------------- Usage:

#include <iostream>
using namespace std;
auto main()
    -> int
{
    int const   a[]     = {3, 1, 4, 1, 5, 9, 2, 6, 5, 4};
    Size const  n       = n_items( a );
    int         b[n]    = {};       // An array of the same size as a.
    
    (void) b;
    cout << "Size = " << n << "\n";
}

```


The C idiom for array size, `sizeof(a)/sizeof(a[0])`, will accept a pointer as argument and will then generally yield an incorrect result.

> 数组大小的C习惯用法“sizeof（a）/sizeof（a[0]）”将接受指针作为参数，然后通常会产生不正确的结果。

For C++11

using C++11 you can do:

```cpp
std::extent<decltype(MyArray)>::value;

```

Example:

```cpp
char MyArray[] = { 'X','o','c','e' };
const auto n = std::extent<decltype(MyArray)>::value;
std::cout << n << "\n"; // Prints 4

```


Up till C++17 (forthcoming as of this writing) C++ had no built-in core language or standard library utility to obtain the size of an array, but this can be implemented by passing the array **by reference** to a function template, as shown above. Fine but important point: the template size parameter is a `size_t`, somewhat inconsistent with the signed `Size` function result type, in order to accommodate the g++ compiler which sometimes insists on `size_t` for template matching.

> 直到C++17（在撰写本文时即将发布），C++还没有内置的核心语言或标准库实用程序来获取数组的大小，但这可以通过将数组**通过引用**传递给函数模板来实现，如上所示。很好但很重要的一点是：模板大小参数是一个“size_t”，与带符号的“size”函数结果类型有些不一致，以适应g++编译器，后者有时坚持使用“size _t”进行模板匹配。


With C++17 and later one may instead use [`std::size`](http://en.cppreference.com/w/cpp/iterator/size), which is specialized for arrays.

> 对于C++17及更高版本，可以使用[`std:：size`](http://en.cppreference.com/w/cpp/iterator/size)，专门用于阵列。



## Expanding dynamic size array by using std::vector.


```cpp
// Example of std::vector as an expanding dynamic size array.
#include <algorithm>            // std::sort
#include <iostream>
#include <vector>               // std::vector
using namespace std;

int int_from( std::istream& in ) { int x = 0; in >> x; return x; }

int main()
{
    cout << "Sorting integers provided by you.\n";
    cout << "You can indicate EOF via F6 in Windows or Ctrl+D in Unix-land.\n";
    vector<int> a;      // ← Zero size by default.

    while( cin )
    {
        cout << "One number, please, or indicate EOF: ";
        int const x = int_from( cin );
        if( !cin.fail() ) { a.push_back( x ); }  // Expands as necessary.
    }

    sort( a.begin(), a.end() );
    int const n = a.size();
    for( int i = 0; i < n; ++i ) { cout << a[i] << ' '; }
    cout << '\n';
}

```

`std::vector` is a standard library class template that provides the notion of a variable size array. It takes care of all the memory management, and the buffer is contiguous so a pointer to the buffer (e.g. `&v[0]` or `v.data()`) can be passed to API functions requiring a raw array. A `vector` can even be expanded at run time, via e.g. the `push_back` member function that appends an item.


The complexity of the sequence of **n** `push_back` operations, including the copying or moving involved in the vector expansions, is amortized O(**n**). “Amortized”: on average.

> **n**“push_back”操作序列的复杂性，包括向量展开中涉及的复制或移动，分摊为O（**n**）。“摊销”：平均。


Internally this is usually achieved by the vector **doubling** its buffer size, its capacity, when a larger buffer is needed. E.g. for a buffer starting out as size 1, and being repeatedly doubled as needed for **n**=17 `push_back` calls, this involves 1 + 2 + 4 + 8 + 16 = 31 copy operations, which is less than 2×**n** = 34. And more generally the sum of this sequence can't exceed 2×**n**.

> 在内部，当需要更大的缓冲区时，这通常通过向量**将其缓冲区大小和容量加倍来实现。例如，对于一个大小为1的缓冲区，并根据需要对**n**=17个“push_back”调用重复加倍，这涉及1+2+4+8+16=31个复制操作，小于2×**n**=34。更一般地，这个序列的和不能超过2×。


Compared to the dynamic size raw array example, this `vector`-based code does not require the user to supply (and know) the number of items up front. Instead the vector is just expanded as necessary, for each new item value specified by the user.

> 与动态大小的原始数组示例相比，这种基于“矢量”的代码不需要用户预先提供（并知道）项目的数量。相反，对于用户指定的每个新项目值，矢量只是根据需要进行扩展。



## A dynamic size matrix using std::vector for storage.



Unfortunately as of C++14 there's no dynamic size matrix class in the C++ standard library. Matrix classes that support dynamic size are however available from a number of 3<sup>rd</sup> party libraries, including the Boost Matrix library (a sub-library within the Boost library).

> 不幸的是，从C++14开始，C++标准库中没有动态大小矩阵类。然而，支持动态大小的矩阵类可从许多3<sup>rd</sup>方库中获得，包括Boost矩阵库（Boost库中的一个子库）。


If you don't want a dependency on Boost or some other library, then one poor man's dynamic size matrix in C++ is just like

> 如果你不想依赖Boost或其他库，那么C++中一个穷人的动态大小矩阵就像

```cpp
vector<vector<int>> m( 3, vector<int>( 7 ) );

```


… where `vector` is `std::vector`. The matrix is here created by copying a row vector **n** times where **n** is the number of rows, here 3. It has the advantage of providing the same `m[y][x]` indexing notation as for a fixed size raw array matrix, but it's a bit inefficient because it involves a dynamic allocation for each row, and it's a bit unsafe because it's possible to inadvertently resize a row.

> …其中“vector”是“std:：vector”。矩阵是通过复制行向量**n**次来创建的，其中**n**是行数，这里是3。它的优点是提供与固定大小的原始数组矩阵相同的“m[y][x]”索引表示法，但它有点低效，因为它涉及到对每一行的动态分配，而且它有点不安全，因为可能会无意中调整行的大小。


A more safe and efficient approach is to use a single vector as **storage** for the matrix, and map the client code's (**x**, **y**) to a corresponding index in that vector:

> 一种更安全有效的方法是使用单个向量作为矩阵的**存储**，并将客户端代码的（**x**，**y**）映射到该向量中的相应索引：

```cpp
// A dynamic size matrix using std::vector for storage.

//--------------------------------------------- Machinery:
#include <algorithm>        // std::copy
#include <assert.h>         // assert
#include <initializer_list> // std::initializer_list
#include <vector>           // std::vector
#include <stddef.h>         // ptrdiff_t

namespace my {
    using Size = ptrdiff_t;
    using std::initializer_list;
    using std::vector;

    template< class Item >
    class Matrix
    {
    private:
        vector<Item>    items_;
        Size            n_cols_;
        
        auto index_for( Size const x, Size const y ) const
            -> Size
        { return y*n_cols_ + x; }

    public:
        auto n_rows() const -> Size { return items_.size()/n_cols_; }
        auto n_cols() const -> Size { return n_cols_; }

        auto item( Size const x, Size const y )
            -> Item&
        { return items_[index_for(x, y)]; }
        
        auto item( Size const x, Size const y ) const
            -> Item const&
        { return items_[index_for(x, y)]; }

        Matrix(): n_cols_( 0 ) {}

        Matrix( Size const n_cols, Size const n_rows )
            : items_( n_cols*n_rows )
            , n_cols_( n_cols )
        {}
        
        Matrix( initializer_list< initializer_list<Item> > const& values )
            : items_()
            , n_cols_( values.size() == 0? 0 : values.begin()->size() )
        {
            for( auto const& row : values )
            {
                assert( Size( row.size() ) == n_cols_ );
                items_.insert( items_.end(), row.begin(), row.end() );
            }
        }
    };
}  // namespace my

//--------------------------------------------- Usage:
using my::Matrix;

auto some_matrix()
    -> Matrix<int>
{
    return
    {
        {  1,  2,  3,  4,  5,  6,  7 },
        {  8,  9, 10, 11, 12, 13, 14 },
        { 15, 16, 17, 18, 19, 20, 21 }
    };
}

#include <iostream>
#include <iomanip>
using namespace std;
auto main() -> int
{
    Matrix<int> const m = some_matrix();
    assert( m.n_cols() == 7 );
    assert( m.n_rows() == 3 );
    for( int y = 0, y_end = m.n_rows(); y < y_end; ++y )
    {
        for( int x = 0, x_end = m.n_cols(); x < x_end; ++x )
        {
            cout << setw( 4 ) << m.item( x, y );        // ← Note: not `m[y][x]`!
        }
        cout << '\n';
    }
}

```

Output:


The above code is not industrial grade: it's designed to show the basic principles, and serve the needs of students learning C++.

> 上面的代码不是工业级的：它旨在展示基本原理，并满足学习C++的学生的需求。


For example, one may define `operator()` overloads to simplify the indexing notation.

> 例如，可以定义“operator（）”重载以简化索引表示法。

