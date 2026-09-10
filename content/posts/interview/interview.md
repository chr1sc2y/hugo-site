---
title: "Backend Engineering Interview Notes After Two Years of Experience"
date: 2022-01-26T15:37:36+08:00
draft: false
categories: ["interview"]
description: "A translated technical note on Backend Engineering Interview Notes After Two Years of Experience, preserving the examples and context of the original article."
---
# Backend Engineering Interview Notes After Two Years of Experience

> Originally published in Chinese on 2022-01-26; this English edition preserves the original scope and technical context.

## Byte

### One Way

- coding: For an array, pick one element with a single traversal, where each element is chosen with equal probability (from a global perspective, each element has a probability of 1/n)
  - For the i-th element, its probability of being chosen in the i-th round is 1/i
  - From then on, once an element is picked, it gets eliminated; for the i+1-th round, its elimination probability is 1/(i+1), making its survival probability 1 - 1/(i+1)
- Finally, the probability of each element being chosen is as follows. The first `1/i` represents it being chosen on the `i`-th occasion, with the rest representing it being left in subsequent rounds.
  - ![1](https://raw.githubusercontent.com/chr1sc2y/warehouse-deprecated/refs/heads/main/resources/interview/1.png)

- followup: randomly pick k elements with equal probability
  - For the ith element, its probability of being chosen in the ith round is k/i
  - From then on, the only scenario for it to be eliminated is when a new element is chosen, and it is equally likely to be picked in the remaining choices; for the ith+1 round, its elimination probability is k/(i+1) * 1/k = 1/(i+1), so the probability of it being left is 1 - k/(i+1) * 1/k = 1 - 1/(i+1)
- Finally, the probability of each element being chosen is as follows. The first $k/i$ represents it being chosen in the $i$-th round, while the other numbers represent it being left out in subsequent rounds.
  - ![2](https://raw.githubusercontent.com/chr1sc2y/warehouse-deprecated/refs/heads/main/resources/interview/2.png)

- coding: Implement Fisher–Yates Shuffle
    ```cpp
    void shuffle(vector<int> v)
    {
        int n = v.size();
        for (int i = n - 1; i >= 1; --i)
        {
            int j = rand() % (i + 1);
            swap(v[i], v[j]);
        }
    }
    ```
### Two-Face

- System Design: How to push this message to his tens of millions of followers once a streamer goes live
  - Design Data Warehouse (Database can also be used)
  - Message Queue
- Follow-up: What to do after the streamer goes off-air
- Follow-up: How to ensure message push or consumption
- Follow-up: How to avoid message duplication

### Three-Sided

- DB Migration Process
  1. Set snapshot, offline migrate existing data
  2. Dual write, write new data stream after snapshot to the new DB
  3. Route switch, redirect read requests to the new DB

- coding: Write a rough framework for a service.

## Hotstar

### One Way

- Understanding coroutines
- What is a CSRF token? Cross-site request forgery (CSRF); An attacker induces a victim to visit a third-party website, where the attacker sends a cross-site request to the target website and retrieves registration credentials to perform a specific operation on behalf of the user without backend verification
- Understanding heuristic search, research and optimization of the heuristic function
- Coding: Implement Trie
- Coding: Implement Aho-Corasick Automaton

### Two-Face

- Project Experience, details
- Understanding of tf
- Coding: Implement Object Pool
- Coding: Implement Memory Pool

### Three-Sided

- How to convert a long URL into a short URL:
  - Use numbering to assign a short number to each long address.
  - Use 62 characters (0-9, a-z, A-Z) in a base-62 system to save length.
  - Different numbering ranges are used by different numbering systems in distributed systems (System A uses numbers 0-999, System B uses numbers 1000-1999).
- coding: Implement the decorator pattern in design patterns.
- coding: After a segment of code that can be compiled is encoded and saved, the comments in the string are removed through string processing.
- Remove the comments content after the `//` and the content within the `/* */`.
- Line breaks need to be retained.

plaintext
Four Sides

plaintext
Four Sides

plaintext
Four Sides


- Job-hopping Motivation
- Short and Long-term Career Plans for the Future
- Done

## Microsoft

### One Way

- **Project Overview**
- **coding**: Implement a class `TextProcessor` with the following functions:
  - `void set_variable(string k, string v)`: Map `k` to `v`
  - `void get_text(string s)`: Replace content between `{` and `}` in string `s`
  - **Example**:
  ```cpp
  auto text_processor = new TextProcessor();
  text_processor->set_variable("Name", "abc");
  text_processor->set_variable("Date", "2021-11-23");
  auto content = text_process->get_text("Dear {Name}, welcome! {Date}");
  printf("%s\n", content);
  // "Dear abc, welcome! 2021-11-23"
  ```
Markdown

  - Very simple string processing, pay attention to cases with multiple escape characters `\`

### Second Interview

- Talk about the project
- Differences between Prometheus and MySQL, Elasticsearch
- Understanding of Kubernetes
- Coding: 24-Sum Game, refer to LeetCode 679 problem

### Third Interview

- Differences between C++ and Java
- Mechanism and implementation of dynamic dispatch in C++
- Can a constructor be a virtual function? Why
- Coding: A piece of C++ code that compiles and saves, then processes strings to remove all comments (including `//` and `/* */`), same as the coding in the Third Interview with Hotstar

### Fourth Interview

- Chit-chat
- Reasons for switching jobs, future plans
- System Design: Design for a Map Navigation App on the backend
Considerations at every stage, from project initiation to production launch.
  - Consideration details when designing various technical solutions

### Fifth Interview

The interviewer is Indian, and the accent is not too heavy

- Self-introduction, educational background
- Talk about the project with the most sense of achievement, why
- Coding: Count the number of rectangles in a graph (not found the original problem)

## Amazon

### Preliminary Test

1. Two groups of shows, each group is given the start time and duration. Find the earliest time point to finish watching one show from each group; for example, Group A has 3 shows with start times [1, 2, 3] and durations [1, 1, 1]; Group B also has 3 shows with start times [1, 2, 3] and durations [10, 5, 1]. The earliest finish time is 4, where Group A finishes the first show at 1+1=2, and Group B finishes the third show at 3+1=4. Use greedy, find the earliest finish time for the first group, then look for the earliest finish time for the second group from this point. Additionally, you need to check the order of B before A and then check again.
2. There are `n` servers, with each server's startup power being `A[n]` and continuous power being `B[n]`. Only continuous servers can form a cluster, with the total startup power being `max(A[i...j])` and the total continuous power being `sum(A[i...j]) * (j-i+1)`. Given a value `p`, find the maximum number of servers that can form a cluster such that the total startup and continuous power is less than or equal to `p`. For example, `A[n] = {3,6,1,3,4}`, `B[n] = {2,1,3,4,5}`, and `p = 25`. The cluster formed by the fourth and fifth servers has a total power of `4 + (4+5)*2 = 22`, which is less than `25`. Therefore, the number of servers in this cluster is `2`.

    - Servers need to be continuous, so a sliding window is used to determine the range. The startup power is calculated using a monotonic queue.

### One Interview

- Coding: LeetCode 252 Meeting Rooms, traverse once, use difference array twice
- Coding: LeetCode 103 Zigzag Tree, traverse twice using two stacks for level-order traversal, or traverse recursively in pre-order and reverse the even-indexed arrays

### Two Interview

- System Design: E-commerce website

### Three Interview

- System Design: Design a browsing record feature, considering high concurrency

### Four Interview

- Coding: LeetCode 101 Symmetric Binary Tree
- Followup: Symmetric Multi-Tree
