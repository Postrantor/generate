---
tip: translate by baidu@2023-10-25 08:26:14
---
---
metaTitle: "C++ | Iterators"
description: "Overview, Vector Iterator, Map Iterator, Reverse Iterators, Stream Iterators, C Iterators (Pointers), Write your own generator-backed iterator"
---

# Iterators



## Overview


### Iterators are Positions


Iterators are a means of navigating and operating on a sequence of elements and are a generalized extension of pointers. Conceptually it is important to remember that iterators are positions, not elements. For example, take the following sequence:

> 迭代程序是对元素序列进行导航和操作的一种方法，是指针的广义扩展。从概念上讲，重要的是要记住迭代器是位置，而不是元素。例如，采用以下顺序：

```cpp
A B C

```

The sequence contains three elements and four positions

```cpp
+---+---+---+---+
| A | B | C |   |
+---+---+---+---+

```


Elements are things within a sequence. Positions are places where meaningful operations can happen to the sequence. For example, one inserts into a position, **before** or **after** element A, not into an element. Even deletion of an element (`erase(A)`) is done by first finding its position, then deleting it.

> 元素是序列中的事物。位置是可以对序列进行有意义的操作的位置。例如，插入到一个位置，**在**之前或**在**元素a之后，而不是插入到元素中。即使删除一个元素（“擦除（A）”），也要先找到它的位置，然后删除它。

### From Iterators to Values

To convert from a position to a value, an iterator is **dereferenced**:

```cpp
auto my_iterator = my_vector.begin(); // position
auto my_value = *my_iterator; // value

```


One can think of an iterator as dereferencing to the value it refers to in the sequence. This is especially useful in understanding why you should never dereference the `end()` iterator in a sequence:

> 可以将迭代器看作是对它在序列中引用的值的取消引用。这对于理解为什么永远不应该取消引用序列中的“end（）”迭代器特别有用：

```cpp
+---+---+---+---+
| A | B | C |   |
+---+---+---+---+
  ↑           ↑
  |           +-- An iterator here has no value. Do not dereference it!
  +-------------- An iterator here dereferences to the value A.

```


In all the sequences and containers found in the C++ standard library, `begin()` will return an iterator to the first position, and `end()` will return an iterator to one past the last position (**not** the last position!). Consequently, the names of these iterators in algorithms are oftentimes labelled `first` and `last`:

> 在C++标准库中找到的所有序列和容器中，“begin（）”将把迭代器返回到第一个位置，而“end（）”则将迭代器返回到最后一个位置（**而不是**最后一个！）。因此，算法中这些迭代器的名称通常被标记为“first”和“last”：

```cpp
+---+---+---+---+
| A | B | C |   |
+---+---+---+---+
  ↑           ↑
  |           |
  +- first    +- last

```


It is also possible to obtain an iterator to **any sequence**, because even an empty sequence contains at least one position:

> 也可以获得**任何序列**的迭代器，因为即使是空序列也至少包含一个位置：

```cpp
+---+
|   |
+---+

```


In an empty sequence, `begin()` and `end()` will be the same position, and **neither** can be dereferenced:

> 在一个空序列中，“begin（）”和“end（）”将位于同一位置，**这两个**都不能被取消引用：

```cpp
+---+
|   |
+---+
  ↑
  |
  +- empty_sequence.begin()
  |
  +- empty_sequence.end()

```


The alternative visualization of iterators is that they mark the positions **between** elements:

> 迭代器的另一种可视化方式是，它们标记**元素之间的位置**：

```cpp
+---+---+---+
| A | B | C |
+---+---+---+
↑   ^   ^   ↑
|           |
+- first    +- last

```


and dereferencing an iterator returns a reference to the element coming after the iterator. Some situations where this view is particularly useful are:

> 并且解引用迭代器返回对迭代器之后的元素的引用。这种观点特别有用的一些情况是：

- `insert` operations will insert elements into the position indicated by the iterator,

- `erase` operations will return an iterator corresponding to the same position as the one passed in,

> -“擦除”操作将返回一个迭代器，该迭代器对应于与传入的位置相同的位置，

