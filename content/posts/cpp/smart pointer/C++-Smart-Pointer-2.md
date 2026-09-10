---
title: "C++ Smart Pointers (2): unique_ptr"
date: 2019-01-19T01:02:02+11:00
draft: false
categories: ["C++"]
description: "A translated technical note on C++ Smart Pointers (2): unique_ptr, preserving the examples and context of the original article."
---
# C++ Smart Pointers (2): unique_ptr

> Originally published in Chinese on 2019-01-19; this English edition preserves the original scope and technical context.

## Analysis

When using `AutoPointer`, there are issues with ownership transfer and memory leaks. Therefore, we can modify the `AutoPointer` class to fix these problems.

Ownership Transfer

To avoid potential ownership transfers, we can directly disallow the use of the copy constructor and assignment operator.
```c++
    UniquePointer(UniquePointer<T> &other) = delete;

    UniquePointer<T> &operator=(const UniquePointer<T> &other) = delete;
```
But often we need to use operations involving pointers. If we only use the `deleted` function to prohibit copy constructor and assignment operators, the meaning of this smart pointer becomes less significant. We can achieve move semantics through move semantics to implement move constructor and move assignment operators. This way, when using `UniquePointer`, we can transfer ownership under certain circumstances.
```c++
    UniquePointer(UniquePointer<T> &&other) noexcept;

    UniquePointer &operator=(UniquePointer &&other) noexcept;
```
### Memory Leaks

To prevent memory leaks, we can add a destructor to the private member of UniquePointer and specify the destructor based on the type of the current pointer object. This prevents memory leaks.
```c++
class Deleter {
    template<typename T>
    void operator()(T *p) {
        if (p)
            delete p;
    }
};

template<typename T, typename D>
class UniquePointer {
  ...
private:
    T *pointer;
    Deleter deleter;
};
```
## Implementation

According to the source code of `unique_ptr`, `UniquePointer` class can be roughly implemented.
```c++
template<typename T, typename D>
class UniquePointer {
public:
    explicit UniquePointer(T *t, const D &d);

    ~UniquePointer();

    T &operator*();

    T *operator->();

    T *release();

    void reset(T *p);

    UniquePointer(UniquePointer &&other) noexcept;

    UniquePointer &operator=(UniquePointer &&other) noexcept;

    UniquePointer(const UniquePointer &other) = delete;

    UniquePointer &operator=(const UniquePointer &other) = delete;

private:
    T *pointer;
    D deleter;
};

template<typename T, typename D>
UniquePointer<T, D>::UniquePointer(T *t, const D &d) {
    std::cout << "UniquePointer " << this << " constructor called." << std::endl;
    this->pointer = t;
    this->deleter = d;
}

template<typename T, typename D>
UniquePointer<T, D>::~UniquePointer() {
    std::cout << "UniquePointer " << this << " destructor called." << std::endl;
    deleter(this->pointer);
}

template<typename T, typename D>
T &UniquePointer<T, D>::operator*() {
    return *this->pointer;
}

template<typename T, typename D>
T *UniquePointer<T, D>::operator->() {
    return this->pointer;
}

template<typename T, typename D>
T *UniquePointer<T, D>::release() {
    T *new_pointer = this->pointer;
    this->pointer = nullptr;
    return new_pointer;
}

template<typename T, typename D>
void UniquePointer<T, D>::reset(T *p) {
    if (this->pointer != p) {
        deleter(this->pointer);
        this->pointer = p;
    }
}

template<typename T, typename D>
UniquePointer<T, D>::UniquePointer(UniquePointer<T, D> &&other) noexcept {
    std::cout << "UniquePointer " << this << " move constructor called." << std::endl;
    this->pointer = other.release();
    deleter(std::move(other.deleter));
}

template<typename T, typename D>
UniquePointer<T, D> &UniquePointer<T, D>::operator=(UniquePointer<T, D> &&other) noexcept {
    std::cout << "UniquePointer " << this << " assignment operator called." << std::endl;
    if (this->pointer != other.pointer) {
        reset(other.release());
        deleter = std::move(other.deleter);
    }
    return *this;
}
```
## Test

