---
tip: translate by baidu@2023-10-25 08:23:21
---
---

metaTitle: "C++ | Copy Elision"
description: "Purpose of copy elision, Guaranteed copy elision, Parameter elision, Return value elision, Named return value elision, Copy initialization elision"
-----------------------------------------------------------------------------------------------------------------------------------------------------------------

# Copy Elision

## Purpose of copy elision

There are places in the standard where an object is copied or moved in order to initialize an object. Copy elision (sometimes called return value optimization) is an optimization whereby, under certain specific circumstances, a compiler is permitted to avoid the copy or move even though the standard says that it must happen.

> 标准中有一些地方复制或移动对象以初始化对象。复制省略（有时称为返回值优化）是一种优化，在某些特定情况下，允许编译器避免复制或移动，即使标准规定必须这样做。

Consider the following function:

```cpp
std::string get_string()
{
  return std::string("I am a string.");
}

```

According to the strict wording of the standard, this function will initialize a temporary `std::string`, then copy/move that into the return value object, then destroy the temporary. The standard is very clear that this is how the code is interpreted.

> 根据标准的严格措辞，此函数将初始化一个临时的“std:：string”，然后将其复制/移动到返回值对象中，然后销毁临时的。标准非常清楚，这就是代码的解释方式。

Copy elision is a rule that permits a C++ compiler to **ignore** the creation of the temporary and its subsequent copy/destruction. That is, the compiler can take the initializing expression for the temporary and initialize the function's return value from it directly. This obviously saves performance.

> 拷贝省略是一条允许 C++ 编译器**忽略**临时文件及其后续拷贝/销毁的规则。也就是说，编译器可以将初始化表达式作为临时表达式，并直接从中初始化函数的返回值。这显然节省了性能。

However, it does have two visible effects on the user:

<li>

The type must have the copy/move constructor that would have been called. Even if the compiler elides the copy/move, the type must still be able to have been copied/moved.

> 该类型必须具有本应调用的复制/移动构造函数。即使编译器取消了复制/移动，该类型也必须能够被复制/移动。

</li>
<li>

Side-effects of copy/move constructors are not guaranteed in circumstances where elision can happen. Consider the following:

> 在可能发生省略的情况下，不能保证复制/移动构造函数的副作用。考虑以下内容：

</li>

```cpp
struct my_type
{
  my_type() = default;
  my_type(const my_type &) {std::cout <<"Copying\n";}
  my_type(my_type &&) {std::cout <<"Moving\n";}
};

my_type func()
{
  return my_type();
}

```

What will calling `func` do? Well, it will never print "Copying", since the temporary is an rvalue and `my_type` is a moveable type. So will it print "Moving"?

> 调用“func”会做什么？好吧，它永远不会打印“复制”，因为临时是一个右值，而“my_type”是一个可移动的类型。那么它会印“移动”吗？

Without the copy elision rule, this would be required to always print "Moving". But because the copy elision rule exists, the move constructor may or may not be called; it is implementation-dependent.

> 如果没有副本省略规则，则需要始终打印“移动”。但是由于复制省略规则的存在，移动构造函数可能被调用，也可能不被调用；它依赖于实现。

And therefore, you cannot depend on the calling of copy/move constructors in contexts where elision is possible.

> 因此，在可能省略的上下文中，不能依赖于对复制/移动构造函数的调用。

Because elision is an optimization, your compiler may not support elision in all cases. And regardless of whether the compiler elides a particular case or not, the type must still support the operation being elided. So if a copy construction is elided, the type must still have a copy constructor, even though it will not be called.

> 因为省略是一种优化，所以编译器可能不支持所有情况下的省略。无论编译器是否消除了特定的情况，类型都必须仍然支持被消除的操作。因此，如果取消了复制构造，则该类型必须仍然有一个复制构造函数，即使它不会被调用。

## Guaranteed copy elision

Normally, elision is an optimization. While virtually every compiler support copy elision in the simplest of cases, having elision still places a particular burden on users. Namely, the type who's copy/move is being elided **must** still have the copy/move operation that was elided.

> 通常，省略是一种优化。虽然几乎每个编译器在最简单的情况下都支持拷贝省略，但省略仍然会给用户带来特别的负担。也就是说，正在取消复制/移动的类型**必须**仍然具有已取消的复制/移动操作。

For example:

```cpp
std::mutex a_mutex;
std::lock_guard<std::mutex> get_lock()
{
  return std::lock_guard<std::mutex>(a_mutex);
}

```

This might be useful in cases where `a_mutex` is a mutex that is privately held by some system, yet an external user might want to have a scoped lock to it.

> 在“a_mutex”是某个系统私有的互斥体，但外部用户可能希望对其具有作用域锁的情况下，这可能很有用。

This is also not legal, because `std::lock_guard` cannot be copied or moved. Even though virtually every C++ compiler will elide the copy/move, the standard still **requires** the type to have that operation available.

> 这也是不合法的，因为不能复制或移动“std:：lock_guard”。尽管几乎每个 C++ 编译器都会消除复制/移动，但标准仍然需要**类型才能使用该操作。

