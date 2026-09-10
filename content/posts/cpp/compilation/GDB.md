---
title: "An Introduction to Debugging with GDB"
date: 2020-04-22T17:46:06+08:00
draft: false
categories: ["C++"]
description: "A translated technical note on An Introduction to Debugging with GDB, preserving the examples and context of the original article."
---
# An Introduction to Debugging with GDB

> Originally published in Chinese on 2020-04-22; this English edition preserves the original scope and technical context.

## Zero. Introduction

Debugging is an essential part of the development process. When developing on Windows or MacOS, one can use debugging features in IDEs like VS and CLion to set breakpoints or inspect variables and stacks. However, on Linux, there is no graphical interface available. If one relies solely on logging to find issues, the efficiency will be very low. At this point, we can leverage GDB to enhance our development efficiency.

[GDB](https://en.wikipedia.org/wiki/GNU_Debugger) is the short name for the GNU Debugger, a standard debugger in the GNU software system. GDB possesses various debugging features including breakpoints, single stepping, printing variables, inspecting registers, and tracking function call stacks. It effectively tracks and warns about the execution of functions; while debugging with GDB, one can monitor and modify program variables independently of the main program. GDB is primarily used for debugging compiled languages, supporting C, C++, Go, and Fortran, but it does not support interpreted languages.

## 1. Setup Environment

### 1.1 Writing Programs

To prepare for debugging, we need to prepare a simple C++ program:
```cpp
$ cat test.cpp
#include <iostream>

void Func(const char *s) {
    int *p = nullptr;
    int &r = static_cast<int&>(*p);

    int num = std::atoi(s);
    r = num;
    printf("%d\n", r);
}

int main (int argc, char *argv[]) {
    if (argc != 2) {
        printf("test [int]\n");
        return -1;
    }
    Func(argv[1]);
    return 0;
}

```
### 1.2 Compilation

For C/C++ programs, when compiling with gcc/clang, the parameter `-g` must be added to generate complete debugging information for debugging in GDB.
```cpp
$ clang++ -g -std=c++11 -m64 -o test test.cpp
```
## 2. Debugging Example

GDB has many features, and when we forget how to use these features, we can view the commands by inputting ```help``` or ```help all``` in the GDB interactive interface.
```
(gdb) help
List of classes of commands:
...
```
### 2.1 Launch

We can launch the debugging by using ```gdb [executable file]```:
```
$ gdb test.cpp
...
Reading symbols from test...done.
(gdb)
```
You can also directly use `gdb` to enter the interactive interface, and then use `file` to specify the program to debug:
```
$ gdb
...
(gdb) file test
Reading symbols from test...done.
```
### 2.2 Running

For a program with parameters, we can use `set args [arg] ...` to set the parameters and then use `run` to run it:
```
(gdb) set args 1
(gdb) run
Starting program: test 1
...
```
Of course, parameters can be directly added after ```run``` to run it:
```
(gdb) run 1
Starting program: test 1

Program received signal SIGSEGV, Segmentation fault.
0x0000000000400749 in Func (s=0x7fffffffe167 "/data/home/joelzychen/test/test") at test.cpp:8
8           r = num;
```
We discovered a segmentation fault in the program, at which point we can use breakpoints to debug the program.

### 2.3 Breakpoints

From the error message, we see that the issue occurs at line 8 of the function `Func` function in the file `test.cpp`. Therefore, we set two breakpoints: one at the point where the function `Func` is entered, and another at line 8 of the `test.cpp` file. We can view the breakpoints we have set by using `info breakpoints` or `info b`:
```
(gdb) b test.cpp:8
Breakpoint 1 at 0x400742: file test.cpp, line 8.
(gdb) b Func
Breakpoint 2 at 0x40071c: file test.cpp, line 4.
(gdb) info b
Num     Type           Disp Enb Address            What
1       breakpoint     keep y   0x0000000000400742 in Func(char const*) at test.cpp:8
2       breakpoint     keep y   0x000000000040071c in Func(char const*) at test.cpp:4
```
### 2.4 Debugging

Set up the breakpoints and then start running with ```run``` to begin debugging:
```
(gdb) run
Starting program: test 1

Breakpoint 2, Func (s=0x7fffffffe167 "/data/home/joelzychen/test/test") at test.cpp:4
4           int *p = nullptr;
```
Because we set a breakpoint at the beginning of the `Func` function, the program paused when entering the `Func` function; we can use ```list``` or ```l [function name]``` to view the context of the function:
```
(gdb) l Func
1       #include <iostream>
2
3       void Func(const char *s) {
4           int *p = nullptr;
5           int &r = static_cast<int&>(*p);
6
7           int num = std::atoi(s);
8           r = num;
9           printf("%d\n", r);
10      }
```
We can use ```print``` or ```p``` to view the variables as follows:

print(variable_name)
# or
p(variable_name)

```
(gdb) p s
$1 = 0x7fffffffe187 "1"
(gdb) p *s
$2 = 49 '1'
```
To move on to the next breakpoint, we can use either ```continue``` or ```c```:

c

```
(gdb) c
Continuing.

Breakpoint 2, Func (s=0x7fffffffe187 "1") at test.cpp:8
8           r = num;
```
View the variables in the current scope:
```
(gdb) p s
$8 = 0x7fffffffe187 "1"
(gdb) p p
$9 = (int *) 0x0
(gdb) p *p
Cannot access memory at address 0x0
(gdb) p r
$10 = (int &) @0x0: <error reading variable>
(gdb) p &r
$11 = (int *) 0x0
(gdb) p num
$12 = 1
```
Discover that `r` is an `int&` bound to a null pointer, so `&r == nullptr`, making it impossible to read the value of `r`. We can use `next` or `n` for single-step execution to see what happens next:
```
(gdb) n

Program received signal SIGSEGV, Segmentation fault.
0x0000000000400749 in Func (s=0x7fffffffe187 "1") at test.cpp:8
8           r = num;
```
Indeed, a segmentation fault occurred.

## 3. GDB Commands

According to the previous example, some commonly used GDB commands can be summarized.

### 3.1 Start

1. Connect GDB to a runnable file and start it.
   ```
   gdb [executable file]

   gdb
   (gdb) file [executable file]
   ```
2. Link GDB to a Running Process and Start It

bash
gdb /path/to/executable
(gdb) attach <process-id>
(gdb) continue

   ```
   gdb
   (gdb) attach [PID]
   ```
### 3.2 Breakpoints

1. Add Breakpoints

   - ```b [function name]```
   - ```b [file:line]```

2. View Breakpoints

   - ```info b```

3. Remove Breakpoints
   ```
   delete [breakpoint number]
   d [breakpoint number]
  clear # Clear the current breakpoint
   clear [function name] # Clear the breakpoint at a specific function
   clear [file:line] # Clear the breakpoint at a specific line
   ```
4. Disable and Enable Breakpoints
   ```
  disable # Disable all breakpoints
   disable [breakpoint number] # Disable a specific breakpoint
   enable # Enable all breakpoints
   enable [breakpoint number] # Enable a specific breakpoint
   ```
### 3.4 View

1. View the code
   - ```run```
2. Single Step Execution
   - `next` or `n`
3. Enter Function Internally
   - `step` or `s`
   - `stepi` Executes One Machine Instruction
   ```
  list # Print from beginning, default printing 10 lines at a time
   l # Same as above
   l [function name] # Start printing from the function definition
   l [file:line] # Start printing from a specific line
   set listsize 20 # Change the number of lines printed at once
   ```
2. Viewing Variables
   ```
  print [expression] # Print expression
   p [expression] # Print expression
capitalizations
   ptype [expression] # Print type of expression
   info args # Print function arguments
   info locals # Print local variables
   info registers # Print register information
   ```
### 3.5 Modification

1. Modify the variable
   ```

set variable i = 10 # set variable i to 10
set var i = 10 # same as above
p i = 10 # set variable i to 10 and print

   ```
### 3.6 Call Information

1. View the stack trace information
   - ```backtrace``` or ```bt``
   - ```where```
2. View the current frame
   - ```frame``` or ```f```

## 4. corefile

A core dump, crash dump, memory dump, or system dump refers to a snapshot of a program's memory at a specific time when the program crashes. It contains critical information such as registers (including program counter and stack pointer), memory management information, and operating system flags. A corefile is a snapshot taken during a dump, and it can be re-executed to debug error information.

### 4.1 Generation

To enable the system to generate a corefile, it is necessary to check the configuration:
```
$ ulimit -c
unlimited
```
If the result is 0, it means that the system prohibits the generation of the corefile, and you need to execute ```ulimit -c unlimited``` to allow the corefile to be generated normally. For the example given earlier, first run the test file to generate a corefile:
```
$ ./test 1
Core Dump (Segmentation Fault)
$ ll /data/corefile
-rw------- 1 joelzychen dev 450560 Apr 22 17:07 core_test_1587546447.28284
```
It generates a corefile named `core_test_1587546447.28284` in a specific directory (which can be modified). We use `gdb` to debug this corefile.

### 4.2 Debugging

To execute the corefile, you need to prepare the corresponding executable file. Run `gdb [executable file] [corefile]` to start debugging:
```
$ gdb test /data/corefile/core_test_1587546447.28284
...
Core was generated by `./test 1'.
Program terminated with signal 11, Segmentation fault.
#0  0x0000000000400749 in Func (s=0x7ffed098c19e "1") at test.cpp:8
8           r = num;
```
Because the example is relatively simple, the function callstack is also minimal. We can start by printing the function call stack information using ```bt```.
```
(gdb) bt
#0  0x0000000000400749 in Func (s=0x7ffed098c19e "1") at test.cpp:8
#1  0x00000000004007c0 in main (argc=2, argv=0x7ffed098afb8) at test.cpp:17
```
We enter the crash stack frame via `frame 0` or `f 0` to view the information:

f 0

```
(gdb) f 0
#0  0x0000000000400749 in Func (s=0x7ffed098c19e "1") at test.cpp:8
8           r = num;
(gdb) info args
s = 0x7ffed098c19e "1"
(gdb) info locals
p = 0x0
r = @0x0: <error reading variable>
num = 1
(gdb) ptype r
type = int &
(gdb) ptype p
type = int *
```
View the assembly code of the current stack frame:
```
(gdb) disas
Dump of assembler code for function Func(char const*):
   0x0000000000400710 <+0>:     push   %rbp
   0x0000000000400711 <+1>:     mov    %rsp,%rbp
   0x0000000000400714 <+4>:     sub    $0x20,%rsp
   0x0000000000400718 <+8>:     mov    %rdi,-0x8(%rbp)
   0x000000000040071c <+12>:    movq   $0x0,-0x10(%rbp)
   0x0000000000400724 <+20>:    mov    -0x10(%rbp),%rdi
   0x0000000000400728 <+24>:    mov    %rdi,-0x18(%rbp)
   0x000000000040072c <+28>:    mov    -0x8(%rbp),%rdi
   0x0000000000400730 <+32>:    callq  0x4005a0 <atoi@plt>
   0x0000000000400735 <+37>:    movabs $0x400860,%rdi
   0x000000000040073f <+47>:    mov    %eax,-0x1c(%rbp)
   0x0000000000400742 <+50>:    mov    -0x1c(%rbp),%eax
   0x0000000000400745 <+53>:    mov    -0x18(%rbp),%rcx
=> 0x0000000000400749 <+57>:    mov    %eax,(%rcx)
   0x000000000040074b <+59>:    mov    -0x18(%rbp),%rcx
   0x000000000040074f <+63>:    mov    (%rcx),%esi
   0x0000000000400751 <+65>:    mov    $0x0,%al
   0x0000000000400753 <+67>:    callq  0x400550 <printf@plt>
   0x0000000000400758 <+72>:    mov    %eax,-0x20(%rbp)
   0x000000000040075b <+75>:    add    $0x20,%rsp
   0x000000000040075f <+79>:    pop    %rbp
   0x0000000000400760 <+80>:    retq
End of assembler dump.
```
Viewing Register Status:
```
(gdb) i r
rax            0x1      1
rbx            0x0      0
rcx            0x0      0
rdx            0xa      10
rsi            0x0      0
rdi            0x400860 4196448
rbp            0x7ffed098aea0   0x7ffed098aea0
rsp            0x7ffed098ae80   0x7ffed098ae80
r8             0x7f9c88bbf060   140310285643872
r9             0x7ffed098c19f   140732398092703
r10            0x1      1
r11            0x0      0
r12            0x40061c 4195868
r13            0x7ffed098afb0   140732398088112
r14            0x0      0
r15            0x0      0
rip            0x400749 0x400749 <Func(char const*)+57>
eflags         0x10206  [ PF IF RF ]
cs             0x33     51
ss             0x2b     43
ds             0x0      0
es             0x0      0
fs             0x0      0
gs             0x0      0
```
## Summary

This excerpt demonstrates the basic usage of GDB in a Linux environment through an example. Common GDB commands are summarized, and steps for debugging C/C++ programs and corefiles are outlined. In actual application, GDB greatly enhances development and debugging efficiency. More usage tips require practice.

## Original references

- [Reference 1](https://zh.wikipedia.org/wiki/GNU)
- [Reference 2](https://en.wikipedia.org/wiki/Debugger)
- [Reference 3](https://en.wikipedia.org/wiki/Core_dump)
