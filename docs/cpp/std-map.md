---
tip: translate by baidu@2023-10-25 08:32:17
---
---

metaTitle: "C++ | std::map"
description: "Accessing elements, Inserting elements, Searching in std::map or in std::multimap, Initializing a std::map or std::multimap, Checking number of elements, Types of Maps, Deleting elements, Iterating over std::map or std::multimap, Creating std::map with user-defined types as key"
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# std::map

## Accessing elements

An [`std::map`](http://en.cppreference.com/w/cpp/container/map) takes `(key, value)` pairs as input.

> [`std:：映射`](http://en.cppreference.com/w/cpp/container/map)将“（key，value）”对作为输入。

Consider the following example of [`std::map`](http://en.cppreference.com/w/cpp/container/map) initialization:

> 考虑以下[`std:：map`]的示例([http://en.cppreference.com/w/cpp/container/map](http://en.cppreference.com/w/cpp/container/map))初始化：

```cpp
std::map < std::string, int > ranking { std::make_pair("stackoverflow", 2), 
                                        std::make_pair("docs-beta", 1) };

```

In an [`std::map`](http://en.cppreference.com/w/cpp/container/map) , elements can be inserted as follows:

> 在[`std:：map`]中([http://en.cppreference.com/w/cpp/container/map](http://en.cppreference.com/w/cpp/container/map))，元素可以按如下方式插入：

```cpp
ranking["stackoverflow"]=2;
ranking["docs-beta"]=1;

```

In the above example, if the key `stackoverflow` is already present, its value will be updated to 2. If it isn't already present, a new entry will be created.

> 在上面的例子中，如果键“stackoverflow”已经存在，则其值将更新为 2。如果它还不存在，将创建一个新条目。

In an [`std::map`](http://en.cppreference.com/w/cpp/container/map), elements can be accessed directly by giving the key as an index:

> 在[`std:：map`]中([http://en.cppreference.com/w/cpp/container/map](http://en.cppreference.com/w/cpp/container/map))，元素可以通过将键作为索引直接访问：

```cpp
std::cout << ranking[ "stackoverflow" ] << std::endl;

```

Note that using the `operator[]` on the map will actually **insert a new value** with the queried key into the map. This means that you cannot use it on a `const std::map`, even if the key is already stored in the map. To prevent this insertion, check if the element exists (for example by using `find()`) or use `at()` as described below.

> 请注意，在映射上使用“operator[]”实际上会将带有查询键的新值**插入到映射中。这意味着您不能在“conststd:：map”上使用它，即使密钥已经存储在映射中。为了防止这种插入，请检查元素是否存在（例如使用“find（）”）或使用“at（）”，如下所述。

Elements of a `std::map` can be accessed with `at()`:

```cpp
std::cout << ranking.at("stackoverflow") << std::endl;

```

Note that `at()` will throw an `std::out_of_range` exception if the container does not contain the requested element.

> 请注意，如果容器不包含请求的元素，“at（）”将引发“std:：out_of_range”异常。

In both containers `std::map` and [`std::multimap`](http://en.cppreference.com/w/cpp/container/multimap), elements can be accessed using iterators:

> 在两个容器“std:：map”和“[`std:：multimap”中([http://en.cppreference.com/w/cpp/container/multimap](http://en.cppreference.com/w/cpp/container/multimap))，元素可以使用迭代器访问：

```cpp
// Example using begin()
std::multimap < int, std::string > mmp { std::make_pair(2, "stackoverflow"),
                                         std::make_pair(1, "docs-beta"),
                                         std::make_pair(2, "stackexchange")  };
auto it = mmp.begin();
std::cout << it->first << " : " << it->second << std::endl; // Output: "1 : docs-beta"
it++;
std::cout << it->first << " : " << it->second << std::endl; // Output: "2 : stackoverflow"
it++;
std::cout << it->first << " : " << it->second << std::endl; // Output: "2 : stackexchange"

// Example using rbegin()
std::map < int, std::string > mp {  std::make_pair(2, "stackoverflow"),
                                    std::make_pair(1, "docs-beta"),
                                    std::make_pair(2, "stackexchange")  };
auto it2 = mp.rbegin();
std::cout << it2->first << " : " << it2->second << std::endl; // Output: "2 : stackoverflow"
it2++;
std::cout << it2->first << " : " << it2->second << std::endl; // Output: "1 : docs-beta"

```

## Inserting elements

An element can be inserted into a `std::map` only if its key is not already present in the map. Given for example:

> 只有当元素的键尚未出现在映射中时，才能将其插入到“std:：map”中。例如：

```cpp
std::map< std::string, size_t > fruits_count;

```

<li>

A key-value pair is inserted into a `std::map` through the `insert()` member function. It requires a `pair` as an argument:

> 键值对通过“insert（）”成员函数插入到“std:：map”中。它需要一个“pair”作为参数：

```cpp
fruits_count.insert({"grapes", 20});
fruits_count.insert(make_pair("orange", 30));
fruits_count.insert(pair<std::string, size_t>("banana", 40));
fruits_count.insert(map<std::string, size_t>::value_type("cherry", 50));

```

The `insert()` function returns a `pair` consisting of an iterator and a `bool` value:

> “insert（）”函数返回一个由迭代器和“bool”值组成的“对”：

<ul>

- If the insertion was successful, the iterator points to the newly inserted element, and the `bool` value is `true`.

> -如果插入成功，迭代器将指向新插入的元素，“bool”值为“true”。

- If there was already an element with the same `key`, the insertion fails. When that happens, the iterator points to the element causing the conflict, and the `bool` is value is `false`.

> -如果已经存在具有相同“键”的元素，则插入失败。当这种情况发生时，迭代器指向导致冲突的元素，“bool”is 值为“false”。

The following method can be used to combine insertion and searching operation:

> 以下方法可用于组合插入和搜索操作：

```cpp
auto success = fruits_count.insert({"grapes", 20});
if (!success.second) {           // we already have 'grapes' in the map
    success.first->second += 20; // access the iterator to update the value
}

```

For convenience, the `std::map` container provides the subscript operator to access elements in the map and to insert new ones if they don't exist:

> 为了方便起见，“std:：map”容器提供了下标运算符来访问映射中的元素，并在不存在元素时插入新元素：

```cpp
fruits_count["apple"] = 10;

```

While simpler, it prevents the user from checking if the element already exists. If an element is missing, `std::map::operator[]` implicitly creates it, initializing it with the default constructor before overwriting it with the supplied value.

> 虽然更简单，但它可以防止用户检查元素是否已经存在。如果缺少元素，“std:：map:：operator[]”会隐式创建它，在用提供的值覆盖它之前，先用默认构造函数初始化它。

<li>
`insert()` can be used to add several elements at once using a braced list of pairs. This version of insert() returns void:

```cpp
fruits_count.insert({{"apricot", 1}, {"jackfruit", 1}, {"lime", 1}, {"mango", 7}}); 

```

</li>
<li>
`insert()` can also be used to add elements by using iterators denoting the begin and end of `value_type` values:

```cpp
std::map< std::string, size_t > fruit_list{ {"lemon", 0}, {"olive", 0}, {"plum", 0}};
fruits_count.insert(fruit_list.begin(), fruit_list.end()); 

```

</li>

Example:

```cpp
std::map<std::string, size_t> fruits_count;
std::string fruit;
while(std::cin >> fruit){
    // insert an element with 'fruit' as key and '1' as value
    // (if the key is already stored in fruits_count, insert does nothing)
    auto ret = fruits_count.insert({fruit, 1});
    if(!ret.second){            // 'fruit' is already in the map 
        ++ret.first->second;    // increment the counter
    }
}

```

Time complexity for an insertion operation is O(log n) because `std::map` are implemented as trees.

> 插入操作的时间复杂度为 O（logn），因为“std:：map”是作为树实现的。

A `pair` can be constructed explicitly using `make_pair()` and `emplace()`:

> “pair”可以使用“make_pair（）”和“template（）”显式构造：

```cpp
std::map< std::string , int > runs;
runs.emplace("Babe Ruth", 714);
runs.insert(make_pair("Barry Bonds", 762));

```

If we know where the new element will be inserted, then we can use `emplace_hint()` to specify an iterator `hint`. If the new element can be inserted just before `hint`, then the insertion can be done in constant time. Otherwise it behaves in the same way as `emplace()`:

> 如果我们知道新元素将插入到哪里，那么我们可以使用“template_hint（）”来指定迭代器“hint”。如果新元素可以在“提示”之前插入，那么插入可以在恒定时间内完成。否则，它的行为方式与“template（）”相同：

```cpp
std::map< std::string , int > runs;
auto it = runs.emplace("Barry Bonds", 762); // get iterator to the inserted element
// the next element will be before "Barry Bonds", so it is inserted before 'it'
runs.emplace_hint(it, "Babe Ruth", 714);

```

## Searching in std::map or in std::multimap

There are several ways to search a key in `std::map` or in `std::multimap`.

> 有几种方法可以在“std:：map”或“std::：multimap”中搜索关键字。

<li>

To get the iterator of the first occurrence of a key, the `find()` function can be used. It returns `end()` if the key does not exist.

> 要获得键第一次出现的迭代器，可以使用“find（）”函数。如果键不存在，则返回“end（）”。

```cpp
  std::multimap< int , int > mmp{ {1, 2}, {3, 4}, {6, 5}, {8, 9}, {3, 4}, {6, 7} };
  auto it = mmp.find(6);
  if(it!=mmp.end())
      std::cout << it->first << ", " << it->second << std::endl; //prints: 6, 5
  else
      std::cout << "Value does not exist!" << std::endl;

  it = mmp.find(66);
  if(it!=mmp.end())
      std::cout << it->first << ", " << it->second << std::endl; 
  else
      std::cout << "Value does not exist!" << std::endl; // This line would be executed.

```

</li>
<li>

Another way to find whether an entry exists in `std::map` or in `std::multimap` is using the `count()` function, which counts how many values are associated with a given key. Since `std::map` associates only one value with each key, its `count()` function can only return 0 (if the key is not present) or 1 (if it is).  For `std::multimap`, `count()` can return values greater than 1 since there can be several values associated with the same key.

> 另一种查找条目是否存在于“std:：map”或“std::：multimap”中的方法是使用“count（）”函数，该函数计算与给定键关联的值的数量。由于“std:：map”只将一个值与每个键关联，因此其“count（）”函数只能返回 0（如果键不存在）或 1（如果键存在）。对于“std:：multimap”，“count（）”可以返回大于 1 的值，因为可以有多个值与同一个键关联。

```cpp
 std::map< int , int > mp{ {1, 2}, {3, 4}, {6, 5}, {8, 9}, {3, 4}, {6, 7} };
 if(mp.count(3) > 0) // 3 exists as a key in map
     std::cout << "The key exists!" << std::endl; // This line would be executed.
 else
     std::cout << "The key does not exist!" << std::endl;

```

If you only care whether some element exists, `find` is strictly better: it documents your intent and, for `multimaps`, it can stop once the first matching element has been found.

> 如果你只关心某个元素是否存在，“find”会更好：它记录了你的意图，对于“multimaps”，一旦找到第一个匹配的元素，它就可以停止。

</li>
<li>

In the case of `std::multimap`, there could be several elements having the same key. To get this range, the `equal_range()` function is used which returns `std::pair` having iterator lower bound (inclusive) and upper bound (exclusive) respectively. If the key does not exist, both iterators would point to `end()`.

> 在“std:：multimap”的情况下，可能有几个元素具有相同的键。为了获得这个范围，使用了“equal_range（）”函数，该函数返回分别具有迭代器下界（包含）和上界（排除）的“std:：pair”。如果键不存在，两个迭代器都将指向“end（）”。

```cpp
  auto eqr = mmp.equal_range(6);
  auto st = eqr.first, en = eqr.second;
  for(auto it = st; it != en; ++it){
      std::cout << it->first << ", " << it->second << std::endl; 
  }
      // prints: 6, 5
      //         6, 7

```

</li>

## Initializing a std::map or std::multimap

`std::map` and `std::multimap` both can be initialized by providing key-value pairs separated by comma. Key-value pairs could be provided by either `{key, value}` or can be explicitly created by `std::make_pair(key, value)`. As `std::map` does not allow duplicate keys and comma operator performs right to left, the pair on right would be overwritten with the pair with same key on the left.

```cpp
std::multimap < int, std::string > mmp { std::make_pair(2, "stackoverflow"),
                                     std::make_pair(1, "docs-beta"),
                                     std::make_pair(2, "stackexchange")  };
// 1 docs-beta
// 2 stackoverflow
// 2 stackexchange

std::map < int, std::string > mp {  std::make_pair(2, "stackoverflow"),
                                std::make_pair(1, "docs-beta"),
                                std::make_pair(2, "stackexchange")  }; 
// 1 docs-beta
// 2 stackoverflow

```

Both could be initialized with iterator.

```cpp
// From std::map or std::multimap iterator
std::multimap< int , int > mmp{ {1, 2}, {3, 4}, {6, 5}, {8, 9}, {6, 8}, {3, 4}, 
                               {6, 7} };
                       // {1, 2}, {3, 4}, {3, 4}, {6, 5}, {6, 8}, {6, 7}, {8, 9}
auto it = mmp.begin();
std::advance(it,3); //moved cursor on first {6, 5}
std::map< int, int > mp(it, mmp.end()); // {6, 5}, {8, 9}

//From std::pair array
std::pair< int, int > arr[10];
arr[0] = {1, 3};
arr[1] = {1, 5};
arr[2] = {2, 5};
arr[3] = {0, 1};
std::map< int, int > mp(arr,arr+4); //{0 , 1}, {1, 3}, {2, 5}

//From std::vector of std::pair
std::vector< std::pair<int, int> > v{ {1, 5}, {5, 1}, {3, 6}, {3, 2} };
std::multimap< int, int > mp(v.begin(), v.end()); 
                        // {1, 5}, {3, 6}, {3, 2}, {5, 1}

```

## Checking number of elements

The container `std::map` has a member function `empty()`, which returns `true` or `false`, depending on whether the map is empty or not. The member function `size()` returns the number of element stored in a `std::map` container:

> 容器“std:：map”有一个成员函数“empty（）”，它返回“true”或“false”，具体取决于映射是否为空。成员函数“size（）”返回存储在“std:：map”容器中的元素数：

```cpp
std::map<std::string , int> rank {{"facebook.com", 1} ,{"google.com", 2}, {"youtube.com", 3}};
if(!rank.empty()){
    std::cout << "Number of elements in the rank map: " << rank.size() << std::endl;
}
else{
    std::cout << "The rank map is empty" << std::endl;
}

```

## Types of Maps

### Regular Map

A map is an associative container, containing key-value pairs.

```cpp
#include <string>
#include <map>
std::map<std::string, size_t> fruits_count;

```

In the above example, `std::string` is the **key** type, and `size_t` is a **value**.

> 在上面的示例中，“std:：string”是**键**类型，“size_t”是**值**。

The key acts as an index in the map. Each key must be unique, and must be ordered.

> 键在映射中充当索引。每个密钥必须是唯一的，并且必须是有序的。

<li>

If you need mutliple elements with the same key, consider using `multimap` (explained below)

> 如果您需要具有相同键的多个元素，请考虑使用“multimap”（如下所述）

</li>
<li>

If your value type does not specify any ordering, or you want to override the default ordering, you may provide one:

> 如果您的值类型没有指定任何排序，或者您想覆盖默认排序，可以提供一个：

```cpp
#include <string>
#include <map>
#include <cstring>
struct StrLess {
    bool operator()(const std::string& a, const std::string& b) {
        return strncmp(a.c_str(), b.c_str(), 8)<0;
               //compare only up to 8 first characters
    }
}
std::map<std::string, size_t, StrLess> fruits_count2;

```

If `StrLess` comparator returns `false` for two keys, they are considered the same even if their actual contents differ.

> 如果“StrLess”比较器为两个键返回“false”，则即使它们的实际内容不同，也会认为它们是相同的。

</li>

### Multi-Map

Multimap allows multiple key-value pairs with the same key to be stored in the map. Otherwise, its interface and creation is very similar to the regular map.

> 多映射允许在映射中存储具有相同键的多个键值对。除此之外，它的界面和创建与常规贴图非常相似。

```

#include <string>
 #include <map>
 std::multimap<std::string, size_t> fruits_count;
 std::multimap<std::string, size_t, StrLess> fruits_count2;

```

### Hash-Map (Unordered Map)

A hash map stores key-value pairs similar to a regular map. It does not order the elements with respect to the key though. Instead, a [hash](https://en.wikipedia.org/wiki/Hash_function) value for the key is used to quickly access the needed key-value pairs.

> 散列映射存储类似于常规映射的键值对。不过，它并没有根据键对元素进行排序。相反，一个[散列](https://en.wikipedia.org/wiki/Hash_function)键的值用于快速访问所需的键值对。

```cpp
#include <string>
#include <unordered_map>
std::unordered_map<std::string, size_t> fruits_count;

```

Unordered maps are usually faster, but the elements are not stored in any predictable order. For example, iterating over all elements in an `unordered_map` gives the elements in a seemingly random order.

> 无序映射通常会更快，但元素不会以任何可预测的顺序存储。例如，对“unrdered_map”中的所有元素进行迭代，会以看似随机的顺序给出元素。

## Deleting elements

Removing all elements:

```cpp
std::multimap< int , int > mmp{ {1, 2}, {3, 4}, {6, 5}, {8, 9}, {3, 4}, {6, 7} };
mmp.clear(); //empty multimap

```

Removing element from somewhere with the help of iterator:

```cpp
std::multimap< int , int > mmp{ {1, 2}, {3, 4}, {6, 5}, {8, 9}, {3, 4}, {6, 7} };
                            // {1, 2}, {3, 4}, {3, 4}, {6, 5}, {6, 7}, {8, 9}
auto it = mmp.begin();
std::advance(it,3); // moved cursor on first {6, 5}
mmp.erase(it); // {1, 2}, {3, 4}, {3, 4}, {6, 7}, {8, 9}

```

Removing all elements in a range:

```cpp
std::multimap< int , int > mmp{ {1, 2}, {3, 4}, {6, 5}, {8, 9}, {3, 4}, {6, 7} };
                            // {1, 2}, {3, 4}, {3, 4}, {6, 5}, {6, 7}, {8, 9}
auto it = mmp.begin();
auto it2 = it;
it++; //moved first cursor on first {3, 4}
std::advance(it2,3);  //moved second cursor on first {6, 5}
mmp.erase(it,it2); // {1, 2}, {6, 5}, {6, 7}, {8, 9}

```

Removing all elements having a provided value as key:

```cpp
std::multimap< int , int > mmp{ {1, 2}, {3, 4}, {6, 5}, {8, 9}, {3, 4}, {6, 7} };
                            // {1, 2}, {3, 4}, {3, 4}, {6, 5}, {6, 7}, {8, 9}
mmp.erase(6); // {1, 2}, {3, 4}, {3, 4}, {8, 9}

```

Removing elements that satisfy a predicate `pred`:

```cpp
std::map<int,int> m;
auto it = m.begin();
while (it != m.end())
{
   if (pred(*it))
       it = m.erase(it);
   else
       ++it;
}

```

## Iterating over std::map or std::multimap

`std::map` or `std::multimap` could be traversed by the following ways:

```cpp
std::multimap< int , int > mmp{ {1, 2}, {3, 4}, {6, 5}, {8, 9}, {3, 4}, {6, 7} };
                               
//Range based loop - since C++11
for(const auto &x: mmp) 
    std::cout<< x.first <<":"<< x.second << std::endl;

//Forward iterator for loop: it would loop through first element to last element
//it will be a std::map< int, int >::iterator
for (auto it = mmp.begin(); it != mmp.end(); ++it)
std::cout<< it->first <<":"<< it->second << std::endl; //Do something with iterator

//Backward iterator for loop: it would loop through last element to first element
//it will be a std::map< int, int >::reverse_iterator
for (auto it = mmp.rbegin(); it != mmp.rend(); ++it)
std::cout<< it->first <<" "<< it->second << std::endl; //Do something with iterator

```

While iterating over a `std::map` or a `std::multimap`, the use of `auto` is preferred to avoid useless implicit conversions (see [this SO answer](http://stackoverflow.com/questions/32510183/can-the-use-of-c11s-auto-improve-performance) for more details).

> 当迭代“std:：map”或“std：：multimap”时，最好使用“auto”以避免无用的隐式转换（请参见 [this SO answer](http://stackoverflow.com/questions/32510183/can-the-use-of-c11s-auto-improve-performance) 了解更多详细信息）。

## Creating std::map with user-defined types as key

In order to be able to use a class as the key in a map, all that is required of the key is that it be `copiable` and `assignable`.

> 为了能够使用类作为映射中的键，键所需要的就是它是“可复制的”和“可分配的”。
> The ordering within the map is defined by the third argument to the
> template (and the argument to the constructor, if used).  This
> **defaults** to `std::less<KeyType>`, which defaults to the `<` operator,
> but there's no requirement to use the defaults.  Just write a comparison
> operator (preferably as a functional object):

```cpp
struct CmpMyType
{
    bool operator()( MyType const& lhs, MyType const& rhs ) const
    {
        //  ...
    }
};

```

In C++, the "compare" predicate must be a [strict weak ordering](http://www.sgi.com/tech/stl/StrictWeakOrdering.html). In particular, `compare(X,X)` must return `false` for any `X`. i.e. if `CmpMyType()(a, b)` returns true, then `CmpMyType()(b, a)` must return false, and if both return false, the elements are considered equal (members of the same equivalence class).

> 在 C++ 中，“compare”谓词必须是[严格弱排序](http://www.sgi.com/tech/stl/StrictWeakOrdering.html)。特别是，对于任何“X”，“compare（X，X）”都必须返回“false”。即，如果“CmpMyType（）（a，b）”返回 true，则“CmpMyStype（）（b，a）”必须返回 false，如果两者都返回 false，则元素被视为相等（相同等价类的成员）。

### Strict Weak Ordering

This is a mathematical term to define a relationship between two objects.<br />

> 这是一个数学术语，用来定义两个对象之间的关系<br/>
> Its definition is:

>

Two objects x and y are equivalent if both f(x, y) and f(y, x) are false. Note that an object is always (by the irreflexivity invariant) equivalent to itself.

> 如果 f（x，y）和 f（y，x）都为 false，那么两个对象 x 和 y 是等价的。注意，一个对象总是（通过非弹性不变量）等价于它自己。

In terms of C++ this means if you have two objects of a given type, you should return the following values when compared with the operator <.

> 就 C++ 而言，这意味着如果你有两个给定类型的对象，当与运算符 < 进行比较时，你应该返回以下值。

```cpp
X    a;
X    b;

Condition:                  Test:     Result
a is equivalent to b:       a < b     false
a is equivalent to b        b < a     false

a is less than b            a < b     true
a is less than b            b < a     false

b is less than a            a < b     false
b is less than a            b < a     true

```

How you define equivalent/less is totally dependent on the type of your object.

> 如何定义 equival/less 完全取决于对象的类型。

#### Remarks

<li>

To use any of `std::map` or `std::multimap` the header file `<map>` should be included.

> 要使用“std:：map”或“std::：multimap”中的任何一个，应包括头文件“＜map＞”。

</li>
<li>
`std::map` and `std::multimap` both keep their elements sorted according to the ascending order of keys. In case of `std::multimap`, no sorting occurs for the values of the same key.
</li>
<li>

The basic difference between `std::map` and `std::multimap` is that the `std::map` one does not allow duplicate values for the same key where `std::multimap` does.

> std:：map 和 std:：multimap 之间的基本区别在于，std:：map'不允许对同一个键使用重复值，而 std:：polymap 则允许。

</li>
<li>

Maps are implemented as binary search trees. So `search()`, `insert()`, `erase()` takes Θ(log n) time in average. For constant time operation use `std::unordered_map`.

> 映射被实现为二进制搜索树。因此，“search（）”、“insert（）”和“erase（）”平均花费 Θ（logn）时间。对于恒定时间运算，请使用“std:：unordered_map”。

</li>
<li>
`size()` and `empty()` functions have Θ(1) time complexity, number of nodes is cached to avoid walking through tree each time these functions are called.
</li>
