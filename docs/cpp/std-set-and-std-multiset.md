---
tip: translate by baidu@2023-10-25 08:32:38
---
---

metaTitle: "C++ | std::set and std::multiset"
description: "Changing the default sort of a set, Deleting values from a set, Inserting values in a set, Inserting values in a multiset, Searching values in set and multiset"
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# std::set and std::multiset

`set` is a type of container whose elements are sorted and unique.  `multiset` is similar, but, in the case of `multiset`, multiple elements can have the same value.

## Changing the default sort of a set

`set` and `multiset` have default compare methods, but in some cases you may need to overload them.

Let's imagine we are storing string values in a set, but we know those strings contain only numeric values. By default the sort will be a lexicographical string comparison, so the order won't match the numerical sort. If you want to apply a sort equivalent to what you would have with `int` values, you need a functor to overload the compare method:

> 假设我们将字符串值存储在一个集合中，但我们知道这些字符串只包含数值。默认情况下，排序将是字典式字符串比较，因此顺序与数字排序不匹配。如果你想应用与“int”值等效的排序，你需要一个函子来重载 compare 方法：

```cpp
#include <iostream>
#include <set>
#include <stdlib.h> 

struct custom_compare final
{
    bool operator() (const std::string& left, const std::string& right) const
    {
        int nLeft = atoi(left.c_str());
        int nRight = atoi(right.c_str());
        return nLeft < nRight;
    }
};

int main ()
{
    std::set<std::string> sut({"1", "2", "5", "23", "6", "290"});
  
    std::cout << "### Default sort on std::set<std::string> :" << std::endl;
    for (auto &&data: sut)
        std::cout << data << std::endl;
  
    std::set<std::string, custom_compare> sut_custom({"1", "2", "5", "23", "6", "290"},
                                                     custom_compare{}); //< Compare object optional as its default constructible.
  
    std::cout << std::endl << "### Custom sort on set :" << std::endl;
    for (auto &&data : sut_custom)
        std::cout << data << std::endl;
  
    auto compare_via_lambda = [](auto &&lhs, auto &&rhs){ return lhs > rhs; };
    using set_via_lambda = std::set<std::string, decltype(compare_via_lambda)>;
    set_via_lambda sut_reverse_via_lambda({"1", "2", "5", "23", "6", "290"},
                                          compare_via_lambda);
  
    std::cout << std::endl << "### Lambda sort on set :" << std::endl;
    for (auto &&data : sut_reverse_via_lambda)
        std::cout << data << std::endl;

    return 0;
}

```

Output will be:

```cpp
### Default sort on std::set<std::string> :
1
2
23
290
5
6
### Custom sort on set :
1
2
5
6
23
290  

### Lambda sort on set :
6
5
290
23
2
1

```

In the example above, one can find 3 different ways of adding compare operations to the `std::set`, each of them is useful in its own context.

> 在上面的例子中，可以找到向“std:：set”添加比较操作的三种不同方法，每种方法在自己的上下文中都很有用。

### Default sort

This will use the compare operator of the key (first template argument).

Often, the key will already provide a good default for the `std::less<T>` function. Unless this function is specialized, it uses the `operator<` of the object.

> 通常，键已经为“std:：less＜T＞”函数提供了一个良好的默认值。除非此函数是专门化的，否则它将使用对象的“运算符 <”。

This is especially useful when other code also tries to use some ordering, as this allows consistency over the whole code base.

> 当其他代码也尝试使用某些排序时，这一点尤其有用，因为这允许在整个代码库中保持一致性。

Writing the code this way, will reduce the effort to update your code when the key changes is API, like: a class containing 2 members which changes to a class containing 3 members. By updating the `operator<` in the class, all occurrences will get updated.

> 当密钥更改为 API 时，以这种方式编写代码将减少更新代码的工作量，例如：一个包含 2 个成员的类更改为一个包含 3 个成员的类时。通过更新类中的“operator<”，所有出现的情况都将得到更新。

As you might expect, using the default sort is a reasonable default.

### Custom sort

Adding a custom sort via an object with a compare operator is often used when the default comparison doesn't comply. In the example above this is because the strings are referring to integers. In other cases, it's often used when you want to compare (smart) pointers based upon the object they refer to or because you need different constraints for comparing (example: comparing `std::pair` by the value of `first`).

> 当默认比较不符合时，通常使用通过带有比较运算符的对象添加自定义排序。在上面的例子中，这是因为字符串指的是整数。在其他情况下，当您想要根据指针所引用的对象来比较（智能）指针时，或者因为需要不同的约束进行比较时，通常会使用它（例如：将“std:：pair”与“first”的值进行比较）。

When creating a compare operator, this should be a stable sorting. If the result of the compare operator changes after insert, you will have undefined behavior. As a good practice, your compare operator should only use the constant data (const members, const functions ...).

> 创建比较运算符时，这应该是一个稳定的排序。如果插入后比较运算符的结果发生更改，则会出现未定义的行为。作为一种好的做法，比较运算符应该只使用常量数据（const 成员、const 函数…）。

