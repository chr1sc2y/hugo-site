---
title: "A Simple Smart Pointer Implementation in C++"
date: 2021-02-21T21:20:18+08:00
draft: false
categories: ["C++ Pointer"]
description: "A translated technical note on A Simple Smart Pointer Implementation in C++, preserving the examples and context of the original article."
---
# A Simple Smart Pointer Implementation in C++

> Originally published in Chinese on 2021-02-21; this English edition preserves the original scope and technical context.

## 1 std::auto_ptr

In C++, memory leaks often occur due to the lack of `delete` for pointers, such as with an `Object` template class class:

cpp
template <typename T>
class Object {
public:
    ~Object() {
        delete this->ptr;
    }

    void* ptr;
};

```cpp
template<typename T>
class Object
{
public:
    // constructor
    Object() : t_() { cout << "Object::Constructor " << this << endl; }
    Object(T t) : t_(t) { cout << "Object::Constructor " << this << endl; }

    // copy-ctor
    Object(const Object &other) { cout << "Object::Copy-ctor " << this << endl; }

    // destructor
    ~Object() { cout << "Object::Destructor " << this << endl; }

    void Set(T t) { t_ = t; }

    void Print() { cout << t_ << endl; }

private:
    T t_;
};
```
If objects of a class are allocated on the heap and not deallocated when their scope is exited, a memory leak occurs:

c
#include <iostream>
#include <memory>

class MyClass {
public:
    MyClass() {
        std::cout << "MyClass constructed" << std.allocator<MyClass>::allocate();
    }
    ~MyClass() {
        std::cout << "MyClass destructed" << std.allocator<MyClass>::deallocate();
    }
};

void test() {
    std::unique_ptr<MyClass> ptr = std::make_unique<MyClass>();
    // ptr is automatically deallocated when it goes out of scope
}

int main() {
    test();
}


Output:

MyClass constructed
MyClass destructed


In this example, `std::unique_ptr` ensures that the `MyClass` object is automatically deallocated when it goes out of scope, preventing a memory leak.
```cpp
void AutoPointerFoo()
{
    Object<int>* o = new Object<int>(1);
    o->Print();
}
```


```shell
$ ./bin/smart-pointer
Object::Constructor 0x7b7058 # o
1
```
To address this issue, C++98 added the most primitive smart pointer `std::auto_ptr` in the standard, which provides automatic memory management using RAII mechanisms. It manages heap memory by utilizing stack objects, automatically releasing the managed heap variable in its destructor when the smart pointer object leaves its scope. It can help reduce the occurrence of memory leaks to some extent. Here is a simplified version of the `AutoPointer` class, based on the `std::auto_ptr` implementation GCC implementation, with some additions to track the resource allocation process:

cpp
template <typename T>
class AutoPointer {
public:
    AutoPointer(T* ptr) : ptr_(ptr) {
        std::cout << "AutoPointer constructed with: " << ptr_ << std::endl;
    }

    ~AutoPointer() {
        std::cout << "AutoPointer destructed with: " << ptr_ << std::endl;
        delete ptr_;
    }

    T& operator*() const { return *ptr_; }
    T* operator->() const { return ptr_; }

private:
    T* ptr_;
};





To address this issue, C++98 added the most primitive smart pointer `std::auto_ptr` in the standard, which provides automatic memory management using RAII mechanisms. It manages heap memory by utilizing stack objects, automatically releasing the managed heap variable in its destructor when the smart pointer object leaves its scope. It can help reduce the occurrence of memory leaks to some extent. Here is a simplified version of the `AutoPointer` class, based on the `std::auto_ptr` implementation GCC implementation, with some additions to track the resource allocation process:

cpp
template <typename T>
class AutoPointer {
public:
    AutoPointer(T* ptr) : ptr_(ptr) {
        std::cout << "AutoPointer constructed with: " << ptr_ << std::endl;
    }

    ~AutoPointer() {
        std::cout << "AutoPointer destructed with: " << ptr_ << std::endl;
        delete ptr_;
    }

    T& operator*() const { return *ptr_; }
    T* operator->() const { return ptr_; }

private:
    T* ptr_;
};



