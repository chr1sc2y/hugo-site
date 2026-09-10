---
title: "LeetCode: Heaps"
date: 2019-08-05T19:12:25+10:00
draft: false
categories: ["LeetCode"]
description: "A translated technical note on LeetCode: Heaps, preserving the examples and context of the original article."
---
# LeetCode: Heaps

> Originally published in Chinese on 2019-08-05; this English edition preserves the original scope and technical context.

## Title

#### [Kth largest element in 215 array](https://leetcode-cn.com/problems/kth-largest-element-in-an-array/)

The simplest heap application.
```c++
class Solution {
public:
    int findKthLargest(vector<int> &nums, int k) {
        priority_queue<int, vector<int>, greater<>> heap;
        for (auto &m:nums) {
            if (heap.size() < k || m > heap.top())
                heap.push(m);
            if (heap.size() > k)
                heap.pop();
        }
        return heap.top();
    }
};
```
#### [347 Top K high-frequency elements](https://leetcode-cn.com/problems/top-k-frequent-elements/submissions/)

First traverse once to count the number of occurrences of each element in the array, then use a large root heap to save the first k high-frequency elements, and finally pop these elements out in sequence and store them in the result array. The time complexity is O(n) and the space complexity is O(n).
```c++
class Solution {
public:
    vector<int> topKFrequent(vector<int> &nums, int k) {
        priority_queue<pair<int, int>, vector<pair<int, int>>, greater<>> freq;
        unordered_map<int, int> count;
        vector<int> res(k, 0);
        for (auto &m:nums)
            ++count[m];
        for (auto &c:count) {
            freq.push(pair<int, int>(c.second, c.first));
            if (freq.size() > k)
                freq.pop();
        }
        for (int i = 0; i < k; ++i) {
            pair<int, int> p = freq.top();
            freq.pop();
            res[i] = p.second;
        }
        return res;
    }
};
```
#### [451 Sort by character frequency](https://leetcode-cn.com/problems/sort-characters-by-frequency/)

Use a large root heap to save the frequency of each character, and then overwrite the original string in sequence.
```c++
class Solution {
    struct Element {
        int asc;
        int num;

        Element() : num(0) {}
    };

    struct Compare {
        bool operator()(Element &e1, Element &e2) {
            return e1.num < e2.num;
        }
    };

public:
    string frequencySort(string s) {
        vector<Element> count(200);
        for (int i = 0; i < 200; ++i)
            count[i].asc = i;
        for (auto c:s)
            ++count[c].num;
        priority_queue<Element, vector<Element>, Compare> heap;
        for (auto &c:count)
            if (c.num != 0)
                heap.push(c);
        int i = 0;
        while (!heap.empty()) {
            auto t = heap.top();
            heap.pop();
            for (int j = 0; j < t.num; ++j)
                s[i++] = t.asc;
        }
        return s;
    }
};
```
#### [692 Top K high-frequency words](https://leetcode-cn.com/problems/top-k-frequent-words/)

Use a pair<string, int> or structure to save information about each string and its count, and maintain a small root heap to ensure that the top K high-frequency words are stored in the heap and all elements in the heap are returned.
```c++
class Solution {
    struct Comp {
        bool operator()(pair<string, int> const &p1, pair<string, int> const &p2) {
            if (p1.second == p2.second)
                return p1.first < p2.first;
            return p1.second > p2.second;
        }
    };

public:
    vector<string> topKFrequent(vector<string> &words, int k) {
        vector<string> res(k);
        unordered_map<string, int> count;
        priority_queue<pair<string, int>, vector<pair<string, int>>, Comp> heap;
        for (auto &word:words)
            ++count[word];
        for (auto &c:count) {
            heap.push(c);
            if (heap.size() > k)
                heap.pop();
        }
        for (int i = 0; i < k; ++i) {
            res[i] = heap.top().first;
            heap.pop();
        }
        reverse(res.begin(), res.end());
        return res;
    }
};
```
#### [778 Swimming in a rising water pool](https://leetcode-cn.com/problems/swim-in-rising-water/)

