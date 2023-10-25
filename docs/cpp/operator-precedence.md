---
tip: translate by baidu@2023-10-25 08:28:54
---
---
metaTitle: "C++ | operator precedence"
description: "Logical && and || operators: short-circuit, Arithmetic operators, Logical AND and OR operators, Unary Operators"
---

# operator precedence



## Logical && and || operators: short-circuit



&& has precedence over ||, this means that parentheses are placed to evaluate what would be evaluated together.

> &&优先于||，这意味着括号用于计算将一起计算的内容。


c++ uses short-circuit evaluation in && and || to not do unnecessary executions.<br />

> c++在&&和||中使用短路求值，以避免执行不必要的操作<br/>

If the left hand side of || returns true the right hand side does not need to be evaluated anymore.

> 如果||的左手边返回true，则右手边不需要再进行求值。

```cpp
#include <iostream>
#include <string>

using namespace std;

bool True(string id){
    cout << "True" << id << endl;
    return true;
}

bool False(string id){
    cout << "False" << id << endl;
    return false;
}


int main(){
    bool result;
    //let's evaluate 3 booleans with || and && to illustrate operator precedence
    //precedence does not mean that && will be evaluated first but rather where    
    //parentheses would be added
    //example 1
    result =
        False("A") || False("B") && False("C"); 
                // eq. False("A") || (False("B") && False("C"))
    //FalseA
    //FalseB
    //"Short-circuit evaluation skip of C"
    //A is false so we have to evaluate the right of ||,
    //B being false we do not have to evaluate C to know that the result is false
    

    
    result =
        True("A") || False("B") && False("C"); 
                // eq. True("A") || (False("B") && False("C"))
    cout << result << " :=====================" << endl;
    //TrueA
    //"Short-circuit evaluation skip of B"
    //"Short-circuit evaluation skip of C"
    //A is true so we do not have to evaluate 
    //        the right of || to know that the result is true
    //If || had precedence over && the equivalent evaluation would be:
    // (True("A") || False("B")) && False("C")
    //What would print
    //TrueA
    //"Short-circuit evaluation skip of B"
    //FalseC
    //Because the parentheses are placed differently 
    //the parts that get evaluated are differently
    //which makes that the end result in this case would be False because C is false
}

```



## Arithmetic operators



Arithmetic operators in C++ have the same precedence as they do in mathematics:

> C++中的算术运算符具有与数学中相同的优先级：


Multiplication and division have left associativity(meaning that they will be evaluated from left to right) and they have higher precedence than addition and subtraction, which also have left associativity.

> 乘法和除法具有左结合性（意味着它们将从左到右进行计算），并且它们的优先级高于加法和减法，后者也具有左关联性。


We can also force the precedence of expression using parentheses `(` `)`. Just the same way as you would do that in normal mathematics.

> 我们还可以使用括号“（``）”强制表达式的优先级。就像你在普通数学中那样。

```cpp
// volume of a spherical shell = 4 pi R^3 - 4 pi r^3
double vol = 4.0*pi*R*R*R/3.0 - 4.0*pi*r*r*r/3.0;

//Addition:

int a = 2+4/2;          // equal to: 2+(4/2)         result: 4
int b = (3+3)/2;        // equal to: (3+3)/2         result: 3

//With Multiplication

int c = 3+4/2*6;        // equal to: 3+((4/2)*6)     result: 15
int d = 3*(3+6)/9;      // equal to: (3*(3+6))/9     result: 3

//Division and Modulo

int g = 3-3%1;          // equal to: 3 % 1 = 0  3 - 0 = 3
int h = 3-(3%1);        // equal to: 3 % 1 = 0  3 - 0 = 3
int i = 3-3/1%3;        // equal to: 3 / 1 = 3  3 % 3 = 0  3 - 0 = 3
int l = 3-(3/1)%3;      // equal to: 3 / 1 = 3  3 % 3 = 0  3 - 0 = 3
int m = 3-(3/(1%3));    // equal to: 1 % 3 = 1  3 / 1 = 3  3 - 3 = 0

```



## Logical AND and OR operators


These operators have the usual precedence in C++: AND before OR.

```cpp
// You can drive with a foreign license for up to 60 days
bool can_drive = has_domestic_license || has_foreign_license && num_days <= 60;

```

This code is equivalent to the following:

```cpp
// You can drive with a foreign license for up to 60 days
bool can_drive = has_domestic_license || (has_foreign_license && num_days <= 60);

```


Adding the parenthesis does not change the behavior, though, it does make it easier to read. By adding these parentheses, no confusion exist about the intent of the writer.

