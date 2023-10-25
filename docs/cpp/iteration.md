---
tip: translate by baidu@2023-10-25 08:26:13
---
---
metaTitle: "C++ | Iteration"
description: "break, continue, do, for, while, range-based for loop"
---

# Iteration



## break


Jumps out of the nearest enclosing loop or `switch` statement.

```cpp
// print the numbers to a file, one per line
for (const int num : num_list) {
    errno = 0;
    fprintf(file, "%d\n", num);
    if (errno == ENOSPC) {
        fprintf(stderr, "no space left on device; output will be truncated\n");
        break;
    }
}

```



## continue


Jumps to the end of the smallest enclosing loop.

```cpp
int sum = 0;
for (int i = 0; i < N; i++) {
    int x;
    std::cin >> x;
    if (x < 0) continue;
    sum += x;
    // equivalent to: if (x >= 0) sum += x;
}

```



## do



Introduces a [do-while loop](http://stackoverflow.com/documentation/c%2b%2b/589/loops/7821/do-while-loop).

> 引入[do-while循环](http://stackoverflow.com/documentation/c%2b%2b/589/loops/7821/do-while循环）。

```cpp
// Gets the next non-whitespace character from standard input
char read_char() {
    char c;
    do {
        c = getchar();
    } while (isspace(c));
    return c;
}

```



## for



Introduces a [for loop](http://stackoverflow.com/documentation/c%2b%2b/589/loops/1945/for-loop) or, in C++11 and later, a [range-based for loop](http://stackoverflow.com/documentation/c%2b%2b/589/loops/1926/range-based-for).

> 引入[for循环](http://stackoverflow.com/documentation/c%2b%2b/589/loops/1945/for-循环），或者在C++11及更高版本中，[基于循环的范围](http://stackoverflow.com/documentation/c%2b%2b/589/loops/1926/range-基于）。

```cpp
// print 10 asterisks
for (int i = 0; i < 10; i++) {
    putchar('*');
}

```



## while



Introduces a [while loop](http://stackoverflow.com/documentation/c%2B%2B/589/loops/3301/while-loop#t=201611112017446296865).

> 引入[while循环](http://stackoverflow.com/documentation/c%2B%2B/589/loops/3301/while-循环#t=201611112017446296865）。

```cpp
int i = 0;
// print 10 asterisks
while (i < 10) {
    putchar('*');
    i++;
}

```



## range-based for loop


```cpp
std::vector<int> primes = {2, 3, 5, 7, 11, 13};

for(auto prime : primes) {
    std::cout << prime << std::endl;
}

```

