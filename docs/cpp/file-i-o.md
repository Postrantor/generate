---
tip: translate by baidu@2023-10-25 08:24:40
---
---

metaTitle: "C++ | File I/O"
description: "Writing to a file, Opening a file, Reading from a file, Opening modes, Reading an ASCII file into a std::string, Writing files with non-standard locale settings, Flushing a stream, Reading a file into a container, Checking end of file inside a loop condition, bad practice?, Closing a file, Reading a `struct` from a formatted text file., Copying a file"
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# File I/O

C++ file I/O is done via **streams**. The key abstractions are:

`std::istream` for reading text.

`std::ostream` for writing text.

`std::streambuf` for reading or writing characters.

**Formatted input** uses `operator>>`.

**Formatted output** uses `operator<<`.

Streams use `std::locale`, e.g., for details of the formatting and for translation between external encodings and the internal encoding.

> 流使用“std:：locale”，例如，用于格式化的详细信息以及外部编码和内部编码之间的转换。

More on streams: [<iostream> Library](http://stackoverflow.com/documentation/c%2B%2B/7660/iostream-library)

> 更多关于流的信息：[<iostream>Library]([http://stackoverflow.com/documentation/c%2b%2b/7660/iostream](http://stackoverflow.com/documentation/c%2B%2B/7660/iostream)-图书馆）

## Writing to a file

There are several ways to write to a file. The easiest way is to use an output file stream (`ofstream`) together with the stream insertion operator (`<<`):

> 有几种方法可以写入文件。最简单的方法是将输出文件流（“ofstream”）与流插入运算符（“<<”）一起使用：

```cpp
std::ofstream os("foo.txt");
if(os.is_open()){
    os << "Hello World!";
}

```

Instead of `<<`, you can also use the output file stream's member function `write()`:

> 您也可以使用输出文件流的成员函数“write（）”而不是“<<”：

```cpp
std::ofstream os("foo.txt");
if(os.is_open()){
    char data[] = "Foo";

    // Writes 3 characters from data -> "Foo".
    os.write(data, 3);
}

```

After writing to a stream, you should always check if error state flag `badbit` has been set, as it indicates whether the operation failed or not. This can be done by calling the output file stream's member function `bad()`:

> 写入流后，应始终检查是否设置了错误状态标志“badbit”，因为它指示操作是否失败。这可以通过调用输出文件流的成员函数“bad（）”来实现：

```cpp
os << "Hello Badbit!"; // This operation might fail for any reason.
if (os.bad())
    // Failed to write!

```

## Opening a file

Opening a file is done in the same way for all 3 file streams (`ifstream`, `ofstream`, and `fstream`).

> 对于所有 3 个文件流（“ifstream”、“ofstream”和“fstream”），以相同的方式打开文件。

You can open the file directly in the constructor:

```cpp
std::ifstream ifs("foo.txt");  // ifstream: Opens file "foo.txt" for reading only.

std::ofstream ofs("foo.txt");  // ofstream: Opens file "foo.txt" for writing only.

std::fstream iofs("foo.txt");  // fstream:  Opens file "foo.txt" for reading and writing.

```

Alternatively, you can use the file stream's member function `open()`:

```cpp
std::ifstream ifs;
ifs.open("bar.txt");           // ifstream: Opens file "bar.txt" for reading only.

std::ofstream ofs;
ofs.open("bar.txt");           // ofstream: Opens file "bar.txt" for writing only.

std::fstream iofs;
iofs.open("bar.txt");          // fstream:  Opens file "bar.txt" for reading and writing.

```

You should **always** check if a file has been opened successfully (even when writing). Failures can include: the file doesn't exist, file hasn't the right access rights, file is already in use, disk errors occurred, drive disconnected ...

> 您应该**始终**检查文件是否已成功打开（即使在写入时）。故障可能包括：文件不存在、文件没有正确的访问权限、文件已在使用、发生磁盘错误、驱动器断开连接。。。
> Checking can be done as follows:

```cpp
// Try to read the file 'foo.txt'.
std::ifstream ifs("fooo.txt");  // Note the typo; the file can't be opened.

// Check if the file has been opened successfully.
if (!ifs.is_open()) {
    // The file hasn't been opened; take appropriate actions here.
    throw CustomException(ifs, "File could not be opened");
}

```

When file path contains backslashes (for example, on Windows system) you should properly escape them:

> 当文件路径包含反斜杠时（例如，在 Windows 系统上），您应该正确地对其进行转义：

```cpp
// Open the file 'c:\folder\foo.txt' on Windows.
std::ifstream ifs("c:\\folder\\foo.txt"); // using escaped backslashes

```

or use raw literal:

```cpp
// Open the file 'c:\folder\foo.txt' on Windows.
std::ifstream ifs(R"(c:\folder\foo.txt)"); // using raw literal

```

or use forward slashes instead:

```cpp
// Open the file 'c:\folder\foo.txt' on Windows.
std::ifstream ifs("c:/folder/foo.txt");

```

If you want to open file with non-ASCII characters in path on Windows currently you can use **non-standard** wide character path argument:

> 如果当前要在 Windows 上打开路径中包含非 ASCII 字符的文件，可以使用**非标准**宽字符路径参数：

```cpp
// Open the file 'пример\foo.txt' on Windows.
std::ifstream ifs(LR"(пример\foo.txt)"); // using wide characters with raw literal

```

## Reading from a file

There are several ways to read data from a file.

If you know how the data is formatted, you can use the stream extraction operator (`>>`). Let's assume you have a file named **foo.txt** which contains the following data:

> 如果您知道数据的格式，可以使用流提取运算符（`>>`）。假设您有一个名为 **foo.txt** 的文件，其中包含以下数据：

```cpp
John Doe 25 4 6 1987
Jane Doe 15 5 24 1976

```

Then you can use the following code to read that data from the file:

```cpp
// Define variables.
std::ifstream is("foo.txt");
std::string firstname, lastname;
int age, bmonth, bday, byear;

// Extract firstname, lastname, age, bday month, bday day, and bday year in that order.
// Note: '>>' returns false if it reached EOF (end of file) or if the input data doesn't
// correspond to the type of the input variable (for example, the string "foo" can't be
// extracted into an 'int' variable).
while (is >> firstname >> lastname >> age >> bmonth >> bday >> byear)
    // Process the data that has been read.

```

The stream extraction operator `>>` extracts every character and stops if it finds a character that can't be stored or if it is a special character:

> 流提取运算符“>>”提取每个字符，如果发现无法存储的字符或是特殊字符，则停止：

- For string types, the operator stops at a whitespace (``) or at a newline (`\n`).
- For numbers, the operator stops at a non-number character.

This means that the following version of the file **foo.txt** will also be successfully read by the previous code:

> 这意味着以下版本的文件 **foo.txt** 也将被之前的代码成功读取：

```cpp
John 
Doe 25
4 6 1987


Jane
Doe 
15 5
24
1976

```

The stream extraction operator `>>` always returns the stream given to it. Therefore, multiple operators can be chained together in order to read data consecutively. However, a stream can also be used as a Boolean expression (as shown in the `while` loop in the previous code). This is because the stream classes have a conversion operator for the type `bool`. This `bool()` operator will return `true` as long as the stream has no errors. If a stream goes into an error state (for example, because no more data can be extracted), then the `bool()` operator will return `false`. Therefore, the `while` loop in the previous code will be exited after the input file has been read to its end.

> 流提取运算符“>>”总是返回给定给它的流。因此，为了连续读取数据，可以将多个运算符链接在一起。然而，流也可以用作布尔表达式（如前一代码中的“while”循环所示）。这是因为流类具有类型“bool”的转换运算符。只要流没有错误，这个“bool（）”运算符就会返回“true”。如果流进入错误状态（例如，因为无法提取更多数据），则“bool（）”运算符将返回“false”。因此，前一代码中的“while”循环将在输入文件被读取到末尾后退出。

If you wish to read an entire file as a string, you may use the following code:

> 如果您希望将整个文件作为字符串读取，可以使用以下代码：

```cpp
// Opens 'foo.txt'.
std::ifstream is("foo.txt");
std::string whole_file;

// Sets position to the end of the file.
is.seekg(0, std::ios::end);

// Reserves memory for the file.
whole_file.reserve(is.tellg());

// Sets position to the start of the file.
is.seekg(0, std::ios::beg);

// Sets contents of 'whole_file' to all characters in the file.
whole_file.assign(std::istreambuf_iterator<char>(is),
  std::istreambuf_iterator<char>());

```

This code reserves space for the `string` in order to cut down on unneeded memory allocations.

> 此代码为“字符串”保留空间，以减少不必要的内存分配。

If you want to read a file line by line, you can use the function [`getline()`](http://en.cppreference.com/w/cpp/string/basic_string/getline):

> 如果要逐行读取文件，可以使用函数[`getline（）`](http://en.cppreference.com/w/cpp/string/basic_string/getline)：

```cpp
std::ifstream is("foo.txt");   

// The function getline returns false if there are no more lines.
for (std::string str; std::getline(is, str);) {
    // Process the line that has been read.
}

```

If you want to read a fixed number of characters, you can use the stream's member function `read()`:

> 如果您想读取固定数量的字符，可以使用流的成员函数“read（）”：

```cpp
std::ifstream is("foo.txt");
char str[4];

// Read 4 characters from the file.
is.read(str, 4);

```

After executing a read command, you should always check if the error state flag `failbit` has been set, as it indicates whether the operation failed or not. This can be done by calling the file stream's member function `fail()`:

> 执行读取命令后，应始终检查错误状态标志“failbit”是否已设置，因为它指示操作是否失败。这可以通过调用文件流的成员函数“fail（）”来实现：

```cpp
is.read(str, 4); // This operation might fail for any reason.

if (is.fail())
    // Failed to read!

```

## Opening modes

When creating a file stream, you can specify an opening mode. An opening mode is basically a setting to control how the stream opens the file.

> 创建文件流时，可以指定打开模式。打开模式基本上是控制流如何打开文件的设置。

(All modes can be found in the `std::ios` namespace.)

An opening mode can be provided as second parameter to the constructor of a file stream or to its `open()` member function:

> 打开模式可以作为第二个参数提供给文件流的构造函数或其“open（）”成员函数：

```cpp
std::ofstream os("foo.txt", std::ios::out | std::ios::trunc);

std::ifstream is;
is.open("foo.txt", std::ios::in | std::ios::binary);

```

It is to be noted that you have to set `ios::in` or `ios::out` if you want to set other flags as they are not implicitly set by the iostream members although they have a correct default value.

> 需要注意的是，如果要设置其他标志，则必须设置“ios::in”或“ios::：out”，因为它们不是由 iostream 成员隐式设置的，尽管它们具有正确的默认值。

If you don't specify an opening mode, then the following default modes are used:

> 如果未指定打开模式，则使用以下默认模式：

- `ifstream` - `in`
- `ofstream` - `out`
- `fstream` - `in` and `out`

**The file opening modes that you may specify by design are:**

| Mode     | Meaning  | For          | Description                                |  |  |  |  |  |  |
| -------- | -------- | ------------ | ------------------------------------------ | - | - | - | - | - | - |
| `app`    | append   | Output       | Appends data at the end of the file.       |  |  |  |  |  |  |
| `binary` | binary   | Input/Output | Input and output is done in binary.        |  |  |  |  |  |  |
| `in`     | input    | Input        | Opens the file for reading.                |  |  |  |  |  |  |
| `out`    | output   | Output       | Opens the file for writing.                |  |  |  |  |  |  |
| `trunc`  | truncate | Input/Output | Removes contents of the file when opening. |  |  |  |  |  |  |
| `ate`    | at end   | Input        | Goes to the end of the file when opening.  |  |  |  |  |  |  |

**Note:** Setting the `binary` mode lets the data be read/written exactly as-is; not setting it enables the translation of the newline `'\n'` character to/from a platform specific end of line sequence.

> **注意：**设置“二进制”模式可以使数据按原样读取/写入；如果不设置，则可以将换行符“\n”转换为特定于平台的行尾序列。

For example on Windows the end of line sequence is CRLF (`"\r\n"`).<br />
Write: `"\n"`  => `"\r\n"`<br />
Read: `"\r\n"` => `"\n"`

## Reading an ASCII file into a std::string

```cpp
std::ifstream f("file.txt");

if (f)
{
  std::stringstream buffer;
  buffer << f.rdbuf();
  f.close();

  // The content of "file.txt" is available in the string `buffer.str()`
}

```

The [`rdbuf()`](http://en.cppreference.com/w/cpp/io/basic_ios/rdbuf) method returns a pointer to a [`streambuf`](http://en.cppreference.com/w/cpp/io/basic_streambuf) that can be pushed  into `buffer` via the [`stringstream::operator<<`](http://en.cppreference.com/w/cpp/io/basic_ostream/operator_ltlt) member function.

> [`rdbuf（）`](http://en.cppreference.com/w/cpp/io/basic_ios/rdbuf)方法返回指向[`streambuf`]的指针([http://en.cppreference.com/w/cpp/io/basic_streambuf](http://en.cppreference.com/w/cpp/io/basic_streambuf))可以通过[`stringstream:：operator<<`]推送到 `buffer`([http://en.cppreference.com/w/cpp/io/basic_ostream/operator_ltlt](http://en.cppreference.com/w/cpp/io/basic_ostream/operator_ltlt))成员函数。

Another possibility (popularized in [Effective STL](http://rads.stackoverflow.com/amzn/click/0201749629) by [Scott Meyers](http://www.aristeia.com/)) is:

> 另一种可能性（在[有效 STL]中推广([http://rads.stackoverflow.com/amzn/click/0201749629](http://rads.stackoverflow.com/amzn/click/0201749629))作者 [Scott Meyers](http://www.aristeia.com/))是：

```cpp
std::ifstream f("file.txt");

if (f)
{
  std::string str((std::istreambuf_iterator<char>(f)),
                  std::istreambuf_iterator<char>());

  // Operations on `str`...
}

```

This is nice because requires little code (and allows reading a file directly into any STL container, not only strings) but can be slow for big files.

> 这很好，因为它只需要很少的代码（并且允许将文件直接读取到任何 STL 容器中，而不仅仅是字符串），但对于大文件来说可能会很慢。

**NOTE**: the extra parentheses around the first argument to the string constructor are essential to prevent the **most vexing parse** problem.

> **注意**：字符串构造函数的第一个参数周围的额外括号对于防止**最麻烦的解析**问题至关重要。

Last but not least:

```cpp
std::ifstream f("file.txt");

if (f)
{
  f.seekg(0, std::ios::end);
  const auto size = f.tellg();

  std::string str(size, ' ');
  f.seekg(0);
  f.read(&str[0], size); 
  f.close();

  // Operations on `str`...
}

```

which is probably the fastest option (among the three proposed).

## Writing files with non-standard locale settings

If you need to write a file using different locale settings to the default, you can use [`std::locale`](http://en.cppreference.com/w/cpp/locale/locale) and [`std::basic_ios::imbue()`](http://en.cppreference.com/w/cpp/io/basic_ios/imbue) to do that for a specific file stream:

> 如果您需要使用与默认设置不同的区域设置来编写文件，可以使用[`std:：locale`](http://en.cppreference.com/w/cpp/locale/locale)和[`std:：basic_ios:：imbue（）`](http://en.cppreference.com/w/cpp/io/basic_ios/imbue)要对特定文件流执行此操作，请执行以下操作：

**Guidance for use:**

- You should always apply a local to a stream before opening the file.
- Once the stream has been imbued you should not change the locale.

**Reasons for Restrictions:**

Imbuing a file stream with a locale has undefined behavior if the current locale is not state independent or not pointing at the beginning of the file.

> 如果当前区域设置不是独立于状态的或不指向文件的开头，则将文件流与区域设置相融合会产生未定义的行为。

UTF-8 streams (and others) are not state independent.

Also a file stream with a UTF-8 locale may try and read the BOM marker from the file when it is opened; so just opening the file may read characters from the file and it will not be at the beginning.

> 此外，具有 UTF-8 语言环境的文件流可能会在打开文件时尝试从文件中读取 BOM 标记；因此，仅仅打开文件就可以读取文件中的字符，而不是在开始时。

```cpp
#include <iostream>
#include <fstream>
#include <locale>

int main()
{
  std::cout << "User-preferred locale setting is "
            << std::locale("").name().c_str() << std::endl;

  // Write a floating-point value using the user's preferred locale.
  std::ofstream ofs1;
  ofs1.imbue(std::locale(""));
  ofs1.open("file1.txt");
  ofs1 << 78123.456 << std::endl;

  // Use a specific locale (names are system-dependent)
  std::ofstream ofs2;
  ofs2.imbue(std::locale("en_US.UTF-8"));
  ofs2.open("file2.txt");
  ofs2 << 78123.456 << std::endl;

  // Switch to the classic "C" locale
  std::ofstream ofs3;
  ofs3.imbue(std::locale::classic());
  ofs3.open("file3.txt");
  ofs3 << 78123.456 << std::endl;
}

```

Explicitly switching to the classic "C" locale is useful if your program uses a different default locale and you want to ensure a fixed standard for reading and writing files.  With a "C" preferred locale, the example writes

> 如果您的程序使用不同的默认区域设置，并且您希望确保读取和写入文件的固定标准，那么显式切换到经典的“C”区域设置是有用的。对于“C”首选语言环境，示例写道

```cpp
78,123.456
78,123.456
78123.456

```

If, for example, the preferred locale is German and hence uses a different number format, the example writes

> 例如，如果首选的语言环境是德语，因此使用不同的数字格式，则该示例写道

```cpp
78 123,456
78,123.456
78123.456

```

(note the decimal comma in the first line).

## Flushing a stream

File streams are buffered by default, as are many other types of streams. This means that writes to the stream may not cause the underlying file to change immediately. In oder to force all buffered writes to take place immediately, you can **flush** the stream. You can do this either directly by invoking the `flush()` method or through the `std::flush` stream manipulator:

> 默认情况下，文件流是缓冲的，许多其他类型的流也是如此。这意味着对流的写入可能不会导致底层文件立即更改。为了强制所有缓冲写入立即发生，您可以**刷新**流。您可以直接通过调用“flush（）”方法或通过“std:：flush”流操纵器来执行此操作：

```cpp
std::ofstream os("foo.txt");
os << "Hello World!" << std::flush;

char data[3] = "Foo";
os.write(data, 3);
os.flush();

```

There is a stream manipulator `std::endl` that combines writing a newline with flushing the stream:

> 有一个流操纵器“std:：endl”，它结合了写换行和刷新流：

```cpp
// Both following lines do the same thing
os << "Hello World!\n" << std::flush;
os << "Hello world!" << std::endl;

```

Buffering can improve the performance of writing to a stream. Therefore, applications that do a lot of writing should avoid flushing unnecessarily. Contrary, if I/O is done infrequently, applications should consider flushing frequently in order to avoid data getting stuck in the stream object.

> 缓冲可以提高写入流的性能。因此，进行大量写入的应用程序应该避免不必要的刷新。相反，如果 I/O 不频繁，应用程序应该考虑频繁刷新，以避免数据卡在流对象中。

## Reading a file into a container

In the example below we use `std::string` and `operator>>` to read items from the file.

> 在下面的示例中，我们使用“std:：string”和“operator>>”从文件中读取项目。

```

   std::ifstream file("file3.txt");

    std::vector<std::string>  v;

    std::string s;
    while(file >> s) // keep reading until we run out
    {
        v.push_back(s);
    }

```

In the above example we are simply iterating through the file reading one "item" at a time using `operator>>`. This same affect can be achieved using the `std::istream_iterator` which is an input iterator that reads one "item" at a time from the stream. Also most containers can be constructed using two iterators so we can simplify the above code to:

> 在上面的例子中，我们只是使用“operator>>”在文件中迭代，每次读取一个“项”。使用“std:：istream_iterator”也可以实现同样的效果，它是一个输入迭代器，每次从流中读取一个“项”。此外，大多数容器都可以使用两个迭代器构建，因此我们可以将上面的代码简化为：

```

   std::ifstream file("file3.txt");

    std::vector<std::string>  v(std::istream_iterator<std::string>{file},
                                std::istream_iterator<std::string>{});

```

We can extend this to read any object types we like by simply specifying the object we want to read as the template parameter to the `std::istream_iterator`. Thus we can simply extend the above to read lines (rather than words) like this:

> 我们可以通过简单地将要读取的对象指定为“std:：istream_iterator”的模板参数，将其扩展为读取任何我们喜欢的对象类型。因此，我们可以简单地将上面的内容扩展为这样的行（而不是单词）：

```cpp
// Unfortunately there is  no built in type that reads line using >>
// So here we build a simple helper class to do it. That will convert
// back to a string when used in string context.
struct Line
{
    // Store data here
    std::string data;
    // Convert object to string
    operator std::string const&() const {return data;}
    // Read a line from a stream.
    friend std::istream& operator>>(std::istream& stream, Line& line)
    {
        return std::getline(stream, line.data);
    }
};


    std::ifstream file("file3.txt");

    // Read the lines of a file into a container.
    std::vector<std::string>  v(std::istream_iterator<Line>{file},
                                std::istream_iterator<Line>{});

```

## Checking end of file inside a loop condition, bad practice?

[`eof`](http://en.cppreference.com/w/cpp/io/basic_ios/eof) returns `true` only **after** reading the end of file. It does NOT indicate that the next read will be the end of stream.

> [`eof`](http://en.cppreference.com/w/cpp/io/basic_ios/eof)在**读取文件末尾后仅返回“true”。它并不表示下一次读取将是流的末尾。

```cpp
while (!f.eof())
{
  // Everything is OK

  f >> buffer;

  // What if *only* now the eof / fail bit is set?

  /* Use `buffer` */
}

```

You could correctly write:

```cpp
while (!f.eof()) 
{  
  f >> buffer >> std::ws;

  if (f.fail())
    break;

  /* Use `buffer` */
}

```

but

```cpp
while (f >> buffer)
{
  /* Use `buffer` */
}

```

is simpler and less error prone.

Further references:

- [`std::ws`](http://en.cppreference.com/w/cpp/io/manip/ws): discards leading whitespace from an input stream
- [`std::basic_ios::fail`](http://en.cppreference.com/w/cpp/io/basic_ios/fail): returns `true` if an error has occurred on the associated stream

## Closing a file

Explicitly closing a file is rarely necessary in C++, as a file stream will automatically close its associated file in its destructor. However, you should try to limit the lifetime of a file stream object, so that it does not keep the file handle open longer than necessary. For example, this can be done by putting all file operations into an own scope (`{}`):

> 在 C++ 中很少需要显式关闭文件，因为文件流会自动关闭其析构函数中的关联文件。但是，您应该尝试限制文件流对象的生存期，这样它就不会使文件句柄的打开时间超过必要的时间。例如，这可以通过将所有文件操作放入自己的范围（“｛｝”）来实现：

```cpp
std::string const prepared_data = prepare_data();
{
    // Open a file for writing.
    std::ofstream output("foo.txt");

    // Write data.
    output << prepared_data;
}  // The ofstream will go out of scope here.
   // Its destructor will take care of closing the file properly.

```

Calling `close()` explicitly is only necessary if you want to reuse the same `fstream` object later, but don't want to keep the file open in between:

> 只有当您以后想重用同一个“fstream”对象，但又不想在两者之间保持文件打开时，才需要显式调用“close（）”：

```cpp
// Open the file "foo.txt" for the first time.
std::ofstream output("foo.txt");

// Get some data to write from somewhere.
std::string const prepared_data = prepare_data();

// Write data to the file "foo.txt".
output << prepared_data;

// Close the file "foo.txt".
output.close();

// Preparing data might take a long time. Therefore, we don't open the output file stream
// before we actually can write some data to it.
std::string const more_prepared_data = prepare_complex_data();

// Open the file "foo.txt" for the second time once we are ready for writing.
output.open("foo.txt");

// Write the data to the file "foo.txt".
output << more_prepared_data;

// Close the file "foo.txt" once again.
output.close();

```

## Reading a `struct` from a formatted text file.

```cpp
struct info_type
{
    std::string name;
    int age;
    float height;
    
    // we define an overload of operator>> as a friend function which
    // gives in privileged access to private data members 
    friend std::istream& operator>>(std::istream& is, info_type& info)
    {
        // skip whitespace
        is >> std::ws;
        std::getline(is, info.name);
        is >> info.age;
        is >> info.height;
        return is;
    }
};

void func4()
{
    auto file = std::ifstream("file4.txt");

    std::vector<info_type> v;

    for(info_type info; file >> info;) // keep reading until we run out
    {
        // we only get here if the read succeeded
        v.push_back(info);
    }

    for(auto const& info: v)
    {
        std::cout << "  name: " << info.name << '\n';
        std::cout << "   age: " << info.age << " years" << '\n';
        std::cout << "height: " << info.height << "lbs" << '\n';
        std::cout << '\n';
    }
}

```

**file4.txt**

```cpp
Wogger Wabbit
2
6.2
Bilbo Baggins
111
81.3
Mary Poppins
29
154.8

```

**Output:**

```cpp
name: Wogger Wabbit
 age: 2 years
height: 6.2lbs

name: Bilbo Baggins
 age: 111 years
height: 81.3lbs

name: Mary Poppins
 age: 29 years
height: 154.8lbs

```

## Copying a file

```cpp
std::ifstream  src("source_filename", std::ios::binary);
std::ofstream  dst("dest_filename",   std::ios::binary);
dst << src.rdbuf();

```

With C++17 the standard way to copy a file is including the [`<filesystem>`](http://en.cppreference.com/w/cpp/filesystem) header and using [`copy_file`](http://en.cppreference.com/w/cpp/filesystem/copy_file):

> 使用 C++17，复制文件的标准方法是包含[`<filesystem>`](http://en.cppreference.com/w/cpp/filesystem)标头并使用[`copy_file`](http://en.cppreference.com/w/cpp/filesystem/copy_file)：

```cpp
std::fileystem::copy_file("source_filename", "dest_filename");

```

The filesystem library was originally developed as `boost.filesystem` and finally merged to ISO C++ as of C++17.

> 文件系统库最初被开发为“boost.filesystem”，最终在 C++17 时被合并到 ISO C++。
