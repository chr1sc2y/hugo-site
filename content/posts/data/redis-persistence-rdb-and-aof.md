---
title: "Redis Persistence with RDB and AOF"
date: 2023-02-01T18:02:04+11:00
draft: false
categories: ["Redis"]
description: "A translated technical note on Redis Persistence with RDB and AOF, preserving the examples and context of the original article."
aliases:
  - /posts/data/redis-%E6%8C%81%E4%B9%85%E5%8C%96%E6%9C%BA%E5%88%B6-rdb-%E5%92%8C-aof-72a88c70207a4518824d1fe4be7ebb9e/
---
# Redis Persistence with RDB and AOF

> Originally published in Chinese on 2023-02-01; this English edition preserves the original scope and technical context.

# Redis Persistence

Why Do We Need Persistence?

Redis is a database based on memory. Once the service goes down, all data in memory will be lost. Typically, these data can be recovered from the database, which puts a huge read pressure on the database and is very slow, causing the program to respond slowly. Therefore, Redis provides a mechanism to persistently store data to disk and recover data through backup files, known as the persistence mechanism.

Persistence Methods

Currently, the Redis documentation on persistence supports the following schemes:

1. RDB (Redis Database): Generate a snapshot of the data at a certain point in time and save it to disk.
2. AOF (Append Only File): Record every received write operation to disk. These operations can be replayed when Redis restarts and used to rebuild the Redis database.
3. RDB + AOF: Hybrid mode of AOF and RDB.

# RDB

RDB saves a point-in-time snapshot of the entire dataset; it is used for Redis data backup, transfer, and recovery. It is the default persistence method for Redis.

## Functionality

RDB uses the copy-on-write (Copy-on-Write) mechanism provided by the operating system for persistence, i.e., when the main process P forks a child process Q, Q and P share the same memory space. When P prepares to write to a particular memory page, P will duplicate that page and modify the data on the new copy. Q, on the other hand, continues to read the original memory page. This allows Redis instances to continue serving external traffic while minimizing costs for data persistence. However, this means that write operations during the persistence process are not recorded.

