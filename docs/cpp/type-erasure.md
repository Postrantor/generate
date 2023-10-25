---
tip: translate by baidu@2023-10-25 08:35:03
---
---
metaTitle: "C++ | Type Erasure"
description: "A move-only `std::function`, Erasing down to a Regular type with manual vtable, Basic mechanism, Erasing down to a contiguous buffer of T, Type erasing type erasure with std::any"
---

# Type Erasure



Type erasure is a set of techniques for creating a type that can provide a uniform interface to various underlying types, while hiding the underlying type information from the client. `std::function<R(A...)>`, which has the ability to hold callable objects of various types, is perhaps the best known example of type erasure in C++.

> 类型擦除是一组用于创建类型的技术，该类型可以为各种底层类型提供统一的接口，同时向客户端隐藏底层类型信息`std:：function＜R（A…）＞`能够保存各种类型的可调用对象，它可能是C++中最著名的类型擦除示例。



## A move-only `std::function`


`std::function` type erases down to a few operations.  One of the things it requires is that the stored value be copyable.


This causes problems in a few contexts, like lambdas storing unique ptrs.  If you are using the `std::function` in a context where copying doesn't matter, like a thread pool where you dispatch tasks to threads, this requirement can add overhead.

> 这会在一些上下文中引起问题，比如lambdas存储唯一的ptr。如果在复制无关紧要的上下文中使用“std:：函数”，比如在线程池中向线程分派任务，则此要求可能会增加开销。


In particular, `std::packaged_task<Sig>` is a callable object that is move-only.  You can store a `std::packaged_task<R(Args...)>` in a `std::packaged_task<void(Args...)>`, but that is a pretty heavy-weight and obscure way to create a move-only callable type-erasure class.

> 特别是，“std:：packaged_task＜Sig＞”是一个只能移动的可调用对象。您可以将“std:：packaged_task<R（Args…）>”存储在“std::：package d_task<void（Args..）>”中，但这是一种非常沉重且晦涩的方法来创建仅移动可调用的类型擦除类。


Thus the `task`.  This demonstrates how you could write a simple `std::function` type.  I omitted the copy constructor (which would involve adding a `clone` method to `details::task_pimpl<...>` as well).

> 因此，“任务”。这演示了如何编写一个简单的“std:：function”类型。我省略了复制构造函数（这将涉及到在“details:：task_pimpl<…>”中添加一个“clone”方法）。

