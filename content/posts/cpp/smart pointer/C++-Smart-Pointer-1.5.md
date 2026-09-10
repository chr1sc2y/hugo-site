---
title: "C++ Smart Pointers (1.5): Move Semantics"
date: 2019-01-02T09:55:05+11:00
draft: false
categories: ["C++"]
description: "A translated technical note on C++ Smart Pointers (1.5): Move Semantics, preserving the examples and context of the original article."
---
# C++ Smart Pointers (1.5): Move Semantics

> Originally published in Chinese on 2019-01-02; this English edition preserves the original scope and technical context.

## Move Semantics

### Definition

Right-Value References (Rvalue References) are a new feature introduced in C++ 11, which implements move semantics and perfect forwarding. The primary purposes include:

- Eliminating unnecessary object copies during interactions between two objects, conserving storage resources and improving efficiency.
- Providing a more concise and explicit definition for generic functions.

### Implementation

The implementation of move semantics is quite simple. It transforms the passed parameter `_Tp&& __t` into the corresponding type's right value using static type conversion `static_cast<_Up&&>(__t)`. Thus, by using move semantics, the compiler steals (generally in the move constructor and move assignment operator) the original object's right value, extending its lifetime and using it to assign to other objects without performing any copying of the right value.
```c++
template <class _Tp>
typename remove_reference<_Tp>::type&&
move(_Tp&& __t) _NOEXCEPT
{
    typedef typename remove_reference<_Tp>::type _Up;
    return static_cast<_Up&&>(__t);
}
```
### Testing

Define a `Object` class and a `MoveObject` function that uses move semantics to return an `Object`. This can be seen by calling the move constructor on the `obj` object after returning the right value from the `MoveObject` function.
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
```c++
Object MoveObject() {
    Object obj;
    return move(obj);
}

int main() {
    Object obj = MoveObject();
    return 0;
}
/*
output:
Construct
Move
Destruct
Destruct
*/
```
Return Value Optimization (RVO)

Return value optimization is a compiler optimization technique that allows the compiler to construct a function's return value at the call site directly.

Define a `CopyObject` function that returns an `Object` object. Originally, the `Function` function would copy once when returning, but the debugging results show that the `obj` object is constructed once within the `Function` function and destructed once at the end. This is because the compiler uses the RVO mechanism, where the `Function` function returns a left value, hence known as Named Return Value Optimization (NRVO). This feature was introduced in C++ 11 as Copy Elision.
```c++
Object CopyObject() {
    Object obj;
    return obj; // NRVO (named return value optimisation)
}

int main() {
    Obj obj = Function();
    return 0;
}
/*
output:
Construct
Destruct
 */
```
If MoveObject function returns with move semantics, the actual return value is a temporary reference (Obj&&), not the object defined in the function definition, and RVO is not triggered. In order to trigger RVO, the actual return value type of the function must be consistent with the return value type defined in the function.
```c++
Object MoveObject() {
    Object obj;
    return move(obj);
}

int main() {
    Object obj = MoveObject();
    return 0;
}
/*
output:
Construct
Move
Destruct
Destruct
*/
```
If the return type of the function is also changed to a rvalue reference, the `obj` object in the `main` function will use the move constructor, triggering the RVO mechanism.
```c++
Object &&MoveObject() {
    Object obj;
    return move(obj);
}

int main() {
    Object obj = MoveObject();
    return 0;
}
/*
output:
Construct
Destruct
Move
Destruct
*/
```
In the `CopyObject` function, it turns out that it also invokes the move constructor upon return, rather than triggering the Return Value Optimization (RVO). This is because the compiler uses the parent stack frame to avoid returning a copy, and if a conditional statement is used at return, the compiler cannot determine which return value to use at compile time, thus not triggering RVO. The move constructor is called instead when left-value return is used, with the compiler preferring the move constructor over the copy constructor if it is not supported.
```c++
Object CopyObject(bool flag) {
    Object obj1, obj2;
    if (flag)
        return obj1;
    return obj2;
}

int main() {
    Object obj = CopyObject(true);
    return 0;
}
/*
output:
Construct
Construct
Move
Destruct
Destruct
Destruct
*/
```
## Reference



[RVO V.S. std::move](https://www.ibm.com/developerworks/community/blogs/5894415f-be62-4bc0-81c5-3956e82276f3/entry/RVO_V_S_std_move?lang=en)

[right value references and move semantics](https://www.ibm.com/developerworks/cn/aix/library/1307_lisl_c11/index.html)

## Original references

- [Reference 1](https://en.wikipedia.org/wiki/Call_site)
