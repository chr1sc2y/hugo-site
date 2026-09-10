---
title: "An Introduction to CMake"
date: 2020-06-21T15:46:06+08:00
draft: false
categories: ["C++"]
description: "A translated technical note on An Introduction to CMake, preserving the examples and context of the original article."
---
# An Introduction to CMake

> Originally published in Chinese on 2020-06-21; this English edition preserves the original scope and technical context.

## 0. Preface

CMake is an open-source, cross-platform build tool that makes managing the directory structure of multiple libraries and dependencies easy, and generates `makefile` that can be used with GNU `make` to compile and link a program.

## 1. Building a Single File

### 1.1 Using GCC to Compile

Assume that we wish to write a function to implement safe integer addition to prevent overflow, and this source file has no dependencies or static libraries:
```c++
// safe_add.cpp
#include <iostream>
#include <memory>
#define INT_MAX 2147483647
#define ERROR_DATA_OVERFLOW 2

int SafeIntAdd(std::unique_ptr<int> &sum, int a, int b)
{
    if (a > INT_MAX - b)
    {
        *sum = INT_MAX;
        return ERROR_DATA_OVERFLOW;
    }
    *sum = a + b;
    return EXIT_SUCCESS;
}

int main()
{
    int a, b;
    std::cin >> a >> b;
    std::unique_ptr<int> sum(new int(1));
    int res = SafeIntAdd(sum, a, b);
    std::cout << *sum << std::endl;
    return res;
}

```
We can simply compile and execute this file with a single straightforward `gcc` command:

bash
gcc -o output_file input_file.c
./output_file

```bash
[joelzychen@DevCloud ~/cmake-tutorial]$ g++ main.cc -g -Wall -std=c++11 -o SafeIntAdd
[joelzychen@DevCloud ~/cmake-tutorial]$ ./SafeIntAdd
2100000000 2100000000
2147483647
```
### 1.2 Building with CMake

If you wish to use CMake to generate the `makefile`, you need to first create a `CMakeLists.txt` file. All configurations for CMake are completed within this file. The content of `CMakeLists.txt` is typically as follows:

cmake
cmake_minimum_required(VERSION 3.10)
project(MyProject)

set(CMAKE_BUILD_TYPE Release)
set(CMAKE_CXX_STANDARD 14)

add_executable(MyExecutable main.cpp)


cmake
# CMakeLists.txt
cmake_minimum_required(VERSION 3.10)
project(MyProject)

set(CMAKE_BUILD_TYPE Release)
set(CMAKE_CXX_STANDARD 14)

add_executable(MyExecutable main.cpp)

```cmake
cmake_minimum_required(VERSION 3.10)

project(SafeIntAdd)

set(CMAKE_CXX_COMPILER "c++")
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED True)
set(CMAKE_CXX_FLAGS -g -Wall)
message(STATUS "CMAKE_CXX_FLAGS: " "${CMAKE_CXX_FLAGS}")
string(REPLACE ";" " " CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS}")
message(STATUS "CMAKE_CXX_FLAGS: " "${CMAKE_CXX_FLAGS}")

add_executable(SafeIntAdd main.cc)
```
Here are some basic `cmake` commands with their meanings as follows:

1. `cmake_minimum_required`：Specify the minimum version requirement for `cmake`
2. `project`：Specify the project name
3. `set`：Set ordinary variables, cached variables, or environment variables, as in the above example's
4. `add_executable`：Build an executable from the listed source files

There are a few points to note:

The `CMAKE_MINIMUM_REQUIRED` directive is case-insensitive; it can be written as `CMAKE_MINIMUM_REQUIRED`, `cmake_minimum_required`, or even `cmAkE_mInImUm_rEquIrEd` (though the latter is not recommended).

2. When using the `set` command to specify `CMAKE_CXX_FLAGS`, options are separated by spaces, resulting in `CMAKE_CXX_FLAGS` string as `-g;-Wall`. This needs to be replaced with spaces to `-g -Wall`.

