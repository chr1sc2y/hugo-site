---
title: "Effective C++ Notes"
date: 2020-09-24T16:43:27+08:00
draft: false
categories: ["C++"]
description: "A translated technical note on Effective C++ Notes, preserving the examples and context of the original article."
---
# Effective C++ Notes

> Originally published in Chinese on 2020-09-24; this English edition preserves the original scope and technical context.

## 0 Introduction

### 1 Constructors

Default constructor: A constructor that can be called without any arguments. Such a constructor must either have no parameters or have parameters with default values, for example.
```c++
class Bar {
public:
   // explicit Bar(); // is default constructor
    // explicit Bar(int x = 0) // is not default constructor
    explicit Bar(int x = 0, bool b = true); // is default constructor
private:
    int x;
    bool b;
};
```
explicit keyword: Prevents implicit type conversions, its advantage being the prohibition of the compiler performing unexpected type conversions, such as
```c++
void Foo(Bar obj); // Foo function takes a parameter of type Bar

Bar obj_1; // Construct a Bar type object
Foo (obj_1); // No problem, pass a Bar type object to the Foo function
Foo (Bar()); // No problem, construct a Bar type object and pass it to the Foo function
Foo(2); // If Bar's constructor is not declared explicit, it will implicitly construct a Bar object with a member variable x = 2; if its constructor is declared is declaredexplicit, it won't construct a Bar object.
```
Copy Constructor: Initializes a new object of the same type with another object of the same type. It defines how an object is passed by reference.

Copy Assignment: Assigns the value of another object of the same type to itself, and the "=" operator can also be used to invoke the copy constructor, for example:

cpp
struct MyStruct {
    int value;

    // Copy Constructor
    MyStruct(const MyStruct &other) : value(other.value) {}

    // Copy Assignment Operator
    MyStruct &operator=(const MyStruct &other) {
        if (this != &other) {
            value = other.value;
        }
        return *this;
    }
};

```c++
class Bar {
public:
   Bar(); // default constructor
    Bar(const Bar &rhs); // copy constructor
    Bar& operator=(const Bar &rhs); // copy assignment
};

Bar b1; // Call default constructor
Bar b2(b1); // Call copy constructor
b1 = b2; // Call copy assignment
Bar b3 = b2; // Call copy constructor
```
#### 0.2 Undefined Behavior
```c++
int *p = nullptr;
std::cout << *p; // Dereferencing an uninitialized null pointer results in undefined behavior.

char name[] = "Joel"; // name is an array of 5 chars
char c = name[10]; // Invalid array index leads to undefined behavior
```
### 2. Try Using `const`, `enum`, and `inline` Instead of `#define`

Using Compiler Substitution for Preprocessors; Macros defined with variables are substituted by the compiler before the source code is processed, never entering the token table.

### 3. Use `const` as Much as Possible

#### 3.1 Iterator

STL's `iterator` is similar to a `T*` pointer. If declared as `const`, it is actually a `T* const`, or a `const` pointer toT, which cannot point to another object; however, STL's `const_iterator` is simply a `const T*`, or a `const` `iterator` object, whose value cannot be modified, but its pointer can be changed to point to another object.

For `const` type STL containers, use `const_iterator` for traversal; for non-`const` type STL containers, use `iterator` for traversal.

`iterator` does not need to be bound to `STL::begin()` in the form of `reference` because `STL::begin()` returns a temporary `pointer` variable. For built-in types and STL iterators and functors, passing by value is more efficient than passing by reference.

Using `auto` to traverse an STL container directly iterates over it, rather than using an `iterator` pointer.

#### 3.2 `const` and `multable`

### 4. Ensure data is initialized before use.

#### 4.1 Member Variable Initialization

For built-in types, perform manual initialization as C++ does not guarantee their initialization.

For a class, initialize each member variable in the constructor. Initialization of member variables occurs before entering the constructor body. Using a member initialization list is more efficient and avoids confusion with assignment; in the member initialization list, all member variables are listed in the same order as their declarations.

#### 4.2 Static Variables

