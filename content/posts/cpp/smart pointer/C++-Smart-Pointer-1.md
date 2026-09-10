---
title: "C++ Smart Pointers (1): auto_ptr"
date: 2018-12-27T15:21:35+11:00
draft: false
categories: ["C++"]
description: "A translated technical note on C++ Smart Pointers (1): auto_ptr, preserving the examples and context of the original article."
---
# C++ Smart Pointers (1): auto_ptr

> Originally published in Chinese on 2018-12-27; this English edition preserves the original scope and technical context.

## Analysis

In C++, memory leaks often occur due to the lack of a delete statement, such as with an `Object` class.
```c++
class Object {
public:
    Object() { std::cout << "Construct" << std::endl; }

    Object(const Object &other) { std::cout << "Copy" << std::endl; }

    Object(Object &&other) noexcept { std::cout << "Move" << std::endl; }

    ~Object() { std::cout << "Destruct" << std::endl; }

    void Print() { std::cout << "Print" << std::endl; }
};
```
Creating a Pointer to a Pointer of Type `Object`
```c++
int main() {
    Object *o = new Object();
    o->Print();
    return 0;
}
/*
output:
Construct
Print
*/
```
We did not perform `delete o`, resulting in `o` not being properly destroyed and causing a memory leak. For comparison, let's create an object of type `Obj`.
```c++
int main() {
    Object *o1 = new Object();
    o1->Print();
    Object o2 = Object();
    o2.Print();
    return 0;
}
/*
output:
Construct
Print
Construct
Print
Destruct
*/
```
## Implementation

According to the source code of `auto_ptr`, an `AutoPointer` class can be roughly implemented.
```c++
template<typename T>
class AutoPointer {
public:
    explicit AutoPointer(T *t);

    ~AutoPointer();

    T &operator*();

    T *operator->();

    T *release();

    void reset(T *p);

    AutoPointer(AutoPointer<T> &other);

    AutoPointer<T> &operator=(AutoPointer<T> const &other);

private:
    T *pointer;
};

template<typename T>
AutoPointer<T>::AutoPointer(T *t) {
    std::cout << "AutoPointer " << this << " constructor called." << std::endl;
    this->pointer = t;
}

template<typename T>
AutoPointer<T>::~AutoPointer() {
    std::cout << "AutoPointer " << this << " destructor called." << std::endl;
    delete this->pointer;
}

template<typename T>
T &AutoPointer<T>::operator*() {
    return *this->pointer;
}

template<typename T>
T *AutoPointer<T>::operator->() {
    return this->pointer;
}

template<typename T>
T *AutoPointer<T>::release() {
    T *new_pointer = this->pointer;
    this->pointer = nullptr;
    return new_pointer;
}

template<typename T>
void AutoPointer<T>::reset(T *p) {
    if (this->pointer != p) {
        delete this->pointer;
        this->pointer = p;
    }
}

template<typename T>
AutoPointer<T>::AutoPointer(AutoPointer<T> &other) {
    std::cout << "AutoPointer " << this << " copy constructor called." << std::endl;
    this->pointer = other.release();
}

template<typename T>
AutoPointer<T> &AutoPointer<T>::operator=(AutoPointer<T> const &other) {
    std::cout << "AutoPointer " << this << " assignment operator called." << std::endl;
    if (this->pointer != other.pointer)
        this->reset(other.release());
    return *this;
}
```
- The constructor directly points the AutoPointer class's pointer to the address pointed by the passed parameter pointer.
- The copy constructor first releases the pointer of the parameter object, which means setting the private member pointer pointer to `nullptr` and returninging the address it originally points to. Then, it points its own pointer to this address.
- The assignment operator first checks if the passed parameter is the same object itself. If so, it returns the `this` pointer. Otherwise, it first releases the pointer of the parameter object and deletes the `pointer` of the current object, then points its own pointer to the address that the parameter object's pointer originally points to. This implementation effectively avoids the [dangling pointer](https://zh.wikipedia.org/wiki/%E8%BF%B7%E9%80%94%E6%8C%87%E9%92%88) (also known as a dangling pointer or a wild pointer).

## Testing

