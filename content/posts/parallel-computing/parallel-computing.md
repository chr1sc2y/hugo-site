---
title: "An Introduction to Parallel Computing"
date: 2020-08-12T18:32:52+08:00
draft: false
categories: ["Parallel Computing"]
description: "A translated technical note on An Introduction to Parallel Computing, preserving the examples and context of the original article."
---
# An Introduction to Parallel Computing

> Originally published in Chinese on 2020-08-12; this English edition preserves the original scope and technical context.

## 1 Overview

### 1.1 Parallel Computing

**High Performance Computing** (HPC) is a subfield of computer science focused on **performance optimization**. It encompasses techniques such as caching, data structures, algorithms, I/O optimization, instruction reordering, and compiler optimizations;

**Parallel Computing** is a subfield of HPC, focusing on breaking down complex problems into smaller parts that can be computed independently by separate processors (computational resources) to improve efficiency; Different problems require specialized parallel architectures, which can be dedicated hardware with multiple processors or clusters of independent computers connected in a specific manner.

### 1.2 Hardware Architecture

**Central Processing Unit** (CPU) is the primary component responsible for executing computer instructions, consisting of a **control unit** (CU), **arithmetic logic unit** (ALU), **out-of-order control unit** (OoO CU), **branch predictor** (BP), and **data cache** (DC); Designed to efficiently handle various general computing tasks and minimize latency, CPUs are constrained by their concurrency (clock frequency) in terms of parallelism;

