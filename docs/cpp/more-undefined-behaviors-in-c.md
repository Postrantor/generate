---
tip: translate by baidu@2023-10-25 08:27:58
---
---
metaTitle: "C++ | More undefined behaviors in C++"
description: "Referring to non-static members in initializer lists"
---

# More undefined behaviors in C++


More examples on how C++ can go wrong.


Continuation from [Undefined Behavior](http://stackoverflow.com/documentation/c%2b%2b/1812/undefined-behavior#t=201705050731006995217)

> 接[未定义的行为](http://stackoverflow.com/documentation/c%2b%2b/1812/undefined-行为#t=201705050731006995217）



## Referring to non-static members in initializer lists



Referring to non-static members in initializer lists before the constructor has started executing can result in undefined behavior. This results since not all members are constructed at this time. From the standard draft:

> 在构造函数开始执行之前引用初始值设定项列表中的非静态成员可能会导致未定义的行为。这是因为此时并非所有成员都已构造完成。根据标准草案：

> 
<p>§12.7.1: For an object with a non-trivial constructor, referring to
any non-static member or base class of the object before the
constructor begins execution results in undefined behavior.</p>


**Example**

```cpp
struct W { int j; };
struct X : public virtual W { };
struct Y {
   int *p;
   X x;
   Y() : p(&x.j) { // undefined, x is not yet constructed
   }
};

```

