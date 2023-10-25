---
tip: translate by baidu@2023-10-25 08:25:07
---
---
metaTitle: "C++ | Fold Expressions"
description: "Unary Folds, Binary Folds, Folding over a comma"
---

# Fold Expressions



## Unary Folds



Unary folds are used to **fold** [parameter packs](http://stackoverflow.com/documentation/c%2B%2B/7668/parameter-packs#t=201610282037074790782) over a specific operator. There are 2 kinds of unary folds:

> 一元折叠用于**折叠**[参数包](http://stackoverflow.com/documentation/c%2B%2B/7668/parameter-packs#t=201610282037074790782）。一元折叠有两种：

<li>
Unary **Left** Fold  `(... op pack)` which expands as follows:

```cpp
((Pack1 op Pack2) op ...) op PackN

```


</li>
<li>
Unary **Right** Fold  `(pack op ...)` which expands as follows:

```cpp
Pack1 op (... (Pack(N-1) op PackN)) 

```


</li>

Here is an example

```cpp
template<typename... Ts>
int sum(Ts... args)
{
    return (... + args); //Unary left fold
    //return (args + ...); //Unary right fold

    // The two are equivalent if the operator is associative.
    // For +, ((1+2)+3) (left fold) == (1+(2+3)) (right fold)
    // For -, ((1-2)-3) (left fold) != (1-(2-3)) (right fold)
}

int result = sum(1, 2, 3); //  6

```



## Binary Folds



Binary folds are basically [unary folds](https://stackoverflow.com/documentation/c%2b%2b/2676/fold-expressions/8931/unary-folds#t=201608161845476575023), with an extra argument.

> 二元折叠基本上是[一元折叠](https://stackoverflow.com/documentation/c%2b%2b/2676/fold-expressions/8931/一元折叠#t=201608161845476575023），带有一个额外的参数。

There are 2 kinds of binary folds:

<li>
Binary **Left** Fold - `(value op ... op pack)` - Expands as follows:

```cpp
(((Value op Pack1) op Pack2) op ...) op PackN

```


</li>
<li>
Binary **Right** Fold `(pack op ... op value)` - Expands as follows:

```cpp
Pack1 op (... op (Pack(N-1) op (PackN op Value)))

```


</li>

Here is an example:

```cpp
template<typename... Ts>
int removeFrom(int num, Ts... args)
{
    return (num - ... - args); //Binary left fold
    // Note that a binary right fold cannot be used
    // due to the lack of associativity of operator-
}

int result = removeFrom(1000, 5, 10, 15); //'result' is 1000 - 5 - 10 - 15 = 970

```



## Folding over a comma



It is a common operation to need to perform a particular function over each element in a parameter pack. With C++11, the best we can do is:

> 需要对参数包中的每个元素执行特定功能是一种常见的操作。使用C++11，我们能做的最好的事情是：

```cpp
template <class... Ts>
void print_all(std::ostream& os, Ts const&... args) {
    using expander = int[];
    (void)expander{0,
        (void(os << args), 0)...
    };
}

```

But with a fold expression, the above simplifies nicely to:

```cpp
template <class... Ts>
void print_all(std::ostream& os, Ts const&... args) {
    (void(os << args), ...);
}

```

No cryptic boilerplate required.



#### Remarks


Fold Expressions are supported for the following operators

| | | | | | | | | | | | 
|---|---|---|---|---|---|---|---|---|---
|+|-|*|/|%|\ˆ|&|||<<|>>||
|+=|-=|*=|/=|%=|\ˆ=|&=||=|<<=|>>=|=|
|==|!=|<|>|<=|>=|&&||||,|.*|->*|


When folding over an empty sequence, a fold expression is ill-formed, except for the following three operators:

> 在空序列上折叠时，除了以下三个运算符外，折叠表达式的格式是错误的：

|Operator|Value when parameter pack is empty
|---|---|---|---|---|---|---|---|---|---
|&&|true
||||false
|,|void()