```cpp
template<typename T>
class AutoPointer
{
public:
    // constructor
    explicit AutoPointer(T* t = nullptr) noexcept : ptr_(t) { std::cout << "AutoPointer::Constructor " << this << std::endl; }

    // copy-ctor
    AutoPointer(AutoPointer<T>& other) noexcept : ptr_(other.Release()) { std::cout << "AutoPointer::Copyctor " << this << std::endl; }

    // assignment operator
    AutoPointer<T>& operator=(AutoPointer<T>& other)
    {
        std::cout << "AutoPointer::Assignment " << this << std::endl;
        Reset(other.Release());
        return *this;
    }

    // destructor
    ~AutoPointer() noexcept
    {
        std::cout << "AutoPointer::Destructor " << this << std::endl;
        delete ptr_;
    }

    T& operator*() noexcept { return *ptr_; }

    T* operator->() const noexcept { return ptr_; }

    T* Get() const noexcept { return ptr_; }

    T* Release() noexcept
    {
        T* ptr_ret = ptr_;
        ptr_ = nullptr;
        return ptr_ret;
    }

    void Reset(T* ptr_para) noexcept
    {
        if (ptr_ != ptr_para)
        {
            delete ptr_;
            ptr_ = ptr_para;
        }
    }

private:
    T *ptr_;
};
```
At initialization, we need to manually allocate an object on the heap and pass it as a parameter; subsequently, we can use the smart pointer object as a regular pointer, without worrying about its lifetime, and use it with methods that are applicable to regular pointers.
```cpp
void AutoPointerFoo()
{
    Object<int>* o = new Object<int>(1);
    AutoPointer<Object<int>> a(o);
    (*o).Set(2);
    (*o).Print();
    o->Set(3);
    o->Print();
}
```


```shell
$ ./bin/smart-pointer
Object::Constructor 0x48f058 # o
AutoPointer::Constructor 0xbee47668 # a
2
3
AutoPointer::Destructor 0xbee47668 # a
Object::Destructor 0x48f058 # o
```
The two most important functions in the class are `Release` and `Reset`. The former releases the pointer managed by the current object and returns it, while the latter releases the pointer managed by the current object and sets the passed pointer to to a new managed object. Together, they implement the copy constructor and assignment operator. However, the existence of these functions brings the first issue: during the copy construction or assignment, the `AutoPointer` object being operated might lose its ownership of the managed object unintentionally, potentially leading to a `segmentation fault`.
```cpp
void Foo()
{
    AutoPointer<Object<int>> p1(new Object<int>(6));
    AutoPointer<Object<int>> p2(p1);
    cout << "p2: "; p2->Print();
    cout << "p1: "; p1->Print();
}
```


```shell
$ ./bin/smart-pointer
Object::Constructor 0x1cbd058 # o
AutoPointer::Constructor 0xbed23668 # a1
AutoPointer::Copyctor 0xbed23664 # a2
a2: 1
Segmentation fault
```
Second problem is that `AutoPointer` by default only uses `delete` to delete objects. If an `AutoPointer` manages an array, a memory leak can occur when the object goes out of scope. AddressSanitizer can detect this.
```cpp
void AutoPointerFoo()
{
    int *a = new int[1000000];
    AutoPointer<int> p(a);
}
```


```shell
$ ./bin/smart-pointer
AutoPointer::Constructor 0xbe9955e0 # new[]
AutoPointer::Destructor 0xbe9955e0 # delete
=================================================================
==2543==ERROR: AddressSanitizer: alloc-dealloc-mismatch (operator new [] vs operator delete) on 0xb412e800
# ...
```
Otherwise, if the same `Object` pointer is used to initialize multiple `AutoPointer` objects, the `Object` will be deleted multiple times at runtime, causing a `double free` error.
```cpp
void AutoPointerFoo()
{
    Object<int>* o = new Object<int>(1);
    AutoPointer<Object<int>> a1(o);
    AutoPointer<Object<int>> a2(o);
}
```


```shell
$ ./bin/smart-pointer
Object::Constructor 0x9c3058 # o
AutoPointer::Constructor 0xbee80668 # a1
AutoPointer::Constructor 0xbee80664 # a2
AutoPointer::Destructor 0xbee80664 # a2
Object::Destructor 0x9c3058 # o
AutoPointer::Destructor 0xbee80668 # a1
Object::Destructor 0x9c3058 # o
free(): double free detected in tcache 2
Aborted
```
## 2 unique_ptr