```cpp
template<class Sig>
struct task;

// putting it in a namespace allows us to specialize it nicely for void return value:
namespace details {
  template<class R, class...Args>
  struct task_pimpl {
    virtual R invoke(Args&&...args) const = 0;
    virtual ~task_pimpl() {};
    virtual const std::type_info& target_type() const = 0;
  };

  // store an F.  invoke(Args&&...) calls the f
  template<class F, class R, class...Args>
  struct task_pimpl_impl:task_pimpl<R,Args...> {
    F f;
    template<class Fin>
    task_pimpl_impl( Fin&& fin ):f(std::forward<Fin>(fin)) {}
    virtual R invoke(Args&&...args) const final override {
      return f(std::forward<Args>(args)...);
    }
    virtual const std::type_info& target_type() const final override {
      return typeid(F);
    }
  };

  // the void version discards the return value of f:
  template<class F, class...Args>
  struct task_pimpl_impl<F,void,Args...>:task_pimpl<void,Args...> {
    F f;
    template<class Fin>
    task_pimpl_impl( Fin&& fin ):f(std::forward<Fin>(fin)) {}
    virtual void invoke(Args&&...args) const final override {
      f(std::forward<Args>(args)...);
    }
    virtual const std::type_info& target_type() const final override {
      return typeid(F);
    }
  };
};

template<class R, class...Args>
struct task<R(Args...)> {
  // semi-regular:
  task()=default;
  task(task&&)=default;
  // no copy

private:
  // aliases to make some SFINAE code below less ugly:
  template<class F>
  using call_r = std::result_of_t<F const&(Args...)>;
  template<class F>
  using is_task = std::is_same<std::decay_t<F>, task>;
public:
  // can be constructed from a callable F
  template<class F,
    // that can be invoked with Args... and converted-to-R:
    class= decltype( (R)(std::declval<call_r<F>>()) ),
    // and is not this same type:
    std::enable_if_t<!is_task<F>{}, int>* = nullptr
  >
  task(F&& f):
    m_pImpl( make_pimpl(std::forward<F>(f)) )
  {}

  // the meat: the call operator        
  R operator()(Args... args)const {
        return m_pImpl->invoke( std::forward<Args>(args)... );
  }
  explicit operator bool() const {
    return (bool)m_pImpl;
  }
  void swap( task& o ) {
    std::swap( m_pImpl, o.m_pImpl );
  }
  template<class F>
  void assign( F&& f ) {
    m_pImpl = make_pimpl(std::forward<F>(f));    
  }
  // Part of the std::function interface:
  const std::type_info& target_type() const {
    if (!*this) return typeid(void);
    return m_pImpl->target_type();
  }
  template< class T >
  T* target() {
    return target_impl<T>();
  }
  template< class T >
  const T* target() const {
    return target_impl<T>();
  }
  // compare with nullptr    :    
  friend bool operator==( std::nullptr_t, task const& self ) { return !self; }
  friend bool operator==( task const& self, std::nullptr_t ) { return !self; }
  friend bool operator!=( std::nullptr_t, task const& self ) { return !!self; }
  friend bool operator!=( task const& self, std::nullptr_t ) { return !!self; }
private:
  template<class T>
  using pimpl_t = details::task_pimpl_impl<T, R, Args...>;

  template<class F>
  static auto make_pimpl( F&& f ) {
    using dF=std::decay_t<F>;
    using pImpl_t = pimpl_t<dF>;
    return std::make_unique<pImpl_t>(std::forward<F>(f));
  }
  std::unique_ptr<details::task_pimpl<R,Args...>> m_pImpl;

  template< class T >
  T* target_impl() const {
    return dynamic_cast<pimpl_t<T>*>(m_pImpl.get());
  }
};

```


To make this library-worthy, you'd want to add in a small buffer optimization, so it does not store every callable on the heap.

> 为了使这个库有价值，您需要添加一个小的缓冲区优化，这样它就不会将每个可调用的存储在堆上。


Adding SBO would require a non-default `task(task&&)`, some `std::aligned_storage_t` within the class, a `m_pImpl` `unique_ptr` with a deleter that can be set to destroy-only (and not return the memory to the heap), and a `emplace_move_to( void* ) = 0` in the `task_pimpl`.

> 添加SBO需要一个非默认的“task（task&&）”、类中的一些“std:：aligned_storage_t”、一个带有可设置为仅销毁（而不将内存返回堆）的deleter的“m_pImpl”“unique_ptr”，以及“task_pImpl”中的“template_move_to（void*）=0”。


