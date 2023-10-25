---
tip: translate by baidu@2023-10-25 08:32:31
---
---

metaTitle: "C++ | std::optional"
description: "Using optionals to represent the absence of a value, optional as return value, Introduction, value_or, Using optionals to represent the failure of a function"
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# std::optional

## Using optionals to represent the absence of a value

Before C++17, having pointers with a value of `nullptr` commonly represented the absence of a value. This is a good solution for large objects that have been dynamically allocated and are already managed by pointers. However, this solution does not work well for small or primitive types such as `int`, which are rarely ever dynamically allocated or managed by pointers. `std::optional` provides a viable solution to this common problem.

> 在 C++17 之前，指针的值为“nullptr”通常表示没有值。对于已经动态分配并且已经由指针管理的大型对象，这是一个很好的解决方案。然而，这种解决方案对诸如“int”之类的小型或基元类型不太适用，因为它们很少由指针动态分配或管理 `std:：optional 为这个常见问题提供了一个可行的解决方案。

In this example, `struct Person` is defined. It is possible for a person to have a pet, but not necessary. Therefore, the `pet` member of `Person` is declared with an `std::optional` wrapper.

> 在本例中，定义了“struct Person”。一个人可以养宠物，但不是必须的。因此，“Person”的“pet”成员是用“std:：optional”包装器声明的。

```cpp
#include <iostream>
#include <optional>
#include <string>

struct Animal {
    std::string name;
};

struct Person {
    std::string name;
    std::optional<Animal> pet;
};

int main() {
    Person person;
    person.name = "John";

    if (person.pet) {
        std::cout << person.name << "'s pet's name is " <<
            person.pet->name << std::endl;
    }
    else {
        std::cout << person.name << " is alone." << std::endl;
    }
}

```

## optional as return value

```cpp
std::optional<float> divide(float a, float b) {
  if (b!=0.f) return a/b;
  return {};
}

```

Here we return either the fraction `a/b`, but if it is not defined (would be infinity) we instead return the empty optional.

> 在这里，我们返回分数“a/b”，但如果它没有定义（将是无穷大），我们将返回空的可选值。

A more complex case:

```cpp
template<class Range, class Pred>
auto find_if( Range&& r, Pred&& p ) {
  using std::begin; using std::end;
  auto b = begin(r), e = end(r);
  auto r = std::find_if(b, e , p );
  using iterator = decltype(r);
  if (r==e)
    return std::optional<iterator>();
  return std::optional<iterator>(r);
}
template<class Range, class T>
auto find( Range&& r, T const& t ) {
  return find_if( std::forward<Range>(r), [&t](auto&& x){return x==t;} );
}

```

`find( some_range, 7 )` searches the container or range `some_range` for something equal to the number `7`.  `find_if` does it with a predicate.

It returns either an empty optional if it was not found, or an optional containing an iterator tothe element if it was.

> 如果找不到它，它将返回一个空的可选项，如果找到了，则返回一个包含元素迭代器的可选项。

This allows you to do:

```cpp
if (find( vec, 7 )) {
  // code
}

```

or even

```cpp
if (auto oit = find( vec, 7 )) {
  vec.erase(*oit);
}

```

without having to mess around with begin/end iterators and tests.

## Introduction

Optionals (also known as Maybe types) are used to represent a type whose contents may or may not be present. They are implemented in C++17 as the `std::optional` class. For example, an object of type `std::optional<int>` may contain some value of type `int`, or it may contain no value.

> 选项（也称为 Maybe 类型）用于表示其内容可能存在也可能不存在的类型。它们在 C++17 中被实现为“std:：optional”类。例如，“std:：optional＜int＞”类型的对象可能包含某种“int”类型的值，也可能不包含任何值。

Optionals are commonly used either to represent a value that may not exist or as a return type from a function that can fail to return a meaningful result.

> 选项通常用于表示可能不存在的值，或者作为函数的返回类型，该函数可能无法返回有意义的结果。

### Other approaches to optional

There are many other approach to solving the problem that `std::optional` solves, but none of them are quite complete: using a pointer, using a sentinel, or using a `pair<bool, T>`.

> 还有许多其他方法可以解决“std:：optional”解决的问题，但都不完全：使用指针、使用 sentinel 或使用“pair＜bool，T＞”。

### Optional vs Pointer

In some cases, we can provide a pointer to an existing object or `nullptr` to indicate failure. But this is limited to those cases where objects already exist - `optional`, as a value type, can also be used to return new objects without resorting to memory allocation.

> 在某些情况下，我们可以提供一个指向现有对象的指针或“nullptr”来指示失败。但这仅限于对象已经存在的情况——“可选”作为一种值类型，也可以用于返回新对象，而无需进行内存分配。

### Optional vs Sentinel

A common idiom is to use a special value to indicate that the value is meaningless. This may be 0 or -1 for integral types, or `nullptr` for pointers. However, this reduces the space of valid values (you cannot differentiate between a valid 0 and a meaningless 0) and many types do not have a natural choice for the sentinel value.

> 一个常见的成语是用一个特殊的值来表示这个值没有意义。对于整型，它可以是 0 或-1，对于指针，它可以为“nullptr”。然而，这减少了有效值的空间（无法区分有效的 0 和无意义的 0），并且许多类型没有对 sentinel 值的自然选择。

### Optional vs `std::pair<bool, T>`

Another common idiom is to provide a pair, where one of the elements is a `bool` indicating whether or not the value is meaningful.

> 另一个常见的习惯用法是提供一对，其中一个元素是“bool”，表示该值是否有意义。

This relies upon the value type being default-constructible in the case of error, which is not possible for some types and possible but undesirable for others. An `optional<T>`, in the case of error, does not need to construct anything.

> 这取决于在出现错误的情况下，值类型是默认可构造的，这对于某些类型是不可能的，对于其他类型是可能的，但不可取。在出现错误的情况下，“optional＜T＞”不需要构造任何内容。

## value_or

```cpp
void print_name( std::ostream& os, std::optional<std::string> const& name ) {
  std::cout "Name is: " << name.value_or("<name missing>") << '\n';
}

