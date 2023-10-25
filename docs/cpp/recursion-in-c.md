---
tip: translate by baidu@2023-10-25 08:30:21
---
---
metaTitle: "C++ | Recursion in C++"
description: "Using tail recursion and Fibonnaci-style recursion to solve the Fibonnaci sequence, Recursion with memoization"
---

# Recursion in C++



## Using tail recursion and Fibonnaci-style recursion to solve the Fibonnaci sequence



The simple and most obvious way to use recursion to get the Nth term of the Fibonnaci sequence is this

> 使用递归来获得Fibonnaci序列的第N项的最简单、最明显的方法是

```cpp
int get_term_fib(int n)
{
  if (n == 0)
    return 0;
  if (n == 1)
    return 1;
  return get_term_fib(n - 1) + get_term_fib(n - 2);
}

```


However, this algorithm does not scale for higher terms: for bigger and bigger `n`, the number of function calls that you need to make grows exponentially. This can be replaced with a simple tail recursion.

> 然而，这种算法不能扩展到更高的项：对于越来越大的“n”，需要进行的函数调用数量呈指数级增长。这可以用一个简单的尾部递归来代替。

```cpp
int get_term_fib(int n, int prev = 0, int curr = 1)
{
  if (n == 0)
    return prev;
  if (n == 1)
    return curr;
  return get_term_fib(n - 1, curr, prev + curr);
}

```


Each call to the function now immediately calculates the next term in the Fibonnaci sequence, so the number of function calls scales linearly with `n`.

> 现在，对函数的每次调用都会立即计算Fibonnaci序列中的下一个项，因此函数调用的数量与“n”成线性关系。



## Recursion with memoization



Recursive functions can get quite expensive. If they are pure functions (functions that always return the same value when called with the same arguments, and that neither depend on nor modify external state), they can be made considerably faster at the expense of memory by storing the values already calculated.

> 递归函数可能会变得相当昂贵。如果它们是纯函数（当用相同的参数调用时，总是返回相同的值，并且既不依赖也不修改外部状态的函数），那么通过存储已经计算的值，它们可以以牺牲内存为代价变得更快。


The following is an implementation of the Fibonacci sequence with memoization:

> 以下是具有记忆功能的斐波那契序列的实现：

```cpp
#include <map>

int fibonacci(int n)
{
  static std::map<int, int> values;
  if (n==0 || n==1)
    return n;
  std::map<int,int>::iterator iter = values.find(n);
  if (iter == values.end())
  {
    return values[n] = fibonacci(n-1) + fibonacci(n-2);
  }
  else
  {
    return iter->second;
  }
}

```


Note that despite using the simple recursion formula, on first call this function is $O(n)$. On subsequent calls with the same value, it is of course $O(1)$.

> 注意，尽管使用了简单的递归公式，但在第一次调用时，该函数是$O（n）$。在具有相同值的后续调用中，它当然是$O（1）$。


Note however that this implementation is not reentrant. Also, it doesn't allow to get rid of stored values. An alternative implementation would be to allow the map to be passed as additional argument:

> 但是请注意，此实现是不可重入的。此外，它不允许删除存储的值。另一种实现方式是允许将映射作为附加参数传递：

```cpp
#include <map>

int fibonacci(int n, std::map<int, int> values)
{
  if (n==0 || n==1)
    return n;
  std::map<int,int>::iterator iter = values.find(n);
  if (iter == values.end())
  {
    return values[n] = fibonacci(n-1) + fibonacci(n-2);
  }
  else
  {
    return iter->second;
  }
}

```


For this version, the caller is required to maintain the map with the stored values. This has the advantage that the function is now reentrant, and that the caller can remove values that are no longer needed, saving memory. It has the disadvantage that it breaks encapsulation; the caller can change the output by populating the map with incorrect values.

> 对于这个版本，调用程序需要使用存储的值来维护映射。这样做的优点是，函数现在是可重入的，调用方可以删除不再需要的值，从而节省内存。它的缺点是破坏了封装；调用方可以通过使用不正确的值填充映射来更改输出。