To address the issues with `std::auto_ptr`, C++11 drew inspiration from the design of `boost::unique_ptr`, and introduced `std::unique_ptr` in the standard library. Below is a simplified template class `UniquePointer` that references its implementation:
```cpp
template<typename ElementType, typename DeleterType = DefaultDeleter>
class UniquePointer
{
public:
    // constructors
    UniquePointer() noexcept : ptr_(nullptr) { std::cout << "UniquePointer::Constructor " << this << std::endl; }
    explicit UniquePointer(ElementType* p) noexcept : ptr_(p) { std::cout << "UniquePointer::Constructor " << this << std::endl; }
    UniquePointer(ElementType* p, DeleterType d) noexcept : ptr_(p), deleter_(d) { std::cout << "UniquePointer::Constructor " << this << std::endl; }

    // move-ctor
    UniquePointer(UniquePointer<ElementType, DeleterType>&& other) noexcept : ptr_(other.Release()), deleter_(std::move(other.deleter_)) { std::cout << "UniquePointer::Move-ctor " << this << std::endl; }
    // move assignment operator
    UniquePointer<ElementType, DeleterType>& operator=(UniquePointer<ElementType, DeleterType>&& other) noexcept
    {
        std::cout << "UniquePointer::MoveAssignment " << this << std::endl;
        ptr_ = other.Release();
        deleter_ = std::move(other.deleter_);
        return *this;
    }

    // copy-ctor
    UniquePointer(UniquePointer<ElementType, DeleterType>& other) noexcept = delete;
    // assignment operator
    UniquePointer<ElementType, DeleterType>& operator=(UniquePointer<ElementType, DeleterType>& other) = delete;

    // destructor
    ~UniquePointer() noexcept
    {
        std::cout << "UniquePointer::Destructor " << this << std::endl;
        if (ptr_)
        {
            GetDeleter()(ptr_);
            ptr_ = nullptr;
        }
    }

    ElementType& operator*() noexcept { return *ptr_; }

    ElementType* operator->() const noexcept { return ptr_; }

    ElementType* Get() const noexcept { return ptr_; }

    const DeleterType& GetDeleter() const noexcept { return deleter_; }

    ElementType* Release() noexcept
    {
        ElementType* ret = nullptr;
        std::swap(ptr_, ret);
        return ret;
    }

    void Reset(ElementType* p) noexcept
    {
        if (ptr_ != p)
        {
            delete ptr_;
            ptr_ = p;
        }
    }

private:
    ElementType* ptr_;
    DeleterType deleter_;
};
```
Compared to `AutoPointer`, the changes made by `UniquePointer` include two points: the first is that `UniquePointer` holds exclusive ownership of the pointer managed by it, preventing ownership transfer through disabling the copy constructor and assignment operator.
```cpp
void UniquePointerFoo()
{
    Object<int>* o = new Object<int>(1);
    UniquePointer<Object<int>> u1(o);
    UniquePointer<Object<int>> u2{ u1 }; // error: use of deleted function ‘UniquePointer<ElementType, DeleterType>::UniquePointer(UniquePointer<ElementType, DeleterType>&) [with ElementType = Object<int>; DeleterType = DefaultDeleter]’
}
```
**Simultaneously, we have added move constructors and move assignment operators, allowing us to explicitly transfer pointers in specific cases via move semantics:**

1. **Move Constructors:**
   - `MyClass(MyClass&& other)` noexcept;

2. **Move Assignment Operators:**
   - `MyClass& MyClass::operator=(MyClass&& other)` noexcept;

3. **Move Constructor in Copy Constructor:**
   - `MyClass::MyClass(const MyClass& other)` : `MyClass(other)` {}

4. **Move Constructor in Move Constructor:**
   - `MyClass::MyClass(MyClass&& other)` : `MyClass(std::move(other))` {}

5. **Move Constructor in Copy Constructor:**
   - `MyClass::MyClass(const MyClass& other)` : `MyClass(other)` {}

6. **Move Constructor in Move Constructor:**
   - `MyClass::MyClass(MyClass&& other)` : `MyClass(std::move(other))` {}

7. **Move Constructor in Copy Constructor:**
   - `MyClass::MyClass(const MyClass& other)` : `MyClass(other)` {}

