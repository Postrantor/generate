---
tip: translate by baidu@2023-10-25 08:24:03
---
---

metaTitle: "C++ | Exceptions"
description: "Catching exceptions, Rethrow (propagate) exception, Best practice: throw by value, catch by const reference, std::uncaught_exceptions, Custom exception, Function Try Block for regular function, Nested exception, Function Try Blocks In constructor, Function Try Blocks In destructor"
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# Exceptions

## Catching exceptions

A `try/catch` block is used to catch exceptions. The code in the `try` section is the code that may throw an exception, and the code in the `catch` clause(s) handles the exception.

> “try/catch”块用于捕获异常。“try”部分中的代码是可能引发异常的代码，而“catch”子句中的代码处理异常。

```cpp
#include <iostream>
#include <string>
#include <stdexcept>

int main() {
  std::string str("foo");
  
  try {
      str.at(10); // access element, may throw std::out_of_range
  } catch (const std::out_of_range& e) {
      // what() is inherited from std::exception and contains an explanatory message
      std::cout << e.what();
  }
}

```

Multiple `catch` clauses may be used to handle multiple exception types. If multiple `catch` clauses are present, the exception handling mechanism tries to match them **in order** of their appearance in the code:

> 多个“catch”子句可以用于处理多个异常类型。如果存在多个“catch”子句，则异常处理机制会尝试按它们在代码中的出现顺序**匹配它们：

```cpp
std::string str("foo");
  
try {
    str.reserve(2); // reserve extra capacity, may throw std::length_error
    str.at(10); // access element, may throw std::out_of_range
} catch (const std::length_error& e) {
    std::cout << e.what();
} catch (const std::out_of_range& e) {
    std::cout << e.what();
}

```

Exception classes which are derived from a common base class can be caught with a single `catch` clause for the common base class. The above example can replace the two `catch` clauses for `std::length_error` and `std::out_of_range` with a single clause for `std:exception`:

> 从公共基类派生的异常类可以用公共基类的单个“catch”子句捕获。上面的例子可以用一个“std:exception”子句替换“std:：length_error”和“std:”out_of_range”的两个“catch”子句：

```cpp
std::string str("foo");
  
try {
    str.reserve(2); // reserve extra capacity, may throw std::length_error
    str.at(10); // access element, may throw std::out_of_range
} catch (const std::exception& e) {
    std::cout << e.what();
}

```

Because the `catch` clauses are tried in order, be sure to write more specific catch clauses first, otherwise your exception handling code might never get called:

> 因为“catch”子句是按顺序尝试的，所以一定要先编写更具体的 catch 子句，否则您的异常处理代码可能永远不会被调用：

```cpp
try {
    /* Code throwing exceptions omitted. */
} catch (const std::exception& e) {
    /* Handle all exceptions of type std::exception. */
} catch (const std::runtime_error& e) {
    /* This block of code will never execute, because std::runtime_error inherits
       from std::exception, and all exceptions of type std::exception were already
       caught by the previous catch clause. */
}

```

Another possibility is the catch-all handler, which will catch any thrown object:

> 另一种可能性是 catch-all 处理程序，它将捕获任何抛出的对象：

```cpp
try {
    throw 10;
} catch (...) {
    std::cout << "caught an exception";
}

```

## Rethrow (propagate) exception

Sometimes you want to do something with the exception you catch (like write to log or print a warning) and let it bubble up to the upper scope to be handled. To do so, you can rethrow any exception you catch:

> 有时，您想对捕获的异常执行某些操作（例如写入日志或打印警告），并让它上升到要处理的上限。为此，您可以重新抛出捕获到的任何异常：

```cpp
try {
    ... // some code here
} catch (const SomeException& e) {
    std::cout << "caught an exception";
    throw;
}

```

Using `throw;` without arguments will re-throw the currently caught exception.

> 使用 `throw；` 如果没有参数，将重新抛出当前捕获的异常。

To rethrow a managed `std::exception_ptr`, the C++ Standard Library has the `rethrow_exception` function that can be used by including the `<exception>` header in your program.

> 要重新抛出托管的“std:：exception_ptr”，C++ 标准库具有“rethrow_exception”函数，可通过在程序中包含“＜exception＞”标头来使用该函数。

```cpp
#include <iostream>
#include <string>
#include <exception>
#include <stdexcept>
 
void handle_eptr(std::exception_ptr eptr) // passing by value is ok
{
    try {
        if (eptr) {
            std::rethrow_exception(eptr);
        }
    } catch(const std::exception& e) {
        std::cout << "Caught exception \"" << e.what() << "\"\n";
    }
}
 
int main()
{
    std::exception_ptr eptr;
    try {
        std::string().at(1); // this generates an std::out_of_range
    } catch(...) {
        eptr = std::current_exception(); // capture
    }
    handle_eptr(eptr);
} // destructor for std::out_of_range called here, when the eptr is destructed

```

