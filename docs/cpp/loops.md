---
tip: translate by baidu@2023-10-25 08:27:19
---
---

metaTitle: "C++ | Loops"
description: "Range-Based For, For loop, While loop, Do-while loop, Loop Control statements : Break and Continue, Declaration of variables in conditions, Range-for over a sub-range"
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# Loops

A loop statement executes a group of statements repeatedly until a condition is met.

> 循环语句重复执行一组语句，直到满足条件为止。
> There are 3 types of primitive loops in C++: for, while, and do...while.

## Range-Based For

`for` loops can be used to iterate over the elements of a iterator-based range, without using a numeric index or directly accessing the iterators:

```cpp
vector<float> v = {0.4f, 12.5f, 16.234f};

for(auto val: v)
{
    std::cout << val << " ";
}

std::cout << std::endl;

```

This will iterate over every element in `v`, with `val` getting the value of the current element. The following statement:

> 这将遍历“v”中的每个元素，“val”将获得当前元素的值。以下声明：

```cpp
for (for-range-declaration : for-range-initializer ) statement

```

is equivalent to:

```cpp
{
    auto&& __range = for-range-initializer;
    auto __begin = begin-expr, __end = end-expr;
    for (; __begin != __end; ++__begin) {
        for-range-declaration = *__begin;
        statement
    }
}

```

```cpp
{
    auto&& __range = for-range-initializer;
    auto __begin = begin-expr;
    auto __end = end-expr; // end is allowed to be a different type than begin in C++17
    for (; __begin != __end; ++__begin) {
        for-range-declaration = *__begin;
        statement
    }
}

```

This change was introduced for the planned support of Ranges TS in C++20.

In this case, our loop is equivalent to:

```cpp
{
    auto&& __range = v;
    auto __begin = v.begin(), __end = v.end();
    for (; __begin != __end; ++__begin) {
        auto val = *__begin;
        std::cout << val << " ";
    }
}

```

Note that `auto val` declares a value type, which will be a copy of a value stored in the range (we are copy-initializing it from the iterator as we go). If the values stored in the range are expensive to copy, you may want to use `const auto &val`. You are also not required to use `auto`; you can use an appropriate typename, so long as it is implicitly convertible from the range's value type.

> 请注意，“auto-val”声明了一个值类型，它将是存储在范围中的值的副本（我们正在从迭代器对其进行复制初始化）。如果存储在该范围中的值复制成本很高，则可能需要使用“const auto&val”。您也不需要使用“auto”；您可以使用适当的 typename，只要它可以从范围的值类型隐式转换。

If you need access to the iterator, range-based for cannot help you (not without some effort, at least).

> 如果您需要访问迭代器，基于范围的 for 无法帮助您（至少需要付出一些努力）。

If you wish to reference it, you may do so:

```cpp
vector<float> v = {0.4f, 12.5f, 16.234f};

for(float &val: v)
{
    std::cout << val << " ";
}

```

You could iterate on `const` reference if you have `const` container:

```cpp
const vector<float> v = {0.4f, 12.5f, 16.234f};

for(const float &val: v)
{
    std::cout << val << " ";
}

```

One would use forwarding references when the sequence iterator returns a proxy object and you need to operate on that object in a non-`const` way. Note: it will most likely confuse readers of your code.

> 当序列迭代器返回一个代理对象，并且您需要以非常量的方式对该对象进行操作时，可以使用转发引用。注意：这很可能会让代码的读者感到困惑。

```cpp
vector<bool> v(10);

for(auto&& val: v)
{
    val = true;
}

```

The "range" type provided to range-based `for` can be one of the following:

> 提供给基于范围的“for”的“range”类型可以是以下类型之一：

<li>
Language arrays:

```cpp
float arr[] = {0.4f, 12.5f, 16.234f};

for(auto val: arr)
{
    std::cout << val << " ";
}

```

Note that allocating a dynamic array does not count:

```cpp
float *arr = new float[3]{0.4f, 12.5f, 16.234f};

for(auto val: arr) //Compile error.
{
    std::cout << val << " ";
}

```

</li>
<li>

Any type which has member functions `begin()` and `end()`, which return iterators to the elements of the type. The standard library containers qualify, but user-defined types can be used as well:

> 任何具有成员函数“begin（）”和“end（）”的类型，它们将迭代器返回到该类型的元素。标准库容器符合条件，但也可以使用用户定义的类型：

