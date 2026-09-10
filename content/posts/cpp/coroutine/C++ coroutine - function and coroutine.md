---
title: "C++ Coroutines (1): Functions and Coroutines"
date: 2020-01-20T20:15:05+08:00
draft: false
categories: ["C++"]
description: "A translated technical note on C++ Coroutines (1): Functions and Coroutines, preserving the examples and context of the original article."
---
# C++ Coroutines (1): Functions and Coroutines

> Originally published in Chinese on 2020-01-20; this English edition preserves the original scope and technical context.

This article aims to explore the mechanism and usage of coroutines in C++, and how to leverage the properties of coroutines to build libraries and applications at the upper layer.

## Stack Frames and Functions

Stack frames are environments for a function call, including parameters, return addresses, and local variables of the function. Each time an operating system invokes a function, it allocates a new stack frame for it. Related concepts include:

- ESP: Extended Stack Pointer, which always points to the top of the topmost stack frame in the stack memory.
- EBP: Extended Base Pointer, which always points to the bottom of the topmost stack frame in the stack memory.
- Function Stack Frame: The memory space between ESP and EBP represents the current stack frame. EBP identifies the bottom of the current stack frame, while ESP identifies the top.

For ordinary functions, we can generally perform two operations: call (invoke) and return. To facilitate comparison, this discussion does not consider the case of throwing an exception. When running a C++ program, the C++ runtime is executed first, followed by the invocation of the main function, which then invokes other functions.

A typical call operation involves the following steps:

1. Stack Parameters: Parameters are pushed from right to left onto the stack.
2. Return Address Pushed: The next instruction to be executed is pushed onto the stack at the current stack area, to be executed after the function returns.
3. Code Branch: The processor branches to the entry of the called function.
4. Frame Adjustment, Including:
   1. Saving the current frame state values, push EBP onto the stack.
   2. Switching from the current frame to the new frame, updating EBP to the value of ESP.
   3. Allocating memory space for the new frame, updating ESP to subtract the required space size from it.

When a function returns via a return statement, the execution steps are reversed compared to when it is called:
We can store coroutines on the heap without clearing stack frames, so we cannot use the stack data structure to strictly manage the lifecycle of active stack frames. We can divide the coroutine's stack frames into two parts. One part is the **execution stack frame**, which only exists during the execution of the current coroutine and is released when the coroutine suspends. The other part is the **data stack frame**, which exists even when the coroutine suspends.

### 2.1 Suspend

Co-routines execute suspend operations through certain specific statements. In C++ Coroutine TS, there are `co_await` and `co_yield`. During the execution of a suspend operation, we should ensure two points:

1. Save the data in the current **stack frame** to the **data stack frame**.
2. Write the position where the coroutine is suspended into the **data stack frame**. This allows the subsequent resume operation to know where to continue from, or the destroy operation to know which part to destroy.

Next, a coroutine can transfer execution rights to the caller, and the **stack frame** will be released.

### 2.2. Resume

We can use the `resume` operation to recover a suspended coroutine. Similar to the `call` operation for functions, the `resume` operation allocates a new **execution frame** to store the data in the **data frame** that has been saved, along with the return address of the caller. Subsequently, the coroutine loads the position where it was suspended and continues execution.

### 2.3 Destroy

The `Destroy` operation can only be executed on a suspended coroutine, similar to `resume`. It will first allocate an execution stack frame and store the return address of the caller within it. However, it does not continue executing the function body after the `suspend` position; instead, it executes the destructor for all local variables in the current scope and releases these resources.

### 2.4 Call and Return

Coroutine's call assigns it an active stack frame, pushing parameters and return addresses onto the stack and transferring control to the coroutine. The coroutine, in turn, allocates an **execution stack frame** on the heap and copies the parameters into it to ensure they can be correctly removed later.

Coroutine return operation differs slightly from that of a regular function. When a coroutine executes a return operation, it stores the return value at another address, then deletes all local variables, and transfers execution to the caller.

## Function and Coroutine Execution Process

Assuming `func()` is a function that calls calls `co_func(int x)` is called within its body. The compiler creates a new active stack frame on the stack during the call, pushing the parameters and return address onto the stack and moving ESP to the top of the new active stack frame as shown below.
```
Stack                       Register                Heap (Coroutine Manager)
                            +----+
+------------+  <----------  ESP
func()                      +----+
+------------+
...
```
Next, the coroutine manager will allocate a new block of memory on the heap as the coroutine's execution stack frame. At this point, the compiler will have EBP point to the top of the execution stack frame, as shown below.
```
Stack                       Register                Heap (Coroutine Manager)

+------------+  <-------                            +------------+
co_func()              |                  ------->   co_func()
x = 68                 |                  |          x = 68
ret = func() + 0x789   |    +----+        |         +------------+
+------------+         ----  ESP          |
func()                      +----+        |
+------------+               EBP  --------|
...                         +----+
```
If the execution of `co_func` is suspended at some point, the data in the **execution stack frame** will be saved to the **data stack frame**, and the coroutine will return some return values to the caller. These return values typically contain the location of the `suspend`, as well as the coroutine's suspension handle. This handle can be used to resume the coroutine during the `resume` operation, as shown below.
```
Stack                       Register                Heap (Coroutine Manager)
                            +----+        ------->  +------------+
+------------+  <----------  ESP          |          co_func()
func()                      +----+        |          x = 68
+------------+               EBP          |          resume point = co_func() + 16
handle  ---------------     +----+        |
...                   |                   |
                      |                   |
                      ---------------------
```
Now, when a coroutine is resumed due to some reason, the `resume` function is called to recover the coroutine. At this point, the compiler creates a new active stack frame to record the parameters and return address. The execution stack frame then reads data from the data stack frame to recover the coroutine, as shown below.


Now, when a coroutine is resumed due to some reason, the `resume` function is called to recover the coroutine. At this point, the compiler creates a new active stack frame to record the parameters and return address. The execution stack frame then reads data from the data stack frame to recover the coroutine, as shown below.



Now, when a coroutine is resumed due to some reason, the `resume` function is called to recover the coroutine. At this point, the compiler creates a new active stack frame to record the parameters and return address. The execution stack frame then reads data from the data stack frame to recover the coroutine, as shown below.

```
Stack                       Register                Heap (Coroutine Manager)

+------------+  <-------                            +------------+
co_func()              |                  ------->   co_func()
x = 68                 |                  |          x = 68
ret = func() + 0x789   |    +----+        |         +------------+
+------------+         ----  ESP          |
func()                      +----+        |
+------------+               EBP  --------|
handle                      +----+
...
```
