---
tip: translate by baidu@2023-10-25 08:26:10
---
---
metaTitle: "C++ | Internationalization in C++"
description: "Understanding C++ string characteristics"
---

# Internationalization in C++



## Understanding C++ string characteristics


```cpp
#include <iostream>
#include <string>

int main()
{
    const char * C_String = "This is a line of text w";
    const char * C_Problem_String = "This is a line of text ኚ";
    std::string Std_String("This is a second line of text w");
    std::string Std_Problem_String("This is a second line of ϯϵxϯ ኚ");

    std::cout << "String Length: " << Std_String.length() << '\n';
    std::cout << "String Length: " << Std_Problem_String.length() << '\n';

    std::cout << "CString Length: " << strlen(C_String) << '\n';
    std::cout << "CString Length: " << strlen(C_Problem_String) << '\n';
    return 0;
}

```


Depending on platform (windows, OSX, etc) and compiler (GCC, MSVC, etc), this program **may fail to compile, display different values, or display the same values**.

> 根据平台（windows、OSX等）和编译器（GCC、MSVC等）的不同，此程序**可能无法编译、显示不同的值或显示相同的值**。

Example output under the Microsoft MSVC compiler:

> 
<p>String Length: 31<br />
String Length: 31<br />
CString Length: 24<br />
CString Length: 24</p>



This shows that under MSVC each of the extended-characters used is considered a single "character", and this platform fully supports internationalised languages.<br />

> 这表明，在MSVC下，使用的每个扩展字符都被视为一个“字符”，并且该平台完全支持国际化语言<br/>

**It should be noted however that this behaviour is unusual, these international characters are stored internally as Unicode and thus are actually several bytes long. **This may cause unexpected errors****

> **然而，应该注意的是，这种行为是不寻常的，这些国际字符以Unicode的形式存储在内部，因此实际上有几个字节长**这可能会导致意外错误****

Under the GNC/GCC compiler the program output is:

> 
<p>String Length: 31<br />
String Length: 36<br />
CString Length: 24<br />
CString Length: 26</p>



This example demonstrates that while the GCC compiler used on this (Linux) platform does support these extended-characters, it also uses (**correctly)** several bytes to store an individual character.<br />

> 此示例表明，虽然此（Linux）平台上使用的GCC编译器确实支持这些扩展字符，但它也使用（**正确）**几个字节来存储单个字符<br/>

In this case the use of Unicode characters is possible, but the programmer must take great care in remembering that the length of a "string" in this scenario is the ****length in bytes****, not the ****length in readable characters****.

> 在这种情况下，可以使用Unicode字符，但程序员必须非常小心地记住，在这种情况中，“字符串”的长度是以字节为单位的***长度，而不是以可读字符为单位的****长度。


These differences are due to how international languages are handled on a per-platform basis - and more importantly, that the C and C++ strings used in this example can be considered ****an array of bytes****, such that (for this usage) ****the C++ language considers a character (char) to be a single byte****.

> 这些差异是由于国际语言在每个平台上的处理方式，更重要的是，本例中使用的C和C++字符串可以被视为***字节数组***，因此（对于这种用法）***C++语言将字符（char）视为单个字节***。



#### Remarks



The C++ language does not dictate any character-set, some compilers may ****support**** the use of UTF-8, or even UTF-16. However there is no certainty that anything beyond simple ANSI/ASCII characters will be provided.

> C++语言不规定任何字符集，一些编译器可能***支持***使用UTF-8，甚至UTF-16。但是，除了简单的ANSI/ASCII字符之外，不一定会提供任何其他字符。


Thus all international language support is implementation defined, reliant on what platform, operating system, and compiler you may be using.

> 因此，所有的国际语言支持都是由实现定义的，取决于您可能使用的平台、操作系统和编译器。


Several third party libraries (such as the International Unicode Committee Library) that can be used to extend the international support of the platform.

> 几个第三方库（如国际Unicode委员会库）可用于扩展平台的国际支持。