3. `message` can output some information to `stdout` during the build process. The example output information from the previous snippet is:
   ```bash
   -- CMAKE_CXX_FLAGS: -g;-Wall
   -- CMAKE_CXX_FLAGS: -g -Wall
   ```
4. Similar to bash scripts, when outputting variables in CMakeLists.txt, one should use "${CMAKE_CXX_FLAGS}" instead of directly using CMAKE_CXX_FLAGS.

After editing the CMakeLists.txt, we can create a build directory and use cmake within it to build. If the build is successful, we can then use make to compile and link, ultimately resulting in the executable named SafeAdd.
```bash
[joelzychen@DevCloud ~/cmake-tutorial]$ mkdir build/
[joelzychen@DevCloud ~/cmake-tutorial]$ cd build/
[joelzychen@DevCloud ~/cmake-tutorial/build]$ cmake ..
-- The C compiler identification is GNU 4.8.5
-- The CXX compiler identification is GNU 4.8.5
-- Check for working C compiler: /usr/bin/cc
-- Check for working C compiler: /usr/bin/cc - works
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Detecting C compile features
-- Detecting C compile features - done
-- Check for working CXX compiler: /usr/bin/c++
-- Check for working CXX compiler: /usr/bin/c++ - works
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Detecting CXX compile features
-- Detecting CXX compile features - done
-- CMAKE_CXX_FLAGS: -g;-Wall
-- CMAKE_CXX_FLAGS: -g -Wall
-- Configuring done
-- Generating done
-- Build files have been written to: /home/joelzychen/cmake-tutorial/build
[joelzychen@DevCloud ~/cmake-tutorial/build]$ make
Scanning dependencies of target SafeIntAdd
[ 50%] Building CXX object CMakeFiles/SafeIntAdd.dir/main.cc.o
[100%] Linking CXX executable SafeIntAdd
[100%] Built target SafeIntAdd
[joelzychen@DevCloud ~/cmake-tutorial/build]$ ./SafeIntAdd
2100000000 2100000000
2147483647
```
## Building Multiple Files

### 2.1 Using GCC Compiler

Assuming we wish to place the addition function in a separate file and include this file in the source file where the main function resides:
```c++
// main.cc
#include "math.h"
#include "error_code.h"
#include <iostream>

int main()
{
    int a{ 0 }, b{ 0 }, c{ 0 };
    std::cin >> a >> b >> c;
    int sum{ 0 };
    int ret_val = SafeAdd(sum, a, b, c);
    std::cout << sum << std::endl;
    return ret_val;
}

```
```c++
// util/math.h
#ifndef UTIL_MATH_H
#define UTIL_MATH_H

#include "error_code.h"
#include <limits>

template<typename ValueType>
ValueType ValueTypeMax(ValueType)
{
    return std::numeric_limits<ValueType>::max();
}

template<typename ValueType>
int SafeAdd(ValueType &sum)
{
    return exit_success;
}

template<typename ValueType, typename ...ValueTypes>
int SafeAdd(ValueType &sum, const ValueType &value, const ValueTypes &...other_values)
{
    int ret_val = SafeAdd<ValueType>(sum, other_values...);
    if (ret_val != exit_success)
    {
        return ret_val;
    }
    if (sum > ValueTypeMax(value) - value)
    {
        sum = ValueTypeMax(value);
        return error_data_overflow;
    }
    sum += value;
    return exit_success;
}

#endif
```
```c++
// definition/error_code.h
#ifndef DEFINITION_ERROR_CODE_H
#define DEFINITION_ERROR_CODE_H

constexpr int exit_success = 0;
constexpr int exit_failure = 1;
constexpr int error_data_overflow = 2;

#endif
```
We can use the `-I` parameter with GCC to specify the directories containing the header files:
```bash
[joelzychen@DevCloud ~/safe_add]$ g++ -g -Wall -std=c++11 -Ilib -Idefinition -o SafeAdd main.cc
[joelzychen@DevCloud ~/safe_add]$ ./SafeAdd
20000 50000 80000
150000
```
### 2.2 Building with CMake
```cmake
cmake_minimum_required(VERSION 3.10)

project(SafeIntAdd)

set(CMAKE_CXX_COMPILER "c++")
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED True)
set(CMAKE_CXX_FLAGS -g -Wall)
string(REPLACE ";" " " CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS}")

include_directories(lib/ definition/)

aux_source_directory(./ SOURCE_DIR)

add_executable(SafeIntAdd ${SOURCE_DIR})
```
Instead of building a single file, we used two commands:

