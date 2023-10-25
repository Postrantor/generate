---
tip: translate by baidu@2023-10-25 08:24:56
---
---

metaTitle: "C++ | Flow Control"
description: "case, switch, catch, default, if, else, goto, return, throw, try, Conditional Structures: if, if..else, Jump statements : break, continue, goto, exit."
---------------------------------------------------------------------------------------------------------------------------------------------------------------------

# Flow Control

## case

Introduces a case label of a switch statement. The operand must be a constant expression and match the switch condition in type. When the switch statement is executed, it will jump to the case label with operand equal to the condition, if any.

> 引入 switch 语句的大小写标签。操作数必须是常量表达式，并且与类型中的切换条件匹配。执行 switch 语句时，它将跳转到操作数等于条件（如果有的话）的大小写标签。

```cpp
char c = getchar();
bool confirmed;
switch (c) {
  case 'y':
    confirmed = true;
    break;
  case 'n':
    confirmed = false;
    break;
  default:
    std::cout << "invalid response!\n";
    abort();
}

```

## switch

According to the C++ standard,

>

<p>The `switch` statement causes control to be transferred to one of several statements depending on the value
of a condition.</p>

The keyword `switch` is followed by a parenthesized condition and a block, which may contain `case` labels and an optional `default` label. When the switch statement is executed, control will be transferred either to a `case` label with a value matching that of the condition, if any, or to the `default` label, if any.

> 关键字“switch”后面跟着一个带括号的条件和一个块，其中可能包含“case”标签和一个可选的“default”标签。执行 switch 语句时，控制权将转移到值与条件匹配的“case”标签（如果有的话），或转移到“default”标签（如有的话）。

The condition must be an expression or a declaration, which has either integer or enumeration type, or a class type with a conversion function to integer or enumeration type.

> 条件必须是具有整型或枚举类型的表达式或声明，或者是具有向整型或枚举型转换函数的类类型。

```cpp
char c = getchar();
bool confirmed;
switch (c) {
  case 'y':
    confirmed = true;
    break;
  case 'n':
    confirmed = false;
    break;
  default:
    std::cout << "invalid response!\n";
    abort();
}

```

## catch