## Best practice: throw by value, catch by const reference

In general, it is considered good practice to throw by value (rather than by pointer), but catch by (const) reference.

> 通常，按值抛出（而不是按指针），但按（const）引用捕获被认为是一种好的做法。

```cpp
try {
    // throw new std::runtime_error("Error!");   // Don't do this!
    // This creates an exception object
    // on the heap and would require you to catch the
    // pointer and manage the memory yourself. This can
    // cause memory leaks!
    
    throw std::runtime_error("Error!");
} catch (const std::runtime_error& e) {
    std::cout << e.what() << std::endl;
}

```

One reason why catching by reference is a good practice is that it eliminates the need to reconstruct the object when being passed to the catch block (or when propagating through to other catch blocks). Catching by reference also allows the exceptions to be handled polymorphically and avoids object slicing. However, if you are rethrowing an exception (like `throw e;`, see example below), you can still get object slicing because the `throw e;` statement makes a copy of the exception as whatever type is declared:

> 通过引用捕获之所以是一种良好的做法，一个原因是它消除了在将对象传递到捕获块时（或在传播到其他捕获块时）重构对象的需要。通过引用捕获还允许以多态方式处理异常，并避免对象切片。但是，如果您正在重新抛出一个异常（如“throw e；”，请参阅下面的示例），您仍然可以获得对象切片，因为“throw e；”语句将异常复制为声明的任何类型：

```cpp
#include <iostream>

struct BaseException {
    virtual const char* what() const { return "BaseException"; }
};

struct DerivedException : BaseException {
    // "virtual" keyword is optional here
    virtual const char* what() const { return "DerivedException"; }
};

int main(int argc, char** argv) {
    try {
        try {
            throw DerivedException();
        } catch (const BaseException& e) {
            std::cout << "First catch block: " << e.what() << std::endl;
            // Output ==> First catch block: DerivedException

            throw e; // This changes the exception to BaseException
                     // instead of the original DerivedException!
        }
    } catch (const BaseException& e) {
        std::cout << "Second catch block: " << e.what() << std::endl;
        // Output ==> Second catch block: BaseException
    }
    return 0;
}

```

If you are sure that you are not going to do anything to change the exception (like add information or modify the message), catching by const reference allows the compiler to make optimizations and can improve performance. But this can still cause object splicing (as seen in the example above).

> 如果您确信不会对异常进行任何更改（如添加信息或修改消息），则通过 const 引用捕获可以使编译器进行优化并提高性能。但这仍然会导致对象拼接（如上面的示例所示）。

**Warning:** Beware of throwing unintended exceptions in `catch` blocks, especially related to allocating extra memory or resources. For example, constructing `logic_error`, `runtime_error` or their subclasses might throw `bad_alloc` due to memory running out when copying the exception string, I/O streams might throw during logging with respective exception masks set, etc.

> **警告：**小心在“catch”块中抛出意外异常，尤其是与分配额外内存或资源有关的异常。例如，由于复制异常字符串时内存耗尽，构造“logic_error”、“runtime_error”或其子类可能会抛出“bad_alloc”，I/O 流可能会在设置了相应异常掩码的日志记录过程中抛出，等等。

## std::uncaught_exceptions

C++17 introduces `int std::uncaught_exceptions()` (to replace the limited `bool std::uncaught_exception()`) to know how many exceptions are currently uncaught.

> C++17 引入了“int std:：uncaught_exceptions（）”（以替换有限的“bool std:：unaught_eexception（）”），以了解当前有多少异常未被捕获。

That allows for a class to determine if it is destroyed during a stack unwinding or not.

> 这允许类确定它是否在堆栈展开过程中被销毁。

```cpp
#include <exception>
#include <string>
#include <iostream>

// Apply change on destruction:
// Rollback in case of exception (failure)
// Else Commit (success)
class Transaction
{
public:
    Transaction(const std::string& s) : message(s) {}
    Transaction(const Transaction&) = delete;
    Transaction& operator =(const Transaction&) = delete;
    void Commit() { std::cout << message << ": Commit\n"; }
    void RollBack() noexcept(true) { std::cout << message << ": Rollback\n"; }

    // ...

    ~Transaction() {
        if (uncaughtExceptionCount == std::uncaught_exceptions()) {
            Commit(); // May throw.
        } else { // current stack unwinding
            RollBack();
        }
    }

private:
    std::string message;
    int uncaughtExceptionCount = std::uncaught_exceptions();
};

class Foo
{
public:
    ~Foo() {
        try {
            Transaction transaction("In ~Foo"); // Commit,
                                            // even if there is an uncaught exception
            //...
        } catch (const std::exception& e) {
            std::cerr << "exception/~Foo:" << e.what() << std::endl;
        }
    }
};

int main()
{
    try {
        Transaction transaction("In main"); // RollBack
        Foo foo; // ~Foo commit its transaction.
        //...
        throw std::runtime_error("Error");
    } catch (const std::exception& e) {
        std::cerr << "exception/main:" << e.what() << std::endl;
    }
}

```