1. `include_directories`：Add multiple directories for header file search paths; paths are separated by spaces. If both `lib` and `definition` directories are added to the search path, relative paths are not needed when using `include`.
2. `aux_source_directory`：Search all source files in a directory and store these files in the variable `SOURCE_DIR`; Note that this command does not recursively include subdirectories.

Next, build within the `build` directory:
```bash
[joelzychen@DevCloud ~/cmake-tutorial/build]$ rm -rf *
[joelzychen@DevCloud ~/cmake-tutorial/build]$ cmake ..
-- The C compiler identification is GNU 4.8.5
-- The CXX compiler identification is GNU 4.8.5
-- Check for working C compiler: /usr/bin/cc
-- Check for working C compiler: /usr/bin/cc - works
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Detecting C compile features
-- Detecting C compile features - done
-- Check for working CXX compiler: /usr/bin/c++
-- Check for working CXX compiler: /usr/bin/c++ - works
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Detecting CXX compile features
-- Detecting CXX compile features - done
-- Configuring done
-- Generating done
-- Build files have been written to: /home/joelzychen/cmake-tutorial/build
[joelzychen@DevCloud ~/cmake-tutorial/build]$ make
Scanning dependencies of target SafeIntAdd
[ 50%] Building CXX object CMakeFiles/SafeIntAdd.dir/main.cc.o
[100%] Linking CXX executable SafeIntAdd
[100%] Built target SafeIntAdd
[joelzychen@DevCloud ~/cmake-tutorial/build]$ ./SafeIntAdd
2000000000 1900000000
2147483647
```
## 3. Build a Project Dependent on a Static Library