A single instance of the AutoPointer class object works properly.
```c++
int main() {
    Object *o = new Object();
    AutoPointer<Object> a1(o);
    (*a1).Print();
    a1->Print();
    return 0;
}
/*
output:
Construct
AutoPointer 0x7fe680c02ab0 constructor called.
Print
Print
AutoPointer 0x7fe680c02ab0 destructor called.
Destruct
*/
```
Creating two `AutoPointer` class objects by initializing them with the same `Object` pointer results in the `Object` being destructed twice when `AutoPointer` objects are destroyed. In other words, the same address is deleted twice, leading to a runtime error.
```c++
int main() {
    Object *o = new Object();
    AutoPointer<Object> a1(o);
    AutoPointer<Object> a2(o);
    return 0;
}
/*
output:
Construct
AutoPointer 0x7ffee3fa0178 constructor called.
AutoPointer 0x7ffee3fa0170 constructor called.
AutoPointer 0x7ffee3fa0170 destructor called.
Destruct
AutoPointer 0x7ffee3fa0178 destructor called.
Destruct
cpp(9015,0x1197a25c0) malloc: *** error for object 0x7fe9dec02b40: pointer being freed was not allocated
cpp(9015,0x1197a25c0) malloc: *** set a breakpoint in malloc_error_break to debug
*/
```
Using the copy constructor to copy the `AutoPointer` object `a1` to another `AutoPointer` object `a2`, the pointer `o` originally belonged to `a1` becomes a null pointer after `a2` invokes the copy constructor. `a1` no longer owns `o`, and `s2` now owns the pointer, resulting in ownership transfer.
```c++
int main() {
    Object *o = new Object();
    AutoPointer<Object> a1(o);
    AutoPointer<Object> a2(a1);
    return 0;
}
/*
output:
Construct
AutoPointer 0x7fd15bc02ab0 constructor called.
AutoPointer 0x7fd15bc02ab0 copy constructor called.
AutoPointer 0x7fd15bc02ab0 destructor called.
Destruct
AutoPointer 0x0 destructor called.
*/
```
Using the assignment operator also involves ownership transfer issues.
```c++
int main() {
    Object *o = new Object();
    AutoPointer<Object> a1(o);
    AutoPointer<Object> a2 = a1;
    return 0;
}
/*
output:
Construct
AutoPointer 0x7ff5d5402ab0 constructor called.
AutoPointer 0x7ff5d5402ab0 copy constructor called.
AutoPointer 0x7ff5d5402ab0 destructor called.
Destruct
AutoPointer 0x0 destructor called.
*/
```
## Summary

AutoPointer effectively solves the wild pointer issue, but introduces some other problems such as.

1. Ownership Transfer
    - Copy constructing or assigning an `AutoPointer` as a parameter causes ownership transfer.

2. Memory Leaks
    - In the destructor, deleting a pointer with `delete` was used, but if an array pointer is initialized as `AutoPointer<int> s1(new int[10])`, it will cause a memory leak due to not properly deallocating the other elements of the array.

## auto_ptr Source Code
```c++
template<class _Tp>
class _LIBCPP_TEMPLATE_VIS auto_ptr
{
private:
    _Tp* __ptr_;
public:
    typedef _Tp element_type;

    _LIBCPP_INLINE_VISIBILITY explicit auto_ptr(_Tp* __p = 0) throw() : __ptr_(__p) {}
    _LIBCPP_INLINE_VISIBILITY auto_ptr(auto_ptr& __p) throw() : __ptr_(__p.release()) {}
    template<class _Up> _LIBCPP_INLINE_VISIBILITY auto_ptr(auto_ptr<_Up>& __p) throw()
        : __ptr_(__p.release()) {}
    _LIBCPP_INLINE_VISIBILITY auto_ptr& operator=(auto_ptr& __p) throw()
        {reset(__p.release()); return *this;}
    template<class _Up> _LIBCPP_INLINE_VISIBILITY auto_ptr& operator=(auto_ptr<_Up>& __p) throw()
        {reset(__p.release()); return *this;}
    _LIBCPP_INLINE_VISIBILITY auto_ptr& operator=(auto_ptr_ref<_Tp> __p) throw()
        {reset(__p.__ptr_); return *this;}
    _LIBCPP_INLINE_VISIBILITY ~auto_ptr() throw() {delete __ptr_;}

    _LIBCPP_INLINE_VISIBILITY _Tp& operator*() const throw()
        {return *__ptr_;}
    _LIBCPP_INLINE_VISIBILITY _Tp* operator->() const throw() {return __ptr_;}
    _LIBCPP_INLINE_VISIBILITY _Tp* get() const throw() {return __ptr_;}
    _LIBCPP_INLINE_VISIBILITY _Tp* release() throw()
    {
        _Tp* __t = __ptr_;
        __ptr_ = 0;
        return __t;
    }
    _LIBCPP_INLINE_VISIBILITY void reset(_Tp* __p = 0) throw()
    {
        if (__ptr_ != __p)
            delete __ptr_;
        __ptr_ = __p;
    }

    _LIBCPP_INLINE_VISIBILITY auto_ptr(auto_ptr_ref<_Tp> __p) throw() : __ptr_(__p.__ptr_) {}
    template<class _Up> _LIBCPP_INLINE_VISIBILITY operator auto_ptr_ref<_Up>() throw()
        {auto_ptr_ref<_Up> __t; __t.__ptr_ = release(); return __t;}
    template<class _Up> _LIBCPP_INLINE_VISIBILITY operator auto_ptr<_Up>() throw()
        {return auto_ptr<_Up>(release());}
};
```

## Original references

- [Reference 1](https://isocpp.org/blog/2015/09/stack-heap-pool-tony-bulldozer00-bd00-dasilva)