Output:

```cpp
In ~Foo: Commit
In main: Rollback
exception/main:Error

```

## Custom exception

You shouldn't throw raw values as exceptions, instead use one of the standard exception classes or make your own.

> 您不应该将原始值作为异常抛出，而是使用一个标准异常类或创建自己的异常类。

Having your own exception class inherited from `std::exception` is a good way to go about it. Here's a custom exception class which directly inherits from `std::exception`:

> 从“std:：exception”继承自己的异常类是一个很好的方法

```cpp
#include <exception>

class Except: virtual public std::exception {
    
protected:

    int error_number;               ///< Error number
    int error_offset;               ///< Error offset
    std::string error_message;      ///< Error message
    
public:

    /** Constructor (C++ STL string, int, int).
     *  @param msg The error message
     *  @param err_num Error number
     *  @param err_off Error offset
     */
    explicit 
    Except(const std::string& msg, int err_num, int err_off):
        error_number(err_num),
        error_offset(err_off),
        error_message(msg)
        {}

    /** Destructor.
     *  Virtual to allow for subclassing.
     */
    virtual ~Except() throw () {}

    /** Returns a pointer to the (constant) error description.
     *  @return A pointer to a const char*. The underlying memory
     *  is in possession of the Except object. Callers must
     *  not attempt to free the memory.
     */
    virtual const char* what() const throw () {
       return error_message.c_str();
    }
    
    /** Returns error number.
     *  @return #error_number
     */
    virtual int getErrorNumber() const throw() {
        return error_number;
    }
    
    /**Returns error offset.
     * @return #error_offset
     */
    virtual int getErrorOffset() const throw() {
        return error_offset;
    }

};

```

An example throw catch:

```cpp
try {
    throw(Except("Couldn't do what you were expecting", -12, -34));
} catch (const Except& e) {
    std::cout<<e.what()
             <<"\nError number: "<<e.getErrorNumber()
             <<"\nError offset: "<<e.getErrorOffset();
}

```

As you are not only just throwing a dumb error message, also some other values representing what the error exactly was, your error handling becomes much more efficient and meaningful.

> 由于您不仅抛出了一个愚蠢的错误消息，还抛出了一些其他值来表示错误的确切内容，因此您的错误处理将变得更加高效和有意义。

There's an exception class that let's you handle error messages nicely :`std::runtime_error`

> 有一个异常类可以让您很好地处理错误消息：`std:：runtime_error`

You can inherit from this class too:

```cpp
#include <stdexcept>

class Except: virtual public std::runtime_error {
    
protected:

    int error_number;               ///< Error number
    int error_offset;               ///< Error offset
    
public:

    /** Constructor (C++ STL string, int, int).
     *  @param msg The error message
     *  @param err_num Error number
     *  @param err_off Error offset
     */
    explicit 
    Except(const std::string& msg, int err_num, int err_off):
        std::runtime_error(msg)
        {
            error_number = err_num;
            error_offset = err_off;
            
        }

    /** Destructor.
     *  Virtual to allow for subclassing.
     */
    virtual ~Except() throw () {}
    
    /** Returns error number.
     *  @return #error_number
     */
    virtual int getErrorNumber() const throw() {
        return error_number;
    }
    
    /**Returns error offset.
     * @return #error_offset
     */
    virtual int getErrorOffset() const throw() {
        return error_offset;
    }

};

```

Note that I haven't overridden the `what()` function from the base class (`std::runtime_error`) i.e we will be using the base class's version of `what()`. You can override it if you have further agenda.

> 请注意，我还没有重写基类的“what（）”函数（“std:：runtime_error”），即我们将使用基类的“what（））”版本。如果你有进一步的议程，你可以覆盖它。

## Function Try Block for regular function

```cpp
void function_with_try_block() 
try
{
    // try block body
} 
catch (...) 
{ 
    // catch block body
}

```

Which is equivalent to

```cpp
void function_with_try_block() 
{
    try
    {
        // try block body
    } 
    catch (...) 
    { 
        // catch block body
    }
}

```

Note that for constructors and destructors, the behavior is different as the catch block  re-throws an exception anyway (the caught one if there is no other throw in the catch block body).

> 请注意，对于构造函数和析构函数，行为是不同的，因为 catch 块无论如何都会重新抛出异常（如果 catch 块体中没有其他抛出，则为捕获的异常）。

