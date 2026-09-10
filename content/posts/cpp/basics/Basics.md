---
title: "C++ Fundamentals"
date: 2015-07-05T08:57:52+10:00
draft: false
categories: ["C++"]
description: "A translated technical note on C++ Fundamentals, preserving the examples and context of the original article."
---
# C++ Fundamentals

> Originally published in Chinese on 2015-07-05; this English edition preserves the original scope and technical context.

## const Related

1. #define，typedef，const
- `#` is a macro that does not perform type checking but only performs simple substitution; it is processed before compilation, and the result of macro processing is the code of the compilation stage.
- `typedef` is used to declare custom data types, simplifying the code.
- `const` is used to define constants with a data type, and the compiler performs type checking on them.

2. const and Pointer
    - const char *p: p is a pointer to const char
- char const *p; p is a pointer to char const.
    - char *const p: p is a const pointerchar
    ```c++
    int main() {
    const char *p1 = new char('a');
    char const *p2 = new char('b');
    char *const p3 = new char('c');
    *p1 = 'd'; // error: read-only variable is notassignable
    *p2 = 'e'; // error: read-only variable is notassignable
    p3 = new char('f'); // error: cannot assign tovariable 'p3' with const-qualified type 'char *const'
    p3 = nullptr;  // error: cannot assign to variable'p3' with const-qualified type 'char *const'
    }
    ```
3. `const` and Class Members
    - When `const` is used to modify a member function of a class, the function cannot modify the class's member variables or call non-`const` member functions of the class.
    - When `const` is used to modify a function parameter, the value of the parameter cannot be modified within the function.
    - When `const` is used to modify a function return value, the variable receiving the return value must also be declared as `const`.
    ```c++
    class Base {
    public:
        int i = 1;

        const int *Func(const int &j) const {
            const int *k = new int(1);
            i = 1; // error: cannot assign to non-static data member within const member function 'Func'
            j = 1; // error: cannot assign to variable 'j' with const-qualified type 'const int &'
            return k;
        }

        void Test() {
            i = 2;
        }
    };

    int main() {
        Base obj;
        int j = 1;
        int *k = obj.Func(j); // error: cannot initialize a variable of type 'int *' with an rvalue of type 'const int *'
        return 0;
    }
    ```
## Static Related

### 1. Procedural Approach

- **Modifiers static global variables, the scope of the static global variables only applies to the current source file.
    - The construction of global variables occurs before the `main` function.
- **Modifiers static global functions, the scope of the static global functions only applies to the current source file.
- **Modifiers local variables, the scope of the static local variables only applies to the current function; the variable is initialized for the first time it is encountered, allocated in the global data area, and deallocated when the function ends.

### Object-Oriented

- For member variables of a class, static member variables affect all instances of the class, and their memory is allocated in the global data area; these variables can be directly accessed through the class name.

- For member functions of a class, static member functions cannot operate on non-static member functions within the class; these functions can be directly accessed through the class name.

- Used for polymorphic type conversions, pointers can be moved throughout the class hierarchy, including upcasting and downcasting.

### 3. Type Conversion

- static_cast
- No type checking; akin to C's unsafe coercive type conversion, typically used for numerical data type conversions, such as converting `int` to `char`, or converting `void*` to another type of pointer (unsafe)
- Basic data type conversions, such as converting `int` to `char`
- Pointers can move up the class hierarchy, converting subclasses to superclasses (safe upcast)
- Safe (upcast), converting superclasses to subclasses (safe downcast)

- dynamic_cast
- Perform runtime type checks
- Only applicable to pointers or references, pointer conversions to an ambiguous type will fail (return `nullptr`), but no exceptions are thrown
- If a forced conversion to a reference type fails, the `dynamic_cast` operator throws a `bad_cast` exception
- Used for polymorphic type conversions, pointers can move up and down the entire class hierarchy, including upcasting and downcasting

- const_cast
- Remove `const`, `volatile`, and `__unaligned` attributes, such as converting `const int` to `int`.

- reinterpret_cast
- Simple reinterpretation for bits
- Allows any pointer to be converted to any other pointer type, such as `char*` to `int*`
- Allows any integer type to be converted to any pointer type and vice versa
- One practical use case is in hash functions, mapping values to indices such that two different values almost never map to the same index

## Memory Allocation

1. Stack Area
    - Assigned and released by the compiler, storing function parameters and local variables
    - Throws an exception when system space is less than the allocated space, indicating stack overflow
    - Within a process, the user stack, located at the top of the user virtual address space, is used by the compiler to implement function prologues

2. Heap Zone
### Memory Allocation Methods

