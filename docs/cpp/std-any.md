---
tip: translate by baidu@2023-10-25 08:32:02
---
---
metaTitle: "C++ | std::any"
description: "Basic usage"
---

# std::any




## Basic usage


```cpp
std::any an_object{ std::string("hello world") };
if (an_object.has_value()) {
    std::cout << std::any_cast<std::string>(an_object) << '\n';
}

try {
  std::any_cast<int>(an_object);
} catch(std::bad_any_cast&) {
    std::cout << "Wrong type\n";
}

std::any_cast<std::string&>(an_object) = "42";
std::cout << std::any_cast<std::string>(an_object) << '\n';

```

Output

```cpp
hello world
Wrong type
42

```



#### Remarks



The class `std::any` provides a type-safe container to which we can put single values of any type.

> 类“std:：any”提供了一个类型安全的容器，我们可以将任何类型的单个值放入其中。

