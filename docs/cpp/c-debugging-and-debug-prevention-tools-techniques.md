---
tip: translate by baidu@2023-10-25 08:21:34
---
---

metaTitle: "C++ | C++ Debugging and Debug-prevention Tools & Techniques"
description: "Segfault analysis with GDB, Static analysis, Clean code, My C++ program ends with segfault - valgrind, Safe-stack (Stack corruptions)"
----------------------------------------------------------------------------------------------------------------------------------------------------

# C++ Debugging and Debug-prevention Tools & Techniques

A lot of time from C++ developers is spent debugging. This topic is meant to assist with this task and give inspiration for techniques. Don't expect an extensive list of issues and solutions fixed by the tools or a manual on the mentioned tools.

> C++ 开发人员花了大量时间进行调试。本主题旨在帮助完成这项任务，并为技术提供灵感。不要指望工具会列出大量问题和解决方案，也不要指望上面提到的工具会有手册。

## Segfault analysis with GDB

Lets use the same code as above for this example.

```cpp
#include <iostream>

void fail() {
    int *p1;
    int *p2(NULL);
    int *p3 = p1;
    if (p3) {
        std::cout << *p2 << std::endl;
    } 
}

int main() { 
    fail();
}

```

First lets compile it

```cpp
g++ -g -o main main.cpp

```

Lets run it with gdb

```cpp
gdb ./main

```

Now we will be in gdb shell. Type run.

```cpp
(gdb) run
The program being debugged has been started already.
Start it from the beginning? (y or n) y
Starting program: /home/opencog/code-snippets/stackoverflow/a.out 

Program received signal SIGSEGV, Segmentation fault.
0x0000000000400850 in fail () at debugging_with_gdb.cc:11
11            std::cout << *p2 << std::endl;

```

We see the segmentation fault is happening at line 11. So the only variable being used at this line is pointer p2. Lets examine its content typing print.

> 我们看到分段故障发生在第 11 行。因此，这一行使用的唯一变量是指针 p2。让我们检查其内容键入打印。

```cpp
(gdb) print p2
$1 = (int *) 0x0 

```

Now we see that p2 was initialized to 0x0 which stands for NULL. At this line, we know that we are trying to dereference a NULL pointer. So we go and fix it.

> 现在我们看到 p2 被初始化为 0x0，0x0 代表 NULL。在这一行，我们知道我们正在尝试取消引用一个 NULL 指针。所以我们去解决它。

## Static analysis

Static analysis is the technique in which on checks the code for patterns linked to known bugs. Using this technique is less time consuming than a code review, though, its checks are only limited to those programmed in the tool.

> 静态分析是一种 on 检查代码中与已知错误相关的模式的技术。使用这种技术比代码审查耗时更少，不过，它的检查仅限于工具中编程的检查。

Checks can include the incorrect semi-colon behind the if-statement (`if (var);`) till advanced graph algorithms which determine if a variable is not initialized.

> 检查可能在 if 语句后面包含不正确的分号（`if（var）；`）直到确定变量是否未初始化的高级图算法。

### Compiler warnings

Enabling static analysis is easy, the most simplistic version is already build-in in your compiler:

> 启用静态分析很容易，编译器中已经内置了最简单的版本：