The lifetime of a `static` object extends from its construction until the end of the `static` object. Initializing the order of non-local `static` objects is difficult; a common approach is through implicit template instantiations. The issue can be resolved by using `local static` objects, as `local static` objects are initialized the first time a function is called.

### 5. C++ Default Function Writing and Invocation

#### 5.1 Class
The compiler automatically generates a copy constructor and copy assignment operator for a class declaration; these operators simply copy each non-static member variable from the source object to the target object.

Compilers automatically declare destructor for non-virtual functions;

When **no constructors** are provided, the compiler generates a default constructor for the class declaration; these functions are inline and public, and are only generated at compile time if they are called, provided that the generated code is valid. Classes that contain `const` members or have the copy assignment declared as `private` in the base class cannot have corresponding functions automatically generated.

### For functions automatically generated by the compiler that are not wanted, explicitly reject them.

Function parameter names do not need to be specified.

Will the copy constructor and copy assignment operator be declared as private and not implemented? This will prevent the compiler from generating these functions and also prevent them from being called externally. However, if they are called from member functions or friend functions, the linker will report an error. We should move compile-time errors to link-time errors as much as possible, to detect errors as early as possible.

Can design a base class specifically to prevent copying, as the copy function and copy assignment in the base class are private. Derivative classes cannot call the copy function and copy assignment, so the compiler will refuse to generate these functions for the subclass and report an error.
```c++
class Uncopyable {
protected:
    Uncopyable();
    ~Uncopyable();
private:
    Uncopyable(const Uncopyable &);
    Uncopyable & operator=(const Uncopyable &);
}

class Foo: private Uncopyable {
    ...
}
```
C++ 11 introduced the `default` and `delete` keywords. The former allows the compiler to generate a function, while the latter directly prohibits the use of a function.
```c++
class Foo: private Uncopyable {
public:
    A() = default;
    A(const A &) = delete;
}
```
### 7. For Base Class Declaring Virtual Destructor

When a derived class object is deleted via a base class pointer, if its destructor is not virtual, the compiler does not look up the correct destructor on the vtable and only executes the base class's destructor, leading to partial destruction.

If you do not intend for a function to be part of a virtual base class implementation, do not declare its destructor as virtual; only declare the destructor as virtual if the class contains at least one virtual function.

None of the STL containers have virtual destructors.

For an abstract class class, declare its destructor as pure virtual.

The behavior of destructors is to call the destructor of the most-derived class from the base class's destructor, so a definition must be provided for pure virtual destructor.

### 8. Do Not Throw Exceptions in Destructors

In C++, if two exceptions occur simultaneously, undefined behavior (or termination of execution) occurs.

### 9. Do Not Call Virtual Functions in Constructors and Destructors

For a derived class class, its object is constructed from the base class's constructor to the derived class class's constructor.

### 10. Make Operator= Return a Reference to *this
```c++
class Foo {
public:
    ...
    Foo &operator=(const Foo &rhs) {
        ...
        return *this;
    }
}
```
STL containers all use this protocol.

### 11. Handle Self-Assignment in `operator=`

Add an identity test at the beginning of `operator=`, to prevent pointer aliasing issues arising from self-assignment:
```c++
Foo& Foo::operator=(const Foo &rhs) {
    if (this == &rhs)
        return *this;
    delete p_bm;
    p_bm = new Bitmap(*rhs.p_bm);
    return *this;
}
```
Such an implementation lacks exception safety since `new Bitmap` could throw an exception due to insufficient memory or a faulty copy constructor.
```c++
Foo& Foo::operator=(const Foo &rhs) {
    Bitmap *p_bm_temp = p_bm;
    p_bm = new Bitmap(*rhs.p_bm);
    delete p_bm_temp;
    return *this;
}
```
### 12. Copying Objects Ensures Replication of Each Member

When writing a copying function (including the copy constructor and copy assignment), ensure the following: 1. Copy all local member variables; 2. Call the corresponding copying functions for all base classes.

If the copy constructor and copy assignment have similar code, a new private initialization function type can be defined for reuse.

### 13. Manage Resources Using Objects

Acquire resources immediately and put them into a management object. This is known as the Resource Acquisition Is Initialization (RAII) principle, where resources are acquired immediately and used to initialize a management object; the management object ensures resource release through its destructor. For example, using smart pointers.