- an iterator and its corresponding [reverse iterator](https://stackoverflow.com/documentation/c%2B%2B/473/iterators/5101/reverse-iterators#t=201610021532190324666) are located in the same .position between elements

> -迭代器及其对应的[reverse迭代器](https://stackoverflow.com/documentation/c%2B%2B/473/iterators/5101/reverse-迭代器#t=201610021532190324666）位于元素之间的相同位置

### Invalid Iterators


An iterator becomes **invalidated** if (say, in the course of an operation) its position is no longer a part of a sequence. An invalidated iterator cannot be dereferenced until it has been reassigned to a valid position. For example:

> 如果（比如在操作过程中）迭代器的位置不再是序列的一部分，则迭代器将变为**无效**。无效的迭代器在重新分配到有效位置之前无法取消引用。例如：

```cpp
std::vector<int>::iterator first;
{
    std::vector<int> foo;
    first = foo.begin(); // first is now valid
} // foo falls out of scope and is destroyed
// At this point first is now invalid

```


The many algorithms and sequence member functions in the C++ standard library have rules governing when iterators are invalidated. Each algorithm is different in the way they treat (and invalidate) iterators.

> C++标准库中的许多算法和序列成员函数都有规则来管理迭代器何时无效。每种算法处理（和使）迭代器无效的方式不同。

### Navigating with Iterators


As we know, iterators are for navigating sequences. In order to do that an iterator must migrate its position throughout the sequence. Iterators can advance forward in the sequence and some can advance backwards:

> 正如我们所知，迭代器用于导航序列。为了做到这一点，迭代器必须在整个序列中迁移其位置。迭代程序可以在序列中向前推进，有些可以向后推进：

```cpp
auto first = my_vector.begin();
++first;                                             // advance the iterator 1 position
std::advance(first, 1);                              // advance the iterator 1 position
first = std::next(first);                            // returns iterator to the next element
std::advance(first, -1);                             // advance the iterator 1 position backwards
first = std::next(first, 20);                        // returns iterator to the element 20 position forward
first = std::prev(first, 5);                         // returns iterator to the element 5 position backward
auto dist = std::distance(my_vector.begin(), first); // returns distance between two iterators.

```


Note, second argument of std::distance should be reachable from the first one(or, in other words `first` should be less or equal than `second`).

> 请注意，std:：distance的第二个自变量应该从第一个自变量到达（或者，换句话说，“first”应该小于或等于“second”）。


Even though you can perform arithmetic operators with iterators, not all operations are defined for all types of iterators. `a = b + 3;` would work for Random Access Iterators, but wouldn't work for Forward or Bidirectional Iterators, which still can be advanced by 3 position with something like `b = a; ++b; ++b; ++b;`. So it is recommended to use special functions in case you are not sure what is iterator type (for example, in a template function accepting iterator).

> 尽管您可以使用迭代器执行算术运算符，但并非所有操作都是为所有类型的迭代器定义的`a=b+3；`适用于随机访问迭代程序，但不适用于正向或双向迭代程序，后者仍然可以通过类似“b=a++b++b++b、 ”。因此，如果您不确定迭代器类型是什么，建议使用特殊函数（例如，在接受迭代器的模板函数中）。

### Iterator Concepts


The C++ standard describes several different iterator concepts. These are grouped according to how they behave in the sequences they refer to. If you know the concept an iterator **models** (behaves like), you can be assured of the behavior of that iterator **regardless of the sequence to which it belongs**. They are often described in order from the most to least restrictive (because the next iterator concept is a step better than its predecessor):

> C++标准描述了几种不同的迭代器概念。这些是根据它们在引用的序列中的行为进行分组的。如果你知道迭代器**模型**（行为类似）的概念，那么无论它属于哪个序列，你都可以确信迭代器的行为**。它们通常按照从限制性最大到限制性最小的顺序进行描述（因为下一个迭代器概念比其前一个概念好一步）：


- Input Iterators                    : Can be dereferenced **only once** per position. Can only advance, and only one position at a time.

> -输入迭代器：每个位置只能取消引用**一次。只能前进，一次只能前进一个位置。

- Forward Iterators                  : An input iterator that can be dereferenced any number of times.

> -Forward迭代器：一个输入迭代器，可以被多次取消引用。

- Bidirectional Iterators            : A forward iterator that can also advance **backwards** one position at a time.

> -双向迭代器：一个前向迭代器，也可以一次向前**向后**一个位置。

- Random Access Iterators            : A bidirectional iterator that can advance forwards or backwards any number of positions at a time.

> -随机访问迭代器：一个双向迭代器，可以一次向前或向后推进任意数量的位置。

- Contiguous Iterators (since C++17) : A random access iterator that guaranties that underlying data is contiguous in memory.

> -连续迭代器（从C++17开始）：一个随机访问迭代器，它保证底层数据在内存中是连续的。


Algorithms can vary depending on the concept modeled by the iterators they are given. For example, although `random_shuffle` can be implemented for forward iterators, a more efficient variant that requires random access iterators could be provided.

> 算法可能会根据迭代器所建模的概念而有所不同。例如，尽管“random_shuffle”可以用于前向迭代器，但可以提供一种需要随机访问迭代器的更有效的变体。

### Iterator traits


Iterator traits provide uniform interface to the properties of iterators. They allow you to retrieve value, difference, pointer, reference types and also category of iterator:

> 迭代器特性为迭代器的属性提供了统一的接口。它们允许您检索值、差异、指针、引用类型以及迭代器的类别：

```cpp
template<class Iter>
Iter find(Iter first, Iter last, typename std::iterator_traits<Iter>::value_type val)  {
    while (first != last) {
        if (*first == val)
            return first;
        ++first;
    }
    return last;
}

```

Category of iterator can be used to specialize algorithms:

```cpp
template<class BidirIt>
void test(BidirIt a, std::bidirectional_iterator_tag)  {
    std::cout << "Bidirectional iterator is used" << std::endl;
}
 
template<class ForwIt>
void test(ForwIt a, std::forward_iterator_tag)  {
    std::cout << "Forward iterator is used" << std::endl;
}
 
template<class Iter>
void test(Iter a)  {
    test(a, typename std::iterator_traits<Iter>::iterator_category());
}

```


Categories of iterators are basically iterators concepts, except Contiguous Iterators don't have their own tag, since it was found to break code.

> 迭代器的类别基本上是迭代器概念，除了连续迭代器没有自己的标记，因为它被发现会破坏代码。



## Vector Iterator


`begin` returns an `iterator` to the first element in the sequence container.

`end` returns an `iterator` to the first element past the end.


If the vector object is `const`, both `begin` and `end` return a `const_iterator`. If you want a `const_iterator` to be returned even if your vector is not `const`, you can use `cbegin` and `cend`.

> 如果矢量对象是“const”，则“begin”和“end”都返回“const_iterator”。如果希望返回“const_iterator”，即使向量不是“const”，也可以使用“cbegin”和“cend”。

Example:

```cpp
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v = { 1, 2, 3, 4, 5 };  //intialize vector using an initializer_list

    for (std::vector<int>::iterator it = v.begin(); it != v.end(); ++it) {
        std::cout << *it << " ";
    }

    return 0;
}

```

Output:

> 
1 2 3 4 5




## Map Iterator


An iterator to the first element in the container.


If a map object is const-qualified, the function returns a `const_iterator`. Otherwise, it returns an `iterator`.

> 如果映射对象是const限定的，则函数将返回一个“const_iterator”。否则，它将返回一个“迭代器”。

```cpp
// Create a map and insert some values
std::map<char,int> mymap;
mymap['b'] = 100;
mymap['a'] = 200;
mymap['c'] = 300;

// Iterate over all tuples
for (std::map<char,int>::iterator it = mymap.begin(); it != mymap.end(); ++it)
    std::cout << it->first << " => " << it->second << '\n';

```

Output:

> 
<p>a => 200<br />
b => 100<br />
c => 300</p>




## Reverse Iterators



If we want to iterate backwards through a list or vector we can use a `reverse_iterator`. A reverse iterator is made from a bidirectional, or random access iterator which it keeps as a member which can be accessed through `base()`.

> 如果我们想在列表或向量中向后迭代，我们可以使用“reverse_iterator”。反向迭代器是由双向或随机访问迭代器组成的，它将其作为成员保留，可以通过“base（）”访问。


To iterate backwards use `rbegin()` and `rend()` as the iterators for the end of the collection, and the start of the collection respectively.

> 要向后迭代，请分别使用“rbegin（）”和“rend（）”作为集合末尾和集合开头的迭代器。

For instance, to iterate backwards use:

```cpp
std::vector<int> v{1, 2, 3, 4, 5};
for (std::vector<int>::reverse_iterator it = v.rbegin(); it != v.rend(); ++it)
{
    cout << *it;
} // prints 54321

```


A reverse iterator can be converted to a forward iterator via the `base()` member function. The relationship is that the reverse iterator references one element past the `base()` iterator:

> 反向迭代器可以通过“base（）”成员函数转换为正向迭代器。这种关系是，反向迭代器引用的一个元素超过了“base（）”迭代器：

```cpp
std::vector<int>::reverse_iterator r = v.rbegin();
std::vector<int>::iterator i = r.base();
assert(&*r == &*(i-1)); // always true if r, (i-1) are dereferenceable
                        // and are not proxy iterators

 +---+---+---+---+---+---+---+
 |   | 1 | 2 | 3 | 4 | 5 |   |
 +---+---+---+---+---+---+---+
   ↑   ↑               ↑   ↑
   |   |               |   |
rend() |         rbegin()  end()
       |                   rbegin().base()
     begin()
     rend().base()

```


In the visualization where iterators mark positions between elements, the relationship is simpler:

> 在迭代器标记元素之间位置的可视化中，关系更简单：

```

 +---+---+---+---+---+
  | 1 | 2 | 3 | 4 | 5 |
  +---+---+---+---+---+
  ↑                   ↑
  |                   |
  |                 end()
  |                 rbegin()
begin()             rbegin().base()
rend()
rend().base()

```



## Stream Iterators



Stream iterators are useful when we need to read a sequence or print formatted data from a container:

> 当我们需要从容器中读取序列或打印格式化数据时，流迭代器非常有用：

```cpp
// Data stream. Any number of various whitespace characters will be OK.
std::istringstream istr("1\t 2     3 4");
std::vector<int> v;

// Constructing stream iterators and copying data from stream into vector.
std::copy(
    // Iterator which will read stream data as integers.
    std::istream_iterator<int>(istr),
    // Default constructor produces end-of-stream iterator.
    std::istream_iterator<int>(),
    std::back_inserter(v));

// Print vector contents.
std::copy(v.begin(), v.end(),
    //Will print values to standard output as integers delimeted by " -- ".
    std::ostream_iterator<int>(std::cout, " -- "));

```

The example program will print `1 -- 2 -- 3 -- 4 --` to standard output.



## C Iterators (Pointers)


```cpp
// This creates an array with 5 values.
const int array[] = { 1, 2, 3, 4, 5 };

#ifdef BEFORE_CPP11

// You can use `sizeof` to determine how many elements are in an array.
const int* first = array;
const int* afterLast = first + sizeof(array) / sizeof(array[0]);

// Then you can iterate over the array by incrementing a pointer until
// it reaches past the end of our array.
for (const int* i = first; i < afterLast; ++i) {
    std::cout << *i << std::endl;
}

#else

// With C++11, you can let the STL compute the start and end iterators:
for (auto i = std::begin(array); i != std::end(array); ++i) {
    std::cout << *i << std::endl;
}

#endif

```


This code would output the numbers 1 through 5, one on each line like this:

> 该代码将输出数字1到5，每行一个，如下所示：

> 
<p>1<br />
2<br />
3<br />
4<br />
5</p>


### Breaking It Down

```cpp
const int array[] = { 1, 2, 3, 4, 5 };

```


This line creates a new integer array with 5 values. C arrays are just pointers to memory where each value is stored together in a contiguous block.

> 这一行创建一个具有5个值的新整数数组。C数组只是指向内存的指针，其中每个值都存储在一个连续的块中。

```cpp
const int* first = array;
const int* afterLast = first + sizeof(array) / sizeof(array[0]);

```


These lines create two pointers. The first pointer is given the value of the array pointer, which is the address of the first element in the array. The `sizeof` operator when used on a C array returns the size of the array in bytes. Divided by the size of an element this gives the number of elements in the array. We can use this to find the address of the block **after** the array.

> 这些行创建两个指针。第一个指针被赋予数组指针的值，该值是数组中第一个元素的地址。在C数组上使用“sizeof”运算符时，会返回以字节为单位的数组大小。除以一个元素的大小，得出数组中元素的数量。我们可以使用它来查找数组**之后的块**的地址。

```cpp
for (const int* i = first; i < afterLast; ++i) {

```


Here we create a pointer which we will use as an iterator. It is initialized with the address of the first element we want to iterate over, and we'll continue to iterate as long as `i` is less than `afterLast`, which means as long as `i` is pointing to an address within `array`.

> 在这里，我们创建一个指针，将其用作迭代器。它是用我们要迭代的第一个元素的地址初始化的，只要“i”小于“afterLast”，我们就会继续迭代，也就是说，只要“i”指向“array”中的地址。

```

   std::cout << *i << std::endl;

```


Finally, within the loop we can access the value our iterator `i` is pointing to by dereferencing it. Here the dereference operator `*` returns the value at the address in `i`.

> 最后，在循环中，我们可以通过取消引用迭代器“i”所指向的值来访问它。这里的取消引用运算符“*”返回“i”中地址处的值。



## Write your own generator-backed iterator



A common pattern in other languages is having a function that produces a "stream" of objects, and being able to use loop-code to loop over it.

> 其他语言中的一个常见模式是具有一个生成对象“流”的函数，并且能够使用循环代码对其进行循环。

We can model this in C++ as

```cpp
template<class T>
struct generator_iterator {
  using difference_type=std::ptrdiff_t;
  using value_type=T;
  using pointer=T*;
  using reference=T;
  using iterator_category=std::input_iterator_tag;
  std::optional<T> state;
  std::function< std::optional<T>() > operation;
  // we store the current element in "state" if we have one:
  T operator*() const {
    return *state;
  }
  // to advance, we invoke our operation.  If it returns a nullopt
  // we have reached the end:
  generator_iterator& operator++() {
    state = operation();
    return *this;        
  }
  generator_iterator operator++(int) {
    auto r = *this;
    ++(*this);
    return r;
  }
  // generator iterators are only equal if they are both in the "end" state:
  friend bool operator==( generator_iterator const& lhs, generator_iterator const& rhs ) {
    if (!lhs.state && !rhs.state) return true;
    return false;
  }
  friend bool operator!=( generator_iterator const& lhs, generator_iterator const& rhs ) {
    return !(lhs==rhs);
  }
  // We implicitly construct from a std::function with the right signature:
  generator_iterator( std::function< std::optional<T>() > f ):operation(std::move(f))
  {
    if (operation)
      state = operation();
  }
  // default all special member functions:
  generator_iterator( generator_iterator && ) =default;
  generator_iterator( generator_iterator const& ) =default;
  generator_iterator& operator=( generator_iterator && ) =default;
  generator_iterator& operator=( generator_iterator const& ) =default;
  generator_iterator() =default;
};

```

[live example](http://coliru.stacked-crooked.com/a/cbb93f9ff193cdba).


We store the generated element early so we can more easily detect if we are already at the end.

> 我们提前存储生成的元素，这样我们就可以更容易地检测到我们是否已经处于末尾。


As the function of an end generator iterator is never used, we can create a range of generator iterators by only copying the `std::function` once.  A default constructed generator iterator compares equal to itself, and to all other end-generator-iterators.

> 由于从未使用过端生成器迭代器的函数，因此我们只需复制一次“std:：function”就可以创建一系列生成器迭代程序。默认构造的生成器迭代器与自身以及所有其他端生成器迭代程序进行比较。

