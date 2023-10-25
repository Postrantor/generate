---
tip: translate by baidu@2023-10-25 08:33:12
---
---
metaTitle: "C++ | std::vector"
description: "Accessing Elements, Initializing a std::vector, Deleting Elements, Iterating Over std::vector, vector<bool>: The Exception To So Many, So Many Rules, Inserting Elements, Using std::vector as a C array, Finding an Element in std::vector, Concatenating Vectors, Reducing the Capacity of a Vector, Using a Sorted Vector for Fast Element Lookup, Matrices Using Vectors, Iterator/Pointer Invalidation, Vector size and capacity, Find max and min Element and Respective Index in a Vector, Converting an array to std::vector, Functions Returning Large Vectors"
---

# std::vector



A vector is a dynamic array with automatically handled storage. The elements in a vector can be accessed just as efficiently as those in an array with the advantage being that vectors can dynamically change in size.

> 向量是一个具有自动处理存储的动态数组。向量中的元素可以像数组中的元素一样有效地访问，其优点是向量可以动态地改变大小。


In terms of storage the vector data is (usually) placed in dynamically allocated memory thus requiring some minor overhead; conversely `C-arrays` and `std::array` use automatic storage relative to the declared location and thus do not have any overhead.

> 在存储方面，矢量数据（通常）被放置在动态分配的存储器中，因此需要一些较小的开销；相反，“C-arrays”和“std:：array”使用相对于声明位置的自动存储，因此没有任何开销。



## Accessing Elements