[live example](http://coliru.stacked-crooked.com/a/6e6811e8626a37d1) of the above code (with no SBO).

> [实例](http://coliru.stacked-crooked.com/a/6e6811e8626a37d1)上述代码的（没有SBO）。



## Erasing down to a Regular type with manual vtable



C++ thrives on what is known as a Regular type (or at least Pseudo-Regular).

> C++在所谓的正则类型（或者至少是伪正则类型）上蓬勃发展。


A Regular type is a type that can be constructed and assigned-to and assigned-from via copy or move, can be destroyed, and can be compared equal-to.  It can also be constructed from no arguments.  Finally, it also has support for a few other operations that are highly useful in various `std` algorithms and containers.

> Regular类型是一种可以通过复制或移动构造、分配给和从中分配的类型，可以销毁，并且可以与进行比较。它也可以由无参数构造而成。最后，它还支持其他一些在各种“std”算法和容器中非常有用的操作。


[This is the root paper](http://www.stepanovpapers.com/DeSt98.pdf), but in C++11 would want to add `std::hash` support.

> [这是根文件](http://www.stepanovpapers.com/DeSt98.pdf)，但在C++11中，希望添加“std:：hash”支持。

I will use the manual vtable approach to type erasure here.

```cpp
using dtor_unique_ptr = std::unique_ptr<void, void(*)(void*)>;
template<class T, class...Args>
dtor_unique_ptr make_dtor_unique_ptr( Args&&... args ) {
  return {new T(std::forward<Args>(args)...), [](void* self){ delete static_cast<T*>(self); }};
}
struct regular_vtable {
  void(*copy_assign)(void* dest, void const* src); // T&=(T const&)
  void(*move_assign)(void* dest, void* src); // T&=(T&&)
  bool(*equals)(void const* lhs, void const* rhs); // T const&==T const&
  bool(*order)(void const* lhs, void const* rhs); // std::less<T>{}(T const&, T const&)
  std::size_t(*hash)(void const* self); // std::hash<T>{}(T const&)
  std::type_info const&(*type)(); // typeid(T)
  dtor_unique_ptr(*clone)(void const* self); // T(T const&)
};

template<class T>
regular_vtable make_regular_vtable() noexcept {
  return {
    [](void* dest, void const* src){ *static_cast<T*>(dest) = *static_cast<T const*>(src); },
    [](void* dest, void* src){ *static_cast<T*>(dest) = std::move(*static_cast<T*>(src)); },
    [](void const* lhs, void const* rhs){ return *static_cast<T const*>(lhs) == *static_cast<T const*>(rhs); },
    [](void const* lhs, void const* rhs) { return std::less<T>{}(*static_cast<T const*>(lhs),*static_cast<T const*>(rhs)); },
    [](void const* self){ return std::hash<T>{}(*static_cast<T const*>(self)); },
    []()->decltype(auto){ return typeid(T); },
    [](void const* self){ return make_dtor_unique_ptr<T>(*static_cast<T const*>(self)); }
  };
}
template<class T>
regular_vtable const* get_regular_vtable() noexcept {
  static const regular_vtable vtable=make_regular_vtable<T>();
  return &vtable;
}

struct regular_type {
  using self=regular_type;
  regular_vtable const* vtable = 0;
  dtor_unique_ptr ptr{nullptr, [](void*){}};
  
  bool empty() const { return !vtable; }

  template<class T, class...Args>
  void emplace( Args&&... args ) {
    ptr = make_dtor_unique_ptr<T>(std::forward<Args>(args)...);
    if (ptr)
      vtable = get_regular_vtable<T>();
    else
      vtable = nullptr;
  }
  friend bool operator==(regular_type const& lhs, regular_type const& rhs) {
    if (lhs.vtable != rhs.vtable) return false;
    return lhs.vtable->equals( lhs.ptr.get(), rhs.ptr.get() );
  }
  bool before(regular_type const& rhs) const {
    auto const& lhs = *this;
    if (!lhs.vtable || !rhs.vtable)
      return std::less<regular_vtable const*>{}(lhs.vtable,rhs.vtable);
    if (lhs.vtable != rhs.vtable)
      return lhs.vtable->type().before(rhs.vtable->type());
    return lhs.vtable->order( lhs.ptr.get(), rhs.ptr.get() );
  }
  // technically friend bool operator< that calls before is also required

  std::type_info const* type() const {
    if (!vtable) return nullptr;
    return &vtable->type();
  }
  regular_type(regular_type&& o):
    vtable(o.vtable),
    ptr(std::move(o.ptr))
  {
    o.vtable = nullptr;
  }
  friend void swap(regular_type& lhs, regular_type& rhs){
    std::swap(lhs.ptr, rhs.ptr);
    std::swap(lhs.vtable, rhs.vtable);
  }
  regular_type& operator=(regular_type&& o) {
    if (o.vtable == vtable) {
      vtable->move_assign(ptr.get(), o.ptr.get());
      return *this;
    }
    auto tmp = std::move(o);
    swap(*this, tmp);
    return *this;
  }
  regular_type(regular_type const& o):
    vtable(o.vtable),
    ptr(o.vtable?o.vtable->clone(o.ptr.get()):dtor_unique_ptr{nullptr, [](void*){}})
  {
    if (!ptr && vtable) vtable = nullptr;
  }
  regular_type& operator=(regular_type const& o) {
    if (o.vtable == vtable) {
      vtable->copy_assign(ptr.get(), o.ptr.get());
      return *this;
    }
    auto tmp = o;
    swap(*this, tmp);
    return *this;
  }
  std::size_t hash() const {
    if (!vtable) return 0;
    return vtable->hash(ptr.get());
  }
  template<class T,
    std::enable_if_t< !std::is_same<std::decay_t<T>, regular_type>{}, int>* =nullptr
  >
  regular_type(T&& t) {
    emplace<std::decay_t<T>>(std::forward<T>(t));
  }
};
namespace std {
  template<>
  struct hash<regular_type> {
    std::size_t operator()( regular_type const& r )const {
      return r.hash();
    }
  };
  template<>
  struct less<regular_type> {
    bool operator()( regular_type const& lhs, regular_type const& rhs ) const {
      return lhs.before(rhs);
    }
  };
}    

```

[live example](http://coliru.stacked-crooked.com/a/28ef6be761012a81).


Such a regular type can be used as a key for a `std::map` or a `std::unordered_map` that accepts **anything regular** for a key, like:

> 这样的正则类型可以用作“std:：map”或“std：：unordereded_map”的键，后者接受**任何正则**作为键，如：

```cpp
std::map<regular_type, std::any>

```

would be basically a map from anothing regular, to anything copyable.


Unlike `any`, my `regular_type` does no small object optimization nor does it support getting the original data back.  Getting the original type back isn't hard.

> 与“any”不同，我的“regular_type”不进行小对象优化，也不支持取回原始数据。找回原来的类型并不难。


Small object optimization requires that we store an aligned storage buffer within the `regular_type`, and carefully tweak the deleter of the `ptr` to only destroy the object and not delete it.

> 小对象优化要求我们在“regular_type”中存储一个对齐的存储缓冲区，并仔细调整“ptr”的deleter，使其仅销毁对象，而不删除它。


I would start at `make_dtor_unique_ptr` and teach it how to sometimes store the data in a buffer, and then in the heap if no room in the buffer.  That may be sufficient.

> 我会从“make_dtor_unique_ptr”开始，教它如何有时将数据存储在缓冲区中，如果缓冲区中没有空间，则存储在堆中。这可能就足够了。



## Basic mechanism



Type erasure is a way to hide the type of an object from code using it, even though it is not derived from a common base class. In doing so, it provides a bridge between the worlds of static polymorphism (templates; at the place of use, the exact type must be known at compile time, but it need not be declared to conform to an interface at definition) and dynamic polymorphism (inheritance and virtual functions; at the place of use, the exact type need not be known at compile time, but must be declared to conform to an interface at definition).

> 类型擦除是一种对使用对象的代码隐藏对象类型的方法，即使它不是从公共基类派生的。在这样做的过程中，它提供了静态多态性（模板；在使用时，确切的类型必须在编译时已知，但不需要在定义时声明为符合接口）和动态多态性之间的桥梁（继承和虚拟函数；在使用时，确切的类型在编译时不需要知道，但必须在定义时声明为符合接口）。

The following code shows the basic mechanism of type erasure.

```cpp
#include <ostream>

class Printable
{
public:
  template <typename T>
  Printable(T value) : pValue(new Value<T>(value)) {}
  ~Printable() { delete pValue; }
  void print(std::ostream &os) const { pValue->print(os); }

private:
  Printable(Printable const &)        /* in C++1x: =delete */; // not implemented
  void operator = (Printable const &) /* in C++1x: =delete */; // not implemented
  struct ValueBase
  {
      virtual ~ValueBase() = default;
      virtual void print(std::ostream &) const = 0;
  };
  template <typename T>
  struct Value : ValueBase
  {
      Value(T const &t) : v(t) {}
      virtual void print(std::ostream &os) const { os << v; }
      T v;
  };
  ValueBase *pValue;
};

```


At the use site, only the above definition need to be visible, just as with base classes with virtual functions. For example:

> 在使用站点，只有上面的定义需要是可见的，就像具有虚拟函数的基类一样。例如：

```cpp
#include <iostream>

void print_value(Printable const &p)
{
    p.print(std::cout);
}

```


Note that this is **not** a template, but a normal function that only needs to be declared in a header file, and can be defined in an implementation file (unlike templates, whose definition must be visible at the place of use).

> 请注意，这不是**模板，而是一个普通函数，只需要在头文件中声明，并且可以在实现文件中定义（与模板不同，模板的定义必须在使用位置可见）。


At the definition of the concrete type, nothing needs to be known about `Printable`, it just needs to conform to an interface, as with templates:

> 在具体类型的定义中，不需要知道“可打印”，它只需要符合接口，就像模板一样：

```cpp
struct MyType { int i; };
ostream& operator << (ostream &os, MyType const &mc)
{
  return os << "MyType {" << mc.i << "}";
}

```

We can now pass an object of this class to the function defined above:

```cpp
MyType foo = { 42 };
print_value(foo);

```



## Erasing down to a contiguous buffer of T



Not all type erasure involves virtual inheritance, allocations, placement new, or even function pointers.

> 并不是所有类型的擦除都涉及虚拟继承、分配、新放置，甚至函数指针。


What makes type erasure type erasure is that it describes a (set of) behavior(s), and takes any type that supports that behavior and wraps it up.  All information that isn't in that set of behaviors is "forgotten" or "erased".

> 使类型擦除成为类型擦除的原因是它描述了一组行为，并采用任何支持该行为的类型并将其封装起来。所有不属于这组行为的信息都会被“遗忘”或“抹去”。


An `array_view` takes its incoming range or container type and erases everything except the fact it is a contiguous buffer of `T`.

> “array_view”接受其传入的范围或容器类型，并擦除除它是“T”的连续缓冲区之外的所有内容。

```cpp
// helper traits for SFINAE:
template<class T>
using data_t = decltype( std::declval<T>().data() );

template<class Src, class T>
using compatible_data = std::integral_constant<bool, std::is_same< data_t<Src>, T* >{} || std::is_same< data_t<Src>, std::remove_const_t<T>* >{}>;

template<class T>
struct array_view {
  // the core of the class:
  T* b=nullptr;
  T* e=nullptr;
  T* begin() const { return b; }
  T* end() const { return e; }

  // provide the expected methods of a good contiguous range:
  T* data() const { return begin(); }
  bool empty() const { return begin()==end(); }
  std::size_t size() const { return end()-begin(); }

  T& operator[](std::size_t i)const{ return begin()[i]; }
  T& front()const{ return *begin(); }
  T& back()const{ return *(end()-1); }

  // useful helpers that let you generate other ranges from this one
  // quickly and safely:
  array_view without_front( std::size_t i=1 ) const {
    i = (std::min)(i, size());
    return {begin()+i, end()};
  }
  array_view without_back( std::size_t i=1 ) const {
    i = (std::min)(i, size());
    return {begin(), end()-i};
  }

  // array_view is plain old data, so default copy:
  array_view(array_view const&)=default;
  // generates a null, empty range:
  array_view()=default;

  // final constructor:
  array_view(T* s, T* f):b(s),e(f) {}
  // start and length is useful in my experience:
  array_view(T* s, std::size_t length):array_view(s, s+length) {}

  // SFINAE constructor that takes any .data() supporting container
  // or other range in one fell swoop:
  template<class Src,
    std::enable_if_t< compatible_data<std::remove_reference_t<Src>&, T >{}, int>* =nullptr,
    std::enable_if_t< !std::is_same<std::decay_t<Src>, array_view >{}, int>* =nullptr
  >
  array_view( Src&& src ):
    array_view( src.data(), src.size() )
  {}

  // array constructor:
  template<std::size_t N>
  array_view( T(&arr)[N] ):array_view(arr, N) {}

  // initializer list, allowing {} based:
  template<class U,
    std::enable_if_t< std::is_same<const U, T>{}, int>* =nullptr
  >
  array_view( std::initializer_list<U> il ):array_view(il.begin(), il.end()) {}
};

```


an `array_view` takes any container that supports `.data()` returning a pointer to `T` and a `.size()` method, or an array, and erases it down to being a random-access range over contiguous `T`s.

> “array_view”接受任何支持“.data（）”返回指向“T”的指针和“.size（）”方法或数组的容器，并将其擦除为连续“T”上的随机访问范围。


It can take a `std::vector<T>`, a `std::string<T>` a `std::array<T, N>` a `T[37]`, an initializer list (including `{}` based ones), or something else you make up that supports it (via `T* x.data()` and `size_t x.size()`).

> 它可以采用一个“std:：vector＜T＞”、一个“std：：string＜T＞’a”std:：array＜T，N＞“a”T[37]“、一个初始值设定项列表（包括基于“｛｝”的列表）或其他支持它的东西（通过“T*x.data（）”和“size_T x.size（）”）。


In this case, the data we can extract from the thing we are erasing, together with our "view" non-owning state, means we don't have to allocate memory or write custom type-dependent functions.

> 在这种情况下，我们可以从正在擦除的东西中提取数据，再加上我们的“视图”非拥有状态，这意味着我们不必分配内存或编写自定义类型相关函数。

[Live example](http://coliru.stacked-crooked.com/a/c9f8e013a309ca66).


An improvement would be to use a non-member `data` and a non-member `size` in an ADL-enabled context.

> 一种改进是在启用ADL的上下文中使用非成员“数据”和非成员“大小”。



## Type erasing type erasure with std::any



This example uses C++14 and `boost::any`.  In C++17 you can swap in `std::any` instead.

> 此示例使用C++14和“boost:：any”。在C++17中，您可以使用“std:：any”进行交换。

The syntax we end up with is:

```cpp
const auto print =
  make_any_method<void(std::ostream&)>([](auto&& p, std::ostream& t){ t << p << "\n"; });

super_any<decltype(print)> a = 7;

(a->*print)(std::cout);

```

which is almost optimal.


This example is based off of work by [@dyp](http://coliru.stacked-crooked.com/a/2ab8d7e41d24e616) and [@cpplearner](http://stackoverflow.com/a/38865269/1774667) as well as my own.

> 此示例基于[@dyp]的工作(http://coliru.stacked-crooked.com/a/2ab8d7e41d24e616)和[@cpplearner](http://stackoverflow.com/a/38865269/1774667)以及我自己。

First we use a tag to pass around types:

```cpp
template<class T>struct tag_t{constexpr tag_t(){};};
template<class T>constexpr tag_t<T> tag{};

```

This trait class gets the signature stored with an `any_method`:


This creates a function pointer type, and a factory for said function pointers, given an `any_method`:

> 这将创建一个函数指针类型，以及给定“any_method”的所述函数指针的工厂：

```cpp
template<class any_method>
using any_sig_from_method = typename any_method::signature;

template<class any_method, class Sig=any_sig_from_method<any_method>>
struct any_method_function;

template<class any_method, class R, class...Args>
struct any_method_function<any_method, R(Args...)>
{
  template<class T>
  using decorate = std::conditional_t< any_method::is_const, T const, T >;
  
  using any = decorate<boost::any>;
  
  using type = R(*)(any&, any_method const*, Args&&...);
  template<class T>
  type operator()( tag_t<T> )const{
    return +[](any& self, any_method const* method, Args&&...args) {
      return (*method)( boost::any_cast<decorate<T>&>(self), decltype(args)(args)... );
    };
  }
};

```

`any_method_function::type` is the type of a function pointer we will store alongside the instance.  `any_method_function::operator()` takes a `tag_t<T>` and writes a custom instance of the `any_method_function::type` that assumes the `any&` is going to be a `T`.


We want to be able to type-erase more than one method at a time.  So we bundle them up in a tuple, and write a helper wrapper to stick the tuple into static storage on a per-type basis and maintain a pointer to them.

> 我们希望能够一次键入擦除多个方法。因此，我们将它们捆绑在一个元组中，并编写一个辅助包装器，以按类型将元组粘贴到静态存储中，并维护指向它们的指针。

```cpp
template<class...any_methods>
using any_method_tuple = std::tuple< typename any_method_function<any_methods>::type... >;

template<class...any_methods, class T>
any_method_tuple<any_methods...> make_vtable( tag_t<T> ) {
  return std::make_tuple(
    any_method_function<any_methods>{}(tag<T>)...
  );
}

template<class...methods>
struct any_methods {
private:
  any_method_tuple<methods...> const* vtable = 0;
  template<class T>
  static any_method_tuple<methods...> const* get_vtable( tag_t<T> ) {
    static const auto table = make_vtable<methods...>(tag<T>);
    return &table;
  }
public:
  any_methods() = default;
  template<class T>
  any_methods( tag_t<T> ): vtable(get_vtable(tag<T>)) {}
  any_methods& operator=(any_methods const&)=default;
  template<class T>
  void change_type( tag_t<T> ={} ) { vtable = get_vtable(tag<T>); }
    
  template<class any_method>
  auto get_invoker( tag_t<any_method> ={} ) const {
    return std::get<typename any_method_function<any_method>::type>( *vtable );
  }
};

```


We could specialize this for a cases where the vtable is small (for example, 1 item), and use direct pointers stored in-class in those cases for efficiency.

> 我们可以将其专门用于vtable较小的情况（例如，1项），并在这些情况下使用存储在类中的直接指针以提高效率。


Now we start the `super_any`.  I use `super_any_t` to make the declaration of `super_any` a bit easier.

> 现在我们开始“super_any”。我使用“super_any_t”来简化“super_any”的声明。

```cpp
template<class...methods>
struct super_any_t;

```


This searches the methods that the super any supports for SFINAE and better error messages:

> 这将搜索super-any支持的SFINAE和更好的错误消息的方法：

```cpp
template<class super_any, class method>
struct super_method_applies_helper : std::false_type {};

template<class M0, class...Methods, class method>
struct super_method_applies_helper<super_any_t<M0, Methods...>, method> :
    std::integral_constant<bool, std::is_same<M0, method>{}  || super_method_applies_helper<super_any_t<Methods...>, method>{}>
{};

template<class...methods, class method>
auto super_method_test( super_any_t<methods...> const&, tag_t<method> )
{
  return std::integral_constant<bool, super_method_applies_helper< super_any_t<methods...>, method >{} && method::is_const >{};
}
template<class...methods, class method>
auto super_method_test( super_any_t<methods...>&, tag_t<method> )
{
  return std::integral_constant<bool, super_method_applies_helper< super_any_t<methods...>, method >{} >{};
}

template<class super_any, class method>
struct super_method_applies:
    decltype( super_method_test( std::declval<super_any>(), tag<method> ) )
{};

```

Next we create the `any_method` type.  An `any_method` is a pseudo-method-pointer.  We create it globally and `const`ly using syntax like:

```cpp
const auto print=make_any_method( [](auto&&self, auto&&os){ os << self; } );

```

or in C++17:

```cpp
const any_method print=[](auto&&self, auto&&os){ os << self; };

```


Note that using a non-lambda can make things hairy, as we use the type for a lookup step.  This can be fixed, but would make this example longer than it already is.  So always initialize an any method from a lambda, or from a type parametarized on a lambda.

> 请注意，使用非lambda可能会让事情变得棘手，因为我们将该类型用于查找步骤。这是可以修复的，但会使这个例子比现在更长。因此，始终从lambda或lambda上的参数化类型初始化any方法。

```cpp
template<class Sig, bool const_method, class F>
struct any_method {
  using signature=Sig;
  enum{is_const=const_method};
private:
  F f;
public:

  template<class Any,
    // SFINAE testing that one of the Anys's matches this type:
    std::enable_if_t< super_method_applies< Any&&, any_method >{}, int>* =nullptr
  >
  friend auto operator->*( Any&& self, any_method const& m ) {
    // we don't use the value of the any_method, because each any_method has
    // a unique type (!) and we check that one of the auto*'s in the super_any
    // already has a pointer to us.  We then dispatch to the corresponding
    // any_method_data...

    return [&self, invoke = self.get_invoker(tag<any_method>), m](auto&&...args)->decltype(auto)
    {
      return invoke( decltype(self)(self), &m, decltype(args)(args)... );
    };
  }
  any_method( F fin ):f(std::move(fin)) {}
  
  template<class...Args>
  decltype(auto) operator()(Args&&...args)const {
    return f(std::forward<Args>(args)...);
  }
};

```

A factory method, not needed in C++17 I believe:

```cpp
template<class Sig, bool is_const=false, class F>
any_method<Sig, is_const, std::decay_t<F>>
make_any_method( F&& f ) {
  return {std::forward<F>(f)};
}

```


This is the augmented `any`.  It is both an `any`, and it carries around a bundle of type-erasure function pointers that change whenever the contained `any` does:

> 这是扩充的“any”。它既是一个“any”，它携带着一束类型擦除函数指针，每当包含的“any”发生变化时，这些指针就会发生变化：

```cpp
template<class... methods>
struct super_any_t:boost::any, any_methods<methods...> {
  using vtable=any_methods<methods...>;
public:
  template<class T,
    std::enable_if_t< !std::is_base_of<super_any_t, std::decay_t<T>>{}, int> =0
  >
  super_any_t( T&& t ):
    boost::any( std::forward<T>(t) )
  {
    using dT=std::decay_t<T>;
    this->change_type( tag<dT> );
  }
  
  boost::any& as_any()&{return *this;}
  boost::any&& as_any()&&{return std::move(*this);}
  boost::any const& as_any()const&{return *this;}
  super_any_t()=default;
  super_any_t(super_any_t&& o):
    boost::any( std::move( o.as_any() ) ),
    vtable(o)
  {}
  super_any_t(super_any_t const& o):
    boost::any( o.as_any() ),
    vtable(o)
  {}
  template<class S,
    std::enable_if_t< std::is_same<std::decay_t<S>, super_any_t>{}, int> =0
  >
  super_any_t( S&& o ):
    boost::any( std::forward<S>(o).as_any() ),
    vtable(o)
  {}
  super_any_t& operator=(super_any_t&&)=default;
  super_any_t& operator=(super_any_t const&)=default;
  
  template<class T,
    std::enable_if_t< !std::is_same<std::decay_t<T>, super_any_t>{}, int>* =nullptr
  >
  super_any_t& operator=( T&& t ) {
    ((boost::any&)*this) = std::forward<T>(t);
    using dT=std::decay_t<T>;
    this->change_type( tag<dT> );
    return *this;
  }  
};

```


Because we store the `any_method`s as `const` objects, this makes making a `super_any` a bit easier:

> 因为我们将“any_method”存储为“const”对象，所以这使“super_any”变得更容易：

```cpp
template<class...Ts>
using super_any = super_any_t< std::remove_cv_t<Ts>... >;

```

Test code:

```cpp
const auto print = make_any_method<void(std::ostream&)>([](auto&& p, std::ostream& t){ t << p << "\n"; });
const auto wprint = make_any_method<void(std::wostream&)>([](auto&& p, std::wostream& os ){ os << p << L"\n"; });

int main()
{
  super_any<decltype(print), decltype(wprint)> a = 7;
  super_any<decltype(print), decltype(wprint)> a2 = 7;

  (a->*print)(std::cout);
  (a->*wprint)(std::wcout);
}

```

[live example](http://coliru.stacked-crooked.com/a/fbd10edb3336cce1).


Originally posted [here](http://stackoverflow.com/a/38837687/1774667) in a SO self question & answer (and people noted above helped with the implementation).

> 最初发布于[此处](http://stackoverflow.com/a/38837687/1774667)在SO自我问答中（上面提到的人帮助实现了）。