```cpp
struct Rng
{
    float arr[3];

    // pointers are iterators
    const float* begin() const {return &arr[0];}
    const float* end() const   {return &arr[3];}
    float* begin() {return &arr[0];}
    float* end()   {return &arr[3];}
};

int main()
{
    Rng rng = {{0.4f, 12.5f, 16.234f}};

    for(auto val: rng)
    {
        std::cout << val << " ";
    }
}

```

</li>
<li>

Any type which has non-member `begin(type)` and `end(type)` functions which can found via argument dependent lookup, based on `type`. This is useful for creating a range type without having to modify class type itself:

> 任何具有非成员“begin（type）”和“end（type））”函数的类型，这些函数可以基于“type”通过依赖于参数的查找找到。这对于创建范围类型而不必修改类类型本身非常有用：

```cpp
namespace Mine
{
    struct Rng {float arr[3];};

    // pointers are iterators
    const float* begin(const Rng &rng) {return &rng.arr[0];}
    const float* end(const Rng &rng) {return &rng.arr[3];}
    float* begin(Rng &rng) {return &rng.arr[0];}
    float* end(Rng &rng) {return &rng.arr[3];}
}

int main()
{
    Mine::Rng rng = {{0.4f, 12.5f, 16.234f}};

    for(auto val: rng)
    {
        std::cout << val << " ";
    }
}

```

</li>

## For loop

A `for` loop executes statements in the `loop body`, while the loop `condition` is true. Before the loop `initialization statement` is executed exactly once. After each cycle, the `iteration execution` part is executed.

> “for”循环执行“循环主体”中的语句，而循环“条件”为 true。在循环之前，“初始化语句”只执行一次。在每个循环之后，执行“迭代执行”部分。

A `for` loop is defined as follows:

```cpp
for (/*initialization statement*/; /*condition*/; /*iteration execution*/)
{
    // body of the loop
}

```

Explanation of the placeholder statements:

- `initialization statement`: This statement gets executed only once, at the beginning of the `for` loop. You can enter a declaration of multiple variables of one type, such as `int i = 0, a = 2, b = 3`. These variables are only valid in the scope of the loop. Variables defined before the loop with the same name are hidden during execution of the loop.

> -“initialization statement”：此语句只执行一次，位于“for”循环的开头。您可以输入一种类型的多个变量的声明，例如“int i=0，a=2，b=3”。这些变量仅在循环的范围内有效。在循环执行过程中，在循环之前定义的同名变量将被隐藏。

- `condition`: This statement gets evaluated ahead of each **loop body** execution, and aborts the loop if it evaluates to `false`.

> -“condition”：此语句在每次执行**循环体**之前进行求值，如果求值结果为“false”，则中止循环。

- `iteration execution`: This statement gets executed after the loop **body**, ahead of the next **condition** evaluation, unless the `for` loop is aborted in the **body** (by `break`, `goto`, `return` or an exception being thrown). You can enter multiple statements in the `iteration execution` part, such as `a++, b+=10, c=b+a`.

> -“迭代执行”：此语句在循环**正文**之后执行，在下一个**条件**求值之前执行，除非“for”循环在**正文**中中止（通过“break”、“goto”、“return”或抛出异常）。您可以在“迭代执行”部分输入多个语句，例如“a++，b+=10，c=b+a”。

The rough equivalent of a `for` loop, rewritten as a `while` loop is:

```cpp
/*initialization*/
while (/*condition*/)
{
    // body of the loop; using 'continue' will skip to increment part below
    /*iteration execution*/
}

```

The most common case for using a `for` loop is to execute statements a specific number of times. For example, consider the following:

> 使用“for”循环最常见的情况是执行特定次数的语句。例如，考虑以下内容：

```cpp
for(int i = 0; i < 10; i++) {
    std::cout << i << std::endl;
}

```

A valid loop is also:

```cpp
for(int a = 0, b = 10, c = 20; (a+b+c < 100); c--, b++, a+=c) {
    std::cout << a << " " << b << " " << c << std::endl; 
}

```

An example of hiding declared variables before a loop is:

```cpp
int i = 99; //i = 99
for(int i = 0; i < 10; i++) { //we declare a new variable i
    //some operations, the value of i ranges from 0 to 9 during loop execution
}
//after the loop is executed, we can access i with value of 99

```

But if you want to use the already declared variable and not hide it, then omit the declaration part:

> 但是，如果您希望使用已声明的变量而不隐藏它，则可以省略声明部分：

```cpp
int i = 99; //i = 99
for(i = 0; i < 10; i++) { //we are using already declared variable i
    //some operations, the value of i ranges from 0 to 9 during loop execution
}
//after the loop is executed, we can access i with value of 10

```