There are two primary ways of accessing elements in a [`std::vector`](http://en.cppreference.com/w/cpp/container/vector)

> 访问[`std:：vector`]中的元素有两种主要方式(http://en.cppreference.com/w/cpp/container/vector)

- index-based access
- [iterators](https://stackoverflow.com/documentation/c%2B%2B/473/iterators/1709/overview#t=20160613174720948501)

### Index-based access:


This can be done either with the subscript operator [`[]`](http://en.cppreference.com/w/cpp/container/vector/operator_at), or the member function [`at()`](http://en.cppreference.com/w/cpp/container/vector/at).

> 这可以使用下标运算符[`[]`]来完成(http://en.cppreference.com/w/cpp/container/vector/operator_at)，或成员函数[`at（）`](http://en.cppreference.com/w/cpp/container/vector/at)。


Both return a reference to the element at the respective position in the `std::vector` (unless it's a [`vector<bool>`](http://stackoverflow.com/documentation/c%2b%2b/511/stdvector/2561/vectorbool-the-exception-to-many-many-rules#t=20160725190704070024)), so that it can be read as well as modified (if the vector is not `const`).

> 两者都返回对“std:：vector”中相应位置的元素的引用（除非它是[`vector<bool>`](http://stackoverflow.com/documentation/c%2b%2b/511/stdvector/2561/vectorbool-许多规则的例外#t=20160725190704070024），以便它既可以被读取也可以被修改（如果向量不是“const”）。

`[]` and `at()` differ in that `[]` is not guaranteed to perform any bounds checking, while `at()` does. Accessing elements where `index < 0` or `index >= size` is [undefined behavior](http://stackoverflow.com/documentation/c%2b%2b/1812/undefined-behavior#t=201607221823520289181) for `[]`, while `at()` throws a [`std::out_of_range`](http://en.cppreference.com/w/cpp/error/out_of_range) exception.


****Note:**** The examples below use C++11-style initialization for clarity, but the operators can be used with all versions (unless marked C++11).

> ****注意：***为了清楚起见，下面的示例使用C++11风格的初始化，但运算符可以用于所有版本（除非标记为C++11）。

```cpp
std::vector<int> v{ 1, 2, 3 };

```

```cpp
// using []
int a = v[1];    // a is 2
v[1] = 4;        // v now contains { 1, 4, 3 }

// using at()
int b = v.at(2); // b is 3
v.at(2) = 5;     // v now contains { 1, 4, 5 }
int c = v.at(3); // throws std::out_of_range exception

```


Because the `at()` method performs bounds checking and can throw exceptions, it is slower than `[]`. This makes `[]` preferred code where the semantics of the operation guarantee that the index is in bounds.

> 因为“at（）”方法执行边界检查并可以抛出异常，所以它比“[]”慢。这使得“[]”成为首选代码，其中操作的语义保证索引在边界内。

In any case, accesses to elements of vectors are done in constant time. That means accessing to the first element of the vector has the same cost (in time) of accessing the second element, the third element and so on.

> 在任何情况下，对向量元素的访问都是在恒定时间内完成的。这意味着访问向量的第一个元素与访问第二个元素、第三个元素等具有相同的成本（在时间上）。

For example, consider this loop

```cpp
for (std::size_t i = 0; i < v.size(); ++i) {
    v[i] = 1;
}

```


Here we know that the index variable `i` is always in bounds, so it would be a waste of CPU cycles to check that `i` is in bounds for every call to `operator[]`.

> 这里我们知道索引变量“i”总是在界中，所以每次调用“operator[]”都要检查“i”是否在界中是浪费CPU周期的。


The [`front()`](http://en.cppreference.com/w/cpp/container/vector/front) and [`back()`](http://en.cppreference.com/w/cpp/container/vector/back) member functions allow easy reference access to the first and last element of the vector, respectively. These positions are frequently used, and the special accessors can be more readable than their alternatives using `[]`:

> [`front（）`](http://en.cppreference.com/w/cpp/container/vector/front)和[`back（）`](http://en.cppreference.com/w/cpp/container/vector/back)成员函数允许分别对向量的第一个和最后一个元素进行简单的引用访问。这些位置经常使用，特殊访问者可能比使用“[]”的其他访问者更具可读性：

```cpp
std::vector<int> v{ 4, 5, 6 }; // In pre-C++11 this is more verbose

int a = v.front();   // a is 4, v.front() is equivalent to v[0]
v.front() = 3;       // v now contains {3, 5, 6}
int b = v.back();    // b is 6, v.back() is equivalent to v[v.size() - 1]
v.back() = 7;        // v now contains {3, 5, 7}

```


****Note****: It is [undefined behavior](http://stackoverflow.com/documentation/c%2b%2b/1812/undefined-behavior#t=201608052302241671564) to invoke `front()` or `back()` on an empty vector. You need to check that the container is not empty using the [`empty()`](http://en.cppreference.com/w/cpp/container/vector/empty) member function (which checks if the container is empty) before calling `front()` or `back()`.  A simple example of the use of 'empty()' to test for an empty vector follows:

> ****注意***：这是【未定义的行为】(http://stackoverflow.com/documentation/c%2b%2b/1812/undefined-behavior#t=201608052302241671664）来调用空向量上的“front（）”或“back（）”。您需要使用[`empty（）`]检查容器是否为空(http://en.cppreference.com/w/cpp/container/vector/empty)成员函数（检查容器是否为空），然后调用“front（）”或“back（）”。使用“empty（）”测试空向量的简单示例如下：

```cpp
int main ()
{
  std::vector<int> v;
  int sum (0);

  for (int i=1;i<=10;i++) v.push_back(i);//create and initialize the vector

  while (!v.empty())//loop through until the vector tests to be empty
  {
     sum += v.back();//keep a running total
     v.pop_back();//pop out the element which removes it from the vector
  }

  std::cout << "total: " << sum << '\n';//output the total to the user

  return 0;
}

```


The example above creates a vector with a sequence of numbers from 1 to 10.  Then it pops the elements of the vector out until the vector is empty (using 'empty()') to prevent undefined behavior.  Then the sum of the numbers in the vector is calculated and displayed to the user.

> 上面的例子创建了一个具有从1到10的数字序列的向量。然后，它弹出向量的元素，直到向量为空（使用“empty（）”），以防止未定义的行为。然后计算矢量中的数字之和并将其显示给用户。


The [`data()`](http://en.cppreference.com/w/cpp/container/vector/data) method returns a pointer to the raw memory used by the `std::vector` to internally store its elements. This is most often used when passing the vector data to legacy code that expects a C-style array.

> [`data（）`](http://en.cppreference.com/w/cpp/container/vector/data)方法返回一个指针，指向“std:：vector”用于内部存储其元素的原始内存。这在将矢量数据传递给需要C样式数组的遗留代码时最常用。

```cpp
std::vector<int> v{ 1, 2, 3, 4 }; // v contains {1, 2, 3, 4}
int* p = v.data(); // p points to 1
*p = 4;            // v now contains {4, 2, 3, 4}
++p;               // p points to 2
*p = 3;            // v now contains {4, 3, 3, 4}
p[1] = 2;          // v now contains {4, 3, 2, 4}
*(p + 2) = 1;      // v now contains {4, 3, 2, 1}

```


Before C++11, the `data()` method can be simulated by calling `front()` and taking the address of the returned value:

> 在C++11之前，可以通过调用“front（）”并获取返回值的地址来模拟“data（）”方法：

```cpp
std::vector<int> v(4);
int* ptr = &(v.front()); // or &v[0]

```


This works because vectors are always guaranteed to store their elements in contiguous memory locations, assuming the contents of the vector doesn't override unary `operator&`.  If it does, you'll have to re-implement [`std::addressof`](http://en.cppreference.com/w/cpp/memory/addressof) in pre-C++11. It also assumes that the vector isn't empty.

> 这是因为向量总是保证将其元素存储在连续的内存位置，假设向量的内容不会覆盖一元“运算符&”。如果是这样，您将不得不重新实现[`std:：addressof`](http://en.cppreference.com/w/cpp/memory/addressof)在C++11之前。它还假设向量不是空的。

### Iterators:


Iterators are explained in more detail in the example "Iterating over `std::vector`" and the article [Iterators](https://stackoverflow.com/documentation/c%2B%2B/473/iterators/1709/overview#t=20160613174720948501). In short, they act similarly to pointers to the elements of the vector:

> Iterators在例子“Iterating over `std:：vector`”和文章[Iterators]中有更详细的解释(https://stackoverflow.com/documentation/c%2B%2B/473/iterators/1709/overview#t=20160613174720948501)。简而言之，它们的作用类似于指向向量元素的指针：

```cpp
std::vector<int> v{ 4, 5, 6 };

auto it = v.begin();
int i = *it;        // i is 4
++it; 
i = *it;            // i is 5
*it = 6;            // v contains { 4, 6, 6 }
auto e = v.end();   // e points to the element after the end of v. It can be 
                    // used to check whether an iterator reached the end of the vector:
++it; 
it == v.end();      // false, it points to the element at position 2 (with value 6)
++it;
it == v.end();      // true

```


It is consistent with the standard that a `std::vector<T>`'s iterators actually **be** `T*`s, but most standard libraries do not do this.  Not doing this both improves error messages, catches non-portable code, and can be used to instrument the iterators with debugging checks in non-release builds.  Then, in release builds, the class wrapping around the underlying pointer is optimized away.

> 与标准一致的是，“std:：vector＜T＞”的迭代器实际上**是**`T*'，但大多数标准库不这样做。不这样做既可以改进错误消息，捕获不可移植的代码，也可以用于在非发布版本中通过调试检查来检测迭代器。然后，在发行版构建中，围绕底层指针的类被优化掉了。


You can persist a reference or a pointer to an element of a vector for indirect access.  These references or pointers to elements in the `vector` remain stable and access remains defined unless you add/remove elements at or before the element in the `vector`, or you cause the `vector` capacity to change.  This is the same as the rule for invalidating iterators.

> 您可以保留指向向量元素的引用或指针，以便进行间接访问。对“vector”中元素的这些引用或指针保持稳定，访问保持定义，除非您在“vector'”中的元素处或之前添加/删除元素，或者导致“vector”容量发生变化。这与使迭代器无效的规则相同。

```cpp
std::vector<int> v{ 1, 2, 3 };
int* p = v.data() + 1;     // p points to 2
v.insert(v.begin(), 0);    // p is now invalid, accessing *p is a undefined behavior.
p = v.data() + 1;          // p points to 1
v.reserve(10);             // p is now invalid, accessing *p is a undefined behavior.
p = v.data() + 1;          // p points to 1
v.erase(v.begin());        // p is now invalid, accessing *p is a undefined behavior.

```



## Initializing a std::vector



A [`std::vector`](http://en.cppreference.com/w/cpp/container/vector) can be [initialized](http://en.cppreference.com/w/cpp/container/vector/vector) in several ways while declaring it:

> [`std:：vector`](http://en.cppreference.com/w/cpp/container/vector)可以[初始化](http://en.cppreference.com/w/cpp/container/vector/vector)在声明它时有几种方式：

```cpp
std::vector<int> v{ 1, 2, 3 };  // v becomes {1, 2, 3}

// Different from std::vector<int> v(3, 6)
std::vector<int> v{ 3, 6 };     // v becomes {3, 6}

```

```cpp
// Different from std::vector<int> v{3, 6} in C++11
std::vector<int> v(3, 6);  // v becomes {6, 6, 6}

std::vector<int> v(4);     // v becomes {0, 0, 0, 0}

```

A vector can be initialized from another container in several ways:


Copy construction (from another vector only), which copies data from `v2`:

> 复制构造（仅来自另一个矢量），它从“v2”复制数据：

```cpp
std::vector<int> v(v2);
std::vector<int> v = v2;

```

Move construction (from another vector only), which moves data from `v2`:

```cpp
std::vector<int> v(std::move(v2));
std::vector<int> v = std::move(v2);

```

Iterator (range) copy-construction, which copies elements into `v`:

```cpp
// from another vector
std::vector<int> v(v2.begin(), v2.begin() + 3); // v becomes {v2[0], v2[1], v2[2]}

// from an array
int z[] = { 1, 2, 3, 4 };
std::vector<int> v(z, z + 3);                   // v becomes {1, 2, 3}

// from a list
std::list<int> list1{ 1, 2, 3 };
std::vector<int> v(list1.begin(), list1.end()); // v becomes {1, 2, 3}

```


Iterator move-construction, using [`std::make_move_iterator`](http://en.cppreference.com/w/cpp/iterator/make_move_iterator), which moves elements into `v`:

> 迭代程序移动构造，使用[`std:：make_move_Iterator`](http://en.cppreference.com/w/cpp/iterator/make_move_iterator)，它将元素移动到“v”中：

```cpp
// from another vector
std::vector<int> v(std::make_move_iterator(v2.begin()),
                   std::make_move_iterator(v2.end());

// from a list
std::list<int> list1{ 1, 2, 3 };
std::vector<int> v(std::make_move_iterator(list1.begin()),
                   std::make_move_iterator(list1.end()));

```


With the help of the [`assign()`](http://en.cppreference.com/w/cpp/container/vector/assign) member function, a `std::vector` can be reinitialized after its construction:

> 在[`assign（）`]的帮助下(http://en.cppreference.com/w/cpp/container/vector/assign)成员函数，“std:：vector”可以在构造后重新初始化：

```cpp
v.assign(4, 100);                      // v becomes {100, 100, 100, 100}

v.assign(v2.begin(), v2.begin() + 3);  // v becomes {v2[0], v2[1], v2[2]}

int z[] = { 1, 2, 3, 4 };
v.assign(z + 1, z + 4);                // v becomes {2, 3, 4}

```



## Deleting Elements


### Deleting the last element:

```cpp
std::vector<int> v{ 1, 2, 3 };
v.pop_back();                           // v becomes {1, 2}

```

### Deleting all elements:

```cpp
std::vector<int> v{ 1, 2, 3 };
v.clear();                              // v becomes an empty vector

```

### Deleting element by index:

```cpp
std::vector<int> v{ 1, 2, 3, 4, 5, 6 };
v.erase(v.begin() + 3);                 // v becomes {1, 2, 3, 5, 6}

```


**Note:** For a `vector` deleting an element which is not the last element, all elements beyond the deleted element have to be copied or moved to fill the gap, see the note below and [std::list](http://stackoverflow.com/documentation/c%2B%2B/783/standard-library-containers/2688/stdlist#t=201607251402013099142).

> **注：**对于删除不是最后一个元素的“vector”元素，必须复制或移动删除元素之外的所有元素以填补空白，请参阅下面的注释和[std:：list](http://stackoverflow.com/documentation/c%2B%2B/783/standard-库容器/268/stdlist#t=2016072514402013099142）。

### Deleting all elements in a range:

```cpp
std::vector<int> v{ 1, 2, 3, 4, 5, 6 };
v.erase(v.begin() + 1, v.begin() + 5);  // v becomes {1, 6}

```


****Note:**** The above methods do not change the capacity of the vector, only the size. See [Vector Size and Capacity](http://stackoverflow.com/documentation/c%2B%2B/511/stdvector/3694/vector-size-and-capacity#t=201608311621108011517).

> ****注：***以上方法不改变矢量的容量，只改变矢量的大小。请参阅[矢量大小和容量](http://stackoverflow.com/documentation/c%2B%2B/511/stdvector/3694/vector-尺寸和容量#t=20160831121108011517）。


The [`erase`](http://en.cppreference.com/w/cpp/container/vector/erase) method, which removes a range of elements, is often used as a part of the [**erase-remove**](https://en.wikipedia.org/wiki/Erase%E2%80%93remove_idiom) idiom. That is, first [`std::remove`](http://en.cppreference.com/w/cpp/algorithm/remove) moves some elements to the end of the vector, and then `erase` chops them off. This is a relatively inefficient operation for any indices less than the last index of the vector because all elements after the erased segments must be relocated to new positions. For speed critical applications that require efficient removal of arbitrary elements in a container, see [std::list](http://stackoverflow.com/documentation/c%2B%2B/783/standard-library-containers/2688/stdlist#t=201607251402013099142).

> [`erase`](http://en.cppreference.com/w/cpp/container/vector/erase)方法删除一系列元素，通常用作[**erase-remove**]的一部分(https://en.wikipedia.org/wiki/Erase%E2%80%93remove_idiom)成语。也就是说，首先[`std:：remove`](http://en.cppreference.com/w/cpp/algorithm/remove)将一些元素移动到矢量的末尾，然后“擦除”将其截断。对于小于矢量最后一个索引的索引，这是一种相对低效的操作，因为擦除段之后的所有元素都必须重新定位到新的位置。有关需要有效删除容器中任意元素的速度关键型应用程序，请参阅[std:：list](http://stackoverflow.com/documentation/c%2B%2B/783/standard-库容器/268/stdlist#t=2016072514402013099142）。

### Deleting elements by value:

```cpp
std::vector<int> v{ 1, 1, 2, 2, 3, 3 };
int value_to_remove = 2;
v.erase(std::remove(v.begin(), v.end(), value_to_remove), v.end()); // v becomes {1, 1, 3, 3}

```

### Deleting elements by condition:

```cpp
// std::remove_if needs a function, that takes a vector element as argument and returns true, 
// if the element shall be removed
bool _predicate(const int& element) {
    return (element > 3); // This will cause all elements to be deleted that are larger than 3
}
...
std::vector<int> v{ 1, 2, 3, 4, 5, 6 };
v.erase(std::remove_if(v.begin(), v.end(), _predicate), v.end()); // v becomes {1, 2, 3}

```

### Deleting elements by lambda, without creating additional predicate function

```cpp
std::vector<int> v{ 1, 2, 3, 4, 5, 6 };
v.erase(std::remove_if(v.begin(), v.end(),
     [](auto& element){return element > 3;} ), v.end()
);

```

### Deleting elements by condition from a loop:

```cpp
std::vector<int> v{ 1, 2, 3, 4, 5, 6 };
std::vector<int>::iterator it = v.begin();
while (it != v.end()) {
    if (condition)
        it = v.erase(it); // after erasing, 'it' will be set to the next element in v
    else
        ++it;             // manually set 'it' to the next element in v
}

```


While it is important **not** to increment `it` in case of a deletion, you should consider using a different method when then erasing repeatedly in a loop. Consider `remove_if` for a more efficient way.

> 虽然在删除的情况下增加“it”很重要，但在循环中重复擦除时，应考虑使用不同的方法。考虑“remove_if”以获得更有效的方法。

### Deleting elements by condition from a reverse loop:

```cpp
std::vector<int> v{ -1, 0, 1, 2, 3, 4, 5, 6 };
typedef std::vector<int>::reverse_iterator rev_itr;
rev_itr it = v.rbegin();

while (it != v.rend()) { // after the loop only '0' will be in v
    int value = *it;
    if (value) {
        ++it;
        // See explanation below for the following line.
        it = rev_itr(v.erase(it.base()));
    } else
        ++it;
}

```

Note some points for the preceding loop:

<li>

Given a reverse iterator `it` pointing to some element, the method [`base`](http://en.cppreference.com/w/cpp/iterator/reverse_iterator/base) gives the regular (non-reverse) iterator pointing to the same element.

> 给定一个指向某个元素的反向迭代器“it”，方法[`base`](http://en.cppreference.com/w/cpp/iterator/reverse_iterator/base)给出指向同一元素的常规（非反向）迭代器。
</li>
<li>
`vector::erase(iterator)` erases the element pointed to by an iterator, and returns an iterator to the element that followed the given element.
</li>
<li>
`reverse_iterator::reverse_iterator(iterator)` constructs a reverse iterator from an iterator.
</li>


Put altogether, the line `it = rev_itr(v.erase(it.base()))` says: take the reverse iterator `it`, have `v` erase the element pointed by its regular iterator; take the resulting iterator, construct a reverse iterator from it, and assign it to the reverse iterator `it`.

> 总之，行“it=rev_itr（v.erase（it.base（）））”表示：取反向迭代器“it”，让“v”擦除其常规迭代器所指向的元素；获取生成的迭代器，从中构造一个反向迭代器并将其分配给反向迭代程序“it”。


Deleting all elements using `v.clear()` does not free up memory ([`capacity()`](http://en.cppreference.com/w/cpp/container/vector/capacity) of the vector remains unchanged). To reclaim space, use:

> 使用“v.clear（）”删除所有元素不会释放内存（[`capacity（）`](http://en.cppreference.com/w/cpp/container/vector/capacity)矢量的保持不变）。要回收空间，请使用：

```cpp
std::vector<int>().swap(v);

```


[`shrink_to_fit()`](http://en.cppreference.com/w/cpp/container/vector/shrink_to_fit) frees up unused vector capacity:

> [`shrink_to_fit（）`](http://en.cppreference.com/w/cpp/container/vector/shrink_to_fit)释放未使用的矢量容量：

```cpp
v.shrink_to_fit();

```


The `shrink_to_fit` does not guarantee to really reclaim space, but most current implementations do.

> “shrink_to_fit”不能保证真正回收空间，但大多数当前的实现都可以。



## Iterating Over std::vector



You can iterate over a [`std::vector`](http://en.cppreference.com/w/cpp/container/vector) in several ways. For each of the following sections, `v` is defined as follows:

> 您可以迭代[`std:：vector`](http://en.cppreference.com/w/cpp/container/vector)在几个方面。对于以下各节，“v”的定义如下：

```cpp
std::vector<int> v;

```

### Iterating in the Forward Direction

```cpp
// Range based for
for(const auto& value: v) {
    std::cout << value << "\n";
}

// Using a for loop with iterator
for(auto it = std::begin(v); it != std::end(v); ++it) {
    std::cout << *it << "\n";
}

// Using for_each algorithm, using a function or functor:
void fun(int const& value) {
    std::cout << value << "\n";
}

std::for_each(std::begin(v), std::end(v), fun);

// Using for_each algorithm. Using a lambda:
std::for_each(std::begin(v), std::end(v), [](int const& value) {
    std::cout << value << "\n";
});

```

```cpp
// Using a for loop with iterator
for(std::vector<int>::iterator it = std::begin(v); it != std::end(v); ++it) {
    std::cout << *it << "\n";
}

```

```cpp
// Using a for loop with index
for(std::size_t i = 0; i < v.size(); ++i) {
    std::cout << v[i] << "\n";
}

```

### Iterating in the Reverse Direction

```cpp
// There is no standard way to use range based for for this.
// See below for alternatives.

// Using for_each algorithm
// Note: Using a lambda for clarity. But a function or functor will work
std::for_each(std::rbegin(v), std::rend(v), [](auto const& value) {
    std::cout << value << "\n";
});

// Using a for loop with iterator
for(auto rit = std::rbegin(v); rit != std::rend(v); ++rit) {
    std::cout << *rit << "\n";
}

```

```cpp
// Using a for loop with index
for(std::size_t i = 0; i < v.size(); ++i) {
    std::cout << v[v.size() - 1 - i] << "\n";
}

```


Though there is no built-in way to use the range based for to reverse iterate; it is relatively simple to fix this. The range based for uses `begin()` and `end()` to get iterators and thus simulating this with a wrapper object can achieve the results we require.

> 尽管没有内置的方法来使用基于范围的for进行反向迭代；解决这个问题相对简单。基于范围的for使用“begin（）”和“end（）”来获取迭代器，因此使用包装器对象来模拟迭代器可以获得我们需要的结果。

```cpp
template<class C>
struct ReverseRange {
  C c; // could be a reference or a copy, if the original was a temporary
  ReverseRange(C&& cin): c(std::forward<C>(cin)) {}
  ReverseRange(ReverseRange&&)=default;
  ReverseRange& operator=(ReverseRange&&)=delete;
  auto begin() const {return std::rbegin(c);}
  auto end()   const {return std::rend(c);}
};
// C is meant to be deduced, and perfect forwarded into
template<class C>
ReverseRange<C> make_ReverseRange(C&& c) {return {std::forward<C>(c)};}

int main() {
    std::vector<int> v { 1,2,3,4};
    for(auto const& value: make_ReverseRange(v)) {
        std::cout << value << "\n";
    }
}

```

### Enforcing const elements


Since C++11 the `cbegin()` and `cend()` methods allow you to obtain a **constant iterator** for a vector, even if the vector is non-const.  A constant iterator allows you to read but not modify the contents of the vector which is useful to enforce const correctness:

> 由于C++11，“cbegin（）”和“cend（）”方法允许您获得向量的**常量迭代器**，即使该向量是非常量的。常量迭代器允许您读取但不修改向量的内容，这对强制常量正确性很有用：

```cpp
// forward iteration
for (auto pos = v.cbegin(); pos != v.cend(); ++pos) {
   // type of pos is vector<T>::const_iterator
   // *pos = 5; // Compile error - can't write via const iterator
}

// reverse iteration
for (auto pos = v.crbegin(); pos != v.crend(); ++pos) {
   // type of pos is vector<T>::const_iterator
   // *pos = 5; // Compile error - can't write via const iterator
}

// expects Functor::operand()(T&) 
for_each(v.begin(), v.end(), Functor());

// expects Functor::operand()(const T&)
for_each(v.cbegin(), v.cend(), Functor())

```


[as_const](http://en.cppreference.com/w/cpp/utility/as_const) extends this to range iteration:

> [as_const](http://en.cppreference.com/w/cpp/utility/as_const)将其扩展到范围迭代：

```cpp
for (auto const& e : std::as_const(v)) {
  std::cout << e << '\n';
}

```

This is easy to implement in earlier versions of C++:

```cpp
template <class T>
constexpr std::add_const_t<T>& as_const(T& t) noexcept {
  return t;
}

```

### A Note on Efficiency


Since the class `std::vector` is basically a class that manages a dynamically allocated contiguous array, the same principle explained [here](http://stackoverflow.com/documentation/c/322/arrays/8977/iterating-through-an-array-efficiently-and-row-major-order) applies to C++ vectors. Accessing the vector's content by index is much more efficient when following the row-major order principle. Of course, each access to the vector also puts its management content into the cache as well, but as has been debated many times (notably [here](http://stackoverflow.com/questions/381621/using-arrays-or-stdvectors-in-c-whats-the-performance-gap) and [here](http://stackoverflow.com/questions/3664272/is-stdvector-so-much-slower-than-plain-arrays)), the difference in performance for iterating over a `std::vector` compared to a raw array is negligible. So the same principle of efficiency for raw arrays in C also applies for C++'s `std::vector`.

> 由于类“std:：vector”基本上是一个管理动态分配的连续数组的类，因此[此处]解释了相同的原理(http://stackoverflow.com/documentation/c/322/arrays/8977/iterating-through-an-array-efficiently-and-row-major-order)适用于C++向量。当遵循行主序原则时，通过索引访问向量的内容要高效得多。当然，对向量的每次访问也会将其管理内容放入缓存，但正如人们多次争论的那样（尤其是[此处](http://stackoverflow.com/questions/381621/using-arrays-or-stdvectors-in-c-whats-the-performance-gap)和[这里](http://stackoverflow.com/questions/3664272/is-stdvector-so-much-slower-than-plain-arrays))，与原始数组相比，在“std:：vector”上迭代的性能差异可以忽略不计。因此，C中原始数组的效率原理也适用于C++的“std:：vector”。



## vector<bool>: The Exception To So Many, So Many Rules



The standard (section 23.3.7) specifies that a specialization of `vector<bool>` is provided, which optimizes space by packing the `bool` values, so that each takes up only one bit. Since bits aren't addressable in C++, this means that several requirements on `vector` are not placed on `vector<bool>`:

> 该标准（第23.3.7节）规定提供“vector＜bool＞”的特殊化，通过打包“bool”值来优化空间，使每个值只占一位。由于位在C++中是不可寻址的，这意味着对“vector”的几个要求没有放在“vector＜bool＞”上：


- The data stored is not required to be contiguous, so a `vector<bool>` can't be passed to a C API which expects a `bool` array.

> -存储的数据不需要是连续的，因此“vector＜bool＞”不能传递给需要“bool”数组的C API。

- `at()`, `operator []`, and dereferencing of iterators do not return a reference to `bool`. Rather they return a proxy object that (imperfectly) simulates a reference to a `bool` by overloading its assignment operators. As an example, the following code may not be valid for `std::vector<bool>`, because dereferencing an iterator does not return a reference:

> -迭代器的`at（）`、`operator[]`和取消引用不会返回对`bool`的引用。相反，它们返回一个代理对象，该对象（不完美地）通过重载其赋值运算符来模拟对“bool”的引用。例如，以下代码可能对“std:：vector＜bool＞”无效，因为取消引用迭代器不会返回引用：

```cpp
std::vector<bool> v = {true, false};
for (auto &b: v) { } // error

```


Similarly, functions expecting a `bool&` argument cannot be used with the result of `operator []` or `at()` applied to `vector<bool>`, or with the result of dereferencing its iterator:

> 类似地，期望“bool&”参数的函数不能与应用于“vector＜bool＞”的“operator[]”或“at（）”的结果一起使用，也不能与取消引用其迭代器的结果一起一起使用：

```

 void f(bool& b);
  f(v[0]);             // error
  f(*v.begin());       // error

```


The implementation of `std::vector<bool>` is dependent on both the compiler and architecture. The specialisation is implemented by packing `n` Booleans into the lowest addressable section of memory. Here, `n` is the size in bits of the lowest addressable memory. In most modern systems this is 1 byte or 8 bits. This means that one byte can store 8 Boolean values. This is an improvement over the traditional implementation where 1 Boolean value is stored in 1 byte of memory.

> “std:：vector＜bool＞”的实现依赖于编译器和体系结构。专门化是通过将“n”布尔值打包到内存的最低可寻址部分来实现的。这里，‘n’是最低可寻址存储器的以位为单位的大小。在大多数现代系统中，这是1字节或8位。这意味着一个字节可以存储8个布尔值。这是对传统实现的改进，在传统实现中，1个布尔值存储在1个字节的内存中。


****Note:**** The below example shows possible bitwise values of individual bytes in a traditional vs. optimized `vector<bool>`. This will not always hold true in all architectures. It is, however, a good way of visualising the optimization. In the below examples a byte is represented as [x, x, x, x, x, x, x, x].

> ****注意：***以下示例显示了传统与优化“vector＜bool＞”中单个字节的可能逐位值。这并不总是适用于所有体系结构。然而，这是一个可视化优化的好方法。在以下示例中，字节表示为[x，x，x、x、x，x。

****Traditional** `std::vector<char>` storing 8 Boolean values:**

```cpp
std::vector<char> trad_vect = {true, false, false, false, true, false, true, true};

```

**Bitwise representation:**

```cpp
[0,0,0,0,0,0,0,1], [0,0,0,0,0,0,0,0], [0,0,0,0,0,0,0,0], [0,0,0,0,0,0,0,0], 
[0,0,0,0,0,0,0,1], [0,0,0,0,0,0,0,0], [0,0,0,0,0,0,0,1], [0,0,0,0,0,0,0,1]

```

****Specialized** `std::vector<bool>` storing 8 Boolean values:**

```cpp
std::vector<bool> optimized_vect = {true, false, false, false, true, false, true, true};

```

**Bitwise representation:**

```cpp
[1,0,0,0,1,0,1,1]

```


Notice in the above example, that in the traditional version of `std::vector<bool>`, 8 Boolean values take up 8 bytes of memory, whereas in the optimized version of `std::vector<bool>`, they only use 1 byte of memory. This is a significant improvement on memory usage. If you need to pass a `vector<bool>` to an C-style API, you may need to copy the values to an array, or find a better way to use the API, if memory and performance are at risk.

> 请注意，在上面的示例中，在“std:：vector＜bool＞”的传统版本中，8个布尔值占用8字节的内存，而在“std：：vector＞bool＞’”的优化版本中，它们只使用1字节的内存。这是内存使用率的显著提高。如果需要将“vector＜bool＞”传递给C样式的API，则可能需要将值复制到数组中，或者在内存和性能面临风险的情况下找到更好的方法来使用API。



## Inserting Elements


Appending an element at the end of a vector (by copying/moving):

```cpp
struct Point {
  double x, y;
  Point(double x, double y) : x(x), y(y) {}
};
std::vector<Point> v;
Point p(10.0, 2.0);
v.push_back(p);  // p is copied into the vector.

```


Appending an element at the end of a vector by constructing the element in place:

> 通过在适当位置构造元素，将元素追加到向量的末尾：

```cpp
std::vector<Point> v;
v.emplace_back(10.0, 2.0); // The arguments are passed to the constructor of the
                           // given type (here Point). The object is constructed
                           // in the vector, avoiding a copy.

```


Note that `std::vector` does **not** have a `push_front()` member function due to performance reasons. Adding an element at the beginning causes all existing elements in the vector to be moved. If you want to frequently insert elements at the beginning of your container, then you might want to use `std::list` or `std::deque` instead.

> 请注意，由于性能原因，“std:：vector”**没有“push_front（）”成员函数。在开头添加元素会导致向量中所有现有元素都被移动。如果您想经常在容器的开头插入元素，那么您可能需要使用“std:：list”或“std：：deque”。

Inserting an element at any position of a vector:

```cpp
std::vector<int> v{ 1, 2, 3 };
v.insert(v.begin(), 9);          // v now contains {9, 1, 2, 3}

```


Inserting an element at any position of a vector by constructing the element in place:

> 通过在适当位置构造元素，在向量的任何位置插入元素：

```cpp
std::vector<int> v{ 1, 2, 3 };
v.emplace(v.begin()+1, 9);     // v now contains {1, 9, 2, 3}

```

Inserting another vector at any position of the vector:

```cpp
std::vector<int> v(4);      // contains: 0, 0, 0, 0
std::vector<int> v2(2, 10); // contains: 10, 10
v.insert(v.begin()+2, v2.begin(), v2.end()); // contains: 0, 0, 10, 10, 0, 0

```

Inserting an array at any position of a vector:

```cpp
std::vector<int> v(4); // contains: 0, 0, 0, 0
int a [] = {1, 2, 3}; // contains: 1, 2, 3
v.insert(v.begin()+1, a, a+sizeof(a)/sizeof(a[0])); // contains: 0, 1, 2, 3, 0, 0, 0

```


Use [`reserve()`](http://www.cplusplus.com/reference/vector/vector/reserve/) before inserting multiple elements if resulting vector size is known beforehand to avoid multiple reallocations (see [vector size and capacity](http://stackoverflow.com/documentation/c%2b%2b/511/stdvector/3694/vector-size-and-capacity#t=201607270933320554687)):

> 使用[`reserve（）`](http://www.cplusplus.com/reference/vector/vector/reserve/)在插入多个元素之前，如果结果向量大小事先已知，以避免多次重新定位（请参见[向量大小和容量](http://stackoverflow.com/documentation/c%2b%2b/511/stdvector/3694/vector-尺寸和容量#t=2016072709933320554687）：

```cpp
std::vector<int> v;
v.reserve(100);
for(int i = 0; i < 100; ++i)
    v.emplace_back(i);

```


Be sure to not make the mistake of calling [`resize()`](http://www.cplusplus.com/reference/vector/vector/resize/) in this case, or you will inadvertently create a vector with 200 elements where only the latter one hundred will have the value you intended.

> 请确保不要犯调用[`resize（）`]的错误(http://www.cplusplus.com/reference/vector/vector/resize/)在这种情况下，否则您将无意中创建一个包含200个元素的向量，其中只有后一百个元素具有您想要的值。



## Using std::vector as a C array



There are several ways to use a `std::vector` as a C array (for example, for compatibility with C libraries). This is possible because the elements in a vector are stored contiguously.

> 有几种方法可以将“std:：vector”用作C数组（例如，为了与C库兼容）。这是可能的，因为向量中的元素是连续存储的。

```cpp
std::vector<int> v{ 1, 2, 3 };
int* p = v.data();

```


In contrast to solutions based on previous C++ standards (see below), the member function `.data()` may also be applied to empty vectors, because it doesn't cause undefined behavior in this case.

> 与基于以前C++标准的解决方案（见下文）不同，成员函数`.data（）`也可以应用于空向量，因为在这种情况下它不会导致未定义的行为。


Before C++11, you would take the address of the vector's first element to get an equivalent pointer, if the vector isn't empty, these both methods are interchangeable:

> 在C++11之前，您可以使用向量的第一个元素的地址来获得等效指针，如果向量不为空，这两种方法是可互换的：

```cpp
int* p = &v[0];      // combine subscript operator and 0 literal

int* p = &v.front(); // explicitly reference the first element

```


****Note:**** If the vector is empty, `v[0]` and `v.front()` are undefined and cannot be used.

> ****注意：***如果向量为空，则“v[0]”和“v.front（）”是未定义的，不能使用。


When storing the base address of the vector's data, note that many operations (such as `push_back`, `resize`, etc.) can change the data memory location of the vector, thus [invalidating previous data pointers](http://stackoverflow.com/documentation/c%2B%2B/511/stdvector/1914/iterator-pointer-invalidation). For example:

> 在存储矢量数据的基地址时，请注意，许多操作（如“push_back”、“resize”等）都会更改矢量的数据存储位置，从而[使以前的数据指针无效](http://stackoverflow.com/documentation/c%2B%2B/511/stdvector/1914/iterator-指针无效）。例如：

```cpp
std::vector<int> v;
int* p = v.data();
v.resize(42);      // internal memory location changed; value of p is now invalid

```



## Finding an Element in std::vector



The function [`std::find`](http://en.cppreference.com/w/cpp/algorithm/find), defined in the [`<algorithm>`](http://en.cppreference.com/w/cpp/algorithm) header, can be used to find an element in a `std::vector`.

> 函数[`std:：find`](http://en.cppreference.com/w/cpp/algorithm/find)，在[`<algorithm>`]中定义(http://en.cppreference.com/w/cpp/algorithm)标头，可用于查找“std:：vector”中的元素。

`std::find` uses the `operator==` to compare elements for equality. It returns an iterator to the first element in the range that compares equal to the value.


If the element in question is not found, `std::find` returns `std::vector::end` (or `std::vector::cend` if the vector is `const`).

> 如果未找到有问题的元素，“std:：find”将返回“std：：vector:：end”（如果向量为“const”，则返回“std：：vector：：cend”）。

```cpp
static const int arr[] = {5, 4, 3, 2, 1};
std::vector<int> v (arr, arr + sizeof(arr) / sizeof(arr[0]) );

std::vector<int>::iterator it = std::find(v.begin(), v.end(), 4);
std::vector<int>::difference_type index = std::distance(v.begin(), it);
// `it` points to the second element of the vector, `index` is 1

std::vector<int>::iterator missing = std::find(v.begin(), v.end(), 10);
std::vector<int>::difference_type index_missing = std::distance(v.begin(), missing);
// `missing` is v.end(), `index_missing` is 5 (ie. size of the vector)

```

```cpp
std::vector<int> v { 5, 4, 3, 2, 1 };

auto it = std::find(v.begin(), v.end(), 4);
auto index = std::distance(v.begin(), it);
// `it` points to the second element of the vector, `index` is 1

auto missing = std::find(v.begin(), v.end(), 10);
auto index_missing = std::distance(v.begin(), missing);
// `missing` is v.end(), `index_missing` is 5 (ie. size of the vector)

```


If you need to perform many searches in a large vector, then you may want to consider sorting the vector first, before using the [`binary_search`](http://en.cppreference.com/w/cpp/algorithm/binary_search) algorithm.

> 如果您需要在一个大向量中执行多次搜索，那么在使用[`binary_search`]之前，您可能需要考虑先对向量进行排序(http://en.cppreference.com/w/cpp/algorithm/binary_search)算法。


To find the first element in a vector that satisfies a condition, `std::find_if` can be used. In addition to the two parameters given to `std::find`, `std::find_if` accepts a third argument which is a function object or function pointer to a predicate function. The predicate should accept an element from the container as an argument and return a value convertible to `bool`, without modifying the container:

> 要找到满足条件的向量中的第一个元素，可以使用“std:：find_if”。除了提供给“std:：find”的两个参数外，“std：：find_if”还接受第三个参数，该参数是函数对象或指向谓词函数的函数指针。谓词应接受容器中的元素作为参数，并返回可转换为“bool”的值，而不修改容器：

```cpp
bool isEven(int val) {
    return (val % 2 == 0); 
}

struct moreThan {
    moreThan(int limit) : _limit(limit) {}
    
    bool operator()(int val) {
        return val > _limit;
    }
    
    int _limit;
};

static const int arr[] = {1, 3, 7, 8};
std::vector<int> v (arr, arr + sizeof(arr) / sizeof(arr[0]) );
    
std::vector<int>::iterator it = std::find_if(v.begin(), v.end(), isEven);
// `it` points to 8, the first even element

std::vector<int>::iterator missing = std::find_if(v.begin(), v.end(), moreThan(10));
// `missing` is v.end(), as no element is greater than 10

```

```cpp
// find the first value that is even
std::vector<int> v = {1, 3, 7, 8};
auto it = std::find_if(v.begin(), v.end(), [](int val){return val % 2 == 0;});
// `it` points to 8, the first even element

auto missing = std::find_if(v.begin(), v.end(), [](int val){return val > 10;});
// `missing` is v.end(), as no element is greater than 10

```



## Concatenating Vectors



One `std::vector` can be append to another by using the member function [`insert()`](http://en.cppreference.com/w/cpp/container/vector/insert):

> 一个“std:：vector”可以通过使用成员函数[`insert（）`]附加到另一个上(http://en.cppreference.com/w/cpp/container/vector/insert)：

```cpp
std::vector<int> a = {0, 1, 2, 3, 4};
std::vector<int> b = {5, 6, 7, 8, 9};

a.insert(a.end(), b.begin(), b.end());

```


However, this solution fails if you try to append a vector to itself, because the standard specifies that iterators given to `insert()` must not be from the same range as the receiver object's elements.

> 但是，如果您试图将向量附加到其本身，则此解决方案将失败，因为标准指定给“insert（）”的迭代器不能与接收器对象的元素来自同一范围。


Instead of using the vector's member functions, the functions [`std::begin()`](http://en.cppreference.com/w/cpp/iterator/begin) and [`std::end()`](http://en.cppreference.com/w/cpp/iterator/end) can be used:

> 函数[`std:：begin（）`]不是使用向量的成员函数(http://en.cppreference.com/w/cpp/iterator/begin)和[`std:：end（）`](http://en.cppreference.com/w/cpp/iterator/end)可以使用：

```cpp
a.insert(std::end(a), std::begin(b), std::end(b));

```


This is a more general solution, for example, because `b` can also be an array. However, also this solution doesn't allow you to append a vector to itself.

> 例如，这是一个更通用的解决方案，因为“b”也可以是一个数组。然而，这个解决方案也不允许将向量附加到其本身。


If the order of the elements in the receiving vector doesn't matter, considering the number of elements in each vector can avoid unnecessary copy operations:

> 如果接收向量中元素的顺序无关紧要，则考虑每个向量中的元素数量可以避免不必要的复制操作：

```cpp
if (b.size() < a.size())
  a.insert(a.end(), b.begin(), b.end());
else
  b.insert(b.end(), a.begin(), a.end());

```



## Reducing the Capacity of a Vector



A `std::vector` automatically increases its capacity upon insertion as needed, but it never reduces its capacity after element removal.

> “std:：vector”在根据需要插入时会自动增加其容量，但在移除元素后不会减少其容量。

```cpp
// Initialize a vector with 100 elements
std::vector<int> v(100);

// The vector's capacity is always at least as large as its size
auto const old_capacity = v.capacity();
// old_capacity >= 100

// Remove half of the elements
v.erase(v.begin() + 50, v.end());  // Reduces the size from 100 to 50 (v.size() == 50),
                                   // but not the capacity (v.capacity() == old_capacity)

```


To reduce its capacity, we can copy the contents of a vector to a new temporary vector. The new vector will have the minimum capacity that is needed to store all elements of the original vector. If the size reduction of the original vector was significant, then the capacity reduction for the new vector is likely to be significant. We can then swap the original vector with the temporary one to retain its minimized capacity:

> 为了减少其容量，我们可以将向量的内容复制到一个新的临时向量。新矢量将具有存储原始矢量的所有元素所需的最小容量。如果原始矢量的大小减少是显著的，那么新矢量的容量减少可能是显著的。然后，我们可以将原始矢量与临时矢量交换，以保持其最小化的容量：

```cpp
std::vector<int>(v).swap(v);

```


In C++11 we can use the `shrink_to_fit()` member function for a similar effect:

> 在C++11中，我们可以使用“shrink_to_fit（）”成员函数来获得类似的效果：

```cpp
v.shrink_to_fit();

```


Note: The `shrink_to_fit()` member function is a request and doesn't guarantee to reduce capacity.

> 注意：“shrink_to_fit（）”成员函数是一个请求，不能保证减少容量。



## Using a Sorted Vector for Fast Element Lookup



The [`<algorithm>`](http://en.cppreference.com/w/cpp/header/algorithm) header provides a number of useful functions for working with sorted vectors.

> [`<算法>`](http://en.cppreference.com/w/cpp/header/algorithm)header提供了许多有用的函数，用于处理已排序的向量。


An important prerequisite for working with sorted vectors is that the stored values are comparable with `<`.

> 使用排序向量的一个重要先决条件是存储的值与“<”相当。


An unsorted vector can be sorted by using the function [`std::sort()`](http://en.cppreference.com/w/cpp/algorithm/sort):

> 可以使用函数[`std:：sort（）`]对未排序的向量进行排序(http://en.cppreference.com/w/cpp/algorithm/sort)：

```cpp
std::vector<int> v;
// add some code here to fill v with some elements
std::sort(v.begin(), v.end());

```


Sorted vectors allow efficient element lookup using the function [`std::lower_bound()`](http://en.cppreference.com/w/cpp/algorithm/lower_bound). Unlike [`std::find()`](http://en.cppreference.com/w/cpp/algorithm/find), this performs an efficient binary search on the vector. The downside is that it only gives valid results for sorted input ranges:

> 排序向量允许使用函数[`std:：lower_bound（）`]进行高效的元素查找(http://en.cppreference.com/w/cpp/algorithm/lower_bound)。与[`std:：find（）`]不同(http://en.cppreference.com/w/cpp/algorithm/find)，这对向量执行有效的二进制搜索。缺点是它只为排序的输入范围提供有效的结果：

```cpp
// search the vector for the first element with value 42
std::vector<int>::iterator it = std::lower_bound(v.begin(), v.end(), 42);
if (it != v.end() && *it == 42) {
    // we found the element!
}

```


****Note:**** If the requested value is not part of the vector, `std::lower_bound()` will return an iterator to the first element that is **greater** than the requested value. This behavior allows us to insert a new element at its right place in an already sorted vector:

> ****注意：***如果请求的值不是向量的一部分，“std:：lower_bound（）”将向比请求值**大**的第一个元素返回迭代器。这种行为允许我们在已经排序的向量中的正确位置插入新元素：

```cpp
int const new_element = 33;
v.insert(std::lower_bound(v.begin(), v.end(), new_element), new_element);

```


If you need to insert a lot of elements at once, it might be more efficient to call `push_back()` for all them first and then call `std::sort()` once all elements have been inserted. In this case, the increased cost of the sorting can pay off against the reduced cost of inserting new elements at the end of the vector and not in the middle.

> 如果您需要同时插入许多元素，那么首先对所有元素调用“push_back（）”，然后在插入所有元素后调用“std:：sort（）”可能会更有效。在这种情况下，增加的排序成本可以抵消在向量的末端而不是在中间插入新元素的降低成本。


If your vector contains multiple elements of the same value, `std::lower_bound()` will try to return an iterator to the first element of the searched value. However, if you need to insert a new element **after** the last element of the searched value, you should use the function [`std::upper_bound()`](http://en.cppreference.com/w/cpp/algorithm/upper_bound) as this will cause less shifting around of elements:

> 如果向量包含多个相同值的元素，“std:：lower_bound（）”将尝试向搜索值的第一个元素返回迭代器。但是，如果需要在搜索值的最后一个元素**之后插入一个新元素**，则应使用函数[`std:：upper_bound（）`](http://en.cppreference.com/w/cpp/algorithm/upper_bound)因为这将导致元件的较少移位：

```cpp
v.insert(std::upper_bound(v.begin(), v.end(), new_element), new_element);

```


If you need both the upper bound and the lower bound iterators, you can use the function [`std::equal_range()`](http://en.cppreference.com/w/cpp/algorithm/equal_range) to retrieve both of them efficiently with one call:

> 如果同时需要上界和下界迭代器，可以使用函数[`std:：equal_range（）`](http://en.cppreference.com/w/cpp/algorithm/equal_range)为了用一个调用有效地检索这两个：

```cpp
std::pair<std::vector<int>::iterator,
          std::vector<int>::iterator> rg = std::equal_range(v.begin(), v.end(), 42);
std::vector<int>::iterator lower_bound = rg.first;
std::vector<int>::iterator upper_bound = rg.second;

```


In order to test whether an element exists in a sorted vector (although not specific to vectors), you can use the function [`std::binary_search()`](http://en.cppreference.com/w/cpp/algorithm/binary_search):

> 为了测试排序向量中是否存在元素（尽管不是特定于向量），可以使用函数[`std:：binary_search（）`](http://en.cppreference.com/w/cpp/algorithm/binary_search)：

```cpp
bool exists = std::binary_search(v.begin(), v.end(), value_to_find);

```



## Matrices Using Vectors



Vectors can be used as a 2D matrix by defining them as a vector of vectors.

> 通过将矢量定义为矢量的矢量，可以将矢量用作2D矩阵。


A matrix with 3 rows and 4 columns with each cell initialised as 0 can be defined as:

> 具有3行4列且每个单元初始化为0的矩阵可以定义为：

```cpp
std::vector<std::vector<int> > matrix(3, std::vector<int>(4));

```


The syntax for initializing them using initialiser lists or otherwise are similar to that of a normal vector.

> 使用初始化器列表或其他方式初始化它们的语法与法线向量的语法相似。

```

 std::vector<std::vector<int>> matrix = { {0,1,2,3},
                                           {4,5,6,7}, 
                                           {8,9,10,11}
                                         };

```

Values in such a vector can be accessed similar to a 2D array

```cpp
int var = matrix[0][2];

```


Iterating over the entire matrix is similar to that of a normal vector but with an extra dimension.

> 在整个矩阵上迭代类似于法向量，但有一个额外的维度。

```cpp
for(int i = 0; i < 3; ++i)
{
    for(int j = 0; j < 4; ++j)
    {
        std::cout << matrix[i][j] << std::endl;
    }
}

```

```cpp
for(auto& row: matrix)
{
    for(auto& col : row)
    { 
        std::cout << col << std::endl;
    }
}

```


A vector of vectors is a convenient way to represent a matrix but it's not the most efficient: individual vectors are scattered around memory and the data structure isn't cache friendly.

> 向量的向量是表示矩阵的一种方便方法，但它不是最有效的：单个向量分散在内存中，数据结构对缓存不友好。


Also, in a proper matrix, the length of every row must be the same (this isn't the case for a vector of vectors). The additional flexibility can be a source of errors.

> 此外，在一个适当的矩阵中，每一行的长度必须相同（向量中的向量不是这样）。额外的灵活性可能是错误的来源。



## Iterator/Pointer Invalidation



Iterators and pointers pointing into an `std::vector` can become invalid, but only when performing certain operations. Using invalid iterators/pointers will result in undefined behavior.

> 指向“std:：vector”的迭代器和指针可能无效，但仅在执行某些操作时无效。使用无效的迭代器/指针将导致未定义的行为。

Operations which invalidate iterators/pointers include:

<li>

Any insertion operation which changes the `capacity` of the `vector` will invalidate **all** iterators/pointers:

> 任何改变“vector”的“capacity”的插入操作都将使**所有**迭代器/指针无效：

```cpp
vector<int> v(5); // Vector has a size of 5; capacity is unknown.
int *p1 = &v[0];
v.push_back(2);   // p1 may have been invalidated, since the capacity was unknown.

v.reserve(20);    // Capacity is now at least 20.
int *p2 = &v[0];
v.push_back(4);   // p2 is *not* invalidated, since the size of `v` is now 7.
v.insert(v.end(), 30, 9); // Inserts 30 elements at the end. The size exceeds the
                          // requested capacity of 20, so `p2` is (probably) invalidated.
int *p3 = &v[0];
v.reserve(v.capacity() + 20); // Capacity exceeded, thus `p3` is invalid.

```


</li>

```cpp
auto old_cap = v.capacity();
v.shrink_to_fit();
if(old_cap != v.capacity())
    // Iterators were invalidated.

```


<li>

Any insertion operation, which does not increase the capacity, will still invalidate iterators/pointers pointing to elements at the insertion position and past it. This includes the `end` iterator:

> 任何插入操作都不会增加容量，但仍然会使指向插入位置并经过插入位置的元素的迭代器/指针无效。这包括“end”迭代器：

```cpp
vector<int> v(5);
v.reserve(20);                 // Capacity is at least 20.
int *p1 = &v[0];
int *p2 = &v[3];
v.insert(v.begin() + 2, 5, 0); // `p2` is invalidated, but since the capacity
                               // did not change, `p1` remains valid.
int *p3 = &v[v.size() - 1];
v.push_back(10); // The capacity did not change, so `p3` and `p1` remain valid.

```


</li>
<li>

Any removal operation will invalidate iterators/pointers pointing to the removed elements and to any elements past the removed elements. This includes the `end` iterator:

> 任何删除操作都将使指向已删除元素和已删除元素之后的任何元素的迭代器/指针无效。这包括“end”迭代器：

```cpp
vector<int> v(10);
int *p1 = &v[0];
int *p2 = &v[5];
v.erase(v.begin() + 3, v.end()); // `p2` is invalid, but `p1` remains valid.

```


</li>
<li>
`operator=` (copy, move, or otherwise) and `clear()` will invalidate all iterators/pointers pointing into the vector.
</li>



## Vector size and capacity


**Vector size** is simply the number of elements in the vector:

<li>

Current vector **size** is queried by `size()` member function. Convenience `empty()` function returns `true` if size is 0:

> 当前矢量**size**由“size（）”成员函数查询。如果大小为0，则便利`empty（）`函数返回`true`：

```cpp
vector<int> v = { 1, 2, 3 }; // size is 3
const vector<int>::size_type size = v.size();
cout << size << endl; // prints 3
cout << boolalpha << v.empty() << endl; // prints false

```


</li>
<li>
Default constructed vector starts with a size of 0:

```cpp
vector<int> v; // size is 0
cout << v.size() << endl; // prints 0

```


</li>
<li>

Adding `N` elements to vector increases **size** by `N` (e.g. by `push_back()`, `insert()` or `resize()` functions).

> 向向量中添加“N”个元素会使**大小**增加“N”（例如通过“push_back（）”、“insert（）”或“resize（）”函数）。
</li>
<li>

Removing `N` elements from vector decreases **size** by `N` (e.g. by `pop_back()`, `erase()` or `clear()` functions).

> 从向量中删除“N”元素会将**大小**减少“N”（例如通过“pop_back（）”、“erase（）”或“clear（）”函数）。
</li>
<li>

Vector has an implementation-specific upper limit on its size, but you are likely to run out of RAM before reaching it:

> Vector的大小有一个特定于实现的上限，但在达到它之前，您可能会耗尽RAM：

```cpp
vector<int> v;
const vector<int>::size_type max_size = v.max_size();
cout << max_size << endl; // prints some large number
v.resize( max_size ); // probably won't work
v.push_back( 1 ); // definitely won't work

```


</li>

Common mistake: **size** is not necessarily (or even usually) `int`:

```cpp
// !!!bad!!!evil!!!
vector<int> v_bad( N, 1 ); // constructs large N size vector
for( int i = 0; i < v_bad.size(); ++i ) { // size is not supposed to be int!
    do_something( v_bad[i] );
}

```


**Vector capacity** differs from **size**. While **size** is simply how many elements the vector currently has, **capacity** is for how many elements it allocated/reserved memory for. That is useful, because too frequent (re)allocations of too large sizes can be expensive.

> **矢量容量**与**大小**不同。虽然**大小**只是矢量当前具有的元素数量，但**容量**是它为多少元素分配/保留内存。这是有用的，因为太频繁（重新）分配太大的规模可能会很昂贵。

<li>

Current vector **capacity** is queried by `capacity()` member function. **Capacity** is always greater or equal to **size**:

> 当前矢量**capacity**由`capacity（）`成员函数查询**容量**始终大于或等于**大小**：

```cpp
vector<int> v = { 1, 2, 3 }; // size is 3, capacity is >= 3
const vector<int>::size_type capacity = v.capacity();
cout << capacity << endl; // prints number >= 3

```


</li>
<li>

You can manually reserve capacity by `reserve( N )` function (it changes vector capacity to `N`):

> 您可以通过“reserve（N）”功能手动保留容量（它将矢量容量更改为“N”）：

```cpp
// !!!bad!!!evil!!!
vector<int> v_bad;
for( int i = 0; i < 10000; ++i ) {
    v_bad.push_back( i ); // possibly lot of reallocations
}

// good
vector<int> v_good;
v_good.reserve( 10000 ); // good! only one allocation
for( int i = 0; i < 10000; ++i ) {
    v_good.push_back( i ); // no allocations needed anymore
}

```


</li>
<li>

You can request for the excess capacity to be released by `shrink_to_fit()` (but the implementation doesn't have to obey you). This is useful to conserve used memory:

> 您可以通过“shrink_to_fit（）”请求释放多余的容量（但实现不必服从您）。这有助于节省已用内存：

```cpp
vector<int> v = { 1, 2, 3, 4, 5 }; // size is 5, assume capacity is 6
v.shrink_to_fit(); // capacity is 5 (or possibly still 6)
cout << boolalpha << v.capacity() == v.size() << endl; // prints likely true (but possibly false)

```


</li>


Vector partly manages capacity automatically, when you add elements it may decide to grow. Implementers like to use 2 or 1.5 for the grow factor (golden ratio would be the ideal value - but is impractical due to being rational number). On the other hand vector usually do not automatically shrink. For example:

> Vector部分自动管理容量，当您添加元素时，它可能会决定增长。实现者喜欢使用2或1.5作为增长因子（黄金比例是理想值，但由于是有理数，因此不切实际）。另一方面，矢量通常不会自动收缩。例如：

```cpp
vector<int> v; // capacity is possibly (but not guaranteed) to be 0
v.push_back( 1 ); // capacity is some starter value, likely 1
v.clear(); // size is 0 but capacity is still same as before!

v = { 1, 2, 3, 4 }; // size is 4, and lets assume capacity is 4.
v.push_back( 5 ); // capacity grows - let's assume it grows to 6 (1.5 factor)
v.push_back( 6 ); // no change in capacity
v.push_back( 7 ); // capacity grows - let's assume it grows to 9 (1.5 factor)
// and so on
v.pop_back(); v.pop_back(); v.pop_back(); v.pop_back(); // capacity stays the same

```



## Find max and min Element and Respective Index in a Vector



To find the largest or smallest element stored in a vector, you can use the methods [`std::max_element`](http://en.cppreference.com/w/cpp/algorithm/max_element) and [`std::min_element`](http://en.cppreference.com/w/cpp/algorithm/min_element), respectively. These methods are defined in [`<algorithm>`](http://en.cppreference.com/w/cpp/algorithm) header. If several elements are equivalent to the greatest (smallest) element, the methods return the iterator to the first such element. Return `v.end()` for empty vectors.

> 要查找存储在向量中的最大或最小元素，可以使用方法[`std:：max_element`](http://en.cppreference.com/w/cpp/algorithm/max_element)和[`std:：min_element`](http://en.cppreference.com/w/cpp/algorithm/min_element)分别。这些方法在[`<algorithm>`]中定义(http://en.cppreference.com/w/cpp/algorithm)收割台。如果几个元素等价于最大（最小）元素，则方法将迭代器返回到第一个这样的元素。对于空向量，返回“v.end（）”。

```cpp
std::vector<int> v = {5, 2, 8, 10, 9}; 
int maxElementIndex = std::max_element(v.begin(),v.end()) - v.begin();
int maxElement = *std::max_element(v.begin(), v.end());

int minElementIndex = std::min_element(v.begin(),v.end()) - v.begin();
int minElement = *std::min_element(v.begin(), v.end());

std::cout << "maxElementIndex:" << maxElementIndex << ", maxElement:" << maxElement << '\n';
std::cout << "minElementIndex:" << minElementIndex << ", minElement:" << minElement << '\n';

```

Output:

> 
<p>maxElementIndex:3, maxElement:10<br />
minElementIndex:1, minElement:2</p>



The minimum and maximum element in a vector can be retrieved at the same time by using the method [`std::minmax_element`](http://en.cppreference.com/w/cpp/minmax_element), which is also defined in [`<algorithm>`](http://en.cppreference.com/w/cpp/algorithm) header:

> 可以使用方法[`std:：minmax_element`]同时检索向量中的最小元素和最大元素(http://en.cppreference.com/w/cpp/minmax_element)，也在[`<algorithm>`]中定义(http://en.cppreference.com/w/cpp/algorithm)页眉：

```cpp
std::vector<int> v = {5, 2, 8, 10, 9}; 
auto minmax = std::minmax_element(v.begin(), v.end());

std::cout << "minimum element: " << *minmax.first << '\n';
std::cout << "maximum element: " << *minmax.second << '\n';

```

Output:

> 
<p>minimum element: 2<br />
maximum element: 10</p>




## Converting an array to std::vector



An array can easily be converted into a `std::vector` by using [`std::begin`](http://en.cppreference.com/w/cpp/iterator/begin) and [`std::end`](http://en.cppreference.com/w/cpp/iterator/end):

> 使用[`std:：begin`]可以很容易地将数组转换为`std::vector`(http://en.cppreference.com/w/cpp/iterator/begin)和[`std:：end`](http://en.cppreference.com/w/cpp/iterator/end)：

```cpp
int values[5] = { 1, 2, 3, 4, 5 }; // source array

std::vector<int> v(std::begin(values), std::end(values)); // copy array to new vector

for(auto &x: v)
    std::cout << x << " ";
std::cout << std::endl;

```

> 
1 2 3 4 5


```cpp
int main(int argc, char* argv[]) {
    // convert main arguments into a vector of strings.
    std::vector<std::string>  args(argv, argv + argc);
}

```


A C++11 initializer_list<> can also be used to initialize the vector at once

> C++11 initializer_list＜＞也可以用于一次初始化矢量

```cpp
initializer_list<int> arr = { 1,2,3,4,5 };
vector<int> vec1 {arr};

for (auto & i : vec1)
    cout << i << endl;

```



## Functions Returning Large Vectors



In C++11, compilers are required to implicitly move from a local variable that is being returned. Moreover, most compilers can perform [copy elision](http://stackoverflow.com/documentation/c%2B%2B/2489/copy-elision) in many cases and elide the move altogether. As a result of this, returning large objects that can be moved cheaply no longer requires special handling:

> 在C++11中，编译器需要隐式地从正在返回的局部变量中移出。此外，大多数编译器都可以执行[复制省略](http://stackoverflow.com/documentation/c%2B%2B/2489/copy-elision），并完全取消该动作。因此，返回可以廉价移动的大型对象不再需要特殊处理：

```cpp
#include <vector>
#include <iostream>

// If the compiler is unable to perform named return value optimization (NRVO)
// and elide the move altogether, it is required to move from v into the return value.
std::vector<int> fillVector(int a, int b) {
    std::vector<int> v;
    v.reserve(b-a+1);
    for (int i = a; i <= b; i++) {
        v.push_back(i);
    }
    return v; // implicit move
}

int main() { // declare and fill vector
    std::vector<int> vec = fillVector(1, 10);

    // print vector
    for (auto value : vec)
        std::cout << value << " "; // this will print "1 2 3 4 5 6 7 8 9 10 "

    std::cout << std::endl;

    return 0;
}

```


Before C++11, copy elision was already allowed and implemented by most compilers. However, due to the absence of move semantics, in legacy code or code that has to be compiled with older compiler versions which don't implement this optimization, you can find vectors being passed as output arguments to prevent the unneeded copy:

> 在C++11之前，大多数编译器已经允许并实现了复制省略。然而，由于缺乏移动语义，在遗留代码或必须使用未实现此优化的旧编译器版本编译的代码中，您可以找到作为输出参数传递的向量，以防止不必要的副本：

```cpp
#include <vector>
#include <iostream>

// passing a std::vector by reference
void fillVectorFrom_By_Ref(int a, int b, std::vector<int> &v) {
    assert(v.empty());
    v.reserve(b-a+1);
    for (int i = a; i <= b; i++) {
        v.push_back(i);
    }
}

int main() {// declare vector
    std::vector<int> vec;
    
    // fill vector
    fillVectorFrom_By_Ref(1, 10, vec);
    // print vector
    for (std::vector<int>::const_iterator it = vec.begin(); it != vec.end(); ++it)
        std::cout << *it << " "; // this will print "1 2 3 4 5 6 7 8 9 10 "
    std::cout << std::endl;
    return 0;
}

```



#### Remarks



Use of an `std::vector` requires the inclusion of the `<vector>` header using `#include <vector>`.

> 使用“std:：vector”需要使用“#include＜vector＞”包含“＜vector>”标头。


Elements in a `std::vector` are stored contiguously on the free store. It should be noted that when vectors are nested as in `std::vector<std::vector<int> >`, the elements of each vector are contiguous, but each vector allocates its own underlying buffer on the free store.

> “std:：vector”中的元素连续存储在空闲存储中。需要注意的是，当向量嵌套为“std:：vector＜std:：vector＜int＞”时，每个向量的元素是连续的，但每个向量在空闲存储上分配自己的底层缓冲区。

