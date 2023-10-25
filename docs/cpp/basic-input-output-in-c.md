---
tip: translate by baidu@2023-10-25 08:20:45
---
---

metaTitle: "C++ | Basic input/output in c++"
description: "user input and standard output"
---------------------------------------------

# Basic input/output in c++

## user input and standard output

```cpp
#include <iostream>

int main()
{
    int value;
    std::cout << "Enter a value: " << std::endl;
    std::cin >> value;
    std::cout << "The square of entered value is: " << value * value << std::endl;
    return 0;
}

```

#### Remarks

The standard library `<iostream>` defines few streams for input and output:

> 标准库“＜iostream＞”为输入和输出定义了几个流：

```cpp
|stream | description                      |
|---|---|---|---|---|---|---|---|---|----|---|---|---|---|---|---|---|---|---|-------------------------------|
|cin    | standard input stream            |
|cout   | standard output stream           |
|cerr   | standard error (output) stream   |
|clog   | standard logging (output) stream |

```

Out of four streams mentioned above `cin` is basically used for user input and other three are used for outputting the data. In general or in most coding environments `cin` (**console input** or standard input) is keyboard and `cout` (**console output** or standard output) is monitor.

> 在上述四个流中，“cin”基本上用于用户输入，其他三个流用于输出数据。在一般情况下或在大多数编码环境中，“cin”（**控制台输入**或标准输入）是键盘，“cout”（***控制台输出**或标准输出）是监视器。

```cpp
cin >> value

cin   - input stream
'>>'  - extraction operator
value - variable (destination)

```

`cin` here extracts the input entered by the user and feeds in variable value. The value is extracted only after user presses ENTER key.

```cpp
cout << "Enter a value: "

cout              - output stream
'<<'              - insertion operator
"Enter a value: " - string to be displayed

```

`cout` here takes the string to be displayed and inserts it to standard output or monitor

All four streams are located in standard namespace `std` so we need to print `std::stream` for stream `stream` to use it.

> 所有四个流都位于标准命名空间“std”中，因此我们需要为流“stream”打印“std:：stream”才能使用它。

There is also a manipulator `std::endl` in code. It can be used only with output streams. It inserts end of line `'\n'` character in the stream and flushes it. It causes immediately producing output.

> 代码中还有一个操纵器“std:：endl”。它只能与输出流一起使用。它在流中插入行尾字符“”并刷新它。它会立即产生输出。