Notes:

- The initialization and increment statements can perform operations unrelated to the condition statement, or nothing at all - if you wish to do so. But for readability reasons, it is best practice to only perform operations directly relevant to the loop.

> -initialization 和 increment 语句可以执行与条件语句无关的操作，或者根本不执行任何操作（如果您愿意的话）。但出于可读性的原因，最好只执行与循环直接相关的操作。

- A variable declared in the initialization statement is visible only inside the scope of the `for` loop and is released upon termination of the loop.

> -初始化语句中声明的变量仅在“for”循环的范围内可见，并在循环结束时释放。

- Don't forget that the variable which was declared in the `initialization statement` can be modified during the loop, as well as the variable checked in the `condition`.

> -不要忘记，在“initialization statement”中声明的变量以及在“condition”中检查的变量都可以在循环期间进行修改。

Example of a loop which counts from 0 to 10:

```cpp
for (int counter = 0; counter <= 10; ++counter)
{
    std::cout << counter << '\n';
}
// counter is not accessible here (had value 11 at the end)

```

Explanation of the code fragments:

- `int counter = 0` initializes the variable `counter` to 0. (This variable can only be used inside of the `for` loop.)

> -“int counter=0”将变量“counter”初始化为 0。（此变量只能在“for”循环内部使用。）

- `counter <= 10` is a Boolean condition that checks whether `counter` is less than or equal to 10. If it is `true`, the loop executes. If it is `false`, the loop ends.

> -“counter＜＝10”是一个布尔条件，用于检查“counter”是否小于或等于 10。如果为“true”，则执行循环。如果为“false”，则循环结束。

- `++counter` is an increment operation that increments the value of `counter` by 1 ahead of the next condition check.

> -“++ 计数器”是在下一个条件检查之前将“计数器”的值增加 1 的增量操作。

By leaving all statements empty, you can create an infinite loop:

```cpp
// infinite loop
for (;;)
    std::cout << "Never ending!\n";

```

The `while` loop equivalent of the above is:

```cpp
// infinite loop
while (true)
    std::cout << "Never ending!\n";

```

However, an infinite loop can still be left by using the statements `break`, `goto`, or `return` or by throwing an exception.

> 但是，通过使用语句“break”、“goto”或“return”或抛出异常，仍然可以留下无限循环。

The next common example of iterating over all elements from an STL collection (e.g., a `vector`) without using the `<algorithm>` header is:

> 在不使用“＜algorithm＞”标头的情况下迭代 STL 集合中的所有元素（例如，“向量”）的下一个常见示例是：

```cpp
std::vector<std::string> names = {"Albert Einstein", "Stephen Hawking", "Michael Ellis"};
for(std::vector<std::string>::iterator it = names.begin(); it != names.end(); ++it) {
    std::cout << *it << std::endl;
}

```

## While loop

A `while` loop executes statements repeatedly until the given condition evaluates to `false`. This control statement is used when it is not known, in advance, how many times a block of code is to be executed.

> “while”循环重复执行语句，直到给定条件的计算结果为“false”。当事先不知道一个代码块要执行多少次时，就会使用此控制语句。

For example, to print all the numbers from 0 up to 9, the following code can be used:

> 例如，要打印从 0 到 9 的所有数字，可以使用以下代码：

```cpp
int i = 0;
while (i < 10)
{
    std::cout << i << " ";
    ++i; // Increment counter
}
std::cout << std::endl; // End of line; "0 1 2 3 4 5 6 7 8 9" is printed to the console

```

Note that since C++17, the first 2 statements can be combined

```cpp
while (int i = 0; i < 10)
//... The rest is the same

```

To create an infinite loop, the following construct can be used:

```cpp
while (true)
{
    // Do something forever (however, you can exit the loop by calling 'break'
}

```

