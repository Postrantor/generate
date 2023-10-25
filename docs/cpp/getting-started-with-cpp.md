---
tip: translate by baidu@2023-10-25 08:25:22
---
---
metaTitle: "C++ | Getting started with C++"
description: "Hello World, Comments, The standard C++ compilation process, Function, Visibility of function prototypes and declarations, Preprocessor"
---

# Getting started with C++



## Hello World


This program prints `Hello World!` to the standard output stream:

```cpp
#include <iostream>

int main()
{
    std::cout << "Hello World!" << std::endl;
}

```


See it [live on Coliru](http://coliru.stacked-crooked.com/a/ba766ad8ca2fae56).

> 看吧[住在科利鲁](http://coliru.stacked-crooked.com/a/ba766ad8ca2fae56)。

### Analysis

Let's examine each part of this code in detail:

<li>
`#include <iostream>` is a **preprocessor directive** that includes the content of the standard C++ header file `iostream`.
`iostream` is a **standard library header file** that contains definitions of the standard input and output streams. These definitions are included in the `std` namespace, explained below.

The **standard input/output (I/O) streams** provide ways for programs to get input from and output to an external system -- usually the terminal.

> **标准输入/输出（I/O）流**为程序提供了从外部系统（通常是终端）获得输入和输出的方法。
</li>
<li>
`int main() { ... }` defines a new [function](http://stackoverflow.com/documentation/c%2B%2B/206/getting-started-with-c-language/25460/function) named `main`. By convention, the `main` function is called upon execution of the program. There must be only one `main` function in a C++ program, and it must always return a number of the `int` type.
<p>Here, the `int` is what is called the function's [return type](http://stackoverflow.com/documentation/c%2B%2B/572/lambdas/1855/specifying-the-return-type#t=20170126184705835537).
The value returned by the `main` function is an **exit code.**</p>

By convention, a program exit code of `0` or `EXIT_SUCCESS` is interpreted as success by a system that executes the program. Any other return code is associated with an error.

> 按照惯例，执行程序的系统将程序退出代码“0”或“exit_SUCCESS”解释为成功。任何其他返回代码都与错误相关联。

If no `return` statement is present, the `main` function (and thus, the program itself) returns `0` by default. In this example, we don't need to explicitly write `return 0;`.

> 如果不存在“return”语句，则“main”函数（以及程序本身）默认返回“0”。在本例中，我们不需要显式地编写“return 0；”。

All other functions, except those that return the `void` type, must explicitly return a value according to their return type, or else must not return at all.

> 除返回“void”类型的函数外，所有其他函数都必须根据其返回类型显式返回值，否则根本不能返回。
</li>

<li>
`std::cout << "Hello World!" << std::endl;` prints "Hello World!" to the standard output stream:
<ul>
<li>
`std` is a [namespace](http://stackoverflow.com/documentation/c%2B%2B/495/namespaces#t=201701261847520776598), and `::` is the **scope resolution operator** that allows look-ups for objects by name within a namespace.

There are many namespaces. Here, we use `::` to show we want to use `cout` from the `std` namespace. For more information refer to [Scope Resolution Operator - Microsoft Documentation](https://msdn.microsoft.com/en-us/library/b451xz31.aspx/).

> 有许多名称空间。在这里，我们使用“：：”来表示我们希望使用“std”命名空间中的“cout”。有关详细信息，请参阅[Scope Resolution Operator-Microsoft Documentation](https://msdn.microsoft.com/en-us/library/b451xz31.aspx/)。
</li>
<li>
`std::cout` is the **standard output stream** object, defined in `iostream`, and it prints to the standard output (`stdout`).
</li>
<li>
`<<` is, **in this context**, the **stream insertion operator**, so called because it **inserts** an object into the **stream** object.
<p>The standard library defines the `<<` operator to perform data insertion for certain data types into output streams. `stream << content` inserts `content` into the stream and returns the same, but updated stream. This allows stream insertions to be chained:
`std::cout << "Foo" << " Bar";`
prints "FooBar" to the console.</p>
</li>
<li>
`"Hello World!"` is a **character string literal**, or a "text literal." The stream insertion operator for character string literals is defined in file `iostream`.
</li>
<li>

[`std::endl`](http://en.cppreference.com/w/cpp/io/manip/endl) is a special **I/O stream manipulator** object, also defined in file `iostream`. Inserting a manipulator into a stream changes the state of the stream.

> [`std:：endl`](http://en.cppreference.com/w/cpp/io/manip/endl)是一个特殊的**I/O流操纵器**对象，也在文件“iostream”中定义。将操纵器插入流中会更改流的状态。

The stream manipulator `std::endl` does two things: first it inserts the end-of-line character and then it flushes the stream buffer to force the text to show up on the console. This ensures that the data inserted into the stream actually appear on your console. (Stream data is usually stored in a buffer and then "flushed" in batches unless you force a flush immediately.)

> 流操纵器“std:：endl”做两件事：首先插入行尾字符，然后刷新流缓冲区以强制文本显示在控制台上。这样可以确保插入流中的数据实际显示在控制台上。（流数据通常存储在缓冲区中，然后分批“刷新”，除非您立即强制刷新。）
An alternate method that avoids the flush is:

```cpp
std::cout << "Hello World!\n";

```



where `\n` is the **character escape sequence** for the newline character.

> 其中“\n”是换行符的**字符转义序列**。
</li>
<li>

The semicolon (`;`) notifies the compiler that a statement has ended. All C++ statements and class definitions require an ending/terminating semicolon.

> 分号（`；`）通知编译器语句已结束。所有C++语句和类定义都需要以分号结尾/终止。
</li>



## Comments



A **comment** is a way to put arbitrary text inside source code without having the C++ compiler interpret it with any functional meaning. Comments are used to give insight into the design or method of a program.

> **注释**是一种将任意文本放入源代码中的方法，而无需C++编译器对其进行任何函数意义的解释。注释用于深入了解程序的设计或方法。

There are two types of comments in C++:

### Single-Line Comments


The double forward-slash sequence `//` will mark all text until a newline as a comment:

> 双正斜杠序列“//”将标记所有文本，直到换行作为注释：

```cpp
int main()
{
   // This is a single-line comment.
   int a;  // this also is a single-line comment
   int i;  // this is another single-line comment
}

```

### C-Style/Block Comments


The sequence `/*` is used to declare the start of the comment block and the sequence `*/` is used to declare the end of comment. All text between the start and end sequences is interpreted as a comment, even if the text is otherwise valid C++ syntax. These are sometimes called "C-style" comments, as this comment syntax is inherited from C++'s predecessor language, C:

> 序列“/*”用于声明注释块的开始，序列“*/”用于声明评论的结束。起始序列和结束序列之间的所有文本都被解释为注释，即使该文本在其他方面是有效的C++语法。这些注释有时被称为“C风格”注释，因为这种注释语法继承自C++的前代语言C:

```cpp
int main()
{
   /*
    *  This is a block comment.
    */
   int a;
}

```


In any block comment, you can write anything you want. When the compiler encounters the symbol `*/`, it terminates the block comment:

> 在任何区块评论中，你可以写任何你想要的东西。当编译器遇到符号“*/”时，它终止块注释：

```cpp
int main()
{
   /* A block comment with the symbol /*
      Note that the compiler is not affected by the second /*
      however, once the end-block-comment symbol is reached,
      the comment ends.
   */
   int a;
}

```


The above example is valid C++ (and C) code. However, having additional `/*` inside a block comment might result in a warning on some compilers.

> 上面的例子是有效的C++（和C）代码。但是，在块注释中包含额外的“/*”可能会对某些编译器产生警告。


Block comments can also start and end **within** a single line. For example:

> 阻止注释也可以在一行**内开始和结束**。例如：

```cpp
void SomeFunction(/* argument 1 */ int a, /* argument 2 */ int b);

```

### Importance of Comments

As with all programming languages, comments provide several benefits:

- Explicit documentation of code to make it easier to read/maintain
- Explanation of the purpose and functionality of code
- Details on the history or reasoning behind the code

- Placement of copyright/licenses, project notes, special thanks, contributor credits, etc. directly in the source code.

> -在源代码中直接放置版权/许可证、项目说明、特别感谢、贡献者信用等。

However, comments also have their downsides:

- They must be maintained to reflect any changes in the code
- Excessive comments tend to make the code **less** readable


The need for comments can be reduced by writing clear, self-documenting code. A simple example is the use of explanatory names for variables, functions, and types. Factoring out logically related tasks into discrete functions goes hand-in-hand with this.

> 通过编写清晰的、自我文档化的代码，可以减少对注释的需求。一个简单的例子是对变量、函数和类型使用解释性名称。将逻辑相关的任务分解为离散函数与此密切相关。

### Comment markers used to disable code


During development, comments can also be used to quickly disable portions of code without deleting it. This is often useful for testing or debugging purposes, but is not good style for anything other than temporary edits. This is often referred to as “commenting out”.

> 在开发过程中，注释也可以用来快速禁用代码的某些部分而不删除它。这通常对测试或调试很有用，但对于临时编辑之外的任何事情来说都不是一种好的风格。这通常被称为“评论”。


Similarly, keeping old versions of a piece of code in a comment for reference purposes is frowned upon, as it clutters files while offering little value compared to exploring the code's history via a versioning system.

> 类似地，在注释中保留一段代码的旧版本以供参考是不受欢迎的，因为它会弄乱文件，而与通过版本控制系统探索代码的历史相比，它几乎没有什么价值。



## The standard C++ compilation process


Executable C++ program code is usually produced by a compiler.


A **compiler** is a program that translates code from a programming language into another form which is (more) directly executable for a computer. Using a compiler to translate code is called **compilation.**

> **编译器**是一种将代码从编程语言翻译成另一种形式的程序，这种形式（更）可直接在计算机上执行。使用编译器翻译代码称为**编译**


C++ inherits the form of its compilation process from its "parent" language, C. Below is a list showing the four major steps of compilation in C++:

> C++从其“父”语言C继承了其编译过程的形式。下面列出了C++编译的四个主要步骤：


1. The C++ preprocessor copies the contents of any included header files into the source code file, generates macro code, and replaces symbolic constants defined using #define with their values.

> 1.C++预处理器将任何包含的头文件的内容复制到源代码文件中，生成宏代码，并将使用#define定义的符号常量替换为它们的值。

1. The expanded source code file produced by the C++ preprocessor is compiled into assembly language appropriate for the platform.

> 1.将C++预处理器生成的扩展源代码文件编译成适合该平台的汇编语言。

1. The assembler code generated by the compiler is assembled into appropriate object code for the platform.

> 1.由编译器生成的汇编代码被汇编成适合平台的目标代码。

1. The object code file generated by the assembler is linked together with the object code files for any library functions used to produce an executable file.

> 1.由汇编器生成的目标代码文件与用于生成可执行文件的任何库函数的对象代码文件链接在一起。


- Note: some compiled code is linked together, but not to create a final program. Usually, this "linked" code can also be packaged into a format that can be used by other programs. This "bundle of packaged, usable code" is what C++ programmers refer to as a **library.**

> -注意：一些编译后的代码是链接在一起的，但不能创建最终程序。通常，这种“链接”的代码也可以打包成其他程序可以使用的格式。这个“打包的、可用的代码包”就是C++程序员所说的**库**


Many C++ compilers may also merge or un-merge certain parts of the compilation process for ease or for additional analysis. Many C++ programmers will use different tools, but all of the tools will generally follow this generalized process when they are involved in the production of a program.

> 许多C++编译器也可以合并或取消合并编译过程的某些部分，以便于进行额外的分析。许多C++程序员会使用不同的工具，但当他们参与程序的生产时，所有的工具通常都会遵循这个通用的过程。


The link below extends this discussion and provides a nice graphic to help.

> 下面的链接扩展了这个讨论，并提供了一个很好的图形来提供帮助。

[1]: [http://faculty.cs.niu.edu/~mcmahon/CS241/Notes/compile.html](http://faculty.cs.niu.edu/%7Emcmahon/CS241/Notes/compile.html)

> [1] ：[http://faculty.cs.niu.edu/~mcmahon/CS241/Notes/compile.html](http://faculty.cs.niu.edu/%7Emcmahon/CS241/Notes/compile.html)



## Function



A **function** is a unit of code that represents a sequence of statements.

> **函数**是表示一系列语句的代码单元。


Functions can accept **arguments** or values and **return** a single value (or not). To use a function, a **function call** is used on argument values and the use of the function call itself is replaced with its return value.

> 函数可以接受**参数**或值，并且**可以返回**单个值（也可以不返回）。若要使用函数，请对参数值使用**函数调用**，并将函数调用本身的使用替换为其返回值。


Every function has a **type signature** -- the types of its arguments and the type of its return type.

> 每个函数都有一个**类型的签名**——参数的类型和返回类型的类型。


Functions are inspired by the concepts of the procedure and the mathematical function.

> 函数的灵感来源于过程和数学函数的概念。


- Note: C++ functions are essentially procedures and do not follow the exact definition or rules of mathematical functions.

> -注意：C++函数本质上是过程，不遵循数学函数的确切定义或规则。


Functions are often meant to perform a specific task. and can be called from other parts of a program. A function must be declared and defined before it is called elsewhere in a program.

> 函数通常用于执行特定任务。并且可以从程序的其他部分调用。在程序的其他地方调用函数之前，必须声明和定义函数。


- Note: popular function definitions may be hidden in other included files (often for convenience and reuse across many files). This is a common use of header files.

> -注意：流行的函数定义可能隐藏在其他包含的文件中（通常是为了方便和在许多文件中重用）。这是头文件的常见用法。

### Function Declaration


A **function declaration** is declares the existence of a function with its name and type signature to the compiler. The syntax is as the following:

> **函数声明**向编译器声明函数的存在及其名称和类型签名。语法如下：

```cpp
int add2(int i); // The function is of the type (int) -> (int)

```


In the example above, the `int add2(int i)` function declares the following to the compiler:

> 在上面的示例中，“int add2（int i）”函数向编译器声明以下内容：

- The **return type** is `int`.
- The **name** of the function is `add2`.
<li>The **number of arguments** to the function is 1:
<ul>
- The first argument is of the type `int`.
- The first argument will be referred to in the function's contents by the name `i`.


The argument name is optional; the declaration for the function could also be the following:

> 参数名称是可选的；函数的声明也可以是以下内容：

```cpp
int add2(int); // Omitting the function arguments' name is also permitted.

```


Per the **one-definition rule**, a function with a certain type signature can only be declared or defined once in an entire C++ code base visible to the C++ compiler. In other words, functions with a specific type signature cannot be re-defined -- they must only be defined once. Thus, the following is not valid C++:

> 根据**一个定义规则**，在C++编译器可见的整个C++代码库中，具有特定类型签名的函数只能声明或定义一次。换句话说，具有特定类型签名的函数不能重新定义——它们只能定义一次。因此，以下内容不是有效的C++：

```cpp
int add2(int i);  // The compiler will note that add2 is a function (int) -> int
int add2(int j);  // As add2 already has a definition of (int) -> int, the compiler
                  // will regard this as an error.

```


If a function returns nothing, its return type is written as `void`. If it takes no parameters, the parameter list should be empty.

> 如果函数不返回任何内容，则其返回类型将写为“void”。如果不包含任何参数，则参数列表应为空。

```cpp
void do_something(); // The function takes no parameters, and does not return anything.
                     // Note that it can still affect variables it has access to.

```

### Function Call


A function can be called after it has been declared. For example, the following program calls `add2` with the value of `2` within the function of `main`:

> 函数可以在声明后调用。例如，以下程序在“main”的函数中调用值为“2”的“add2”：

```cpp
#include <iostream>

int add2(int i);    // Declaration of add2

// Note: add2 is still missing a DEFINITION.
// Even though it doesn't appear directly in code,
// add2's definition may be LINKED in from another object file.

int main()
{
    std::cout << add2(2) << "\n";  // add2(2) will be evaluated at this point,
                                   // and the result is printed.
    return 0;  
}

```

Here, `add2(2)` is the syntax for a function call.

### Function Definition


A **function definition*** is similar to a declaration, except it also contains the code that is executed when the function is called within its body.

> **函数定义***类似于声明，只是它还包含在函数体内调用函数时执行的代码。

An example of a function definition for `add2` might be:

```cpp
int add2(int i)       // Data that is passed into (int i) will be referred to by the name i
{                     // while in the function's curly brackets or "scope."
                    
    int j = i + 2;    // Definition of a variable j as the value of i+2.
    return j;         // Returning or, in essence, substitution of j for a function call to
                      // add2.
}

```

### Function Overloading


You can create multiple functions with the same name but different parameters.

> 您可以创建多个具有相同名称但不同参数的函数。

```cpp
int add2(int i)           // Code contained in this definition will be evaluated
{                         // when add2() is called with one parameter.
    int j = i + 2;
    return j;
}

int add2(int i, int j)    // However, when add2() is called with two parameters, the
{                         // code from the initial declaration will be overloaded,
    int k = i + j + 2 ;   // and the code in this declaration will be evaluated
    return k;             // instead.
}

```


Both functions are called by the same name `add2`, but the actual function that is called depends directly on the amount and type of the parameters in the call. In most cases, the C++ compiler can compute which function to call. In some cases, the type must be explicitly stated.

> 两个函数都用相同的名称“add2”调用，但实际调用的函数直接取决于调用中参数的数量和类型。在大多数情况下，C++编译器可以计算调用哪个函数。在某些情况下，必须明确说明类型。

### Default Parameters


Default values for function parameters can only be specified in function declarations.

> 函数参数的默认值只能在函数声明中指定。

```cpp
int multiply(int a, int b = 7); // b has default value of 7.
int multiply(int a, int b)
{
    return a * b;               // If multiply() is called with one parameter, the
}                               // value will be multiplied by the default, 7.

```


In this example, `multiply()` can be called with one or two parameters. If only one parameter is given, `b` will have default value of 7. Default arguments must be placed in the latter arguments of the function. For example:

> 在本例中，可以使用一个或两个参数调用“multiply（）”。如果只给定一个参数，则“b”的默认值为7。默认参数必须放在函数的后一个参数中。例如：

```cpp
int multiply(int a = 10, int b = 20); // This is legal 
int multiply(int a = 10, int b);      // This is illegal since int a is in the former

```

### Special Function Calls - Operators


There exist special function calls in C++ which have different syntax than `name_of_function(value1, value2, value3)`. The most common example is that of operators.

> C++中存在一些特殊的函数调用，它们的语法与“name_of_function（value1，value2，value3）”不同。最常见的例子是运算符。


Certain special character sequences that will be reduced to function calls by the compiler, such as `!`, `+`, `-`, `*`, `%`, and `<<` and many more. These special characters are normally associated with non-programming usage or are used for aesthetics (e.g. the `+` character is commonly recognized as the addition symbol both within C++ programming as well as in  elementary math).

> 某些特殊字符序列将被编译器简化为函数调用，如“！”、“+”、“-”、“*”、“%”，以及“<<”等等。这些特殊字符通常与非编程用法有关，或用于美观（例如，在C++编程和初等数学中，“+”字符通常被认为是加法符号）。


C++ handles these character sequences with a special syntax; but, in essence, each occurrence of an operator is reduced to a function call. For example, the following C++ expression:

> C++用一种特殊的语法处理这些字符序列；但是，本质上，运算符的每次出现都被简化为一个函数调用。例如，以下C++表达式：

```cpp
3+3

```

is equivalent to the following function call:

```cpp
operator+(3, 3)

```

All operator function names start with `operator`.


While in C++'s immediate predecessor, C, operator function names cannot be assigned different meanings by providing additional definitions with different type signatures, in C++, this is valid. "Hiding" additional function definitions under one unique function name is referred to as **operator overloading** in C++, and is a relatively common, but not universal, convention in C++.

> 虽然在C++的直接前身C中，不能通过提供具有不同类型签名的附加定义来为运算符函数名赋予不同的含义，但在C++中，这是有效的。在C++中，将额外的函数定义“隐藏”在一个唯一的函数名下被称为**运算符重载**，这在C++中是一种相对常见但不通用的约定。



## Visibility of function prototypes and declarations



In C++, code must be declared or defined before usage. For example, the following produces a compile time error:

> 在C++中，代码必须在使用前声明或定义。例如，以下操作会产生编译时错误：

```cpp
int main()
{
  foo(2); // error: foo is called, but has not yet been declared
}

void foo(int x) // this later definition is not known in main
{
}

```


There are two ways to resolve this: putting either the definition or declaration of `foo()` before its usage in `main()`. Here is one example:

> 有两种方法可以解决这个问题：将“foo（）”的定义或声明放在“main（）”中使用之前。以下是一个示例：

```cpp
void foo(int x) {}  //Declare the foo function and body first

int main()
{
  foo(2); // OK: foo is completely defined beforehand, so it can be called here.
}

```


However it is also possible to "forward-declare" the function by putting only a "prototype" declaration before its usage and then defining the function body later:

> 然而，也可以通过在函数使用之前只放一个“原型”声明，然后在之后定义函数体来“前向声明”函数：

```cpp
void foo(int);  // Prototype declaration of foo, seen by main
                // Must specify return type, name, and argument list types
int main()
{
  foo(2); // OK: foo is known, called even though its body is not yet defined
}

void foo(int x) //Must match the prototype
{
    // Define body of foo here
}

```


The prototype must specify the return type (`void`), the name of the function (`foo`), and the argument list variable types (`int`), but the [names of the arguments are NOT required](https://stackoverflow.com/questions/5234169/why-do-function-prototypes-include-parameter-names-when-theyre-not-required).

> 原型必须指定返回类型（“void”）、函数名称（“foo”）和参数列表变量类型（“int”），但[参数名称不是必需的](https://stackoverflow.com/questions/5234169/why-do-function-prototypes-include-parameter-names-when-theyre-not-required)。


One common way to integrate this into the organization of source files is to make a header file containing all of the prototype declarations:

> 将其集成到源文件组织中的一种常见方法是制作一个包含所有原型声明的头文件：

```cpp
// foo.h
void foo(int); // prototype declaration

```

and then provide the full definition elsewhere:

```cpp
// foo.cpp --> foo.o
#include "foo.h" // foo's prototype declaration is "hidden" in here
void foo(int x) { } // foo's body definition

```


and then, once compiled, link the corresponding object file `foo.o` into the compiled object file where it is used in the linking phase, `main.o`:

> 然后，一旦编译，将相应的对象文件“foo.o”链接到编译的对象文件中，在链接阶段使用该文件“main.o”：

```cpp
// main.cpp --> main.o
#include "foo.h" // foo's prototype declaration is "hidden" in here
int main() { foo(2); } // foo is valid to call because its prototype declaration was beforehand.
// the prototype and body definitions of foo are linked through the object files

```


An “unresolved external symbol” error occurs when the function **prototype** and **call** exist, but the function **body** is not defined. These can be trickier to resolve as the compiler won't report the error until the final linking stage, and it doesn't know which line to jump to in the code to show the error.

> 当函数**原型**和**调用**存在，但函数**主体**未定义时，会出现“未解析的外部符号”错误。这些问题可能更难解决，因为编译器直到最后一个链接阶段才会报告错误，而且它不知道应该跳到代码中的哪一行来显示错误。



## Preprocessor



The preprocessor is an important part of the [compiler.](http://stackoverflow.com/documentation/c%2B%2B/8209/the-c-compilation-process#t=201702151617103019866)

> 预处理器是[编译器]的重要组成部分(http://stackoverflow.com/documentation/c%2B%2B/8209/the-c编译过程#t=201702151617103019866）


It edits the source code, cutting some bits out, changing others, and adding other things.

> 它编辑源代码，删除一些部分，更改其他部分，并添加其他内容。


In source files, we can include preprocessor directives. These directives tells the preprocessor to perform specific actions. A directive starts with a # on a new line.

> 在源文件中，我们可以包含预处理器指令。这些指令告诉预处理器执行特定的操作。指令在新行中以#开头。
Example:

```cpp
#define ZERO 0

```

The first preprocessor directive you will meet is probably the

```cpp
#include <something>

```


directive. What it does is takes all of `something` and inserts it in your file where the directive was. The [hello world](http://stackoverflow.com/documentation/c%2B%2B/206/getting-started-with-c-language/774/hello-world) program starts with the line

> 指令。它所做的是获取所有“某物”，并将其插入指令所在的文件中。[你好世界](http://stackoverflow.com/documentation/c%2B%2B/206/getting-started-with-c-language/774/helloworld）程序以以下行开始

```cpp
#include <iostream>

```


This line adds the [functions](http://stackoverflow.com/documentation/c%2B%2B/206/getting-started-with-c-language/25460/function) and objects that let you use the standard input and output.

> 此行添加了[函数](http://stackoverflow.com/documentation/c%2B%2B/206/getting-started-with-c-language/25460/函数）和允许您使用标准输入和输出的对象。


The C language, which also uses the preprocessor, does not have as many [header files](http://stackoverflow.com/documentation/c%2B%2B/7211/header-files#t=20170215162302552333) as the C++ language, but in C++ you can use all the C header files.

> 同样使用预处理器的C语言没有那么多[头文件](http://stackoverflow.com/documentation/c%2B%2B/7211/header-files#t=20170215162302552333）作为C++语言，但在C++中，您可以使用所有的C头文件。

The next important directive is probably the

```cpp
#define something something_else

```


directive. This tells the preprocessor that as it goes along the file, it should replace every occurrence of `something` with `something_else`. It can also make things similar to functions, but that probably counts as advanced C++.

> 指令。这告诉预处理器，当它处理文件时，它应该将每次出现的“something”替换为“something_else”。它也可以制作类似于函数的东西，但这可能算是高级C++。


The `something_else` is not needed, but if you define `something` as nothing, then outside preprocessor directives, all occurrences of `something` will vanish.

> 不需要“something_else”，但如果将“something”定义为“nothing”，则在预处理器指令之外，所有出现的“somethings”都将消失。


This actually is useful, because of the `#if`,`#else` and `#ifdef` directives. The format for these would be the following:

> 这实际上很有用，因为有“#if”、“#else”和“#ifdef”指令。其格式如下：

```cpp
#if something==true
//code
#else
//more code
#endif

#ifdef thing_that_you_want_to_know_if_is_defined
//code
#endif

```


These directives insert the code that is in the true bit, and deletes the false bits. this can be used to have bits of code that are only included on certain operating systems, without having to rewrite the whole code.

> 这些指令插入真位中的代码，并删除假位。这可以用于只包含在某些操作系统上的代码位，而不必重写整个代码。



#### Remarks



The 'Hello World' program is a common example that can be simply used to check compiler and library presence. It uses the C++ standard library, with `std::cout` from `<iostream>`, and has only one file to compile, minimizing the chance of possible user error during compilation.

> “Hello World”程序是一个常见的示例，可以简单地用于检查编译器和库的存在。它使用C++标准库，带有“＜iostream＞”中的“std:：cout”，并且只有一个文件要编译，从而最大限度地减少了编译过程中可能出现的用户错误。


The process for compiling a C++ program inherently differs between compilers and operating systems. The topic [Compiling and Building](https://stackoverflow.com/documentation/c%2B%2B/4708/compiling-and-building) contains the details about how to compile C++ code on different platforms for a variety of compilers.

> 编译C++程序的过程在编译器和操作系统之间本质上是不同的。主题[编译与构建](https://stackoverflow.com/documentation/c%2B%2B/4708/compiling-和构建）包含了如何在不同平台上为各种编译器编译C++代码的详细信息。

