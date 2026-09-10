---
title: "LeetCode: Sorting"
date: 2019-08-17T19:12:25+10:00
draft: false
categories: ["LeetCode"]
description: "A translated technical note on LeetCode: Sorting, preserving the examples and context of the original article."
---
# LeetCode: Sorting

> Originally published in Chinese on 2019-08-17; this English edition preserves the original scope and technical context.

## Title

#### [56 merge intervals](https://leetcode-cn.com/problems/merge-intervals/)

Sort according to the start of the interval, and determine whether the start of the next interval is greater than the end of the previous interval. If it is greater, add the previous interval to the result array, otherwise continue to expand the current interval.
```c++
class Solution {
public:
    vector<vector<int>> merge(vector<vector<int>> &intervals) {
        vector<vector<int>> res;
        int n = intervals.size();
        if (n == 0)
            return res;
        sort(intervals.begin(), intervals.end(), [](vector<int> const &v1, vector<int> const &v2) {
            return v1[0] < v2[0];
        });
        int start = intervals[0][0], end = intervals[0][1];
        for (int i = 1; i < n; ++i) {
            if (intervals[i][0] > end) {
                res.push_back(vector<int>{start, end});
                start = intervals[i][0];
            }
            end = max(end, intervals[i][1]);
        }
        res.push_back(vector<int>{start, end});
        return res;
    }
};
```
#### [179 maximum number](https://leetcode-cn.com/problems/largest-number/)

First convert the numbers into strings, and then customize a sorting rule similar to lexicographic order s1 + s2 > s2 + s1 for sorting.
```c++
class Solution {
public:
    string largestNumber(vector<int> &nums) {
        int i = 0, n = nums.size();
        string res;
        vector<string> strs(n);
        for (i = 0; i < n; ++i)
            strs[i] = to_string(nums[i]);
        sort(strs.begin(), strs.end(), [](const string &s1, const string &s2) {
            return s1 + s2 > s2 + s1;
        });
        for (i = 0; i < n; ++i)
            res += strs[i];
        i = 0;
        while (i < res.size() - 1 && res[i] == '0')
            ++i;
        return res.substr(i);
    }
};
```
#### [324 Wiggle Sort II](https://leetcode-cn.com/problems/wiggle-sort-ii/)

First sort the array, and then place the numbers in the new array in ascending order, so that the numbers in the even digits must be larger than the numbers in the odd digits. If the array length is even, then the initial position is n - 2 (second to last position), otherwise n - 1 (last position), not starting from the first position because of some. The time complexity of doing this is O(nlogn) and the space complexity is O(n).
```c++
class Solution {
public:
    void wiggleSort(vector<int> &nums) {
        int n = nums.size(), i = n % 2 == 0 ? n - 2 : n - 1;
        vector<int> res(n);
        sort(nums.begin(), nums.end());
        for (int j = 0; j < n; ++j) {
            res[i] = nums[j];
            i -= 2;
            if (i < 0)
                i = n % 2 == 0 ? n - 1 : n - 2;
        }
        nums = res;
    }
};
```
#### [524 Matches to the longest word in the dictionary by deleting letters](https://leetcode-cn.com/problems/longest-word-in-dictionary-through-deleting/)

Use double pointers to determine whether each string in the dictionary meets the requirements, and compare the string that meets the requirements with the result string, and find the one with the longest length and the smallest lexicographic order and return it.
```c++
class Solution {
public:
    string findLongestWord(string s, vector<string> &d) {
        string res;
        for (auto &ds:d) {
            int l = 0;
            for (int i = 0; i < s.size(); ++i) {
                if (s[i] == ds[l])
                    ++l;
                if (l == ds.size()) {
                    if (ds.size() > res.size() || (ds.size() == res.size() && ds < res))
                        res = ds;
                    break;
                }
            }
        }
        return res;
    }
};
```
#### [969 Pancake sorting](https://leetcode-cn.com/problems/pancake-sorting/)

Each time you move the largest number in [0, j] to the position of A[j] and --j, you need to flip it twice each time. One is to flip the largest number in [0, j] to A[0], and the other is to flip A[0] to A[j]. This ensures that the largest number is moved to the end of [0, j] every time. Repeat this process until j is reduced to 1. At this time, there is only 1 number left, and the loop ends. Because the flip operation is done within the loop, the time complexity is O(n^2) and the space complexity is O(1).
```c++
class Solution {
public:
    vector<int> pancakeSort(vector<int> &A) {
        vector<int> res;
        int n = A.size(), j = n;
        while (j > 1) {
            int idx = 0;
            for (int i = 0; i < j; ++i)
                if (A[i] > A[idx])
                    idx = i;
            if (idx != j - 1) {
                if (idx != 0) {
                    reverse(A.begin(), A.begin() + idx + 1);
                    res.push_back(idx + 1);
                }
                reverse(A.begin(), A.begin() + j);
                res.push_back(j);
            }
            --j;
        }
        return res;
    }
};
```
#### [1122 Relative sorting of arrays](https://leetcode-cn.com/problems/relative-sort-array/)

Assign weights to the numbers in arr2 according to the subscripts, and sort them according to the weight array.
```c++
class Solution {
public:
    vector<int> relativeSortArray(vector<int>& arr1, vector<int>& arr2) {
        vector<int> auth(1001);
        for (int i = 0; i <= 1000; ++i)
            auth[i] = 1001 + i;
        for (int i = 0; i < arr2.size(); ++i)
            auth[arr2[i]] = i;
        sort(arr1.begin(), arr1.end(), [&](const int &c1, const int &c2) {
            return auth[c1] < auth[c2];
        });
        return arr1;
    }
};
```

## Original references

- [Reference 1](https://leetcode-cn.com/tag/sort/)