### 14. When Handling Copying Behavior in Resource Management Classes Be Cautious

### Using `new` and `delete` in pairs must take the same form.

The memory layout of a single object differs from that of an array object, which also includes a record of the array size to know how many destructors to call with `delete`. `new[]` and `delete[]` must be used together.

## 4 Design and Declaration

### 18. Make interfaces easy to use correctly and difficult to misuse.
```
Inconsistency creates psychological and mental friction and disputes for developers, and no one IDE can fully eliminate it.
```
### Must return an object, do not return its reference.

In cases where an object must be returned, avoid returning its reference.
```
Whenever you see a reference declaration, you should immediately ask yourself, what is its other name?
```
Regardless of whether the object is allocated on the heap or the stack, returning a reference will cause issues. Returning an object on the heap will lead to memory leaks, while returning an object on the stack will result in undefined behavior.
```
Never return a pointer or reference to a local stack, or return a reference to a heap-allocated object, or return a pointer or reference to a local static.
```
### 22\. Declare member variables as private.
```
Hide members behind the function interface to provide flexibility for "all possible implementations." For example, this allows members to be easily notified when read or written to, enforces constraints on classes, verifies function preconditions and postconditions, and enables synchronization in multi-threaded environments, among others.
```
### 23. Use non-member, non-friend functions instead of member functions.

From the perspective of encapsulation, there are only two access levels: private (for encapsulation) and other (not for encapsulation); protected is not more encapsulated than public, as modifying either a protected or a public variable would require significant code changes.
```
Place all the traversal functions in separate header files but under the same namespace, meaning customers can easily extend this set of traversal functions. They need only add more non-member non-friend functions to this namespace.
```
### 24. If all parameters require type conversion, make this function a non-member

### 25. Support a swap function that does not throw exceptions

All STL containers provide public swap functions and a specialized version of std::swap.
```
Defining variables too quickly may delay efficiency; Overusing casting can slow down code and make it harder to maintain, introducing subtle hard-to-understand errors; Returning a handle to the internal data of an object may break encapsulation and leave clients with dangling handles; Ignoring the impact of exceptions may lead to resource leaks and corrupted data; Overzealous inlining may inflate code size; Excessive coupling can lead to bloated builds.
```
### 26. Delay Variable Definition Occurrence

Delay the definition of the variable until just before it is needed, otherwise incur additional construction and destruction costs, and avoid the meaningless default construction behavior.

For use in loops repeatedly.

### 27. Use Casting Less

C++ style type conversions have four kinds:

- `const_cast`: Remove constness from an object
- `dynamic_cast`: Performs safe downcasting, determining whether an object belongs to a certain type within the inheritance hierarchy; it cannot be performed with C-style type conversions but may incur significant costs
- `reinterpret_cast`: Executes a low-level conversion, whose action and result depend on the compiler
- `static_cast`: Performs implicit conversions, such as converting a non-const object to a const object, or an int to a double

C++ style type conversions are preferable because:

- Easier to Identify
- Each transformation action has a clearer meaning, making debugging easier.

Any type conversion results in the generation of runtime code.

### Avoid Returning Handles Pointing to Internal Object Members

Objects contain members beyond just member variables, including private member functions. They should never return a reference to a member function with a lower access level.

Avoid returning handles pointing to internal members to enhance encapsulation. This makes the behavior of const member functions truly const and reduces the risk of dangling pointers.

### 29. Write Exception-Safe Code

When an exception is thrown, the function with exception safety will:

- Leaking no resources, i.e., not releasing any subsequent resources due to a blocked process.
- Not allowing data corruption, i.e., disallowing the occurrence of dangling pointers.

The exception-safe function provides any one of the following three guarantees:

1. Basic Guarantee: No objects or data structures are destroyed when an exception is thrown, and all objects are in a consistent state.
2. Strong Guarantee: If an exception is thrown, the program state does not change; if the program fails, it will revert to the state before the calling function.
3. No Exception Guarantee: The program ensures no exceptions are thrown because it can always complete its promised functionality.