As in the example above, you will often encounter classes without members as compare operators. This results in default constructors and copy constructors. The default constructor allows you to omit the instance at construction time and the copy constructor is required as the set takes a copy of the compare operator.

> 在上面的例子中，您经常会遇到没有成员作为比较运算符的类。这会产生默认构造函数和复制构造函数。默认构造函数允许您在构造时省略实例，并且由于集合获取比较运算符的副本，因此需要复制构造函数。

### Lambda sort

[Lambdas](http://stackoverflow.com/documentation/c%2B%2B/572/lambdas) are a shorter way to write function objects. This allows writing the compare operator on less lines, making the overall code more readable.

> [兰达斯](http://stackoverflow.com/documentation/c%2B%2B/572/lambdas)是编写函数对象的较短方法。这允许在更少的行上编写比较运算符，使整个代码更可读。

The disadvantage of the use of lambdas is that each lambda gets a specific type at compile time, so `decltype(lambda)` will be different for each compilation of the same compilation unit (cpp file) as over multiple compilation units (when included via header file). For this reason, its recommended to use function objects as compare operator when used within header files.

> 使用 lambda 的缺点是，每个 lambda 在编译时都有一个特定的类型，因此对于同一编译单元（cpp 文件）的每次编译，“decltype（lambda）”将与多个编译单元（通过头文件包含时）不同。因此，建议在头文件中使用函数对象作为比较运算符。

This construction is often encountered when a `std::set` is used within the local scope of a function instead, while the function object is preferred when used as function arguments or class members.

> 当在函数的本地范围内使用“std:：set”时，经常会遇到这种构造，而函数对象在用作函数参数或类成员时是首选对象。

### Other sort options

As the compare operator of `std::set` is a template argument, all [callable objects](http://stackoverflow.com/documentation/c%2B%2B/6073/callable-objects) can be used as compare operator and the examples above are only specific cases. The only restrictions these callable objects have are:

> 由于“std:：set”的比较运算符是模板参数，因此所有[可调用对象]([http://stackoverflow.com/documentation/c%2b%2b/6073/callable](http://stackoverflow.com/documentation/c%2B%2B/6073/callable)-对象）可以用作比较运算符，并且上面的示例只是特定情况。这些可调用对象的唯一限制是：

- They must be copy constructable
- They must be callable with 2 arguments of the type of the key. (implicit conversions are allowed, though not recommended as it can hurt performance)

> -它们必须可通过键类型的 2 个参数调用。（允许隐式转换，但不建议使用，因为这会影响性能）

## Deleting values from a set

The most obvious method, if you just want to reset your set/multiset to an empty one, is to use `clear`:

> 如果您只想将集合/多集合重置为空集合，最明显的方法是使用“clear”：

```

 std::set<int> sut;
  sut.insert(10);
  sut.insert(15);
  sut.insert(22);
  sut.insert(3); 
  sut.clear(); //size of sut is 0

```

Then the `erase` method can be used.  It offers some possibilities looking somewhat equivalent to the insertion:

> 然后可以使用“擦除”方法。它提供了一些看起来与插入有点相似的可能性：

```cpp
std::set<int> sut;
std::set<int>::iterator it;
          
sut.insert(10);
sut.insert(15);
sut.insert(22);
sut.insert(3); 
sut.insert(30);
sut.insert(33);
sut.insert(45);
    
// Basic deletion
sut.erase(3);
    
// Using iterator
it = sut.find(22);
sut.erase(it);
          
// Deleting a range of values
it = sut.find(33);
sut.erase(it, sut.end());
    
std::cout << std::endl << "Set under test contains:" << std::endl;
for (it = sut.begin(); it != sut.end(); ++it)
{
  std::cout << *it << std::endl;
}

```

Output will be:

```cpp
Set under test contains:                                                                                                                                                                                                                                                                                                  
10                                                                                                                                                                                                                                                                                                                        
15                                                                                                                                                                                                                                                                                                                        
30 

```

All those methods also apply to `multiset`. Please note that if you ask to delete an element from a `multiset`, and it is present multiple times, **all the equivalent values will be deleted**.

> 所有这些方法也适用于“multiset”。请注意，如果您要求从“multiset”中删除一个元素，并且该元素多次出现，**所有等效值都将被删除**。

## Inserting values in a set

Three different methods of insertion can used with sets.

- First, a simple insert of the value. This method returns a pair allowing the caller to check whether the insert really occurred.

> -首先，简单插入值。此方法返回一对，允许调用方检查插入是否真的发生。

- Second, an insert by giving a hint of where the value will be inserted. The objective is to optimize the insertion time in such a case, but knowing where a value should be inserted is not the common case. **Be careful in that case; the way to give a hint differs with compiler versions**.

> -第二，通过提示值将被插入的位置来进行插入。在这种情况下，目标是优化插入时间，但知道值应该插入到哪里并不是常见的情况**在这种情况下要小心；给出提示的方式随编译器版本的不同而不同。

- Finally you can insert a range of values by giving a starting and an ending pointer. The starting one will be included in the insertion, the ending one is excluded.

> -最后，您可以通过给出起始指针和结束指针来插入一系列值。开始的将包含在插入中，结束的将被排除在外。

```cpp
#include <iostream>
#include <set>

int main ()
{
  std::set<int> sut;
  std::set<int>::iterator it;
  std::pair<std::set<int>::iterator,bool> ret;

  // Basic insert
  sut.insert(7);
  sut.insert(5);
  sut.insert(12);
  
  ret = sut.insert(23);
  if (ret.second==true)
    std::cout << "# 23 has been inserted!" << std::endl;
    
  ret = sut.insert(23); // since it's a set and 23 is already present in it, this insert should fail
  if (ret.second==false)
    std::cout << "# 23 already present in set!" << std::endl;
 
  // Insert with hint for optimization
  it = sut.end();
  // This case is optimized for C++11 and above
  // For earlier version, point to the element preceding your insertion
  sut.insert(it, 30); 

  // inserting a range of values
  std::set<int> sut2;
  sut2.insert(20);
  sut2.insert(30);
  sut2.insert(45);
  std::set<int>::iterator itStart = sut2.begin();
  std::set<int>::iterator itEnd = sut2.end();
  
  sut.insert (itStart, itEnd); // second iterator is excluded from insertion

  std::cout << std::endl << "Set under test contains:" << std::endl;
  for (it = sut.begin(); it != sut.end(); ++it)
  {
    std::cout << *it << std::endl;
  }

  return 0;
}

```

Output will be:

```cpp
# 23 has been inserted!                                                                                                                                                                                                                                                                                                   
# 23 already present in set!                                                                                                                                                                                                                                                                                              
                                                                                                                                                                                                                                                                                                                          
Set under test contains:                                                                                                                                                                                                                                                                                                  
5                                                                                                                                                                                                                                                                                                                         
7                                                                                                                                                                                                                                                                                                                         
12                                                                                                                                                                                                                                                                                                                        
20                                                                                                                                                                                                                                                                                                                        
23                                                                                                                                                                                                                                                                                                                        
30                                                                                                                                                                                                                                                                                                                        
45   

```

## Inserting values in a multiset

All the insertion methods from sets also apply to multisets. Nevertheless, another possibility exists, which is providing an initializer_list:

> 集合中的所有插入方法也适用于多集合。然而，存在另一种可能性，即提供 initializer_list：

```cpp
auto il = { 7, 5, 12 };
std::multiset<int> msut;
msut.insert(il);

```

## Searching values in set and multiset

There are several ways to search a given value in `std::set` or in `std::multiset`:

> 有几种方法可以在“std:：set”或“std::：multiset”中搜索给定值：

To get the iterator of the first occurrence of a key, the `find()` function can be used. It returns `end()` if the key does not exist.

> 要获得键第一次出现的迭代器，可以使用“find（）”函数。如果键不存在，则返回“end（）”。

```

 std::set<int> sut;
  sut.insert(10);
  sut.insert(15);
  sut.insert(22);
  sut.insert(3); // contains 3, 10, 15, 22    

  auto itS = sut.find(10); // the value is found, so *itS == 10
  itS = sut.find(555); // the value is not found, so itS == sut.end()   

  std::multiset<int> msut;
  sut.insert(10);
  sut.insert(15);
  sut.insert(22);
  sut.insert(15);
  sut.insert(3); // contains 3, 10, 15, 15, 22  

  auto itMS = msut.find(10);

```

Another way is using the `count()` function, which counts how many corresponding values have been found in the `set`/`multiset` (in case of a `set`, the return value can be only 0 or 1). Using the same values as above, we will have:

> 另一种方法是使用“count（）”函数，该函数统计在“set”/“multiset”中找到的对应值的数量（如果是“set”，则返回值只能是 0 或 1）。使用与上述相同的值，我们将获得：

```cpp
int result = sut.count(10); // result == 1
result = sut.count(555); // result == 0

result = msut.count(10); // result == 1
result = msut.count(15); // result == 2

```

In the case of `std::multiset`, there could be several elements having the same value. To get this range, the `equal_range()` function can be used. It returns `std::pair` having iterator lower bound (inclusive) and upper bound (exclusive) respectively. If the key does not exist, both iterators would point to the nearest superior value (based on compare method used to sort the given `multiset`).

> 在“std:：multiset”的情况下，可能有几个元素具有相同的值。要获得这个范围，可以使用 `equal_range（）` 函数。它返回分别具有迭代器下界（包含）和上界（排除）的“std:：pair”。如果键不存在，两个迭代器都会指向最近的上级值（基于用于对给定“multiset”进行排序的 compare 方法）。

```cpp
auto eqr = msut.equal_range(15);
auto st = eqr.first; // point to first element '15'
auto en = eqr.second; // point to element '22'

eqr = msut.equal_range(9); // both eqr.first and eqr.second point to element '10'

```

#### Remarks

Different styles of C++ have been used in those examples.  Be careful that if you are using a C++98 compiler; some of this code may not be usable.

> 在这些例子中使用了不同风格的 C++。如果您使用的是 C++98 编译器，请注意；其中一些代码可能不可用。