8. **Move Constructor in Move Constructor:**
   - `MyClass::MyClass(MyClass&& other)` : `MyClass(std::move(other))` {}

9. **Move Constructor in Copy Constructor:**
   - `MyClass::MyClass(const MyClass& other)` : `MyClass
```cpp
void UniquePointerFoo()
{
    Object<int>* o = new Object<int>(1);
    UniquePointer<Object<int>> u1(o);
    UniquePointer<Object<int>> u2(std::move(u1));
    UniquePointer<Object<int>> u3;
    u3 = std::move(u2);
}
```

```shell
$ ./bin/smart-pointer
Object::Constructor 0x3af058				# o
UniquePointer::Constructor 0xbec29664		# u1
UniquePointer::Move-ctor 0xbec2965c			# u2
UniquePointer::Constructor 0xbec29654		# u3
UniquePointer::MoveAssignment 0xbec29654	# u3
UniquePointer::Destructor 0xbec29654		# u3
Object::Destructor 0x3af058					# o
UniquePointer::Destructor 0xbec2965c		# u2
UniquePointer::Destructor 0xbec29664		# u1
```
Second, a custom deleter is added in the template parameters. A deleter is a functor, and we can define the behavior of the `operator()` in its `operator()` to perform actions on the pointer managed by `UniquePointer` at destruction, such as using `delete[]` to release memory or closing related sockets, etc.
```cpp
struct ArrayDeleter
{
    template<typename T>
    void operator()(T* p) const
    {
        static_assert(sizeof(p) > 0, "can't delete pointer to incomplete type");
        delete[] p;
    }
};

void UniquePointerFoo()
{
    int* int_arr = new int[1000000];
    ArrayDeleter array_deleter;
    UniquePointer<int, ArrayDeleter> u(int_arr, array_deleter);
}
```
```shell
$ ./bin/smart-pointer
UniquePointer::Constructor 0xbe9ad660
UniquePointer::Destructor 0xbe9ad660
```
## 3 shared_ptr

`std::shared_ptr` is utilized when multiple smart pointers need to share the same pointer, and it ensures that the pointer is destroyed when all smart pointers owning it are out of scope. It uses a reference counter to track how many smart pointers are currently sharing the pointer. When this reference counter reaches 0, it indicates that no smart pointers hold the pointer, at which point the resources are destroyed.
```cpp
// A smart pointer with reference-counted copy semantics.  The
// object pointed to is deleted when the last shared_ptr pointing to
// it is destroyed or reset.
template<typename _Tp, _Lock_policy _Lp> class __shared_ptr
// ...
```
Pointers and references managed by `std::shared_ptr`, along with their reference counters, are stored on the heap. If two smart pointers hold the same resource in different threads simultaneously, it may lead to thread safety issues. Therefore, we need to use certain mechanisms to prevent such problems. Typically, increments and decrements of the reference counter are atomic, but access to shared resources requires mechanisms like mutexes to ensure thread safety.

Here is the implementation of the `SharedPointer` class, analogous to `std::shared_ptr`:
```cpp
template<typename ElementType, typename DeleterType = DefaultDeleter>
class SharedPointer
{
public:
    // constructors
    SharedPointer() noexcept : ptr_(nullptr), ref_count_(nullptr), deleter_(nullptr), mutex_(nullptr) { std::cout << "SharedPointer::Constructor " << this << std::endl; }
    explicit SharedPointer(ElementType* p) noexcept : ptr_(p), ref_count_(new int(1)), deleter_(new DeleterType()), mutex_(new mutex()) { std::cout << "SharedPointer::Constructor " << this << std::endl; }
    SharedPointer(ElementType* p, DeleterType *d) noexcept : ptr_(p), ref_count_(new int(1)), deleter_(d), mutex_(new mutex()) { std::cout << "SharedPointer::Constructor " << this << std::endl; }
    explicit SharedPointer(const WeakPointer<ElementType, DeleterType>& wp) noexcept : ptr_(wp.ptr_), ref_count_(wp.ref_count_), deleter_(wp.deleter_), mutex_(wp.mutex_)
    {
        std::cout << "SharedPointer::Constructor " << this << std::endl;
        IncreaseReferenceCount();
    }