Try Using Move Constructors
```c++
class Deleter {
public:
    template<typename T>
    void operator()(T *p) {
        if (p)
            delete p;
    }
};
```
```c++
int main() {
    Deleter deleter;
    Obj *o = new Obj();
    UniquePointer<Obj, Deleter> u1(o, deleter);
    UniquePointer<Obj, Deleter> u2(move(u1));
    return 0;
}
/*
output:
Construct
UniquePointer 0x7ffee7dada08 constructor called.
UniquePointer 0x7ffee7dad9f8 move constructor called.
UniquePointer 0x7ffee7dad9f8 destructor called.
Destruct
UniquePointer 0x7ffee7dada08 destructor called.
*/
```
### Move Assignment Operator

| Function | Description |
| --- | --- |
| `=` | Ordinary assignment operator, copies the value on the right to the left. |
| `std::move` | Moves the resource to the left rather than copying. |

### Example Code

cpp
#include <iostream>
#include <string>

struct Point {
    int x, y;
    Point(int x, int y) : x(x), y(y) {}
};

struct Rectangle {
    Point top_left, bottom_right;
    Rectangle(Point tl, Point br) : top_left(tl), bottom_right(br) {}
    Rectangle() : top_left(0, 0), bottom_right(0, 0) {}
};

void print(const Rectangle& rect) {
    std::cout << "Rectangle: (" << rect.top_left.x << ", " << rect.top_left.y << ") - (" << rect.bottom_right.x << ", " << rect.bottom_right.y << ")" << std::endl;
}

