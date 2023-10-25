---
tip: translate by baidu@2023-10-25 08:30:28
---
---
metaTitle: "C++ | References"
description: "Defining a reference, C++ References are Alias of existing variables"
---

# References



## Defining a reference



References behaves similarly, but not entirely like const pointers. A reference is defined by suffixing an ampersand `&` to a type name.

> 引用的行为类似，但并不完全像常量指针。引用是通过在类型名称后面加上“与”和“&”来定义的。

```cpp
int i = 10;
int &refi = i;

```

Here, `refi` is a reference bound to `i`.<br />

References abstracts the semantics of pointers, acting like an alias to the underlying object:

> 引用抽象了指针的语义，就像底层对象的别名一样：

```cpp
refi = 20; // i = 20;

```

You can also define multiple references in a single definition:

```cpp
int i = 10, j = 20;
int &refi = i, &refj = j;

// Common pitfall :
// int& refi = i, k = j;
// refi will be of type int&.
// though, k will be of type int, not int&!

```


References **must** be initialized correctly at the time of definition, and cannot be modified afterwards. The following piece of codes causes a compile error:

> 引用**在定义时必须**正确初始化，之后不能修改。以下代码导致编译错误：

```cpp
int &i; // error: declaration of reference variable 'i' requires an initializer

```

You also cannot bind directly a reference to `nullptr`, unlike pointers:

```cpp
int *const ptri = nullptr;
int &refi = nullptr; // error: non-const lvalue reference to type 'int' cannot bind to a temporary of type 'nullptr_t'

```



## C++ References are Alias of existing variables



A Reference in C++ is just an `Alias` or another name of a variable. Just like most of us can be referred using our passport name and nick name.

> C++中的引用只是一个“别名”或变量的另一个名称。就像我们大多数人可以用护照名和昵称来推荐一样。


References doesn't exist literally and they don't occupy any memory. If we print the address of reference variable it will print the same address as that of the variable its referring to.

> 引用并不存在，它们也不占用任何内存。如果我们打印引用变量的地址，它将打印与其引用的变量的地址相同的地址。

```cpp
int main() {
    int i = 10;
    int &j = i;
    
    cout<<&i<<endl;
    cout<<&b<<endl;
    return 0;
}

```


In the above example, both `cout` will print the same address. The situation will be same if we take a variable as a reference in a function

> 在上面的例子中，两个“cout”将打印相同的地址。如果我们把一个变量作为函数中的引用，情况也是一样的

```cpp
void func (int &fParam ) {
   cout<<"Address inside function => "<<fParam<<endl;
}

int main() {
    int i = 10;
    cout<<"Address inside Main => "<<&i<<endl;    

    func(i);

    return 0;
}


```

In this example also, both `cout` will print the same address.


As we know by now that `C++ References` are just alias, and for an alias to be created, we need to have something which the Alias can refer to.

> 正如我们现在所知，“C++引用”只是别名，要创建别名，我们需要有别名可以引用的东西。


That's the precise reason why the statement like this will throw a compiler error

> 这就是为什么像这样的语句会引发编译器错误的确切原因

```cpp
int &i;


```

Because, the alias is not referring to anything.