1. `malloc`: Allocate a block of memory of a specified size. The initial value of the allocated memory is indeterminate.
2. `calloc`: Allocate memory for a block of objects of a specified size. The allocated memory is initialized to zero.
3. `realloc`: Change the size of previously allocated memory. If the size is increased, the previous allocation may need to be moved to a larger area, and the initial values of the new area are indeterminate.
4. `alloca`: Allocate memory on the stack. The memory is automatically released when the function returns. `alloca` is non-portable and difficult to implement on machines without a traditional stack. It is not suitable for programs that need to be widely portable.

## Pointer and Reference

1. `void *`
    - `void *` is a pointer that points to a region of memory, but it has no type information or auxiliary information about the region it points to. Therefore, it can be interpreted as any type of data. Before use, the system needs to be informed of how many bytes to extract from the memory region it points to.
    - `void *` cannot be dereferenced.

2. Distinguishing Pointer Types
    - `int *p[10]`
        - `int *p[10]` means an array of pointers, emphasizing the array concept. It is an array variable with a size of 10, and each element is a pointer to an `int` type variable.
- `int (*p)[10]`
- `int (*p)[10]` indicates an array pointer, emphasizing that it is a pointer, with only one variable, of pointer type, but it points to an array of `int`, which has a size of 10.

- `int *p(int)`
- `int *p(int)` is a function declaration. The function name is `p`, the parameter is of `int` type, and the return value is of `int *` type.

- `int (*p)(int)`
- `int (*p)(int)` is a function pointer, indicating a pointer that points to a function with an `int` parameter and an `int` return value.

3. Arrays and Pointers
    ```
    int a[10];
    int (*p)[10] = &a;
    ```
4. Array Names and Pointers to the First Element of an Array Differences

- Arrays can be accessed by adding or subtracting offsets.
- The array name is not a true pointer and can be understood as a constant pointer, so the array name does not support increment or decrement operations.
- When an array name is passed as a parameter to a function, it loses its original characteristics and becomes a general pointer. It gains increment and decrement operations, but `sizeof` no longer returns the size of the original array.

5. Wild Pointer

- Null pointer, refers to a pointer pointing to garbage memory
- Causes:
    - Uninitialized pointer variables
    - A pointer that has been freed or deleted is not nullified

6. Frequent References

- Use constant references like constant pointers, `const typename &refname = varname`
- When using constant references, the value of the original variable is not modified by the constant reference
- Constant references are typically used as read-only aliases for variables or as parameter references

## Smart Pointers

### unique_ptr

- `unique_ptr` implements concepts of exclusive ownership or strict ownership, ensuring that only one smart pointer can point to an object at any given time
- Once the owner is destroyed or a new object is owned, the previous pointer object is destroyed, and the corresponding resources are released
- Ownership can be transferred
- Used to avoid memory leaks, such as forgetting to use `delete` after `new`

### shared_ptr

- `shared_ptr` implements the concept of shared ownership (shared ownership)
- Multiple smart pointers can reference the same object, and the object and its associated resources will be released when the last reference is destroyed; `weak_ptr`, `bad_weak_ptr`, and `enable_shared_from_this` auxiliary classes are needed to implement this
- Custom deleters are supported
- **Prevent Cross-DLL issues (objects are created in one DLL and destroyed in another DLL, automatically releasing mutexes)**

### **weak_ptr**

- **weak_ptr allows sharing but does not own an object**
- **Once the last owning smart pointer loses ownership, any weak_ptr automatically becomes empty**
- **Therefore, weak_ptr provides a constructor that accepts a shared_ptr, apart from the default and copy constructors**
- **It breaks cycles of references (two objects that are no longer in use mutually reference each other, making them appear still "in use")**

### **auto_ptr**

- **Lack of language features like std::move semantics for construction and assignment**
- **auto_ptr vs unique_ptr comparison**
    - **auto_ptr can be assigned and copied, which transfers ownership upon copy; unique_ptr does not have copy assignment semantics but implements move semantics**
    - **auto_ptr objects cannot manage arrays (destructor uses delete), unique_ptr can manage arrays (destructor uses delete[])**

## Lambda Expression

### **1. Lambda Expressions Form**
```[capture list] (params list) mutable exception-> return type { function body }```

- capture list: List of captured external variables
- params list: Parameter list
- mutable: Indicates whether captured variables can be modified
- exception: Exception settings
- return type: Return type
- function body: Function body
- Only the capture list and function body are mandatory

### Capture List of External Variables

- Lambda expressions can access external variables within their scope, but must explicitly declare which external variables they can use.
- Lambda expressions capture external variables by preceding them with the square brackets [], which is also known as Lambda expression capturing. This is similar to value passing in parameter passing.
- External variable capture methods include:
    - Value Capture
        - Value capture is similar to value passing, where the value of the captured variable is copied and passed to the Lambda expression at creation time. Modifications within the function body do not affect the external value.
    - Reference Capture
        - Capture references using the `&a` syntax, and both value and reference captures must be explicitly listed.
    - Implicit Capture
        - Let the compiler infer which variables to capture based on the function body, which is referred to as implicit capture. Two forms exist: `[=]` for value capture and `[&]` for reference capture.
