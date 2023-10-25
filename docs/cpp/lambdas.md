---
tip: translate by baidu@2023-10-25 08:26:43
---
---
metaTitle: "C++ | Lambdas"
description: "What is a lambda expression?, Specifying the return type, Capture by value, Recursive lambdas, Default capture, Capture by reference, Generic lambdas, Class lambdas and capture of this, Generalized capture, Conversion to function pointer, Using lambdas for inline parameter pack unpacking, Porting lambda functions to C++03 using functors"
---

# Lambdas




## What is a lambda expression?



A **lambda expression** provides a concise way to create simple function objects. A lambda expression is a prvalue whose result object is called [closure object](https://en.wikipedia.org/wiki/Closure_(computer_programming)), which behaves like a function object.

> **lambda表达式**提供了一种创建简单函数对象的简洁方法。lambda表达式是一个prvalue，其结果对象称为[close-object](https://en.wikipedia.org/wiki/Closure_（computer_programing）），其行为类似于函数对象。


The name 'lambda expression' originates from [lambda calculus](https://en.wikipedia.org/wiki/Lambda_calculus), which is a mathematical formalism invented in the 1930s by Alonzo Church to investigate questions about logic and computability. Lambda calculus formed the basis of [LISP](https://en.wikipedia.org/wiki/Lisp_(programming_language)), a functional programming language. Compared to lambda calculus and LISP, C++ lambda expressions share the properties of being unnamed, and to capture variables from the surrounding context, but they lack the ability to operate on and return functions.

> 名称“lambda expression”源自[lambda演算](https://en.wikipedia.org/wiki/Lambda_calculus)，是阿隆佐·丘奇在20世纪30年代发明的一种数学形式主义，用于研究有关逻辑和可计算性的问题。Lambda演算是[LISP]的基础(https://en.wikipedia.org/wiki/Lisp_（programming_language）），一种函数式编程语言。与lambda演算和LISP相比，C++lambda表达式共享未命名的特性，以及从周围上下文捕获变量的特性，但它们缺乏对函数进行操作和返回函数的能力。


A lambda expression is often used as an argument to functions that take a callable object. That can be simpler than creating a named function, which would be only used when passed as the argument. In such cases, lambda expressions are generally preferred because they allow defining the function objects inline.

> lambda表达式通常用作接受可调用对象的函数的参数。这可能比创建一个命名函数更简单，该函数仅在作为参数传递时使用。在这种情况下，lambda表达式通常是首选，因为它们允许内联定义函数对象。


A lambda consists typically of three parts: a capture list `[]`, an optional parameter list `()` and a body `{}`, all of which can be empty:

> lambda通常由三部分组成：捕获列表“[]”、可选参数列表“（）”和正文“｛｝”，所有这些部分都可以为空：

```cpp
[](){}                // An empty lambda, which does and returns nothing

```

**Capture list**

`[]` is the **capture list**. By default, variables of the enclosing scope cannot be accessed by a lambda. **Capturing** a variable makes it accessible inside the lambda, either [as a copy](http://stackoverflow.com/documentation/c%2b%2b/572/lambdas/1856/capture-by-value#t=201607271323349545754) or [as a reference](http://stackoverflow.com/documentation/c%2b%2b/572/lambdas/1951/capture-by-reference#t=201607271323349545754). Captured variables become a part of the lambda; in contrast to function arguments, they do not have to be passed when calling the lambda.

```cpp
int a = 0;                       // Define an integer variable
auto f = []()   { return a*9; }; // Error: 'a' cannot be accessed
auto f = [a]()  { return a*9; }; // OK, 'a' is "captured" by value
auto f = [&a]() { return a++; }; // OK, 'a' is "captured" by reference
                                 //      Note: It is the responsibility of the programmer
                                 //      to ensure that a is not destroyed before the
                                 //      lambda is called.
auto b = f();                    // Call the lambda function. a is taken from the capture list and not passed here.

```

**Parameter list**

`()` is the **parameter list**, which is almost the same as in regular functions. If the lambda takes no arguments, these parentheses can be omitted (except if you need to declare the lambda `mutable`). These two lambdas are equivalent:

```cpp
auto call_foo  = [x](){ x.foo(); };
auto call_foo2 = [x]{ x.foo(); };

```


The parameter list can use the placeholder type `auto` instead of actual types. By doing so, this argument behaves like a template parameter of a function template. Following lambdas are equivalent when you want to sort a vector in generic code:

> 参数列表可以使用占位符类型“auto”而不是实际类型。通过这样做，此参数的行为类似于函数模板的模板参数。当您想要在泛型代码中对向量进行排序时，以下lambda是等效的：

```cpp
auto sort_cpp11 = [](std::vector<T>::const_reference lhs, std::vector<T>::const_reference rhs) { return lhs < rhs; }; 
auto sort_cpp14 = [](const auto &lhs, const auto &rhs) { return lhs < rhs; }; 

```

**Function body**

`{}` is the **body**, which is the same as in regular functions.

**Calling a lambda**


A lambda expression's result object is a [closure](https://en.wikipedia.org/wiki/Closure_(computer_programming)), which can be called using the `operator()` (as with other function objects):

> lambda表达式的结果对象是[闭包](https://en.wikipedia.org/wiki/Closure_（computer_programing）），可以使用“operator（）”调用（与其他函数对象一样）：

```cpp
int multiplier = 5;
auto timesFive = [multiplier](int a) { return a * multiplier; }; 
std::out << timesFive(2); // Prints 10

multiplier = 15;
std::out << timesFive(2); // Still prints 2*5 == 10

```

**Return Type**

By default, the return type of a lambda expression is deduced.

```cpp
[](){ return true; };

```

In this case the return type is `bool`.

You can also manually specify the return type using the following syntax:

```cpp
[]() -> bool { return true; };

```

**Mutable Lambda**


Objects captured by value in the lambda are by default immutable. This is because the `operator()` of the generated closure object is `const` by default.

> 默认情况下，由lambda中的值捕获的对象是不可变的。这是因为生成的闭包对象的“operator（）”默认为“const”。

```cpp
auto func = [c = 0](){++c; std::cout << c;};  // fails to compile because ++c
                                              // tries to mutate the state of
                                              // the lambda.

```


Modifying can be allowed by using the keyword `mutable`, which make the closer object's `operator()` non-`const`:

> 可以通过使用关键字“mutable”进行修改，这会使closer对象的“operator（）”非“const”：

```cpp
auto func = [c = 0]() mutable {++c; std::cout << c;};

```

If used together with the return type, `mutable` comes before it.

```cpp
auto func = [c = 0]() mutable -> int {++c; std::cout << c; return c;};

```

**An example to illustrate the usefulness of lambdas**

Before C++11:

```cpp
// Generic functor used for comparison
struct islessthan
{
    islessthan(int threshold) : _threshold(threshold) {}

    bool operator()(int value) const
    {
        return value < _threshold;
    }
private:
    int _threshold;
};

// Declare a vector
const int arr[] = { 1, 2, 3, 4, 5 };
std::vector<int> vec(arr, arr+5);

// Find a number that's less than a given input (assume this would have been function input)
int threshold = 10;
std::vector<int>::iterator it = std::find_if(vec.begin(), vec.end(), islessthan(threshold));

```

Since C++11:

```cpp
// Declare a vector
std::vector<int> vec{ 1, 2, 3, 4, 5 };

// Find a number that's less than a given input (assume this would have been function input)
int threshold = 10;
auto it = std::find_if(vec.begin(), vec.end(), [threshold](int value) { return value < threshold; });

```



## Specifying the return type



For lambdas with a single return statement, or multiple return statements whose expressions are of the same type, the compiler can deduce the return type:

> 对于具有单个返回语句的lambda，或具有相同类型表达式的多个返回语句，编译器可以推导出返回类型：

```cpp
// Returns bool, because "value > 10" is a comparison which yields a Boolean result
auto l = [](int value) {
    return value > 10;
}

```


For lambdas with multiple return statements of **different** types, the compiler can't deduce the return type:

> 对于具有**不同**类型的多个返回语句的lambda，编译器无法推导出返回类型：

```cpp
// error: return types must match if lambda has unspecified return type
auto l = [](int value) {
    if (value < 10) {
        return 1;
    } else {
        return 1.5;
    }
};

```

In this case you have to specify the return type explicitly:

```cpp
// The return type is specified explicitly as 'double'
auto l = [](int value) -> double {
    if (value < 10) {
        return 1;
    } else {
        return 1.5;
    }
};

```


The rules for this match the rules for `auto` type deduction. Lambdas without explicitly specified return types never return references, so if a reference type is desired it must be explicitly specified as well:

> 这方面的规则与“自动”类型推导的规则相匹配。没有显式指定返回类型的Lambdas永远不会返回引用，因此如果需要引用类型，也必须显式指定它：

```cpp
auto copy = [](X& x) { return x; };       // 'copy' returns an X, so copies its input
auto ref  = [](X& x) -> X& { return x; }; // 'ref' returns an X&, no copy

```



## Capture by value



If you specify the variable's name in the capture list, the lambda will capture it by value. This means that the generated closure type for the lambda stores a copy of the variable. This also requires that the variable's type be **copy-constructible**:

> 如果在捕获列表中指定变量的名称，lambda将按值捕获它。这意味着为lambda生成的闭包类型存储变量的副本。这也要求变量的类型是可复制的**：

```cpp
int a = 0;

[a]() {
    return a;   // Ok, 'a' is captured by value
};

```

```cpp
auto p = std::unique_ptr<T>(...);

[p]() {         // Compile error; `unique_ptr` is not copy-constructible
    return p->createWidget(); 
};

```


From C++14 on, it is possible to initialize variables on the spot. This allows move only types to be captured in the lambda.

> 从C++14开始，可以在现场初始化变量。这允许在lambda中捕获仅移动类型。

```cpp
auto p = std::make_unique<T>(...);

[p = std::move(p)]() {
    return p->createWidget(); 
};

```


Even though a lambda captures variables by value when they are given by their name, such variables cannot be modified within the lambda body by default. This is because the closure type puts the lambda body in a declaration of `operator() const`.

> 即使lambda在按名称给定变量时按值捕获变量，默认情况下也不能在lambda主体中修改此类变量。这是因为闭包类型将lambda主体放在“operator（）const”的声明中。


The `const` applies to accesses to member variables of the closure type, and captured variables that are members of the closure (all appearances to the contrary):

> “const”适用于对闭包类型的成员变量以及作为闭包成员的捕获变量的访问（所有表现都相反）：

```cpp
int a = 0;

[a]() {
    a = 2;      // Illegal, 'a' is accessed via `const`

    decltype(a) a1 = 1; 
    a1 = 2; // valid: variable 'a1' is not const
};

```


To remove the `const`, you have to specify the keyword `mutable` on the lambda:

> 要删除“const”，必须在lambda上指定关键字“mutable”：

```cpp
int a = 0;

[a]() mutable {
    a = 2;      // OK, 'a' can be modified
    return a;
};

```


Because `a` was captured by value, any modifications done by calling the lambda will not affect `a`. The value of `a` was copied into the lambda when it was constructed, so the lambda's copy of `a` is separate from the external `a` variable.

> 因为“a”是由值捕获的，所以通过调用lambda所做的任何修改都不会影响“a”。“a”的值在构造时被复制到lambda中，因此lambda的“a”副本与外部“a”变量是分开的。

```cpp
int a = 5 ; 
auto plus5Val = [a] (void) { return a + 5 ; } ; 
auto plus5Ref = [&a] (void) {return a + 5 ; } ; 

a = 7 ; 
std::cout << a << ", value " << plus5Val() << ", reference " << plus5Ref() ;
// The result will be "7, value 10, reference 12"

```



## Recursive lambdas



Let's say we wish to write Euclid's `gcd()` as a lambda. As a function, it is:

> 假设我们希望将欧几里得的“gcd（）”写成lambda。作为一个函数，它是：

```cpp
int gcd(int a, int b) {
    return b == 0 ? a : gcd(b, a%b);
}

```


But a lambda cannot be recursive, it has no way to invoke itself. A lambda has no name and using `this` within the body of a lambda refers to a captured `this` (assuming the lambda is created in the body of a member function, otherwise it is an error). So how do we solve this problem?

> 但是lambda不能是递归的，它没有办法调用自己。lambda没有名称，在lambda的主体中使用“this”指的是捕获的“this”（假设lambda是在成员函数的主体中创建的，否则就是错误）。那么我们该如何解决这个问题呢？

### Use `std::function`


We can have a lambda capture a reference to a not-yet constructed `std::function`:

> 我们可以让lambda捕获对尚未构造的“std:：函数”的引用：

```cpp
std::function<int(int, int)> gcd = [&](int a, int b){
    return b == 0 ? a : gcd(b, a%b);
};

```


This works, but should be used sparingly. It's slow (we're using type erasure now instead of a direct function call), it's fragile (copying `gcd` or returning `gcd` will break since the lambda refers to the original object), and it won't work with generic lambdas.

> 这是有效的，但应该谨慎使用。它很慢（我们现在使用类型擦除而不是直接的函数调用），它很脆弱（复制“gcd”或返回“gcd”会中断，因为lambda引用了原始对象），而且它不能与泛型lambda一起使用。

### Using two smart pointers:

```cpp
auto gcd_self = std::make_shared<std::unique_ptr< std::function<int(int, int)> >>();
*gcd_self = std::make_unique<std::function<int(int, int)>>(
  [gcd_self](int a, int b){
    return b == 0 ? a : (**gcd_self)(b, a%b);
  };
};

```


This adds a lot of indirection (which is overhead), but it can be copied/returned, and all copies share state.  It does let you return the lambda, and is otherwise less fragile than the above solution.

> 这增加了很多间接性（这是开销），但它可以被复制/返回，并且所有副本都共享状态。它确实允许您返回lambda，并且在其他方面不如上面的解决方案那么脆弱。

### Use a Y-combinator


With the help of a short utility struct, we can solve all of these problems:

> 借助一个简短的实用程序结构，我们可以解决所有这些问题：

```cpp
template <class F>
struct y_combinator {
    F f; // the lambda will be stored here

    // a forwarding operator():
    template <class... Args>
    decltype(auto) operator()(Args&&... args) const {
        // we pass ourselves to f, then the arguments.
        // the lambda should take the first argument as `auto&& recurse` or similar.
        return f(*this, std::forward<Args>(args)...);
    }
};
// helper function that deduces the type of the lambda:
template <class F>
y_combinator<std::decay_t<F>> make_y_combinator(F&& f) {
    return {std::forward<F>(f)};
}
// (Be aware that in C++17 we can do better than a `make_` function)

```

we can implement our `gcd` as:

```cpp
auto gcd = make_y_combinator(
  [](auto&& gcd, int a, int b){
    return b == 0 ? a : gcd(b, a%b);
  }
);

```


The `y_combinator` is a concept from the lambda calculus that lets you have recursion without being able to name yourself until you are defined.  This is exactly the problem lambdas have.

> “y_combinator”是lambda演算中的一个概念，它允许您进行递归，而在定义之前无法命名自己。这正是lambdas所面临的问题。


You create a lambda that takes "recurse" as its first argument.  When you want to recurse, you pass the arguments to recurse.

> 您创建了一个以“recurse”作为第一个参数的lambda。当您想要递归时，您可以将参数传递给递归。


The `y_combinator` then returns a function object that calls that function with its arguments, but with a suitable "recurse" object (namely the `y_combinator` itself) as its first argument.  It forwards the rest of the arguments you call the `y_combinator` with to the lambda as well.

> 然后，“y_combinator”返回一个函数对象，该对象用其参数调用该函数，但用一个合适的“递归”对象（即“y_cominator”本身）作为其第一个参数。它还将调用“y_combinator”的其余参数转发给lambda。

In short:

```cpp
auto foo = make_y_combinator( [&](auto&& recurse, some arguments) {
  // write body that processes some arguments
  // when you want to recurse, call recurse(some other arguments)
});

```


and you have recursion in a lambda with no serious restrictions or significant overhead.

> 在lambda中有递归，没有严重的限制或显著的开销。



## Default capture



By default, local variables that are not explicitly specified in the capture list, cannot be accessed from within the lambda body. However, it is possible to implicitly capture variables named by the lambda body:

> 默认情况下，无法从lambda主体中访问捕获列表中未显式指定的局部变量。但是，可以隐式捕获由lambda主体命名的变量：

```cpp
int a = 1;
int b = 2;

// Default capture by value
[=]() { return a + b; }; // OK; a and b are captured by value

// Default capture by reference
[&]() { return a + b; }; // OK; a and b are captured by reference

```


Explicit capturing can still be done alongside implicit default capturing. The explicit capture definition will override the default capture:

> 显式捕获仍然可以与隐式默认捕获一起进行。显式捕获定义将覆盖默认捕获：

```cpp
int a = 0;
int b = 1;

[=, &b]() {
    a = 2; // Illegal; 'a' is capture by value, and lambda is not 'mutable'
    b = 2; // OK; 'b' is captured by reference
};

```



## Capture by reference



If you precede a local variable's name with an `&`, then the variable will be captured by reference. Conceptually, this means that the lambda's closure type will have a reference variable, initialized as a reference to the corresponding variable from outside of the lambda's scope. Any use of the variable in the lambda body will refer to the original variable:

> 如果局部变量的名称前面有一个“&”，则该变量将通过引用捕获。从概念上讲，这意味着lambda的闭包类型将有一个引用变量，初始化为对lambda范围外相应变量的引用。lambda主体中变量的任何使用都将引用原始变量：

```cpp
// Declare variable 'a'
int a = 0;

// Declare a lambda which captures 'a' by reference
auto set = [&a]() {
    a = 1;
};

set();
assert(a == 1);

```

The keyword `mutable` is not needed, because `a` itself is not `const`.


Of course, capturing by reference means that the lambda **must not** escape the scope of the variables it captures. So you could call functions that take a function, but you must not call a function that will **store** the lambda beyond the scope of your references. And you must not return the lambda.

> 当然，通过引用捕获意味着lambda**不能**脱离它捕获的变量的范围。因此，您可以调用接受函数的函数，但不能调用将**存储**lambda超出引用范围的函数。并且不能返回lambda。



## Generic lambdas



Lambda functions can take arguments of arbitrary types. This allows a lambda to be more generic:

> Lambda函数可以采用任意类型的参数。这使得lambda更加通用：

```cpp
auto twice = [](auto x){ return x+x; };

int i = twice(2); // i == 4
std::string s = twice("hello"); // s == "hellohello"

```


This is implemented in C++ by making the closure type's `operator()` overload a template function. The following type has equivalent behavior to the above lambda closure:

> 这是在C++中通过使闭包类型的“operator（）”重载模板函数来实现的。以下类型具有与上述lambda闭包等效的行为：

```cpp
struct _unique_lambda_type
{
  template<typename T>
  auto operator() (T x) const {return x + x;}
};

```

Not all parameters in a generic lambda need be generic:

```cpp
[](auto x, int y) {return x + y;}

```


Here, `x` is deduced based on the first function argument, while `y` will always be `int`.

> 这里，“x”是根据第一个函数参数推导出来的，而“y”将始终是“int”。


Generic lambdas can take arguments by reference as well, using the usual rules for `auto` and `&`. If a generic parameter is taken as `auto&&`, this is a [**forwarding** reference](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2014/n4164.pdf) to the passed in argument and not an [**rvalue** reference](http://en.cppreference.com/w/cpp/language/reference):

> 泛型lambda也可以通过引用接受参数，使用“auto”和“&”的常用规则。如果一个通用参数被取为“auto&&”，这是一个[**转发**引用](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2014/n4164.pdf)传递到传入的参数，而不是[**rvalue**reference](http://en.cppreference.com/w/cpp/language/reference)：

```cpp
auto lamb1 = [](int &&x) {return x + 5;};
auto lamb2 = [](auto &&x) {return x + 5;};
int x = 10;
lamb1(x); // Illegal; must use `std::move(x)` for `int&&` parameters.
lamb2(x); // Legal; the type of `x` is deduced as `int&`.

```

Lambda functions can be variadic and perfectly forward their arguments:

```cpp
auto lam = [](auto&&... args){return f(std::forward<decltype(args)>(args)...);};

```

or:

```cpp
auto lam = [](auto&&... args){return f(decltype(args)(args)...);};

```

which only works "properly" with variables of type `auto&&`.

A strong reason to use generic lambdas is for visiting syntax.

```cpp
boost::variant<int, double> value;
apply_visitor(value, [&](auto&& e){
  std::cout << e;
});

```


Here we are visiting in a polymorphic manner; but in other contexts, the names of the type we are passing isn't interesting:

> 我们以多态的方式访问这里；但在其他情况下，我们传递的类型的名称并不有趣：

```cpp
mutex_wrapped<std::ostream&> os = std::cout;
os.write([&](auto&& os){
  os << "hello world\n";
});

```


Repeating the type of `std::ostream&` is noise here; it would be like having to mention the type of a variable every time you use it.  Here we are creating a visitor, but no a polymorphic one; `auto` is used for the same reason you might use `auto` in a `for(:)` loop.

> 重复“std:：ostream&”类型在这里是噪声；这就像每次使用变量时都要提到它的类型一样。这里我们创建了一个访问者，但没有多态的访问者`auto的使用原因与在for（：）循环中使用auto的原因相同。



## Class lambdas and capture of this



A lambda expression evaluated in a class' member function is implicitly a friend of that class:

> 在类的成员函数中计算的lambda表达式隐式地是该类的友元：

```cpp
class Foo
{
private:
    int i;
    
public:
    Foo(int val) : i(val) {}
    
    // definition of a member function
    void Test()
    {
        auto lamb = [](Foo &foo, int val)
        {
            // modification of a private member variable
            foo.i = val;
        };
        
        // lamb is allowed to access a private member, because it is a friend of Foo
        lamb(*this, 30);
    }
};

```


Such a lambda is not only a friend of that class, it has the same access as the class it is declared within has.

> 这样的lambda不仅是该类的友元，它还具有与在其中声明的类相同的访问权限。


Lambdas can capture the `this` pointer which represents the object instance the outer function was called on. This is done by adding `this` to the capture list:

> Lambdas可以捕获表示调用外部函数的对象实例的“this”指针。这可以通过将“this”添加到捕获列表中来完成：

```cpp
class Foo
{
private:
    int i;
    
public:
    Foo(int val) : i(val) {}
    
    void Test()
    {
        // capture the this pointer by value
        auto lamb = [this](int val)
        {
            i = val;
        };
        
        lamb(30);
    }
};

```


When `this` is captured, the lambda can use member names of its containing class as though it were in its containing class. So an implicit `this->` is applied to such members.

> 当捕获“this”时，lambda可以使用其包含类的成员名，就好像它在其包含类中一样。因此，一个隐含的“this->”应用于这样的成员。


Be aware that `this` is captured by value, but not the value of the type. It is captured by the value of `this`, which is a **pointer**. As such, the lambda does not **own** `this`. If the lambda out lives the lifetime of the object that created it, the lambda can become invalid.

> 请注意，“this”是由值捕获的，而不是由类型的值捕获的。它由“this”的值捕获，该值是一个**指针**。因此，lambda不**拥有**“this”。如果lambda超出了创建它的对象的生存期，则lambda可能会变得无效。


This also means that the lambda can modify `this` without being declared `mutable`. It is the pointer which is `const`, not the object being pointed to. That is, unless the outer member function was itself a `const` function.

> 这也意味着lambda可以修改“This”而不必声明为“mutable”。是指针是“const”，而不是被指向的对象。也就是说，除非外部成员函数本身是“const”函数。


Also, be aware that the default capture clauses, both `[=]` and `[&]`, will **also** capture `this` implicitly. And they both capture it by the value of the pointer. Indeed, it is an error to specify `this` in the capture list when a default is given.

> 此外，请注意，默认捕获子句“[=]”和“[&]'也将**隐式捕获“this”。它们都通过指针的值来捕获它。事实上，当给定默认值时，在捕获列表中指定“this”是错误的。


Lambdas can capture a copy of the `this` object, created at the time the lambda is created. This is done by adding `*this` to the capture list:

> Lambdas可以捕获在创建lambda时创建的“this”对象的副本。这是通过将“*This”添加到捕获列表中来完成的：

```cpp
class Foo
{
private:
    int i;
    
public:
    Foo(int val) : i(val) {}
    
    void Test()
    {
        // capture a copy of the object given by the this pointer
        auto lamb = [*this](int val) mutable
        {
            i = val;
        };
        
        lamb(30); // does not change this->i
    }
};

```



## Generalized capture



Lambdas can capture expressions, rather than just variables. This permits lambdas to store move-only types:

> Lambdas可以捕获表达式，而不仅仅是变量。这允许Lambda存储仅移动类型：

```cpp
auto p = std::make_unique<T>(...);

auto lamb = [p = std::move(p)]() //Overrides capture-by-value of `p`.
{
  p->SomeFunc();
};

```


This moves the outer `p` variable into the lambda capture variable, also called `p`. `lamb` now owns the memory allocated by `make_unique`. Because the closure contains a type that is non-copyable, this means that `lamb` is itself non-copyable. But it can be moved:

> 这会将外部的“p”变量移动到lambda捕获变量中，也称为“p”`lamb现在拥有由make_unique分配的内存。因为闭包包含一个不可复制的类型，这意味着“lamb”本身就是不可复制。但它可以移动：

```cpp
auto lamb_copy = lamb; //Illegal
auto lamb_move = std::move(lamb); //legal.

```

Now `lamb_move` owns the memory.


Note that `std::function<>` requires that the values stored be copyable. You can write your own [move-only-requiring `std::function`](https://stackoverflow.com/documentation/c%2b%2b/2872/type-erasure/18042/a-move-only-stdfunction#t=201608061624094409548), or you could just stuff the lambda into a `shared_ptr` wrapper:

> 请注意，“std:：function＜＞”要求存储的值是可复制的。您可以编写自己的[move只需要`std:：function`](https://stackoverflow.com/documentation/c%2b%2b/2872/type-擦除/18042/a-move-only-stdfunction#t=201608061624094409548），或者您可以将lambda填充到“shared_ptr”包装器中：

```cpp
auto shared_lambda = [](auto&& f){
  return [spf = std::make_shared<std::decay_t<decltype(f)>>(decltype(f)(f))]
  (auto&&...args)->decltype(auto) {
    return (*spf)(decltype(args)(args)...);
  };
};
auto lamb_shared = shared_lambda(std::move(lamb_move));

```


takes our move-only lambda and stuffs its state into a shared pointer then returns a lambda that **can** be copied, and then stored in a `std::function` or similar.

> 获取我们的move-only lambda，并将其状态填充到共享指针中，然后返回一个可以**复制的lambda，然后存储在“std:：function”或类似的函数中。


Generalized capture uses `auto` type deduction for the variable's type. It will declare these captures as values by default, but they can be references as well:

> 广义捕获对变量的类型使用“auto”类型推导。默认情况下，它会将这些捕获声明为值，但它们也可以是引用：

```cpp
int a = 0;

auto lamb = [&v = a](int add) //Note that `a` and `v` have different names
{
  v += add; //Modifies `a`
};

lamb(20); //`a` becomes 20.

```


Generalize capture does not need to capture an external variable at all. It can capture an arbitrary expression:

> 泛化捕获根本不需要捕获外部变量。它可以捕获任意表达式：

```cpp
auto lamb = [p = std::make_unique<T>(...)]()
{
    p->SomeFunc();
}

```


This is useful for giving lambdas arbitrary values that they can hold and potentially modify, without having to declare them externally to the lambda. Of course, that is only useful if you do not intend to access those variables after the lambda has completed its work.

> 这对于为lambda提供任意值非常有用，它们可以保存并可能修改这些值，而不必在lambda外部声明这些值。当然，只有当您不打算在lambda完成工作后访问这些变量时，这才有用。



## Conversion to function pointer



If a lambda's capture list is empty, then the lambda has an implicit conversion to a function pointer that takes the same arguments and returns the same return type:

> 如果lambda的捕获列表为空，则lambda将隐式转换为采用相同参数并返回相同返回类型的函数指针：

```cpp
auto sorter = [](int lhs, int rhs) -> bool {return lhs < rhs;};

using func_ptr = bool(*)(int, int);
func_ptr sorter_func = sorter; // implicit conversion

```

Such a conversion may also be enforced using unary plus operator:

```cpp
func_ptr sorter_func2 = +sorter; // enforce implicit conversion

```


Calling this function pointer behaves exactly like invoking `operator()` on the lambda. This function pointer is in no way reliant on the source lambda closure's existence. It therefore may outlive the lambda closure.

> 调用此函数指针的行为与在lambda上调用“operator（）”完全相同。这个函数指针决不依赖于源lambda闭包的存在。因此，它可能比lambda闭包更长寿。


This feature is mainly useful for using lambdas with APIs that deal in function pointers, rather than C++ function objects.

> 这个特性主要适用于将lambdas与处理函数指针而不是C++函数对象的API一起使用。


Conversion to a function pointer is also possible for generic lambdas with an empty capture list. If necessary, template argument deduction will be used to select the correct specialization.

> 对于具有空捕获列表的通用lambda，也可以转换为函数指针。如有必要，将使用模板参数推导来选择正确的专业化。

```cpp
auto sorter = [](auto lhs, auto rhs) { return lhs < rhs; };
using func_ptr = bool(*)(int, int);
func_ptr sorter_func = sorter;  // deduces int, int
// note however that the following is ambiguous
// func_ptr sorter_func2 = +sorter;

```



## Using lambdas for inline parameter pack unpacking



Parameter pack unpacking traditionally requires writing a helper function for each time you want to do it.

> 传统上，参数包拆包需要在每次需要时编写一个辅助函数。

In this toy example:

```cpp
template<std::size_t...Is>
void print_indexes( std::index_sequence<Is...> ) {
  using discard=int[];
  (void)discard{0,((void)(
    std::cout << Is << '\n' // here Is is a compile-time constant.
  ),0)...};
}
template<std::size_t I>
void print_indexes_upto() {
  return print_indexes( std::make_index_sequence<I>{} );
}

```


The `print_indexes_upto` wants to create and unpack a parameter pack of indexes.  In order to do so, it must call a helper function.  Every time you want to unpack a parameter pack you created, you end up having to create a custom helper function to do it.

> “print_indexes_upto”想要创建和解压缩一个索引参数包。为了做到这一点，它必须调用一个助手函数。每次您想要解压缩创建的参数包时，最终都必须创建一个自定义的辅助函数来完成。

This can be avoided with lambdas.


You can unpack parameter packs into a set of invocations of a lambda, like this:

> 您可以将参数包解压到lambda的一组调用中，如下所示：

```cpp
template<std::size_t I>
using index_t = std::integral_constant<std::size_t, I>;
template<std::size_t I>
constexpr index_t<I> index{};

template<class=void, std::size_t...Is>
auto index_over( std::index_sequence<Is...> ) {
  return [](auto&& f){
    using discard=int[];
    (void)discard{0,(void(
      f( index<Is> )
    ),0)...};
  };
}

template<std::size_t N>
auto index_over(index_t<N> = {}) {
  return index_over( std::make_index_sequence<N>{} );
}

```

With fold expressions, `index_over()` can be simplified to:

```cpp
template<class=void, std::size_t...Is>
auto index_over( std::index_sequence<Is...> ) {
  return [](auto&& f){
    ((void)(f(index<Is>)), ...);
  };
}

```


Once you have done that, you can use this to replace having to manually unpack parameter packs with a second overload in other code, letting you unpack parameter packs "inline":

> 一旦您完成了这一操作，您就可以使用它来代替在其他代码中使用第二个重载手动解压缩参数包，从而使您可以“内联”地解压缩参数包：

```cpp
template<class Tup, class F>
void for_each_tuple_element(Tup&& tup, F&& f) {
  using T = std::remove_reference_t<Tup>;
  using std::tuple_size;
  auto from_zero_to_N = index_over< tuple_size<T>{} >();

  from_zero_to_N(
    [&](auto i){
      using std::get;
      f( get<i>( std::forward<Tup>(tup) ) );
    }
  );
}

```


The `auto i` passed to the lambda by the `index_over` is a `std::integral_constant<std::size_t, ???>`.  This has a `constexpr` conversion to `std::size_t` that does not depend on the state of `this`, so we can use it as a compile-time constant, such as when we pass it to `std::get<i>` above.

> “index_over”传递给lambda的“auto i”是“std:：integral_content<std:：size_t，？？？>”。这有一个到“std:：size_t”的“constexpr”转换，它不依赖于“This”的状态，所以我们可以将其用作编译时常数，例如当我们将其传递给上面的“std::get＜i＞”时。

To go back to the toy example at the top, rewrite it as:

```cpp
template<std::size_t I>
void print_indexes_upto() {
  index_over(index<I>)([](auto i){
    std::cout << i << '\n'; // here i is a compile-time constant
  });
}

```

which is much shorter, and keeps logic in the code that uses it.


[Live example](http://coliru.stacked-crooked.com/a/32c204301f7163c9) to play with.

> [现场示例](http://coliru.stacked-crooked.com/a/32c204301f7163c9)玩。



## Porting lambda functions to C++03 using functors



Lambda functions in C++ are syntactic sugar that provide a very concise syntax for writing [functors](http://stackoverflow.com/documentation/c%2B%2B/1412/class-functors#t=201607221029133733631). As such, equivalent functionality can be obtained in C++03 (albeit much more verbose) by converting the lambda function into a functor:

> C++中的Lambda函数是语法糖，为编写[functors]提供了非常简洁的语法(http://stackoverflow.com/documentation/c%2B%2B/1412/class-函子#t=201607221029133733631）。因此，通过将lambda函数转换为函子，可以在C++03中获得等效的功能（尽管要详细得多）：

```cpp
// Some dummy types:
struct T1 {int dummy;};
struct T2 {int dummy;};
struct R {int dummy;};

// Code using a lambda function (requires C++11)
R use_lambda(T1 val, T2 ref) {
  // Use auto because the type of the lambda is unknown.
  auto lambda = [val, &ref](int arg1, int arg2) -> R {
    /* lambda-body */
    return R();
  };
  return lambda(12, 27);
}

// The functor class (valid C++03)
// Similar to what the compiler generates for the lambda function.
class Functor {
  // Capture list.
  T1 val;
  T2& ref;

public:
  // Constructor
  inline Functor(T1 val, T2& ref) : val(val), ref(ref) {}

  // Functor body
  R operator()(int arg1, int arg2) const {
    /* lambda-body */
    return R();
  }
};

// Equivalent to use_lambda, but uses a functor (valid C++03).
R use_functor(T1 val, T2 ref) {
  Functor functor(val, ref);
  return functor(12, 27);
}

// Make this a self-contained example.
int main() {
  T1 t1;
  T2 t2;
  use_functor(t1,t2);
  use_lambda(t1,t2);
  return 0;
}

```


If the lambda function is `mutable` then make the functor's call-operator non-const, i.e.:

> 如果lambda函数是“可变的”，则使函数的调用运算符为非常量，即：

```cpp
R operator()(int arg1, int arg2) /*non-const*/ {
  /* lambda-body */
  return R();
}

```



#### Syntax


- [**default-capture**, **capture-list**] (**argument-list**) mutable **throw-specification** **attributes** -> **return-type** { **lambda-body** } // Order of lambda specifiers and attributes.
- [**capture-list**] (**argument-list**) { **lambda-body** } // Common lambda definition.
- [=] (**argument-list**) { **lambda-body** } // Captures all needed local variables by value.
- [&] (**argument-list**) { **lambda-body** } // Captures all needed local variables by reference.
- [**capture-list**] { **lambda-body** } // Argument list and specifiers can be omitted.



#### Parameters


|Parameter|Details
|---|---|---|---|---|---|---|---|---|---
|**default-capture**|Specifies how all non-listed variables are captured. Can be `=` (capture by value) or `&` (capture by reference). If omitted, non-listed variables are inaccessible within the **lambda-body**. The **default-capture** must precede the **capture-list**.
|**capture-list**|Specifies how local variables are made accessible within the **lambda-body**. Variables without prefix are captured by value. Variables prefixed with `&` are captured by reference. Within a class method, `this` can be used to make all its members accessible by reference. Non-listed variables are inaccessible, unless the list is preceded by a **default-capture**.
|**argument-list**|Specifies the arguments of the lambda function.
|mutable|**(optional)** Normally variables captured by value are `const`. Specifying `mutable` makes them non-const. Changes to those variables are retained between calls.
|**throw-specification**|**(optional)** Specifies the exception throwing behavior of the lambda function. For example: `noexcept` or `throw(std::exception)`.
|**attributes**|**(optional)** Any attributes for the lambda function. For example, if the **lambda-body** always throws an exception then `[[noreturn]]` can be used.
|-> **return-type**|**(optional)** Specifies the return type of the lambda function. Required when the return type cannot be determined by the compiler.
|**lambda-body**|A code block containing the implementation of the lambda function.



#### Remarks



C++17 (the current draft) introduces `constexpr` lambdas, basically lambdas that can be evaluated at compile time. A lambda is automatically `constexpr` if it satisfies `constexpr` requirements, but you can also specify it using the `constexpr` keyword:

> C++17（当前的草案）引入了“constexpr”lambdas，基本上是可以在编译时计算的lambdas。如果lambda满足“constexpr”要求，则它会自动成为“constexpr”，但您也可以使用“constexper”关键字指定它：

```cpp
//Explicitly define this lambdas as constexpr
[]() constexpr {
    //Do stuff
}

```

