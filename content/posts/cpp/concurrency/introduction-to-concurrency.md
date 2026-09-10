---
title: "An Introduction to C++ Concurrency with LeetCode 1114"
date: 2020-09-30T16:20:25+08:00
draft: false
categories: ["concurrency"]
description: "A translated technical note on An Introduction to C++ Concurrency with LeetCode 1114, preserving the examples and context of the original article."
---
# An Introduction to C++ Concurrency with LeetCode 1114

> Originally published in Chinese on 2020-09-30; this English edition preserves the original scope and technical context.

## Problem

Directly solve the problem: [1114 Print in Order](https://leetcode.com/problems/print-in-order/)

## Solution

### 1. `std::mutex`

If you are somewhat familiar with C++ 11, you might think of using `std::mutex` to solve this problem. In the constructor of the function (main thread), lock the `std::mutex`, and then unlock the `std::mutex` object in each thread's function:
```cpp
class Foo {
    mutex mtx1, mtx2;
public:
    Foo() {
        mtx1.lock(), mtx2.lock();
    }

    void first(function<void()> printFirst) {
        printFirst();
        mtx1.unlock();
    }

    void second(function<void()> printSecond) {
        mtx1.lock();
        printSecond();
        mtx1.unlock();
        mtx2.unlock();
    }

    void third(function<void()> printThird) {
        mtx2.lock();
        printThird();
        mtx2.unlock();
    }
};
```
Mutex, or **mutual exclusion**, is a mechanism used to prevent multiple threads from simultaneously accessing shared resources. Only one thread can hold a `mutex` object at any given time. When another thread calls the `std::mutex::lock` function, it will block until it acquires the lock.

This approach, however, is incorrect according to the C++ standard. When a thread attempts to call `unlock` on a `mutex` object, the ownership of the `mutex` must be held by the same thread; otherwise, undefined behavior occurs. The code mentions that `first`, `second`, and `third` functions are called by three different threads, but the `mutex` objects are locked during the construction of the `Foo` object (either in the main thread that creates these threads or in one of the threads itself). Therefore, at least one of the threads that call `first` or `second` is attempting to acquire the ownership of a `mutex` held by another thread.

Furthermore, if we were to discuss any potential optimizations for this approach, it is best to use the RAII (Resource Acquisition Is Initialization) mechanisms provided by `lock_guard` or `unique_lock` to manage the `mutex` objects. Directly manipulating `mutex` objects is not recommended; instead, `lock_guard` provides a simple RAII implementation, while `unique_lock` is a complete ownership wrapper for `mutex`, encapsulating all of its functions.
```cpp
class Foo {
    mutex mtx_1, mtx_2;
    unique_lock<mutex> lock_1, lock_2;
public:
    Foo() : lock_1(mtx_1, try_to_lock), lock_2(mtx_2, try_to_lock) {
    }

    void first(function<void()> printFirst) {
        printFirst();
        lock_1.unlock();
    }

    void second(function<void()> printSecond) {
        lock_guard<mutex> guard(mtx_1);
        printSecond();
        lock_2.unlock();
    }

    void third(function<void()> printThird) {
        lock_guard<mutex> guard(mtx_2);
        printThird();
    }
};
```
### 2. std::condition_variable

[`std::condition_variable`](https://en.cppreference.com/w/cpp/thread/condition_variable) is a **synchronization primitive** that **must be used in conjunction with `std::unique_lock`**:
```cpp
class Foo {
    condition_variable cv;
    mutex mtx;
    int k = 0;
public:
    void first(function<void()> printFirst) {
        printFirst();
        k = 1;
       cv.notify_all();														// Notify all threads waiting on the wake-up queue.
    }

    void second(function<void()> printSecond) {
        unique_lock<mutex> lock(mtx);								// lock mtx
       cv.Wait(lock, [this]{ return k == 1; });	// Unlock mtx and block waiting for wakeup notification, continue only if k == 1.
        printSecond();
        k = 2;
       cv.notify_one();														// notify_one one of the unspecified threads waiting in the wake-up queue
    }

    void third(function<void()> printThird) {
        unique_lock<mutex> lock(mtx);								// lock mtx
       cv.Wait(lock, [this]{ return k == 2; });	// Unlock mtx and block waiting for a wakeup notification, continue only if k == 2.
        printThird();
    }
};

```
`std::condition_variable::wait` function executes three operations: first, it adds the current thread to the wake-up queue, then `unlock`s the `mutex` object, and finally blocks the current thread. It has two overloads; the first one only receives a `std::mutex` object. In this case, the thread is immediately awakened upon receiving a wake-up signal (via `std::condition_variable::notify_one` or `std::condition_variable::notify_all`) and re-locks the `mutex`. The second overload form also receives a condition (typically a variable or `std::function`), meaning the thread can only be awakened when this condition is satisfied. The implementation in GCC is quite simple, adding a `while` loop to ensure the thread is awakened only when the given condition is met, otherwise, it calls `wait` again.
```cpp
template<typename _Predicate>
void wait(unique_lock<mutex>& __lock, _Predicate __p)
{
    while (!__p())
        wait(__lock);
}
```
**Condition Variable** `std::condition_variable` operates similarly to **semaphore**, both building upon the foundation of `mutex` for achieving synchronized access to shared resources. Unfortunately, the standard library does not provide an implementation or encapsulation of semaphores, but we can still solve the problem using the `<semaphore.h>` library in C.
```cpp
#include <semaphore.h>

class Foo {
private:
    sem_t sem_1, sem_2;

public:
    Foo() {
        sem_init(&sem_1, 0, 0), sem_init(&sem_2, 0, 0);
    }

    void first(function<void()> printFirst) {
        printFirst();
        sem_post(&sem_1);
    }

    void second(function<void()> printSecond) {
        sem_wait(&sem_1);
        printSecond();
        sem_post(&sem_2);
    }

    void third(function<void()> printThird) {
        sem_wait(&sem_2);
        printThird();
    }
};

```
### 3. std::future

**`std::future` is a template class used to obtain the result of an asynchronous operation**; [`std::packaged_task`](https://en.cppreference.com/w/cpp/thread/packaged_task), [`std::promise`](https://en.cppreference.com/w/cpp/thread/promise), and [`std::async`](https://en.cppreference.com/w/cpp/thread/async) can perform asynchronous operations and have a `std::future` object that stores the value (or exception) returned or set by them, which will be modified at some future point and saved in the corresponding `std::future` object:

For `std::promise`, the value can be set and the `std::future` object notified by calling `std::promise::set_value`.
```c++
class Foo {
    promise<void> pro1, pro2;

public:
    void first(function<void()> printFirst) {
        printFirst();
        pro1.set_value();
    }

    void second(function<void()> printSecond) {
        pro1.get_future().wait();
        printSecond();
        pro2.set_value();
    }

    void third(function<void()> printThird) {
        pro2.get_future().wait();
        printThird();
    }
};
```
`std::future<T>::wait` and `std::future<T>::get` both block until the `promise` object owned by the future returns its stored value, with the latter also retrieving the `T`-typed object; this problem leverages the asynchronous communication mechanism without returning any actual values.

`std::packaged_task` is a functor that owns a `std::future` object, encapsulating a series of operations. It stores the returned value in its owned `std::future<T>` object upon function completion; similarly, this problem utilizes its mechanism of notifying the `std::future` object upon function completion.
```cpp
class Foo {
    function<void()> task = []() {};
    packaged_task<void()> pt_1{ task }, pt_2{ task };

public:
    void first(function<void()> printFirst) {
        printFirst();
        pt_1();
    }

    void second(function<void()> printSecond) {
        pt_1.get_future().wait();
        printSecond();
        pt_2();
    }

    void third(function<void()> printThird) {
        pt_2.get_future().wait();
        printThird();
    }
};
```
### 4. std::atomic

We usually perform data modifications as non-atomic operations, which can lead to data contention among multiple threads modifying the same object, resulting in undefined behavior. **Atomic operations ensure that multiple threads proceed in sequence without data contention**; during their execution, no other thread can modify the same atomic object. C++11 provides the `std::atomic<T>` template class to construct atomic types.
```c++
class Foo {
    std::atomic<bool> a{ false };
    std::atomic<bool> b{ false };
public:
    void first(function<void()> printFirst) {
        printFirst();
        a = true;
    }

    void second(function<void()> printSecond) {
        while (!a)
            this_thread::sleep_for(chrono::milliseconds(1));
        printSecond();
        b = true;
    }

    void third(function<void()> printThird) {
        while (!b)
            this_thread::sleep_for(chrono::milliseconds(1));
        printThird();
    }
};
```
Notably, the implementation of atomic operations is processor and operating system kernel-dependent, which is why the C++ standard does not specify whether `atomic` is lock-free (lock-free). It only mandates the provision of an `is_lock_free()` to query whether the current compiler implementation of `atomic` is lock-free.

## Original references

- [Reference 1](https://en.cppreference.com/w/cpp/thread/mutex)
- [Reference 2](https://en.cppreference.com/w/cpp/language/raii)
- [Reference 3](https://en.cppreference.com/w/cpp/thread/lock_guard)
- [Reference 4](https://en.cppreference.com/w/cpp/thread/unique_lock)