### Understanding inlining
```
On a machine with limited memory, overindulging in inlining can make the program size too large; even with virtual memory, the code bloat from inlining will lead to additional paging behavior, reducing the instruction cache hit rate and resulting in efficiency loss.
```
inline can be metaphorically stated, such as defining a function within the class definition.

Functions are typically placed in header files, as inlining is a compile-time behavior in most C++ programs. The compiler needs to know the function's implementation to replace function calls calls with the function body. Templates are similar.

Limiting most inlining to small and frequently called functions makes debugging and binary updates easier, and minimizes potential code bloat issues, while improving program performance.

### 31. Minimize Compilation Dependencies Across Files

An `#include` in a definition file and the corresponding header file create a compilation dependency. If any header file changes, or any header files it depends on change, all files that include the changed header will be recompiled. Any files that use the definitions in the included file will also need to be recompiled, known as cascading compilation dependencies.

When the compiler sees the definition, it must know how much memory to allocate.

## 6. Inheritance and Object-Oriented Design

### 32. Ensure public inheritance is an `is-a` relationship

Every function and object applicable to the base class class must also be applicable to the derived class class.

### 33. Avoid shadowing inherited names

Names in inner scopes shadow names in outer scopes. When the compiler is inside a scope, it first looks for a variable name in the local scope. If it doesn't find it, it looks elsewhere.

Variables and functions from other scopes can be made visible in the current scope using the `using` keyword.

### 34. Distinguish between interface inheritance and implementation inheritance

Purpose of different types of member functions:

- Pure virtual function: to inherit the interface of a function
- Virtual function: to inherit the interface of a function and provide a default implementation
- Non-virtual function: to inherit the interface of a function and provide a mandatory implementation

If a member function is non-virtual, it means it doesn't intend to be overridden in a derived class class; a non-virtual function represents its invariance over specialization.
```
A typical programmer spends 80% of his execution time on 20% of the code; this means that, on average, 80% of function calls can be virtual without impacting the program's overall efficiency. Therefore, before worrying about the cost of virtual functions, focus on the 20% of code.
```
### 35. �allback

1. Use a non-virtual interface to implement the Template Method pattern.

non-virtual interface (NVI): A client indirectly invokes the private virtual function through the public non-virtual member function, which is a unique manifestation of the template method pattern. This public non-virtual function is referred to as the wrapper of the virtual function.

2. Implement the Strategy Pattern using function pointers.

### 36. NEVER REDEFINE NON-VIRTUAL FUNCTIONS THAT ARE INHERITED

Non-virtual functions are statically bound, the called function corresponds to the type of the pointer itself; virtual functions are dynamically bound, the called function corresponds to the type of the object pointed to by the pointer.

### 37. NEVER REDEFINE DEFAULT PARAMETER VALUES BY INHERITANCE

- Static Type: The type adopted when declared in a program.
- Dynamic Type: Refers to the type that a pointer actually points to; the dynamic type can indicate what behavior an object will have, and the dynamic type can change during execution.
```c++
Shape *p_s;
Shape *p_c = new Circle();
Shape *p_r = new Rectangle();
```
### 38. Through Composition to Build `has-a` or `is-implemented-in-terms-of` Relationships

In the application domain, composition means `has-a`; in the implementation domain, it means `is-implemented-in-terms-of`.

### 39. Use Private Inheritance Wisely

Private inheritance means implemented-in-terms-of, and the public and protected functions and objects in the base class are private.

Use composition whenever possible, and use private inheritance only when necessary.

### 40. Use Multiple Inheritance Wisely

Multiple inheritance can lead to multiple ambiguities:

- Two base classes have the same function signature: they have the same matching degree but no best match, to resolve this, you must explicitly state which base class function to call.
- Diamond inheritance: there are multiple paths from a base class to a derived class class, and the derived class class will copy the base classes' data multiple times; if using virtual inheritance, the derived class class will only keep one copy. The standard library classes `basic_ios`, `basic_istream`, `basic_ostream`, and `basic_iostream` are diamond inheritance structures, and they use virtual inheritance.

Generally, public inheritance should be virtual inheritance.