- Modifying captured variables
    - If a value is captured, modifications within the function body cannot modify the external variable, resulting in a compile error.

### 3. Argument List

- Lambda Expressions pass parameters with some restrictions
    - Parameter lists cannot have default parameters
    - Variable arguments are not supported
    - All parameters must have parameter names

## RTTI

- Runtime Type Identification

### Purpose

- Allow a program to retrieve the actual type pointed to or referenced by a pointer or reference to a base class type at runtime.
- Identify the types of all basic types through the `typeid` operator.

### 2. Use

- `typeid` operator, which returns the actual type of its expression or type name
- `dynamic_cast` operator, which safely casts a base pointer or reference to a pointer or reference of a derived type



## Template

- Implement generic programming.
    ```template <class type> ret-type func-name(parameter list) { }```
- Templates classes can use virtual functions
- Template classes' member functions cannot be virtual functions

## Keywords

1. volatile
    - A variable marked with volatile can be changed by unknown factors
    - A volatile variable fetches its value from the memory address every time it is accessed; non-volatile variables, due to compiler optimizations, can directly fetch values from CPU registers
    - A variable not marked with volatile is fetched from a CPU register (from where the value was saved in a register) every time it is accessed

2. inline
    - Inline functions are expanded at compile time, skipping the steps of entering the function and directly executing the function body
    - Compilers generally do not inline functions with loops, recursions, or switches
    - Functions declared in a class declaration are implicitly converted to inline functions by default
    - Pros
        - Expands the function at the call site, avoiding the overhead of parameter stacking, stack frame allocation and deallocation, and return operations
        - Inline functions perform safety checks or automatic type conversions, whereas macros do not
        - Members declared within a class are implicitly converted to inline functions
    - Cons
        - Code bloat; inline functions expand the total code size, consuming more memory space
        - Changes to inline functions cannot be automatically upgraded with function libraries. Changes to inline functions require recompilation

3. friend
    - Friend classes and friend functions can access other classes' private members
    - Breaks encapsulation, is not transitive, and is unidirectional

4. decltype
    - Retrieves the type of a variable
    - Checks the type and value category of a declared entity or an expression

## C++ Compilation

- Preprocessing (`.i`)
- Optimization (`.s`)
- Assembly program (`.obj`, `.o`)
- Linking (executable file)

### 1. Compilation: Translate text form source code into machine language and form a target file

