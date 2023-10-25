---
tip: translate by baidu@2023-10-25 08:23:35
---
---
metaTitle: "C++ | C++ Streams"
description: "String streams, Printing collections with iostream, Reading a file till the end, Parsing files"
---

# C++ Streams



## String streams



[`std::ostringstream`](http://www.cplusplus.com/reference/sstream/ostringstream/) is a class whose objects look like an output stream (that is, you can write to them via `operator<<`), but actually store the writing results, and provide them in the form of a stream.

> [`std:：ostringstream`](http://www.cplusplus.com/reference/sstream/ostringstream/)是一个类，其对象看起来像一个输出流（也就是说，您可以通过`operator<<`对它们进行写入），但实际上存储了写入结果，并以流的形式提供它们。

Consider the following short code:

```cpp
#include <sstream>
#include <string>                                                                                                                          

using namespace std;

int main()
{
    ostringstream ss;
    ss << "the answer to everything is " << 42;
    const string result = ss.str(); 
}   

```

The line

```cpp
ostringstream ss;

```


creates such an object. This object is first manipulated like a regular stream:

> 创建这样一个对象。此对象首先像常规流一样进行操作：

```cpp
ss << "the answer to everything is " << 42;

```

Following that, though, the resulting stream can be obtained like this:

```cpp
const string result = ss.str();

```


(the string `result` will be equal to `"the answer to everything is 42"`).

> （字符串“result”将等于“一切的答案都是42”）。


This is mainly useful when we have a class for which stream serialization has been defined, and for which we want a string form. For example, suppose we have some class

> 当我们有一个已经定义了流序列化的类，并且我们想要一个字符串形式时，这主要是有用的

```cpp
class foo 
{   
    // All sort of stuff here.
};  

ostream &operator<<(ostream &os, const foo &f);

```

To get the string representation of a `foo` object,

```cpp
foo f;

```

we could use

```cpp
ostringstream ss; 
ss << f;
const string result = ss.str();        

```

Then `result` contains the string representation of the `foo` object.



## Printing collections with iostream


### Basic printing

`std::ostream_iterator` allows to print contents of an STL container to any output stream without explicit loops. The second argument of `std::ostream_iterator` constructor sets the delimiter. For example, the following code:

```cpp
std::vector<int> v = {1,2,3,4};
std::copy(v.begin(), v.end(), std::ostream_iterator<int>(std::cout, " ! "));

```

will print

```cpp
1 ! 2 ! 3 ! 4 !

```

### Implicit type cast

`std::ostream_iterator` allows to cast container's content type implicitly. For example, let's tune `std::cout` to print floating-point values with 3 digits after decimal point:

```cpp
std::cout << std::setprecision(3);
std::fixed(std::cout);

```


and instantiate `std::ostream_iterator` with `float`, while the contained values remain `int`:

> 并用“float”实例化“std:：ostream_iterator”，而包含的值保持为“int”：

```cpp
std::vector<int> v = {1,2,3,4};
std::copy(v.begin(), v.end(), std::ostream_iterator<float>(std::cout, " ! "));

```

so the code above yields

```cpp
1.000 ! 2.000 ! 3.000 ! 4.000 !

```

despite `std::vector` holds `int`s.

### Generation and transformation

`std::generate`, `std::generate_n` and `std::transform` functions provide a very powerful tool for on-the-fly data manipulation. For example, having a vector:

```cpp
std::vector<int> v = {1,2,3,4,8,16};

```


we can easily print boolean value of "x is even" statement for each element:

> 我们可以很容易地为每个元素打印“x is even”语句的布尔值：

```cpp
std::boolalpha(std::cout); // print booleans alphabetically
std::transform(v.begin(), v.end(), std::ostream_iterator<bool>(std::cout, " "),
[](int val) {
    return (val % 2) == 0;
});

```

or print the squared element:

```cpp
std::transform(v.begin(), v.end(), std::ostream_iterator<int>(std::cout, " "),
[](int val) {
    return val * val;
});

```

Printing N space-delimited random numbers:

```cpp
const int N = 10;
std::generate_n(std::ostream_iterator<int>(std::cout, " "), N, std::rand);

```

### Arrays


As in the section about reading text files, almost all these considerations may be applied to native arrays. For example, let's print squared values from a native array:

> 在关于读取文本文件的部分中，几乎所有这些注意事项都可以应用于本机数组。例如，让我们打印本机数组中的平方值：

```cpp
int v[] = {1,2,3,4,8,16};
std::transform(v, std::end(v), std::ostream_iterator<int>(std::cout, " "),
[](int val) {
    return val * val;
});

```



## Reading a file till the end


### Reading a text file line-by-line


A proper way to read a text file line-by-line till the end is usually not clear from `ifstream` documentation. Let's consider some common mistakes done by beginner C++ programmers, and a proper way to read the file.

> “ifstream”文档中通常不清楚逐行读取文本文件直到最后的正确方法。让我们考虑一下初学者C++程序员所犯的一些常见错误，以及读取文件的正确方法。

### Lines without whitespace characters


For the sake of simplicity, let's assume that each line in the file contains no whitespace symbols.

> 为了简单起见，我们假设文件中的每一行都不包含空白符号。

`ifstream` has `operator bool()`, which returns true when a stream has no errors and is ready to read. Moreover, `ifstream::operator >>` returns a reference to the stream itself, so we can read and check for EOF (as well as for errors) in one line with very elegant syntax:

```cpp
std::ifstream ifs("1.txt");
std::string s;
while(ifs >> s) {
    std::cout << s << std::endl;
}

```

### Lines with whitespace characters

`ifstream::operator >>` reads the stream until any whitespace character occurs, so the above code will print the words from a line on separate lines. To read everything till the end of line, use `std::getline` instead of `ifstream::operator >>`. `getline` returns reference to the thread it worked with, so the same syntax is available:

```cpp
while(std::getline(ifs, s)) {
    std::cout << s << std::endl;
}

```


Obviously, `std::getline` should also be used for reading a single-line file till the end.

> 显然，“std:：getline”也应该用于读取单行文件直到最后。

### Reading a file into a buffer at once


Finally, let's read the file from the beginning till the end without stopping at any character, including whitespaces and newlines. If we know the exact file size or upper bound of the length is acceptable, we can resize the string and then read:

> 最后，让我们从头读到尾，不要停留在任何字符上，包括空格和换行符。如果我们知道确切的文件大小或长度上限是可以接受的，我们可以调整字符串的大小，然后读取：

```cpp
s.resize(100);
std::copy(std::istreambuf_iterator<char>(ifs), std::istreambuf_iterator<char>(),
    s.begin());

```


Otherwise, we need to insert each character to the end of the string, so `std::back_inserter` is what we need:

> 否则，我们需要将每个字符插入字符串的末尾，因此我们需要“std:：back_inserter”：

```cpp
std::copy(std::istreambuf_iterator<char>(ifs), std::istreambuf_iterator<char>(),
    std::back_inserter(s));

```


Alternatively, it is possible to initialize a collection with stream data, using a constructor with iterator range arguments:

> 或者，也可以使用带有迭代器范围参数的构造函数，用流数据初始化集合：

```cpp
std::vector v(std::istreambuf_iterator<char>(ifs),
    std::istreambuf_iterator<char>());

```


Note that these examples are also applicable if `ifs` is opened as binary file:

> 请注意，如果“ifs”作为二进制文件打开，这些示例也适用：

```cpp
std::ifstream ifs("1.txt", std::ios::binary);

```

### Copying streams

A file may be copied to another file with streams and iterators:

```cpp
std::ofstream ofs("out.file");
std::copy(std::istreambuf_iterator<char>(ifs), std::istreambuf_iterator<char>(),
    std::ostream_iterator<char>(ofs));
ofs.close();

```

or redirected to any other type of stream with a compatible interface. For example Boost.Asio network stream:

```cpp
boost::asio::ip::tcp::iostream stream;
stream.connect("example.com", "http");
std::copy(std::istreambuf_iterator<char>(ifs), std::istreambuf_iterator<char>(),
    std::ostream_iterator<char>(stream));
stream.close();

```

### Arrays


As iterators might be thought of as a generalization of pointers, STL containers in the examples above may be replaced with native arrays. Here is how to parse numbers into array:

> 由于迭代器可能被认为是指针的泛化，所以上面示例中的STL容器可能会被本地数组所取代。以下是如何将数字解析为数组：

```cpp
int arr[100];
std::copy(std::istream_iterator<char>(ifs), std::istream_iterator<char>(), arr);

```


Beware of buffer overflow, as arrays cannot be resized on-the-fly after they were allocated. For example, if the code above will be fed with a file that contains more than 100 integer numbers, it will attempt to write outside the array and run into undefined behavior.

> 小心缓冲区溢出，因为数组在分配后无法动态调整大小。例如，如果上面的代码将被提供一个包含100多个整数的文件，那么它将尝试在数组之外写入，并运行到未定义的行为中。



## Parsing files


### Parsing files into STL containers

`istream_iterator`s are very useful for reading sequences of numbers or other parsable data into STL containers without explicit loops in the code.

Using explicit container size:

```cpp
std::vector<int> v(100);
std::copy(std::istream_iterator<int>(ifs), std::istream_iterator<int>(),
    v.begin());

```

or with inserting iterator:

```cpp
std::vector<int> v;
std::copy(std::istream_iterator<int>(ifs), std::istream_iterator<int>(),
    std::back_inserter(v));

```


Note that the numbers in the input file may be divided by any number of any whitespace characters and newlines.

> 请注意，输入文件中的数字可以除以任意数量的空白字符和换行符。

### Parsing heterogeneous text tables


As `istream::operator>>` reads text until a whitespace symbol, it may be used in `while` condition to parse complex data tables. For example, if we have a file with two real numbers followed by a string (without spaces) on each line:

> 当“istream:：operator>>”读取文本直到空白符号时，它可以在“while”条件中用于解析复杂的数据表。例如，如果我们有一个文件，其中每行有两个实数，后面跟着一个字符串（不带空格）：

```cpp
1.12 3.14 foo
2.1 2.2 barr

```

it may be parsed like this:

```cpp
std::string s;
double a, b;
while(ifs >> a >> b >> s) {
    std::cout << a << " " << b << " " << s << std::endl;
}

```

### Transformation


Any range-manipulating function may be used with `std::istream_iterator` ranges. One of them is `std::transform`, which allows to process data on-the-fly. For example, let's read integer values, multiply them by 3.14 and store the result into floating-point container:

> 任何范围操作函数都可以与“std:：istream_iterator”范围一起使用。其中之一是“std:：transform”，它允许动态处理数据。例如，让我们读取整数值，将它们乘以3.14，并将结果存储到浮点容器中：

```cpp
std::vector<double> v(100);
std::transform(std::istream_iterator<int>(ifs), std::istream_iterator<int>(),
v.begin(),
[](int val) {
    return val * 3.14;
});

```



#### Remarks



Default constructor of `std::istream_iterator` constructs an iterator which represents the end of the stream. Thus, `std::copy(std::istream_iterator<int>(ifs), std::istream_iterator<int>(), ....` means to copy from the current position in `ifs` to the end.

> “std:：istream_iterator”的默认构造函数构造一个迭代器，该迭代器表示流的末尾。因此，`std:：copy（std:：istream_iterator<int>（ifs），std:：distream_itelator<int>[），……`意思是从“如果”中的当前位置复制到末尾。