## 7.### 41. Understand Implicit Interfaces and Compiler Polymorphism

- Object-oriented programming always solves problems with explicit interfaces and run-time polymorphism. Explicit interfaces are defined by the function signatures.
- Template parameters have implicit interfaces. Implicit interfaces are defined by valid expressions, meaning that for template variables, they must provide the required member functions and operators; template polymorphism is resolved at the compile-time.

### 42. Understand the Double Meaning of `typename`

When using templates, `typename` and `class` have the same meaning.

In templates, nested dependent names are nested dependent names that depend on no parameters, while non-dependent names are not dependent on any parameters.
Nested Dependent Names Can Complicate Parsing. During Compilation, if the compiler encounters a nested dependent name within a template, it assumes the name is not a type, for example:
```c++
template <typename T>
void Foo(const T &t) {
    T::const_iterator iter(t.begin()); // Warning: Missing 'typename' prior to dependent type name 'T::const_iterator'
}
```
Compiler assumes `T::const_iterator` is not a type, hence issuing a warning. This can be resolved by adding the `typename` keyword in front of it; however, `typename` cannot appear in base class lists, nor can it be used as a base specifier in the member initialization list, such as:
```c++
template <typename T>
class Derived : public Base<T>::Nested {} // Nested classes does not need to use typename in base list
public:
  explicit Derived(int x):Base<T>::Nested(x) { // Member initialization list cannot use typename
typename T::Nested temp; // `typename` may precede an ordinary nested dependent name
    }
};
```
### 43. Learn to Handle Name Issues within Template Base Classes

### 44. Extract Code Independent of Parameters into Templates

## 8. Customizing new and delete

### 49. Understand the Behavior of the `new` Keyword

When `operator new` cannot satisfy the memory allocation allocation requirements, it throws an exception;

Before `operator new` throws an exception, it first calls the `new-handler` error-handling function, which can be set using `std::set_new_handler`.

| c++ 11        | operator new                                                 |
| ------------- | ------------------------------------------------------------ |
| throwing (1)  | void* operator new (std::size_t size);                       |
| nothrow (2)   | void* operator new (std::size_t size, const std::nothrow_t& nothrow_value) noexcept; |
| placement (3) | void* operator new (std::size_t size, void* ptr) noexcept;   |

A well-designed `new-handler` should accomplish:

1. Allocate More Memory; Otherwise Call Another `new-handler`, or Throw an Exception
2. Catch `bad_alloc`, Call `abort`, or `exit`

### 50. Understand the Rational Time to Replace new and delete

Reasons for Replacing the Default `operator new` and `operator delete`:

cpp
// Replace the default operator new and operator delete
// To improve performance and flexibility.


cpp
// Example of custom operator new
void* custom_new(size_t size) {
    // Custom logic to allocate memory
    return malloc(size);
}

// Example of custom operator delete
void custom_delete(void* ptr) {
    // Custom logic to deallocate memory
    free(ptr);
}


cpp
// In C++, use custom_new and custom_delete.
extern "C" void* operator new(size_t size) { return custom_new(size); }
extern "C" void operator delete(void* ptr) { custom_delete(ptr); }


cpp
// Example: Replacing default operator new and operator delete with custom_new and custom_delete
void* custom_new(size_t size) {
    // Custom logic to allocate memory


1. Detect runtime errors
2. Optimize performance: Increase allocation and deallocation speeds, reduce additional space overhead, compensate for suboptimal alignment
3. Statistical data

### 51. When writing `new` and `delete`, one must adhere to the conventions.

## 9. Miscellany

### 53. Pay Attention to Compiler Warnings

USE `-Wall -Wextra -Werror` AT COMPILE TIME.

### Familiar with [TR1](https://en.wikipedia.org/wiki/C%2B%2B_Technical_Report_1) and Standard Library content
tr1 is a 2007 addition to the standard library, including functions and types such as `shared_ptr`, `function`, `bind`, `unordered_map`, `unordered_set`, `regex`, `tuple`, `array`, `mem_fn`, and `reference_wrapper`. It also includes templates like `type_traits` and `result_of`.

### 55. Familiarize Yourself with boost
