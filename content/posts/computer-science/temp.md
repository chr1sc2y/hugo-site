---
title: "Linux and C++ Systems Notes"
date: 2022-01-18T19:50:22+08:00
draft: false
categories: ["C++"]
description: "A translated technical note on Linux and C++ Systems Notes, preserving the examples and context of the original article."
---
# Linux and C++ Systems Notes

> Originally published in Chinese on 2022-01-18; this English edition preserves the original scope and technical context.

Linux:

I mostly complete my learning on Linux. I currently have two computers, an iMac and a laptop, where the laptop is running Linux. I use many commands daily, such as using pipeline outputs with `grep` for regular expression searches, `ifconfig` and `netstat` to check network status, `top` to view process resources, and `iostat` to monitor CPU and disk states. I also use basic commands like `cd`, `ls`, `cp`, `rm`, `mkdir`, `find`, and `whereis`. Additionally, I use `scp` for remote operations. I also use system calls (`open`, `close`, `read`, `write`, `ioctl`, etc.) to interact with the file system in Linux, whereas on macOS, I use `dtruss`.

Debugging:
I use Vim for editing (I know some Vim shortcuts, such as `dd` to delete a line, `gg` to go to the top, and `6$` to go to a specific line). I use `gdb` for debugging and output logs to inspect the program. I have used `log4cplus` for logging. For memory errors or leaks, I use `valgrind` for debugging. I also use `GNU profiler` to view the execution of functions and gather relevant information. For unit tests, I use `gtest` on Linux with `GCC` and on macOS with the built-in `clang`.

C++ Memory Allocation:
1. Stack: Parameters and local variables
2. Heap: `new`, grows towards higher addresses, non-contiguous
3. Free Store: `malloc`
4. Global/Static Storage: Global and static variables
5. Constant Storage: Constants

`new` fails:
1. `int* p = new (std::nothrow) int(1);` returns a null pointer
2. Use `try {} catch (const bad_alloc &b) {}` to catch the exception

`auto_ptr`: Has copy constructor and assignment operator, transfers ownership
`unique_ptr`: Exclusively owns the pointer, ensures that only one smart pointer can point to a specific pointer at a time, and is automatically destroyed. It can transfer ownership.
`shared_ptr`: Multiple smart pointers can point to the same pointer object. The object is released when the last `shared_ptr` is destroyed. `weak_ptr` and `bad_weak_ptr` are auxiliary classes to implement this, and a custom deleter can be used to customize the deletion process. By default, only the pointer is deleted, but if it's an array, the rest of the array is not deleted.

A "circular reference" means that two classes have pointers to each other as members, and they point to each other's smart pointers. This creates a circular reference, making it impossible to release the objects correctly.
weak_ptr: References only, not counting. If a pointer is simultaneously referenced by some `shared_ptr` and `weak_ptr`, and all `shared_ptr`s are released, the memory will be freed regardless of whether any `weak_ptr` still references it. This resolves circular references.

Thread Communication: Locks (mutex, read-write locks, spinlocks, condition variables, atomic variables, critical sections), semaphores, signals, barriers.
