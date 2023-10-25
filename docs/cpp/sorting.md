---
tip: translate by baidu@2023-10-25 08:31:41
---
---
metaTitle: "C++ | Sorting"
description: "Sorting and sequence containers, Sorting sequence containers with specifed ordering, Sorting sequence containers by overloaded less operator, Sorting sequence containers using compare function, Sorting sequence containers using lambda expressions (C++11), sorting with std::map (ascending and descending), Sorting built-in arrays"
---

# Sorting



## Sorting and sequence containers


`std::sort`, found in the standard library header `algorithm`, is a standard library algorithm for sorting a range of values, defined by a pair of iterators. `std::sort` takes as the last parameter a functor used to compare two values; this is how it determines the order. Note that `std::sort` is not [stable](https://en.wikipedia.org/wiki/Sorting_algorithm#Stability).


The comparison function **must** impose a [Strict, Weak Ordering](https://en.wikipedia.org/wiki/Weak_ordering#Strict_weak_orderings) on the elements. A simple less-than (or greater-than) comparison will suffice.

> 比较函数**必须**强制执行[严格，弱排序](https://en.wikipedia.org/wiki/Weak_ordering#Strict_weak_orderings)关于元素。一个简单的小于（或大于）比较就足够了。


A container with random-access iterators can be sorted using the `std::sort` algorithm:

> 具有随机访问迭代器的容器可以使用“std:：sort”算法进行排序：

```cpp
#include <vector>
#include <algorithm>

std::vector<int> MyVector = {3, 1, 2}

//Default comparison of <
std::sort(MyVector.begin(), MyVector.end());

```

`std::sort` requires that its iterators are random access iterators. The sequence containers `std::list` and `std::forward_list` (requiring C++11) do not provide random access iterators, so they cannot be used with `std::sort`. However, they do have `sort` member functions which implement a sorting algorithm that works with their own iterator types.

```cpp
#include <list>
#include <algorithm>

std::list<int> MyList = {3, 1, 2}

//Default comparison of <
//Whole list only.
MyList.sort();

```


Their member `sort` functions always sort the entire list, so they cannot sort a sub-range of elements. However, since `list` and `forward_list` have fast splicing operations, you could extract the elements to be sorted from the list, sort them, then stuff them back where they were quite efficiently like this:

> 它们的成员“排序”函数总是对整个列表进行排序，因此不能对元素的子范围进行排序。然而，由于“list”和“forward_list”具有快速拼接操作，您可以从列表中提取要排序的元素，对它们进行排序，然后将它们填充回原来的位置，非常有效，如下所示：

```cpp
void sort_sublist(std::list<int>& mylist, std::list<int>::const_iterator start, std::list<int>::const_iterator end) {
    //extract and sort half-open sub range denoted by start and end iterator 
    std::list<int> tmp;
    tmp.splice(tmp.begin(), list, start, end);
    tmp.sort();
    //re-insert range at the point we extracted it from
    list.splice(end, tmp);
}

```



## Sorting sequence containers with specifed ordering



If the values in a container have certain operators already overloaded, `std::sort` can be used with specialized functors to sort in either ascending or descending order:

> 如果容器中的值已经重载了某些运算符，则“std:：sort”可以与专用的函子一起使用，以升序或降序排序：

```cpp
#include <vector>
#include <algorithm>
#include <functional>

std::vector<int> v = {5,1,2,4,3};

//sort in ascending order (1,2,3,4,5)
std::sort(v.begin(), v.end(), std::less<int>());

// Or just:
std::sort(v.begin(), v.end());

//sort in descending order (5,4,3,2,1)
std::sort(v.begin(), v.end(), std::greater<int>());

//Or just:
std::sort(v.rbegin(), v.rend());

```


In C++14, we don't need to provide the template argument for the comparison function objects and instead let the object deduce based on what it gets passed in:

> 在C++14中，我们不需要为比较函数对象提供模板参数，而是让对象根据传入的内容进行推导：

```cpp
std::sort(v.begin(), v.end(), std::less<>());     // ascending order
std::sort(v.begin(), v.end(), std::greater<>());  // descending order

```



## Sorting sequence containers by overloaded less operator



If no ordering function is passed, `std::sort` will order the elements by calling `operator<` on pairs of elements, which must return a type contextually convertible to `bool` (or just `bool`). Basic types (integers, floats, pointers etc) have already build in comparison operators.

> 如果没有传递排序函数，“std:：sort”将通过对元素对调用“operator<”来对元素进行排序，这必须返回一个在上下文中可转换为“bool”（或仅为“bool'”）的类型。基本类型（整数、浮点、指针等）已经内置了比较运算符。


We can overload this operator to make the default `sort` call work on user-defined types.

> 我们可以重载此运算符，使默认的“排序”调用适用于用户定义的类型。

```cpp
// Include sequence containers
#include <vector>
#include <deque>
#include <list>

// Insert sorting algorithm
#include <algorithm>    

class Base {
 public:

    // Constructor that set variable to the value of v
    Base(int v): variable(v) {
    }
    
    // Use variable to provide total order operator less
    //`this` always represents the left-hand side of the compare.
    bool operator<(const Base &b) const { 
        return this->variable < b.variable;
    }
    
    int variable;
};

int main() {
    std::vector <Base> vector;
    std::deque <Base> deque;
    std::list <Base> list;
    
    // Create 2 elements to sort
    Base a(10);
    Base b(5);
    
    // Insert them into backs of containers
    vector.push_back(a);
    vector.push_back(b);
    
    deque.push_back(a);
    deque.push_back(b);
    
    list.push_back(a);
    list.push_back(b);
    
    // Now sort data using operator<(const Base &b) function
    std::sort(vector.begin(), vector.end());
    std::sort(deque.begin(), deque.end());
    // List must be sorted differently due to its design
    list.sort();

    return 0;
}

```



## Sorting sequence containers using compare function


```cpp
// Include sequence containers
#include <vector>
#include <deque>
#include <list>

// Insert sorting algorithm
#include <algorithm>

class Base {
 public:

    // Constructor that set variable to the value of v
    Base(int v): variable(v) {
    }
       
    int variable;
};

bool compare(const Base &a, const Base &b) {
    return a.variable < b.variable;
}

int main() {
    std::vector <Base> vector;
    std::deque <Base> deque;
    std::list <Base> list;
    
    // Create 2 elements to sort
    Base a(10);
    Base b(5);
    
    // Insert them into backs of containers
    vector.push_back(a);
    vector.push_back(b);
    
    deque.push_back(a);
    deque.push_back(b);
    
    list.push_back(a);
    list.push_back(b);
    
    // Now sort data using comparing function
    std::sort(vector.begin(), vector.end(), compare);
    std::sort(deque.begin(), deque.end(), compare);
    list.sort(compare);

    return 0;
}

```



## Sorting sequence containers using lambda expressions (C++11)


```cpp
// Include sequence containers
#include <vector>
#include <deque>
#include <list>
#include <array>
#include <forward_list>

// Include sorting algorithm
#include <algorithm>

class Base {
 public:

    // Constructor that set variable to the value of v
    Base(int v): variable(v) {
    }
    
    int variable;
};


int main() {
    // Create 2 elements to sort
    Base a(10);
    Base b(5);
    
    // We're using C++11, so let's use initializer lists to insert items.
    std::vector <Base> vector = {a, b};
    std::deque <Base> deque = {a, b};
    std::list <Base> list = {a, b};
    std::array <Base, 2> array = {a, b};
    std::forward_list<Base> flist = {a, b};
    
    // We can sort data using an inline lambda expression
    std::sort(std::begin(vector), std::end(vector),
      [](const Base &a, const Base &b) { return a.variable < b.variable;});

    // We can also pass a lambda object as the comparator
    // and reuse the lambda multiple times
    auto compare = [](const Base &a, const Base &b) {
                     return a.variable < b.variable;};
    std::sort(std::begin(deque), std::end(deque), compare);
    std::sort(std::begin(array), std::end(array), compare);
    list.sort(compare);
    flist.sort(compare);

    return 0;
}

```



## sorting with std::map (ascending and descending)



This example sorts elements in **ascending** order of a **key** using a map.

> 本例使用映射按**键**的**升序对元素进行排序。

You can use any type, including class, instead of `std::string`, in the example below.

> 在下面的示例中，您可以使用任何类型，包括class，而不是“std:：string”。

```cpp
#include <iostream>
#include <utility>
#include <map>

int main()
{
    std::map<double, std::string> sorted_map;
    // Sort the names of the planets according to their size
    sorted_map.insert(std::make_pair(0.3829, "Mercury"));
    sorted_map.insert(std::make_pair(0.9499, "Venus"));
    sorted_map.insert(std::make_pair(1,      "Earth"));
    sorted_map.insert(std::make_pair(0.532,  "Mars"));
    sorted_map.insert(std::make_pair(10.97,  "Jupiter"));
    sorted_map.insert(std::make_pair(9.14,   "Saturn"));
    sorted_map.insert(std::make_pair(3.981,  "Uranus"));
    sorted_map.insert(std::make_pair(3.865,  "Neptune"));

    for (auto const& entry: sorted_map)
    {
        std::cout << entry.second << " (" << entry.first << " of Earth's radius)" << '\n';
    }
}

```

Output:

```cpp
Mercury (0.3829 of Earth's radius)
Mars (0.532 of Earth's radius)
Venus (0.9499 of Earth's radius)
Earth (1 of Earth's radius)
Neptune (3.865 of Earth's radius)
Uranus (3.981 of Earth's radius)
Saturn (9.14 of Earth's radius)
Jupiter (10.97 of Earth's radius)

```


If entries with equal keys are possible, use `multimap` instead of `map` (like in the following example).

> 如果可以使用相同键的条目，请使用“multimap”而不是“map”（如以下示例中所示）。


To sort elements in **descending** manner, declare the map with a proper comparison functor (`std::greater<>`):

> 要以**降序**的方式对元素进行排序，请使用适当的比较函子（`std:：greater<>`）声明映射：

```cpp
#include <iostream>
#include <utility>
#include <map>

int main()
{
    std::multimap<int, std::string, std::greater<int>> sorted_map;
    // Sort the names of animals in descending order of the number of legs
    sorted_map.insert(std::make_pair(6,   "bug"));
    sorted_map.insert(std::make_pair(4,   "cat"));
    sorted_map.insert(std::make_pair(100, "centipede"));
    sorted_map.insert(std::make_pair(2,   "chicken"));
    sorted_map.insert(std::make_pair(0,   "fish"));
    sorted_map.insert(std::make_pair(4,   "horse"));
    sorted_map.insert(std::make_pair(8,   "spider"));

    for (auto const& entry: sorted_map)
    {
        std::cout << entry.second << " (has " << entry.first << " legs)" << '\n';
    }
}

```

Output

```cpp
centipede (has 100 legs)
spider (has 8 legs)
bug (has 6 legs)
cat (has 4 legs)
horse (has 4 legs)
chicken (has 2 legs)
fish (has 0 legs)

```



## Sorting built-in arrays



The `sort` algorithm sorts a sequence defined by two iterators. This is enough to sort a built-in (also known as c-style) array.

> “排序”算法对由两个迭代器定义的序列进行排序。这足以对内置（也称为c样式）数组进行排序。

```cpp
int arr1[] = {36, 24, 42, 60, 59};

// sort numbers in ascending order
sort(std::begin(arr1), std::end(arr1));

// sort numbers in descending order
sort(std::begin(arr1), std::end(arr1), std::greater<int>());

```


Prior to C++11, end of array had to be "calculated" using the size of the array:

> 在C++11之前，必须使用数组的大小“计算”数组的末尾：

```cpp
// Use a hard-coded number for array size
sort(arr1, arr1 + 5);

// Alternatively, use an expression
const size_t arr1_size = sizeof(arr1) / sizeof(*arr1);
sort(arr1, arr1 + arr1_size);

```



#### Remarks


The `std::sort` function family is found in the `algorithm` library.

