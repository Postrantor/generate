---
tip: translate by baidu@2023-10-25 08:29:39
---
---

metaTitle: "C++ | Polymorphism"
description: "Define polymorphic classes, Safe downcasting, Polymorphism & Destructors"
---------------------------------------------------------------------------------------

# Polymorphism

## Define polymorphic classes

The typical example is an abstract shape class, that can then be derived into squares, circles, and other concrete shapes.

> 典型的例子是一个抽象的形状类，然后可以将其派生为正方形、圆形和其他具体形状。

**The parent class:**

Let's start with the polymorphic class:

```cpp
class Shape {
public:
    virtual ~Shape() = default;
    virtual double get_surface() const = 0;
    virtual void describe_object() const { std::cout << "this is a shape" << std::endl; }  

    double get_doubled_surface() const { return 2 * get_surface(); } 
};

```

How to read this definition ?

<li>

You can define polymorphic behavior by introduced member functions with the keyword `virtual`.  Here `get_surface()` and `describe_object()` will obviously be implemented differently for a square than for a circle.  When the function is invoked on an object, function corresponding to the real class of the object will be determined at runtime.

> 您可以通过使用关键字“virtual”引入成员函数来定义多态行为。在这里，“get_surface（）”和“describe_object（）”对于正方形和圆形的实现方式显然不同。当在对象上调用函数时，将在运行时确定与该对象的真实类相对应的函数。

</li>
<li>

It makes no sense to define `get_surface()` for an abstract shape. This is why the function is followed by `= 0`. This means that the function is **pure virtual function**.

> 为抽象形状定义“get_surface（）”是没有意义的。这就是为什么函数后面跟着“=0”的原因。这意味着该函数是**纯虚拟函数**。

</li>
<li>
A polymorphic class should always define a virtual destructor.
</li>
<li>

You may define non virtual member functions.  When these function will be invoked for an object, the function will be chosen depending on the class used at compile-time. Here `get_double_surface()` is defined in this way.

> 您可以定义非虚拟成员函数。当为对象调用这些函数时，将根据编译时使用的类来选择该函数。这里“get_double_surface（）”是这样定义的。

</li>
<li>

A class that contains at least one pure virtual function is an abstract class. Abstract classes cannot be instantiated. You may only have pointers or references of an abstract class type.

> 至少包含一个纯虚拟函数的类是抽象类。抽象类无法实例化。您可能只有抽象类类型的指针或引用。

</li>

**Derived classes**

Once a polymorphic base class is defined you can derive it.  For example:

```cpp
class Square : public Shape {
    Point top_left;
    double side_length;
public: 
    Square (const Point& top_left, double side)
       : top_left(top_left), side_length(side_length) {}

    double get_surface() override { return side_length * side_length; }   
    void describe_object() override { 
        std::cout << "this is a square starting at " << top_left.x << ", " << top_left.y
                  << " with a length of " << side_length << std::endl; 
    }  
};

```

Some explanations:

- You can define or override any of the virtual functions of the parent class.  The fact that a function was virtual in the parent class makes it virtual in the derived class.  No need to tell the compiler the keyword `virtual` again.  But it's recommended to add the keyword `override` at the end of the function declaration, in order to prevent subtle bugs caused by unnoticed variations in the function signature.

> -您可以定义或覆盖父类的任何虚拟函数。函数在父类中是虚拟的，这一事实使它在派生类中成为虚拟的。无需再次告诉编译器关键字“virtual”。但建议在函数声明的末尾添加关键字“override”，以防止函数签名中未被注意到的变化导致的细微错误。

- If all the pure virtual functions of the parent class are defined you can instantiate objects for this class, else it will also become an abstract class.

> -如果定义了父类的所有纯虚拟函数，则可以为该类实例化对象，否则它也将成为一个抽象类。

- You are not obliged to override all the virtual functions.  You can keep the version of the parent if it suits your need.

> -您没有义务覆盖所有虚拟功能。如果适合您的需要，您可以保留父级的版本。

**Example of instantiation**

```cpp
int main() {

    Square square(Point(10.0, 0.0), 6); // we know it's a square, the compiler also
    square.describe_object(); 
    std::cout << "Surface: " << square.get_surface() << std::endl; 

    Circle circle(Point(0.0, 0.0), 5);

    Shape *ps = nullptr;  // we don't know yet the real type of the object
    ps = &circle;         // it's a circle, but it could as well be a square
    ps->describe_object(); 
    std::cout << "Surface: " << ps->get_surface() << std::endl;
}

```

## Safe downcasting

Suppose that you have a pointer to an object of a polymorphic class:

```cpp
Shape *ps;                       // see example on defining a polymorphic class
ps =  get_a_new_random_shape();  // if you don't have such a function yet, you 
                                 // could just write ps = new Square(0.0,0.0, 5);

```

a downcast would be to cast from a general polymorphic `Shape` down to one of its derived and more specific shape like `Square` or `Circle`.

