---
title: "A Template-Based Singleton in C++"
date: 2020-09-10T21:25:43+08:00
draft: false
categories: ["Design Pattern"]
description: "A translated technical note on A Template-Based Singleton in C++, preserving the examples and context of the original article."
---
# A Template-Based Singleton in C++

> Originally published in Chinese on 2020-09-10; this English edition preserves the original scope and technical context.

The singleton pattern is a creative design pattern. A class designed using the singleton pattern has only one instance (single instance) in the program. This class is called a singleton class, which provides a global access point. For a discussion of the singleton pattern, please refer to [Singleton revisited](http://www.italiancpp.org/2017/03/19/singleton-revisited-eng/); Based on these two characteristics, the singleton pattern can have the following implementations:

## Meyer’s Singleton

*Scott Meyers* proposed a singleton pattern implemented using C++'s `static` keyword in *Item 4: Make sure that objects are initialized before they're used* in *Effective C++*. This implementation is very simple and efficient. Its characteristics are:

1. Only when the program executes the `GetInstance` function for the first time, the initialization of the `instance` object is performed;
2. After C++ 11, variables modified with `static` can be guaranteed to be thread-safe;
```cpp
template<typename T>
class Singleton
{
public:
    static T& GetInstance()
    {
        static T instance;
        return instance;
    }

    Singleton(T&&) = delete;
    Singleton(const T&) = delete;
    void operator= (const T&) = delete;

protected:
    Singleton() = default;
    virtual ~Singleton() = default;
};
```
By disabling the copy constructor, move constructor and operator= of the singleton class, you can prevent the only instance of the class from being copied or moved; not exposing the constructor and destructor of the singleton class ensures that the singleton class will not be instantiated through other means, and defining them as protected allows them to be inherited and used by subclasses.

## Lazy Singleton

Lazy Singleton is a relatively traditional implementation method. As you can see from its name, it also has the characteristics of lazy-evaluation, but thread safety issues need to be considered when implementing it:
```cpp
template<typename T, bool is_thread_safe = true>
class LazySingleton
{
private:
    static unique_ptr<T> t_;
    static mutex mtx_;

public:
    static T& GetInstance()
    {
        if (is_thread_safe == false)
        {
            if (t_ == nullptr)
                t_ = unique_ptr<T>(new T);
            return *t_;
        }

        if (t_ == nullptr)
        {
            unique_lock<mutex> unique_locker(mtx_);
            if (t_ == nullptr)
                t_ = unique_ptr<T>(new T);
            return *t_;
        }

    }

    LazySingleton(T&&) = delete;
    LazySingleton(const T&) = delete;
    void operator= (const T&) = delete;

protected:
    LazySingleton() = default;
    virtual ~LazySingleton() = default;
};

template<typename T, bool is_thread_safe>
unique_ptr<T> LazySingleton<T, is_thread_safe>::t_;

template<typename T, bool is_thread_safe>
mutex LazySingleton<T, is_thread_safe>::mtx_;
```
We control whether this class is thread-safe through the template parameter `is_thread_safe`, because in some scenarios we will want each thread to have an instance:

1. When `is_thread_safe == false`, that is, it is not thread-safe, we directly determine it in the `GetInstance` function, initialize and return the singleton object; `unique_ptr` is used here to prevent memory leaks when the thread is destroyed, and the pointer can also be destroyed in the destructor;
2. When `is_thread_safe == true`, we use double-checked locking to check and lock to prevent the singleton class from being instantiated on each thread.

## Eager Singleton

Contrary to Lazy Singleton, Eager Singleton uses the characteristics of static member variables to initialize before the program enters the main function, thus bypassing the issue of thread safety:
```cpp
template<typename T>
class EagerSingleton
{
private:
    static T* t_;

public:
    static T& GetInstance()
    {
        return *t_;
    }

    EagerSingleton(T&&) = delete;
    EagerSingleton(const T&) = delete;
    void operator= (const T&) = delete;

protected:
    EagerSingleton() = default;
    virtual ~EagerSingleton() = default;
};

template<typename T>
T* EagerSingleton<T>::t_ = new (std::nothrow) T;
```
But it also has two problems:

1. Even if the singleton object is not used, the singleton class object will be initialized;
2. [static initialization order fiasco](https://isocpp.org/wiki/faq/ctors#static-init-order), that is, the initialization order of the t_ object and the `GetInstance` function is not fixed;

## Testing

Inherit the four Singletons implemented above and pass them into the thread object as a functor for testing:
```cpp
class Foo : public Singleton<Foo>
{
public:
    void operator() ()
    {
        cout << &GetInstance() << endl;
    }
};

class LazyFoo : public LazySingleton<LazyFoo, false>
{
public:
    void operator() ()
    {
        cout << &GetInstance() << endl;
    }
};

class ThreadSafeLazyFoo : public LazySingleton<ThreadSafeLazyFoo>
{
public:
    void operator() ()
    {
        cout << &GetInstance() << endl;
    }
};

class EagerFoo : public EagerSingleton<EagerFoo>
{
public:
    void operator() ()
    {
        cout << &GetInstance() << endl;
    }
};

void SingletonTest()
{
    thread t1((Foo()));
    thread t2((Foo()));
    t1.join();
    t2.join();
    this_thread::sleep_for(chrono::milliseconds(100));

    t1 = thread((LazyFoo()));
    t2 = thread((LazyFoo()));
    t1.join();
    t2.join();
    this_thread::sleep_for(chrono::milliseconds(100));

    t1 = thread((ThreadSafeLazyFoo()));
    t2 = thread((ThreadSafeLazyFoo()));
    t1.join();
    t2.join();
    this_thread::sleep_for(chrono::milliseconds(100));

    t1 = thread((EagerFoo()));
    t2 = thread((EagerFoo()));
    t1.join();
    t2.join();
}
```
The output is:
```bash
0x60d110
0x60d110
0x7f92380008c0
0x7f92300008c0
0x7f92300008e0
0x7f92300008e0
0x1132010
0x1132010
```
It can be seen that only the instance addresses output by the second group of non-thread-safe `LazySingleton` in the two threads are different, and the other Singletons are all thread-safe.
