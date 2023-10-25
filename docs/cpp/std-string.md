---
tip: translate by baidu@2023-10-25 08:32:47
---
---

metaTitle: "C++ | std::string"
description: "Tokenize, Conversion to (const) char*, Using the std::string_view class, Conversion to std::wstring, Trimming characters at start/end, Lexicographical comparison, String replacement, Converting to std::string, Splitting, Accessing a character, Looping through each character, Checking if a string is a prefix of another, Concatenation, Conversion to integers/floating point types, Converting between character encodings, Finding character(s) in a string"
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# std::string

Strings are objects that represent sequences of characters. The standard `string` class provides a simple, safe and versatile alternative to using explicit arrays of `char` s when dealing with text and other sequences of characters. The C++ `string` class is part of the `std` namespace and was standardized in 1998.

> 字符串是表示字符序列的对象。在处理文本和其他字符序列时，标准的“string”类提供了一种简单、安全且通用的替代方法，以代替使用“char”的显式数组。C++“string”类是“std”命名空间的一部分，于 1998 年标准化。

## Tokenize

Listed from least expensive to most expensive at run-time:

<li>
`str::strtok` is the cheapest standard provided tokenization method, it also allows the delimiter to be modified between tokens, but it incurs 3 difficulties with modern C++:
<ul>