    // copy-ctor
    SharedPointer(const SharedPointer<ElementType>& other) noexcept : ptr_(other.ptr_), ref_count_(other.ref_count_), deleter_(other.deleter_), mutex_(other.mutex_)
    {
        std::cout << "SharedPointer::Copy-ctor " << this << std::endl;
        IncreaseReferenceCount();
    }

    // assignment operator
    SharedPointer& operator=(SharedPointer<ElementType>& other)
    {
        if (ptr_ != other.ptr_)
        {
            Release();
            ptr_ = other.ptr_;
            ref_count_ = other.ref_count_;
            mutex_ = other.mutex_;
            deleter_ = other.deleter_;
            IncreaseReferenceCount();
        }
        return *this;
    }

    // destructor
    ~SharedPointer() noexcept
    {
        std::cout << "SharedPointer::Destructor " << this << std::endl;
        Release();
    }

    void Swap(SharedPointer<ElementType, DeleterType>& other)
    {
        std::swap(ptr_, other.ptr_);
        std::swap(ref_count_, other.ref_count_);
        std::swap(deleter_, other.deleter_);
        std::swap(mutex_, other.mutex_);
    }

    void Reset() { SharedPointer().Swap(*this); }
    void Reset(ElementType* p, DeleterType* d = nullptr) { SharedPointer(p, d).Swap(*this); }

    int UseCount() { return ref_count_ ? *ref_count_ : 0; }

    ElementType& operator*() noexcept { return *ptr_; }

    ElementType* operator->() const noexcept { return ptr_; }

    ElementType* Get() const noexcept { return ptr_; }

    const DeleterType& GetDeleter() const noexcept { return *deleter_; }

    void Release()
    {
        if (!ptr_)
            return;
        bool delete_flag = false;
        mutex_->lock();
        if (--(*ref_count_) == 0)
        {
            GetDeleter()(ptr_);
            delete ref_count_;
            delete deleter_;
            delete_flag = true;
        }
        mutex_->unlock();

        if (delete_flag)
        {
            delete mutex_;
        }

    }

	void IncreaseReferenceCount()
	{
        if (!ptr_)
            return;
		mutex_->lock();
        ++(*ref_count_);
		mutex_->unlock();
	}

    ElementType* ptr_;
    int *ref_count_;
    DeleterType* deleter_;
	mutex* mutex_;
};
```
Now it is possible to have a pointer held by multiple smart pointers:
```cpp
void SharedPointerFoo()
{
    Object<int>* o = new Object<int>(1);
    SharedPointer<Object<int>> s1(o);
    SharedPointer<Object<int>> s2(s1);
    s1.Reset(nullptr);
    s2.Reset(nullptr);
}
```
```shell
$ ./bin/smart-pointer
Object::Constructor 0x1ac7058 # o1
SharedPointer::Constructor 0xbe89c65c # p1
SharedPointer::Copy-ctor 0xbe89c64c # p2
SharedPointer::Constructor 0xbe89c630 # p1.Reset
SharedPointer::Destructor 0xbe89c630 # p1.Reset
SharedPointer::Constructor 0xbe89c630 # p2.Reset
SharedPointer::Destructor 0xbe89c630 # p2.Reset
Object::Destructor 0x1ac7058 # o1
SharedPointer::Destructor 0xbe89c64c # p2
SharedPointer::Destructor 0xbe89c65c # p1
```
But circular references also arise, such as:
```cpp
void SharedPointerFoo()
{
    struct Node
    {
        int i_;
        SharedPointer<Node> prev_;
        SharedPointer<Node> next_;

        Node(int i) : i_(i) { std::cout << "Node::Constructor " << this << std::endl; }
        ~Node() { std::cout << "Node::Destructor " << this << std::endl; }
    };

    Node *n1 = new Node(1), *n2 = new Node(2);
    SharedPointer<Node> s1(n1), s2(n2);
    cout << s1.UseCount() << ' ' << s2.UseCount() << endl;
    s1->next_ = s2;
    s2->prev_ = s1;
    cout << s1.UseCount() << ' ' << s2.UseCount() << endl;
}
```
```shell
$ ./bin/smart-pointer
SharedPointer::Constructor 0x3f905c # n1->prev_
SharedPointer::Constructor 0x3f906c # n1->next_
Node::Constructor 0x3f9058 # n1
SharedPointer::Constructor 0x3f948c # n2->prev_
SharedPointer::Constructor 0x3f949c # n2->next_
Node::Constructor 0x3f9488 # n2
SharedPointer::Constructor 0xbeca0658 # s1
SharedPointer::Constructor 0xbeca0648 # s2
1 1
2 2
SharedPointer::Destructor 0xbeca0648 # s2
SharedPointer::Destructor 0xbeca0658 # s1
```
Here, although the `s1` and `s2` `SharedPointer<Node>` are successfully destroyed at the end of the function, the objects `n1` and `n2` they hold are not destroyed, as neither `s1` nor `s2` has a reference count decremented to zero. The reference counts can only be decremented to zero when `s1->next_` no longer points to `s2`, and `s2->prev_` no longer points to `s1`.

## 4 weak_ptr

`std::weak_ptr` is a weak reference smart pointer, the only difference from `std::shared_ptr` is that it must be explicitly constructed from either `std::shared_ptr` or `std::weak_ptr`, and cannot be constructed from a newly created object. Therefore, the resources it manages are actually held by another `std::shared_ptr`, and it merely provides access to the managed resources without affecting the lifecycle of the managed resources, i.e., it does not modify the reference count of the `std::shared_ptr`.

The following is an implementation of the `WeakPointer` class, based on `std::weak_ptr`:
```cpp
template<typename ElementType, typename DeleterType> class SharedPointer;

