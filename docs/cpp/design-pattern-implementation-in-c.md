---
tip: translate by baidu@2023-10-25 08:23:49
---
---
metaTitle: "C++ | Design pattern implementation in C++"
description: "Adapter Pattern, Observer pattern, Factory Pattern, Builder Pattern with Fluent API"
---

# Design pattern implementation in C++



On this page, you can find examples of how design patterns are implemented in C++. For the details on these patterns, you can check out [the design patterns documentation](http://stackoverflow.com/documentation/design-patterns/topics).

> 在这个页面上，您可以找到如何在C++中实现设计模式的示例。有关这些模式的详细信息，您可以查看[设计模式文档](http://stackoverflow.com/documentation/design-patterns/topics)。



## Adapter Pattern



Convert the interface of a class into another interface clients expect. Adapter (or Wrapper) lets classes work together that couldn't otherwise because of incompatible interfaces. Adapter pattern's motivation is that we can reuse existing software if we can modify the interface.

> 将类的接口转换为客户端期望的另一个接口。Adapter（或Wrapper）允许类协同工作，否则由于接口不兼容而无法协同工作。适配器模式的动机是，如果我们可以修改接口，我们就可以重用现有的软件。

<li>
Adapter pattern relies on object composition.
</li>
<li>
Client calls operation on Adapter object.
</li>
<li>
Adapter calls Adaptee to carry out the operation.
</li>
<li>
<p>In STL, stack adapted from vector:

When stack executes push(), underlying vector does vector::push_back().</p>

> 当堆栈执行push（）时，底层向量执行vector:：push_back（）</p>
</li>

**Example:**

```cpp
#include <iostream>

// Desired interface (Target)
class Rectangle 
{
  public:
    virtual void draw() = 0;
};

// Legacy component (Adaptee)
class LegacyRectangle 
{
  public:
    LegacyRectangle(int x1, int y1, int x2, int y2) {
        x1_ = x1;
        y1_ = y1;
        x2_ = x2;
        y2_ = y2;
        std::cout << "LegacyRectangle(x1,y1,x2,y2)\n";
    }
    void oldDraw() {
        std::cout << "LegacyRectangle:  oldDraw(). \n";
    }
  private:
    int x1_;
    int y1_;
    int x2_;
    int y2_;
};

// Adapter wrapper
class RectangleAdapter: public Rectangle, private LegacyRectangle 
{
  public:
    RectangleAdapter(int x, int y, int w, int h):
      LegacyRectangle(x, y, x + w, y + h) {
         std::cout << "RectangleAdapter(x,y,x+w,x+h)\n";
      }

    void draw() {
        std::cout << "RectangleAdapter: draw().\n"; 
        oldDraw();
    }
};

int main()
{
  int x = 20, y = 50, w = 300, h = 200;
  Rectangle *r = new RectangleAdapter(x,y,w,h);
  r->draw();
}

//Output:
//LegacyRectangle(x1,y1,x2,y2)
//RectangleAdapter(x,y,x+w,x+h)

```

**Summary of the code:**

<li>
The client thinks he is talking to a `Rectangle`
</li>
<li>

The target is the `Rectangle` class. This is what the client invokes method on.

> 目标是“Rectangle”类。这就是客户端调用的方法。

```cpp
 Rectangle *r = new RectangleAdapter(x,y,w,h);
 r->draw();

```


</li>
<li>
Note that the adapter class uses multiple inheritance.

```cpp
 class RectangleAdapter: public Rectangle, private LegacyRectangle {
     ...
 }

```


</li>
<li>

The Adapter `RectangleAdapter` lets the `LegacyRectangle` responds to request (`draw()` on a `Rectangle`) by inheriting BOTH classes.

> 适配器“RectangleAdapter”允许“LegacyRectangle”通过继承BOTH类来响应请求（“矩形”上的“draw（）”）。
</li>
<li>
<p>The `LegacyRectangle` class does not have the same methods (`draw()`) as `Rectangle`,

but the `Adapter(RectangleAdapter)` can take the `Rectangle` method calls and turn around and invoke method on the `LegacyRectangle`, `oldDraw()`.</p>

> 但是“Adapter（RectangleAdapter）”可以接受“Rectangle”方法调用，并在“LegacyRectangle（）”、“oldDraw（）”上转身调用方法</p>

```cpp
 class RectangleAdapter: public Rectangle, private LegacyRectangle {
   public:
     RectangleAdapter(int x, int y, int w, int h):
       LegacyRectangle(x, y, x + w, y + h) {
         std::cout << "RectangleAdapter(x,y,x+w,x+h)\n";
       }

     void draw() {
         std::cout << "RectangleAdapter: draw().\n"; 
         oldDraw();
     }
 };

```


</li>


**Adapter** design pattern translates the interface for one class into a compatible but different interface. So, this is similar to the **proxy** pattern in that it's a single-component wrapper. But the interface for the adapter class and the original class may be different.

> **适配器**设计模式将一个类的接口转换为兼容但不同的接口。因此，这类似于**proxy**模式，因为它是一个单独的组件包装器。但是适配器类和原始类的接口可能不同。


As we've seen in the example above, this **adapter** pattern is useful to expose a different interface for an existing API to allow it to work with other code. Also, by using adapter pattern, we can take heterogeneous interfaces, and transform them to provide consistent API.

> 正如我们在上面的示例中看到的，这个**适配器**模式对于为现有的API公开不同的接口非常有用，以允许它与其他代码一起工作。此外，通过使用适配器模式，我们可以获取异构接口，并对其进行转换以提供一致的API。


**Bridge pattern** has a structure similar to an object adapter, but Bridge has a different intent: It is meant to **separate** an interface from its implementation so that they can be varied easily and independently. An **adapter** is meant to **change the interface** of an **existing** object.

> **Bridge模式**的结构类似于对象适配器，但Bridge有不同的意图：它旨在**将接口与其实现分离，以便它们可以轻松独立地进行更改。**适配器**用于**更改**现有**对象的接口**。



## Observer pattern



Observer Pattern's intent is to define a one-to-many dependency between objects so that when one object changes state, all its dependents are notified and updated automatically.

> Observer Pattern的目的是定义对象之间的一对多依赖关系，这样当一个对象更改状态时，它的所有依赖关系都会自动得到通知和更新。


The subject and observers define the one-to-many relationship. The observers are dependent on the subject such that when the subject's state changes, the observers get notified. Depending on the notification, the observers may also be updated with new values.

> 主体和观察者定义了一对多关系。观察者依赖于主体，因此当主体的状态发生变化时，观察者会得到通知。根据通知的不同，观察员也可能被更新为新的值。

Here is the example from the book "Design Patterns" by Gamma.

```cpp
#include <iostream>
#include <vector>

class Subject; 

class Observer 
{ 
public:
    virtual ~Observer() = default;
    virtual void Update(Subject&) = 0;
};

class Subject 
{ 
public: 
     virtual ~Subject() = default;
     void Attach(Observer& o) { observers.push_back(&o); }
     void Detach(Observer& o)
     {
         observers.erase(std::remove(observers.begin(), observers.end(), &o));
     }
     void Notify()
     {
         for (auto* o : observers) {
             o->Update(*this);
         }
     }
private:
     std::vector<Observer*> observers; 
};

class ClockTimer : public Subject 
{ 
public:

    void SetTime(int hour, int minute, int second)
    {
        this->hour = hour; 
        this->minute = minute;
        this->second = second;

        Notify(); 
    }

    int GetHour() const { return hour; }
    int GetMinute() const { return minute; }
    int GetSecond() const { return second; }

private: 
    int hour;
    int minute;
    int second;
}; 

class DigitalClock: public Observer 
{ 
public: 
     explicit DigitalClock(ClockTimer& s) : subject(s) { subject.Attach(*this); }
     ~DigitalClock() { subject.Detach(*this); }
     void Update(Subject& theChangedSubject) override
     {
         if (&theChangedSubject == &subject) {
             Draw();
         }
     }

     void Draw()
     {
         int hour = subject.GetHour(); 
         int minute = subject.GetMinute(); 
         int second = subject.GetSecond(); 

         std::cout << "Digital time is " << hour << ":" 
                   << minute << ":" 
                   << second << std::endl;           
     }

private:
     ClockTimer& subject;
};

class AnalogClock: public Observer 
{ 
public: 
     explicit AnalogClock(ClockTimer& s) : subject(s) { subject.Attach(*this); }
     ~AnalogClock() { subject.Detach(*this); }
     void Update(Subject& theChangedSubject) override
     {
         if (&theChangedSubject == &subject) {
             Draw();
         }
     }
     void Draw()
     {
         int hour = subject.GetHour(); 
         int minute = subject.GetMinute(); 
         int second = subject.GetSecond(); 

         std::cout << "Analog time is " << hour << ":" 
                   << minute << ":" 
                   << second << std::endl; 
     }
private:
     ClockTimer& subject;
};

int main()
{ 
    ClockTimer timer; 

    DigitalClock digitalClock(timer); 
    AnalogClock analogClock(timer); 

    timer.SetTime(14, 41, 36);
}

```

Output:

```cpp
Digital time is 14:41:36
Analog time is 14:41:36

```

Here are the summary of the pattern:

<li>

Objects (`DigitalClock` or `AnalogClock` object) use the Subject interfaces (`Attach()` or `Detach()`) either to subscribe (register) as observers or unsubscribe (remove) themselves from being observers (`subject.Attach(*this);` , `subject.Detach(*this);`.

> 对象（`DigitalClock`或`AnalogClock`对象）使用Subject接口（`Attach（）`或`Detach（）`）订阅（注册）为观察者或取消订阅（删除）自身为观察者（`Subject.Attach（*this）；``主题分离（*this）；`。
</li>
<li>
Each subject can have many observers( `vector<Observer*> observers;`).
</li>
<li>

All observers need to implement the Observer interface. This interface just has one method, `Update()`, that gets called when the Subject's state changes (`Update(Subject &)`)

> 所有观察者都需要实现观察者界面。这个接口只有一个方法“Update（）”，当Subject的状态更改时调用它（“Update（Subject&）”）
</li>
<li>

In addition to the `Attach()` and `Detach()` methods, the concrete subject implements a `Notify()` method that is used to update all the current observers whenever state changes. But in this case, all of them are done in the parent class, `Subject` (`Subject::Attach (Observer&)`, `void Subject::Detach(Observer&)` and `void Subject::Notify()` .

> 除了“Attach（）”和“Detach（）”方法外，具体主题还实现了一个“Notify（）”法，用于在状态发生变化时更新所有当前观察者。但在这种情况下，所有这些操作都是在父类“Subject”（“Subject：：Attach（Observer&）”、“void Subject：:Detach（Observer和）”和“void Subject:：Notify（）”中完成的。
</li>
<li>

The Concrete object may also have methods for setting and getting its state.

> Concrete对象也可能具有用于设置和获取其状态的方法。
</li>
<li>

Concrete observers can be any class that implements the Observer interface. Each observer subscribe (register) with a concrete subject to receive update (`subject.Attach(*this);` ).

> 具体的观察者可以是实现观察者接口的任何类。每个观察者订阅（注册）一个具体的主题以接收更新（`subject.Attach（*this）；`）。
</li>
<li>

The two objects of Observer Pattern are **loosely coupled**, they can interact but with little knowledge of each other.

> 观察者模式的两个对象**是松散耦合的**，它们可以交互，但对彼此知之甚少。
</li>

**Variation:**

Signal and Slots


Signals and slots is a language construct introduced in Qt, which makes it easy to implement the Observer pattern while avoiding boilerplate code. The concept is that controls (also known as widgets) can send signals containing event information which can be received by other controls using special functions known as slots. The slot in Qt must be a class member declared as such.

> Signals and slots是Qt中引入的一种语言结构，它使实现Observer模式变得容易，同时避免了样板代码。其概念是，控件（也称为小部件）可以发送包含事件信息的信号，其他控件可以使用称为插槽的特殊功能接收这些信号。Qt中的slot必须是这样声明的类成员。

The signal/slot system fits well with the way Graphical User Interfaces are designed. Similarly, the signal/slot system can be used for asynchronous I/O (including sockets, pipes, serial devices, etc.) event notification or to associate timeout events with appropriate object instances and methods or functions. No registration/deregistration/invocation code need be written, because Qt's Meta Object Compiler (MOC) automatically generates the needed infrastructure.

> 信号/插槽系统非常适合图形用户界面的设计方式。类似地，信号/插槽系统可以用于异步I/O（包括套接字、管道、串行设备等）事件通知，或者将超时事件与适当的对象实例、方法或函数相关联。不需要编写注册/注销/调用代码，因为Qt的元对象编译器（MOC）会自动生成所需的基础设施。


The C# language also supports a similar construct although with a different terminology and syntax: events play the role of signals, and delegates are the slots. Additionally, a delegate can be a local variable, much like a function pointer, while a slot in Qt must be a class member declared as such.

> C#语言也支持类似的结构，尽管有不同的术语和语法：事件扮演信号的角色，委托是槽。此外，委托可以是局部变量，就像函数指针一样，而Qt中的槽必须是这样声明的类成员。



## Factory Pattern



Factory pattern decouples object creation and allows creation by name using a common interface:

> 工厂模式将对象创建解耦，并允许使用公共接口按名称创建：

```cpp
class Animal{
public:
    virtual std::shared_ptr<Animal> clone() const = 0;
    virtual std::string  getname() const = 0;
};

class Bear: public Animal{
public:
    virtual std::shared_ptr<Animal> clone() const override
    {
        return std::make_shared<Bear>(*this);
    }
    virtual std::string getname() const override
    {
        return "bear";
    }
};


class Cat: public Animal{
public:
    virtual std::shared_ptr<Animal> clone() const override
    {
        return std::make_shared<Cat>(*this);
    }
    virtual std::string  getname() const override
    {
        return "cat";
    }
};

class AnimalFactory{
public:
    static std::shared_ptr<Animal> getAnimal( const std::string&   name )
    {
      if ( name == "bear" )
        return std::make_shared<Bear>();
      if ( name == "cat" )
        return std::shared_ptr<Cat>();
     
    return nullptr;
    }


};

```



## Builder Pattern with Fluent API



The Builder Pattern decouples the creation of the object from the object itself. The main idea behind is that **an object does not have to be responsible for its own creation**. The correct and valid assembly of a complex object may be a complicated task in itself, so this task can be delegated to another class.

> 生成器模式将对象的创建与对象本身解耦。背后的主要思想是**一个对象不必对自己的创建负责**。一个复杂对象的正确有效的程序集本身可能是一个复杂的任务，因此该任务可以委托给另一个类。


Inspired by the [Email Builder in C#](http://stackoverflow.com/documentation/design-patterns/1811/builder-pattern/5912/builder-pattern-c-sharp-fluent-interrface#t=201704231441269920397), I've decided to make a C++ version here. An Email object is not necessarily a **very complex object**, but it can demonstrate the pattern.

> 灵感来自[C#中的电子邮件生成器](http://stackoverflow.com/documentation/design-patterns/1811/builder-pattern/5912/builder-pattern-c-sharp-fluent-interrface#t=201704231441269920397)，我决定在这里制作一个C++版本。电子邮件对象不一定是**非常复杂的对象**，但它可以演示模式。

```cpp
#include <iostream>
#include <sstream>
#include <string>

using namespace std;

// Forward declaring the builder
class EmailBuilder;

class Email
{
  public:
    friend class EmailBuilder;  // the builder can access Email's privates
    
    static EmailBuilder make();
    
    string to_string() const {
        stringstream stream;
        stream << "from: " << m_from
               << "\nto: " << m_to
               << "\nsubject: " << m_subject
               << "\nbody: " << m_body;
        return stream.str();
    }
    
  private:
    Email() = default; // restrict construction to builder
    
    string m_from;
    string m_to;
    string m_subject;
    string m_body;
};

class EmailBuilder
{
  public:
    EmailBuilder& from(const string &from) {
        m_email.m_from = from;
        return *this;
    }
    
    EmailBuilder& to(const string &to) {
        m_email.m_to = to;
        return *this;
    }
    
    EmailBuilder& subject(const string &subject) {
        m_email.m_subject = subject;
        return *this;
    }
    
    EmailBuilder& body(const string &body) {
        m_email.m_body = body;
        return *this;
    }
    
    operator Email&&() {
        return std::move(m_email); // notice the move
    }
    
  private:
    Email m_email;
};

EmailBuilder Email::make()
{
    return EmailBuilder();
}

// Bonus example!
std::ostream& operator <<(std::ostream& stream, const Email& email)
{
    stream << email.to_string();
    return stream;
}


int main()
{
    Email mail = Email::make().from("me@mail.com")
                              .to("you@mail.com")
                              .subject("C++ builders")
                              .body("I like this API, don't you?");
                              
    cout << mail << endl;
}

```


For older versions of C++, one may just ignore the `std::move` operation and remove the && from the conversion operator (although this will create a temporary copy).

> 对于较旧版本的C++，可以忽略“std:：move”操作，并从转换运算符中删除&&（尽管这将创建一个临时副本）。


The builder finishes its work when it releases the built email by the `operator Email&&()`. In this example, the builder is a temporary object and returns the email before being destroyed. You could also use an explicit operation like `Email EmailBuilder::build() {...}` instead of the conversion operator.

> 当构建器通过“操作员电子邮件&&（）”发布构建的电子邮件时，它就完成了工作。在本例中，生成器是一个临时对象，在被销毁之前返回电子邮件。您也可以使用像“电子邮件生成器：：build（）｛…｝”这样的显式操作来代替转换运算符。

### Pass the builder around


A great feature the Builder Pattern provides is the ability to **use several actors to build an object together.** This is done by passing the builder to the other actors that will each one give some more information to the built object. This is specially powerful when you are building some sort of query, adding filters and other specifications.

> Builder Pattern提供的一个很好的功能是能够**使用几个参与者一起构建一个对象。**这是通过将生成器传递给其他参与者来完成的，每个参与者将为构建的对象提供更多信息。当您构建某种查询、添加过滤器和其他规范时，这一功能特别强大。

```cpp
void add_addresses(EmailBuilder& builder)
{
    builder.from("me@mail.com")
           .to("you@mail.com");
}

void compose_mail(EmailBuilder& builder)
{
    builder.subject("I know the subject")
           .body("And the body. Someone else knows the addresses.");
}

int main()
{
    EmailBuilder builder;
    add_addresses(builder);
    compose_mail(builder);
    
    Email mail = builder;
    cout << mail << endl;
}

```

### Design variant : Mutable object


You can change the design of this pattern to fit your needs. I'll give one variant.

> 您可以根据需要更改此图案的设计。我会给出一个变体。


In the given example the Email object is immutable, i.e., it's properties can't be modified because there is no access to them. This was a desired feature. If you need to modify the object after its creation you have to provide some setters to it. Since those setters would be duplicated in the builder, you may consider to do it all in one class (no builder class needed anymore). Nevertheless, I would consider the need to make the built object mutable in the first place.

> 在给定的示例中，电子邮件对象是不可变的，即它的属性不能修改，因为无法访问它们。这是一个理想的功能。如果你需要在对象创建后修改它，你必须为它提供一些setter。由于这些setter在生成器中是重复的，你可以考虑在一个类中完成所有操作（不再需要生成器类）。尽管如此，我还是会首先考虑让构建的对象可变的必要性。



#### Remarks



A design pattern is a general reusable solution to a commonly occurring problem within a given context in software design.

> 设计模式是对软件设计中给定上下文中常见问题的通用可重用解决方案。

