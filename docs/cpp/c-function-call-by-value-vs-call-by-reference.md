---
tip: translate by baidu@2023-10-25 08:21:46
---
---
metaTitle: "C++ | C++ function call by value vs. call by reference"
description: "Call by value"
---

# C++ function "call by value" vs. "call by reference"



The scope of this section is to explain the differences in theory and implementation for what happens with the parameters of a function upon calling.

> 本节的范围是解释函数参数在调用时在理论和实现上的差异。


In detail the parameters can be seen as variables before the function call and inside the function, where the visible behaviour and accessibility to these variables differs with the method used to hand them over.

> 详细地说，参数可以在函数调用之前和函数内部被视为变量，其中这些变量的可见行为和可访问性与用于移交它们的方法不同。


Additionally, the reusability of variables and their respective values after the function call also is explained by this topic.

> 此外，本主题还解释了函数调用后变量及其各自值的可重用性。



## Call by value



Upon calling a function there are new elements created on the program stack. These include some information about the function and also space (memory locations) for the parameters and the return value.

> 在调用一个函数时，程序堆栈上会创建一些新元素。其中包括一些关于函数的信息，以及参数和返回值的空间（内存位置）。


When handing over a parameter to a function the value of the used variable (or literal) is copied into the memory location of the function parameter. This implies that now there a two memory locations with the same value. Inside of the function we only work on the parameter memory location.

> 将参数移交给函数时，所用变量（或文字）的值会复制到函数参数的内存位置。这意味着现在有两个具有相同值的内存位置。在函数内部，我们只处理参数内存位置。


After leaving the function the memory on the program stack is popped (removed) which erases all data of the function call, including the memory location of the parameters we used inside. Thus, the values changed inside the function do not affect the outside variables values.

> 离开函数后，程序堆栈上的内存会弹出（删除），从而擦除函数调用的所有数据，包括我们在内部使用的参数的内存位置。因此，函数内部更改的值不会影响外部变量的值。

```cpp
int func(int f, int b) { 
  //new variables are created and values from the outside copied
  //f has a value of 0
  //inner_b has a value of 1
  f = 1;
  //f has a value of 1
  b = 2;
  //inner_b has a value of 2
  return f+b;
}

int main(void) {
  int a = 0;
  int b = 1; //outer_b
  int c;

  c = func(a,b);
  //the return value is copied to c
  
  //a has a value of 0
  //outer_b has a value of 1   <--- outer_b and inner_b are different variables
  //c has a value of 3
}

```


In this code we create variables inside the main function. These get assigned values. Upon calling the functions there are two new variables created: `f` and `inner_b` where `b` shares the name with the outer variable it does not share the memory location. The behaviour of `a<->f` and `b<->b` is identical.

> 在这段代码中，我们在主函数中创建变量。这些得到了赋值。在调用函数时，会创建两个新变量：“f”和“inner_b”，其中“b”与外部变量共享名称，而外部变量不共享内存位置。“a<->f”和“b<->b”的行为是相同的。


The following graphic symbolizes what is happening on the stack and why there is no change in varibale `b`. The graphic is not fully accurate but emphazises the example.

> 下图表示堆栈上发生的情况以及变量“b”没有变化的原因。这张图并不完全准确，但强调了这个例子。

[<img src="https://i.stack.imgur.com/TgZM0.png" alt="Visualization of Stack during function call" />](https://i.stack.imgur.com/TgZM0.png)

> [<img src=“https://i.stack.imgur.com/TgZM0.png“alt=”函数调用期间堆栈的可视化“/>](https://i.stack.imgur.com/TgZM0.png)


It is called "call by value" because we do not hand over the variables but only the values of these variables.

> 它被称为“按值调用”，因为我们不移交变量，只移交这些变量的值。