template<typename ElementType, typename DeleterType = DefaultDeleter>
class WeakPointer
{
public:
    // constructors
    WeakPointer() noexcept : ptr_(nullptr), ref_count_(nullptr), deleter_(nullptr), mutex_(nullptr) { std::cout << "WeakPointer::Constructor " << this << std::endl; }
    explicit WeakPointer(SharedPointer<ElementType, DeleterType>& sp) noexcept : ptr_(sp.ptr_), ref_count_(sp.ref_count_), deleter_(sp.deleter_), mutex_(sp.mutex_) { std::cout << "WeakPointer::Constructor " << this << std::endl; }

    // copy-ctor
    WeakPointer(WeakPointer<ElementType>& other) noexcept : ptr_(other.ptr_), ref_count_(other.ref_count_), deleter_(other.deleter_), mutex_(other.mutex_)
    {
        std::cout << "WeakPointer::Copy-ctor " << this << std::endl;
    }

    // assignment operator
    WeakPointer& operator=(SharedPointer<ElementType, DeleterType>& sp)
    {
        ptr_ = sp.ptr_;
        ref_count_ = sp.ref_count_;
        mutex_ = sp.mutex_;
        deleter_ = sp.deleter_;
        return *this;
    }

    // destructor
    ~WeakPointer() noexcept { std::cout << "WeakPointer::Destructor " << this << std::endl; }

    void Swap(WeakPointer& other)
    {
        std::swap(ptr_, other.ptr_);
        std::swap(ref_count_, other.ref_count_);
        std::swap(deleter_, other.deleter_);
        std::swap(mutex_, other.mutex_);
    }

    void Reset() { WeakPointer().Swap(*this); }

    SharedPointer<ElementType, DeleterType> Lock() const
    {
        return Expired() ? SharedPointer<ElementType, DeleterType>()
                         : SharedPointer<ElementType, DeleterType>(*this);
    }

    int UseCount() { return *ref_count_; }

    const DeleterType& GetDeleter() const noexcept { return *deleter_; }

    bool Expired() const { return *ref_count_ == 0; }

