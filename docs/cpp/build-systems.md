---
tip: translate by baidu@2023-10-25 08:21:11
---
---
metaTitle: "C++ | Build Systems"
description: "Generating Build Environment with CMake, Compiling with GNU make, Building with SCons, Autotools (GNU), Ninja, NMAKE (Microsoft Program Maintenance Utility)"
---

# Build Systems




## Generating Build Environment with CMake



[CMake](https://cmake.org/) generates build environments for nearly any compiler or IDE from a single project definition. The following examples will demonstrate how to add a CMake file to the [cross-platform "Hello World" C++ code](http://stackoverflow.com/documentation/c%2B%2B/206/introduction-to-c/774/hello-world).

> [CMake](https://cmake.org/)从单个项目定义为几乎任何编译器或IDE生成构建环境。以下示例将演示如何将CMake文件添加到[跨平台“Hello World”C++代码](http://stackoverflow.com/documentation/c%2B%2B/206/introduction-to c/774/你好世界）。


CMake files are always named "CMakeLists.txt" and should already exist in every project's root directory (and possibly in sub-directories too.) A basic CMakeLists.txt file looks like:

> CMake文件总是命名为“CMakeLists.txt”，并且应该已经存在于每个项目的根目录中（也可能存在于子目录中）。基本的CMakeList.txt文件如下所示：

```cpp
cmake_minimum_required(VERSION 2.4)

project(HelloWorld)

add_executable(HelloWorld main.cpp)

```


See it [live on Coliru](http://coliru.stacked-crooked.com/a/616a0939bdd91b8b).

> 看吧[住在科利鲁](http://coliru.stacked-crooked.com/a/616a0939bdd91b8b)。


This file tells CMake the project name, what file version to expect, and instructions to generate an executable called "HelloWorld" that requires `main.cpp`.

> 该文件告诉CMake项目名称、预期的文件版本，以及生成名为“HelloWorld”的需要“main.cpp”的可执行文件的指令。


Generate a build environment for your installed compiler/IDE from the command line:

> 从命令行为已安装的编译器/IDE生成生成环境：

```cpp
> cmake .

```

Build the application with:

```cpp
> cmake --build .

```


This generates the default build environment for the system, depending on the OS and installed tools. Keep source code clean from any build artifacts with use of "out-of-source" builds:

> 这将根据操作系统和安装的工具生成系统的默认生成环境。使用“源代码外”构建，使源代码与任何构建工件保持干净：

```cpp
> mkdir build
> cd build
> cmake ..
> cmake --build .

```


CMake can also abstract the platform shell's basic commands from the previous example:

> CMake还可以从前面的示例中抽象出平台外壳的基本命令：

```cpp
> cmake -E make_directory build
> cmake -E chdir build cmake .. 
> cmake --build build 

```


CMake includes [generators](https://cmake.org/cmake/help/v3.0/manual/cmake-generators.7.html) for a number of common build tools and IDEs. To generate makefiles for [Visual Studio's `nmake`](https://msdn.microsoft.com/en-us/library/dd9y37ha.aspx):

> CMake包括[发电机](https://cmake.org/cmake/help/v3.0/manual/cmake-generators.7.html)用于许多常见的构建工具和IDE。为[Visual Studio的`nmake`]生成生成生成文件(https://msdn.microsoft.com/en-us/library/dd9y37ha.aspx)：

```cpp
> cmake -G "NMake Makefiles" ..
> nmake

```



## Compiling with GNU make


### Introduction


The GNU Make (styled `make`) is a program dedicated to the automation of executing shell commands. GNU Make is one specific program that falls under the Make family. Make remains popular among Unix-like and POSIX-like operating systems, including those derived from the Linux kernel, Mac OS X, and BSD.

> GNU Make（风格为“Make”）是一个专门用于自动化执行shell命令的程序。GNUMake是属于Make家族的一个特定程序。Make在类Unix和类POSIX操作系统中仍然很受欢迎，包括从Linux内核、Mac OS X和BSD派生的操作系统。


GNU Make is especially notable for being attached to the GNU Project, which is attached to the popular GNU/Linux operating system. GNU Make also has compatible versions running on various flavors of Windows and Mac OS X. It is also a very stable version with historical significance that remains popular. It is for these reasons that GNU Make is often taught alongside C and C++.

> GNUMake特别值得注意的是，它与GNU项目相连，后者与流行的GNU/Linux操作系统相连。GNU Make也有兼容的版本运行在各种风格的Windows和Mac OS X上。它也是一个非常稳定的版本，具有历史意义，仍然很受欢迎。正是由于这些原因，GNUMake经常与C和C++一起教授。

### Basic rules


To compile with make, create a Makefile in your project directory. Your Makefile could be as simple as:

> 要使用make进行编译，请在项目目录中创建一个Makefile。您的Makefile可以简单到：

**Makefile**

```cpp
# Set some variables to use in our command
# First, we set the compiler to be g++
CXX=g++

# Then, we say that we want to compile with g++'s recommended warnings and some extra ones.
CXXFLAGS=-Wall -Wextra -pedantic

# This will be the output file
EXE=app

SRCS=main.cpp

# When you call `make` at the command line, this "target" is called.
# The $(EXE) at the right says that the `all` target depends on the `$(EXE)` target.
# $(EXE) expands to be the content of the EXE variable
# Note: Because this is the first target, it becomes the default target if `make` is called without target
all: $(EXE)

# This is equivalent to saying
# app: $(SRCS)
# $(SRCS) can be separated, which means that this target would depend on each file.
# Note that this target has a "method body": the part indented by a tab (not four spaces).
# When we build this target, make will execute the command, which is:
# g++ -Wall -Wextra -pedantic -o app main.cpp
# I.E. Compile main.cpp with warnings, and output to the file ./app
$(EXE): $(SRCS)
    @$(CXX) $(CXXFLAGS) -o $@ $(SRCS)

# This target should reverse the `all` target. If you call
# make with an argument, like `make clean`, the corresponding target
# gets called.
clean:
    @rm -f $(EXE)

```

> 

**NOTE: Make absolutely sure that the indentations are with a tab, not with four spaces. Otherwise, you'll get an error of** `Makefile:10: *** missing separator. Stop.`

> **注意：绝对要确保凹痕带有一个制表符，而不是四个空格。否则，您将得到一个错误**`Makefile:10:***缺少分隔符。停下`


To run this from the command-line, do the following:

```cpp
$ cd ~/Path/to/project
$ make
$ ls
app  main.cpp  Makefile

$ ./app
Hello World!

$ make clean
$ ls
main.cpp  Makefile

```

### Incremental builds


When you start having more files, make becomes more useful. What if you edited **a.cpp** but not **b.cpp**? Recompiling **b.cpp** would take more time.

> 当您开始拥有更多的文件时，make会变得更加有用。如果您编辑了**a.cpp**，但没有编辑**b.cpp**呢？重新编译**b.cpp**需要更多的时间。

With the following directory structure:

```cpp
.
+-- src
|   +-- a.cpp
|   +-- a.hpp
|   +-- b.cpp
|   +-- b.hpp
+-- Makefile

```

This would be a good Makefile:

**Makefile**

```cpp
CXX=g++
CXXFLAGS=-Wall -Wextra -pedantic
EXE=app

SRCS_GLOB=src/*.cpp
SRCS=$(wildcard $(SRCS_GLOB))
OBJS=$(SRCS:.cpp=.o)

all: $(EXE)

$(EXE): $(OBJS)
    @$(CXX) -o $@ $(OBJS)

depend: .depend

.depend: $(SRCS)
    @-rm -f ./.depend
    @$(CXX) $(CXXFLAGS) -MM $^>>./.depend

clean:
    -rm -f $(EXE)
    -rm $(OBJS)
    -rm *~
    -rm .depend

include .depend

```


Again watch the tabs. This new Makefile ensures that you only recompile changed files, minimizing compile time.

> 再次注意标签。这个新的Makefile确保您只重新编译更改的文件，从而最大限度地减少编译时间。

### Documentation


For more on make, see [the official documentation by the Free Software Foundation](https://www.gnu.org/software/make/manual/), [the stackoverflow documentation](https://stackoverflow.com/documentation/makefile) and [dmckee's elaborate answer on stackoverflow](http://stackoverflow.com/a/2481326/1896169).

> 有关make的更多信息，请参阅[自由软件基金会的官方文档](https://www.gnu.org/software/make/manual/)，[stackoverflow文档](https://stackoverflow.com/documentation/makefile)和[Dmcke对stackoverflow的详尽回答](http://stackoverflow.com/a/2481326/1896169)。



## Building with SCons



You can build the [cross-platform "Hello World" C++ code](http://stackoverflow.com/documentation/c%2B%2B/206/introduction-to-c/774/hello-world), using [Scons](http://scons.org/) - A [Python](https://www.python.org/)-language software construction tool.

> 您可以构建[跨平台的“Hello World”C++代码](http://stackoverflow.com/documentation/c%2B%2B/206/introduction-to-c/774/hello-world），使用[Scons](http://scons.org/)-A[Python](https://www.python.org/)-语言软件构建工具。


First, create a file called `SConstruct` (note that SCons will look for a file with this exact name by default). For now, the file should be in a directory right along your `hello.cpp`. Write in the new file the line

> 首先，创建一个名为“SConstruct”的文件（请注意，默认情况下，SCons会查找具有此确切名称的文件）。目前，该文件应该位于“hello.cpp”旁边的目录中

```

Program('hello.cpp')

```

Now, from the terminal, run `scons`. You should see something like

```cpp
$ scons
scons: Reading SConscript files ...
scons: done reading SConscript files.
scons: Building targets ...
g++ -o hello.o -c hello.cpp
g++ -o hello hello.o
scons: done building targets.

```


(although the details will vary depending on your operating system and installed compiler).

> （尽管详细信息会因操作系统和安装的编译器而异）。


The `Environment` and `Glob` classes will help you further configure what to build. E.g., the `SConstruct` file

> “Environment”和“Glob”类将帮助您进一步配置要构建的内容。例如，“SConstruct”文件

```cpp
env=Environment(CPPPATH='/usr/include/boost/',
    CPPDEFINES=[],
    LIBS=[],
    SCONS_CXX_STANDARD="c++11"
    )

env.Program('hello', Glob('src/*.cpp'))     

```


builds the executable `hello`, using all `cpp` files in `src`. Its `CPPPATH` is `/usr/include/boost` and it specifies the C++11 standard.

> 使用“src”中的所有“cpp”文件生成可执行文件“hello”。它的“CPPPATH”是“/usr/include/boost”，它指定了C++11标准。



## Autotools (GNU)


### Introduction


The Autotools are a group of programs that create a GNU Build System for a given software package. It is a suite of tools that work together to produce various build resources, such as a Makefile (to be used with GNU Make). Thus, Autotools can be considered a de facto build system generator.

> Autotools是一组为给定软件包创建GNU构建系统的程序。它是一套协同工作以生成各种构建资源的工具，例如Makefile（将与GNUMake一起使用）。因此，Autotools可以被视为事实上的构建系统生成器。

Some notable Autotools programs include:

- Autoconf
- Automake (not to be confused with `make`)


In general, Autotools is meant to generate the Unix-compatible script and Makefile to allow the following command to build (as well as install) most packages (in the simple case):

> 通常，Autotools旨在生成Unix兼容的脚本和Makefile，以允许以下命令构建（以及安装）大多数包（在简单的情况下）：

```cpp
./configure && make && make install

```


As such, Autotools also has a relationship with certain package managers, especially those that are attached to operating systems that conform to the POSIX Standard(s).

> 因此，Autotools还与某些包管理器有关系，尤其是那些连接到符合POSIX标准的操作系统上的包管理器。



## Ninja


### Introduction


The Ninja build system is described by its project website as ["a small build system with a focus on speed."](https://ninja-build.org/) Ninja is designed to have its files generated by build system file generators, and takes a low-level approach to build systems, in contrast to higher-level build system managers like CMake or Meson.

> 忍者构建系统在其项目网站上被描述为[“一个注重速度的小型构建系统。”](https://ninja-build.org/)Ninja的文件是由构建系统文件生成器生成的，与CMake或Meson等更高级别的构建系统管理器相比，它采用了低级别的方法来构建系统。


Ninja is primarily written in C++ and Python, and was created as an alternative to the SCons build system for the Chromium project.

> Ninja主要是用C++和Python编写的，它是作为Chromium项目的SCons构建系统的替代品创建的。



## NMAKE (Microsoft Program Maintenance Utility)


### Introduction


NMAKE is a command-line utility developed by Microsoft to be used primarily in conjunction with Microsoft Visual Studio and/or the Visual C++ command line tools.

> NMAKE是由Microsoft开发的命令行实用程序，主要与Microsoft Visual Studio和/或Visual C++命令行工具结合使用。


NMAKE is build system that falls under the Make family of build systems, but has certain distinct features that diverge from Unix-like Make programs, such as supporting Windows-specific file path syntax (which itself differs from Unix-style file paths).

> NMAKE是一个属于Make系列构建系统的构建系统，但具有某些与类Unix Make程序不同的独特功能，例如支持Windows特定的文件路径语法（其本身与Unix风格的文件路径不同）。



#### Remarks



Currently, there exists no universal or dominant build system for C++ that is both popular and cross-platform. However, there do exist several major build systems that are attached to major platforms/projects, the most notable being GNU Make with the GNU/Linux operating system and NMAKE with the Visual C++/Visual Studio project system.

> 目前，还没有一个通用的或占主导地位的C++构建系统，既流行又跨平台。然而，确实存在一些连接到主要平台/项目的主要构建系统，最引人注目的是GNU/Linux操作系统的GNU Make和Visual C++/Visual Studio项目系统的NMAKE。


Additionally, some Integrated Development Environments (IDEs) also include specialized build systems to be used specifically with the native IDE. Certain build system generators can generate these native IDE build system/project formats, such as CMake for Eclipse and Microsoft Visual Studio 2012.

> 此外，一些集成开发环境（IDE）还包括专门用于本机IDE的专用构建系统。某些构建系统生成器可以生成这些本机IDE构建系统/项目格式，例如CMake for Eclipse和Microsoft Visual Studio 2012。