Similar to the Greedy Best-First Search method used in AI. Greedy cannot be used because optimal solutions may appear in unvisited paths, such as [[0, 4, 7], [5, 8, 9], [2, 3, 1]]. If greedy is used, the matrix will go through the path 0 -> 4 -> 7 -> 9 -> 1, but in fact the optimal path is 0 -> 5 -> 2 -> 3 -> 1. Therefore, it is necessary to save the value of the node that has not been passed before, so that when the current node is larger than the node that has not been passed before, it can return to the previous position and continue the search. The saving method is to use a small root heap to save all the next nodes that may be passed, so that the optimal value of the next step can be taken from the top of the heap every time. The search strategy uses DFS, and tries unvisited nodes in four directions each time. The time complexity is O(n).
```c++
class Solution {
    struct Element {
        int val;
        int x, y;

        Element(int val, int x, int y) : val(val), x(x), y(y) {}
    };

    struct Compare {
        bool operator()(Element const &e1, Element const &e2) {
            return e1.val > e2.val;
        }
    };

    int dir[4][2] = {{0, -1}, {-1, 0}, {0, 1}, {1, 0}};
public:
    int swimInWater(vector<vector<int>> &grid) {
        int m = grid.size(), n = m ? grid[0].size() : 0, res = 0;
        if (m == 0)
            return 0;
        priority_queue<Element, vector<Element>, Compare> heap;
        vector<vector<bool>> visited(m, vector<bool>(n, false));
        heap.push(Element(grid[0][0], 0, 0));
        visited[0][0] = true;
        while (!heap.empty()) {
            Element e = heap.top();
            heap.pop();
            res = max(res, e.val);
            if (e.x == m - 1 && e.y == n - 1)
                break;
            for (auto &d:dir) {
                int x = e.x + d[0], y = e.y + d[1];
                if (x >= 0 && x < m && y >= 0 && y < n && !visited[x][y]) {
                    heap.push(Element(grid[x][y], x, y));
                    visited[x][y] = true;
                }
            }
        }
        return res;
    }
};
```
#### [703 Kth largest element in data stream](https://leetcode-cn.com/problems/kth-largest-element-in-a-stream/)

Maintain a small root heap of size K. If the new number is larger than the top element of the heap, push the number into it and maintain the heap. When the number of elements in the heap exceeds K, pop out the top element of the heap and maintain the heap.
```c++
class KthLargest {
    priority_queue<int, vector<int>, greater<>> min_heap;
    int k;
public:
    KthLargest(int k, vector<int> &nums) {
        this->k = k;
        min_heap = priority_queue<int, vector<int>, greater<>>();
        for (auto &m:nums) {
            if (min_heap.size() < k || min_heap.top() < m)
                min_heap.push(m);
            if (min_heap.size() > k)
                min_heap.pop();
        }
    }

    int add(int val) {
        if (min_heap.size() < k || min_heap.top() < val)
            min_heap.push(val);
        if (min_heap.size() > k)
            min_heap.pop();
        return min_heap.top();
    }
};
```
#### [Median of 295 data streams](https://leetcode-cn.com/problems/find-median-from-data-stream/)

The optimal approach is to use a large root heap and a small root heap. The former stores the second half of the data stream, and the former stores the first half of the data stream. In this way, the tops of the two heaps are respectively the larger and smaller numbers in the middle of the current data stream. Each time you remove the median, you only need to take the top element of the heap. Therefore, the time complexity of fetching the number is O(1), and the time complexity of inserting the number is O(logk), where k is half of the total number of data in the data stream. You can also use binary search plus insertion sort. The time complexity of the insertion is O(n). All numbers in the data stream that are larger than the inserted data need to be moved to the right. The time complexity of the search is O(logn).
```c++
class MedianFinder {
    priority_queue<double> max_heap;
    priority_queue<double, vector<double>, greater<>> min_heap;
public:
    /** initialize your data structure here. */
    MedianFinder() {
        max_heap = priority_queue<double>();
        min_heap = priority_queue<double, vector<double>, greater<>>();
    }

    void addNum(int num) {
        if (max_heap.empty() || max_heap.top() > num)
            max_heap.push(num);
        else
            min_heap.push(num);
        while (max_heap.size() > min_heap.size() + 1) {
            min_heap.push(max_heap.top());
            max_heap.pop();
        }
        while (min_heap.size() > max_heap.size() + 1) {
            max_heap.push(min_heap.top());
            min_heap.pop();
        }
    }

    double findMedian() {
        int ls = max_heap.size(), rs = min_heap.size();
        if ((ls + rs) % 2 == 0)
            return (max_heap.top() + min_heap.top()) * 1.0 / 2;
        return ls > rs ? max_heap.top() : min_heap.top();
    }
};
```

## Original references

- [Reference 1](https://leetcode-cn.com/tag/heap/)