The function `main` is allowed to have a function try block like any other function, but `main`'s function try block will not catch exceptions that occur during the construction of a non-local static variable or the destruction of any static variable. Instead, `std::terminate` is called.

> 允许函数“main”像任何其他函数一样具有函数 try 块，但“main”的函数 try 框不会捕获在构造非局部静态变量或销毁任何静态变量期间发生的异常。相反，会调用“std:：terminate”。

## Nested exception

During exception handling there is a common use case when you catch a generic exception from a low-level function (such as a filesystem error or data transfer error) and throw a more specific high-level exception which indicates that some high-level operation could not be performed (such as being unable to publish a photo on Web). This allows exception handling to react to specific problems with high level operations and also allows, having only error an message, the programmer to find a place in the application where an exception occurred. Downside of this solution is that exception callstack is truncated and original exception is lost. This forces developers to manually include text of original exception into a newly created one.

> 在异常处理过程中，有一种常见的用例是，您从低级函数中捕获一般异常（如文件系统错误或数据传输错误），并抛出更具体的高级异常，这表明某些高级操作无法执行（如无法在 Web 上发布照片）。这允许异常处理通过高级操作对特定问题做出反应，也允许程序员在只有错误消息的情况下在应用程序中找到发生异常的位置。这个解决方案的缺点是异常调用堆栈被截断，原始异常丢失。这迫使开发人员手动将原始异常的文本包含到新创建的异常中。

Nested exceptions aim to solve the problem by attaching low-level exception, which describes the cause, to a high level exception, which describes what it means in this particular case.

> 嵌套异常旨在通过将描述原因的低级异常附加到描述在这种特定情况下意味着什么的高级异常来解决问题。

`std::nested_exception` allows to nest exceptions thanks to `std::throw_with_nested`:

```cpp
#include <stdexcept>
#include <exception>
#include <string>
#include <fstream>
#include <iostream>

struct MyException
{
    MyException(const std::string& message) : message(message) {}
    std::string message;
};

void print_current_exception(int level)
{
    try {
        throw;
    } catch (const std::exception& e) {
        std::cerr << std::string(level, ' ') << "exception: " << e.what() << '\n';
    } catch (const MyException& e) {
        std::cerr << std::string(level, ' ') << "MyException: " << e.message << '\n';
    } catch (...) {
        std::cerr << "Unkown exception\n";
    }
}

void print_current_exception_with_nested(int level =  0)
{
    try {
        throw;
    } catch (...) {
        print_current_exception(level);
    }    
    try {
        throw;
    } catch (const std::nested_exception& nested) {
        try {
            nested.rethrow_nested();
        } catch (...) {
            print_current_exception_with_nested(level + 1); // recursion
        }
    } catch (...) {
        //Empty // End recursion
    }
}

// sample function that catches an exception and wraps it in a nested exception
void open_file(const std::string& s)
{
    try {
        std::ifstream file(s);
        file.exceptions(std::ios_base::failbit);
    } catch(...) {
        std::throw_with_nested(MyException{"Couldn't open " + s});
    }
}
 
// sample function that catches an exception and wraps it in a nested exception
void run()
{
    try {
        open_file("nonexistent.file");
    } catch(...) {
        std::throw_with_nested( std::runtime_error("run() failed") );
    }
}
 
// runs the sample function above and prints the caught exception
int main()
{
    try {
        run();
    } catch(...) {
        print_current_exception_with_nested();
    }
}

```

Possible output:

```cpp
exception: run() failed
 MyException: Couldn't open nonexistent.file
  exception: basic_ios::clear

```

If you work only with exceptions inherited from `std::exception`, code can even be simplified.

> 如果只处理从“std:：exception”继承的异常，代码甚至可以简化。

## Function Try Blocks In constructor

The only way to catch exception in initializer list:

```cpp
struct A : public B
{
    A() try : B(), foo(1), bar(2)
    {
        // constructor body 
    }
    catch (...)
    {
        // exceptions from the initializer list and constructor are caught here
        // if no exception is thrown here
        // then the caught exception is re-thrown.
    }
 
private:
    Foo foo;
    Bar bar;
};

```

## Function Try Blocks In destructor

```cpp
struct A
{
    ~A() noexcept(false) try
    {
        // destructor body 
    }
    catch (...)
    {
        // exceptions of destructor body are caught here
        // if no exception is thrown here
        // then the caught exception is re-thrown.
    }
};

```

Note that, although this is possible, one needs to be very careful with throwing from destructor, as if a destructor called during stack unwinding throws an exception, `std::terminate` is called.

> 请注意，尽管这是可能的，但从析构函数抛出时需要非常小心，就好像在堆栈展开过程中调用的析构函数会抛出异常一样，即调用“std:：terminate”。