int main() {
    Rectangle rect1(1, 1), rect2(2, 2);
  Rectangle rect3 = rect1; // Use assignment operator
    print(rect3); // Output: Rectangle: (1, 1) - (1, 1)

   Rectangle rect4 = std::move(rect1); // Use move assignment operator
    print(rect4); // Output: Rectangle: ()
```
class Deleter {
public:
    template<typename T>
    void operator()(T *p) {
        if (p)
            delete p;
    }
};

int main() {
    Deleter deleter;
    Obj *o = new Obj();
    UniquePointer<Obj, Deleter> u1(o, deleter);
    UniquePointer<Obj, Deleter> u2(nullptr, deleter);
    u2 = move(u1);
    return 0;
}
/*
output:
Construct
UniquePointer 0x7ffee915da08 constructor called.
UniquePointer 0x7ffee915d9f8 constructor called.
UniquePointer 0x7ffee915d9f8 assignment operator called.
UniquePointer 0x7ffee915d9f8 destructor called.
Destruct
UniquePointer 0x7ffee915da08 destructor called.
*/
```

#define ArrayDeleter [](UniquePointer<int[]> ptr) { delete[] ptr.get(); }

class UniquePointer {
public:
    template<typename T, typename Allocator>
    UniquePointer(T* ptr, Allocator& alloc) : ptr_(ptr), alloc_(alloc) {}

    template<typename T, typename Allocator>
    UniquePointer(UniquePointer&& other) noexcept : ptr_(other.ptr_), alloc_(other.alloc_) {
        other.ptr_ = nullptr;
        other.alloc_ = nullptr;
    }

    template<typename T, typename Allocator>
    UniquePointer& operator=(UniquePointer&& other) noexcept {
        if (this != &other) {
            this->~UniquePointer();
            this->ptr_ = other.ptr_;
            this->alloc_ = other.alloc_;
            other.ptr_ = nullptr;
            other.alloc_ = nullptr;
        }
        return *this;
    }

    template<typename T, typename Allocator>
    ~UniquePointer() {
        if (ptr_ != nullptr) {
            ArrayDeleter(*this);
        }
    }

    T* get() const { return ptr_; }

    T* operator->() const { return ptr_; }

    T& operator*() const { return *ptr_; }

private:
    T* ptr_;
    Allocator* alloc_;
};



// Example usage
#include <memory>
#include <vector>

int main() {
    std::vector<int> vec = {1, 2,
```
class ArrayDeleter {
public:
    template<typename T>
    void operator()(T *p) {
        if (p)
            delete[] p;
    }
};

int main() {
    ArrayDeleter array_deleter;
    Obj *o = new Obj[3];
    UniquePointer<Obj, ArrayDeleter> u(o, array_deleter);
    return 0;
}
/*
output:
Construct
Construct
Construct
UniquePointer 0x7ffeed926a08 constructor called.
UniquePointer 0x7ffeed926a08 destructor called.
Destruct
Destruct
Destruct
*/
```
As a contrast, if the default deleter is used as the deleter for array pointers.
```
class Deleter {
public:
    template<typename T>
    void operator()(T *p) {
        if (p)
            delete p;
    }
};

int main() {
    Deleter deleter;
    Obj *o = new Obj[3];
    UniquePointer<Obj, Deleter> u(o, deleter);
    return 0;
}
/*
output:
Construct
Construct
Construct
UniquePointer 0x7ffee8f85a10 constructor called.
UniquePointer 0x7ffee8f85a10 destructor called.
Destruct
*/
```
Explains that the Deleter can correctly address issues related to memory leaks. Attempts to have two `UniquePointer` objects point to the same pointer.
```
int main() {
    Deleter deleter;
    Obj *o = new Obj();
    UniquePointer<Obj, Deleter> u1(o, deleter);
    UniquePointer<Obj, Deleter> u2(o, deleter);
    return 0;
}
/*
output:
(19576,0x10e00a5c0) malloc: *** error for object 0x7fcfe8c02ab0: pointer being freed was not allocated
(19576,0x10e00a5c0) malloc: *** set a breakpoint in malloc_error_break to debug
Construct
UniquePointer 0x7ffee28a9a10 constructor called.
UniquePointer 0x7ffee28a9a00 constructor called.
UniquePointer 0x7ffee28a9a00 destructor called.
Destruct
UniquePointer 0x7ffee28a9a10 destructor called.
Destruct
*/
```
Still get the error of calling the destructor twice.

## Summary

UniquePointer successfully addresses issues of ownership transfer and memory leaks, but still suffers from the problem of double deallocation.


## unique_ptr source code
```
template <class _Tp, class _Dp = default_delete<_Tp> >
class _LIBCPP_TEMPLATE_VIS unique_ptr {
public:
  typedef _Tp element_type;
  typedef _Dp deleter_type;
  typedef typename __pointer_type<_Tp, deleter_type>::type pointer;

  static_assert(!is_rvalue_reference<deleter_type>::value,
                "the specified deleter type cannot be an rvalue reference");

private:
  __compressed_pair<pointer, deleter_type> __ptr_;

  struct __nat { int __for_bool_; };

#ifndef _LIBCPP_CXX03_LANG
  typedef __unique_ptr_deleter_sfinae<_Dp> _DeleterSFINAE;

  template <bool _Dummy>
  using _LValRefType =
      typename __dependent_type<_DeleterSFINAE, _Dummy>::__lval_ref_type;

  template <bool _Dummy>
  using _GoodRValRefType =
      typename __dependent_type<_DeleterSFINAE, _Dummy>::__good_rval_ref_type;

  template <bool _Dummy>
  using _BadRValRefType =
      typename __dependent_type<_DeleterSFINAE, _Dummy>::__bad_rval_ref_type;

  template <bool _Dummy, class _Deleter = typename __dependent_type<
                             __identity<deleter_type>, _Dummy>::type>
  using _EnableIfDeleterDefaultConstructible =
      typename enable_if<is_default_constructible<_Deleter>::value &&
                         !is_pointer<_Deleter>::value>::type;

  template <class _ArgType>
  using _EnableIfDeleterConstructible =
      typename enable_if<is_constructible<deleter_type, _ArgType>::value>::type;

  template <class _UPtr, class _Up>
  using _EnableIfMoveConvertible = typename enable_if<
      is_convertible<typename _UPtr::pointer, pointer>::value &&
      !is_array<_Up>::value
  >::type;

  template <class _UDel>
  using _EnableIfDeleterConvertible = typename enable_if<
      (is_reference<_Dp>::value && is_same<_Dp, _UDel>::value) ||
      (!is_reference<_Dp>::value && is_convertible<_UDel, _Dp>::value)
    >::type;

  template <class _UDel>
  using _EnableIfDeleterAssignable = typename enable_if<
      is_assignable<_Dp&, _UDel&&>::value
    >::type;

public:
  template <bool _Dummy = true,
            class = _EnableIfDeleterDefaultConstructible<_Dummy>>
  _LIBCPP_INLINE_VISIBILITY
  constexpr unique_ptr() noexcept : __ptr_(pointer()) {}

  template <bool _Dummy = true,
            class = _EnableIfDeleterDefaultConstructible<_Dummy>>
  _LIBCPP_INLINE_VISIBILITY
  constexpr unique_ptr(nullptr_t) noexcept : __ptr_(pointer()) {}

  template <bool _Dummy = true,
            class = _EnableIfDeleterDefaultConstructible<_Dummy>>
  _LIBCPP_INLINE_VISIBILITY
  explicit unique_ptr(pointer __p) noexcept : __ptr_(__p) {}

  template <bool _Dummy = true,
            class = _EnableIfDeleterConstructible<_LValRefType<_Dummy>>>
  _LIBCPP_INLINE_VISIBILITY
  unique_ptr(pointer __p, _LValRefType<_Dummy> __d) noexcept
      : __ptr_(__p, __d) {}

  template <bool _Dummy = true,
            class = _EnableIfDeleterConstructible<_GoodRValRefType<_Dummy>>>
  _LIBCPP_INLINE_VISIBILITY
  unique_ptr(pointer __p, _GoodRValRefType<_Dummy> __d) noexcept
      : __ptr_(__p, _VSTD::move(__d)) {
    static_assert(!is_reference<deleter_type>::value,
                  "rvalue deleter bound to reference");
  }

  template <bool _Dummy = true,
            class = _EnableIfDeleterConstructible<_BadRValRefType<_Dummy>>>
  _LIBCPP_INLINE_VISIBILITY
  unique_ptr(pointer __p, _BadRValRefType<_Dummy> __d) = delete;

  _LIBCPP_INLINE_VISIBILITY
  unique_ptr(unique_ptr&& __u) noexcept
      : __ptr_(__u.release(), _VSTD::forward<deleter_type>(__u.get_deleter())) {
  }

  template <class _Up, class _Ep,
      class = _EnableIfMoveConvertible<unique_ptr<_Up, _Ep>, _Up>,
      class = _EnableIfDeleterConvertible<_Ep>
  >
  _LIBCPP_INLINE_VISIBILITY
  unique_ptr(unique_ptr<_Up, _Ep>&& __u) _NOEXCEPT
      : __ptr_(__u.release(), _VSTD::forward<_Ep>(__u.get_deleter())) {}

#if _LIBCPP_STD_VER <= 14 || defined(_LIBCPP_ENABLE_CXX17_REMOVED_AUTO_PTR)
  template <class _Up>
  _LIBCPP_INLINE_VISIBILITY
  unique_ptr(auto_ptr<_Up>&& __p,
             typename enable_if<is_convertible<_Up*, _Tp*>::value &&
                                    is_same<_Dp, default_delete<_Tp>>::value,
                                __nat>::type = __nat()) _NOEXCEPT
      : __ptr_(__p.release()) {}
#endif

  _LIBCPP_INLINE_VISIBILITY
  unique_ptr& operator=(unique_ptr&& __u) _NOEXCEPT {
    reset(__u.release());
    __ptr_.second() = _VSTD::forward<deleter_type>(__u.get_deleter());
    return *this;
  }

  template <class _Up, class _Ep,
      class = _EnableIfMoveConvertible<unique_ptr<_Up, _Ep>, _Up>,
      class = _EnableIfDeleterAssignable<_Ep>
  >
  _LIBCPP_INLINE_VISIBILITY
  unique_ptr& operator=(unique_ptr<_Up, _Ep>&& __u) _NOEXCEPT {
    reset(__u.release());
    __ptr_.second() = _VSTD::forward<_Ep>(__u.get_deleter());
    return *this;
  }

#else  // _LIBCPP_CXX03_LANG
private:
  unique_ptr(unique_ptr&);
  template <class _Up, class _Ep> unique_ptr(unique_ptr<_Up, _Ep>&);

  unique_ptr& operator=(unique_ptr&);
  template <class _Up, class _Ep> unique_ptr& operator=(unique_ptr<_Up, _Ep>&);

public:
  _LIBCPP_INLINE_VISIBILITY
  unique_ptr() : __ptr_(pointer())
  {
    static_assert(!is_pointer<deleter_type>::value,
                  "unique_ptr constructed with null function pointer deleter");
    static_assert(is_default_constructible<deleter_type>::value,
                  "unique_ptr::deleter_type is not default constructible");
  }
  _LIBCPP_INLINE_VISIBILITY
  unique_ptr(nullptr_t) : __ptr_(pointer())
  {
    static_assert(!is_pointer<deleter_type>::value,
                  "unique_ptr constructed with null function pointer deleter");
  }
  _LIBCPP_INLINE_VISIBILITY
  explicit unique_ptr(pointer __p)
      : __ptr_(_VSTD::move(__p)) {
    static_assert(!is_pointer<deleter_type>::value,
                  "unique_ptr constructed with null function pointer deleter");
  }

  _LIBCPP_INLINE_VISIBILITY
  operator __rv<unique_ptr>() {
    return __rv<unique_ptr>(*this);
  }

  _LIBCPP_INLINE_VISIBILITY
  unique_ptr(__rv<unique_ptr> __u)
      : __ptr_(__u->release(),
               _VSTD::forward<deleter_type>(__u->get_deleter())) {}

  template <class _Up, class _Ep>
  _LIBCPP_INLINE_VISIBILITY
  typename enable_if<
      !is_array<_Up>::value &&
          is_convertible<typename unique_ptr<_Up, _Ep>::pointer,
                         pointer>::value &&
          is_assignable<deleter_type&, _Ep&>::value,
      unique_ptr&>::type
  operator=(unique_ptr<_Up, _Ep> __u) {
    reset(__u.release());
    __ptr_.second() = _VSTD::forward<_Ep>(__u.get_deleter());
    return *this;
  }

  _LIBCPP_INLINE_VISIBILITY
  unique_ptr(pointer __p, deleter_type __d)
      : __ptr_(_VSTD::move(__p), _VSTD::move(__d)) {}
#endif // _LIBCPP_CXX03_LANG

#if _LIBCPP_STD_VER <= 14 || defined(_LIBCPP_ENABLE_CXX17_REMOVED_AUTO_PTR)
  template <class _Up>
  _LIBCPP_INLINE_VISIBILITY
      typename enable_if<is_convertible<_Up*, _Tp*>::value &&
                             is_same<_Dp, default_delete<_Tp> >::value,
                         unique_ptr&>::type
      operator=(auto_ptr<_Up> __p) {
    reset(__p.release());
    return *this;
  }
#endif

  _LIBCPP_INLINE_VISIBILITY
  ~unique_ptr() { reset(); }

  _LIBCPP_INLINE_VISIBILITY
  unique_ptr& operator=(nullptr_t) _NOEXCEPT {
    reset();
    return *this;
  }

  _LIBCPP_INLINE_VISIBILITY
  typename add_lvalue_reference<_Tp>::type
  operator*() const {
    return *__ptr_.first();
  }
  _LIBCPP_INLINE_VISIBILITY
  pointer operator->() const _NOEXCEPT {
    return __ptr_.first();
  }
  _LIBCPP_INLINE_VISIBILITY
  pointer get() const _NOEXCEPT {
    return __ptr_.first();
  }
  _LIBCPP_INLINE_VISIBILITY
  deleter_type& get_deleter() _NOEXCEPT {
    return __ptr_.second();
  }
  _LIBCPP_INLINE_VISIBILITY
  const deleter_type& get_deleter() const _NOEXCEPT {
    return __ptr_.second();
  }
  _LIBCPP_INLINE_VISIBILITY
  _LIBCPP_EXPLICIT operator bool() const _NOEXCEPT {
    return __ptr_.first() != nullptr;
  }

  _LIBCPP_INLINE_VISIBILITY
  pointer release() _NOEXCEPT {
    pointer __t = __ptr_.first();
    __ptr_.first() = pointer();
    return __t;
  }

  _LIBCPP_INLINE_VISIBILITY
  void reset(pointer __p = pointer()) _NOEXCEPT {
    pointer __tmp = __ptr_.first();
    __ptr_.first() = __p;
    if (__tmp)
      __ptr_.second()(__tmp);
  }

  _LIBCPP_INLINE_VISIBILITY
  void swap(unique_ptr& __u) _NOEXCEPT {
    __ptr_.swap(__u.__ptr_);
  }
};
```
