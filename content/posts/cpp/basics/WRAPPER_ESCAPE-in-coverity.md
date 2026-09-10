---
title: "Understanding Coverity's WRAPPER_ESCAPE Warning"
date: 2020-03-15T17:24:27+08:00
draft: false
categories: ["C++"]
description: "A translated technical note on Understanding Coverity's WRAPPER_ESCAPE Warning, preserving the examples and context of the original article."
---
# Understanding Coverity's WRAPPER_ESCAPE Warning

> Originally published in Chinese on 2020-03-15; this English edition preserves the original scope and technical context.

```c++
const char* Foo()
{
    std::string str_msg("test");
    return str_msg.c_str();
}

int main() {
    const char *p_msg = Foo();
    printf("%s\n", p_msg);
    return 0;
}

// output: (empty, or garbled)
D?
```
| Above code's Foo function is reported with the coverity warning WRAPPER_ESCAPE. The detailed explanation is as follows:|

Above code, the `Foo` function reports a `WRAPPER_ESCAPE` warning from Coverity. The issue is detailed as follows:
```
Wrapper object use after free (WRAPPER_ESCAPE)
1. escape: The internal representation of local strMsg escapes, but is destroyed when it exits scope
```
The local variable `str_msg`, which is allocated on the stack within the function `Foo`, will be deallocated when it leaves the function (since `str_msg` is allocated on the stack). When the function `std::string::c_str()` is called to obtain a pointer to the beginning of `str_msg`, the returned pointer becomes a dangling pointer. Returning this dangling pointer to to the caller will result in unpredictable behavior.

While `c_str()` returns a `const char* p`, we cannot directly modify the data pointed to by the pointer `p`. However, we can achieve the effect of modifying the data pointed to by `p` by modifying `str_msg`, as shown in the following code:
```c++
int main() {
    std::string str_msg("test");
    const char *p_msg = str_msg.c_str();
    printf("%s\n", p_msg);
    str_msg[2] = 'x';
    printf("%s\n", p_msg);
    return 0;
}

// output:
test
text
```
To use the returned `const char*` correctly, we can allocate a block of memory on the heap, copy the string into it, and then return it:

c
#include <stdlib.h>
#include <string.h>

char* safe_strdup(const char* str) {
    size_t len = strlen(str) + 1;
    char* copy = malloc(len);
    if (copy == NULL) {
        return NULL; // Error handling_strdup
    }
    memcpy(copy, str, len);
    return copy;
}

```c++
const char* Foo()
{
    std::string str_msg("test");
    uint32_t u32_msg_size = str_msg.size() + 1;
    char *p_return = new char[u32_msg_size];
    strcpy_s(p_return, u32_msg_size, str_msg.c_str());
    return p_return;
}

int main() {
    const char *p_msg = Foo();
    printf("%s\n", p_msg);
    return 0;
}

// output:
test
```
Of course, the caller should delete the `p_msg` at appropriate times to avoid memory leaks.

What should be noted is that, unless you need to use strings immediately in the form of `const char*`, you should avoid using `c_str()` especially when passing them as function arguments.