```

`value_or` either returns the value stored in the optional, or the argument if there is nothing store there.

This lets you take the maybe-null optional and give a default behavior when you actually need a value.  By doing it this way, the "default behavior" decision can be pushed back to the point where it is best made and immediately needed, instead of generating some default value deep in the guts of some engine.

> 这使您可以选择 maybe null 选项，并在实际需要值时提供默认行为。通过这样做，可以将“默认行为”决策推回到最适合做出和立即需要的位置，而不是在某个引擎的内部生成一些默认值。

## Using optionals to represent the failure of a function

Before C++17, a function typically represented failure in one of several ways:

> 在 C++17 之前，函数通常以以下几种方式之一表示失败：

<li>A null pointer was returned.
<ul>

- e.g. Calling a function `Delegate *App::get_delegate()` on an `App` instance that did not have a delegate would return `nullptr`.

> -例如，在没有委托的“App”实例上调用函数“Delegate*App:：get_Delegate（）”将返回“nullptr”。

- This is a good solution for objects that have been dynamically allocated or are large and managed by pointers, but isn't a good solution for small objects that are typically stack-allocated and passed by copying.

> -对于已动态分配的对象或大型对象并由指针管理的对象，这是一个很好的解决方案，但对于通常通过复制进行堆栈分配和传递的小型对象，这不是一个不错的解决方案。

- e.g. Calling a function `unsigned shortest_path_distance(Vertex a, Vertex b)` on two vertices that are not connected may return zero to indicate this fact.

> -例如，在未连接的两个顶点上调用函数“unsigned shortest_path_distance（顶点 a，顶点 b）”可以返回零来指示这一事实。

- e.g. Calling a function `std::pair<int, bool> parse(const std::string &str)` with a string argument that is not an integer would return a pair with an undefined `int` and a `bool` set to `false`.

> -例如，使用非整数的字符串参数调用函数“std:：pair＜int，bool＞parse（const std:：string&str）”将返回一对未定义的“int”和设置为“false”的“bool”。

In this example, John is given two pets, Fluffy and Furball. The function `Person::pet_with_name()` is then called to retrieve John's pet Whiskers. Since John does not have a pet named Whiskers, the function fails and `std::nullopt` is returned instead.

> 在这个例子中，约翰得到了两只宠物，Fluffy 和 Furball。然后调用函数“Person:：pet_with_name（）”来检索约翰的宠物威士忌。由于 John 没有名为 Whiskers 的宠物，因此函数失败，返回“std:：nullopt”。

```cpp
#include <iostream>
#include <optional>
#include <string>
#include <vector>

struct Animal {
    std::string name;
};

struct Person {
    std::string name;
    std::vector<Animal> pets;
    
    std::optional<Animal> pet_with_name(const std::string &name) {
        for (const Animal &pet : pets) {
            if (pet.name == name) {
                return pet;
            }
        }
        return std::nullopt;
    }
};

int main() {
    Person john;
    john.name = "John";
    
    Animal fluffy;
    fluffy.name = "Fluffy";
    john.pets.push_back(fluffy);
    
    Animal furball;
    furball.name = "Furball";
    john.pets.push_back(furball);
    
    std::optional<Animal> whiskers = john.pet_with_name("Whiskers");
    if (whiskers) {
        std::cout << "John has a pet named Whiskers." << std::endl;
    }
    else {
        std::cout << "Whiskers must not belong to John." << std::endl;
    }
}

```
