---
tip: translate by baidu@2023-10-25 08:25:44
---
---
metaTitle: "C++ | Header Files"
description: "Basic Example, Templates in Header Files"
---

# Header Files




## Basic Example



The following example will contain a block of code that is meant to be split into several source files, as denoted by `// filename` comments.

> 以下示例将包含一个代码块，该代码块将被拆分为几个源文件，如“//filename”注释所示。

### Source Files

```cpp
// my_function.h

/* Note how this header contains only a declaration of a function.
 * Header functions usually do not define implementations for declarations
 * unless code must be further processed at compile time, as in templates.
 */

/* Also, usually header files include preprocessor guards so that every header
 * is never included twice.
 *
 * The guard is implemented by checking if a header-file unique preprocessor
 * token is defined, and only including the header if it hasn't been included
 * once before.
 */
#ifndef MY_FUNCTION_H
#define MY_FUNCTION_H

// global_value and my_function() will be
// recognized as the same constructs if this header is included by different files.
const int global_value = 42;
int my_function();

#endif // MY_FUNCTION_H

```

```cpp
// my_function.cpp

/* Note how the corresponding source file for the header includes the interface  
 * defined in the header so that the compiler is aware of what the source file is 
 * implementing.
 *
 * In this case, the source file requires knowledge of the global constant
 * global_value only defined in my_function.h. Without inclusion of the header
 * file, this source file would not compile.
 */
#include "my_function.h" // or #include "my_function.hpp"
int my_function() {
  return global_value; // return 42;
}

```


Header files are then included by other source files that want to use the functionality defined by the header interface, but don't require knowledge of its implementation (thus, reducing code coupling). The following program makes use of the header `my_function.h` as defined above:

> 然后，其他源文件会包含头文件，这些源文件希望使用头接口定义的功能，但不需要了解其实现（从而减少代码耦合）。以下程序使用了上面定义的标题“my_function.h”：

```cpp
// main.cpp

#include <iostream>       // A C++ Standard Library header.
#include "my_function.h"  // A personal header

int main(int argc, char** argv) {
  std::cout << my_function() << std::endl;
  return 0;
}

```

### The Compilation Process


Since header files are often part of a compilation process workflow, a typical compilation process making use of the header/source file convention will usually do the following.

> 由于头文件通常是编译过程工作流的一部分，因此使用头/源文件约定的典型编译过程通常会执行以下操作。


Assuming that the header file and source code file is already in the same directory, a programmer would execute the following commands:

> 假设头文件和源代码文件已经在同一目录中，程序员将执行以下命令：

```cpp
g++ -c my_function.cpp       # Compiles the source file my_function.cpp
                             # --> object file my_function.o

g++ main.cpp my_function.o   # Links the object file containing the 
                             # implementation of int my_function()
                             # to the compiled, object version of main.cpp
                             # and then produces the final executable a.out

```


Alternatively, if one wishes to compile `main.cpp` to an object file first, and

> 或者，如果希望首先将“main.cpp”编译为对象文件，并且
then link only object files together as the final step:

```cpp
g++ -c my_function.cpp
g++ -c main.cpp

g++ main.o my_function.o

```



## Templates in Header Files



Templates require compile-time generation of code: a templated function, for example, will be effectively turned into multiple distinct functions once a templated function is parameterized by use in source code.

> 模板需要在编译时生成代码：例如，一旦在源代码中使用模板化函数进行参数化，模板化函数将有效地转换为多个不同的函数。


This means that template function, member function, and class definitions cannot be delegated to a separate source code file, as any code that will use any templated construct requires knowledge of its definition to generally generate any derivative code.

> 这意味着模板函数、成员函数和类定义不能委托给单独的源代码文件，因为任何使用任何模板化构造的代码都需要了解其定义才能生成任何派生代码。


Thus, templated code, if put in headers, must also contain its definition. An example of this is below:

> 因此，模板化代码如果放在头中，也必须包含其定义。示例如下：

```cpp
// templated_function.h

template <typename T>
T* null_T_pointer() {
  T* type_point = NULL; // or, alternatively, nullptr instead of NULL
                        // for C++11 or later
  return type_point;
} 

```



#### Remarks



In C++, as in C, the C++ compiler and compilation process makes use of the C preprocessor. As specified by the GNU C Preprocessor manual, a header file is defined as the following:

> 在C++中，与在C中一样，C++编译器和编译过程使用C预处理器。根据GNU C预处理器手册的规定，头文件定义如下：

> 
<p>A header file is a file containing C declarations and macro
definitions (see Macros) to be shared between several source files.
You request the use of a header file in your program by including it,
with the C preprocessing directive ‘#include’.</p>
Header files serve two purposes.
<ul>
<li>System header files declare the interfaces to parts of the operating system. You include them in your program to supply the
definitions and declarations you need to invoke system calls and
libraries.</li>
<li>Your own header files contain declarations for interfaces between the source files of your program. Each time you have a group of
related declarations and macro definitions all or most of which are
needed in several different source files, it is a good idea to create
a header file for them.</li>
</ul>



However, to the C preprocessor itself, a header file is no different than a source file.

> 然而，对于C预处理器本身来说，头文件与源文件没有什么不同。


The header/source file organization scheme is simply a strongly-held and standard convention set by various software projects in order to provide separation between interface and implementation.

> 头文件/源文件组织方案只是由各种软件项目设置的一个严格的标准约定，以便在接口和实现之间提供分离。


Although it is not formally enforced by the C++ Standard itself, following the header/source file convention is highly recommended, and, in practice, is already almost ubiquitous.

> 尽管C++标准本身并没有正式强制执行它，但强烈建议遵循头/源文件约定，而且在实践中，这种约定几乎已经无处不在。


Note that header files may be replaced as a project file structure convention by the upcoming feature of modules, which is still to be considered for inclusion in a future C++ Standard as of the time of writing (e.g. C++20).

> 请注意，头文件可能会被即将到来的模块功能取代，作为项目文件结构约定，在撰写本文时，仍需考虑将其纳入未来的C++标准（例如C++20）。

