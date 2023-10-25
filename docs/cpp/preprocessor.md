---
tip: translate by baidu@2023-10-25 08:29:46
---
---
metaTitle: "C++ | Preprocessor"
description: "Include Guards, Conditional logic and cross-platform handling, Macros, X-macros, Preprocessor error messages , Predefined macros, #pragma once, Preprocessor Operators"
---

# Preprocessor



The C preprocessor is a simple text parser/replacer that is run before the actual compilation of the code. Used to extend and ease the use of the C (and later C++) language, it can be used for:

> C预处理器是一个简单的文本解析器/替换程序，在实际编译代码之前运行。用于扩展和简化C（以及后来的C++）语言的使用，它可以用于：

a. **Including other files** using `#include`

b. **Define a text-replacement macro** using `#define`

c. **Conditional Compilation** using`#if` `#ifdef`


d. **Platform/Compiler specific logic** (as an extension of conditional compilation)

> d.**特定于平台/编译器的逻辑**（作为条件编译的扩展）



## Include Guards



A header file may be included by other header files.   A source file (compilation unit) that includes multiple headers may therefore, indirectly, include some headers more than once.   If such a header file that is included more than once contains definitions, the compiler (after preprocessing) detects a violation of the One Definition Rule (e.g. §3.2 of the 2003 C++ standard) and therefore issues a diagnostic and compilation fails.

> 头文件可以包括在其他头文件中。因此，包含多个标头的源文件（编译单元）可能会间接地多次包含某些标头。如果包含一次以上的头文件包含定义，编译器（在预处理后）会检测到违反“一个定义规则”（例如，2003 C++标准的§3.2），因此会发出诊断，编译失败。


Multiple inclusion is prevented using "include guards", which are sometimes also known as header guards or macro guards.    These are implemented using the preprocessor `#define`, `#ifndef`, `#endif` directives.

> 使用“包含保护”可以防止多重包含，有时也称为标头保护或宏保护。这些是使用预处理器“#define”、“#ifndef”和“#endif”指令实现的。

e.g.

```cpp
// Foo.h
#ifndef FOO_H_INCLUDED 
#define FOO_H_INCLUDED

class Foo    //  a class definition
{
};

#endif

```


The key advantage of using include guards is that they will work with all standard-compliant compilers and preprocessors.

> 使用include保护的主要优点是，它们可以与所有符合标准的编译器和预处理器一起工作。


However, include guards also cause some problems for developers, as it is necessary to ensure the macros are unique within all headers used in a project.  Specifically, if two (or more) headers use `FOO_H_INCLUDED` as their include guard, the first of those headers included in a compilation unit will effectively prevent the others from being included.    Particular challenges are introduced if a project uses a number of third-party libraries with header files that happen to use include guards in common.

> 但是，包含保护也会给开发人员带来一些问题，因为有必要确保宏在项目中使用的所有标头中都是唯一的。具体来说，如果两个（或多个）标头使用“FOO_H_include”作为其包含保护，则编译单元中包含的第一个标头将有效地防止其他标头被包含。如果一个项目使用许多第三方库，而这些库的头文件恰好使用共同的include保护，则会带来特殊的挑战。


It is also necessary to ensure that the macros used in include guards do not conflict with any other macros defined in header files.

> 还必须确保包含保护中使用的宏不会与头文件中定义的任何其他宏冲突。