There is another variant of `while` loops, namely the `do...while` construct. See the [do-while loop example](http://stackoverflow.com/documentation/c%2B%2B/589/loops/7821/do-while-loop#t=201607221948036580158) for more information.

> while 循环还有另一种变体，即 do。。。而 ` 构造。请参阅[do while loop 示例]([http://stackoverflow.com/documentation/c%2b%2b/589/loops/7821/do](http://stackoverflow.com/documentation/c%2B%2B/589/loops/7821/do)-而循环#t=201607221948036580158）。

## Do-while loop

A **do-while** loop is very similar to a **while** loop, except that the condition is checked at the end of each cycle, not at the start. The loop is therefore guaranteed to execute at least once.

> **do-while** 循环与 **while** 循环非常相似，只是在每个循环结束时检查条件，而不是在开始时检查条件。因此，保证循环至少执行一次。

The following code will print `0`, as the condition will evaluate to `false` at the end of the first iteration:

> 以下代码将打印“0”，因为条件在第一次迭代结束时将计算为“false”：

```cpp
int i =0;
do
{
    std::cout << i;
    ++i; // Increment counter
}
while (i < 0);
std::cout << std::endl; // End of line; 0 is printed to the console

```

Note: Do not forget the semicolon at the end of `while(condition);`, which is needed in the **do-while** construct.

> 注意：不要忘记 `while（condition）；` 末尾的分号，这在 **dowhile** 构造中是需要的。

In contrast to the **do-while** loop, the following will not print anything, because the condition evaluates to `false` at the beginning of the first iteration:

> 与 **dowhile** 循环相反，以下循环不会打印任何内容，因为条件在第一次迭代开始时计算为“false”：

```cpp
int i =0;
while (i < 0)
{
    std::cout << i;
    ++i; // Increment counter
}    
std::cout << std::endl; // End of line; nothing is printed to the console

```

Note: A **while** loop can be exited without the condition becoming false by using a `break`, `goto`, or `return` statement.

> 注意：通过使用“break”、“goto”或“return”语句，可以在条件不为 false 的情况下退出***while** 循环。

```cpp
int i = 0;
do
{
    std::cout << i;
    ++i; // Increment counter
    if (i > 5) 
    {
        break;
    }
}
while (true);
std::cout << std::endl; // End of line; 0 1 2 3 4 5 is printed to the console

```

A trivial **do-while** loop is also occasionally used to write macros that require their own scope (in which case the trailing semicolon is omitted from the macro definition and required to be provided by the user):

> 一个琐碎的***do while** 循环偶尔也用于编写需要自己作用域的宏（在这种情况下，宏定义中省略了尾部分号，并要求用户提供）：

```cpp
#define BAD_MACRO(x) f1(x); f2(x); f3(x);

// Only the call to f1 is protected by the condition here
if (cond) BAD_MACRO(var);

#define GOOD_MACRO(x) do { f1(x); f2(x); f3(x); } while(0)

// All calls are protected here
if (cond) GOOD_MACRO(var);

```

## Loop Control statements : Break and Continue

Loop control statements are used to change the flow of execution from its normal sequence. When execution leaves a scope, all automatic objects that were created in that scope are destroyed. The `break` and `continue` are loop control statements.

> 循环控制语句用于更改正常序列的执行流。当执行离开一个作用域时，在该作用域中创建的所有自动对象都将被销毁。“break”和“continue”是循环控制语句。

The `break` statement terminates a loop without any further consideration.

> “break”语句终止了一个循环而不作任何进一步考虑。

```cpp
for (int i = 0; i < 10; i++)
{
    if (i == 4)
        break; // this will immediately exit our loop
    std::cout << i << '\n';
}

```

The above code will print out:

```cpp
1
2
3

```

The `continue` statement does not immediately exit the loop, but rather skips the rest of the loop body and goes to the top of the loop (including checking the condition).

> “continue”语句不会立即退出循环，而是跳过循环体的其余部分，转到循环的顶部（包括检查条件）。

```cpp
for (int i = 0; i < 6; i++)
{
    if (i % 2 == 0) // evaluates to true if i is even
        continue; // this will immediately go back to the start of the loop
    /* the next line will only be reached if the above "continue" statement 
       does not execute  */
    std::cout << i << " is an odd number\n";
}

```

The above code will print out:

```cpp
1 is an odd number
3 is an odd number
5 is an odd number

```

Because such control flow changes are sometimes difficult for humans to easily understand, `break` and `continue` are used sparingly.  More straightforward implementation are usually easier to read and understand.  For example, the first `for` loop with the `break` above might be rewritten as:

> 因为这种控制流的变化有时很难让人类轻易理解，所以“中断”和“继续”的使用很少。更直接的实现通常更容易阅读和理解。例如，上面带有“break”的第一个“For”循环可以重写为：

```cpp
for (int i = 0; i < 4; i++)
{
    std::cout << i << '\n';
}

```

The second example with `continue` might be rewritten as:

```cpp
for (int i = 0; i < 6; i++)
{
    if (i % 2 != 0) {
        std::cout << i << " is an odd number\n";
    }
}

```

## Declaration of variables in conditions

In the condition of the `for` and `while` loops, it's also permitted to declare an object.  This object will be considered to be in scope until the end of the loop, and will persist through each iteration of the loop:

> 在“for”和“while”循环的条件下，也允许声明对象。此对象将被视为在范围内，直到循环结束，并将在循环的每次迭代中持续存在：

```cpp
for (int i = 0; i < 5; ++i) {
    do_something(i);
}
// i is no longer in scope.

for (auto& a : some_container) {
    a.do_something();
}
// a is no longer in scope.

while(std::shared_ptr<Object> p = get_object()) {
   p->do_something();
}
// p is no longer in scope.

```

However, it is not permitted to do the same with a `do...while` loop; instead, declare the variable before the loop, and (optionally) enclose both the variable and the loop within a local scope if you want the variable to go out of scope after the loop ends:

> 但是，不允许用“do。。。while `loop；相反，在循环之前声明变量，如果希望变量在循环结束后超出范围，则（可选）将变量和循环都封装在本地范围内：

```cpp
//This doesn't compile
do {
    s = do_something();
} while (short s > 0);

// Good
short s;
do {
    s = do_something();
} while (s > 0);

```

This is because the **statement** portion of a `do...while` loop (the loop's body) is evaluated before the **expression** portion (the `while`) is reached, and thus, any declaration in the **expression** will not be visible during the first iteration of the loop.

> 这是因为“do…”的**语句**部分。。。而“loop”（循环的主体）是在到达**表达式**部分（“while”）之前求值的，因此，在循环的第一次迭代期间，**表达式**中的任何声明都将不可见。

## Range-for over a sub-range

Using range-base loops, you can loop over a sub-part of a given container or other range by generating a proxy object that qualifies for range-based for loops.

> 使用范围基循环，可以通过生成符合基于范围的循环条件的代理对象，在给定容器或其他范围的子部分上循环。

```cpp
template<class Iterator, class Sentinel=Iterator>
struct range_t {
  Iterator b;
  Sentinel e;
  Iterator begin() const { return b; }
  Sentinel end() const { return e; }
  bool empty() const { return begin()==end(); }
  range_t without_front( std::size_t count=1 ) const {
    if (std::is_same< std::random_access_iterator_tag, typename std::iterator_traits<Iterator>::iterator_category >{} ) {
      count = (std::min)(std::size_t(std::distance(b,e)), count);
    }
    return {std::next(b, count), e};
  }
  range_t without_back( std::size_t count=1 ) const {
    if (std::is_same< std::random_access_iterator_tag, typename std::iterator_traits<Iterator>::iterator_category >{} ) {
      count = (std::min)(std::size_t(std::distance(b,e)), count);
    }
    return {b, std::prev(e, count)};
  }
};

template<class Iterator, class Sentinel>
range_t<Iterator, Sentinel> range( Iterator b, Sentinal e ) {
  return {b,e};
}
template<class Iterable>
auto range( Iterable& r ) {
  using std::begin; using std::end;
  return range(begin(r),end(r));
}

template<class C>
auto except_first( C& c ) {
  auto r = range(c);
  if (r.empty()) return r;
  return r.without_front();
}

```

now we can do:

```cpp
std::vector<int> v = {1,2,3,4};

for (auto i : except_first(v))
  std::cout << i << '\n';

```

and print out

```cpp
2
3
4

```

Be aware that intermediate objects generated in the `for(:range_expression)` part of the `for` loop will have expired by the time the `for` loop starts.

> 请注意，在“for”循环的“for（：range_expression）”部分中生成的中间对象将在“for”循环开始时过期。

#### Syntax

- while (**condition**) **statement** ;
- do **statement** while (**expression**) ;
- for (**for-init-statement** ; **condition** ; **expression**) **statement** ;
- for (**for-range-declaration** : **for-range-initializer**) **statement** ;
- break ;
- continue ;

#### Remarks

[`algorithm`](http://en.cppreference.com/w/cpp/algorithm) calls are generally preferable to hand-written loops.

> [`algorithm`](http://en.cppreference.com/w/cpp/algorithm)调用通常比手写循环更可取。

If you want something an algorithm already does (or something very similar), the algorithm call is clearer, often more efficient and less error prone.

> 如果你想要一个算法已经做过的事情（或者非常类似的事情），算法调用会更清晰，通常更高效，也不容易出错。

If you need a loop that does something fairly simple (but would require a confusing tangle of binders and adapters if you were using an algorithm), then just write the loop.

> 如果你需要一个做一些相当简单的事情的循环（但如果你使用的是算法，则需要一堆混乱的绑定器和适配器），那么只需编写循环即可。
