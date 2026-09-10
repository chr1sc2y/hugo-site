---
title: "Closures and Anonymous Functions in C++"
date: 2020-11-14T21:20:18+08:00
draft: false
categories: ["C++ Pointer"]
description: "A translated technical note on Closures and Anonymous Functions in C++, preserving the examples and context of the original article."
---
# Closures and Anonymous Functions in C++

> Originally published in Chinese on 2020-11-14; this English edition preserves the original scope and technical context.

This blog post primarily introduces the concepts of closures and functors in C++.

## 1 Closure and Functors

A **closure** (also known as a **lexical closure** or **function closure**) can be understood as an operation that comes with attached data. Wikipedia defines a closure in programming languages as: "In programming languages, a **closure**, also **lexical closure** or **function closure**, is a technique for implementing **lexically scoped name binding** in a language with **first-class functions**." There are two layers of meaning:

1. Lexically scoped name binding (C++'s lexical scope is static binding, including blocks, functions, classes, namespaces, global scope, etc.): In a lexical scope, the variable name is associated with the identifier of the lexical context, independent of the runtime call stack;
2. Functions as first-class citizens (first-class citizen): At runtime, a function object can be constructed and passed as a parameter to other functions;

Clearly, C++ 98 does not meet these two definitions, so C++ 98 does not have strict closures. However, we can simulate the behavior of closures using functors (functors are classes that overload the parentheses operator, behaving similarly to functions, having their own private member variables, such as:
```cpp
class Adder
{
public:
    int operator()(int num)
    {
        sum += num;
        return sum;
    }

    Adder() : sum(0) {}
    Adder(int num) : sum(num) {}
private:
    int sum;
};

int main()
{
    Adder adder(0);
    cout << adder(1) << endl;
    cout << adder(2) << endl;
    cout << adder(3) << endl;
}
```
```shell
$ g++ -std=c++98 -o adder adder.cpp
$ ./adder
1
3
6
```
Compared, true closures in Go are much simpler:
```go
func adder() func(int) int {
	sum := 0
	return func(num int) int {
		sum += num
		return sum
	}
}

func main() {
	numAdder := adder()
	fmt.Println(numAdder(1))
	fmt.Println(numAdder(2))
	fmt.Println(numAdder(3))
}
```
```shell
$ go run main.go
1
3
6
```
C++ 98's standard library provides many useful functions, such as `std::sort`. When customizing the sorting rules, one can also define a simple function or a simple function object as a parameter. Note that when defining the sorting rules, they must satisfy the [Strict Weak Ordering](https://www.boost.org/sgi/stl/StrictWeakOrdering.html):
```cpp
struct Foo
{
    int a_, b_;
    Foo(int a, int b) : a_(a), b_(b) {}
};

struct FooComparatorGreater
{
    bool operator()(const Foo f1, const Foo f2)
    {
        if (f1.a_ != f2.a_)
            return f1.a_ > f2.a_;
        return f1.b_ > f2.b_;
    }
};

int main()
{
    vector<Foo> foo{ Foo(3, 6), Foo(9, 2), Foo(9, 8) };
    sort(foo.begin(), foo.end(), FooComparatorGreater());
    for (const auto& f : foo)
        cout << f.a_ << ' ' << f.b_ << endl;
    return 0;
}
```
 ```shell
$ g++ -std=c++11 -o sort-functor sort-functor.cpp
$ ./sort-functor
9 8
9 2
3 6
 ```
## 2 Anonymous Functions

Anonymous functions, also known as lambda expressions, originated from the first functional programming language Lisp, and were formally introduced in C++ 11 as a feature. In C++ 11, anonymous functions are called lambda expressions; they are functions that are not bound to any identifier, allowing for the definition of a temporary function object or the passing of a function object to a higher-level function (such as `std::for_each`), which is lightweight in terms of syntax, requiring no separate declaration in a header file like ordinary named functions, and thus **fitting the definition of closures**.

Anonymous functions can replace complex and redundant functors, making the code more understandable and maintainable:
```c++
sort(foo.begin(), foo.end(), [](const Foo& f1, const Foo& f2)
{
    return f1.a_ != f2.a_ ? f1.a_ > f2.a_ : f1.b_ > f2.b_;
});
```
Anonymous functions consist of several parts, of which only parts 1, 2, and 6 are mandatory, while the rest can be omitted:

![lambda-expression-syntax](https://raw.githubusercontent.com/chr1sc2y/warehouse-deprecated/refs/heads/main/resources/cpp/lambda-expression-syntax.png)

1. Capture Clause: Lambda Introducer
2. Parameter List: Lambda Declarator
3. Mutable Specification
   - An anonymous function marked with mutable can modify variables captured by value.
4. Exception Specification
5. Trailing Return Type
6. Lambda Body

### 2.1 Capture Clause

Capturing clauses capture external variables, enabling anonymous function bodies to use these variables. There are two methods of capturing: reference capture and value (copy) capture. Usage is as follows:

`[]` does not catch any variables.

2. `[&]` captures all external variables by reference.

3. `[=]` Capture all external variables by value.

4. `[&, var]` captures `var` by reference by default; captures `var` by value otherwise.

5. `[=, &var]` captures `var` by value by default; captures `var` by reference;

6. `[&, &var]` captures by reference and captures `var` by reference. This has no meaningful effect and will result in a warning.

7. `[=, this]` captures by value and captures `this` pointer by value, which makes no sense and will result in a warning.
   ```cpp
   std::function<void()> AnonyFunc = [=, this]() -> void {};//  warning: explicit by-copy capture of ‘this’ redundant with by-copy capture default
   ```
9. `[this]` captures the `this` pointer by value. Although the `this` pointer cannot be modified, the object it points to can be operated and modified, equivalent to capturing the object pointed to by `this` by reference, i.e., `[&(*this)]`.
   ```cpp
   class Foo
   {
   public:
       void Func()
       {
           int y{ 0 };
           std::function<void()> AnonyFunc = [this]() -> void
           {
x_ = 2; // ok, x_ is a member variable of a class and can be modified.
y = 2; // error: ‘y’ is not captured, the local variable in a function is not captured.
this = nullptr; // error: an lvalue required as left operand of assignment, the captured ‘this’ pointer is a temporary value and cannot be modified.
           };
           AnonyFunc();
       }

   private:
       int x_ = 0;
   };
   ```
10. `[*this]` cannot capture the `this` pointer of the object it refers to in C++11 by value;
    ```cpp
    std::function<void()> AnonyFunc = [*this]() -> void {}; // error: expected identifier before ‘*’ token
    ```
In the usage of the capture clause, some issues need to be taken into consideration:

1. Capturing with 2 or 3 is not recommended (it has a significant performance impact). Instead, clearly indicate the variables that need to be captured by reference;

2. Variables captured by value are read-only (const). Modification of captured variables is only possible if the mutable specification of the anonymous function is explicitly declared as `mutable`.
   ```cpp
   int x{ 0 };

   auto AnonyFunc = [=]() -> void
   {
       x = 1; // error: assignment of read-only variable ‘x’
   }

   auto AnonyFunc = [=]() mutable -> void
   {
       x = 1; // ok
   }
   ```
3. The value of variables captured by value is determined at the time the anonymous function is generated. Modifying the value of an external variable after the generation of the anonymous function does not affect the value of the variables captured within the anonymous function. This is because they are variables from different scopes.
   ```cpp
   int i{ 0 };
   auto AnonyFunc = [i]() -> void
   {
       cout << i << endl;
       cout << &i << endl;
   };
   i = 1;
   cout << i << endl;
   cout << &i << endl;
   AnonyFunc();
   ```
   ```shell
   $ g++ -std=c++11 -o lambda-capture lambda-capture.cpp
   $ ./lambda-capture
   1
   0x7ffe31fced8c
   0
   0x7ffe31fced80
   ```
4. For variables captured by reference (or pointers captured by value), if the reference variable (or the object pointed to by the pointer) is deallocated externally, the reference variable (or the pointer to the object) in the anonymous function becomes a **dangling pointer** ([Dangling Pointer](https://en.wikipedia.org/wiki/Dangling_pointer)):
   ```cpp
   int* x = new int[1000000];
   x[0] = 0;
   auto AnonyFunc = [&x]()
   {
       x[0] = 1; // Segmentation fault
   };
   delete[] x;
   AnonyFunc();
   ```
   ```cpp
   struct Foo
   {
       int x_[1000000];
   };

   int main()
   {
       Foo* f = new Foo();
       f->x_[0] = 0;
       auto AnonyFunc = [f]() -> void
       {
           f->x_[0] = 1; // Segmentation fault
       };
       delete f;
       AnonyFunc();
   }
   ```
### 2.2 Anonymous Functions and Closures

Scott Meyers explains the relationship between lambda expressions (anonymous functions) and closures as follows: *"The distinction between a lambda and the corresponding closure is precisely equivalent to the distinction between a class and an instance of the class. A class exists only in source code; it doesn’t exist at runtime. What exists at runtime are objects of the class type. Closures are to lambdas as objects are to classes. This should not be a surprise, because each lambda expression causes a unique class to be generated (during compilation) and also causes an object of that class type–a closure–to be created (at runtime)."*

This explanation can be broken down into two parts:

1. The relationship between anonymous functions and closures is analogous to that between classes and their instances. Classes exist solely in source code; they do not exist at runtime. What exists at runtime are objects of the class type. Closures are to lambdas as objects are to classess. This should not be surprising, because each lambda expression generates a unique class (during compilation) and creates an object of that class type—a closure—(at runtime).*

Further elaborated by the C++ 11 standard:

- "[C++11: 5.1.2/3]: The type of the lambda-expression (which is also the type of the closure object) is a unique, unnamed non-union class type — called the closure type..." In C++ 11, anonymous functions are essentially implemented using classess (closure types).
- *“[C++11: 5.1.2/5]: The closure type for a lambda-expression has a public inline function call operator (13.5.4) whose parameters and return type are described by the lambda-expression’s parameter-declaration-clause and trailing-return-type respectively. [..]”*，the closure type of a lambda-expression has a public inline function call operator (`operator()`), which is described by the lambda-expression's parameter-declaration-clause for its parameters and its trailing-return-type for its return type.
- "[C++11: 5.1.2/6]: The closure type for a lambda-expression with no lambda-capture has a public non-virtual non-explicit const conversion function to pointer function having the same parameter and return types as the closure type’s function pointer operator. The value by this conversion function shall be the address of a function that, when invoked, has the same effect as invoking the closure type’s function pointer operator." If an anonymous function has no parameters, it will generate a regular function rather than a closure type.

One can know that anonymous functions are implemented as function objects in C++. They are actually a syntax sugar added in C++ 11, though their syntax characteristics comply with the definition of closures.

## Anonymous Functions in C++ 14 and Later Versions

### C++ 14 Generalized Capture

C++ 14 introduces new **generalized lambda captures** (Generalized Lambda Captures), which allows initializing variables in the capture list in any manner within an anonymous function in C++. This enables certain types, which are otherwise disabled due to the absence of a copy constructor, to be captured via `std::move` into an anonymous function:
```cpp
auto ptr_0 = make_unique<int>( 0 );
auto AnonyFunc = [ptr_0 = move(ptr_0)]()
{
    *ptr_0 = 1;
    cout << *ptr_0 << endl;
};
AnonyFunc();
```
Here, the left and right `ptr_0` captured in the list are not the same variable; their scopes are inside and outside the anonymous function, respectively;

Furthermore, generalized lambda captures can also be used to indirectly capture `*this`, i.e., the `this` pointer that cannot be captured by value in C++11.
```cpp
auto AnonyFunc = [this_copy = *this]() mutable
{
    this_copy.x_ = 1;
    cout << this_copy.x_ << endl;
};
AnonyFunc();
```
### C++ 17 Capture `*this`

In C++ 17, capturing `*this` is finally possible, as noted in [P0018R3](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2016/p0018r3.html). P0018R3 suggests that capturing `*this` can be used in concurrent applications that require asynchronous operations, as `this` can become invalidated.
```cpp
auto AnonyFunc = [*this]() mutable
{
    x_ = 1;
    cout << x_ << endl;
};
AnonyFunc();
cout << x_ << endl;
```
```shell
$ g++ -std=c++17 -o lambda lambda.cpp
$ ./lambda
1
0
```

## Original references

- [Reference 1](https://en.wikipedia.org/wiki/Closure_(computer_programming))
- [Reference 2](https://en.wikipedia.org/wiki/Lexically_scoped)
- [Reference 3](https://en.wikipedia.org/wiki/Name_binding)
- [Reference 4](https://en.wikipedia.org/wiki/First-class_citizen)
- [Reference 5](https://www.geeksforgeeks.org/functors-in-cpp/)
- [Reference 6](https://en.wikipedia.org/wiki/Anonymous_function)
- [Reference 7](https://en.wikipedia.org/wiki/Functional_programming)
- [Reference 8](https://en.wikipedia.org/wiki/Lisp_(programming_language)#:~:text=Lisp%20(historically%20LISP)%20is%20a,is%20older%2C%20by%20one%20year)
- [Reference 9](https://en.cppreference.com/w/cpp/language/lambda)
- [Reference 10](https://en.cppreference.com/w/cpp/algorithm/sort)
- [Reference 11](https://en.wikipedia.org/wiki/C++14#Generic_lambdas)