> 添加括号并不会改变行为，但它确实使阅读变得更容易。加上这些括号，就不会混淆作者的意图。



## Unary Operators



Unary operators act on the object upon which they are called and have high precedence. (See Remarks)

> 一元运算符作用于调用它们的对象，并且具有很高的优先级。（见备注）


When used postfix, the action occurs only after the entire operation is evaluated, leading to some interesting arithmetics:

> 当使用后缀时，只有在评估了整个操作之后才会执行操作，从而产生了一些有趣的算法：

```cpp
int a = 1;
++a;            // result: 2
a--;            // result: 1
int minusa=-a;  // result: -1

bool b = true;
!b; // result: true

a=4;
int c = a++/2;      // equal to: (a==4) 4 / 2   result: 2 ('a' incremented postfix)
cout << a << endl;  // prints 5!
int d = ++a/2;      // equal to: (a+1) == 6 / 2 result: 3

int arr[4] =  {1,2,3,4};

int *ptr1 = &arr[0];    // points to arr[0] which is 1
int *ptr2 = ptr1++;     // ptr2 points to arr[0] which is still 1; ptr1 incremented
std::cout << *ptr1++ << std::endl;  // prints  2

int e = arr[0]++;       // receives the value of arr[0] before it is incremented
std::cout << e << std::endl;      // prints 1
std::cout << *ptr2 << std::endl;  // prints arr[0] which is now 2

```



#### Remarks



Operators are listed top to bottom, in descending precedence. Operators with the same number have equal precedence and the same associativity.

> 运算符从上到下按降序排列。具有相同编号的运算符具有相同的优先级和相同的关联性。

1. `::`

1. The postfix operators: `[]` `()` `T(...)` `.` `->` `++` `--`  `dynamic_cast` `static_cast` `reinterpret_cast` `const_cast` `typeid`

> 1.后缀运算符：`[]``（）`T（…）``.`->`++`--``dynamic_cast `static_cast `interpret_cast `const_cast `typeid`

1. The unary prefix operators: `++` `--` `*` `&` `+` `-` `!` `~` `sizeof` `new` `delete` `delete[]`; the C-style cast notation, `(T)...`;  (C++11 and above) `sizeof...` `alignof` `noexcept`

> 1.一元前缀运算符：`++`--`*`&`+`-``！`~``sizeof `new`delete `delete[]`；C样式的强制转换表示法，“（T）…”；（C++11及以上版本）`sizeof…``alignof`noexcept`
1. `.*` and `->*`
1. `*`, `/`, and `%`, binary arithmetic operators
1. `+` and `-`, binary arithmetic operators
1. `<<` and `>>`
1. `<`, `>`, `<=`, `>=`
1. `==` and `!=`
1. `&`, the bitwise AND operator
1. `^`
1. `|`
1. `&&`
1. `||`
1. `?:` (ternary conditional operator)
1. `=`, `*=`, `/=`, `%=`, `+=`, `-=`, `>>=`, `<<=`, `&=`, `^=`, `|=`
1. `throw`
1. `,` (the comma operator)


The assignment, compound assignment, and ternary conditional operators are right-associative. All other binary operators are left-associative.

> 赋值、复合赋值和三元条件运算符是右关联的。所有其他二进制运算符都是关联的。


The rules for the ternary conditional operator are a bit more complicated than simple precedence rules can express.

> 三元条件运算符的规则比简单的优先级规则所能表达的要复杂一些。


- An operand binds less tightly to a `?` on its left or a `:` on its right than to any other operator. Effectively, the second operand of the conditional operator is parsed as though it is parenthesized. This allows an expression such as `a ? b , c : d` to be syntactically valid.

> -操作数与“？”的绑定不那么紧密在其左边或在其右边的“：”。实际上，条件运算符的第二个操作数被解析为带括号。这允许诸如“a？b，c:d`在语法上是有效的。

- An operand binds more tightly to a `?` on its right than to an assignment operator or `throw` on its left, so `a = b ? c : d` is equivalent to `a = (b ? c : d)` and `throw a ? b : c` is equivalent to `throw (a ? b : c)`.

> -操作数与“？”绑定得更紧密在它的右边，而不是在它的左边的赋值运算符或“throw”，所以“a=b？c:d’等价于a=（b？c:d）’和抛出a？b:c相当于throw（a？b:c）。

- An operand binds more tightly to an assignment operator on its right than to `:` on its left, so `a ? b : c = d` is equivalent to `a ? b : (c = d)`.

> -操作数与右边的赋值运算符的绑定比与左边的“：”的绑定更紧密，所以“a？b:c=d`等同于`a？b:（c=d）`。

