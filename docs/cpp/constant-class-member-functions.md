---
tip: translate by baidu@2023-10-25 08:23:12
---
---
metaTitle: "C++ | Constant class member functions"
description: "constant member function"
---

# Constant class member functions



## constant member function


```cpp
#include <iostream>
#include <map>
#include <string>

using namespace std;

class A {
public:
    map<string, string> * mapOfStrings;
public:
    A() {
        mapOfStrings = new map<string, string>();
    }

    void insertEntry(string const & key, string const & value) const {
        (*mapOfStrings)[key] = value;             // This works? Yes it does. 
        delete mapOfStrings;                      // This also works
        mapOfStrings = new map<string, string>(); // This * does * not work
    }

    void refresh() {
        delete mapOfStrings;
        mapOfStrings = new map<string, string>(); // Works as refresh is non const function
    }

    void getEntry(string const & key) const {
        cout << mapOfStrings->at(key);
    }
};

int main(int argc, char* argv[]) {

    A var;
    var.insertEntry("abc", "abcValue");
    var.getEntry("abc");
    getchar();
    return 0;
}

```



#### Remarks



What does 'const member functions' of a class really means. The simple definition seems to be that, a const member function cannot change the object. But what does 'can not change' really means here. It simply means that you cannot do an assignment for class data members.

> 类的“constmemberfunctions”的真正含义是什么。简单的定义似乎是，const成员函数不能更改对象。但“不能改变”在这里真正意味着什么。这只是意味着您不能为类数据成员进行赋值。


However, you can do other indirect operations like inserting an entry into a map as shown in the example. Allowing this might look like this const function is modifying the object (yes, it does in one sense), but it is allowed.

> 但是，您可以执行其他间接操作，如在映射中插入条目，如示例所示。允许这样做可能看起来像是这个const函数在修改对象（是的，从某种意义上来说确实如此），但这是允许的。


So, the real meaning is that a const member function cannot do an assignment for the class data variables. But it can do other stuff like explained in the example.

> 因此，真正的意义是const成员函数不能为类数据变量赋值。但是它可以做其他的事情，就像例子中解释的那样。

