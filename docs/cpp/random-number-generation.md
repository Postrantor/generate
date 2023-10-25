---
tip: translate by baidu@2023-10-25 08:30:16
---
---

metaTitle: "C++ | Random number generation"
description: "True random value generator, Generating a pseudo-random number, Using the generator for multiple distributions"
-----------------------------------------------------------------------------------------------------------------------------

# Random number generation

## True random value generator

To generate true random values that can be used for cryptography `std::random_device` has to be used as generator.

> 要生成可用于加密的真正随机值，必须使用“std:：random_device”作为生成器。

```cpp
#include <iostream>
#include <random>

int main()
{
   std::random_device crypto_random_generator;
   std::uniform_int_distribution<int> int_distribution(0,9);
   
   int actual_distribution[10] = {0,0,0,0,0,0,0,0,0,0};
   
   for(int i = 0; i < 10000; i++) {
       int result = int_distribution(crypto_random_generator);
       actual_distribution[result]++;
   }

   for(int i = 0; i < 10; i++) {
       std::cout << actual_distribution[i] << " ";
   }
   
   return 0;
}

```

`std::random_device` is used in the same way as a pseudo random value generator is used.

However `std::random_device` **may be implemented in terms of an implementation-defined pseudo-random number engine** if a non-deterministic source (e.g. a hardware device) isn't available to the implementation.

> 然而，如果非确定性源（例如硬件设备）不可用于实现，则可以根据实现定义的伪随机数引擎**来实现“std:：random_device”**。

Detecting such implementations should be possible via the [`entropy` member function](http://en.cppreference.com/w/cpp/numeric/random/random_device/entropy) (which return zero when the generator is completely deterministic), but many popular libraries (both GCC's libstdc++ and LLVM's libc++) always return zero, even when they're using high-quality external randomness.

> 应该可以通过[“熵”成员函数]检测这种实现([http://en.cppreference.com/w/cpp/numeric/random/random_device/entropy](http://en.cppreference.com/w/cpp/numeric/random/random_device/entropy))（当生成器完全具有确定性时返回零），但许多流行的库（GCC 的 libstdc++ 和 LLVM 的 libc++）总是返回零，即使它们使用高质量的外部随机性。

## Generating a pseudo-random number

A pseudo-random number generator generates values that can be guessed based on previously generated values. In other words: it is deterministic. Do not use a pseudo-random number generator in situations where a true random number is required.

> 伪随机数生成器生成可以基于先前生成的值进行猜测的值。换句话说：它是确定性的。在需要真实随机数的情况下，不要使用伪随机数生成器。

```cpp
#include <iostream>
#include <random>

int main()
{
   std::default_random_engine pseudo_random_generator;
   std::uniform_int_distribution<int> int_distribution(0, 9);
   
   int actual_distribution[10] = {0,0,0,0,0,0,0,0,0,0};
   
   for(int i = 0; i < 10000; i++) {
       int result = int_distribution(pseudo_random_generator);
       actual_distribution[result]++;
   }

   for(int i = 0; i <= 9; i++) {
       std::cout << actual_distribution[i] << " ";
   }
   
   return 0;
}

```

This code creates a random number generator, and a distribution that generates integers in the range [0,9] with equal likelihood. It then counts how many times each result was generated.

> 该代码创建了一个随机数生成器，以及一个生成具有相等似然性的[0,9]范围内整数的分布。然后计算每个结果生成的次数。

The template parameter of [`std::uniform_int_distribution<T>`](http://en.cppreference.com/w/cpp/numeric/random/uniform_int_distribution) specifies the type of integer that should be generated. Use [`std::uniform_real_distribution<T>`](http://en.cppreference.com/w/cpp/numeric/random/uniform_real_distribution) to generate floats or doubles.

> [`std:：uniform_int_distribution<T>`]的模板参数([http://en.cppreference.com/w/cpp/numeric/random/uniform_int_distribution](http://en.cppreference.com/w/cpp/numeric/random/uniform_int_distribution))指定应生成的整数的类型。使用[`std:：uniform_real_distribution<T>`](http://en.cppreference.com/w/cpp/numeric/random/uniform_real_distribution)以生成浮点或双精度。

## Using the generator for multiple distributions

The random number generator can (and should) be used for multiple distributions.

> 随机数生成器可以（也应该）用于多个分布。

```cpp
#include <iostream>
#include <random>

int main()
{
   std::default_random_engine pseudo_random_generator;
   std::uniform_int_distribution<int> int_distribution(0, 9);
   std::uniform_real_distribution<float> float_distribution(0.0, 1.0);
   std::discrete_distribution<int> rigged_dice({1,1,1,1,1,100});
   
   std::cout << int_distribution(pseudo_random_generator) << std::endl;
   std::cout << float_distribution(pseudo_random_generator) << std::endl;
   std::cout << (rigged_dice(pseudo_random_generator) + 1) << std::endl;
   
   return 0;
}

```

In this example, only one generator is defined. It is subsequently used to generate a random value in three different distributions. The `rigged_dice` distribution will generate a value between 0 and 5, but almost always generates a `5`, because the chance to generate a `5` is `100 / 105`.

> 在本例中，只定义了一个生成器。随后，它被用于生成三种不同分布中的随机值。“rigged_dice”分布将生成一个 0 到 5 之间的值，但几乎总是生成一个“5”，因为生成“5”的机会是“100/105”。

#### Remarks

Random number generation in C++ is provided by the `<random>` header. This header defines random devices, pseudo-random generators and distributions.

> C++ 中的随机数生成是由“＜Random＞”标头提供的。该报头定义了随机设备、伪随机生成器和分布。

Random devices return random numbers provided by operating system. They should either be used for initialization of pseudo-random generators or directly for cryptographic purposes.

> 随机设备返回操作系统提供的随机数。它们应该用于伪随机生成器的初始化，或者直接用于加密目的。

Pseudo-random generators return integer pseudo-random numbers based on their initial seed. The pseudo-random number range typically spans all values of an unsigned type. All pseudo-random generators in the standard library will return the same numbers for the same initial seed for all platforms.

> 伪随机生成器根据其初始种子返回整数伪随机数。伪随机数范围通常跨越无符号类型的所有值。标准库中的所有伪随机生成器将为所有平台的相同初始种子返回相同的数字。

Distributions consume random numbers from pseudo-random generators or random devices and produce random numbers with necessary distribution. Distributions are not platform-independent and can produce different numbers for the same generators with the same initial seeds on different platforms.

> 分布消耗来自伪随机生成器或随机设备的随机数，并产生具有必要分布的随机数。分布与平台无关，并且可以在不同平台上为具有相同初始种子的相同生成器生成不同的数字。