1. `std::strtok` cannot be used on multiple `strings` at the same time (though some implementations do extend to support this, such as: [`strtok_s`](https://msdn.microsoft.com/en-us/library/ftsafwz3.aspx))

> 1.“std:：strtok”不能同时用于多个“字符串”（尽管有些实现确实扩展到支持这一点，例如：[`strtok_s`](https://msdn.microsoft.com/en-us/library/ftsafwz3.aspx))

1. For the same reason `std::strtok` cannot be used on multiple threads simultaneously (this may however be implementation defined, for example: [Visual Studio's implementation is thread safe](https://msdn.microsoft.com/en-us/library/2c8d19sb.aspx#Anchor_3))

> 1.出于同样的原因，“std:：strtok”不能同时用于多个线程（但这可能是实现定义的，例如：[Visual Studio 的实现是线程安全的](https://msdn.microsoft.com/en-us/library/2c8d19sb.aspx#Anchor_3))

1. Calling `std::strtok` modifies the `std::string` it is operating on, so it cannot be used on `const string` s, `const char*` s, or literal strings, to tokenize any of these with `std::strtok` or to operate on a `std::string` who's contents need to be preserved, the input would have to be copied, then the copy could be operated on

> 1.调用“std:：strtok”会修改它正在操作的“std：：string”，因此它不能用于“const string”、“const char*”或文字字符串，也不能用于用“std:”strtok 标记这些字符串中的任何一个，也不能操作需要保留其内容的“std:

</ul>

Generally any of these options cost will be hidden in the allocation cost of the tokens, but if the cheapest algorithm is required and `std::strtok`'s difficulties are not overcomable consider a [hand-spun solution](http://stackoverflow.com/a/38595708/2642059).

> 通常，这些选项成本中的任何一个都会隐藏在代币的分配成本中，但如果需要最便宜的算法，并且“std:：strtok”的困难无法克服，请考虑[手工解决方案](http://stackoverflow.com/a/38595708/2642059)。

</li>

```cpp
// String to tokenize
std::string str{ "The quick brown fox" };
// Vector to store tokens
vector<std::string> tokens;

for (auto i = strtok(&str[0], " "); i != NULL; i = strtok(NULL, " "))
    tokens.push_back(i);

```

[Live Example](http://ideone.com/8kAGoa)

1. The `std::istream_iterator` uses the stream's extraction operator iteratively. If the input `std::string` is white-space delimited this is able to expand on the `std::strtok` option by eliminating its difficulties, allowing inline tokenization thereby supporting the generation of a `const vector<string>`, and by adding support for multiple delimiting white-space character:

> 1.“std:：istream_iterator”迭代使用流的提取运算符。如果输入“std:：string”是用空格分隔的，则可以通过消除其困难，允许内联标记化，从而支持生成“const vector＜string＞”，并通过添加对多个分隔空格字符的支持，来扩展“std：：strtok”选项：

```cpp
// String to tokenize
const std::string str("The  quick \tbrown \nfox");
std::istringstream is(str);
// Vector to store tokens
const std::vector<std::string> tokens = std::vector<std::string>(
                                        std::istream_iterator<std::string>(is),
                                        std::istream_iterator<std::string>());

```

[Live Example](http://ideone.com/gWmfV9)

1. The `std::regex_token_iterator` uses a `std::regex` to iteratively tokenize. It provides for a more flexible delimiter definition. For example, non-delimited commas and white-space:

> 1.“std:：regex_token_iterator”使用“std::regex”来迭代标记化。它提供了更灵活的分隔符定义。例如，非分隔逗号和空格：

```cpp
// String to tokenize
const std::string str{ "The ,qu\\,ick ,\tbrown, fox" };
const std::regex re{ "\\s*((?:[^\\\\,]|\\\\.)*?)\\s*(?:,|$)" };
// Vector to store tokens
const std::vector<std::string> tokens{ 
    std::sregex_token_iterator(str.begin(), str.end(), re, 1), 
    std::sregex_token_iterator() 
};

```

[Live Example](http://ideone.com/q58zoX)

See the [`regex_token_iterator` Example](http://stackoverflow.com/documentation/c%2B%2B/1681/regular-expressions/5425/regex-token-iterator-example) for more details.

> 请参阅[`regex_token_iterator` 示例]([http://stackoverflow.com/documentation/c%2b%2b/1681/regular-expressions/5425/regex](http://stackoverflow.com/documentation/c%2B%2B/1681/regular-expressions/5425/regex) 令牌迭代器示例）。

## Conversion to (const) char*

In order to get `const char*` access to the data of a `std::string` you can use the string's `c_str()` member function. Keep in mind that the pointer is only valid as long as the `std::string` object is within scope and remains unchanged, that means that only `const` methods may be called on the object.

> 为了获得对“std:：string”数据的“const char*”访问权限，可以使用该字符串的“c_str（）”成员函数。请记住，只有当“std:：string”对象在作用域内并且保持不变时，指针才有效，这意味着只能对对象调用“const”方法。

The `data()` member function can be used to obtain a modifiable `char*`, which can be used to manipulate the `std::string` object's data.

> “data（）”成员函数可用于获得可修改的“char*”，该函数可用于操作“std:：string”对象的数据。

A modifiable `char*` can also be obtained by taking the address of the first character: `&s[0]`. Within C++11, this is guaranteed to yield a well-formed, null-terminated string. Note that `&s[0]` is well-formed even if `s` is empty, whereas `&s.front()` is undefined if `s` is empty.

> 还可以通过获取第一个字符的地址来获得可修改的“char*”：“&s[0]”。在 C++11 中，这可以保证生成一个格式良好、以 null 结尾的字符串。请注意，即使“s”为空，“&s[0]”也是格式良好的，而如果“s”是空，则“&s.front（）”是未定义的。

```cpp
std::string str("This is a string.");
const char* cstr = str.c_str(); // cstr points to: "This is a string.\0"
const char* data = str.data();  // data points to: "This is a string.\0"

```

```cpp
std::string str("This is a string.");

// Copy the contents of str to untie lifetime from the std::string object
std::unique_ptr<char []> cstr = std::make_unique<char[]>(str.size() + 1);

// Alternative to the line above (no exception safety):
// char* cstr_unsafe = new char[str.size() + 1];

std::copy(str.data(), str.data() + str.size(), cstr);
cstr[str.size()] = '\0'; // A null-terminator needs to be added

// delete[] cstr_unsafe;
std::cout << cstr.get();

```

## Using the std::string_view class

C++17 introduces `std::string_view`, which is simply a non-owning range of `const char` s, implementable as either a pair of pointers or a pointer and a length. It is a superior parameter type for functions that requires non-modifiable string data. Before C++17, there were three options for this:

> C++17 引入了“std:：string_view”，它只是“const char”的一个非拥有范围，可以实现为一对指针或一个指针和一个长度。对于需要不可修改的字符串数据的函数，它是一种高级参数类型。在 C++17 之前，有三种选择：

```cpp
void foo(std::string const& s);      // pre-C++17, single argument, could incur
                                     // allocation if caller's data was not in a string
                                     // (e.g. string literal or vector<char> )

void foo(const char* s, size_t len); // pre-C++17, two arguments, have to pass them
                                     // both everywhere

void foo(const char* s);             // pre-C++17, single argument, but need to call
                                     // strlen()

template <class StringT>
void foo(StringT const& s);          // pre-C++17, caller can pass arbitrary char data
                                     // provider, but now foo() has to live in a header

```

All of these can be replaced with:

```cpp
void foo(std::string_view s);        // post-C++17, single argument, tighter coupling
                                     // zero copies regardless of how caller is storing
                                     // the data

```

**Note that `std::string_view` **cannot** modify its underlying data**.

`string_view` is useful when you want to avoid unnecessary copies.

It offers a useful subset of the functionality that `std::string` does, although some of the functions behave differently:

> 它提供了“std:：string”所做功能的有用子集，尽管有些功能的行为有所不同：

```cpp
std::string str = "lllloooonnnngggg sssstttrrriiinnnggg"; //A really long string

//Bad way - 'string::substr' returns a new string (expensive if the string is long)
std::cout << str.substr(15, 10) << '\n';

//Good way - No copies are created!
std::string_view view = str;

// string_view::substr returns a new string_view
std::cout << view.substr(15, 10) << '\n';

```

## Conversion to std::wstring

In C++, sequences of characters are represented by specializing the `std::basic_string` class with a native character type. The two major collections defined by the standard library are `std::string` and `std::wstring`:

> 在 C++ 中，字符序列是通过用本机字符类型专门化“std:：basic_string”类来表示的。标准库定义的两个主要集合是“std：：string”和“std:：wstring”：

<li>
`std::string` is built with elements of type `char`
</li>
<li>
`std::wstring` is built with elements of type `wchar_t`
</li>

To convert between the two types, use `wstring_convert`:

```cpp
#include <string>
#include <codecvt>
#include <locale>

std::string input_str = "this is a -string-, which is a sequence based on the -char- type.";
std::wstring input_wstr = L"this is a -wide- string, which is based on the -wchar_t- type.";

// conversion
std::wstring str_turned_to_wstr = std::wstring_convert<std::codecvt_utf8<wchar_t>>().from_bytes(input_str);

std::string wstr_turned_to_str = std::wstring_convert<std::codecvt_utf8<wchar_t>>().to_bytes(input_wstr);

```

In order to improve usability and/or readability, you can define functions to perform the conversion:

> 为了提高可用性和/或可读性，您可以定义执行转换的函数：

```cpp
#include <string>
#include <codecvt>
#include <locale>

using convert_t = std::codecvt_utf8<wchar_t>;
std::wstring_convert<convert_t, wchar_t> strconverter;

std::string to_string(std::wstring wstr)
{
    return strconverter.to_bytes(wstr);
}

std::wstring to_wstring(std::string str)
{
    return strconverter.from_bytes(str);
}

```

Sample usage:

```cpp
std::wstring a_wide_string = to_wstring("Hello World!");

```

That's certainly more readable than `std::wstring_convert<std::codecvt_utf8<wchar_t>>().from_bytes("Hello World!")`.

> 这肯定比“std:：wstring_convert＜std:：codecvt_utf8＜wchar_t＞（）.from_bytes（“Hello World！”）”更可读。

Please note that `char` and `wchar_t` do not imply encoding, and gives no indication of size in bytes. For instance, `wchar_t` is commonly implemented as a 2-bytes data type and typically contains UTF-16 encoded data under Windows (or UCS-2 in versions prior to Windows 2000) and as a 4-bytes data type encoded using UTF-32 under Linux. This is in contrast with the newer types `char16_t` and `char32_t`, which were introduced in C++11 and are guaranteed to be large enough to hold any UTF16 or UTF32 "character" (or more precisely, **code point**) respectively.

> 请注意，“char”和“wchar_t”并不意味着编码，也不表示字节大小。例如，“wchar_t”通常被实现为 2 字节的数据类型，并且通常在 Windows（或在 Windows 2000 之前的版本中的 UCS-2）下包含 UTF-16 编码的数据，以及在 Linux 下使用 UTF-32 编码的 4 字节数据类型。这与 C++11 中引入的较新类型“char16_t”和“char32_t”形成了对比，它们保证足够大，可以分别容纳任何 UTF16 或 UTF32“字符”（或者更准确地说，**代码点**）。

## Trimming characters at start/end

This example requires the headers [`<algorithm>`](http://en.cppreference.com/w/cpp/header/algorithm), [`<locale>`](http://en.cppreference.com/w/cpp/header/locale), and [`<utility>`](http://en.cppreference.com/w/cpp/header/utility).

> 此示例需要标头[`<algorithm>`](http://en.cppreference.com/w/cpp/header/algorithm)，[`<locale>`](http://en.cppreference.com/w/cpp/header/locale)，和[`<utility>`](http://en.cppreference.com/w/cpp/header/utility)。

To **trim** a sequence or string means to remove all leading and trailing elements (or characters) matching a certain predicate. We first trim the trailing elements, because it doesn't involve moving any elements, and then trim the leading elements. Note that the generalizations below work for all types of `std::basic_string` (e.g. `std::string` and `std::wstring`), and accidentally also for sequence containers (e.g. `std::vector` and `std::list`).

> 对序列或字符串进行 **trim** 意味着删除与某个谓词匹配的所有前导和尾随元素（或字符）。我们首先修剪后面的元素，因为它不涉及移动任何元素，然后修剪前面的元素。请注意，下面的概括适用于所有类型的“std:：basic_string”（例如“std：：string”和“std::：wstring”），意外地也适用于序列容器（例如“std：：vector”和“td：：：list”）。

```cpp
template <typename Sequence, // any basic_string, vector, list etc.
          typename Pred>     // a predicate on the element (character) type
Sequence& trim(Sequence& seq, Pred pred) {
    return trim_start(trim_end(seq, pred), pred);
}

```

Trimming the trailing elements involves finding the **last** element not matching the predicate, and erasing from there on:

> 修剪尾部元素包括找到与谓词不匹配的最后一个**元素，然后从中擦除：

```cpp
template <typename Sequence, typename Pred>
Sequence& trim_end(Sequence& seq, Pred pred) {
    auto last = std::find_if_not(seq.rbegin(),
                                 seq.rend(),
                                 pred);
    seq.erase(last.base(), seq.end());
    return seq;
}

```

Trimming the leading elements involves finding the **first** element not matching the predicate and erasing up to there:

> 修剪前导元素包括找到与谓词不匹配的**第一个**元素，并将其擦除：

```cpp
template <typename Sequence, typename Pred>
Sequence& trim_start(Sequence& seq, Pred pred) {
    auto first = std::find_if_not(seq.begin(),
                                  seq.end(),
                                  pred);
    seq.erase(seq.begin(), first);
    return seq;
}

```

To specialize the above for trimming whitespace in a `std::string` we can use the [`std::isspace()`](http://en.cppreference.com/w/cpp/locale/isspace) function as a predicate:

> 要专门处理以上内容以修剪“std:：string”中的空白，我们可以使用[`std:：isspace（）`](http://en.cppreference.com/w/cpp/locale/isspace)充当谓词：

```cpp
std::string& trim(std::string& str, const std::locale& loc = std::locale()) {
    return trim(str, [&loc](const char c){ return std::isspace(c, loc); });
}

std::string& trim_start(std::string& str, const std::locale& loc = std::locale()) {
    return trim_start(str, [&loc](const char c){ return std::isspace(c, loc); });
}

std::string& trim_end(std::string& str, const std::locale& loc = std::locale()) {
    return trim_end(str, [&loc](const char c){ return std::isspace(c, loc); });
}

```

Similarly, we can use the [`std::iswspace()`](http://en.cppreference.com/w/cpp/string/wide/iswspace) function for `std::wstring` etc.

> 类似地，我们可以使用[`std:：iswspace（）`](http://en.cppreference.com/w/cpp/string/wide/iswspace)“std:：wstring”等的函数。

If you wish to create a **new** sequence that is a trimmed copy, then you can use a separate function:

> 如果您希望创建一个**新**序列，该序列是修剪后的副本，那么您可以使用一个单独的函数：

```cpp
template <typename Sequence, typename Pred>
Sequence trim_copy(Sequence seq, Pred pred) { // NOTE: passing seq by value
    trim(seq, pred);
    return seq;
}

```

## Lexicographical comparison

Two `std::string` s can be  compared lexicographically using the operators `==`, `!=`, `<`, `<=`, `>`, and `>=`:

> 两个“std:：string”可以使用运算符“==”、“！=”、“<”、“<=”、，和 `>=`：

```cpp
std::string str1 = "Foo";
std::string str2 = "Bar";

assert(!(str1 < str2));
assert(str > str2);
assert(!(str1 <= str2));
assert(str1 >= str2);
assert(!(str1 == str2));
assert(str1 != str2);

```

All these functions use the underlying `std::string::compare()` method to perform the comparison, and return for convenience boolean values. The operation of these functions may be interpreted as follows, regardless of the actual implementation:

> 所有这些函数都使用底层的“std:：string:：compare（）”方法来执行比较，并为方便起见返回布尔值。无论实际实施情况如何，这些功能的操作可解释如下：

<li>
operator`==`:

If `str1.length() == str2.length()` and each character pair matches, then returns `true`, otherwise returns `false`.

> 如果“str1.length（）==str2.length（（）”且每个字符对匹配，则返回“true”，否则返回“false”。

</li>
<li>
operator`!=`:

If `str1.length() != str2.length()` or one character pair doesn't match, returns `true`, otherwise it returns `false`.

> 如果 `str1.length（）！=str2.length（）` 或一个字符对不匹配，则返回'true'，否则返回'false'。

</li>
<li>
operator`<` or operator`>`:

Finds the first different character pair, compares them then returns the boolean result.

> 查找第一个不同的字符对，对它们进行比较，然后返回布尔结果。

</li>
<li>
operator`<=` or operator`>=`:

Finds the first different character pair, compares them then returns the boolean result.

> 查找第一个不同的字符对，对它们进行比较，然后返回布尔结果。

</li>

****Note:**** The term **character pair** means the corresponding characters in both strings of the same positions. For better understanding, if two example strings are `str1` and `str2`, and their lengths are `n` and `m` respectively, then character pairs of both strings means each `str1[i]` and `str2[i]` pairs where **i = 0, 1, 2, ..., max(n,m)**. If for any **i** where the corresponding character does not exist, that is, when **i** is greater than or equal to `n` or `m`, it would be considered as the lowest value.

> ****注：***术语**字符对**表示相同位置的两个字符串中的对应字符。为了更好地理解，如果两个示例字符串是“str1”和“str2”，并且它们的长度分别是“n”和“m”，那么这两个字符串的字符对意味着每个“str1[i]”和“tr2[i]”对，其中**i＝0，1，2。。。，max（n，m）**。如果对于不存在相应字符的任何**i**，即当 **i** 大于或等于“n”或“m”时，它将被视为最低值。

Here is an example of using `<`:

```cpp
std::string str1 = "Barr";
std::string str2 = "Bar";

assert(str2 < str1);

```

The steps are as follows:

1. Compare the first characters, `'B' == 'B'` - move on.
2. Compare the second characters, `'a' == 'a'` - move on.
3. Compare the third characters, `'r' == 'r'` - move on.
4. The `str2` range is now exhausted, while the `str1` range still has characters. Thus, `str2 < str1`.

> 1.“str2”范围现在已用完，而“str1”范围仍有字符。因此，“str2<str1”。

## String replacement

### Replace by position

To replace a portion of a `std::string` you can use the method `replace` from `std::string`.

> 要替换“std：：string”的一部分，可以使用“std:：string”中的“replace”方法。

`replace` has a lot of useful overloads:

```cpp
//Define string
std::string str = "Hello foo, bar and world!";
std::string alternate = "Hello foobar";

//1)
str.replace(6, 3, "bar"); //"Hello bar, bar and world!"

//2)
str.replace(str.begin() + 6, str.end(), "nobody!"); //"Hello nobody!"

//3)
str.replace(19, 5, alternate, 6, 6); //"Hello foo, bar and foobar!"

```

```cpp
//4)
str.replace(19, 5, alternate, 6); //"Hello foo, bar and foobar!"

```

```cpp
//5)
str.replace(str.begin(), str.begin() + 5, str.begin() + 6, str.begin() + 9);
//"foo foo, bar and world!"

//6)
str.replace(0, 5, 3, 'z'); //"zzz foo, bar and world!"

//7)
str.replace(str.begin() + 6, str.begin() + 9, 3, 'x'); //"Hello xxx, bar and world!"

```

```cpp
//8)
str.replace(str.begin(), str.begin() + 5, { 'x', 'y', 'z' }); //"xyz foo, bar and world!"

```

### Replace occurrences of a string with another string

Replace only the first occurrence of `replace` with `with` in `str`:

```cpp
std::string replaceString(std::string str,
                          const std::string& replace,
                          const std::string& with){
    std::size_t pos = str.find(replace);
    if (pos != std::string::npos)
        str.replace(pos, replace.length(), with);
    return str;
}

```

Replace all occurrence of `replace` with `with` in `str`:

```cpp
std::string replaceStringAll(std::string str,
                             const std::string& replace,
                             const std::string& with) {
    if(!replace.empty()) {
        std::size_t pos = 0;
        while ((pos = str.find(replace, pos)) != std::string::npos) {
            str.replace(pos, replace.length(), with);
            pos += with.length();
        }
    }
    return str;
}

```

## Converting to std::string

[`std::ostringstream`](http://en.cppreference.com/w/cpp/io/basic_ostringstream) can be used to convert any streamable type to a string representation, by inserting the object into a `std::ostringstream` object (with the stream insertion operator `<<`) and then converting the whole `std::ostringstream` to a `std::string`.

> [`std:：ostringstream`](http://en.cppreference.com/w/cpp/io/basic_ostringstream)可用于将任何可流化类型转换为字符串表示，方法是将对象插入“std:：ostringstream”对象（使用流插入运算符“<<”），然后将整个“std:”：ostringstream“转换为“std：：string”。

For `int` for instance:

```cpp
#include <sstream>

int main()
{
    int val = 4;
    std::ostringstream str;
    str << val;
    std::string converted = str.str();
    return 0;
}

```

Writing your own conversion function, the simple:

```cpp
template<class T>
std::string toString(const T& x)
{
  std::ostringstream ss;
  ss << x;
  return ss.str();
}

```

works but isn't suitable for performance critical code.

User-defined classes may implement the stream insertion operator if desired:

> 如果需要，用户定义的类可以实现流插入运算符：

```cpp
std::ostream operator<<( std::ostream& out, const A& a )
{
    // write a string representation of a to out
    return out; 
}

```

Aside from streams, since C++11 you can also use the [`std::to_string`](http://en.cppreference.com/w/cpp/string/basic_string/to_string) (and [`std::to_wstring`](http://en.cppreference.com/w/cpp/string/basic_string/to_wstring)) function which is overloaded for all fundamental types and returns the string representation of its parameter.

> 除了流之外，由于 C++11，您还可以使用[`std:：to_string`](http://en.cppreference.com/w/cpp/string/basic_string/to_string)（和[`std:：to_wstring`](http://en.cppreference.com/w/cpp/string/basic_string/to_wstring))函数，该函数为所有基本类型重载，并返回其参数的字符串表示。

```cpp
std::string s = to_string(0x12f3);  // after this the string s contains "4851"

```

## Splitting

Use [`std::string::substr`](http://en.cppreference.com/w/cpp/string/basic_string/substr) to split a string. There are two variants of this member function.

> 使用[`std:：string:：substr`](http://en.cppreference.com/w/cpp/string/basic_string/substr)把绳子分开。此成员函数有两种变体。

The first takes a **starting position** from which the returned substring should begin. The starting position must be valid in the range `(0, str.length()]`:

> 第一个采用**起始位置**，返回的子字符串应从该位置开始。起始位置必须在“（0，str.length（）]` 范围内有效：

```cpp
std::string str = "Hello foo, bar and world!";
std::string newstr = str.substr(11); // "bar and world!"

```

The second takes a starting position and a total **length** of the new substring. Regardless of the **length**, the substring will never go past the end of the source string:

> 第二个取新子字符串的起始位置和总长度**。无论**长度**如何，子字符串都不会超过源字符串的末尾：

```cpp
std::string str = "Hello foo, bar and world!";
std::string newstr = str.substr(15, 3); // "and"

```

****Note that**** you can also call `substr` with no arguments, in this case an exact copy of the string is returned

> ****请注意，***您也可以不带参数地调用“substr”，在这种情况下，将返回字符串的精确副本

```cpp
std::string str = "Hello foo, bar and world!";
std::string newstr = str.substr(); // "Hello foo, bar and world!"

```

## Accessing a character

There are several ways to extract characters from a `std::string` and each is subtly different.

> 有几种方法可以从“std:：string”中提取字符，每种方法都有细微的不同。

```cpp
std::string str("Hello world!");

```

### operator[](n)

Returns a reference to the character at index n.

`std::string::operator[]` is not bounds-checked and does not throw an exception. The caller is responsible for asserting that the index is within the range of the string:

```cpp
char c = str[6]; // 'w'

```

### at(n)

Returns a reference to the character at index n.

`std::string::at` **is** bounds checked, and will throw `std::out_of_range` if the index is not within the range of the string:

```cpp
char c = str.at(7); // 'o'

```

**Note: Both of these examples will result in [undefined behavior](http://stackoverflow.com/documentation/c%2B%2B/1812/undefined-behavior#t=201607232040171204946) if the string is empty.**

> **注意：这两个例子都会导致[未定义的行为]([http://stackoverflow.com/documentation/c%2B%2B/1812/undefined-behavior#t=201607232040171204946](http://stackoverflow.com/documentation/c%2B%2B/1812/undefined-behavior#t=201607232040171204946)）**

### front()

Returns a reference to the first character:

```cpp
char c = str.front(); // 'H'

```

### back()

Returns a reference to the last character:

```cpp
char c = str.back(); // '!'

```

## Looping through each character

`std::string` supports iterators, and so you can use a **ranged based** loop to iterate through each character:

```cpp
std::string str = "Hello World!";
for (auto c : str)
    std::cout << c;

```

You can use a "traditional" `for` loop to loop through every character:

```cpp
std::string str = "Hello World!";
for (std::size_t i = 0; i < str.length(); ++i)
    std::cout << str[i];

```

## Checking if a string is a prefix of another

In C++14, this is easily done by [`std::mismatch`](http://en.cppreference.com/w/cpp/algorithm/mismatch) which returns the first mismatching pair from two ranges:

> 在 C++14 中，这很容易通过[`std:：mismatch`]完成([http://en.cppreference.com/w/cpp/algorithm/mismatch](http://en.cppreference.com/w/cpp/algorithm/mismatch))其从两个范围返回第一失配对：

```cpp
std::string prefix = "foo";
std::string string = "foobar";

bool isPrefix = std::mismatch(prefix.begin(), prefix.end(),
    string.begin(), string.end()).first == prefix.end();

```

Note that a range-and-a-half version of `mismatch()` existed prior to C++14, but this is unsafe in the case that the second string is the shorter of the two.

> 请注意，在 C++14 之前就存在一个半范围的“mismatch（）”版本，但如果第二个字符串是这两个字符串中较短的一个，则这是不安全的。

We can still use the range-and-a-half version of `std::mismatch()`, but we need to first check that the first string is at most as big as the second:

> 我们仍然可以使用“std:：mismatch（）”的半范围版本，但我们需要首先检查第一个字符串是否最多与第二个字符串一样大：

```cpp
bool isPrefix = prefix.size() <= string.size() &&
    std::mismatch(prefix.begin(), prefix.end(),
        string.begin(), string.end()).first == prefix.end();

```

With `std::string_view`, we can write the direct comparison we want without having to worry about allocation overhead or making copies:

> 使用“std:：string_view”，我们可以编写我们想要的直接比较，而不必担心分配开销或复制：

```cpp
bool isPrefix(std::string_view prefix, std::string_view full)
{
    return prefix == full.substr(0, prefix.size());
}

```

## Concatenation

You can concatenate `std::string` s using the overloaded `+` and `+=` operators.

> 可以使用重载的“+”和“+=”运算符连接“std:：string”。
> Using the `+` operator:

```cpp
std::string hello = "Hello";
std::string world = "world";
std::string helloworld = hello + world; // "Helloworld"

```

Using the `+=` operator:

```cpp
std::string hello = "Hello";
std::string world = "world";
hello += world; // "Helloworld"

```

You can also append C strings, including string literals:

```cpp
std::string hello = "Hello";
std::string world = "world";
const char *comma = ", ";
std::string newhelloworld = hello + comma + world + "!"; // "Hello, world!"

```

You can also use `push_back()` to push back individual `char` s:

```cpp
std::string s = "a, b, ";
s.push_back('c'); // "a, b, c"

```

There is also `append()`, which is pretty much like `+=`:

```cpp
std::string app = "test and ";
app.append("test"); // "test and test"

```

## Conversion to integers/floating point types

A `std::string` containing a number can be converted into an integer type, or a floating point type, using conversion functions.

> 包含数字的“std:：string”可以使用转换函数转换为整数类型或浮点类型。

****Note that**** all of these functions stop parsing the input string as soon as they encounter a non-numeric character, so `"123abc"` will be converted into `123`.

> ****请注意，***所有这些函数一旦遇到非数字字符就停止解析输入字符串，因此“123abc”将转换为“123”。

The `std::ato*` family of functions converts C-style strings (character arrays) to integer or floating-point types:

> “std:：ato*”函数族将 C 样式字符串（字符数组）转换为整数或浮点类型：

```cpp
std::string ten = "10";

double num1 = std::atof(ten.c_str());
int num2 = std::atoi(ten.c_str());
long num3 = std::atol(ten.c_str());

```

```cpp
long long num4 = std::atoll(ten.c_str());

```

However, use of these functions is discouraged because they return `0` if they fail to parse the string. This is bad because `0` could also be a valid result, if for example the input string was "0", so it is impossible to determine if the conversion actually failed.

> 但是，不鼓励使用这些函数，因为如果未能解析字符串，它们将返回“0”。这很糟糕，因为“0”也可能是一个有效的结果，例如，如果输入字符串是“0”，则无法确定转换是否真的失败。

The newer `std::sto*` family of functions convert `std::string` s to integer or floating-point types, and throw exceptions if they could not parse their input. **You should use these functions if possible**:

> 较新的“std:：sto*”函数族将“std:”字符串转换为整数或浮点类型，如果无法解析其输入，则抛出异常**如果可能，您应该使用以下功能**：

```cpp
std::string ten = "10";

int num1 = std::stoi(ten);
long num2 = std::stol(ten);
long long num3 = std::stoll(ten);

float num4 = std::stof(ten);
double num5 = std::stod(ten);
long double num6 = std::stold(ten);

```

Furthermore, these functions also handle octal and hex strings unlike the `std::ato*` family. The second parameter is a pointer to the first unconverted character in the input string (not illustrated here), and the third parameter is the base to use. `0` is automatic detection of octal (starting with `0`) and hex (starting with `0x` or `0X`), and any other value is the base to use

> 此外，与“std:：ato*”系列不同，这些函数还处理八进制和十六进制字符串。第二个参数是指向输入字符串中第一个未转换字符的指针（此处未示出），第三个参数是要使用的基 `“0”是八进制（以“0”开头）和十六进制（以‘0x’或‘0x’开头）的自动检测，任何其他值都是要使用的基数

```cpp
std::string ten = "10";
std::string ten_octal = "12";
std::string ten_hex = "0xA";

int num1 = std::stoi(ten, 0, 2); // Returns 2
int num2 = std::stoi(ten_octal, 0, 8); // Returns 10
long num3 = std::stol(ten_hex, 0, 16);  // Returns 10
long num4 = std::stol(ten_hex);  // Returns 0
long num5 = std::stol(ten_hex, 0, 0); // Returns 10 as it detects the leading 0x

```

## Converting between character encodings

Converting between encodings is easy with C++11 and most compilers are able to deal with it in a cross-platform manner through `<codecvt>` and `<locale>` headers.

> 使用 C++11 可以很容易地在编码之间进行转换，大多数编译器都能够通过“＜codecvt＞”和“＜locale＞”标头以跨平台的方式处理它。

```cpp
#include <iostream>
#include <codecvt>
#include <locale>
#include <string>
using namespace std;

int main() {
    // converts between wstring and utf8 string
    wstring_convert<codecvt_utf8_utf16<wchar_t>> wchar_to_utf8;
    // converts between u16string and utf8 string
    wstring_convert<codecvt_utf8_utf16<char16_t>, char16_t> utf16_to_utf8;
    
    wstring wstr = L"foobar";
    string utf8str = wchar_to_utf8.to_bytes(wstr);
    wstring wstr2 = wchar_to_utf8.from_bytes(utf8str);
    
    wcout << wstr << endl;
    cout << utf8str << endl;
    wcout << wstr2 << endl;
    
    u16string u16str = u"foobar";
    string utf8str2 = utf16_to_utf8.to_bytes(u16str);
    u16string u16str2 = utf16_to_utf8.from_bytes(utf8str2);
    
    return 0;
}

```

Mind that Visual Studio 2015 provides supports for these conversion but a [bug](https://social.msdn.microsoft.com/Forums/vstudio/en-US/8f40dcd8-c67f-4eba-9134-a19b9178e481/vs-2015-rc-linker-stdcodecvt-error?forum=vcgeneral) in their library implementation requires to use a different template for `wstring_convert` when dealing with `char16_t`:

> 请注意，Visual Studio 2015 提供了对这些转换的支持，但存在一个〔bug〕([https://social.msdn.microsoft.com/Forums/vstudio/en-US/8f40dcd8-c67f-4eba-9134-a19b9178e481/vs-2015-rc-linker-stdcodecvt-error?forum=vcgeneral](https://social.msdn.microsoft.com/Forums/vstudio/en-US/8f40dcd8-c67f-4eba-9134-a19b9178e481/vs-2015-rc-linker-stdcodecvt-error?forum=vcgeneral))在他们的库中，当处理“char16_t”时，实现需要为“wstring_convert”使用不同的模板：

```cpp
using utf16_char = unsigned short;
wstring_convert<codecvt_utf8_utf16<utf16_char>, utf16_char> conv_utf8_utf16;

void strings::utf16_to_utf8(const std::u16string& utf16, std::string& utf8)
{
  std::basic_string<utf16_char> tmp;
  tmp.resize(utf16.length());
  std::copy(utf16.begin(), utf16.end(), tmp.begin());
  utf8 = conv_utf8_utf16.to_bytes(tmp);
}
void strings::utf8_to_utf16(const std::string& utf8, std::u16string& utf16)
{ 
  std::basic_string<utf16_char> tmp = conv_utf8_utf16.from_bytes(utf8);
  utf16.clear();
  utf16.resize(tmp.length());
  std::copy(tmp.begin(), tmp.end(), utf16.begin());
}

```

## Finding character(s) in a string

To find a character or another string, you can use [`std::string::find`](http://en.cppreference.com/w/cpp/string/basic_string/find). It returns the position of the first character of the first match. If no matches were found, the function returns [`std::string::npos`](http://en.cppreference.com/w/cpp/string/basic_string/npos)

> 要查找一个字符或另一个字符串，可以使用[`std:：string:：find`](http://en.cppreference.com/w/cpp/string/basic_string/find)。它返回第一个匹配的第一个字符的位置。如果没有找到匹配项，函数将返回[`std:：string:：npos`](http://en.cppreference.com/w/cpp/string/basic_string/npos)

```cpp
std::string str = "Curiosity killed the cat";
auto it = str.find("cat");

if (it != std::string::npos)
    std::cout << "Found at position: " << it << '\n';
else
    std::cout << "Not found!\n";

```

>

Found at position: 21

The search opportunities are further expanded by the following functions:

```cpp
find_first_of     // Find first occurrence of characters 
find_first_not_of // Find first absence of characters 
find_last_of      // Find last occurrence of characters 
find_last_not_of  // Find last absence of characters 

```

These functions can allow you to search for characters from the end of the string, as well as find the negative case (ie. characters that are not in the string). Here is an example:

> 这些函数可以让您搜索字符串末尾的字符，也可以查找负格（即不在字符串中的字符）。以下是一个示例：

```cpp
std::string str = "dog dog cat cat";
std::cout << "Found at position: " << str.find_last_of("gzx") << '\n';

```

>

Found at position: 6

****Note:**** Be aware that the above functions do not search for substrings, but rather for characters contained in the search string. In this case, the last occurrence of `'g'` was found at position `6` (the other characters weren't found).

> ****注意：***请注意，上述函数不搜索子字符串，而是搜索搜索字符串中包含的字符。在这种情况下，最后一个出现的“g”出现在位置“6”（没有找到其他字符）。

#### Syntax

<li>
**// Empty string declaration**
std::string s;
</li>
<li>
**// Constructing from const  char* (c-string)**
std::string s("Hello");
std::string s = "Hello";
</li>
<li>
**// Constructing using copy constructor**
std::string s1("Hello");
std::string s2(s1);
</li>
<li>
**// Constructing from substring**
std::string s1("Hello");

std::string s2(s1, 0, 4); **// Copy 4 characters from position 0 of s1 into s2**

> std:：字符串 s2（s1，0，4）；**//将 4 个字符从 s1 的位置 0 复制到 s2**

</li>
<li>
**// Constructing from a buffer of characters**
<p>std::string s1("Hello World");<br />
std::string s2(s1, 5); **// Copy first 5  characters of s1 into s2**</p>
</li>
<li>
**// Construct using fill constructor (char only)**
std::string s(5, 'a'); **// s contains aaaaa**
</li>
<li>
**// Construct using range constructor and iterator**
std::string s1("Hello World");

std::string s2(s1.begin(), s1.begin()+5); **// Copy first 5 characters of s1 into s2**

> std:：字符串 s2（s1.begin（），s1.begin（）+5）；**//将 s1 的前 5 个字符复制到 s2 中**

</li>

#### Remarks

Before using `std::string`, you should include the header `string`, as it includes functions/operators/overloads that other headers (for example `iostream`) do not include.

> 在使用“std:：string”之前，应包含标头“string”，因为它包含其他标头（例如“iostream”）不包含的函数/运算符/重载。

Using const char* constructor with a nullptr leads to undefined behavior.

```cpp
std::string oops(nullptr);
std::cout << oops << "\n";

```

The method `at` throws an `std::out_of_range` exception if `index >= size()`.

> 如果“index>=size（）”，方法“at”将抛出“std:：out_of_range”异常。

The behavior of `operator[]` is a bit more complicated, in all cases it has undefined behavior if `index > size()`, but when `index == size()`:

> “operator[]”的行为有点复杂，在所有情况下，如果“index>size（）”，它都有未定义的行为，但当“index==size（））”时：

1. On a non-const string, the behavior is **undefined**;
2. On a const string, a reference to a character with value `CharT()` (the **null** character) is returned.

> 1.在 const 字符串上，返回对值为“CharT（）”的字符（**null** 字符）的引用。

1. A reference to a character with value `CharT()` (the **null** character) is returned.

> 1.返回对值为“CharT（）”的字符（***空**字符）的引用。

1. Modifying this reference is **undefined behavior**.

Since C++14, instead of using `"foo"`, it is recommended to use `"foo"s`, as `s` is a [user-defined literal suffix](http://stackoverflow.com/documentation/c%2B%2B/2745/user-defined-literals), which converts the `const char*` `"foo"` to `std::string` `"foo"`.

> 由于 C++14 不使用“foo”，因此建议使用“foo”s，因为“s”是[用户定义的文字后缀]([http://stackoverflow.com/documentation/c%2b%2b/2745/user](http://stackoverflow.com/documentation/c%2B%2B/2745/user)-定义的文字），它将 `const char*``“foo”` 转换为 `std:：string``“foo”`。

*Note: you have to use the namespace `std::string_literals` or `std::literals` to get the literal `s`.*

> *注意：您必须使用名称空间“std:：string_tliterals”或“std：：literalls”来获取文本“s”*