1. Preprocessing
    - The compiler executes preprocessor directives (starting with #, such as `#include`) including copying the code from `#include`-included files, performing `#define` macro replacements, and handling conditional compilation directives (`#ifndef`, `#ifdef`, `#endif`)
    - Generates `.i` file

2. Compilation
    - Syntax analysis, lexical analysis, semantic analysis, generation of intermediate code, code optimization, code generation, etc.
    - Translate into assembly code
    - Generates `.s` file

3. Assembly
    - Translate into machine instructions
    - Generates `.o` target file
- Target files typically consist of at least two segments
    - Code segment: Contains the main program's instructions. This segment is readable and executable, generally not writable
    - Data segment: Stores global variables or static data used by the program. It is readable, writable, and executable

### 2. Linking: Organize the target file and the operating system's startup code and library files into an executable program.

- Concatenate the target files that are relevant.
- Reasons:
    - A library function was called in the program.
    - A source file calls a function or constant from another source file.
- Generate an executable file.

## C++ Function Call

1. Parameter stack push
    - Push parameters from right to left onto the stack
2. Save the frame (save return address)
    - Push the address of the next instruction after the current subroutine entry onto the stack, to continue execution upon function return
3. Execute subroutine
    - Code area jump: Processor jumps from the current code area to the subroutine entry
    - Frame adjustment, including
        - Save the state values of the current frame for later restoration (EBP pushed onto the stack)
        - Switch to the new frame (set ESP to EBP, update the frame bottom)
        - Allocate space for the new frame (decrement ESP by the required space size, raise the stack top)
4. Restore the frame

## STL Containers

### All Containers

|Container|Implementation|Query|Insert/Delete|Features|
|---|---|---|---|---|
|array|array|O(1)|O(1)|Fixed size|
|vector|vector|O(1)|Tail O(1), others O(n)|Variable size, grows with cost|
|deque|deque|O(n)|Head and Tail O(1), others O(n)|Central controller with multiple buffers|
|list|list|O(n)|O(1)| |
|forward_list|forward_list|O(n)|O(1)| |
|stack|deque / list| / |O(1)|Last In First Out (LIFO)|
|queue|deque / list| / |O(1)|First In First Out (FIFO)|
|priority_queue|vector| / |O(logn)|Heap, complete binary tree|
|set|red-black tree|O(logn)|O(logn)| |
|multiset|red-black tree|O(logn)|O(logn)| |
|map|red-black tree|O(logn)|O(logn)| |
|multimap|red-black tree|O(logn)|O(logn)| |
|unordered_set|unordered_map|Average O(1)|Average O(1)| |
|unordered_multiset|unordered_map|Average O(1)|Average O(1)| |
|unordered_map|unordered_map|Average O(1)|Average O(1)| |

|unordered_multimap|Hash Table|Average O(1)|Average O(1)|

### vector

Release the memory allocated for the vector.

- For containers like vector and string, executing the clear() function only sets its size to 0 without altering its capacity.
- The swap() function can be used to clean up the memory of containers like vector and string, replacing them with a temporary right value vector.
- Other STL containers clear() when it empties the memory.
- Test
    ```c++
    vector<int> v;
    printf("v.size() = %d, v.capacity() = %d\n", v.size(), v.capacity());
    for (int i = 0; i < 100000; ++i)
        v.push_back(i);
    printf("v.size() = %d, v.capacity() = %d\n", v.size(), v.capacity());
    vector<int>().swap(v);
    printf("v.size() = %d, v.capacity() = %d\n", v.size(), v.capacity());

    /*
    output:
    v.size() = 0, v.capacity() = 0
    v.size() = 100000, v.capacity() = 131072
    v.size() = 0, v.capacity() = 0
    */
    ```
2. resize Error
    - When the type of the vector is a custom class or struct, the compiler will initialize any portion of the size that exceeds the current size if the resize() function is called with a parameter greater than the current size; if the custom class does not have a constructor, a compile error will occur.

Red-Black Tree

1. Feature
    - Nodes are either red or black
    - The root is black
    - All leaves (null nodes) are black
    - Every red node must have two black children; no two consecutive red nodes along any path from a leaf to the root
    - Any simple path from any node to any leaf contains the same number of black nodes

2. Non-balanced, adjustments are made through color change, left rotation, and right rotation.

## Lvalues and Rvalues

1. Left values can be taken by address; right values are temporary variables that are about to be destroyed.

2. The lifetime of a temporary value ends when the expression it is used in ends. References to constant left values extend the lifetime of the temporary variable.

3. Left-Value References
    - Ordinary references, typically representing the identity or alias of an object.

4. Right-Valued References
    - Must be bound to a right value, generally representing the value of an object
    - Right-valued references can achieve move semantics and perfect forwarding
    - The primary purpose is to eliminate unnecessary object copies during interactions between two objects, saving computational and storage resources, and improving efficiency
    - They can define generic functions more concisely and clearly

## Sorting

|Sorting_Algorithms|Average_Time_Complexity|Worst_Time_Complexity|Space_Complexity|Stable_Data_Structure|
|---|---|---|---|---|
|Bubble Sort|O(n^2)|O(n^2)|O(1)|Stable|
|Selection Sort|O(n^2)|O(n^2)|O(1)|Unstable|
|Insertion Sort|O(n^2)|O(n^2)|O(1)|Stable|
|Quick Sort|O(n*log_2n)|O(n^2)|O(log_2n)|Unstable|
|Heap Sort|O(n*log_2n)|O(n*log_2n)|O(1)|Unstable|
|Merge Sort|O(n*log_2n)|O(n*log_2n)|O(n)|Stable|
|Shell Sort|O(n*log_2n)|O(n^2)|O(1)|Unstable|
|Counting Sort|O(n+m)|O(n+m)|O(n+m)|Stable|
|Bucket Sort|O(n)|O(n)|O(m)|Stable|
|Radix Sort|O(k*n)|O(n^2)||Stable|

## Other

### Overload Operators

1. new
    ```c++
    void *operator new(size_t size)
    {
        cout << "in threee_d new\n";
        return malloc(size);
    }
    ```
- throw(std::bad_alloc)
- const std::nothrow_t& does not throw exceptions

## Reference

- 《C++ Primer》
- 《Effective C++》
- 《More Effective C++》
- 《Deep Understanding of C++ Object Model》
- 《Deep Understanding of C++11》
- 《STL Source Code Analysis》

- `The Sword and Sword Offer`
- 《Programming Pearls》
- 《Programmer’s Bible》

- 《Deep Understanding of Computer System》
- 《Advanced Windows Core Programming》
- 《Advanced Unix Programming》

- 《Unix Network Programming》
- 《TCP/IP Illustrated》

- 《The Way of the Programmer》
- 《Unix Network Programming》