The `catch` keyword introduces an exception handler, that is, a block into which control will be transferred when an exception of compatible type is thrown. The `catch` keyword is followed by a parenthesized **exception declaration**, which is similar in form to a function parameter declaration: the parameter name may be omitted, and the ellipsis `...` is allowed, which matches any type. The exception handler will only handle the exception if its declaration is compatible with the type of the exception. For more details, see [catching exceptions](http://stackoverflow.com/documentation/c%2B%2B/1354/exceptions/4414/catching-exceptions#t=201608050644277417742).

> “catch”关键字引入了一个异常处理程序，也就是说，当抛出兼容类型的异常时，控制权将转移到一个块中。“catch”关键字后面跟着一个带括号的**异常声明**，其形式类似于函数参数声明：参数名称可以省略，省略号“…”是允许的，它匹配任何类型。只有当异常的声明与异常的类型兼容时，异常处理程序才会处理异常。有关更多详细信息，请参阅[捕获异常]([http://stackoverflow.com/documentation/c%2b%2b/1354/exceptions/4414/catching](http://stackoverflow.com/documentation/c%2B%2B/1354/exceptions/4414/catching)-例外#t=201608050644277417742）。

```cpp
try {
    std::vector<int> v(N);
    // do something
} catch (const std::bad_alloc&) {
    std::cout << "failed to allocate memory for vector!" << std::endl;
} catch (const std::runtime_error& e) {
    std::cout << "runtime error: " << e.what() << std::endl;
} catch (...) {
    std::cout << "unexpected exception!" << std::endl;
    throw;
}

```

## default

In a switch statement, introduces a label that will be jumped to if the condition's value is not equal to any of the case labels' values.

> 在 switch 语句中，引入了一个标签，如果条件的值不等于任何 case 标签的值，则该标签将被跳转到。

```cpp
char c = getchar();
bool confirmed;
switch (c) {
  case 'y':
    confirmed = true;
    break;
  case 'n':
    confirmed = false;
    break;
  default:
    std::cout << "invalid response!\n";
    abort();
}

```

Defines a default constructor, copy constructor, move constructor, destructor, copy assignment operator, or move assignment operator to have its default behaviour.

> 将默认构造函数、复制构造函数、移动构造函数、析构函数、复制赋值运算符或移动赋值运算符定义为具有其默认行为。

```cpp
class Base {
    // ...
    // we want to be able to delete derived classes through Base*,
    // but have the usual behaviour for Base's destructor.
    virtual ~Base() = default;
};

```

## if

Introduces an if statement. The keyword `if` must be followed by a parenthesized condition, which can be either an expression or a declaration. If the condition is truthy, the substatement after the condition will be executed.

> 引入 if 语句。关键字“if”后面必须跟一个带括号的条件，该条件可以是表达式，也可以是声明。如果条件是真的，则将执行该条件之后的子语句。

```cpp
int x;
std::cout << "Please enter a positive number." << std::endl;
std::cin >> x;
if (x <= 0) {
    std::cout << "You didn't enter a positive number!" << std::endl;
    abort();
}

```

## else

The first substatement of an if statement may be followed by the keyword `else`. The substatement after the `else` keyword will be executed when the condition is falsey (that is, when the first substatement is not executed).

> if 语句的第一个子语句后面可以跟关键字“else”。当条件为 false 时（即，当不执行第一个子语句时），将执行'else'关键字之后的子语句。

```cpp
int x;
std::cin >> x;
if (x%2 == 0) {
    std::cout << "The number is even\n";
} else {
    std::cout << "The number is odd\n";
}

```

## goto

Jumps to a labelled statement, which must be located in the current function.

> 跳转到一个带标签的语句，该语句必须位于当前函数中。

```cpp
bool f(int arg) {
    bool result = false;
    hWidget widget = get_widget(arg);
    if (!g()) {
        // we can't continue, but must do cleanup still
        goto end;
    }
    // ...
    result = true;
  end:
    release_widget(widget);
    return result;
}

```

## return

Returns control from a function to its caller.

If `return` has an operand, the operand is converted to the function's return type, and the converted value is returned to the caller.

> 如果“return”有操作数，则操作数将转换为函数的返回类型，转换后的值将返回给调用方。

```cpp
int f() {
    return 42;
}
int x = f(); // x is 42
int g() {
    return 3.14;
}
int y = g(); // y is 3

```

If `return` does not have an operand, the function must have `void` return type. As a special case, a `void`-returning function can also return an expression if the expression has type `void`.

> 如果“return”没有操作数，则函数必须具有“void”返回类型。作为一种特殊情况，如果表达式的类型为“void”，则返回“void”的函数也可以返回表达式。

```cpp
void f(int x) {
    if (x < 0) return;
    std::cout << sqrt(x);
}
int g() { return 42; }
void h() {
    return f(); // calls f, then returns
    return g(); // ill-formed
}

```

When `main` returns, `std::exit` is implicitly called with the return value, and the value is thus returned to the execution environment. (However, returning from `main`  destroys automatic local variables, while calling `std::exit` directly does not.)

> 当“main”返回时，将使用返回值隐式调用“std:：exit”，从而将该值返回到执行环境。（但是，从“main”返回会破坏自动局部变量，而直接调用“std:：exit”则不会。）

```cpp
int main(int argc, char** argv) {
    if (argc < 2) {
        std::cout << "Missing argument\n";
        return EXIT_FAILURE; // equivalent to: exit(EXIT_FAILURE);
    }
}

```

## throw

<li>

When `throw` occurs in an expression with an operand, its effect is to throw an [exception](http://stackoverflow.com/documentation/c%2B%2B/1354/exceptions), which is a copy of the operand.

> 当“throw”出现在带有操作数的表达式中时，其效果是引发[异常](http://stackoverflow.com/documentation/c%2B%2B/1354/exceptions)，它是操作数的副本。

```cpp
void print_asterisks(int count) {
    if (count < 0) {
        throw std::invalid_argument("count cannot be negative!");
    }
    while (count--) { putchar('*'); }
}

```

</li>
<li>

When `throw` occurs in an expression without an operand, its effect is to [rethrow the current exception](http://stackoverflow.com/documentation/c%2B%2B/1354/exceptions/5574/rethrow-propagate-exception). If there is no current exception, `std::terminate` is called.

> 当“throw”出现在没有操作数的表达式中时，其效果是[重新引发当前异常]([http://stackoverflow.com/documentation/c%2b%2b/1354/exceptions/5574/rethrow](http://stackoverflow.com/documentation/c%2B%2B/1354/exceptions/5574/rethrow)-传播异常）。如果当前没有异常，则调用“std:：terminate”。

```cpp
try {
    // something risky
} catch (const std::bad_alloc&) {
    std::cerr << "out of memory" << std::endl;
} catch (...) {
    std::cerr << "unexpected exception" << std::endl;
    // hope the caller knows how to handle this exception
    throw;
}

```

</li>
<li>

When `throw` occurs in a function declarator, it introduces a dynamic exception specification, which lists the types of exceptions that the function is allowed to propagate.

> 当函数声明符中出现“throw”时，它引入了一个动态异常规范，该规范列出了允许函数传播的异常类型。

```cpp
// this function might propagate a std::runtime_error,
// but not, say, a std::logic_error
void risky() throw(std::runtime_error);
// this function can't propagate any exceptions
void safe() throw();

```

Dynamic exception specifications are deprecated as of C++11.

</li>

Note that the first two uses of `throw` listed above constitute expressions rather than statements. (The type of a throw expression is `void`.) This makes it possible to nest them within expressions, like so:

> 请注意，上面列出的“throw”的前两个用法构成表达式，而不是语句。（throw 表达式的类型是“void”。）这样就可以将它们嵌套在表达式中，如下所示：

```cpp
unsigned int predecessor(unsigned int x) {
    return (x > 0) ? (x - 1) : (throw std::invalid_argument("0 has no predecessor"));
}

```

## try

The keyword `try` is followed by a block, or by a constructor initializer list and then a block (see [here](http://stackoverflow.com/documentation/c%2B%2B/1354/exceptions/6692/function-try-blocks-in-constructor)). The try block is followed by one or more [catch blocks](http://stackoverflow.com/documentation/c%2B%2B/4891/keywords/18492/catch). If an [exception](http://stackoverflow.com/documentation/c%2B%2B/1354/exceptions) propagates out of the try block, each of the corresponding catch blocks after the try block has the opportunity to handle the exception, if the types match.

> 关键字“try”后面跟着一个块，或者是构造函数初始值设定项列表，然后是一个块（参见[此处]([http://stackoverflow.com/documentation/c%2b%2b/1354/exceptions/6692/function](http://stackoverflow.com/documentation/c%2B%2B/1354/exceptions/6692/function)-在构造函数中尝试块）。try 块后面跟着一个或多个 [catch 块](http://stackoverflow.com/documentation/c%2B%2B/4891/keywords/18492/catch)。如果出现[异常](http://stackoverflow.com/documentation/c%2B%2B/1354/exceptions)传播到 try 块之外，如果类型匹配，则 try 块之后的每个相应 catch 块都有机会处理异常。

```cpp
std::vector<int> v(N);     // if an exception is thrown here,
                           // it will not be caught by the following catch block
try {
    std::vector<int> v(N); // if an exception is thrown here,
                           // it will be caught by the following catch block
    // do something with v
} catch (const std::bad_alloc&) {
    // handle bad_alloc exceptions from the try block
}    

```

## Conditional Structures: if, if..else

**if and else:**

it used to check whether the given expression returns true or false and acts as such:

> 它用于检查给定的表达式是否返回 true 或 false，并按如下方式操作：

```cpp
if (condition) statement

```

the condition can be any valid C++ expression that returns something that be checked against truth/falsehood for example:

> 条件可以是任何有效的 C++ 表达式，其返回要对照真/假进行检查的东西，例如：

```cpp
if (true) { /* code here */ }  // evaluate that true is true and execute the code in the brackets
if (false) { /* code here */ } // always skip the code since false is always false

```

the condition can be anything, a function, a variable, or a comparison for example

> 条件可以是任何东西，例如函数、变量或比较

```cpp
if(istrue()) { } // evaluate the function, if it returns true, the if will execute the code
if(isTrue(var)) { } //evalute the return of the function after passing the argument var
if(a == b) { } // this will evaluate the return of the experssion (a==b) which will be true if equal and false if unequal
if(a) { } //if a is a boolean type, it will evaluate for its value, if it's an integer, any non zero value will be true, 

```

if we want to check for a multiple expressions we can do it in two ways :

**using binary operators** :

```cpp
if (a && b) { } // will be true only if both a and b are true (binary operators are outside the scope here
if (a || b ) { } //true if a or b is true 

```

**using if/ifelse/else**:

for a simple switch either if or else

```cpp
if (a== "test") {
    //will execute if a is a string "test" 
} else {
    // only if the first failed, will execute 
}

```

for multiple choices :

```cpp
if (a=='a') { 
// if a is a char valued 'a'  
} else if (a=='b') {
// if a is a char valued 'b' 
} else if (a=='c') {
// if a is a char valued 'c'
} else { 
//if a is none of the above
}

```

however it must be noted that you should use '**switch**' instead if your code checks for the same variable's value

> 但是，必须注意的是，如果您的代码检查相同变量的值，则应该使用“**开关**”

## Jump statements : break, continue, goto, exit.

**The break instruction:**

Using break we can leave a loop even if the condition for its end is not fulfilled. It can be used to end an infinite loop, or to force it to end before its natural end

> 使用 break，我们可以留下一个循环，即使它的结束条件没有满足。它可以用来结束一个无限循环，也可以强迫它在自然结束之前结束

The syntax is

```cpp
break;

```

**Example**:

we often use `break` in `switch` cases,ie once a case i switch is satisfied then the code block of that condition is executed .

> 我们经常在“switch”的情况下使用“break”，即一旦满足情况 i switch，则执行该条件的代码块。

```cpp
switch(conditon){
case 1: block1;
case 2: block2;
case 3: block3;
default: blockdefault;
} 

```

in this case if case 1 is satisfied then block 1 is executed , what we really want is only the block1 to be processed but instead once the block1 is processed remaining blocks,block2,block3 and blockdefault are also processed even though only case 1 was satified.To avoid this we use break at the end of each block like :

> 在这种情况下，如果满足情况 1，则执行块 1，我们真正想要的只是要处理的块 1，但相反，一旦处理了块 1，则也处理剩余的块、块 2、块 3 和块默认，即使仅满足情况 1。为了避免这种情况，我们在每个块的末尾使用 break，如下所示：

```cpp
switch(condition){
case 1: block1;
        break;
case 2: block2;
        break;
case 3: block3;
        break;
default: blockdefault;
        break;
}

```

so  only one block is processed and the control moves out of the switch loop.

> 因此只处理一个块，并且控制移出切换循环。

break can also be used in other conditional and non conditional loops like `if`,`while`,`for` etc;

> break 也可以用于其他条件和非条件循环，如“if”、“while”、“for”等；

example:

```cpp
if(condition1){
   ....
   if(condition2){
    .......
    break;
    }
 ...
}

```

**The continue instruction:**

The continue instruction causes the program to skip the rest of the loop in the present iteration as if the end of the statement block would have been reached, causing it to jump to the following iteration.

> continue 指令使程序跳过当前迭代中循环的其余部分，就好像已经到达语句块的末尾一样，从而跳到下一次迭代。

The syntax is

```cpp
continue;

```

**Example**
consider the following :

```cpp
for(int i=0;i<10;i++){
if(i%2==0)
continue;
cout<<"\n @"<<i;
}

```

which produces the output:

```

@1
 @3
 @5
 @7
 @9

```

i this code whenever the condition `i%2==0` is satisfied `continue` is processed,this causes the compiler to skip all the remaining code( printing @ and i) and increment/decrement statement of the loop gets executed.

> i 每当满足条件“i%2==0”“continue”时处理此代码，这将导致编译器跳过所有剩余代码（打印 @ 和 i），并执行循环的递增/递减语句。

[<img src="https://i.stack.imgur.com/Fbuep.jpg" alt="in for loop" />](https://i.stack.imgur.com/Fbuep.jpg)

> [<img src=“https://i.stack.imgur.com/Fbuep.jpg“alt=”in for loop“/>](https://i.stack.imgur.com/Fbuep.jpg)

**The goto instruction:**

It allows making an absolute jump to another point in the program. You should use this feature carefully since its execution ignores any type of nesting limitation.

> 它允许绝对跳转到程序中的另一个点。您应该小心使用此功能，因为它的执行会忽略任何类型的嵌套限制。

The destination point is identified by a label, which is then used as an argument for the goto instruction. A label is made of a valid identifier followed by a colon (:)

> 目标点由一个标签标识，然后将其用作 goto 指令的参数。标签由一个有效标识符和一个冒号（：）组成

The syntax is

```cpp
goto label;
..
.
label: statement;

```

**Note:** **Use of goto statement is highly discouraged because it makes difficult to trace the control flow of a program, making the program hard to understand and hard to modify.**

> **注意：***强烈反对使用 goto 语句，因为它会使跟踪程序的控制流变得困难，使程序难以理解和修改**

[<img src="https://i.stack.imgur.com/Li8ZR.jpg" alt="types of goto" />](https://i.stack.imgur.com/Li8ZR.jpg)

> [<img src=“https://i.stack.imgur.com/Li8ZR.jpg“alt=”转到的类型“/>](https://i.stack.imgur.com/Li8ZR.jpg)

**Example :**

```cpp
int num = 1;
STEP:
do{
 
    if( num%2==0 )
    {
        num = num + 1;
        goto STEP;
     }

   cout << "value of num : " << num << endl;
   num = num + 1;
 }while( num < 10 );

```

output :

```cpp
value of num : 1
value of num : 3
value of num : 5
value of num : 7
value of num : 9

```

whenever the condition `num%2==0` is satisfied the goto sends the execution control to the beginning of the `do-while` loop.

> 只要满足条件“num%2==0”，goto 就将执行控制发送到“do-while”循环的开头。

**The exit function:**

`exit` is a function defined in `cstdlib`.

The purpose of `exit` is to terminate the running program with an specific exit code. Its prototype is:

> “退出”的目的是用特定的退出代码终止正在运行的程序。其原型是：

```cpp
void exit (int exit code);

```

`cstdlib` defines the standard exit codes `EXIT_SUCCESS` and `EXIT_FAILURE`.

#### Remarks

Check out the [loops topic](https://stackoverflow.com/documentation/c%2B%2B/589/loops) for the different kind of loops.

> 查看[循环主题](https://stackoverflow.com/documentation/c%2B%2B/589/loops)用于不同类型的循环。