![1](https://raw.githubusercontent.com/chr1sc2y/prov1dence.github.io/refs/heads/master/posts/data/1.png/)

![2](https://raw.githubusercontent.com/chr1sc2y/prov1dence.github.io/refs/heads/master/posts/data/2.png/)


## Trigger Methods

There are two ways to trigger RDB persistence:
1. Manual Trigger. Includes two commands:
    1. `save`: Blocks the Redis process and performs RDB persistence until it completes. This can cause a long-time block on instances with large memory usage.
    2. `bgsave`: Background save, where the Redis process creates a child process through the `fork` operation and performs RDB persistence in the child process. It only blocks the Redis process during the `fork` stage.
2. Automatic Trigger, achieved through the `save` commands configured. The Redis service has a periodic maintenance function `serverCron`, which runs every 100 ms by default. One of its functions is to check if any of the conditions for the `save` commands are met. If automatic triggering is not desired, the `save` commands can be commented out.
```
save x y # Save x seconds if at least y keys change within that time
save 60 900 # Save every 60 seconds if at least 900 keys change within that time

```
## Summary

### Should RDB Trigger Be At The Highest Frequency?

To reduce the amount of data lost during a crash, we might trigger an RDB backup once a minute. Although `bgsave` executes in a sub-process and does not block the main thread, there are still some issues:

1. `bgsave` creates a child process via `fork`, which itself blocks the parent process. The longer the parent process occupies memory, the longer the `fork` operation blocks the parent process.
2. Writing full dataset to disk consumes a lot of bandwidth and puts pressure on the disk, thereby affecting Redis instance performance. If a subsequent write operation is initiated before the previous write operation completes, it may lead to a vicious cycle.

From these two points, it can be considered that the frequency of triggering RDB is not necessarily higher is better. We need to consider the size of the memory occupied by the Redis instance and the speed at which full data is written to the disk.

### Advantages

- RDB files are snapshots at a specific point in time, compressed using the LZF algorithm. The compressed file size is much smaller than the memory size. It is suitable for periodic execution (such as every hour) and uploading RDB files to the disaster recovery data center.
- Compared to AOF, RDB recovery of large datasets is faster.

### Defects

- RDB mode lacks real-time performance and cannot achieve second-level persistence;
- RDB requires forking a subprocess, which has a very high cost for subprocess execution;
- RDB files are encoded in binary format, lacking readability.

# AOF

AOF (Append Only File) persists data by recording logs of each write operation. After each write operation, the command executed is recorded in the AOF log. Upon server restart, the data is restored by sequentially replaying these logs.

Configuration

AOF feature is disabled by default. It needs to be enabled by modifying the `redis.conf` file and then restarting Redis.
```
# no by default
appendonly yes
appendfilename appendonly.aof

```
**Write-Ahead Logging (WAL)**

Unlike MySQL's Write-Ahead Logging (WAL), Redis's Append-Only File (AOF) logs commands after the write operation completes. This ensures Redis remains non-blocking and responsive to write operations while also preventing rollback of invalid commands during runtime. However, if the server crashes after the command execution but before the log is written, data might still be lost.

**Workflow**

The AOF workflow can be summarized into several steps: appending commands, synchronizing file writes, rewriting the file, and reloading the server.

## 1 Appending Commands

When AOF persistence is enabled, Redis appends each write command to its **AOF buffer** after executing the command. This is done in accordance with the **[RESP (Redis Serialization Protocol)](https://redis.io/docs/reference/protocol-spec/)**, which specifies the format for the command.

The **AOF buffer** (aof_buf) is implemented using Redis's SDS (Simple Dynamic String) data structure, depending on the command type, and using specific methods (`catAppendOnlyGenericCommand`, `catAppendOnlyExpireAtCommand`) to process the command, which are then appended to the buffer.

If the AOF is **rewriting**, these commands are appended to the **rewrite buffer** (`aof_rewrite_buffer`).

## 2 Writing and Synchronizing with fsync

Due to the slower I/O performance of disks compared to CPUs, waiting for data to be written to disk after each write operation can significantly degrade system performance. To address this, operating systems provide the **delayed write** mechanism to improve disk I/O performance.

Before each event loop iteration (`beforeSleep`), Redis calls the function `flushAppendOnlyFile`, which writes the data from the **AOF buffer** to the **kernel buffer** and synchronizes the data to the **disk** based on the **appendfsync** configuration. Three options are available:

- `always`: Always calls `fsync()`, providing the highest safety but the lowest performance.
- **no**: Does not call `fsync()`. Best performance, worst security.
- **everysec**: Calls `fsync()` only when the synchronization conditions are met. This is the official recommended strategy, the default configuration, providing a balance between performance and data safety, and only losing data for up to one second in case of sudden system failure.

## 3 **Rewrite**

As time progresses, the size of the AOF file increases, leading to more disk space usage and longer data recovery times. To address this issue, Redis introduces the **AOF rewrite (AOF Rewrite)** mechanism, which creates a new AOF file to consolidate multiple commands from the old file into a single command, replacing the old file to reduce the AOF file size.

### **When does rewrite occur?**

Similar to RDB, AOF rewrite can be triggered manually or automatically.

1. **Manual Trigger**: Calls the `bgrewriteaof` command. If no background save or background rewrite operation is currently running, the rewrite will be executed immediately. Otherwise, it will wait until the background operation is completed.
2. **Automatic Trigger**: Controlled by two configuration options. The rewrite will occur only if both conditions are met:
    - `auto-aof-rewrite-percentage`: The ratio of the increase in the current AOF file size (aof_current_size) to the size of the AOF file at the last rewrite (aof_base_size). Defaults to 100, meaning the rewrite occurs when `aof_current_size == 2 * aof_base_size`.
    - `auto-aof-rewrite-min-size`: The minimum size of the AOF file for the rewrite to occur during a `BGREWRITEAOF` operation. Defaults to 64MB.

### **How does the rewrite process work?**

1. **Triggering the Rewrite**: If the `bgrewriteaof` command is called, the process checks for any existing background save or background rewrite operations. If any are running, the rewrite is postponed until they are completed.
2. **Forking the Process**: The main process forks a child process to prevent blocking and ensure service availability.
3. **Writing to Temporary AOF File**: The child process traverses the Redis memory snapshot and writes the data to a temporary AOF file. It also writes new write instructions to the `aof_buf` and `aof_rewrite_buf` rewrite buffers. The `aof_buf` is used to write back to the old AOF file, while `aof_rewrite_buf` is used to prepare for the final write to the temporary AOF file, preventing lost new write operations during the snapshot traversal.
4. **Notifying the Main Process**: After the temporary AOF file writing is complete, the child process notifies the main process.
5. **Writing to Temporary AOF File**: The main process writes the data from `aof_rewrite_buf` to the temporary AOF log generated by the child process.
6. **Replacing the Old AOF File**: The main process replaces the old AOF file with the temporary AOF file, completing the entire rewrite process.
The entire process can be referenced as follows:

When Redis starts, initialize `aof_base_size` to the size of the AOF file at the time. When the AOF file rewrite operation completes during Redis operation, it updates the file. `aof_current_size` represents the real-time size of the AOF file when `serverCron` executes. The AOF rewrite will trigger if the following two conditions are met:

### Is AOF Rewriting Blocking?

The `bgrewriteaof` background process completes the rewrite process. The main thread forks a background child process `bgrewriteaof`. The `fork` operation copies the main thread's memory to the `bgrewriteaof` child process, containing the latest database data. Then, the `bgrewriteaof` child process sequentially writes the copied data and records the rewrite log. Thus, the main thread is blocked only when the `fork` operation occurs during the rewrite process.
Redis starts after booting by executing the `loadDataFromDisk` function, with the process roughly as follows:

1. When AOF is not enabled, only the RDB file is used for data loading.
2. When AOF is enabled, if the AOF file uses the RDB header, then RDB is used first, followed by AOF. Otherwise, only AOF is used for data loading.

## Summary

### **Can AOF Ensure Data Integrity?**

In case of a crash or full disk during the write operation on the AOF file, due to the delayed write feature, the RESP commands in the AOF log might be incomplete and thus truncated. In such a scenario, Redis will perform different actions based on the configuration value of `aof-load-truncated`:

- **yes**: Load as many data as possible and notify the user via logs.
- **no**: Crash the system in a system error manner and prohibit restarts, requiring manual file repair by the user.

### **Advantages**

- **Better Real-Time Performance**: AOF persistence has a better real-time performance because the default fsync strategy is every second. In extreme cases, it might only lose one second of data.
- **Safe Operations on AOF Log**: Operations on the AOF log only append and do not cause file corruption. Even if the final written data is truncated, it can be easily repaired using the `redis-check-aof` tool.
- **Efficient Rewriting Mechanism**: The rewriting mechanism ensures that the AOF log does not consume too much space and records new write operations into the old log to prevent data loss.
- **Higher Readability and Exportability**: The AOF log is more readable and can be easily exported.

### **Disadvantages**

- **AOF Files Tend to Be Larger**: Generally, AOF files are larger compared to RDB files for the same data set.
- **Higher Latency During Heavy Writes**: When there are many write operations, the latency of AOF is higher.

## Original references

- [Reference 1](https://redis.io/docs/management/persistence/)
- [Reference 2](https://en.wikipedia.org/wiki/Copy-on-write)
- [Reference 3](https://github.com/antirez/sds)
