---
tip: translate by baidu@2023-10-25 08:22:26
---
---
metaTitle: "C++ | Compiling and Building"
description: "Compiling with GCC, Compiling with Visual Studio (Graphical Interface) - Hello World, Online Compilers, Compiling with Visual C++ (Command Line), Compiling with Clang, The C++ compilation process, Compiling with Code::Blocks (Graphical interface)"
---

# Compiling and Building



Programs written in C++ need to be compiled before they can be run. There is a large variety of compilers available depending on your operating system.

> 用C++编写的程序需要经过编译才能运行。根据您的操作系统，有多种编译器可供选择。



## Compiling with GCC



Assuming a single source file named `main.cpp`, the command to compile and link an non-optimized executable is as follows (Compiling without optimization is useful for initial development and debugging, although [`-Og`](https://gcc.gnu.org/onlinedocs/gcc/Optimize-Options.html#index-Og-723) is officially recommended for newer GCC versions).

> 假设有一个名为“main.cpp”的源文件，编译和链接未优化的可执行文件的命令如下（尽管[`-Og`]，但在没有优化的情况下编译对初始开发和调试很有用(https://gcc.gnu.org/onlinedocs/gcc/Optimize-Options.html#index-Og-723）被官方推荐用于较新的GCC版本）。

```cpp
g++ -o app -Wall main.cpp -O0

```


To produce an optimized executable for use in production, use one of the [`-O`](https://gcc.gnu.org/onlinedocs/gcc/Optimize-Options.html#index-O-716) options (see: [`-O1`](https://gcc.gnu.org/onlinedocs/gcc/Optimize-Options.html#index-O1-717), [`-O2`](https://gcc.gnu.org/onlinedocs/gcc/Optimize-Options.html#index-O2-718), [`-O3`](https://gcc.gnu.org/onlinedocs/gcc/Optimize-Options.html#index-O3-719), [`-Os`](https://gcc.gnu.org/onlinedocs/gcc/Optimize-Options.html#index-Os-721), [`-Ofast`](https://gcc.gnu.org/onlinedocs/gcc/Optimize-Options.html#index-Ofast-722)):

> 要生成用于生产的优化可执行文件，请使用[`-O`]之一(https://gcc.gnu.org/onlinedocs/gcc/Optimize-Options.html#index-O-716）选项（参见：[`-O1'](https://gcc.gnu.org/onlinedocs/gcc/Optimize-Options.html#index-O1-717），[`-O2`](https://gcc.gnu.org/onlinedocs/gcc/Optimize-Options.html#index-O2-718），[`-O3`](https://gcc.gnu.org/onlinedocs/gcc/Optimize-Options.html#index-O3-719），[`-Os`](https://gcc.gnu.org/onlinedocs/gcc/Optimize-Options.html#index-Os-721），[`-Ofast`](https://gcc.gnu.org/onlinedocs/gcc/Optimize-Options.html#index-Ofast-722））：

```cpp
g++ -o app -Wall -O2 main.cpp

```


If the -O option is omitted, -O0, which means no optimizations, is used as default (specifying -O without a number resolves to -O1).

> 如果省略了-O选项，则默认使用-O0（表示没有优化）（指定不带数字的-O解析为-O1）。


Alternatively, use optimization flags from the `O` groups (or more experimental optimizations) directly. The following example builds with `-O2` optimization, plus one flag from the `-O3` optimization level:

> 或者，直接使用“O”组中的优化标志（或更多实验优化）。以下示例使用“-O2”优化构建，再加上“-O3”优化级别的一个标志：

```cpp
g++ -o app -Wall -O2 -ftree-partial-pre main.cpp

```


To produce a platform-specific optimized executable (for use in production on the machine with the same architecture), use:

> 要生成特定于平台的优化可执行文件（用于具有相同体系结构的机器上的生产），请使用：

```cpp
g++ -o app -Wall -O2 -march=native main.cpp

```


Either of the above will produce a binary file that can be run with `.\app.exe` on Windows and `./app` on Linux, Mac OS, etc.

> 以上任何一项都将生成一个可以使用“”运行的二进制文件。\app.exe`在Windows和`/Linux、Mac OS等上的应用程序。


The [`-o`](https://gcc.gnu.org/onlinedocs/gcc/Overall-Options.html#index-o-86) flag can also be skipped. In this case, GCC will create default output executable `a.exe` on Windows and `a.out` on Unix-like systems.

> [`-o`](https://gcc.gnu.org/onlinedocs/gcc/Overall-Options.html#index-o-86）标志也可以被跳过。在这种情况下，GCC将在Windows上创建默认的输出可执行文件“a.exe”，在类Unix系统上创建“a.out”。

To compile a file without linking it, use the [`-c`](https://gcc.gnu.org/onlinedocs/gcc/Overall-Options.html#index-c-82) option:

> 要在不链接文件的情况下编译文件，请使用[`-c`](https://gcc.gnu.org/onlinedocs/gcc/Overall-Options.html#index-c-82）选项：

```cpp
g++ -o file.o -Wall -c file.cpp

```


This produces an object file named `file.o` which can later be linked with other files to produce a binary:

> 这将生成一个名为“file.o”的对象文件，该文件稍后可以与其他文件链接以生成二进制文件：

```cpp
g++ -o app file.o otherfile.o

```


More about optimization options can be found at [gcc.gnu.org](https://gcc.gnu.org/onlinedocs/gcc/Optimize-Options.html). Of particular note are `-Og` (optimization with an emphasis on debugging experience -- recommended for the standard edit-compile-debug cycle) and `-Ofast` (all optimizations, including ones disregarding strict standards compliance).

> 有关优化选项的更多信息，请访问[gcc.gnu.org/(https://gcc.gnu.org/onlinedocs/gcc/Optimize-Options.html)。特别值得注意的是“-Og”（强调调试经验的优化——建议用于标准编辑-编译-调试周期）和“-Ofast”（所有优化，包括无视严格标准遵从性的优化）。


The [`-Wall`](https://gcc.gnu.org/onlinedocs/gcc/Warning-Options.html#index-Wall-307) flag enables warnings for many common errors and should always be used. To improve code quality it is often encouraged also to use [`-Wextra`](https://gcc.gnu.org/onlinedocs/gcc/Warning-Options.html#index-Wextra-310) and other warning flags which are not automatically enabled by `-Wall` and `-Wextra`.

> [`-墙`](https://gcc.gnu.org/onlinedocs/gcc/Warning-Options.html#index-Wall-307）标志启用了对许多常见错误的警告，并且应该始终使用。为了提高代码质量，通常还鼓励使用[`-Wextra`](https://gcc.gnu.org/onlinedocs/gcc/Warning-Options.html#index-Wextra-310）和其他未由“-Wall”和“-Wextra”自动启用的警告标志。


If the code expects a specific C++ standard, specify which standard to use by including the [`-std=`](https://gcc.gnu.org/onlinedocs/gcc/C-Dialect-Options.html#index-std-112) flag. Supported values correspond to the year of finalization for each version of the ISO C++ standard. As of GCC 6.1.0, valid values for the `std=` flag are `c++98`/`c++03`, `c++11`, `c++14`, and `c++17`/`c++1z`. Values separated by a forward slash are equivalent.

> 如果代码需要特定的C++标准，请通过包含[`-std=`]来指定要使用的标准(https://gcc.gnu.org/onlinedocs/gcc/C-Dialect-Options.html#index-std-112）标志。支持的值对应于ISO C++标准每个版本的最终确定年份。自GCC 6.1.0起，“std=”标志的有效值为“c++98”/“c++03”、“c++11”、“c++14”和“c++17”/“c++1z”。用正斜杠分隔的值是等效的。

```cpp
g++ -std=c++11 <file>

```


GCC includes some compiler-specific extensions that are disabled when they conflict with a standard specified by the `-std=` flag. To compile with all extensions enabled, the value `gnu++XX` may be used, where `XX` is any of the years used by the `c++` values listed above.

> GCC包括一些编译器特定的扩展，当它们与“-std=”标志指定的标准冲突时，这些扩展将被禁用。要在启用所有扩展的情况下进行编译，可以使用值“gnu++XX”，其中“XX”是上面列出的“c++”值使用的任何年份。


The default standard will be used if none is specified. For versions of GCC prior to 6.1.0, the default is `-std=gnu++03`; in GCC 6.1.0 and greater, the default is `-std=gnu++14`.

> 如果未指定任何标准，则将使用默认标准。对于6.1.0之前的GCC版本，默认值为“-std=gnu++03”；在GCC 6.1.0及更高版本中，默认值为“-std=gnu++14”。


Note that due to bugs in GCC, the `-pthread` flag must be present at compilation and linking for GCC to support the C++ standard threading functionality introduced with C++11, such as `std::thread` and `std::wait_for`. Omitting it when using threading functions may result in [no warnings but invalid results](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=58929) on some platforms.

> 请注意，由于GCC中的错误，GCC在编译和链接时必须存在“-phread”标志，以支持C++11引入的C++标准线程功能，如“std:：thread”和“std::wait_for'”。使用线程函数时忽略它可能会导致[没有警告，但结果无效](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=58929)在某些平台上。

### Linking with libraries:

Use the `-l` option to pass the library name:

```cpp
g++ main.cpp -lpcre2-8
#pcre2-8 is the PCRE2 library for 8bit code units (UTF-8)

```


If the library is not in the standard library path, add the path with `-L` option:

> 如果库不在标准库路径中，请使用“-L”选项添加路径：

```cpp
g++ main.cpp -L/my/custom/path/ -lmylib

```

Multiple libraries can be linked together:

```cpp
g++ main.cpp -lmylib1 -lmylib2 -lmylib3

```


If one library depends on another, put the dependent library **before** the independent library:

> 如果一个库依赖于另一个库，请将依赖库**放在独立库**之前：

```cpp
g++ main.cpp -lchild-lib -lbase-lib

```


Or let the linker determine the ordering itself via `--start-group` and `--end-group` (note: this has significant performance cost):

> 或者让链接器通过“--起始组”和“--结束组”自行确定排序（注意：这会带来显著的性能成本）：

```cpp
g++ main.cpp -Wl,--start-group -lbase-lib -lchild-lib -Wl,--end-group

```



## Compiling with Visual Studio (Graphical Interface) - Hello World



1. Download and install [Visual Studio Community 2015](https://www.visualstudio.com/)

> 1.下载并安装〔Visual Studio社区2015〕(https://www.visualstudio.com/)
1. Open Visual Studio Community
<li>Click File -> New -> Project

[<img src="http://i.stack.imgur.com/bFNzb.png" alt="enter image description here" />](http://i.stack.imgur.com/bFNzb.png)</li>

> [<img src=“http://i.stack.imgur.com/bFNzb.png“alt=”在此处输入图像描述“/>](http://i.stack.imgur.com/bFNzb.png)</li>
<li>Click Templates -> Visual C++ -> Win32 Console Application and then name the project **MyFirstProgram**.

[<img src="http://i.stack.imgur.com/kYTy1.png" alt="enter image description here" />](http://i.stack.imgur.com/kYTy1.png)</li>

> [<img src=“http://i.stack.imgur.com/kYTy1.png“alt=”在此处输入图像描述“/>](http://i.stack.imgur.com/kYTy1.png)</li>
1. Click Ok
<li>Click Next in the following window.

[<img src="http://i.stack.imgur.com/Rebpz.png" alt="enter image description here" />](http://i.stack.imgur.com/Rebpz.png)</li>

> [<img src=“http://i.stack.imgur.com/Rebpz.png“alt=”在此处输入图像描述“/>](http://i.stack.imgur.com/Rebpz.png)</li>
<li>Check the `Empty project` box and then click Finish:

[<img src="http://i.stack.imgur.com/P0P5J.png" alt="enter image description here" />](http://i.stack.imgur.com/P0P5J.png)</li>

> [<img src=“http://i.stack.imgur.com/P0P5J.png“alt=”在此处输入图像描述“/>](http://i.stack.imgur.com/P0P5J.png)</li>
<li>Right click on folder Source File then -> Add --> New Item :

[<img src="http://i.stack.imgur.com/DLwEd.png" alt="enter image description here" />](http://i.stack.imgur.com/DLwEd.png)</li>

> [<img src=“http://i.stack.imgur.com/DLwEd.png“alt=”在此处输入图像描述“/>](http://i.stack.imgur.com/DLwEd.png)</li>
<li>Select C++ File and name the file main.cpp, then click Add:

[<img src="http://i.stack.imgur.com/zQaws.png" alt="enter image description here" />](http://i.stack.imgur.com/zQaws.png)

> [<img src=“http://i.stack.imgur.com/zQaws.png“alt=”在此处输入图像描述“/>](http://i.stack.imgur.com/zQaws.png)
10: Copy and paste the following code in the new file main.cpp:</li>

```cpp
#include <iostream>

int main()
{
    std::cout << "Hello World!\n";
    return 0;
}

```

You environment should look like:

[<img src="http://i.stack.imgur.com/vTBkv.png" alt="enter image description here" />](http://i.stack.imgur.com/vTBkv.png)

> [<img src=“http://i.stack.imgur.com/vTBkv.png“alt=”在此处输入图像描述“/>](http://i.stack.imgur.com/vTBkv.png)

1. Click Debug -> Start **Without** Debugging (or press ctrl + F5) :


[<img src="http://i.stack.imgur.com/B3twO.png" alt="enter image description here" />](http://i.stack.imgur.com/B3twO.png)

> [<img src=“http://i.stack.imgur.com/B3twO.png“alt=”在此处输入图像描述“/>](http://i.stack.imgur.com/B3twO.png)

<li>Done. You should get the following console output :

[<img src="http://i.stack.imgur.com/1AwnS.png" alt="enter image description here" />](http://i.stack.imgur.com/1AwnS.png)</li>

> [<img src=“http://i.stack.imgur.com/1AwnS.png“alt=”在此处输入图像描述“/>](http://i.stack.imgur.com/1AwnS.png)</li>



## Online Compilers



Various websites provide online access to C++ compilers. Online compiler's feature set vary significantly from site to site, but usually they allow to do the following:

> 各种网站提供对C++编译器的在线访问。在线编译器的功能集因站点而异，但通常允许执行以下操作：

- Paste your code into a web form in the browser.
- Select some compiler options and compile the code.
- Collect compiler and/or program output.


Online compiler website behavior is usually quite restrictive as they allow anyone to run compilers and execute arbitrary code on their server side, whereas ordinarily remote arbitrary code execution is considered as vulnerability.

> 在线编译器网站的行为通常非常严格，因为它们允许任何人在服务器端运行编译器并执行任意代码，而通常远程执行任意代码被视为漏洞。

Online compilers may be useful for the following purposes:

- Run a small code snippet from a machine which lacks C++ compiler (smartphones, tablets, etc.).

- Ensure that code compiles successfully with different compilers and runs the same way regardless the compiler it was compiled with.

> -确保代码使用不同的编译器成功编译，并以相同的方式运行，无论使用何种编译器编译。
- Learn or teach basics of C++.

- Learn modern C++ features (C++14 and C++17 in near future) when up-to-date C++ compiler is not available on local machine.

> -当本地机器上没有最新的C++编译器时，学习现代C++功能（不久的将来会有C++14和C++17）。

- Spot a bug in your compiler by comparison with a large set of other compilers. Check if a compiler bug was fixed in future versions, which are unavailable on your machine.

> -通过与大量其他编译器进行比较，发现编译器中的错误。检查在您的计算机上不可用的未来版本中是否修复了编译器错误。
- Solve online judge problems.

What online compilers should **not** be used for:


- Develop full-featured (even small) applications using C++. Usually online compilers do not allow to link with third-party libraries or download build artifacts.

> -使用C++开发全功能（甚至是小型）应用程序。通常，在线编译器不允许链接到第三方库或下载构建工件。

- Perform intensive computations. Sever-side computing resources are limited, so any user-provided program will be killed after a few seconds of execution. The permitted execution time is usually enough for testing and learning.

> -进行密集的计算。服务器端的计算资源是有限的，因此任何用户提供的程序都会在执行几秒钟后被杀死。允许的执行时间通常足以进行测试和学习。
- Attack compiler server itself or any third-party hosts on the net.

Examples:

> 

Disclaimer: documentation author(s) are not affiliated with any resources listed below. Websites are listed alphabetically.

> 免责声明：文档作者不隶属于以下列出的任何资源。网站按字母顺序排列。


- [http://codepad.org/](http://codepad.org/) Online compiler with code sharing. Editing code after compiling with a source code warning or error does not work so well.
- [http://coliru.stacked-crooked.com/](http://coliru.stacked-crooked.com/) Online compiler for which you specify the command line.  Provides both GCC and Clang compilers for use.
- [http://cpp.sh/](http://cpp.sh/) - Online compiler with C++14 support. Does not allow you to edit compiler command line, but some options are available via GUI controls.
- [https://gcc.godbolt.org/](https://gcc.godbolt.org/) - Provides a wide list of compiler versions, architectures, and disassembly output. Very useful when you need to inspect what your code compiles into by different compilers. GCC, Clang, MSVC (`CL`), Intel compiler (`icc`), ELLCC, and Zapcc are present, with one or more of these compilers available for the ARM, ARMv8 (as ARM64), Atmel AVR, MIPS, MIPS64, MSP430, PowerPC, x86, and x64 architecutres. Compiler command line arguments may be edited.
- [https://ideone.com/](https://ideone.com/) - Widely used on the Net to illustrate code snippet behavior.  Provides both GCC and Clang for use, but doesn't allow you to edit the compiler command line.
- [http://melpon.org/wandbox](http://melpon.org/wandbox) - Supports numerous Clang and GNU/GCC compiler versions.
- [http://onlinegdb.com/](http://onlinegdb.com/) - An extremely minimalistic IDE that includes an editor, a compiler (gcc), and a debugger (gdb).
- [http://rextester.com/](http://rextester.com/) - Provides Clang, GCC, and Visual Studio compilers for both C and C++ (along with compilers for other languages), with the Boost library available for use.
- [http://tutorialspoint.com/compile_cpp11_online.php](http://tutorialspoint.com/compile_cpp11_online.php) - Full-featured UNIX shell with GCC, and a user-friendly project explorer.
- [http://webcompiler.cloudapp.net/](http://webcompiler.cloudapp.net/) - Online Visual Studio 2015 compiler, provided by Microsoft as part of RiSE4fun.



## Compiling with Visual C++ (Command Line)



For programmers coming from GCC or Clang to Visual Studio, or programmers more comfortable with the command line in general, you can use the Visual C++ compiler from the command line as well as the IDE.

> 对于从GCC或Clang到Visual Studio的程序员，或者更熟悉命令行的程序员，您可以从命令行和IDE中使用Visual C++编译器。


If you desire to compile your code from the command line in Visual Studio, you first need to set up the command line environment.  This can be done either by opening the [`Visual Studio Command Prompt`/`Developer Command Prompt`/`x86 Native Tools Command Prompt`/`x64 Native Tools Command Prompt` or similar](https://msdn.microsoft.com/en-us/library/f2ccy3wt.aspx#Anchor_0) (as provided by your version of Visual Studio), or at the command prompt, by navigating to the `VC` subdirectory of the compiler's install directory (typically `\Program Files (x86)\Microsoft Visual Studio x\VC`, where `x` is the version number (such as `10.0` for 2010, or `14.0` for 2015) and running the `VCVARSALL` batch file with a command-line parameter specified [here](https://msdn.microsoft.com/en-us/library/f2ccy3wt.aspx#Anchor_1).

> 如果希望在VisualStudio中从命令行编译代码，则首先需要设置命令行环境。这可以通过打开[`Visual Studio命令提示符`/`Developer命令提示符`/\`x86 Native Tools命令提示符`/`x64 Native Tools Command Prompt`或类似命令来完成](https://msdn.microsoft.com/en-us/library/f2ccy3wt.aspx#Anchor_0)（由您的Visual Studio版本提供），或在命令提示符下，导航到编译器安装目录的“VC”子目录（通常为“\Program Files（x86）\Microsoft Visual Studio x\VC”，其中“x”是版本号（如“10.0”表示2010，“14.0”表示2015），并使用指定的命令行参数运行“VCVARSALL”批处理文件[此处](https://msdn.microsoft.com/en-us/library/f2ccy3wt.aspx#Anchor_1)。


Note that unlike GCC, Visual Studio doesn't provide a front-end for the linker (`link.exe`) via the compiler (`cl.exe`), but instead provides the linker as a separate program, which the compiler calls as it exits.  `cl.exe` and `link.exe` can be used separately with different files and options, or `cl` can be told to pass files and options to `link` if both tasks are done together.  Any linking options specified to `cl` will be translated into options for `link`, and any files not processed by `cl` will be passed directly to `link`.  As this is mainly a simple guide to compiling with the Visual Studio command line, arguments for `link` will not be described at this time; if you need a list, see [here](https://msdn.microsoft.com/en-us/library/y0zzbyt4.aspx).

> 请注意，与GCC不同，Visual Studio没有通过编译器（“cl.exe”）为链接器（“link.exe”）提供前端，而是将链接器作为一个单独的程序提供，编译器在退出时会调用该程序`cl.exe和link.exe可以分别与不同的文件和选项一起使用，或者如果两个任务同时完成，则可以告诉cl将文件和选项传递给link。任何指定给“cl”的链接选项都将被转换为“link”的选项，任何未由“cl”处理的文件都将直接传递给“link”。由于这主要是一个使用Visual Studio命令行进行编译的简单指南，因此此时将不描述“link”的参数；如果您需要列表，请参阅[此处](https://msdn.microsoft.com/en-us/library/y0zzbyt4.aspx)。


Note that arguments to `cl` are case-sensitive, while arguments to `link` are not.

> 请注意，“cl”的自变量区分大小写，而“link”的自变量则不区分大小写。


[Be advised that some of the following examples use the Windows shell "current directory" variable, `%cd%`, when specifying absolute path names.  For anyone unfamiliar with this variable, it expands to the current working directory.  From the command line, it will be the directory you were in when you ran `cl`, and is specified in the command prompt by default (if your command prompt is `C:\src>`, for example, then `%cd%` is `C:\src\`).]

> [请注意，以下某些示例在指定绝对路径名时使用Windows shell“当前目录”变量“%cd%”。对于不熟悉此变量的人，它将扩展到当前工作目录。从命令行中，它将是您运行“cl”时所在的目录，默认情况下在命令提示符中指定（例如，如果命令提示符为“C:\src>`，则“%cd%”为“C:\src\`”）。]


Assuming a single source file named `main.cpp` in the current folder, the command to compile and link an unoptimised executable (useful for initial development and debugging) is (use either of the following):

> 假设当前文件夹中有一个名为“main.cpp”的源文件，则编译和链接未优化的可执行文件（对初始开发和调试有用）的命令为（使用以下任一项）：

```cpp
cl main.cpp
// Generates object file "main.obj".
// Performs linking with "main.obj".
// Generates executable "main.exe".

cl /Od main.cpp
// Same as above.
// "/Od" is the "Optimisation: disabled" option, and is the default when no /O is specified.

```


Assuming an additional source file "niam.cpp" in the same directory, use the following:

> 假设同一目录中有一个额外的源文件“niam.cpp”，请使用以下内容：

```cpp
cl main.cpp niam.cpp
// Generates object files "main.obj" and "niam.obj".
// Performs linking with "main.obj" and "niam.obj".
// Generates executable "main.exe".

```

You can also use wildcards, as one would expect:

```cpp
cl main.cpp src\*.cpp
// Generates object file "main.obj", plus one object file for each ".cpp" file in folder
//  "%cd%\src".
// Performs linking with "main.obj", and every additional object file generated.
// All object files will be in the current folder.
// Generates executable "main.exe".

```

To rename or relocate the executable, use one of the following:

```cpp
cl /o name main.cpp
// Generates executable named "name.exe".

cl /o folder\ main.cpp
// Generates executable named "main.exe", in folder "%cd%\folder".

cl /o folder\name main.cpp
// Generates executable named "name.exe", in folder "%cd%\folder".

cl /Fename main.cpp
// Same as "/o name".

cl /Fefolder\ main.cpp
// Same as "/o folder\".

cl /Fefolder\name main.cpp
// Same as "/o folder\name".

```


Both `/o` and `/Fe` pass their parameter (let's call it `o-param`) to `link` as `/OUT:o-param`, appending the appropriate extension (generally `.exe` or `.dll`) to "name" `o-param`s as necessary.  While both `/o` and `/Fe` are to my knowledge identical in functionality, the latter is preferred for Visual Studio.  `/o` is marked as deprecated, and appears to mainly be provided for programmers more familiar with GCC or Clang.

> `/o`和`/Fe`都将它们的参数（让我们称之为`o-param`）作为`/OUT:o-param`传递给`link`，并在必要时将适当的扩展名（通常为`.exe`或`.dll`）附加到“name”`o-param`s。据我所知，“/o”和“/Fe”在功能上完全相同，但后者更适合Visual Studio`/o`被标记为不推荐使用，并且似乎主要是为更熟悉GCC或Clang的程序员提供的。


Note that while the space between `/o` and the specified folder and/or name is optional, there **cannot** be a space between `/Fe` and the specified folder and/or name.

> 请注意，虽然“/o”和指定文件夹和/或名称之间的空格是可选的，但“/Fe”和指定的文件夹和/或者名称之间不能有**空格。


Similarly, to produce an optimised executable (for use in production), use:

> 类似地，要生成优化的可执行文件（用于生产），请使用：

```cpp
cl /O1 main.cpp
// Optimise for executable size.  Produces small programs, at the possible expense of slower
//  execution.

cl /O2 main.cpp
// Optimise for execution speed.  Produces fast programs, at the possible expense of larger
//  file size.

cl /GL main.cpp other.cpp
// Generates special object files used for whole-program optimisation, which allows CL to
//  take every module (translation unit) into consideration during optimisation.
// Passes the option "/LTCG" (Link-Time Code Generation) to LINK, telling it to call CL during
//  the linking phase to perform additional optimisations.  If linking is not performed at this
//  time, the generated object files should be linked with "/LTCG".
// Can be used with other CL optimisation options.

```


Finally, to produce a platform-specific optimized executable (for use in production on the machine with the specified architecture), choose the appropriate command prompt or [`VCVARSALL` parameter](https://msdn.microsoft.com/en-us/library/x4d2c09s.aspx) for the target platform.  `link` should detect the desired platform from the object files; if not, use the [`/MACHINE` option](https://msdn.microsoft.com/en-us/library/5wy54dk2.aspx) to explicitly specify the target platform.

> 最后，要生成特定于平台的优化可执行文件（用于具有指定体系结构的机器上的生产），请选择适当的命令提示符或[`VCVARSAL`参数](https://msdn.microsoft.com/en-us/library/x4d2c09s.aspx)用于目标平台`链接“应该从对象文件中检测所需的平台；如果没有，请使用[`/MACHINE`选项](https://msdn.microsoft.com/en-us/library/5wy54dk2.aspx)以明确指定目标平台。

```cpp
// If compiling for x64, and LINK doesn't automatically detect target platform:
cl main.cpp /link /machine:X64

```


Any of the above will produce an executable with the name specified by `/o` or `/Fe`, or if neither is provided, with a name identical to the first source or object file specified to the compiler.

> 以上任何一项都将产生一个可执行文件，其名称由“/o”或“/Fe”指定，或者如果两者都不提供，则其名称与指定给编译器的第一个源或目标文件相同。

```cpp
cl a.cpp b.cpp c.cpp
// Generates "a.exe".

cl d.obj a.cpp q.cpp
// Generates "d.exe".

cl y.lib n.cpp o.obj
// Generates "n.exe".

cl /o yo zp.obj pz.cpp
// Generates "yo.exe".

```

To compile a file(s) without linking, use:

```cpp
cl /c main.cpp
// Generates object file "main.obj".

```


This tells `cl` to exit without calling `link`, and produces an object file, which can later be linked with other files to produce a binary.

> 这个命令告诉“cl”在不调用“link”的情况下退出，并生成一个对象文件，该文件稍后可以与其他文件链接以生成二进制文件。

```cpp
cl main.obj niam.cpp
// Generates object file "niam.obj".
// Performs linking with "main.obj" and "niam.obj".
// Generates executable "main.exe".

link main.obj niam.obj
// Performs linking with "main.obj" and "niam.obj".
// Generates executable "main.exe".

```


There are other valuable command line parameters as well, which it would be very useful for users to know:

> 还有其他有价值的命令行参数，用户知道这些参数会非常有用：

```cpp
cl /EHsc main.cpp
// "/EHsc" specifies that only standard C++ ("synchronous") exceptions will be caught,
//  and `extern "C"` functions will not throw exceptions.
// This is recommended when writing portable, platform-independent code.

cl /clr main.cpp
// "/clr" specifies that the code should be compiled to use the common language runtime,
//  the .NET Framework's virtual machine.
// Enables the use of Microsoft's C++/CLI language in addition to standard ("native") C++,
//  and creates an executable that requires .NET to run.

cl /Za main.cpp
// "/Za" specifies that Microsoft extensions should be disabled, and code should be
//  compiled strictly according to ISO C++ specifications.
// This is recommended for guaranteeing portability.

cl /Zi main.cpp
// "/Zi" generates a program database (PDB) file for use when debugging a program, without
//  affecting optimisation specifications, and passes the option "/DEBUG" to LINK.

cl /LD dll.cpp
// "/LD" tells CL to configure LINK to generate a DLL instead of an executable.
// LINK will output a DLL, in addition to an LIB and EXP file for use when linking.
// To use the DLL in other programs, pass its associated LIB to CL or LINK when compiling those
//  programs.

cl main.cpp /link /LINKER_OPTION
// "/link" passes everything following it directly to LINK, without parsing it in any way.
// Replace "/LINKER_OPTION" with any desired LINK option(s).

```


For anyone more familiar with *nix systems and/or GCC/Clang, `cl`, `link`, and other Visual Studio command line tools can accept parameters specified with a hyphen (such as `-c`) instead of a slash (such as `/c`).  Additionally, Windows recognises either a slash or a backslash as a valid path separator, so *nix-style paths can be used as well.  This makes it easy to convert simple compiler command lines from `g++` or `clang++` to `cl`, or vice versa, with minimal changes.

> 对于更熟悉*nix系统和/或GCC/Clang的用户，“cl”、“link”和其他Visual Studio命令行工具可以接受用连字符（如“-c”）而不是斜线（如“/c”）指定的参数。此外，Windows将斜线或反斜线识别为有效的路径分隔符，因此也可以使用*nix样式的路径。这使得简单的编译器命令行可以很容易地从“g++”或“clang++”转换为“cl”，反之亦然，只需最少的更改。

```cpp
g++ -o app src/main.cpp
cl  -o app src/main.cpp

```


Of course, when porting command lines that use more complex `g++` or `clang++` options, you need to look up equivalent commands in the applicable compiler documentations and/or on resource sites, but this makes it easier to get things started with minimal time spent learning about new compilers.

> 当然，当移植使用更复杂的“g++”或“clang++”选项的命令行时，您需要在适用的编译器文档和/或资源网站上查找等效命令，但这可以使您更容易地开始使用，只需花费最少的时间来学习新编译器。


In case you need specific language features for your code, a specific release of MSVC was required. From [Visual C++ 2015 Update 3](https://blogs.msdn.microsoft.com/vcblog/2016/06/07/standards-version-switches-in-the-compiler/) on it is possible to choose the version of the standard to compile with via the `/std` flag. Possible values are `/std:c++14` and `/std:c++latest` (`/std:c++17` will follow soon).

> 如果您的代码需要特定的语言功能，则需要特定版本的MSVC。摘自[Visual C++2015更新3](https://blogs.msdn.microsoft.com/vcblog/2016/06/07/standards-version-switches-in-the-compiler/)在上，可以通过“/std”标志选择要编译的标准版本。可能的值为“/std:c++14”和“/std:c++latest”（“/std:c++17”将很快出现）。


Note: In older versions of this compiler, specific feature flags were available however this was mostly used for previews of new features.

> 注意：在该编译器的旧版本中，可以使用特定的功能标志，但这主要用于预览新功能。



## Compiling with Clang



As the [Clang](http://clang.llvm.org/) front-end is designed for being compatible with GCC, most programs that can be compiled via [GCC](https://stackoverflow.com/documentation/c%2B%2B/206/introduction-to-c/1334/compiling-with-gcc) will compile when you swap `g++` by `clang++` in the build scripts. If no `-std=version` is given, gnu11 will be used.

> 作为【Clang】(http://clang.llvm.org/)前端被设计为与GCC兼容，大多数程序都可以通过GCC编译(https://stackoverflow.com/documentation/c%2B%2B/206/introduction-to-c/1334/使用gcc编译）将在编译脚本中将“g++”替换为“clang++”时进行编译。如果没有给出“-std=version”，那么将使用gnu11。


Windows users who are used to [MSVC](https://stackoverflow.com/documentation/c%2B%2B/206/introduction-to-c/5959/compiling-with-visual-c-command-line) can swap `cl.exe` with `clang-cl.exe`. By default, clang tries to be compatible with the highest version of MSVC that has been installed.

> 习惯[MSVC]的Windows用户(https://stackoverflow.com/documentation/c%2B%2B/206/introduction-to-c/5959/compiling-with-visual-c命令行）可以将“cl.exe”与“clang-cl.exe”交换。默认情况下，clang尝试与已安装的MSVC的最高版本兼容。


In the case of compiling with visual studio, clang-cl can be used by changing the `Platform toolset` in the project properties.

> 在使用visual studio进行编译的情况下，可以通过更改项目属性中的“平台工具集”来使用clang cl。


In both cases, clang is only compatible via its front-end, though it also tries to generate binary compatible object files. Users of clang-cl should note that [the compatibility with MSVC is not complete yet](http://clang.llvm.org/docs/MSVCCompatibility.html).

> 在这两种情况下，clang仅通过其前端兼容，尽管它也尝试生成二进制兼容的对象文件。clang cl的用户应该注意，[与MSVC的兼容性尚未完全](http://clang.llvm.org/docs/MSVCCompatibility.html)。


To use clang or clang-cl, one could use the default installation on certain Linux distributions or those bundled with IDEs (like XCode on Mac). For other versions of this compiler or on platforms which don't have this installed, this can be download from the [official download page](http://llvm.org/releases/download.html).

> 要使用clang或clang-cl，可以在某些Linux发行版或与IDE捆绑的发行版（如Mac上的XCode）上使用默认安装。对于该编译器的其他版本或未安装该编译器的平台，可以从[官方下载页]下载(http://llvm.org/releases/download.html)。


If you're using CMake to build your code you can usually switch the compiler by setting the `CC` and `CXX` environment variables like this:

> 如果您使用CMake构建代码，通常可以通过如下设置“CC”和“CXX”环境变量来切换编译器：

```cpp
mkdir build
cd build
CC=clang CXX=clang++ cmake ..
cmake --build .

```


See also [introduction to Cmake](http://stackoverflow.com/documentation/cmake/862/introduction-to-cmake#t=201608042347032067346).

> 另请参阅[Cmake简介](http://stackoverflow.com/documentation/cmake/862/introduction-to-cmake#t=201608042347032067346)。



## The C++ compilation process



When you develop a C++ program, the next step is to compile the program before running it. The compilation is the process which converts the program written in human readable language like C, C++ etc into a machine code, directly understood by the Central Processing Unit.  For example, if you have a C++ source code file named prog.cpp and you execute the compile command,

> 当你开发C++程序时，下一步是在运行之前编译程序。编译是将用C、C++等人类可读语言编写的程序转换为机器代码的过程，中央处理器可以直接理解。例如，如果您有一个名为prog.cpp的C++源代码文件，并且您执行了compile命令，

```

  g++ -Wall -ansi -o prog prog.cpp

```


There are 4 main stages involved in creating an executable file from the source file.

> 从源文件创建可执行文件涉及4个主要阶段。

<li>

The C++ the preprocessor takes a C++ source code file and deals with the headers(#include), macros(#define) and other preprocessor directives.

> C++预处理器采用C++源代码文件，处理头（#include）、宏（#define）和其他预处理器指令。
</li>
<li>
<p>The expanded C++ source code file produced by the C++ preprocessor is
compiled into the assembly language for the platform.</p>
</li>
<li>
<p>The assembler code generated by the compiler is assembled into the
object code for the platform.</p>
</li>
<li>
<p>The object code file produced by the assembler is linked together<br />
with the object code files for any library functions used to produce
either a library or an executable file.</p>
</li>

**Preprocessing**


The preprocessor handles the preprocessor directives, like #include and #define. It is agnostic of the syntax of C++, which is why it must be used with care.

> 预处理器处理预处理器指令，如#include和#define。它不知道C++的语法，这就是为什么必须小心使用它的原因。


It works on one C++ source file at a time by replacing #include directives with the content of the respective files (which is usually just declarations), doing replacement of macros (#define), and selecting different portions of text depending of #if, #ifdef and #ifndef directives.

> 它一次处理一个C++源文件，方法是用相应文件的内容替换#include指令（通常只是声明），替换宏（#define），并根据#if、#ifdef和#ifndef指令选择文本的不同部分。


The preprocessor works on a stream of preprocessing tokens. Macro substitution is defined as replacing tokens with other tokens (the operator ## enables merging two tokens when it make sense).

> 预处理器处理预处理令牌流。宏替换被定义为用其他令牌替换令牌（运算符##允许在有意义时合并两个令牌）。


After all this, the preprocessor produces a single output that is a stream of tokens resulting from the transformations described above. It also adds some special markers that tell the compiler where each line came from so that it can use those to produce sensible error messages.

> 在所有这些之后，预处理器产生单个输出，该输出是由上述转换产生的令牌流。它还添加了一些特殊的标记，告诉编译器每一行的来源，以便使用这些标记生成合理的错误消息。


Some errors can be produced at this stage with clever use of the #if and #error directives.

> 在这个阶段，巧妙地使用#if和#error指令可以产生一些错误。


By using below compiler flag, we can stop the process at preprocessing stage.

> 通过使用下面的编译器标志，我们可以在预处理阶段停止进程。

```cpp
g++ -E prog.cpp

```

**Compilation**


The compilation step is performed on each output of the preprocessor. The compiler parses the pure C++ source code (now without any preprocessor directives) and converts it into assembly code. Then invokes underlying back-end(assembler in toolchain) that assembles that code into machine code producing actual binary file in some format(ELF, COFF, a.out, ...). This object file contains the compiled code (in binary form) of the symbols defined in the input. Symbols in object files are referred to by name.

> 编译步骤是对预处理器的每个输出执行的。编译器解析纯C++源代码（现在没有任何预处理器指令），并将其转换为汇编代码。然后调用底层后端（工具链中的汇编程序），后者将代码汇编成机器代码，以某种格式（ELF、COFF、a.out…）生成实际的二进制文件。该对象文件包含输入中定义的符号的编译代码（二进制形式）。对象文件中的符号是通过名称引用的。


Object files can refer to symbols that are not defined. This is the case when you use a declaration, and don't provide a definition for it. The compiler doesn't mind this, and will happily produce the object file as long as the source code is well-formed.

> 对象文件可以引用未定义的符号。当你使用一个声明，但没有为它提供定义时，就会出现这种情况。编译器并不介意这一点，只要源代码格式正确，就会很乐意生成目标文件。


Compilers usually let you stop compilation at this point. This is very useful because with it you can compile each source code file separately. The advantage this provides is that you don't need to recompile everything if you only change a single file.

> 编译器通常允许您在此时停止编译。这非常有用，因为使用它可以分别编译每个源代码文件。这样做的好处是，如果只更改一个文件，就不需要重新编译所有内容。


The produced object files can be put in special archives called static libraries, for easier reusing later on.

> 生成的对象文件可以放在称为静态库的特殊档案中，以便以后更容易重用。


It's at this stage that "regular" compiler errors, like syntax errors or failed overload resolution errors, are reported.

> 正是在这个阶段，才会报告“常规”编译器错误，如语法错误或失败的重载解析错误。


In order to stop the process after the compile step, we can use the -S option:

> 为了在编译步骤后停止进程，我们可以使用-S选项：

```cpp
g++ -Wall -ansi -S prog.cpp

```

**Assembling**


The assembler creates object code. On a UNIX system you may see files with a .o suffix (.OBJ on MSDOS) to indicate object code files. In this phase the assembler converts those object files from assembly code into machine level instructions and the file created is a relocatable object code. Hence, the compilation phase generates the relocatable object program and this program can be used in different places without having to compile again.

> 汇编程序创建目标代码。在UNIX系统上，您可能会看到后缀为.o（MSDOS上为.OBJ）的文件，以指示目标代码文件。在这个阶段，汇编程序将这些目标文件从汇编代码转换为机器级指令，创建的文件是可重定位的目标代码。因此，编译阶段生成可重定位的对象程序，并且该程序可以在不同的地方使用，而无需再次编译。

To stop the process after the assembly step, you can use the -c option:

```cpp
g++ -Wall -ansi -c prog.cpp

```

**Linking**


The linker is what produces the final compilation output from the object files the assembler produced. This output can be either a shared (or dynamic) library (and while the name is similar, they don't have much in common with static libraries mentioned earlier) or an executable.

> 链接器是从汇编程序生成的对象文件中生成最终编译输出的对象。这个输出可以是共享（或动态）库（虽然名称相似，但它们与前面提到的静态库没有太多共同点），也可以是可执行文件。


It links all the object files by replacing the references to undefined symbols with the correct addresses. Each of these symbols can be defined in other object files or in libraries. If they are defined in libraries other than the standard library, you need to tell the linker about them.

> 它通过用正确的地址替换对未定义符号的引用来链接所有对象文件。这些符号中的每一个都可以在其他对象文件或库中定义。如果它们是在标准库以外的库中定义的，则需要将它们告知链接器。


At this stage the most common errors are missing definitions or duplicate definitions. The former means that either the definitions don't exist (i.e. they are not written), or that the object files or libraries where they reside were not given to the linker. The latter is obvious: the same symbol was defined in two different object files or libraries.

> 在这个阶段，最常见的错误是缺少定义或定义重复。前者意味着定义不存在（即没有编写），或者它们所在的对象文件或库没有提供给链接器。后者是显而易见的：在两个不同的对象文件或库中定义了相同的符号。



## Compiling with Code::Blocks (Graphical interface)


<li>

Download and install Code::Blocks [here](http://www.codeblocks.org/downloads/binaries). If you're on Windows, be careful to select a file for which the name contains `mingw`, the other files don't install any compiler.

> 下载并安装代码：：Blocks[此处](http://www.codeblocks.org/downloads/binaries)。如果你在Windows上，请小心选择一个名称包含“mingw”的文件，其他文件不会安装任何编译器。
</li>
<li>
Open Code::Blocks and click on "Create a new project":

[<img src="https://i.stack.imgur.com/z4Oll.png" alt="enter image description here" />](https://i.stack.imgur.com/z4Oll.png)

> [<img src=“https://i.stack.imgur.com/z4Oll.png“alt=”在此处输入图像描述“/>](https://i.stack.imgur.com/z4Oll.png)
</li>
<li>
Select "Console application" and click "Go":

[<img src="https://i.stack.imgur.com/0wBAn.png" alt="enter image description here" />](https://i.stack.imgur.com/0wBAn.png)

> [<img src=“https://i.stack.imgur.com/0wBAn.png“alt=”在此处输入图像描述“/>](https://i.stack.imgur.com/0wBAn.png)
</li>
<li>

Click "Next", select "C++", click "Next", select a name for your project and choose a folder to save it in, click "Next" and then click "Finish".

> 单击“下一步”，选择“C++”，单击“下”，为您的项目选择一个名称并选择要保存的文件夹，单击“上一步”然后单击“完成”。
</li>
<li>

Now you can edit and compile your code. A default code that prints "Hello world!" in the console is already there. To compile and/or run your program, press one of the three compile/run buttons in the toolbar:

> 现在您可以编辑和编译代码了。在控制台中打印“Hello world！”的默认代码已经存在。要编译和/或运行程序，请按工具栏中的三个编译/运行按钮之一：

[<img src="https://i.stack.imgur.com/wCmdw.png" alt="enter image description here" />](https://i.stack.imgur.com/wCmdw.png)

> [<img src=“https://i.stack.imgur.com/wCmdw.png“alt=”在此处输入图像描述“/>](https://i.stack.imgur.com/wCmdw.png)

To compile without running, press [<img src="https://i.stack.imgur.com/gOkY9.png" alt="Build" />](https://i.stack.imgur.com/gOkY9.png), to run without compiling again, press [<img src="https://i.stack.imgur.com/eLjbt.png" alt="Run" />](https://i.stack.imgur.com/eLjbt.png) and to compile and then run, press [<img src="https://i.stack.imgur.com/Zgyi8.png" alt="Build and run" />](https://i.stack.imgur.com/Zgyi8.png).

> 要在不运行的情况下编译，请按[<img src=“https://i.stack.imgur.com/gOkY9.png“alt=”Build“/>](https://i.stack.imgur.com/gOkY9.png)，要在不重新编译的情况下运行，请按[<img src=“https://i.stack.imgur.com/eLjbt.png“alt=”Run“/>](https://i.stack.imgur.com/eLjbt.png)要编译并运行，请按[<img src=“https://i.stack.imgur.com/Zgyi8.png“alt=”构建并运行“/>](https://i.stack.imgur.com/Zgyi8.png)。

Compiling and running the default "Hello world!" code gives the following result:

> 编译并运行默认的“Hello world！”代码会得到以下结果：

[<img src="https://i.stack.imgur.com/qbVy8.png" alt="enter image description here" />](https://i.stack.imgur.com/qbVy8.png)

> [<img src=“https://i.stack.imgur.com/qbVy8.png“alt=”在此处输入图像描述“/>](https://i.stack.imgur.com/qbVy8.png)
</li>



#### Remarks



Most operating systems ship without a compiler, and they have to be installed later. Some common compilers choices are:

> 大多数操作系统出厂时都没有编译器，必须稍后安装。一些常见的编译器选项包括：

- [GCC, the GNU Compiler Collection](https://gcc.gnu.org/) [g++](/questions/tagged/g%2b%2b)
- [clang: a C language family frontend for LLVM](http://clang.llvm.org/) [clang++](/questions/tagged/clang%2b%2b)
- [MSVC, Microsoft Visual C++ (included in Visual Studio)](https://www.visualstudio.com/) [visual-c++](/questions/tagged/visual-c%2b%2b)
- [C++Builder, Embarcadero C++Builder](https://www.embarcadero.com/products/cbuilder) (included in RAD Studio) [c++builder](/questions/tagged/c%2b%2bbuilder)


Please consult the appropriate compiler manual, on how to compile a C++ program.

> 关于如何编译C++程序，请参阅相应的编译器手册。


Another option to use a specific compiler with its own specific build system, it is possible to let generic [build systems](https://stackoverflow.com/documentation/c%2B%2B/8200/build-systems) configure the project for a specific compiler or for the default installed one.

> 另一种选择是将特定编译器与自己的特定构建系统一起使用，可以让泛型[构建系统](https://stackoverflow.com/documentation/c%2B%2B/8200/build-系统）为特定编译器或为默认安装的编译器配置项目。