[Most C++ implementations](https://en.wikipedia.org/wiki/Pragma_once#Portability) also support the `#pragma once` directive which ensures the file is only included once within a single compilation. This is a [**de facto** standard](https://en.wikipedia.org/wiki/De_facto_standard) directive, but it is not part of any ISO C++ standard. For example:

> [大多数C++实现](https://en.wikipedia.org/wiki/Pragma_once#Portability)还支持“#pragma once”指令，该指令确保文件在一次编译中只包含一次。这是[**事实上的**标准](https://en.wikipedia.org/wiki/De_facto_standard)指令，但它不是任何ISO C++标准的一部分。例如：

```cpp
// Foo.h
#pragma once

class Foo
{
};

```


While `#pragma once` avoids some problems associated with include guards, a `#pragma` - by definition in the standards - is inherently a compiler-specific hook, and will be silently ignored by compilers that don't support it.   Projects which use `#pragma once` are more difficult to port to compilers that don't support it.

> 虽然“#pragma once”避免了与include guard相关的一些问题，但根据标准中的定义，“#pragma”本质上是编译器特定的挂钩，不支持它的编译器会默默忽略它。使用“#practica once”的项目更难移植到不支持它。


A number of coding guidelines and assurance standards for C++ specifically discourage any use of the preprocessor other than to `#include` header files or for the purposes of placing include guards in headers.

> 许多C++的编码准则和保证标准特别禁止使用预处理器，而不是“#include”头文件或在头文件中放置include保护。



## Conditional logic and cross-platform handling



In a nutshell, conditional pre-processing logic is about making code-logic available or unavailable for compilation using macro definitions.

> 简而言之，条件预处理逻辑是关于使用宏定义使代码逻辑可用于或不可用于编译。

Three prominent use-cases are:


- different **app profiles** (e.g. debug, release, testing, optimised) that can be candidates of the same app (e.g. with extra logging).

> -不同的**应用程序配置文件**（例如，调试、发布、测试、优化），可以是同一应用程序的候选者（例如，具有额外的日志记录）。
- **cross-platform compiles** - single code-base, multiple compilation platforms.

- utilising a common code-base for multiple **application versions** (e.g. Basic, Premium and Pro versions of a software) - with slightly different features.

> -为多个**应用程序版本**（例如软件的Basic、Premium和Pro版本）使用通用代码库，功能略有不同。


**Example a:** A cross-platform approach for removing files (illustrative):

> **示例a:**用于删除文件的跨平台方法（示例）：

```cpp
#ifdef _WIN32
#include <windows.h> // and other windows system files
#endif
#include <cstdio>

bool remove_file(const std::string &path) 
{
#ifdef _WIN32
  return DeleteFile(path.c_str());
#elif defined(_POSIX_VERSION) || defined(__unix__)
  return (0 == remove(path.c_str()));
#elif defined(__APPLE__)
  //TODO: check if NSAPI has a more specific function with permission dialog
  return (0 == remove(path.c_str()));
#else 
#error "This platform is not supported"
#endif
}

```


Macros like `_WIN32`, `__APPLE__` or `__unix__` are normally predefined by corresponding implementations.

> 像`_WIN32`、`__APPLE_`或`__unix__`这样的宏通常由相应的实现预定义。

**Example b:** Enabling additional logging for a debug build:

```cpp
void s_PrintAppStateOnUserPrompt()
{
    std::cout << "--------BEGIN-DUMP---------------\n"
              << AppState::Instance()->Settings().ToString() << "\n"
#if ( 1 == TESTING_MODE ) //privacy: we want user details only when testing
              << ListToString(AppState::UndoStack()->GetActionNames())
              << AppState::Instance()->CrntDocument().Name() 
              << AppState::Instance()->CrntDocument().SignatureSHA() << "\n"
#endif
              << "--------END-DUMP---------------\n"
}

```


**Example c:** Enable a premium feature in a separate product build (note: this is illustrative. it is often a better idea to allow a feature to be unlocked without the need to reinstall an application)

> **示例c:**在单独的产品构建中启用高级功能（注意：这只是说明性的。允许解锁功能而无需重新安装应用程序通常是更好的主意）

```cpp
void MainWindow::OnProcessButtonClick()
{
#ifndef _PREMIUM
    CreatePurchaseDialog("Buy App Premium", "This feature is available for our App Premium users. Click the Buy button to purchase the Premium version at our website");
    return;
#endif
    //...actual feature logic here
}

```

**Some common tricks:**

**Defining symbols at invocation time:**


The preprocessor can be called with predefined symbols (with optional initialisation). For example this command (`gcc -E` runs only the preprocessor)

> 预处理器可以用预定义的符号调用（可选初始化）。例如，此命令（`gcc-E`仅运行预处理器）

```cpp
gcc -E -DOPTIMISE_FOR_OS_X -DTESTING_MODE=1 Sample.cpp

```


processes Sample.cpp in the same way as it would if `#define OPTIMISE_FOR_OS_X` and `#define TESTING_MODE 1` were added to the top of Sample.cpp.

> 以与将“#define OPTIMISE_FOR_OS_X”和“#defin TESTING_MODE 1”添加到Sample.cpp顶部时相同的方式处理Sample.cpp。

**Ensuring a macro is defined:**


If a macro isn't defined and its value is compared or checked, the preprocessor almost always silently assumes the value to be `0`. There are a few ways to work with this. One approach is to assume that the default settings are represented as 0, and any changes (e.g. to the app build profile) needs to be explicitly done (e.g. ENABLE_EXTRA_DEBUGGING=0 by default, set -DENABLE_EXTRA_DEBUGGING=1 to override). Another approach is make all definitions and defaults explicit. This can be achieved using a combination of `#ifndef` and `#error` directives:

> 如果没有定义宏，并且比较或检查其值，则预处理器几乎总是默认值为“0”。有几种方法可以解决这个问题。一种方法是假设默认设置表示为0，并且任何更改（例如对应用程序构建配置文件的更改）都需要显式完成（例如，默认情况下ENABLE_EXTRA_DEBUGGING=0，设置-DENABLE_EXTRA_DEBUUGING=1以覆盖）。另一种方法是明确所有定义和默认值。这可以使用“#ifndef”和“#error”指令的组合来实现：

```cpp
#ifndef (ENABLE_EXTRA_DEBUGGING)
// please include DefaultDefines.h if not already included.
#    error "ENABLE_EXTRA_DEBUGGING is not defined"
#else
#    if ( 1 == ENABLE_EXTRA_DEBUGGING )
  //code
#    endif
#endif

```



## Macros



Macros are categorized into two main groups: object-like macros and function-like macros. Macros are treated as a token substitution early in the compilation process. This means that large (or repeating) sections of code can be abstracted into a preprocessor macro.

> 宏分为两大类：类对象宏和类函数宏。在编译过程的早期，宏被视为令牌替换。这意味着可以将代码的大（或重复）部分抽象为预处理器宏。

```cpp
// This is an object-like macro
#define    PI         3.14159265358979

// This is a function-like macro.
// Note that we can use previously defined macros
// in other macro definitions (object-like or function-like)
// But watch out, its quite useful if you know what you're doing, but the
// Compiler doesnt know which type to handle, so using inline functions instead
// is quite recommended (But e.g. for Minimum/Maximum functions it is quite useful)
#define    AREA(r)    (PI*(r)*(r))

// They can be used like this:
double pi_macro   = PI;
double area_macro = AREA(4.6);

```


The Qt library makes use of this technique to create a meta-object system by having the user declare the Q_OBJECT macro at the head of the user-defined class extending QObject.

> Qt库通过让用户在扩展QObject的用户定义类的头部声明Q_object宏，利用这种技术来创建元对象系统。


Macro names are usually written in all caps, to make them easier to differentiate from normal code.  This isn't a requirement, but is merely considered good style by many programmers.

> 宏名称通常是全大写的，以便更容易与普通代码区分开来。这不是一个要求，但许多程序员认为这只是一种好的风格。


When an object-like macro is encountered, it's expanded as a simple copy-paste operation, with the macro's name being replaced with its definition.  When a function-like macro is encountered, both its name and its parameters are expanded.

> 当遇到类似对象的宏时，它将展开为简单的复制粘贴操作，宏的名称将替换为其定义。当遇到类似函数的宏时，它的名称和参数都会展开。

```cpp
double pi_squared = PI * PI;
// Compiler sees:
double pi_squared = 3.14159265358979 * 3.14159265358979;

double area = AREA(5);
// Compiler sees:
double area = (3.14159265358979*(5)*(5))

```


Due to this, function-like macro parameters are often enclosed within parentheses, as in `AREA()` above.  This is to prevent any bugs that can occur during macro expansion, specifically bugs caused by a single macro parameter being composed of multiple actual values.

> 因此，类似函数的宏参数通常被括在括号内，如上面的“AREA（）”中所示。这是为了防止在宏扩展过程中出现任何错误，特别是由多个实际值组成的单个宏参数引起的错误。

```cpp
#define BAD_AREA(r) PI * r * r

double bad_area = BAD_AREA(5 + 1.6);
// Compiler sees:
double bad_area = 3.14159265358979 * 5 + 1.6 * 5 + 1.6;

double good_area = AREA(5 + 1.6);
// Compiler sees:
double good_area = (3.14159265358979*(5 + 1.6)*(5 + 1.6));

```


Also note that due to this simple expansion, care must be taken with the parameters passed to macros, to prevent unexpected side effects.  If the parameter is modified during evaluation, it will be modified each time it is used in the expanded macro, which usually isn't what we want.  This is true even if the macro encloses the parameters in parentheses to prevent expansion from breaking anything.

> 还要注意，由于这种简单的扩展，必须小心传递给宏的参数，以防止意外的副作用。如果在评估过程中修改了参数，则每次在展开的宏中使用该参数时都会对其进行修改，这通常不是我们想要的。即使宏将参数括在括号中以防止展开破坏任何内容，也是如此。

```cpp
int oops = 5;
double incremental_damage = AREA(oops++);
// Compiler sees:
double incremental_damage = (3.14159265358979*(oops++)*(oops++));

```


Additionally, macros provide no type-safety, leading to hard-to-understand errors about type mismatch.

> 此外，宏不提供类型安全性，导致难以理解的类型不匹配错误。


As programmers normally terminate lines with a semicolon, macros that are intended to be used as standalone lines are often designed to "swallow" a semicolon; this prevents any unintended bugs from being caused by an extra semicolon.

> 由于程序员通常用分号终止行，因此打算用作独立行的宏通常被设计为“吞下”分号；这样可以防止任何意外的错误由额外的分号引起。

```cpp
#define IF_BREAKER(Func) Func();

if (some_condition)
    // Oops.
    IF_BREAKER(some_func);
else
    std::cout << "I am accidentally an orphan." << std::endl;

```


In this example, the inadvertent double semicolon breaks the `if...else` block, preventing the compiler from matching the `else` to the `if`.  To prevent this, the semicolon is omitted from the macro definition, which will cause it to "swallow" the semicolon immediately following any usage of it.

> 在本例中，不经意的双分号打断了“if…”。。。else'块，阻止编译器将'else'与'if'匹配。为了防止这种情况，宏定义中省略了分号，这将导致它在使用分号后立即“吞下”分号。

```cpp
#define IF_FIXER(Func) Func()

if (some_condition)
    IF_FIXER(some_func);
else
    std::cout << "Hooray!  I work again!" << std::endl;

```


Leaving off the trailing semicolon also allows the macro to be used without ending the current statement, which can be beneficial.

> 去掉尾部分号还可以在不结束当前语句的情况下使用宏，这可能是有益的。

```cpp
#define DO_SOMETHING(Func, Param) Func(Param, 2)

// ...

some_function(DO_SOMETHING(some_func, 3), DO_SOMETHING(some_func, 42));

```


Normally, a macro definition ends at the end of the line.  If a macro needs to cover multiple lines, however, a backslash can be used at the end of a line to indicate this.  This backslash must be the last character in the line, which indicates to the preprocessor that the following line should be concatenated onto the current line, treating them as a single line.  This can be used multiple times in a row.

> 通常，宏定义结束于行的末尾。但是，如果宏需要覆盖多行，则可以在行的末尾使用反斜杠来表示这一点。这个反斜杠必须是行中的最后一个字符，这向预处理器指示应该将下一行连接到当前行，并将其视为一行。这可以连续使用多次。

```cpp
#define TEXT "I \
am \
many \
lines."

// ...

std::cout << TEXT << std::endl; // Output:   I am many lines.

```


This is especially useful in complex function-like macros, which may need to cover multiple lines.

> 这在类似宏的复杂函数中尤其有用，因为宏可能需要覆盖多行。

```cpp
#define CREATE_OUTPUT_AND_DELETE(Str) \
    std::string* tmp = new std::string(Str); \
    std::cout << *tmp << std::endl; \
    delete tmp;

// ...

CREATE_OUTPUT_AND_DELETE("There's no real need for this to use 'new'.")

```


In the case of more complex function-like macros, it can be useful to give them their own scope to prevent possible name collisions or to cause objects to be destroyed at the end of the macro, similar to an actual function.  A common idiom for this is **do while 0**, where the macro is enclosed in a **do-while** block.  This block is generally **not** followed with a semicolon, allowing it to swallow a semicolon.

> 在类似宏的更复杂的函数的情况下，为它们提供自己的作用域以防止可能的名称冲突或导致对象在宏结束时被破坏（类似于实际函数），这可能很有用。一个常见的习惯用法是**dowhile0**，其中宏包含在**dowhire**块中。这个块通常是**而不是**后面跟一个分号，允许它吞下一个分号。

```cpp
#define DO_STUFF(Type, Param, ReturnVar) do { \
    Type temp(some_setup_values); \
    ReturnVar = temp.process(Param); \
} while (0)

int x;
DO_STUFF(MyClass, 41153.7, x);

// Compiler sees:

int x;
do {
    MyClass temp(some_setup_values);
    x = temp.process(41153.7);
} while (0);

```


There are also variadic macros; similarly to variadic functions, these take a variable number of arguments, and then expand them all in place of a special "Varargs" parameter, `__VA_ARGS__`.

> 还有可变宏；与变参数函数类似，这些函数采用可变数量的参数，然后将它们全部展开，以代替特殊的“Varargs”参数`__VA_ARGS__`。

```cpp
#define VARIADIC(Param, ...) Param(__VA_ARGS__)

VARIADIC(printf, "%d", 8);
// Compiler sees:
printf("%d", 8);

```


Note that during expansion, `__VA_ARGS__` can be placed anywhere in the definition, and will be expanded correctly.

> 请注意，在展开过程中，`__VA_ARGS__`可以放置在定义中的任何位置，并且将正确展开。

```cpp
#define VARIADIC2(POne, PTwo, PThree, ...) POne(PThree, __VA_ARGS__, PTwo)

VARIADIC2(some_func, 3, 8, 6, 9);
// Compiler sees:
some_func(8, 6, 9, 3);

```


In the case of a zero-argument variadic parameter, different compilers will handle the trailing comma differently.  Some compilers, such as Visual Studio, will silently swallow the comma without any special syntax.  Other compilers, such as GCC, require you to place `##` immediately before `__VA_ARGS__`.  Due to this, it is wise to conditionally define variadic macros when portability is a concern.

> 在零参数可变参数的情况下，不同的编译器将以不同的方式处理尾部逗号。一些编译器，如Visual Studio，会在没有任何特殊语法的情况下静默地吞下逗号。其他编译器（如GCC）要求将“##”放在“__VA_ARGS__`”之前。因此，在考虑可移植性时，有条件地定义可变宏是明智的。

```cpp
// In this example, COMPILER is a user-defined macro specifying the compiler being used.

#if       COMPILER == "VS"
    #define VARIADIC3(Name, Param, ...) Name(Param, __VA_ARGS__)
#elif     COMPILER == "GCC"
    #define VARIADIC3(Name, Param, ...) Name(Param, ##__VA_ARGS__)
#endif /* COMPILER */

```



## X-macros



An idiomatic technique for generating repeating code structures at compile time.

> 在编译时生成重复代码结构的惯用技术。


An X-macro consists of two parts: the list, and the execution of the list.

> X宏由两部分组成：列表和列表的执行。

Example:

```cpp
#define LIST \
    X(dog)   \
    X(cat)   \
    X(racoon)

// class Animal {
//  public:
//    void say();
// };

#define X(name) Animal name;
LIST
#undef X

int main() {
#define X(name) name.say();
    LIST
#undef X

    return 0;
}

```

which is expanded by the preprocessor into the following:

```cpp
Animal dog;
Animal cat;
Animal racoon;

int main() {
    dog.say();
    cat.say();
    racoon.say();

    return 0;
}    

```


As lists become bigger (let's say, more than 100 elements), this technique helps to avoid excessive copy-pasting.

> 随着列表越来越大（比如说，超过100个元素），这种技术有助于避免过度的复制粘贴。


Source: [https://en.wikipedia.org/wiki/X_Macro](https://en.wikipedia.org/wiki/X_Macro)

> 来源：[https://en.wikipedia.org/wiki/X_Macro](https://en.wikipedia.org/wiki/X_Macro)


See also: [X-macros](http://stackoverflow.com/documentation/c/628/x-macros)

> 另请参阅：[X-macros](http://stackoverflow.com/documentation/c/628/x-macros)


If defining a seamingly irrelevant `X` before using `LIST` is not to your liking, you can pass a macro name as an argument as well:

> 如果在使用“LIST”之前定义一个完全不相关的“X”不符合您的喜好，您也可以传递一个宏名称作为参数：

```cpp
#define LIST(MACRO) \
    MACRO(dog) \
    MACRO(cat) \
    MACRO(racoon)

```


Now, you explicitly specify which macro should be used when expanding the list, e.g.

> 现在，您可以明确指定在展开列表时应使用的宏，例如。

```cpp
#define FORWARD_DECLARE_ANIMAL(name) Animal name;
LIST(FORWARD_DECLARE_ANIMAL)

```


If each invocation of the `MACRO` should take additional parameters - constant with respect to the list, variadic macros can be used

> 如果每次调用“MACRO”都需要额外的参数（相对于列表为常量），则可以使用可变宏

```cpp
//a walkaround for Visual studio
#define EXPAND(x) x

#define LIST(MACRO, ...) \
    EXPAND(MACRO(dog, __VA_ARGS__)) \
    EXPAND(MACRO(cat, __VA_ARGS__)) \
    EXPAND(MACRO(racoon, __VA_ARGS__))

```


The first argument is supplied by the `LIST`, while the rest is provided by the user in the `LIST` invocation. For example:

> 第一个参数由“LIST”提供，而其余参数由用户在“LIST’调用中提供。例如：

```cpp
#define FORWARD_DECLARE(name, type, prefix) type prefix##name;
LIST(FORWARD_DECLARE,Animal,anim_)
LIST(FORWARD_DECLARE,Object,obj_)

```

will expand to

```cpp
Animal anim_dog;
Animal anim_cat;
Animal anim_racoon;
Object obj_dog;
Object obj_cat;
Object obj_racoon;        

```



## Preprocessor error messages 



Compile errors can be generated using the preprocessor. This is useful for a number of reasons some of which include, notifying a user if they are on an unsupported platform or an unsupported compiler.

> 编译错误可以使用预处理器生成。这是有用的，原因有很多，其中一些原因包括，如果用户在不受支持的平台或不受欢迎的编译器上，则通知用户。

e.g. Return Error if gcc version is 3.0.0 or earlier.

```cpp
#if __GNUC__ < 3
#error "This code requires gcc > 3.0.0"
#endif

```

e.g. Return Error if compiling on an Apple computer.

```cpp
#ifdef __APPLE__
#error "Apple products are not supported in this release"
#endif

```



## Predefined macros



Predefined macros are those that the compiler defines (in contrast to those user defines in the source file). Those macros must not be re-defined or undefined by user.

> 预定义的宏是编译器定义的宏（与用户在源文件中定义的宏不同）。用户不得重新定义或未定义这些宏。

The following macros are predefined by the C++ standard:


- `__LINE__` contains the line number of the line this macro is used on, and can be changed by the `#line` directive.

> -“__LINE___”包含此宏所用行的行号，可以通过“#LINE”指令进行更改。

- `__FILE__` contains the filename of the file this macro is used in, and can be changed by the `#line` directive.

> -`__FILE__`包含此宏所用文件的文件名，可以通过`#line`指令进行更改。

- `__DATE__` contains date (in `"Mmm dd yyyy"` format) of the file compilation, where **Mmm** is formatted as if obtained by a call to `std::asctime()`.

> -`__DATE__`包含文件编译的日期（格式为`Mmm-dd-yyyy`），其中**Mmm**的格式就像通过调用`std:：asctime（）`获得的一样。
- `__TIME__` contains time (in `"hh:mm:ss"` format) of the file compilation.

- `__cplusplus` is defined by (conformant) C++ compilers while compiling C++ files. Its value is the standard version the compiler is **fully** conformant with, i.e. `199711L` for C++98 and C++03, `201103L` for C++11 and `201402L` for C++14 standard.

> -“__cplusplus”是由（一致的）C++编译器在编译C++文件时定义的。它的值是编译器**完全**符合的标准版本，即“199711L”用于C++98和C++03，“201103L”用于C++11，“201402L”用于C++14标准。


- `__STDC_HOSTED__` is defined to `1` if the implementation is **hosted**, or `0` if it is **freestanding**.

> -如果实现是**托管的**，则将`__STDC_HOSTED___`定义为`1'；如果实现是***独立的**，将`0`定义为。


- `__STDCPP_DEFAULT_NEW_ALIGNMENT__` contains a `size_t` literal, which is the alignment used for a call to alignment-unaware `operator new`.

> -`__STDCP_DEFAULT_NEW_ALIGNMENT__`包含一个`size_t`文字，它是用于调用不知道对齐方式的`operator NEW`的对齐方式。


Additionally, the following macros are allowed to be predefined by implementations, and may or may not be present:

> 此外，以下宏允许由实现预定义，并且可以存在也可以不存在：


- `__STDC__` has implementation-dependent meaning, and is usually defined only when compiling a file as C, to signify full C standard compliance.  (Or never, if the compiler decides not to support this macro.)

> -`__STDC_`具有与实现相关的含义，通常仅在将文件编译为C时定义，以表示完全符合C标准。（如果编译器决定不支持此宏，则决不支持。）


- `__STDC_VERSION__` has implementation-dependent meaning, and its value is usually the C version, similarly to how `__cplusplus` is the C++ version.  (Or is not even defined, if the compiler decides not to support this macro.)

> -`__STDC_VERSION__`具有依赖于实现的含义，其值通常是C版本，类似于`__cplusplus`是C++版本。（或者甚至没有定义，如果编译器决定不支持此宏。）

- `__STDC_MB_MIGHT_NEQ_WC__` is defined to `1`, if values of the narrow encoding of the basic character set might not be equal to the values of their wide counterparts (e.g. if `(uintmax_t)'x' != (uintmax_t)L'x'`)

> -如果基本字符集的窄编码值可能不等于其宽对应值（例如，如果`（uintmax_t）'x'！=（uintmax_t）L'x'`）

- `__STDC_ISO_10646__` is defined if `wchar_t` is encoded as Unicode, and expands to an integer constant in the form `yyyymmL`, indicating the latest Unicode revision supported.

> -如果`wchar_t`编码为Unicode，则定义`__STDC_ISO_10646__`，并扩展为`yyyymmL`形式的整数常量，表示支持的最新Unicode修订版。

- `__STDCPP_STRICT_POINTER_SAFETY__` is defined to `1`, if the implementation has **strict pointer safety** (otherwise it has **relaxed pointer safety**)

> -如果实现具有**严格的指针安全性**（否则具有**宽松的指针安全**），则将`__STDCP_STRICT_POINTER_SAFETY_`定义为`1'

- `__STDCPP_THREADS__` is defined to `1`, if the program can have more than one thread of execution (applicable to **freestanding implementation** — **hosted implementations** can always have more than one thread)

> -如果程序可以有多个执行线程（适用于**独立实现**--**托管实现**可以总是有多个线程），则将`__STDCP_THREADS__`定义为`1'


It is also worth mentioning `__func__`, which is not an macro, but a predefined function-local variable. It contains the name of the function it is used in, as a static character array in an implementation-defined format.

> 还值得一提的是“__func___”，它不是一个宏，而是一个预定义的函数局部变量。它包含所用函数的名称，作为实现定义格式的静态字符数组。


On top of those standard predefined macros, compilers can have their own set of predefined macros. One must refer to the compiler documentation to learn those. E.g.:

> 在这些标准的预定义宏之上，编译器可以拥有自己的一组预定义宏。必须参考编译器文档才能了解这些信息。例如：

- [gcc](https://gcc.gnu.org/onlinedocs/cpp/Predefined-Macros.html)
- [Microsoft Visual C++](https://msdn.microsoft.com/en-us/library/b0084kay.aspx#Anchor_2)
- [clang](http://clang.llvm.org/docs/LanguageExtensions.html#builtin-macros)
- [Intel C++ Compiler](https://software.intel.com/en-us/node/514528)

Some of the macros are just to query support of some feature:

```cpp
#ifdef __cplusplus // if compiled by C++ compiler
extern "C"{ // C code has to be decorated
   // C library header declarations here
}
#endif

```

Others are very useful for debugging:

```cpp
bool success = doSomething( /*some arguments*/ );
if( !success ){
    std::cerr << "ERROR: doSomething() failed on line " << __LINE__ - 2
              << " in function " << __func__ << "()"
              << " in file " << __FILE__
              << std::endl;
}

```

And others for trivial version control:

```cpp
int main( int argc, char *argv[] ){
    if( argc == 2 && std::string( argv[1] ) == "-v" ){
        std::cout << "Hello World program\n"
                  << "v 1.1\n" // I have to remember to update this manually
                  << "compiled: " << __DATE__ << ' ' << __TIME__ // this updates automagically
                  << std::endl;
    }
    else{
        std::cout << "Hello World!\n";
    }
}

```



## #pragma once



[Most, but not all, C++ implementations](https://en.wikipedia.org/wiki/Pragma_once#Portability) support the `#pragma once` directive which ensures the file is only included once within a single compilation. It is not part of any ISO C++ standard. For example:

> [大多数（但不是全部）C++实现](https://en.wikipedia.org/wiki/Pragma_once#Portability)支持“#pragmaonce”指令，该指令确保文件在一次编译中只包含一次。它不是任何ISO C++标准的一部分。例如：

```cpp
// Foo.h
#pragma once

class Foo
{
};

```


While `#pragma once` avoids some problems associated with [include guards](http://stackoverflow.com/documentation/c%2b%2b/1098/preprocessor/3525/include-guards), a `#pragma` - by definition in the standards - is inherently a compiler-specific hook, and will be silently ignored by compilers that don't support it. Projects which use `#pragma once` must be modified to be standard-compliant.

> 而“#pragma once”避免了与[include guards]相关的一些问题(http://stackoverflow.com/documentation/c%2b%2b/1098/preprocessor/3525/include-guards），“#pragma”（根据标准中的定义）本质上是编译器特定的挂钩，不支持它的编译器将忽略它。使用“#practica once”的项目必须修改为符合标准。


With some compilers - particularly those that employ [precompiled headers](https://en.wikipedia.org/wiki/Precompiled_header) - `#pragma once` can result in a considerable speedup of the compilation process.  Similarly, some preprocessors achieve speedup of compilation by tracking which headers have employed include guards.  The net benefit, when both `#pragma once` and include guards are employed, depends on the implementation and can be either an increase or decrease of compilation times.

> 对于某些编译器，尤其是那些使用[预编译头]的编译器(https://en.wikipedia.org/wiki/Precompiled_header)-“#pragmaonce”可以显著加快编译过程。类似地，一些预处理器通过跟踪哪些标头使用了包含保护来加快编译速度。当同时使用“#pragma once”和include保护时，净收益取决于实现，可以是增加或减少编译时间。

`#pragma once` combined with [include guards](http://stackoverflow.com/documentation/c%2b%2b/1098/preprocessor/3525/include-guards) was the recommended layout for header files when writing MFC based applications on windows, and was generated by Visual Studio’s `add class`, `add dialog`, `add windows` wizards.   Hence it is very common to find them combined in C++ Windows Applicants.



## Preprocessor Operators


`#` operator or stringizing operator is used to convert a Macro parameter to a string literal. It can only be used with the Macros having arguments.

```cpp
// preprocessor will convert the parameter x to the string literal x
#define PRINT(x) printf(#x "\n")

PRINT(This line will be converted to string by preprocessor);
// Compiler sees
printf("This line will be converted to string by preprocessor""\n");

```


Compiler concatenate two strings and the final `printf()` argument will be a string literal with newline character at its end.

> 编译器连接两个字符串，最后的“printf（）”参数将是一个末尾带有换行符的字符串文字。


Preprocessor will ignore the spaces before or after the macro argument. So below print statement will give us the same result.

> 预处理器将忽略宏参数前后的空格。所以下面的打印声明会给我们同样的结果。

```cpp
PRINT(   This line will be converted to string by preprocessor );

```


If the parameter of the string literal requires an escape sequence like before a double quote() it will automatically be inserted by the preprocessor.

> 如果字符串文字的参数需要一个转义序列，就像在双引号（）之前一样，它将由预处理器自动插入。

```cpp
PRINT(This "line" will be converted to "string" by preprocessor); 
// Compiler sees
printf("This \"line\" will be converted to \"string\" by preprocessor""\n");

```

`##` operator or Token pasting operator is used to concatenate two parameters or tokens of a Macro.

```cpp
// preprocessor will combine the variable and the x
#define PRINT(x) printf("variable" #x " = %d", variable##x)

int variableY = 15;
PRINT(Y);
//compiler sees
printf("variable""Y"" = %d", variableY);

```

and the final output will be

```cpp
variableY = 15

```



#### Remarks



Preprocessor statements are executed before your source files are handed to the compiler. They are capable of very low level conditional logic. Since preprocessor constructs (e.g. object-like macros) aren't typed like normal functions (the preprocessing step happens before compilation) the compiler cannot enforce type checks, they should therefore be carefully used.

> 预处理器语句在源文件交给编译器之前执行。它们能够实现非常低级别的条件逻辑。由于预处理器构造（例如类对象宏）不像普通函数那样类型化（预处理步骤发生在编译之前），编译器无法强制执行类型检查，因此应谨慎使用它们。