Until C++17.

C++17 mandates elision by effectively redefining the very meaning of certain expressions so that no copy/moving takes place. Consider the above code.

> C++17 通过有效地重新定义某些表达式的含义来强制省略，这样就不会发生复制/移动。请考虑上面的代码。

Under pre-C++17 wording, that code says to create a temporary and then use the temporary to copy/move into the return value, but the temporary copy can be elided. Under C++17 wording, that does not create a temporary at all.

> 在 C++17 之前的措辞中，该代码表示创建一个临时，然后使用临时来复制/移动到返回值中，但临时副本可以被删除。根据 C++17 的措辞，这根本不会创建一个临时的。

In C++17, any [prvalue expression](http://stackoverflow.com/documentation/c%2B%2B/763/value-categories/2603/prvalue#t=201607301613191630536), when used to initialize an object of the same type as the expression, does not generate a temporary. The expression directly initializes that object. If you return a prvalue of the same type as the return value, then the type need not have a copy/move constructor. And therefore, under C++17 rules, the above code can work.

> 在 C++17 中，任何[prvalue 表达式]([http://stackoverflow.com/documentation/c%2B%2B/763/value-categories/2603/prvalue#t=201607301613191630536](http://stackoverflow.com/documentation/c%2B%2B/763/value-categories/2603/prvalue#t=201607301613191630536)），当用于初始化与表达式类型相同的对象时，不会生成临时。表达式直接初始化该对象。如果返回与返回值类型相同的 prvalue，则该类型不需要具有复制/移动构造函数。因此，在 C++17 规则下，上述代码可以工作。

The C++17 wording works in cases where the prvalue's type matches the type being initialized. So given `get_lock` above, this will also not require a copy/move:

> C++17 语句适用于 prvalue 的类型与正在初始化的类型匹配的情况。因此，给定上面的“get_lock”，这也不需要复制/移动：

```cpp
std::lock_guard the_lock = get_lock();

```

Since the result of `get_lock` is a prvalue expression being used to initialize an object of the same type, no copying or moving will happen. That expression never creates a temporary; it is used to directly initialize `the_lock`. There is no elision because there is no copy/move to be elided elide.

> 由于“get_lock”的结果是用于初始化相同类型对象的 prvalue 表达式，因此不会发生复制或移动。这个表达从来不会创造一个临时的；它用于直接初始化“锁定”。没有省略，因为没有要被省略的复制/移动。

The term "guaranteed copy elision" is therefore something of a misnomer, but [that is the name of the feature as it is proposed for C++17 standardization](http://wg21.link/P0135). It does not guarantee elision at all; it **eliminates** the copy/move altogether, redefining C++ so that there never was a copy/move to be elided.

> 因此，术语“保证拷贝省略”有点用词不当，但[这是为 C++17 标准化而提出的功能名称](http://wg21.link/P0135)。它根本不能保证省略；它**完全消除了**拷贝/移动，重新定义了 C++，这样就永远不会有拷贝/移动被消除。

This feature only works in cases involving a prvalue expression. As such, this uses the usual elision rules:

> 此功能仅适用于涉及 prvalue 表达式的情况。因此，这使用了通常的省略规则：

```cpp
std::mutex a_mutex;
std::lock_guard<std::mutex> get_lock()
{
  std::lock_guard<std::mutex> my_lock(a_mutex);
  //Do stuff
  return my_lock;
}

```

While this is a valid case for copy elision, C++17 rules do not **eliminate** the copy/move in this case. As such, the type must still have a copy/move constructor to use to initialize the return value. And since `lock_guard` does not, this is still a compile error.

> 虽然这是一个有效的拷贝省略情况，但在这种情况下，C++17 规则不会**消除**拷贝/移动。因此，该类型必须仍然有一个复制/移动构造函数来初始化返回值。由于“lock_guard”没有，这仍然是一个编译错误。

Implementations are allowed to refuse to elide copies when passing or returning an object of trivially-copyable type. This is to allow moving such objects around in registers, which some ABIs might mandate in their calling conventions.

> 当传递或返回一个普通可复制类型的对象时，允许实现拒绝消除副本。这是为了允许在寄存器中移动此类对象，一些 ABI 可能会在其调用约定中强制要求这样做。

```cpp
struct trivially_copyable {
    int a;  
};

void foo (trivially_copyable a) {}

foo(trivially_copyable{}); //copy elision not mandated

```

## Parameter elision

When you pass an argument to a function, and the argument is a [prvalue expression](http://stackoverflow.com/documentation/c%2B%2B/763/value-categories/2603/prvalue#t=201607301618015777687) of the function's parameter type, and this type is not a reference, then the prvalue's construction can be elided.

> 当您将参数传递给函数，并且该参数是[prvalue 表达式]时([http://stackoverflow.com/documentation/c%2B%2B/763/value-categories/2603/prvalue#t=201607301618015777687](http://stackoverflow.com/documentation/c%2B%2B/763/value-categories/2603/prvalue#t=201607301618015777687)），并且该类型不是引用，则可以省略 prvalue 的构造。

```cpp
void func(std::string str) { ... }

func(std::string("foo"));

```

This says to create a temporary `string`, then move it into the function parameter `str`. Copy elision permits this expression to directly create the object in `str`, rather than using a temporary+move.

> 这意味着创建一个临时的“字符串”，然后将其移动到函数参数“str”中。复制省略允许此表达式直接在“str”中创建对象，而不是使用临时 + 移动。

This is a useful optimization for cases where a constructor is declared `explicit`. For example, we could have written the above as `func("foo")`, but only because `string` has an implicit constructor that converts from a `const char*` to a `string`. If that constructor was `explicit`, we would be forced to use a temporary to call the `explicit` constructor. Copy elision saves us from having to do a needless copy/move.

> 对于构造函数被声明为“显式”的情况，这是一个有用的优化。例如，我们本可以将上面的内容写成“func（“foo”）”，但这只是因为“string”有一个隐式构造函数，可以从“const char*”转换为“string”。如果该构造函数是“显式”的，我们将被迫使用临时构造函数来调用“显式的”构造函数。复制省略使我们不必进行不必要的复制/移动。

## Return value elision

If you return a [prvalue expression](http://stackoverflow.com/documentation/c%2B%2B/763/value-categories/2603/prvalue#t=201607301619277418431) from a function, and the prvalue expression has the same type as the function's return type, then the copy from the prvalue temporary can be elided:

> 如果返回[prvalue 表达式]([http://stackoverflow.com/documentation/c%2B%2B/763/value-categories/2603/prvalue#t=201607301619277418431](http://stackoverflow.com/documentation/c%2B%2B/763/value-categories/2603/prvalue#t=201607301619277418431)），并且 prvalue 表达式的类型与函数的返回类型相同，则可以消除来自 prvalue temporary 的副本：

```cpp
std::string func()
{
  return std::string("foo");
}

```

Pretty much all compilers will elide the temporary construction in this case.

> 在这种情况下，几乎所有的编译器都会省略临时构造。

## Named return value elision

If you return an [lvalue expression](http://stackoverflow.com/documentation/c%2B%2B/763/value-categories/2605/lvalue#t=201607301614328409642) from a function, and this lvalue:

> 如果返回[lvalue 表达式]([http://stackoverflow.com/documentation/c%2B%2B/763/value-categories/2605/lvalue#t=201607301614328409642](http://stackoverflow.com/documentation/c%2B%2B/763/value-categories/2605/lvalue#t=201607301614328409642)），并且该左值：

- represents an automatic variable local to that function, which will be destroyed after the `return`

> -表示该函数的本地自动变量，该变量将在“return”之后销毁 `

- the automatic variable is not a function parameter
- and the type of the variable is the same type as the function's return type

If all of these are the case, then the copy/move from the lvalue can be elided:

> 如果所有这些都是这样，则可以消除从左值复制/移动：

```cpp
std::string func()
{
  std::string str("foo");
  //Do stuff
  return str;
}

```

More complex cases are eligible for elision, but the more complex the case, the less likely the compiler will be to actually elide it:

> 更复杂的情况可以省略，但情况越复杂，编译器实际省略的可能性就越小：

```cpp
std::string func()
{
  std::string ret("foo");
  if(some_condition)
  {
    return "bar";
  }
  return ret;
}

```

The compiler could still elide `ret`, but the chances of them doing so go down.

> 编译器仍然可以省略“ret”，但这样做的可能性降低了。

As noted earlier, elision is not permitted for value **parameters**.

```cpp
std::string func(std::string str)
{
  str.assign("foo");
  //Do stuff
  return str; //No elision possible
}

```

## Copy initialization elision

If you use a [prvalue expression](http://stackoverflow.com/documentation/c%2B%2B/763/value-categories/2603/prvalue#t=201607301613191630536) to copy initialize a variable, and that variable has the same type as the prvalue expression, then the copying can be elided.

> 如果使用[prvalue 表达式]([http://stackoverflow.com/documentation/c%2B%2B/763/value-categories/2603/prvalue#t=201607301613191630536](http://stackoverflow.com/documentation/c%2B%2B/763/value-categories/2603/prvalue#t=201607301613191630536)）来复制初始化变量，并且该变量具有与 prvalue 表达式相同的类型，则可以取消复制。

```cpp
std::string str = std::string("foo");

```

Copy initialization effectively transforms this into `std::string str("foo");` (there are minor differences).

> 复制初始化有效地将其转换为 `std:：string str（“foo”）；`（略有不同）。

This also works with return values:

```cpp
std::string func()
{
  return std::string("foo");
}

std::string str = func();

```

Without copy elision, this would provoke 2 calls to `std::string`'s move constructor. Copy elision permits this to call the move constructor 1 or zero times, and most compilers will opt for the latter.

> 如果没有复制省略，这将引发对“std:：string”的 move 构造函数的 2 次调用。复制省略允许它调用 move 构造函数 1 次或 0 次，大多数编译器都会选择后者。
