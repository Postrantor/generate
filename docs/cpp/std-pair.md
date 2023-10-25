---
tip: translate by baidu@2023-10-25 08:32:37
---
---
metaTitle: "C++ | std::pair"
description: "Creating a Pair and accessing the elements, Compare operators"
---

# std::pair



## Creating a Pair and accessing the elements



Pair allows us to treat two objects as one object. Pairs can be easily constructed with the help of template function `std::make_pair`.

> Pair允许我们将两个对象视为一个对象。在模板函数“std:：make_pair”的帮助下，可以很容易地构建对。


Alternative way is to create pair and assign its elements (`first` and `second`) later.

> 另一种方法是创建对并稍后分配其元素（“first”和“second”）。

```cpp
#include <iostream>
#include <utility>

int main()
{
    std::pair<int,int> p = std::make_pair(1,2); //Creating the pair
    std::cout << p.first << " " << p.second << std::endl; //Accessing the elements




    //We can also create a pair and assign the elements later
    std::pair<int,int> p1;
    p1.first = 3;
    p1.second = 4;
    std::cout << p1.first << " " << p1.second << std::endl;

    //We can also create a pair using a constructor
    std::pair<int,int> p2 = std::pair<int,int>(5, 6);
    std::cout << p2.first << " " << p2.second << std::endl;

    return 0;
}

```



## Compare operators


Parameters of these operators are `lhs` and `rhs`


- `operator==` tests if both elements on `lhs` and `rhs` pair are equal. The return value is `true` if both `lhs.first == rhs.first` AND  `lhs.second == rhs.second`, otherwise `false`

> -“operator==”测试“lhs”和“rhs”对上的两个元素是否相等。如果两个`lhs.first==rhs.first`AND`lhs.second==rhs.second`，则返回值为`true`，否则为`false`

```cpp
std::pair<int, int> p1 = std::make_pair(1, 2);
std::pair<int, int> p2 = std::make_pair(2, 2);

if (p1 == p2)
    std::cout << "equals";
else
    std::cout << "not equal"//statement will show this, because they are not identical

```


<li>
`operator!=`  tests if any elements on `lhs` and `rhs` pair are not equal. The return value is `true` if either `lhs.first != rhs.first` OR `lhs.second != rhs.second`, otherwise return `false`.
</li>
<li>
`operator<` tests if `lhs.first<rhs.first`, returns `true`. Otherwise, if `rhs.first<lhs.first` returns `false`. Otherwise, if `lhs.second<rhs.second` returns `true`, otherwise, returns `false`.
</li>
<li>
`operator<=` returns `!(rhs<lhs)`
</li>
<li>
`operator>` returns `rhs<lhs`
</li>
<li>
`operator>=` returns `!(lhs<rhs)`

Another example with containers of pairs. It uses `operator<` because it needs to sort container.

> 另一个例子是成对的容器。它使用“operator<”，因为它需要对容器进行排序。
</li>

```cpp
#include <iostream>
#include <utility>
#include <vector>
#include <algorithm>
#include <string>
 
int main()
{
    std::vector<std::pair<int, std::string>> v = { {2, "baz"},
                                                   {2, "bar"},
                                                   {1, "foo"} };
    std::sort(v.begin(), v.end());
 
    for(const auto& p: v) {
        std::cout << "(" << p.first << "," << p.second << ") ";
        //output: (1,foo) (2,bar) (2,baz)
    }
}

```