![cpu](https://raw.githubusercontent.com/chr1sc2y/warehouse-deprecated/refs/heads/main/resources/parallel-computing/cpu.png)

**Graphics Processing Unit** (GPU) was introduced by NVIDIA in August 1999 with the release of [NVIDIA GeForce 256](https://zh.wikipedia.org/wiki/NVIDIA_GeForce_256); The modern GPU model can be summarized by several key points:

1. Designed to maximize throughput
2. Able to transfer data that can be computed in parallel from the CPU to the GPU
3. Capable of executing computations using a large number of threads
**GPU** possesses significantly more cores compared to CPUs, allowing for thousands of concurrent kernels to execute large-scale parallel computations. Originally specialized for handling graphical data, its powerful parallel processing capabilities have made it popular in the field of deep learning over the past few decades;

![gpu](https://raw.githubusercontent.com/chr1sc2y/warehouse-deprecated/refs/heads/main/resources/parallel-computing/gpu.png)

Subject to the limitations of manufacturing processes, the density and maximum area of chips are finite (Moore's Law), leading to a trade-off between functionality and component count in chip design; to meet the requirement of general-purpose computing, the design of CPU chips must use a variety of components to increase functionality while reducing the number of complex-function components. In contrast, the design of **GPU** chips sacrifices some complex-function components to gain more space and integrate more basic-function components;

![process](https://raw.githubusercontent.com/chr1sc2y/warehouse-deprecated/refs/heads/main/resources/parallel-computing/process.png)

A **GPU** device consists of multiple processor clusters, each associated with a **control unit** and L1 Cache. This design enables the simultaneous execution of hundreds of instruction streams on a single chip; Typically, before exchanging data with the global GDDR-5 memory, a **streaming multiprocessor** will use its associated L1 Cache and L2 Cache to reduce data transfer latency; Given the substantial computational workload of **GPU**, it does not need to frequently fetch data from memory like **CPU**, resulting in smaller cache layers for **GPU** compared to **CPU**.
![cpu-gpu](https://raw.githubusercontent.com/chr1sc2y/warehouse-deprecated/refs/heads/main/resources/parallel-computing/cpu-gpu.png)

With the CPU, GPUs can use fewer and relatively smaller cache layers. This is because GPUs have more transistors dedicated to computation, meaning they don't need to worry about how long it takes to fetch data from memory. As long as the GPU has enough computational work to do, it can mask the potential "wait time" for memory accesses, keeping it busy.

![gpu-vs-cpu](https://raw.githubusercontent.com/chr1sc2y/warehouse-deprecated/refs/heads/main/resources/parallel-computing/gpu-vs-cpu.png)

## 2 Concepts

### 2.1 Memory Model

In a computer with a large number of cores, each core has its own processor and cache; in contrast, processors and storage located on a network or in other nodes are referred to as global. Depending on the network interconnect and the way of accessing memory, a shared memory machine can be categorized into the following types:

1. Uniform Memory Access
**Uniform Memory Access** (UMA) model features include local caches (L1 Cache, L2 Cache) for all processors, uniform sharing of physical storage among processors, and equal access time for any storage word from any processor.

![uma](https://raw.githubusercontent.com/chr1sc2y/warehouse-deprecated/refs/heads/main/resources/parallel-computing/uma.png)

2. **Non-Uniform Memory Access** (NUMA)

    The **Non-Uniform Memory Access** model has distributed shared memory, with all local memories forming a global address space. Accessing local memory by a processor is faster than accessing global memory (shared memory) or the local memory of another processor. The latency of memory access depends on the location of the memory relative to the processor.

    Different processors access shared memory with varying locations, leading to different access delays.

3. **Cache-Only Memory Architecture** (COMA)

    **Cache-Only Memory Architecture** replaces distributed memory in NUMA with cache. Each processor has no storage hierarchy; all caches form a global address space.

    ![numa](https://raw.githubusercontent.com/chr1sc2y/warehouse-deprecated/refs/heads/main/resources/parallel-computing/numa.png)

### 2.2 **Flynn Classification**

Flynn's Taxonomy classifies high-performance computers into four categories based on how instructions and data are executed:

1. **Single Instruction Single Data Model** (SISD)

Generally, a computer with a single-core CPU (excluding hyper-threading technology) operates based on the Single Instruction Single Data (SISD) model. For each CPU clock cycle, the CPU executes the instruction in the sequence of **Fetch** (retrieving data from a register), **Decode** (decoding), and **Execute** (executing and saving the result in another register). Most computers from the previous century were SISD models.
2. **Single Instruction Multi Data (SIMD)**

   A single control unit with multiple processors, where these processors run threads that share the same instruction stream, achieving parallelism in time; **GPU** is a typical SIMD model.

3. **Multi Instruction Single Data (MISD)**

   Multiple processors each with its own control unit and sharing the same memory unit, with less common applications.

4. **Multi Instruction Multi Data (MIMD)**

   Multiple control units asynchronously control multiple processors, allowing processors to run different programs on different data, generally achieved through parallelism at the thread or process level, thus achieving spatial parallelism.

### 2.3 Speedup

1. **Speedup**

    **Speedup** measures how much faster our parallel algorithm is compared to the serial algorithm, i.e., the efficiency gained by parallelizing the algorithm. The formula is:

    ![speedup](https://raw.githubusercontent.com/chr1sc2y/warehouse-deprecated/refs/heads/main/resources/parallel-computing/speedup.svg)

    Where \( p \) represents the number of CPUs, \( T_1 \) is the execution time with serial algorithm, and \( T_p \) is the execution time with parallel algorithm when \( p \) CPUs are used; when \( S_p = p \), i.e., \( T_1 = p \times T_p \), \( S_p \) is called **linear speedup** (Linear Speedup).

2. **Amdahl's Law**

    ![amdahls-law](https://raw.githubusercontent.com/chr1sc2y/warehouse-deprecated/refs/heads/main/resources/parallel-computing/amdahls-law.svg)

    **Amdahl's Law** estimates the maximum speedup achievable by a program. \( W_s \) and \( W_p \) represent the percentage of the program that is serial and parallel, respectively. \( W_s + W_p \) represents the time taken by the serial part of the program, and \( W_s + W_p/p \) represents the time taken by the program with \( p \) processors. When \( p \to \infty \), its upper limit is \( (W_s + W_p) / W_s \).
    ```c++
    for (int i = 0; i < 1000000000; ++i) std::this_thread::sleep_for(std::chrono::seconds(1));   // sequential
    for (int i = 0; i < 1000000000; ++i) std::this_thread::sleep_for(std::chrono::seconds(1));   // parallel
    ```
![amdahl](https://raw.githubusercontent.com/chr1sc2y/warehouse-deprecated/refs/heads/main/resources/parallel-computing/amdahl's-law.png)

3. Gustafson's Law

Gustafson's Law describes the speedup with \( p \) representing the number of processors and \( a \) representing the serialized portion of the program.

![gustafson](https://raw.githubusercontent.com/chr1sc2y/warehouse-deprecated/refs/heads/main/resources/parallel-computing/gustafson.png)

Amdahl's Law states that increasing the number of processors does not necessarily improve the speedup. Only increasing the proportion of the program that can be parallelized can improve the speedup.

Gustafson's Law states that as the parallelism of a program increases, the slope at which the speedup increases with the number of processors also increases.

4. Performance

**Efficiency** (Efficiency) is a metric derived from the speedup, representing the performance per processor. It can indicate the processor's utilization for a specific algorithm. The formula is:

$$
\text{Efficiency} = \frac{1}{\text{Speedup}}
$$

5. Clock Speedup Ratio

$$
\text{S}(p) = \frac{t_s}{t_p}
$$

**Speedup in Wall-Clock Time** is simply calculated by dividing the clock time spent using a sequential algorithm by the clock time spent using a parallel algorithm. However, since clock time includes network latency, I/O, cache contention, and other unrelated factors, it is not correlated with the speedup and the complexity of the algorithm. It is only used to roughly measure the speedup.

## Parallel Computing Framework

### 3.1 OpenMP

OpenMP (Open Multi-Processing) is a set of APIs for parallel programming on multi-processor shared-memory machines. It supports languages such as C, C++, and Fortran, and is supported by mainstream compilers like GCC and Clang.
OpenMP provides a high-level abstraction for parallel programming. The greatest benefit of using OpenMP is that, even when compiling without OpenMP-related options or when the compiler does not support OpenMP, the program can still be compiled and executed in a serial manner; this greatly reduces the difficulty of parallel programming, allowing us to focus more on the parallel algorithms themselves rather than their implementation details. Especially for programs that perform parallel division based on datasets, OpenMP is a good choice.

#### Directive

All OpenMP programming operations are based on the `#pragma omp` macro directive. Each directive is converted into an OpenMP library function call, and OpenMP handles operations related to threads, including thread fork, join, and synchronization. Here is a simple example:

c
#include <omp.h>

int main() {
    #pragma omp parallel
    {
        int thread_id = omp_get_thread_num();
        int num_threads = omp_get_num_threads();
        printf("Thread %d of %d\n", thread_id, num_threads);
    }
}

```c++
#include <omp.h>
#include <iostream>

int main()
{
   #pragma omp parallel
   {
       int tid{ omp_get_thread_num() };
       printf("Hello world from thread %d\n", tid);

       int thread_num{ omp_get_num_threads() };
       if (tid == thread_num - 1)
       {
           printf("tid: %d, thread_num: %d\n", tid, thread_num);
       }
   }
    return 0;
}
```
Noticed that when linking, you need to add `-fopenmp`. This is a high-level flag whose main purpose is to link the `gomp` library (GCC's OpenMP implementation; for `clang`, it links the equivalent implementation from `llvm`, similar to the difference between `libstdc++` and `libc++`). OpenMP is typically implemented using `pthread`, so the `gomp` library also links additional libraries to utilize the operating system's thread functionality:
```bash
[joelzychen@DevCloud ~/parallel-computing]$ g++ -std=c++11 -g -o openmp-case openmp-case.cpp -fopenmp
[joelzychen@DevCloud ~/parallel-computing]$ ./openmp-case
Hello world from thread 5
Hello world from thread 2
Hello world from thread 1
Hello world from thread 3
Hello world from thread 7
tid: 7, thread_num: 8
Hello world from thread 4
Hello world from thread 0
Hello world from thread 6
```

omp_get_thread_num() and omp_get_num_threads() functions are straightforward, each returning the thread ID (which is managed by OpenMP, not the PID) and the total number of threads, respectively. The `#pragma omp parallel` is the basic directive that starts a group of threads to execute in parallel. If no thread number is specified without using the `#pragma omp parallel` directive, it defaults to the number of CPU cores. Since the execution is parallel, we cannot guarantee the order of program execution.

#### Example

Here is another example:
cpp
#include <omp.h>
#include <iostream>

int main() {
    #pragma omp parallel
    {
        int thread_num = omp_get_thread_num();
        int num_threads = omp_get_num_threads();

        std::cout << "Thread ID: " << thread_num << ", Number of Threads: " << num_threads << std::endl;
    }
}


Thread ID: 0, Number of Threads: 4



Thread ID: 1, Number of Threads: 4

...

Thread ID: 3, Number of Threads: 4



Thread ID: 0, Number of Threads: 4

Thread ID: 1, Number of Threads: 4

Thread ID: 2, Number of Threads: 4

Thread ID: 3, Number of Threads: 4



Thread ID: 0, Number of Threads: 4

Thread ID: 1, Number of Threads: 4

Thread ID: 2, Number of Threads: 4

Thread ID: 3, Number of Threads: 4



Thread ID: 0, Number of Threads: 4

Thread ID: 1, Number of Threads: 4

Thread ID: 2, Number of Threads: 4

Thread ID: 3, Number of Threads: 4



Thread ID: 0, Number of Threads: 4

Thread ID: 1, Number of Threads: 4

Thread ID: 2, Number of Threads: 4

Thread ID: 3, Number of Threads: 4



Thread ID: 0, Number of Threads: 4

Thread ID: 1, Number of Threads: 4

Thread ID: 2, Number of Threads: 4

Thread ID: 3, Number of Threads: 4



Thread ID: 0, Number of Threads: 4

Thread ID: 1, Number of Threads: 4

Thread ID: 2, Number of Threads: 4

Thread ID: 3, Number of Threads: 4



Thread ID: 0, Number of Threads: 4

Thread ID: 1, Number of Threads: 4

Thread ID: 2, Number of Threads: 4

Thread ID: 3, Number of Threads: 4



Thread ID: 0, Number of Threads: 4

Thread ID: 1, Number of Threads: 4

Thread ID: 2, Number of Threads: 4

Thread ID: 3, Number of Threads: 4



Thread ID: 0, Number of Threads: 4

Thread ID: 1, Number of Threads: 4

Thread ID: 2, Number of Threads: 4

Thread ID: 3, Number of Threads: 4



Thread ID: 0, Number of Threads: 4

Thread ID: 1, Number of Threads: 4

Thread ID
```c++
#include <omp.h>
#include <iostream>
#include <thread>
#include <chrono>

constexpr int thread_num = 3;
using namespace std;

int main()
{
    std::chrono::steady_clock::time_point time_begin = std::chrono::steady_clock::now();

#pragma omp parallel for schedule(static) num_threads(thread_num)
    for (int i = 0; i < thread_num; ++i)    std::this_thread::sleep_for(std::chrono::seconds(1));

    std::chrono::steady_clock::time_point time_end = std::chrono::steady_clock::now();
    cout << "time: " << std::chrono::duration_cast<std::chrono::milliseconds>(time_end - time_begin).count() << " ms" << endl;

    return 0;
}
```
```bash
[joelzychen@DevCloud ~/parallel-computing]$ g++ -std=c++11 -g -o openmp-case openmp-case.cpp -fopenmp
[joelzychen@DevCloud ~/parallel-computing]$ ./openmp-case
time: 1000 ms
[joelzychen@DevCloud ~/parallel-computing]$ g++ -std=c++11 -g -o openmp-case openmp-case.cpp
[joelzychen@DevCloud ~/parallel-computing]$ ./openmp-case
time: 3000 ms
```
### 3.2 OpenMPI

**OpenMPI** (Open Message Passing Interface) is a parallel programming library for inter-process communication based on message queues. MPI is a cross-language communication protocol, and OpenMPI is merely one of its implementations; in the model of parallel programming based on message queues, each process has its own address space, and one process cannot directly access the data in another process. Communication between processes can only be achieved through explicit message sending and receiving. The overhead of communication using message queues is greater than that of shared memory, thus it is mainly used for the development of parallel programming at a large grain.

#### API

MPI has several fundamental functions that are almost always used in every MPI parallel program:

1. `int MPI_Init (int* argc ,char** argv[] )`

Initialize MPI environment, typically the first MPI function called in `argv[]`.

2. `int MPI_Finalize (void)`

Terminate MPI environment, usually the last MPI function called with `argv[]`.

3. `int MPI_Comm_size (MPI_Comm comm ,int* size )`

Get the number of processes in the communication group. `MPI_Comm comm` is the specified communicator, which manages a group of processes that share a communication space. A communication group consists of a set of processes.

4. `int MPI_Comm_rank (MPI_Comm comm ,int* rank)`
### Example

Let's consider an example where we attempt to solve the 0-1 Knapsack problem using OpenMPI. Suppose the number of items is \( N \), the capacity of the knapsack is \( C \), the weight of the \( i \)-th item is \( weight[i] \), and its value is \( value[i] \). We first solve this problem using conventional linear Dynamic Programming (DP).


| i | weight[i] | value[i] |
|---|-----------|----------|
| 1 | 2         | 3        |
| 2 | 3         | 4        |
| 3 | 4         | 5        |
| 4 | 5         | 6        |
| 5 | 6         | 7        |
| 6 | 7         | 8        |
| 7 | 8         | 9        |
| 8 | 9         | 10       |
| 9 | 10        | 11       |
| 10| 11        | 12       |
| 11| 12        | 13       |
| 12| 13        | 14       |
| 13| 14        | 15       |
| 14| 15        | 16       |
| 15| 16        | 17       |
| 16| 17        | 18       |
| 17| 18        | 19       |
| 18| 19        | 20       |
| 19| 20        | 21       |
| 20| 21        | 22       |
| 21| 22        | 23       |
| 22| 23        | 24       |
| 23| 24        | 25       |
| 24| 25        | 26       |
| 25| 26        | 27       |
| 26| 27        | 28       |
| 27| 28        | 29       |
| 28| 29        | 30       |
| 29| 30
```c++
#include <cstring>
#include <iostream>
#include <fstream>
#include <vector>
#include <thread>
#include <chrono>
#include <omp.h>

using namespace std;

int main()
{
    std::chrono::steady_clock::time_point time_begin = std::chrono::steady_clock::now();

    fstream input_file("input-knapsack.txt");
    int N;
    int64_t Capacity;
    input_file >> N >> Capacity;
    int64_t weight[N], value[N];
    for (int i = 0; i < N; ++i)
        input_file >> weight[i] >> value[i];


    vector<vector<int64_t>> dp(N + 1, vector<int64_t>(Capacity + 1));
    for (int i = 0; i <= N; ++i)
    {
        #pragma omp parallel for
        for (int64_t j = 0; j <= Capacity; ++j)
        {
            if (i == 0 || j == 0)
                dp[i][j] = 0;
            else if (j < weight[i - 1])
                dp[i][j] = dp[i - 1][j];
            else
                dp[i][j] = max(dp[i - 1][j], dp[i - 1][j - weight[i - 1]] + value[i - 1]);
        }
    }
    cout << "max value: " << dp[N][Capacity] << endl;

    std::chrono::steady_clock::time_point time_end = std::chrono::steady_clock::now();
    cout << "time: " << std::chrono::duration_cast<std::chrono::milliseconds>(time_end - time_begin).count() << endl;
    return EXIT_SUCCESS;
}
```
First, use a backpack data generator (see Appendix) to generate the data, then compile and run it to compare the results before and after using OpenMPI:
```bash
[joelzychen@DevCloud ~/parallel-computing]$ g++ -std=c++11 -g -o knapsack-generator knapsack-generator.cpp -lcrypto
[joelzychen@DevCloud ~/parallel-computing]$ ./knapsack-generator 1000 8000
[joelzychen@DevCloud ~/parallel-computing]$ g++ -std=c++11 -g -o knapsack knapsack.cpp
[joelzychen@DevCloud ~/parallel-computing]$ ./knapsack
max value: 13093
time: 175 ms
[joelzychen@DevCloud ~/parallel-computing]$ g++ -std=c++11 -g -o knapsack-openmp knapsack.cpp -fopenmp
[joelzychen@DevCloud ~/parallel-computing]$ ./knapsack-openmp
max value: 13093
time: 75
```
Now, using OpenMPI to transform the DP solution of the 0-1 Knapsack problem:
```c++
#include <cstring>
#include <iostream>
#include <fstream>
#include <vector>
#include <thread>
#include <chrono>
#include <mpi.h>

using namespace std;

int main(int argc, char *argv[])
{
    std::chrono::steady_clock::time_point time_begin = std::chrono::steady_clock::now();

    MPI_Init(&argc, &argv);
    MPI_Comm comm = MPI_COMM_WORLD;
    int rank, size;
    MPI_Comm_rank(comm, &rank);
    MPI_Comm_size(comm, &size);
    MPI_Status status;      // MPI receive
    MPI_Request request;    // MPI send

    fstream input_file("input-knapsack.txt");
    int N;
    int64_t Capacity;
    if (rank == 0)
        input_file >> N >> Capacity;
    MPI_Bcast(&N, 1, MPI_INT, 0, comm);
    MPI_Bcast(&Capacity, 1, MPI_LONG, 0, comm);
    MPI_Barrier(comm);

    int64_t weight[N], value[N];
    if (rank == 0)
        for (int i = 0; i < N; ++i)
            input_file >> weight[i] >> value[i];
    MPI_Bcast(weight, N, MPI_LONG, 0, comm);
    MPI_Bcast(value, N, MPI_LONG, 0, comm);
    MPI_Barrier(comm);


    vector<vector<int64_t>> dp(N + 1, vector<int64_t>(Capacity + 1));
    int64_t prev_max_value;    // mpi send and receive variable

    for (int i = 0; i <= N; ++i)    // for each item from 0 to n
    {
        for (int64_t j = rank; j <= Capacity; j += size)   // for each capacity from 0 to Capacity, each thread computes its own rows
        {
            if (i == 0 || j == 0)
                dp[i][j] = 0;
            else if (j < weight[i - 1])
                dp[i][j] = dp[i - 1][j];
            else
            {
                // int MPI_Recv(void *buf, int count, MPI_Datatype datatype, int source, int tag, MPI_Comm comm, MPI_Status *status)
                MPI_Recv(&prev_max_value, 1, MPI_LONG, (j - weight[i - 1]) % size, i - 1, comm, &status);
                dp[i][j] = max(dp[i - 1][j], prev_max_value + value[i - 1]);
            }

            // send dp[i][j] to the next nodes that may need this curr_max_value
            if (i < N && weight[i] + j <= Capacity)
            {
                // int MPI_Isend(const void *buf, int count, MPI_Datatype datatype, int dest, int tag, MPI_Comm comm, MPI_Request *request)
                MPI_Isend(&dp[i][j], 1, MPI_LONG, (j + weight[i]) % size, i, comm, &request);    // asynchronous operation
            }
        }
        MPI_Barrier(MPI_COMM_WORLD);
    }
    MPI_Barrier(MPI_COMM_WORLD);

    if (rank == Capacity % size)
        printf("max value: %ld\n", dp[N][Capacity]);

    if (rank == 0)
    {
        std::chrono::steady_clock::time_point time_end = std::chrono::steady_clock::now();
        printf("time: %ld ms\n", std::chrono::duration_cast<std::chrono::milliseconds>(time_end - time_begin).count());
    }
    MPI_Finalize();
    return EXIT_SUCCESS;
}
```
Compared to OpenMP, solving the Knapsack problem with OpenMPI is much more complex. First, the threads with rank == 0 handle the input and broadcast the input arrays N, C and weight, value to other threads. Subsequently, the steps of the dynamic programming array dp are processed as follows:

For each item i from 0 to n, execute serially.

2. For the jth thread (where j = rank and 0 <= j < size), have it process the corresponding capacity (capacity == j). Then, have it process the next item by setting j += size.

When `i == 0 || j == 0`, initialize the boundaries to 0.

4. If \( j < \text{weight}[i - 1] \), at this point the backpack capacity is less than \(\text{weight}[i - 1]\), so \( \text{dp}[i][j] = \text{dp}[i - 1][j] \).

5. If $j \leq$ $weight[i - 1]$, at this point the backpack capacity is greater than or equal to $weight[i - 1]$, then $dp[i][j] = \max(dp[i - 1][j], dp[i - 1][j - weight[i - 1]] + value[i - 1])$. However, since the result for a capacity of $j - weight[i - 1]$ (i.e., $dp[i - 1][j - weight[i - 1]}$) may not be from processing thread $j$, the local $dp[i - 1][j - weight[i - 1]}$ may not contain the correct value. Therefore, the corresponding value (i.e., $prev\_max\_value$) needs to be obtained from thread $\left(\frac{j - weight[i - 1]}{size}\right) \bmod size$ through MPI before further processing.

6. For the next item, since the backpack capacity of `j + weight[i]` might utilize the current `dp` result, `dp[i][j]` needs to be sent to the thread processing the capacity of `j + weight[i]` modulo `size`.

Finally, the result processing only requires having the thread that processed `dp[N][Capacity]` output, specifically thread `rank = Capacity % size`. Note that this approach has a bug. If the `weight[i]` of the `i`-th item in the input is `0`, the thread will receive a value from itself while waiting for `recv`, leading to incorrect results.

Before using OpenMPI, download the source code from the [official website](https://www.open-mpi.org/software/ompi/v4.0/) and install it (or use yum to install it). Then compile with `mpic++` and run with `mpirun`.
```bash
[joelzychen@DevCloud ~/parallel-computing/openmpi]$ sudo find / -name "mpic++"
/usr/lib64/openmpi/bin/mpic++
/usr/lib64/mpich/bin/mpic++
[joelzychen@DevCloud ~/parallel-computing]$ /usr/lib64/mpich/bin/mpic++ -g -std=c++11 -o knapsack-openmpi knapsack-openmpi.cpp
[joelzychen@DevCloud ~/parallel-computing]$ /usr/lib64/mpich/bin/mpirun -n 1 ./knapsack-openmpi
max value: 13093
time: 6694 ms
[joelzychen@DevCloud ~/parallel-computing]$ /usr/lib64/mpich/bin/mpirun -n 2 ./knapsack-openmpi
max value: 13093
time: 4863 ms
[joelzychen@DevCloud ~/parallel-computing]$ /usr/lib64/mpich/bin/mpirun -n 4 ./knapsack-openmpi
max value: 13093
time: 3674 ms
[joelzychen@DevCloud ~/parallel-computing]$ /usr/lib64/mpich/bin/mpirun -n 8 ./knapsack-openmpi
max value: 13093
time: 2487 ms
```
### 3.3 CUDA

CUDA's full name is Compute Unified Device Architecture, a platform and API for parallel computing. It allows developers to use **CUDA-supporting GPUs** for parallel programming; GPUs cannot operate independently and must connect to the CPU via PCIe bus to work together. GPU-based parallel computing can be considered an Heterogeneous Computing Architecture, where the CPU handles the logical, serial parts, and the GPU handles the data-intensive, parallel parts. Typically, the CPU is referred to as the host, and the GPU as the device.

#### Kernel

CUDA's **kernel function** executes in parallel on the GPU. This function contains only the parallel part and is executed in parallel by numerous threads on the GPU. Unlike threads on the CPU, threads on the GPU are lighter-weight, with lower creation costs and more flexible thread switching. When entering a CUDA kernel function, one can define a large number of virtual threads, but the number of hardware threads that can execute in parallel is limited. Generally, the execution flow of a CUDA program looks like this:

1.host side performs memory allocation and data initialization, executing the sequential program.
2.device side performs memory allocation and copies data from the host side to the device side.
3.device side executes the kernel function and improves efficiency using a cache.
Copy the computed results from the device to the host.

About all functions of OpenMPI, you can refer to the documentation at [Open MPI v4.0.4 documentation](https://www.open-mpi.org/doc/current/).
5. Release memory on the device and wait for the next kernel call

#### Thread Hierarchy

When executing a kernel in CUDA, threads are organized into a three-level hierarchy:

![grid-block-thread](https://raw.githubusercontent.com/chr1sc2y/warehouse-deprecated/refs/heads/main/resources/parallel-computing/grid-block-thread.png)

1. grid

    A grid is a logical entity, which can be understood as a workspace that operates across the entire GPU. All threads within the same grid share the global memory space;

2. thread block

    A thread block is a set of parallel threads. A block runs within a single streaming multiprocessor. That is, all threads within a block run in the same streaming multiprocessor. They can communicate via shared memory or synchronization primitives. Threads in different blocks generally cannot communicate or collaborate; Each block should be able to run independently;

3. thread

    Threads execute on a CUDA core. As mentioned earlier, threads on the GPU are very lightweight and can provide rapid context switching with the help of large registers (thread handles on the CPU exist in lower memory hierarchies, such as the cache);

When calling a kernel, we specify the number and dimensions of blocks and threads through `<<<block, thread>>>`.

#### Example

Let's look at a simple example, performing approximately 10 billion additions in serially:

cpp
#include <cuda_runtime.h>

__global__ void addKernel(int* a, int* b, int* c) {
    int idx = threadIdx.x + blockIdx.x * blockDim.x;
    c[idx] = a[idx] + b[idx];
}

int main() {
    int size = 1000000000;
    int* d_a, *d_b, *d_c;
    cudaMalloc(&d_a, size * sizeof(int));
    cudaMalloc(&d_b, size * sizeof(int));
    cudaMalloc(&d_c, size * sizeof(int));

    int* h_a = new int[size];
    int* h_b = new int[size];
    int* h_c = new int[size];

    // Initialize h_a and h_b

    cudaMemcpy(d_a, h_a, size * sizeof(int), cudaMemcpyHostToDevice);
    cudaMemcpy(d_b, h_b, size * sizeof(int), cudaMemcpyHostToDevice);

    int threadsPerBlock = 256;
    int blocksPerGrid = (size + threadsPerBlock - 1) / threadsPerBlock;

    addKernel<<<blocksPerGrid, threadsPerBlock>>>(d_a, d_b, d_c);

    cudaMemcpy(h_c, d_c, size * sizeof(int), cudaMemcpyDeviceToHost);

    delete[] h_a;
    delete[] h_b;
    delete[] h_c;

    cudaFree(d_a);
    cudaFree(d_b);
    cudaFree(d_c);

    return 0;
}

```c++
#include <iostream>
#include <cstdlib>
#include <cstring>
#include <chrono>
#include <thread>

using namespace std;
constexpr uint64_t magic_number = 12345;

void Add(int n, uint64_t *x)
{
    for (int i = 0; i < n; ++i)
        x[i] += x[i];
}

int main(void)
{
    int n = 1<<30;
    uint64_t *x = (uint64_t *)malloc(n * sizeof(uint64_t));
    memset(x, magic_number, sizeof(x));

    std::chrono::steady_clock::time_point time_begin = std::chrono::steady_clock::now();
    Add(n, x);
    std::chrono::steady_clock::time_point time_end = std::chrono::steady_clock::now();
    cout << "time: " << std::chrono::duration_cast<std::chrono::milliseconds>(time_end - time_begin).count() << " ms" << endl;

    free(x);
    return 0;
}
```
For ease of comparison, compile and run `nvcc` in Windows PowerShell as follows:
```powershell
PS G:\> nvcc -o add .\add.cpp  -ccbin "C:\Program Files (x86)\Microsoft Visual Studio\2017\Professional\VC\Tools\MSVC\14.16.27023\bin\Hostx64\x64"
add.cpp
PS G:\> .\add.exe
time: 4472 ms
```
One can see that the execution time of the `Add` function is approximately 4472 ms. Now, we modify it to use a parallel program with CUDA:
```c++
#include <iostream>
#include <cstdlib>
#include <cstring>
#include <chrono>
#include <thread>
#include <string>

using namespace std;
constexpr int magic_number = 12345;

__global__ void Add(int n, int *x)
{
    for (int i = 0; i < n; ++i)
        x[i] += x[i];
}

int main(void)
{
    int n = 1<<30;
    int64_t byte_size = n * sizeof(int);
    int *x;
    x = (int*)malloc(byte_size);
    for (int i = 0; i < n; ++i)
        x[i] = magic_number;

    int *cuda_x;
    cudaMalloc((void**)&cuda_x, byte_size);

    // copy from host to device
    cudaMemcpy(cuda_x, x, byte_size, cudaMemcpyHostToDevice);

    std::chrono::steady_clock::time_point time_begin = std::chrono::steady_clock::now();
    Add<<<1, 1>>>(n, cuda_x);
    cudaDeviceSynchronize();
    std::chrono::steady_clock::time_point time_end = std::chrono::steady_clock::now();

    // copy from device to host
    cudaMemcpy(x, cuda_x, byte_size, cudaMemcpyDeviceToHost);

    // check result
    bool result{ true };
    for (uint32_t i = 0; i < n; ++i)
        result = (result && (x[i] == magic_number + magic_number));
    string result_str = (result ? "true" : "false");

    cout << "result: " << result_str << endl;
    cout << "time: " << std::chrono::duration_cast<std::chrono::milliseconds>(time_end - time_begin).count() << " ms" << endl;
    free(x);
    cudaFree(cuda_x);
    return 0;
}
```
```powershell
PS G:\> nvcc -o cuda-add .\cuda-add.cu -ccbin "C:\Program Files (x86)\Microsoft Visual Studio\2017\Professional\VC\Tools\MSVC\14.16.27023\bin\Hostx64\x64"
cuda-add.cu
   Creating library cuda-add.lib and object cuda-add.exp
PS G:\> .\cuda-add.exe
result: true
time: 21904 ms
```
One can see that CUDA programs contain some special keywords and APIs:

`__global__` marks a kernel function, and it can be identified as a kernel function by the CUDA compiler if it is prefixed with `__global__` in the function signature.

2. Since the `Add` function runs on the device, we need to allocate memory via `malloc` and `cudaMalloc` separately for the host and device sides first, and then use `cudaMemcpy` to copy the initialized data from the host side to the device side;

3.3. We need to call the `Add` kernel function on the device side. This operation is asynchronous on the host side; it does not wait for the execution results on the device side. We need to use the `cudaDeviceSynchronize` function to synchronize the execution on the device side and return. If we call multiple kernels in sequence without specifying a control flow on the device side, these kernels will execute in sequence on the device side.

4. After execution on the device, use `cudaMemcpy` to copy the data back to the host for verification.

5. Finally, use `free` and `cudaFree` to release the memory.

Our program runs on the GPU, but it runs slower than on the CPU, as we allocated only 1 block and 1 thread for the kernel (`Add<<<1,1>>>(n, x);`), failing to leverage the GPU's parallel computing capabilities and wasting time on interactions between CPU and GPU. The optimization method is similar to that in OpenMPI; each thread should process its own data, incrementing the step size within the loop:
```cpp
__global__ void Add(int n, int *cuda_x)
{
    int index = blockIdx.x * blockDim.x + threadIdx.x;
    int stride = blockDim.x;
    for (int i = index; i < n; i += stride)
        cuda_x[i] += cuda_x[i];
}

int main(void)
{
    // ...
    Add<<<4096, 256>>>(n, cuda_x);
    // ...
}
```
Where:

- `blockIdx.x` represents the ID of the block, which is the current index of the block;

- `blockDim.x` represents the dimension of the block, i.e., the number of threads contained within a block; it also serves as the stride.

- Similarly, `threadIdx.x` represents the thread ID, which is the current block's index.

- `index` is the position of the data in memory that needs to be operated on.

In this example, we enable 4096 blocks and 256 threads, i.e., `blockIdx.x < 4096`, `blockDim.x == 256`, `threadIdx.x < 256`;

![grid-dim](https://raw.githubusercontent.com/chr1sc2y/warehouse-deprecated/refs/heads/main/resources/parallel-computing/grid-dim.png)

Of course, applying too many blocks does not enhance the efficiency of the computation, as the CUDA cores will waste much time scheduling these blocks. We can modify `<<<block, thread>>>` multiple times to compare the performance under different numbers of blocks and threads:
```powershell
# Add<<<4096, 256>>>(n, x);
PS G:\> nvcc -o cuda-add .\cuda-add.cu -ccbin "C:\Program Files (x86)\Microsoft Visual Studio\2017\Professional\VC\Tools\MSVC\14.16.27023\bin\Hostx64\x64"                                                                                      cuda-add.cu
   Creating library cuda-add.lib and object cuda-add.exp
PS G:\> .\cuda-add.exe
result: true
time: 159454 ms

# Add<<<1, 256>>>(n, x);
PS G:\> .\cuda-add.exe
result: true
time: 1038 ms

# Add<<<1, 1024>>>(n, x);
PS G:\> .\cuda-add.exe
result: true
time: 299 ms
```
![cuda-add](https://raw.githubusercontent.com/chr1sc2y/warehouse-deprecated/refs/heads/main/resources/parallel-computing/cuda-add.gif)

More information on using CUDA can be found in the [CUDA Toolkit Documentation](https://docs.nvidia.com/cuda/).

## 4 Summary

This article mainly introduces the hardware architecture attached to parallel computing and some related concepts. The OpenMP, OpenMPI, and CUDA methods, which are based on shared memory, message passing, and GPU (which is also a shared memory parallel programming), are introduced simply. More development experiences in parallel computing need to be accumulated through practice.

All the codes in this article are included in [https://github.com/chr1sc2y/parallel-computing-demo](https://github.com/chr1sc2y/parallel-computing-demo).

## 5 Appendix

### 0-1 Knapsack Problem Random Generator
```c++
// knapsack-generator.cpp
#include <iostream>
#include <fstream>
#include <openssl/rand.h>

using namespace std;

int main (int argc, char *argv[])
{
    if (argc < 3)
    {
        fprintf(stderr, "usage: %s N C\n", argv[0]);
        exit(1);
    }

    int N = stoi (argv[1]);
    uint64_t C = stoi (argv[2]);

    int m = 4 * C / N;
    unsigned char buff[2 * N];
    RAND_seed(&m, sizeof(m));
    RAND_bytes(buff, sizeof(buff));

    ofstream file_stream;
    file_stream.open("input-knapsack.txt");
    file_stream << N << ' ' << C << endl;
    for (int i = 0; i < N; i++)
    {
        file_stream << buff[2 * i] % m << ' ' << buff[2 * i + 1] % m << endl;;
    }
    file_stream.close();

    return 0;
}
```
### References

1. OpenMP 4.5 API C/C++ Syntax Reference Guide. (2020). Retrieved 10 August 2020, from https://www.openmp.org/wp-content/uploads/OpenMP-4.5-1115-CPP-web.pdf

2. Open MPI v4.0.4 documentation. (2020). Retrieved 10 August 2020, from https://www.open-mpi.org/doc/current/

3. Jiaoyun, Yang & Yun, Xu & Yi, Shang. (2010). An Efficient Parallel Algorithm for Longest Common Subsequence Problem on GPUs. Lecture Notes in Engineering and Computer Science. 1.

4. CUDA Toolkit Documentation. (2020). Retrieved 10 August 2020, from https://docs.nvidia.com/cuda/

5. Harwood, A., & Lanch, A. (2020). COMP90025 Parallel and Multicore. Retrieved 10 August 2020, from School of Computing and Information Systems The University of Melbourne

6. Zeller, C. (2011). CUDA C/C++ Basics Supercomputing. Retrieved 7 August 2020, from https://www.nvidia.com/docs/IO/116711/sc11-cuda-c-basics.pdf

7. Han, J., & Sharma, B. Learn CUDA programming.
8. Ruetsch, G., & Oster, B. (2020). Getting Started with CUDA. Retrieved 10 August 2020, from https://www.nvidia.com/content/cudazone/download/Getting_Started_w_CUDA_Training_NVISION08.pdf

9. Harris, M. (2017). An Even Easier Introduction to CUDA. Retrieved 10 August 2020, from https://developer.nvidia.com/blog/even-easier-introduction-cuda/

10. Modern Parallel Computing (Part 3) - Some Typical GPU Architectures · Infectious Waste. (2020). Retrieved 10 August 2020, from https://infectiouswaste.github.io/2019/02/20/typical-gpu-arch/

11. Cheng, J. (2014). Professional Cuda C programming. Indianapolis, IN: John Wiley and Sons, Inc.

12. Harris, M., Ebersole, M., & Sakharnykh, N. (2020). Unified Memory in CUDA 6 | NVIDIA Developer Blog. Retrieved 10 August 2020, from https://developer.nvidia.com/blog/unified-memory-in-cuda-6/

## Original references

- [Reference 1](https://zh.wikipedia.org/wiki/%E6%91%A9%E5%B0%94%E5%AE%9A%E5%BE%8B)
- [Reference 2](https://zh.wikipedia.org/wiki/%E6%91%A9%E5%B0%94%E5%AE%9A%E5%BE%8B)

## Figures from the original edition

![Figure 1 from the original edition](https://raw.githubusercontent.com/chr1sc2y/warehouse-deprecated/refs/heads/main/resources/parallel-computing/amdahl's-law.png)
![Figure 2 from the original edition](https://raw.githubusercontent.com/chr1sc2y/warehouse-deprecated/refs/heads/main/resources/parallel-computing/efficiency.svg)
![Figure 3 from the original edition](https://raw.githubusercontent.com/chr1sc2y/warehouse-deprecated/refs/heads/main/resources/parallel-computing/heterogeneous-computing.png)
![Figure 4 from the original edition](https://raw.githubusercontent.com/chr1sc2y/warehouse-deprecated/refs/heads/main/resources/parallel-computing/simple-process-flow.png)