About static libraries and dynamic libraries, please refer to [A Deep Look into Static Libraries and Dynamic Libraries](https://zhuanlan.zhihu.com/p/71372182).

### 3.1 Using GCC to Compile Static Library Files

Assuming we wish to encapsulate the `exp2` function into a singleton calculator class and compute the nth power of 2, we have written the following files for this purpose:
```c++
// main.cc
#include "util/calculator.h"
#include "definition/error_code.h"
#include <iostream>

int main()
{
    double a{ 0 };
    std::cin >> a;
    double exp2{ 0 };
    int ret_val = Calculator::GetInstance().Exp2(exp2, a);
    if (ret_val != exit_success)
    {
        return ret_val;
    }
    std::cout << exp2 << std::endl;
    return exit_success;
}

```
```c++
// util/singleton.h
#ifndef UTIL_SINGLETON_H
#define UTIL_SINGLETON_H

template <typename T>
class Singleton
{
public:
    static T& GetInstance()
    {
        static T instance;
        return instance;
    }
    Singleton(Singleton const &) = delete;
    Singleton& operator=(Singleton const &) = delete;

protected:
    Singleton() = default;
    ~Singleton() = default;

};

#endif
```
```cpp
// definition/error_code.h
#ifndef DEFINITION_ERROR_CODE_H
#define DEFINITION_ERROR_CODE_H

constexpr int exit_success = 0;
constexpr int exit_failure = 1;
constexpr int error_data_overflow = 2;

#endif
```
```c++
// util/calculator.h
#ifndef UTIL_CALCULATOR_H
#define UTIL_CALCULATOR_H

#include "singleton.h"
#include "../definition/error_code.h"
#include <limits>
#include <cmath>

class Calculator : public Singleton<Calculator>
{
public:
    template<typename ValueType>
    ValueType ValueTypeMax(ValueType)
    {
        return std::numeric_limits<ValueType>::max();
    }

    int Exp2(double &exp2, const double &val);
};

#endif
```
For ease of generating a static library file, we first place the `calculator.cc` file in the `archive` directory:
```c++
// archive/calculator.cc
#include "../util/calculator.h"

int Calculator::Exp2(double &exp2, const double &val)
{
    if (std::sqrt(ValueTypeMax(val)) < val)
    {
        exp2 = ValueTypeMax(val);
        return error_data_overflow;
    }
    exp2 = std::exp2(val);
    return exit_success;
}

```
For the `calculator.cc` file, we can compile it into an `obj` file using the `-c` parameter and then archive it to create a `.a` library file using `ar`:
```bash
[joelzychen@DevCloud ~/cmake-tutorial/archive]$ g++ calculator.cc -g -Wall -std=c++11 -c
[joelzychen@DevCloud ~/cmake-tutorial/archive]$ ar -crv libcalculator.a calculator.o
a - calculator.o
```
### 3.2 Using GCC to Compile the Project and Link the Static Library

The current directory structure is as follows:
```bash
[joelzychen@DevCloud ~/cmake-tutorial]$ tree
.
|-- archive
|   |-- calculator.cc
|   |-- calculator.o
|   `-- libcalculator.a
|-- CMakeLists.txt
|-- definition
|   `-- error_code.h
|-- main.cc
`-- util
    |-- calculator.h
    `-- singleton.h

3 directories, 8 files
```
Using GCC to compile and link against the corresponding static library yields an executable file:
```bash
[joelzychen@DevCloud ~/cmake-tutorial]$ g++ main.cc -g -Wall -std=c++11 -o Exp2 -Larchive -lcalculator
[joelzychen@DevCloud ~/cmake-tutorial]$ ll Exp2
32K -rwxrwxr-x 1 joelzychen joelzychen 31K Jun 21 14:38 Exp2
[joelzychen@DevCloud ~/cmake-tutorial]$ ./Exp2
8
256
```
Noted points include:

1. `-Llib` specifies the directory for library files
2. `-lsingleton` specifies the library file
3. `-L` and `-l` parameters must come after `-o` parameter

### 3.3 Building Projects Using CMake and Linking Static Libraries

We have already compiled the static library with GCC, so we can add a command to link the static library in the `CMakeLists.txt` as follows:
```cmake
cmake_minimum_required(VERSION 3.10)

project(Exp2)

set(CMAKE_CXX_COMPILER "c++")
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED True)
set(CMAKE_CXX_FLAGS -g -Wall)
string(REPLACE ";" " " CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS}")
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY bin/)

message(STATUS "source dir: " ${PROJECT_SOURCE_DIR})
message(STATUS "binary dir: " ${PROJECT_BINARY_DIR})
message(STATUS "output dir: " ${CMAKE_RUNTIME_OUTPUT_DIRECTORY})

include_directories(./)
aux_source_directory(./ SOURCE_DIR)

link_directories(archive/)
add_executable(Exp2 ${SOURCE_DIR})
target_link_libraries(Exp2 libcalculator.a)
# target_link_libraries(Exp2 calculator)
```
And the additional instructions used compared to before are:

1. `link_directories`：Specify the search paths for static or dynamic libraries
2. `target_link_libraries`：Link specified static libraries to the target executable, `singleton` and `libsingleton.a` are equivalent forms

Then enter the `build` directory for building:
```bash
[joelzychen@DevCloud ~/cmake-tutorial/build]$ rm -rf *
[joelzychen@DevCloud ~/cmake-tutorial/build]$ cmake ../
-- The C compiler identification is GNU 4.8.5
-- The CXX compiler identification is GNU 4.8.5
-- Check for working C compiler: /usr/bin/cc
-- Check for working C compiler: /usr/bin/cc - works
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Detecting C compile features
-- Detecting C compile features - done
-- Check for working CXX compiler: /usr/bin/c++
-- Check for working CXX compiler: /usr/bin/c++ - works
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Detecting CXX compile features
-- Detecting CXX compile features - done
-- source dir: /home/joelzychen/cmake-tutorial
-- binary dir: /home/joelzychen/cmake-tutorial/build
-- output dir: /home/joelzychen/cmake-tutorial/build/bin/
-- Configuring done
-- Generating done
-- Build files have been written to: /home/joelzychen/cmake-tutorial/build
[joelzychen@DevCloud ~/cmake-tutorial/build]$ make
Scanning dependencies of target Exp2
[ 50%] Building CXX object CMakeFiles/Exp2.dir/main.cc.o
[100%] Linking CXX executable bin/Exp2
[100%] Built target Exp2
[joelzychen@DevCloud ~/cmake-tutorial/build]$ ll bin/Exp2
32K -rwxrwxr-x 1 joelzychen joelzychen 31K Jun 21 14:58 bin/Exp2
[joelzychen@DevCloud ~/cmake-tutorial/build]$ ./bin/Exp2
6
64
```
One can see that the size of the binary files generated through CMake is the same as those compiled and linked directly with GCC. This is achieved by setting `CMAKE_RUNTIME_OUTPUT_DIRECTORY` to `bin/`, which places the generated binary files in the `bin` directory. Note that the `bin` directory is the build directory created by CMake (`PROJECT_BINARY_DIR`), not the directory where the `CMakeLists.txt` file resides (`PROJECT_SOURCE_DIR`).

### 3.4 Building Static Library Files and Projects Using CMake

Apart from directly referencing static libraries from external sources, CMake can also compile the source files into static libraries before building:
```cmake
cmake_minimum_required(VERSION 3.10)

set(project_name Exp2)
project(${project_name})

set(CMAKE_CXX_COMPILER "c++")
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED True)
set(CMAKE_CXX_FLAGS -g -Wall)
string(REPLACE ";" " " CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS}")
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY bin/)

message(STATUS "source dir: " ${PROJECT_SOURCE_DIR})
message(STATUS "binary dir: " ${PROJECT_BINARY_DIR})
message(STATUS "output dir: " "${PROJECT_BINARY_DIR}/${CMAKE_RUNTIME_OUTPUT_DIRECTORY}")

include_directories(./)
aux_source_directory(./ SOURCE_DIR)

set(static_lib_source_file archive/calculator.cc)
add_library(calculator_static STATIC ${static_lib_source_file})
add_executable(${project_name} ${SOURCE_DIR})
target_link_libraries(${project_name} calculator_static)
```
Here, a new `add_library` instruction is used to generate a library from specified source files, and then `target_link_libraries` is used to add the generated library to the project. Next, the build is performed:
```bash
[joelzychen@DevCloud ~/cmake-tutorial/build]$ rm -rf *
[joelzychen@DevCloud ~/cmake-tutorial/build]$ cmake ../
-- The C compiler identification is GNU 4.8.5
-- The CXX compiler identification is GNU 4.8.5
-- Check for working C compiler: /usr/bin/cc
-- Check for working C compiler: /usr/bin/cc - works
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Detecting C compile features
-- Detecting C compile features - done
-- Check for working CXX compiler: /usr/bin/c++
-- Check for working CXX compiler: /usr/bin/c++ - works
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Detecting CXX compile features
-- Detecting CXX compile features - done
-- source dir: /home/joelzychen/cmake-tutorial
-- binary dir: /home/joelzychen/cmake-tutorial/build
-- output dir: /home/joelzychen/cmake-tutorial/build/bin/
-- Configuring done
-- Generating done
-- Build files have been written to: /home/joelzychen/cmake-tutorial/build
[joelzychen@DevCloud ~/cmake-tutorial/build]$ make
Scanning dependencies of target calculator_static
[ 25%] Building CXX object CMakeFiles/calculator_static.dir/archive/calculator.cc.o
[ 50%] Linking CXX static library libcalculator_static.a
[ 50%] Built target calculator_static
Scanning dependencies of target Exp2
[ 75%] Building CXX object CMakeFiles/Exp2.dir/main.cc.o
[100%] Linking CXX executable bin/Exp2
[100%] Built target Exp2
[joelzychen@DevCloud ~/cmake-tutorial/build]$ ll | grep calculator
 12K -rw-rw-r-- 1 joelzychen joelzychen  11K Jun 21 15:30 libcalculator_static.a
[joelzychen@DevCloud ~/cmake-tutorial/build]$ ./bin/Exp2
5
32
```
One can see that a static library file named `libcalculator_static.a` has been generated in the `build` directory. This name is specified by the first argument of the `add_library` command.

## 4. Build Projects Dependent on Dynamic Libraries

### 4.1 Using GCC to Compile Dynamic Library Files

Still use the existing file in the `archive` directory to generate a dynamic library file:
```bash
[joelzychen@DevCloud ~/cmake-tutorial/archive]$ rm calculator.o libcalculator.a
[joelzychen@DevCloud ~/cmake-tutorial/archive]$ g++ calculator.cc -g -Wall -std=c++11 -c -fPIC
[joelzychen@DevCloud ~/cmake-tutorial/archive]$ g++ calculator.o -g -Wall -std=c++11 -shared -o libcalculator.so
```
Divide into two steps:

1. Use `-c` and `-fPIC` to generate position-independent code `.o` files
2. Use `-shared` to generate `.so` shared library files

Also combine it into one step:
```bash
[joelzychen@DevCloud ~/cmake-tutorial/archive]$ g++ calculator.cc -g -Wall -std=c++11 -shared -fPIC -o libcalculator.so
```
### 4.2 Using GCC to Compile the Project and Link a Dynamic Library

And similarly to linking a static library, one can generate the binary file by linking the dynamic library after compiling:
```bash
[joelzychen@DevCloud ~/cmake-tutorial]$ g++ main.cc -g -Wall -std=c++11 -o Exp2 -Larchive -lcalculator
[joelzychen@DevCloud ~/cmake-tutorial]$ ./Exp2
./Exp2: error while loading shared libraries: libcalculator.so: cannot open shared object file: No such file or directory
```
We discovered that a "no such dynamic library" error occurs when running a binary file, which is due to the dynamic library not being found in the directory specified in the environment variables. We can either add the corresponding directory to the environment variables or copy the dynamic library to the directory specified in the environment variables:
```bash
[joelzychen@DevCloud ~/cmake-tutorial]$ echo $LD_LIBRARY_PATH

[joelzychen@DevCloud ~/cmake-tutorial]$ export LD_LIBRARY_PATH="/usr/lib/"
[joelzychen@DevCloud ~/cmake-tutorial]$ sudo cp archive/libcalculator.so /usr/lib/
```
Next, you can run the executable file. It is seen that the executable file generated using the dynamic library linkage is smaller than the executable file generated using static library linkage. You can also use the `ldd` command to verify that the executable file correctly references the corresponding dynamic library file.
```bash
[joelzychen@DevCloud ~/cmake-tutorial]$ ./Exp2
9
512
[joelzychen@DevCloud ~/cmake-tutorial]$ ll Exp2
28K -rwxrwxr-x 1 joelzychen joelzychen 28K Jun 21 14:25 Exp2
[joelzychen@DevCloud ~/cmake-tutorial]$ ldd Exp2 | grep calculator
        libcalculator.so => /usr/lib/libcalculator.so (0x00007fd2391f0000)
```
### 4.3 Building the Project and Linking a Dynamic Library Using CMake

And just like building a static library, you can build a dynamic library by modifying the `CMakeLists.txt` instructions to link a dynamic library instead.
```cmake
cmake_minimum_required(VERSION 3.10)

project(Exp2)

set(CMAKE_CXX_COMPILER "c++")
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED True)
set(CMAKE_CXX_FLAGS -g -Wall)
string(REPLACE ";" " " CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS}")
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY bin/)

message(STATUS "source dir: " ${PROJECT_SOURCE_DIR})
message(STATUS "binary dir: " ${PROJECT_BINARY_DIR})
message(STATUS "output dir: " "${PROJECT_BINARY_DIR}/${CMAKE_RUNTIME_OUTPUT_DIRECTORY}")

include_directories(./)
aux_source_directory(./ SOURCE_DIR)

link_directories(archive/)
add_executable(Exp2 ${SOURCE_DIR})
target_link_libraries(Exp2 libcalculator.so)
```
```bash
[joelzychen@DevCloud ~/cmake-tutorial/build]$ rm -rf *
[joelzychen@DevCloud ~/cmake-tutorial/build]$ cmake ../
-- The C compiler identification is GNU 4.8.5
-- The CXX compiler identification is GNU 4.8.5
-- Check for working C compiler: /usr/bin/cc
-- Check for working C compiler: /usr/bin/cc - works
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Detecting C compile features
-- Detecting C compile features - done
-- Check for working CXX compiler: /usr/bin/c++
-- Check for working CXX compiler: /usr/bin/c++ - works
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Detecting CXX compile features
-- Detecting CXX compile features - done
-- source dir: /home/joelzychen/cmake-tutorial
-- binary dir: /home/joelzychen/cmake-tutorial/build
-- output dir: /home/joelzychen/cmake-tutorial/build/bin/
-- Configuring done
-- Generating done
-- Build files have been written to: /home/joelzychen/cmake-tutorial/build
[joelzychen@DevCloud ~/cmake-tutorial/build]$ make
Scanning dependencies of target Exp2
[ 50%] Building CXX object CMakeFiles/Exp2.dir/main.cc.o
[100%] Linking CXX executable bin/Exp2
[100%] Built target Exp2
[joelzychen@DevCloud ~/cmake-tutorial/build]$ ll bin/Exp2
28K -rwxrwxr-x 1 joelzychen joelzychen 28K Jun 21 15:04 bin/Exp2
[joelzychen@DevCloud ~/cmake-tutorial/build]$ ldd bin/Exp2 | grep calculator
        libcalculator.so => /home/joelzychen/cmake-tutorial/archive/libcalculator.so (0x00007f4c87e35000)
[joelzychen@DevCloud ~/cmake-tutorial/build]$ ./bin/Exp2
6
64
```
One can see that the size of the binary file generated through CMake is the same as that obtained by directly compiling and linking a dynamic library with GCC.

### 4.4 Building Dynamic Library Files and Projects Using CMake

Steps for building a dynamic library using CMake are nearly identical to building a static library, except you simply need to change the `STATIC` parameter of `add_library` to `SHARED`:
```cmake
cmake_minimum_required(VERSION 3.10)

set(project_name Exp2)
project(${project_name})

set(CMAKE_CXX_COMPILER "c++")
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED True)
set(CMAKE_CXX_FLAGS -g -Wall)
string(REPLACE ";" " " CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS}")
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY bin/)

message(STATUS "source dir: " ${PROJECT_SOURCE_DIR})
message(STATUS "binary dir: " ${PROJECT_BINARY_DIR})
message(STATUS "output dir: " "${PROJECT_BINARY_DIR}/${CMAKE_RUNTIME_OUTPUT_DIRECTORY}")

include_directories(./)
aux_source_directory(./ SOURCE_DIR)

set(shared_lib_source_file archive/calculator.cc)
add_library(calculator_shared SHARED ${shared_lib_source_file})
add_executable(${project_name} ${SOURCE_DIR})
target_link_libraries(${project_name} calculator_shared)
```
```bash
[joelzychen@DevCloud ~/cmake-tutorial/build]$ rm -rf *
[joelzychen@DevCloud ~/cmake-tutorial/build]$ cmake ..
-- The C compiler identification is GNU 4.8.5
-- The CXX compiler identification is GNU 4.8.5
-- Check for working C compiler: /usr/bin/cc
-- Check for working C compiler: /usr/bin/cc - works
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Detecting C compile features
-- Detecting C compile features - done
-- Check for working CXX compiler: /usr/bin/c++
-- Check for working CXX compiler: /usr/bin/c++ - works
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Detecting CXX compile features
-- Detecting CXX compile features - done
-- source dir: /home/joelzychen/cmake-tutorial
-- binary dir: /home/joelzychen/cmake-tutorial/build
-- output dir: /home/joelzychen/cmake-tutorial/build/bin/
-- Configuring done
-- Generating done
-- Build files have been written to: /home/joelzychen/cmake-tutorial/build
[joelzychen@DevCloud ~/cmake-tutorial/build]$ make
Scanning dependencies of target calculator_shared
[ 25%] Building CXX object CMakeFiles/calculator_shared.dir/archive/calculator.cc.o
[ 50%] Linking CXX shared library libcalculator_shared.so
[ 50%] Built target calculator_shared
Scanning dependencies of target Exp2
[ 75%] Building CXX object CMakeFiles/Exp2.dir/main.cc.o
[100%] Linking CXX executable bin/Exp2
[100%] Built target Exp2
[joelzychen@DevCloud ~/cmake-tutorial/build]$ ll | grep calculator
 16K -rwxrwxr-x 1 joelzychen joelzychen  13K Jun 21 15:34 libcalculator_shared.so
[joelzychen@DevCloud ~/cmake-tutorial/build]$ ll bin/Exp2
28K -rwxrwxr-x 1 joelzychen joelzychen 28K Jun 21 15:34 bin/Exp2
[joelzychen@DevCloud ~/cmake-tutorial/build]$ ldd bin/Exp2 | grep calculator
        libcalculator_shared.so => /home/joelzychen/cmake-tutorial/build/libcalculator_shared.so (0x00007f1fa5aa5000)
[joelzychen@DevCloud ~/cmake-tutorial/build]$ ./bin/Exp2
7
128
```
One can see that the information and usage of the binary file obtained through various methods are the same.

## 5. Common Commands

### 5.1 Project Related

- `cmake_minimum_required(VERSION 3.10)`：Specify the minimum version requirement for CMake
- `project(project_name)`：Specify the project name
- `set(CMAKE_CXX_FLAGS -g -Wall -pthread)`：Set the variable `CMAKE_CXX_FLAGS`

### 5.2 Compilation Related

- `include_directories`: Specifies the directories of header files, equivalent to the `-I` parameter in GCC compilation; this adds the header file directories to the `CPLUS_INCLUDE_PATH` environment variable
- `aux_source_directory`: Searches for all source files in the directory (excluding subdirectories) and stores these files in the variable `SOURCE_DIR`
- `add_executable`: Builds an executable using the listed source files
- `add_definitions`: Macro definitions

### 5.3 Link Related

- `link_directories`: Specifies the directories of the library files, equivalent to the `-L` parameter in GCC linking; this is equivalent to adding the directories of the library files to the `LD_LIBRARY_PATH` environment variable
- `target_link_libraries`: Links the specified static library to the target executable, in forms of `singleton` and `libsingleton.a` are equivalent

## 6. Summary

This excerpt compares the basic steps of compiling and building a program on Linux using GCC and CMake through several examples. It introduces some fundamental CMake commands. In actual development, projects are often very large, making it impractical to compile the entire project directly using GCC. In such scenarios, using CMake for building can save time and improve efficiency, allowing us to focus on project development. When writing `CMakeLists.txt`, one often encounters numerous issues, unfamiliar commands, and other usage tips. These require further learning through tutorials and practice, such as [here](https://cmake.org/cmake/help/latest/index.html).

## Original references

- [Reference 1](https://cmake.org/cmake/help/latest/command/add_library.html?highlight=add_library)