> 下转换是从一般的多态“形状”向下转换为其派生的更具体的形状之一，如“正方形”或“圆形”。

**Why to downcast ?**

Most of the time, you would not need to know which is the real type of the object, as the virtual functions allow you to manipulate your object independently of its type:

> 大多数情况下，您不需要知道哪个是对象的真实类型，因为虚拟函数允许您独立于对象类型来操作对象：

```cpp
std::cout << "Surface: " << ps->get_surface() << std::endl; 

```

If you don't need any downcast, your design would be perfect.

However, you may need sometimes to downcast.  A typical example is when you want to invoke a non virtual function that exist only for the child class.

> 然而，有时你可能需要沮丧。一个典型的例子是，当您想要调用只存在于子类的非虚拟函数时。

Consider for example circles.  Only circles have a diameter. So the class would be defined as :

> 以圆圈为例。只有圆才有直径。因此，类将被定义为：

```cpp
class Circle: public Shape { // for Shape, see example on defining a polymorphic class
    Point center;
    double radius;
public: 
    Circle (const Point& center, double radius)
       : center(center), radius(radius) {}

    double get_surface() const override { return r * r * M_PI; }   

    // this is only for circles. Makes no sense for other shapes 
    double get_diameter() const { return 2 * r; }
};

```

The `get_diameter()` member function only exist for circles. It was not defined for a `Shape` object:

> “get_dame（）”成员函数只存在于圆中。它不是为“Shape”对象定义的：

```cpp
Shape* ps = get_any_shape();
ps->get_diameter(); // OUCH !!! Compilation error 

```

**How to downcast ?**

If you'd know for sure that `ps` points to a circle you could opt for a `static_cast`:

> 如果你确信“ps”指向一个圆圈，你可以选择“static_cast”：

```cpp
std::cout << "Diameter: " << static_cast<Circle*>(ps)->get_diameter() << std::endl;

```

This will do the trick. But it's very risky:  if `ps` appears to by anything else than a `Circle` the behavior of your code will be undefined.

> 这就行了。但这是非常危险的：如果“ps”出现在“Circle”以外的任何东西上，则代码的行为将是未定义的。

So rather than playing Russian roulette, you should safely use a `dynamic_cast`.  This is specifically for polymorphic classes  :

> 因此，与其玩俄罗斯轮盘赌，不如安全地使用“dynamic_cast”。这是专门针对多态类的：

```cpp
int main() {
    Circle circle(Point(0.0, 0.0), 10);
    Shape &shape = circle;

    std::cout << "The shape has a surface of " << shape.get_surface() << std::endl;

    //shape.get_diameter();   // OUCH !!! Compilation error 

    Circle *pc = dynamic_cast<Circle*>(&shape); // will be nullptr if ps wasn't a circle 
    if (pc) 
        std::cout << "The shape is a circle of diameter " << pc->get_diameter() << std::endl;
    else
        std::cout << "The shape isn't a circle !" << std::endl; 
}        

```

Note that `dynamic_cast` is not possible on a class that is not polymorphic.  You'd need at least one virtual function in the class or its parents to be able to use it.

> 请注意，“dynamic_cast”在非多态类上是不可能的。您需要在类或其父类中至少有一个虚拟函数才能使用它。

## Polymorphism & Destructors

If a class is intended to be used polymorphically, with derived instances being stored as base pointers/references, its base class' destructor should be either `virtual` or `protected`.  In the former case, this will cause object destruction to check the `vtable`, automatically calling the correct destructor based on the dynamic type.  In the latter case, destroying the object through a base class pointer/reference is disabled, and the object can only be deleted when explicitly treated as its actual type.

> 如果一个类打算以多态方式使用，并且派生实例被存储为基指针/引用，那么它的基类“析构函数”应该是“虚拟的”或“受保护的”。在前一种情况下，这将导致对象销毁检查“vtable”，根据动态类型自动调用正确的析构函数。在后一种情况下，将禁用通过基类指针/引用销毁对象，并且只有当显式地将对象视为其实际类型时，才能删除该对象。

```cpp
struct VirtualDestructor {
    virtual ~VirtualDestructor() = default;
};

struct VirtualDerived : VirtualDestructor {};

struct ProtectedDestructor {
  protected:
    ~ProtectedDestructor() = default;
};

struct ProtectedDerived : ProtectedDestructor {
    ~ProtectedDerived() = default;
};

// ...

VirtualDestructor* vd = new VirtualDerived;
delete vd; // Looks up VirtualDestructor::~VirtualDestructor() in vtable, sees it's
           // VirtualDerived::~VirtualDerived(), calls that.

ProtectedDestructor* pd = new ProtectedDerived;
delete pd; // Error: ProtectedDestructor::~ProtectedDestructor() is protected.
delete static_cast<ProtectedDerived*>(pd); // Good.

```

Both of these practices guarantee that the derived class' destructor will always be called on derived class instances, preventing memory leaks.

> 这两种做法都保证派生类的析构函数将始终在派生类实例上调用，从而防止内存泄漏。