- [`clang++ -Wall -Weverything -Werror ...`](https://clang.llvm.org/docs/DiagnosticsReference.html)
- [`g++ -Wall -Weverything -Werror ...`](https://gcc.gnu.org/onlinedocs/gcc/Warning-Options.html)
- [`cl.exe /W4 /WX ...`](https://docs.microsoft.com/en-us/cpp/build/reference/compiler-option-warning-level)

If you enable these options, you will notice that each compiler will find bugs the others don't and that you will get errors on techniques which might be valid or valid in a specific context. `while (staticAtomicBool);` might be acceptable even if `while (localBool);` ain't.

> 如果您启用这些选项，您会注意到每个编译器都会发现其他编译器没有的错误，并且您会在特定上下文中发现有效或有效的技术错误 `while（staticTomicBool）；` 即使“while（localBool）；”不是。

So unlike code review, you are fighting a tool which understands your code, tells you a lot of useful bugs and sometimes disagrees with you. In this last case, you might have to suppress the warning locally.

> 因此，与代码审查不同的是，你正在与一个理解你的代码的工具作战，告诉你很多有用的 bug，有时会与你意见相左。在最后一种情况下，您可能必须在本地取消显示警告。

As the options above enable all warnings, they might enable warnings you don't want. (Why should your code be C++98 compatible?) If so, you can simply disable that specific warning:

> 由于上面的选项启用了所有警告，它们可能会启用您不想要的警告。（为什么你的代码应该与 C++98 兼容？）如果是这样，你可以简单地禁用特定的警告：

- `clang++ -Wall -Weverything -Werror -Wno-errortoaccept ...`
- `g++ -Wall -Weverything -Werror -Wno-errortoaccept ...`
- `cl.exe /W4 /WX /wd<no of warning>...`

Where compiler warnings assist you during development, they slow down compilation quite a bit. That is why you might not always want to enable them by default. Either you run them by default or you enable some continuous integration with the more expensive checks (or all of them).

> 当编译器警告在开发过程中对您有帮助时，它们会大大降低编译速度。这就是为什么您可能并不总是希望在默认情况下启用它们。要么默认运行它们，要么启用一些与更昂贵的检查（或所有检查）的连续集成。

### External tools

If you decide to have some continuous integration, the use of other tools ain't such a stretch. A tool like [clang-tidy](http://clang.llvm.org/extra/clang-tidy/) has an [list of checks](http://clang.llvm.org/extra/clang-tidy/checks/list.html) which covers a wide range of issues, some examples:

> 如果你决定进行一些持续的集成，那么使用其他工具就不那么容易了。像[叮当作响]这样的工具([http://clang.llvm.org/extra/clang-tidy/](http://clang.llvm.org/extra/clang-tidy/))有[支票清单](http://clang.llvm.org/extra/clang-tidy/checks/list.html)它涵盖了广泛的问题，例如：

<li>Actual bugs
<ul>
- Prevention of slicing
- Asserts with side effects

- Misleading indentation
- Check identifier naming
- Use make_unique()
- Use nullptr
- Find unneeded copies
- Find inefficient algorithm calls

The list might not be that large, as Clang already has a lot of compiler warnings, however it will bring you one step closer to a high quality code base.

> 这个列表可能没有那么大，因为 Clang 已经有很多编译器警告，但它将使您离高质量的代码库更近一步。

### Other tools

Other tools with similar purpose exist, like:

- [the visual studio static analyzer](https://blogs.msdn.microsoft.com/hkamel/2013/10/24/visual-studio-2013-static-code-analysis-in-depth-what-when-and-how/) as external tool
- [clazy](https://blogs.kde.org/2015/11/15/new-cqt-code-checks-clazy-static-analyzer), a Clang compiler plugin for checking Qt code

### Conclusion

A lot static analysis tools exist for C++, both build-in in the compiler as external tools. Trying them out doesn't take that much time for easy setups and they will find bugs you might miss in code review.

> C++ 有很多静态分析工具，它们都是作为外部工具内置在编译器中的。尝试它们不会花那么多时间来进行简单的设置，而且它们会发现你在代码审查中可能会错过的错误。

## Clean code

Debugging starts with understanding the code you are trying to debug.

Bad code:

```cpp
int main() {
    int value;
    std::vector<int> vectorToSort;
    vectorToSort.push_back(42); vectorToSort.push_back(13);
    for (int i = 52; i; i = i - 1)
    {
    vectorToSort.push_back(i *2);
    }
    /// Optimized for sorting small vectors
    if (vectorToSort.size() == 1);
    else
        {
        if (vectorToSort.size() <= 2)
            std::sort(vectorToSort.begin(), std::end(vectorToSort));
        }
    for (value : vectorToSort) std::cout << value << ' ';
return 0; }

```

Better code:

```cpp
std::vector<int> createSemiRandomData() {
    std::vector<int> data;
    data.push_back(42);
    data.push_back(13);
    for (int i = 52; i; --i)
        vectorToSort.push_back(i *2);
    return data;
}

/// Optimized for sorting small vectors
void sortVector(std::vector &v) {
    if (vectorToSort.size() == 1)
        return;
    if (vectorToSort.size() > 2)
        return;

    std::sort(vectorToSort.begin(), vectorToSort.end());
}

void printVector(const std::vector<int> &v) {
    for (auto i : v)
        std::cout << i << ' ';
}

int main() {
    auto vectorToSort = createSemiRandomData();
    sortVector(std::ref(vectorToSort));
    printVector(vectorToSort);

    return 0;
 }

```

Regardless of the coding styles you prefer and use, having a consistent coding (and formatting) style will help you understanding the code.

> 无论您喜欢和使用哪种编码样式，具有一致的编码（和格式）样式都将有助于您理解代码。

Looking at the code above, one can identify a couple of improvements to improve readability and debuggability:

> 查看上面的代码，可以发现一些改进，以提高可读性和可调试性：

### The use of separate functions for separate actions

The use of separate functions allow you to skip over some functions in the debugger if you ain't interested in the details. In this specific case, you might not be interested in the creation or printing of the data and only want to step into the sorting.

> 如果您对细节不感兴趣，使用单独的函数可以跳过调试器中的一些函数。在这种特定情况下，您可能对数据的创建或打印不感兴趣，只想进行排序。

Another advantage is that you need to read less code (and memorize it) while stepping through the code. You now only need to read 3 lines of code in `main()` in order to understand it, instead of the whole function.

> 另一个优点是，在遍历代码时，您需要阅读更少的代码（并记住它）。您现在只需要阅读“main（）”中的 3 行代码就可以理解它，而不需要阅读整个函数。

The third advantage is that you simply have less code to look at, which helps a trained eye in spotting this bug within seconds.

> 第三个优点是，你只需要看更少的代码，这有助于训练有素的眼睛在几秒钟内发现这个错误。

### Using consistent formatting/constructions

The use of consistent formatting and constructions will remove clutter from the code making it easier to focus on the code instead of text. A lot of discussions have been fed on the 'right' formatting style. Regardless of that style, having a single consistent style in the code will improve familiarity and make it easier to focus on the code.

> 使用一致的格式和构造将消除代码中的混乱，从而更容易将注意力集中在代码而不是文本上。关于“正确”的格式样式，已经进行了大量讨论。不管这种风格如何，在代码中拥有一个一致的风格将提高熟悉度，并使其更容易专注于代码。

As formatting code is time consuming task, it is recommended to use a dedicated tool for this. Most IDEs have at least some kind of support for this and can do formatting more consistent than humans.

> 由于格式化代码是一项耗时的任务，因此建议使用专用工具。大多数 IDE 至少对此有某种支持，并且可以比人类更一致地进行格式化。

You might note that the style is not limited to spaces and newlines as we no longer mix the free-style and the member functions to get begin/end of the container. (`v.begin()` vs `std::end(v)`).

> 您可能会注意到，样式不限于空格和换行符，因为我们不再混合使用自由样式和成员函数来获取容器的开始/结束。（`v.begin（）` vs `std:：end（v）`）。

### Point attention to the important parts of your code.

Regardless of the style you determine to choose, the above code contains a couple of markers which might give you a hint on what might be important:

> 无论您决定选择哪种风格，上面的代码都包含一些标记，这些标记可能会提示您哪些可能是重要的：

- A comment stating `optimized`, this indicates some fancy techniques
- Some early returns in `sortVector()` indicate that we are doing something special
- The `std::ref()` indicates that something is going on with the `sortVector()`

### Conclusion

Having clean code will help you understanding the code and will reduce the time you need to debug it. In the second example, a code reviewer might even spot the bug at first glance, while the bug might be hidden in the details in the first one. (PS: The bug is in the compare with `2`.)

> 拥有干净的代码将有助于您理解代码，并减少调试所需的时间。在第二个示例中，代码审查人员甚至可能第一眼就发现错误，而错误可能隐藏在第一个示例的细节中。（附言：这个错误正在与“2”进行比较。）

## My C++ program ends with segfault - valgrind

Let's have a basic failing program:

```cpp
#include <iostream>

void fail() {
    int *p1;
    int *p2(NULL);
    int *p3 = p1;
    if (p3) {
        std::cout << *p3 << std::endl;
    } 
}

int main() { 
    fail();
}

```

Build it (add -g to include debug info):

```cpp
g++ -g -o main main.cpp

```

Run:

```cpp
$ ./main
Segmentation fault (core dumped)
$

```

Let's debug it with valgrind:

```cpp
$ valgrind ./main
==8515== Memcheck, a memory error detector
==8515== Copyright (C) 2002-2015, and GNU GPL'd, by Julian Seward et al.
==8515== Using Valgrind-3.11.0 and LibVEX; rerun with -h for copyright info
==8515== Command: ./main
==8515==
==8515== Conditional jump or move depends on uninitialised value(s)
==8515==    at 0x400813: fail() (main.cpp:7)
==8515==    by 0x40083F: main (main.cpp:13)
==8515==
==8515== Invalid read of size 4
==8515==    at 0x400819: fail() (main.cpp:8)
==8515==    by 0x40083F: main (main.cpp:13)
==8515==  Address 0x0 is not stack'd, malloc'd or (recently) free'd
==8515==
==8515==
==8515== Process terminating with default action of signal 11 (SIGSEGV): dumping core
==8515==  Access not within mapped region at address 0x0
==8515==    at 0x400819: fail() (main.cpp:8)
==8515==    by 0x40083F: main (main.cpp:13)
==8515==  If you believe this happened as a result of a stack
==8515==  overflow in your program's main thread (unlikely but
==8515==  possible), you can try to increase the size of the
==8515==  main thread stack using the --main-stacksize= flag.
==8515==  The main thread stack size used in this run was 8388608.
==8515==
==8515== HEAP SUMMARY:
==8515==     in use at exit: 72,704 bytes in 1 blocks
==8515==   total heap usage: 1 allocs, 0 frees, 72,704 bytes allocated
==8515==
==8515== LEAK SUMMARY:
==8515==    definitely lost: 0 bytes in 0 blocks
==8515==    indirectly lost: 0 bytes in 0 blocks
==8515==      possibly lost: 0 bytes in 0 blocks
==8515==    still reachable: 72,704 bytes in 1 blocks
==8515==         suppressed: 0 bytes in 0 blocks
==8515== Rerun with --leak-check=full to see details of leaked memory
==8515==
==8515== For counts of detected and suppressed errors, rerun with: -v
==8515== Use --track-origins=yes to see where uninitialised values come from
==8515== ERROR SUMMARY: 2 errors from 2 contexts (suppressed: 0 from 0)
$

```

First we focus on this block:

```cpp
==8515== Invalid read of size 4
==8515==    at 0x400819: fail() (main.cpp:8)
==8515==    by 0x40083F: main (main.cpp:13)
==8515==  Address 0x0 is not stack'd, malloc'd or (recently) free'd

```

The first line tells us that segfault is caused by reading 4 bytes. The second and the third lines are call stack. It means that the invalid read is performed at the `fail()` function, line 8 of main.cpp, which is called by main, line 13 of main.cpp.

> 第一行告诉我们 segfault 是由读取 4 个字节引起的。第二行和第三行是调用堆栈。这意味着无效读取是在 main.cpp 第 8 行的“fail（）”函数处执行的，该函数由 main.cpp 的第 13 行的 main 调用。

Looking at line 8 of main.cpp we see

```cpp
std::cout << *p3 << std::endl;

```

But we check the pointer first, so what's wrong? Lets check the other block:

> 但是我们先检查指针，那么怎么了？让我们检查另一个块：

```cpp
==8515== Conditional jump or move depends on uninitialised value(s)
==8515==    at 0x400813: fail() (main.cpp:7)
==8515==    by 0x40083F: main (main.cpp:13)

```

It tells us that there is an unitialized variable at line 7 and we read it:

> 它告诉我们在第 7 行有一个单元化变量，我们读到：

```cpp
if (p3) {

```

Which points us to the line where we check p3 instead of p2. But how is it possible that p3 is uninitialized? We initialize it by:

> 这将我们指向一条线，在这里我们检查 p3 而不是 p2。但是 p3 怎么可能是未初始化的呢？我们通过以下方式初始化它：

```cpp
int *p3 = p1;

```

Valgrind advices us to rerun with `--track-origins=yes`, let's do it:

```cpp
valgrind --track-origins=yes ./main

```

The argument for valgrind is just after valgrind. If we put it after our program, it would be passed to our program.

> 对 valgrind 的争论就在 valgrind 之后。如果我们把它放在我们的程序之后，它就会被传递到我们的程序中。

The output is almost the same, there is only one difference:

```cpp
==8517== Conditional jump or move depends on uninitialised value(s)
==8517==    at 0x400813: fail() (main.cpp:7)
==8517==    by 0x40083F: main (main.cpp:13)
==8517==  Uninitialised value was created by a stack allocation
==8517==    at 0x4007F6: fail() (main.cpp:3)

```

Which tells us that the uninitialized value we used at line 7 was created at line 3:

> 这告诉我们，我们在第 7 行使用的未初始化值是在第 3 行创建的：

```cpp
int *p1;

```

which guides us to our uninitialized pointer.

## Safe-stack (Stack corruptions)

Stack corruptions are annoying bugs to look at. As the stack is corrupted, the debugger often can't give you a good stack trace of where you are and how you got there.

> 堆栈损坏是一个令人讨厌的 bug。由于堆栈已损坏，调试器通常无法为您提供一个良好的堆栈跟踪，以跟踪您所在的位置以及如何到达那里。

This is where safe-stack comes into play. Instead of using a single stack for your threads, it will use two: A safe stack and a dangerous stack. The safe stack works exactly like it did before, except that some parts are moved to the dangerous stack.

> 这就是安全堆栈发挥作用的地方。它将使用两个堆栈：安全堆栈和危险堆栈，而不是为线程使用单个堆栈。安全堆栈的工作方式与以前完全一样，只是有些部件被移到了危险的堆栈中。

### Which parts of the stack get moved?

Every part which has the potential of corrupting the stack will get moved out of the safe stack. As soon as a variable on the stack gets passed by reference or one takes the address of this variable, the compiler will decide to allocate this on the second stack instead of the safe one.

> 每一个有可能损坏堆栈的部分都会被移出安全堆栈。一旦堆栈上的一个变量通过引用传递，或者获取了该变量的地址，编译器就会决定将其分配到第二个堆栈，而不是安全的堆栈。

As a result, any operation you do with those pointers, any modification you make on the memory (based on those pointers/references) can only effect the memory in the second stack. As one never gets a pointer which is close to the safe stack, the stack cannot corrupt the stack and the debugger can still read all functions on the stack to give a nice trace.

> 因此，您对这些指针所做的任何操作，对内存所做的所有修改（基于这些指针/引用）都只能影响第二个堆栈中的内存。由于永远不会得到接近安全堆栈的指针，因此堆栈不会损坏堆栈，调试器仍然可以读取堆栈上的所有函数以提供良好的跟踪。

### What is it actually used for?

The safe stack was not invented to give you better debugging experience, however, it is a nice side effect for nasty bugs. It's original purpose is as part of the [Code-Pointer Integrity (CPI) Project](http://dslab.epfl.ch/proj/cpi/), in which they try to prevent overriding the return addresses to prevent code injection. In other words, they try to prevent executing a hackers code.

> 安全堆栈的发明并不是为了给你更好的调试体验，然而，它对讨厌的 bug 来说是一个很好的副作用。它最初的目的是作为[代码指针完整性（CPI）项目]的一部分(http://dslab.epfl.ch/proj/cpi/)，其中他们试图防止重写返回地址以防止代码注入。换句话说，他们试图阻止执行黑客代码。

For this reason, the feature has been activated on chromium and has been [reported](http://linuxplumbersconf.org/2015/ocw//system/presentations/3261/original/Beyond%20Traditional%20Compilation.pdf) to have a <1% CPU overhead.

> 因此，该功能已在铬上激活，并已[报道](http://linuxplumbersconf.org/2015/ocw//system/presentations/3261/original/Beyond%20Traditional%20Compilation.pdf) CPU 开销小于 1%。

### How to enable it?

Right now, the option is only available in the [clang compiler](https://clang.llvm.org/docs/SafeStack.html), where one can pass `-fsanitize=safe-stack` to the compiler. A [proposal](https://gcc.gnu.org/ml/gcc/2016-04/msg00083.html) was made to implement the same feature in GCC.

> 目前，该选项仅在[crang 编译器]中可用([https://clang.llvm.org/docs/SafeStack.html](https://clang.llvm.org/docs/SafeStack.html))，其中可以将 `-fsanitize=safe-stack` 传递给编译器。A [提案](https://gcc.gnu.org/ml/gcc/2016-04/msg00083.html)是为了在 GCC 中实现相同的功能。

### Conclusion

Stack corruptions can become easier to debug when safe stack is enabled. Due to a low performance overhead, you can even activated by default in your build configuration.

> 启用安全堆栈后，堆栈损坏可以变得更容易调试。由于性能开销较低，您甚至可以在构建配置中默认激活。

#### Remarks

This topic ain't complete yet, examples on following techniques/tools would be useful:

> 本主题尚未完成，以下技术/工具的示例将很有用：

- Mention more static analysis tools
- Binary instrumentation tools (like UBSan, TSan, MSan, ESan ...)
- Hardening (CFI ...)
- Fuzzing