    ElementType* ptr_;
    int *ref_count_;
    DeleterType* deleter_;
	mutex* mutex_;
};
```
`std::weak_ptr` cannot control the lifecycle of the managed resource, so we need to check whether the managed resource exists before using it. We can achieve this by leveraging `std::weak_ptr::lock` to obtain a new `std::shared_ptr` object to safely access the resource.
```cpp
void WeakPointerFoo()
{
    Object<string> *o = new Object<string>("test");
    SharedPointer<Object<string>> s(o);
    WeakPointer<Object<string>> w(s);
    s.Reset();

    auto p = w.Lock().Get();
    cout << w.Expired() << endl;
    cout << static_cast<void*>(p) << endl;
}
```
```shell
$ ./bin/smart-pointer
Object::Constructor 0xf8c058 # o
SharedPointer::Constructor 0xbe97a630 # s
WeakPointer::Constructor 0xbe97a620 # w
SharedPointer::Constructor 0xbe97a608 # temporary variable in s.Reset()
SharedPointer::Destructor 0xbe97a608 # temporary variable in s.Reset()
Object::Destructor 0xf8c058 # o
SharedPointer::Constructor 0xbe97a65c # temporary variable in w.Lock()
SharedPointer::Destructor 0xbe97a65c # temporary variable in w.Lock()
1 # w.Expired()
0 # p
WeakPointer::Destructor 0xbe97a620 # w
SharedPointer::Destructor 0xbe97a630 # s
```
If the `s.Reset()` above is removed, then `w.Lock()` returns a `SharedPointer` containing an `Object<string>`. The `*o` will not be destroyed after `s.Reset()`:

cpp
// Example of usage
{
    SharedPointer<Object<string>> o = w.Lock();
    // Use o...
}

// If s.Reset() is removed
{
    SharedPointer<Object<string>> o = w.Lock();
    // Use o...
}

```cpp
void WeakPointerFoo()
{
    Object<string> *o = new Object<string>("test");
    SharedPointer<Object<string>> s(o);
    WeakPointer<Object<string>> w(s);
    // s.Reset();

    auto p = w.Lock().Get();
    cout << w.Expired() << endl;
    cout << static_cast<void*>(p) << endl;
}
```
```shell
$ ./bin/smart-pointer
Object::Constructor 0xb5c058 # o
SharedPointer::Constructor 0xbef4562c # s
WeakPointer::Constructor 0xbef4561c # w
SharedPointer::Constructor 0xbef45658 # temporary variable in s.Reset()
SharedPointer::Destructor 0xbef45658 # temporary variable in s.Reset()
0 # w.Expired()
0xb5c058 # p
WeakPointer::Destructor 0xbef4561c # temporary variable in w.Lock()
SharedPointer::Destructor 0xbef4562c # # temporary variable in w.Lock()
Object::Destructor 0xb5c058 # o
```
For issues with cyclic references, converting the smart pointers that mutually reference to `WeakPointer` can successfully avoid the problem of pointers failing to destruct normally:
```cpp
void WeakPointerFoo()
{
	struct Node
    {
        int i_;
        WeakPointer<Node> prev_;
        WeakPointer<Node> next_;

        Node(int i) : i_(i) { std::cout << "Node::Constructor " << this << std::endl; }
        ~Node() { std::cout << "Node::Destructor " << this << std::endl; }
    };

    Node *n1 = new Node(1), *n2 = new Node(2);
    SharedPointer<Node> p1(n1), p2(n2);
    cout << p1.UseCount() << ' ' << p2.UseCount() << endl;
    p1->next_ = p2;
    p2->prev_ = p1;
    cout << p1.UseCount() << ' ' << p2.UseCount() << endl;
}
```
```shell
$ ./bin/smart-pointer
WeakPointer::Constructor 0xb505c # n1->prev_
WeakPointer::Constructor 0xb506c # n1->next_
Node::Constructor 0xb5058 # n1
WeakPointer::Constructor 0xb548c # n2->prev_
WeakPointer::Constructor 0xb549c # n2->next_
Node::Constructor 0xb5488 # n2
SharedPointer::Constructor 0xbe904658 # s1
SharedPointer::Constructor 0xbe904648 # s2
1 1
1 1
SharedPointer::Destructor 0xbe904648 # s2
Node::Destructor 0xb5488 # n2
WeakPointer::Destructor 0xb549c # n2->next_
WeakPointer::Destructor 0xb548c # n2->prev_
SharedPointer::Destructor 0xbe904658 # s1
Node::Destructor 0xb5058 # n1
WeakPointer::Destructor 0xb506c # n1->next_
WeakPointer::Destructor 0xb505c # n1->prev_
```

## Original references

- [Reference 1](https://en.wikipedia.org/wiki/Smart_pointer)
- [Reference 2](https://en.wikipedia.org/wiki/Resource_acquisition_is_initialization)
- [Reference 3](https://www.learncpp.com/cpp-tutorial/circular-dependency-issues-with-stdshared_ptr-and-stdweak_ptr/)
