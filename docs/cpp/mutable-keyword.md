---
tip: translate by baidu@2023-10-25 08:28:09
---
---
metaTitle: "C++ | mutable keyword"
description: "mutable lambdas, non-static class member modifier"
---

# mutable keyword



## mutable lambdas



By default, the implicit `operator()` of a lambda is `const`. This disallows performing non-`const` operations on the lambda. In order to allow modifying members, a lambda may be marked `mutable`, which makes the implicit `operator()` non-`const`:

> 默认情况下，lambda的隐式“operator（）”是“const”。这不允许对lambda执行非const操作。为了允许修改成员，lambda可以标记为“mutable”，这使得隐式“operator（）”非“const”：

```cpp
int a = 0;

auto bad_counter = [a] {
    return a++;   // error: operator() is const
                  // cannot modify members
};

auto good_counter = [a]() mutable {
    return a++;  // OK
}

good_counter(); // 0
good_counter(); // 1
good_counter(); // 2

```



## non-static class member modifier


`mutable` modifier in this context is used to indicate that a data field of a const object may be modified without affecting the externally-visible state of the object.


If you are thinking about caching a result of expensive computation, you should probably use this keyword.

> 如果您正在考虑缓存昂贵计算的结果，则可能应该使用此关键字。


If you have a lock (for example, `std::unique_lock`) data field which is locked and unlocked inside a const method, this keyword is also what you could use.

> 如果您在const方法中有一个锁定和解锁的锁（例如，“std:：unique_lock”）数据字段，那么您也可以使用此关键字。

You should not use this keyword to break logical const-ness of an object.

Example with caching:

```cpp
class pi_calculator {
 public:
     double get_pi() const {
         if (pi_calculated) {
             return pi;
         } else {
             double new_pi = 0;
             for (int i = 0; i < 1000000000; ++i) {
                 // some calculation to refine new_pi
             }
             // note: if pi and pi_calculated were not mutable, we would get an error from a compiler
             // because in a const method we can not change a non-mutable field
             pi = new_pi;
             pi_calculated = true;
             return pi;
         }
     }
 private:
     mutable bool pi_calculated = false;
     mutable double pi = 0;
};

```

