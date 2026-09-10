---
title: "LeetCode: Two Pointers"
date: 2019-07-31T19:12:25+10:00
draft: false
categories: ["LeetCode"]
description: "A translated technical note on LeetCode: Two Pointers, preserving the examples and context of the original article."
---
# LeetCode: Two Pointers

> Originally published in Chinese on 2019-07-31; this English edition preserves the original scope and technical context.

## Title

#### [26 Remove duplicates from sorted array](https://leetcode-cn.com/problems/remove-duplicates-from-sorted-array/)

Use two pointers len and i to represent the subscripts of items without duplicates and the subscripts of the traversed array respectively. Copy the items without duplicates to nums[len] and then use ++len.
```c++
class Solution {
public:
    int removeDuplicates(vector<int>& nums) {
        int count = 0, len = 1, n = nums.size();
        if (n == 0)
            return 0;
        for (int i = 1; i < n; ++i) {
            if (nums[i] == nums[i - 1])
                continue;
            nums[len] = nums[i];
            ++len;
        }
        return len;
    }
};
```
#### [80 Remove duplicates from sorted array II](https://leetcode-cn.com/problems/remove-duplicates-from-sorted-array-ii/)

Use two pointers len and i to represent the subscripts of items that are not repeated at most 2 times and the subscripts of the traversed array respectively. Copy the items with the number of repetitions less than or equal to 1 to nums[len] and then use ++len.
```c++
class Solution {
public:
    int removeDuplicates(vector<int>& nums) {
        int count = 0, len = 1, n = nums.size();
        if (n == 0)
            return 0;
        for (int i = 1; i < nums.size(); ++i) {
            if (nums[i] == nums[i - 1]) {
                if (count > 0)
                    continue;
                ++count;
            }
            else
                count = 0;
            nums[len] = nums[i];
            ++len;
        }
        return len;
    }
};
```
#### [922 Sort array by parity II](https://leetcode-cn.com/problems/sort-array-by-parity-ii/)

Use two subscripts i and j to represent the subscripts of even digits and odd digits respectively. If the number corresponding to the even digit subscript is not an even number, then exchange it with the number corresponding to the odd digit subscript is not an odd number.
```c++
class Solution {
public:
    vector<int> sortArrayByParityII(vector<int>& A) {
        for (int i = 0, j = 1; i < A.size(); i += 2)
            if (A[i] % 2 != 0) {
                while (j < A.size() && A[j] % 2 != 0)
                    j += 2;
                swap(A[i], A[j]);
            }
        return A;
    }
};
```
#### [11 Container with most water](https://leetcode-cn.com/problems/container-with-most-water/)

Use two pointers to represent the head and tail of the array respectively, move the subscript of the lower element to the middle each time, and update the result at the same time.
```c++
class Solution {
public:
    int maxArea(vector<int>& h) {
        int i = 0, j = h.size() - 1, res = 0;
        while (i < j) {
            res = max(res, min(h[i], h[j]) * (j - i));
            if (h[i] < h[j])
                ++i;
            else
                --j;
        }
        return res;
    }
};
```
#### [287 Find the duplicate number](https://leetcode-cn.com/problems/find-the-duplicate-number/submissions/)

Use the absolute value - 1 of the appearing number as a subscript, and multiply the number at the corresponding position by -1 to mark. Because only one number is repeated, if the number at the corresponding position is found to be negative during marking, it means that the same subscript has appeared, and the number can be returned.
```c++
class Solution {
public:
    int findDuplicate(vector<int>& nums) {
        for (int i = 0; i < nums.size(); ++i) {
            int index = abs(nums[i]) - 1;
            if (nums[index] < 0)
                return abs(nums[i]);
            nums[index] *= -1;
        }
        return 0;
    }
};
```
#### [75 Color Classification](https://leetcode-cn.com/problems/sort-colors/)

Similar to sorting an array with only two numbers, you only need to use two variables idx_0 = 0, idx_2 = n - 1 to represent the subscripts of both ends respectively, replace 0 and 2 at both ends of the array, and leave 1 in the middle.
```c++
class Solution {
public:
    void sortColors(vector<int> &nums) {
        int n = nums.size(), idx_0 = 0, idx_2 = n - 1;
        for (int i = 0; i <= idx_2; ++i) {
            if (nums[i] == 2) {
                swap(nums[i], nums[idx_2]);
                --idx_2, --i;
            } else if (nums[i] == 0) {
                swap(nums[i], nums[idx_0]);
                ++idx_0;
            }
        }
    }
};
```
#### [15 sum of three numbers](https://leetcode-cn.com/problems/3sum/)

First, clarify the method of adding the sum of two numbers: after sorting, use two pointers to traverse from the beginning and the end to the middle, and move the pointers according to the size relationship. The sum of three numbers is nothing more than fixing a number first so that the sum of the other two numbers is equal to the negative of this number. Therefore, the array still needs to be sorted first. In order to fix a number, a for loop needs to be used to traverse the array, and all subsequent elements are summed using the sum of the two numbers. In order to prevent duplication, it is necessary to continuously move the pointer after calculating the sum of the two numbers until the current element is different from the previous/following element. The time complexity is O(n ^ 2) and the space complexity is O(1).
```c++
class Solution {
public:
    vector<vector<int>> threeSum(vector<int> &nums) {
        sort(nums.begin(), nums.end());
        vector<vector<int>> res;
        int n = nums.size(), i = 0;
        while (i < n) {
            int j = i + 1, k = n - 1, target = -nums[i];
            while (j < k) {
                if (nums[j] + nums[k] == target) {
                    res.push_back({nums[i], nums[j], nums[k]});
                    do
                        ++j;
                    while (j < k && nums[j] == nums[j - 1]);
                    do
                        --k;
                    while (j < k && nums[k] == nums[k + 1]);
                } else if (nums[j] + nums[k] < target)
                    ++j;
                else
                    --k;
            }
            do
                ++i;
            while (i < n && nums[i] == nums[i - 1]);
        }
        return res;
    }
};
```
#### [The longest repeating character after 424 replacement](https://leetcode-cn.com/problems/longest-repeating-character-replacement/)

For a substring, we only need to know the number of occurrences of the most frequent character in the substring, and then we can know whether the substring can be replaced by a repeated substring according to j - i + 1 - max_count <= k. Therefore, a sliding window method is used to fix a substring. If the substring meets the conditions, then we will continue to move the right end j of the sliding window back. Otherwise, we need to move the left end back until the substring meets the conditions. j - i + 1 is the length of the longest possible repeated substring. The time complexity is O(n) and the space complexity is O(1).
```c++
class Solution {
public:
    int characterReplacement(string s, int k) {
        int i = 0, j = 0, res = 0, n = s.size(), max_count = 0;
        vector<int> count(26, 0);
        while (j < n) {
            ++count[s[j] - 'A'];
            max_count = max(max_count, count[s[j] - 'A']);
            while (j - i + 1 - max_count > k) {
                --count[s[i] - 'A'];
                ++i;
                for (auto &c:count)
                    max_count = max(max_count, c);
            }
            res = max(res, j - i + 1);
            ++j;
        }
        return res;
    }
};
```
#### [1004 Maximum number of consecutive 1’s III](https://leetcode-cn.com/problems/max-consecutive-ones-iii/)

Use the left and right pointers to ensure that there are less than or equal to K 0s in the sliding window. If the current bit is 1, then the right pointer continues to move backward. If the current bit is 0 and there are already K 0s, then the left pointer moves to the right until 0 appears, skip this 0, treat the 0 of the right pointer as 1, and update the result.
```c++
class Solution {
public:
    int longestOnes(vector<int> &A, int K) {
        int i = 0, res = 0;
        for (int j = 0; j < A.size(); ++j) {
            if (A[j] == 0) {
                if (K > 0)
                    --K;
                else {
                    while (A[i] == 1)
                        ++i;
                    ++i;
                }
            }
            res = max(res, j - i + 1);
        }
        return res;
    }
};
```
#### [42 Trapping rainwater](https://leetcode-cn.com/problems/trapping-rain-water/submissions/)

You can first save the heights of the tallest pillars on the left and right of each location, and then calculate the lower of the two minus the number of pillars at the current location to get the amount of rainwater that the current location can catch.
```c++
class Solution {
public:
    int trap(vector<int> &height) {
        int n = height.size(), res = 0;
        vector<int> left(n, 0), right(n, 0);
        for (int i = 1; i < n; ++i)
            left[i] = max(left[i - 1], height[i - 1]);
        for (int i = n - 2; i >= 0; --i)
            right[i] = max(right[i + 1], height[i + 1]);
        for (int i = 0; i < n; ++i)
            res += max(0, min(left[i], right[i]) - height[i]);
        return res;
    }
};
```
You can also use two variables l_max and r_max to record the highest column heights on the left and right sides respectively. Each time the lower side is checked, the amount of rainwater that can be caught is equal to min(l_max, r_max) minus the current column height, and the maximum height of the column is updated at the same time.
```c++
class Solution {
public:
    int trap(vector<int> &height) {
        int n = height.size(), res = 0, l_max = 0, r_max = 0, i = 0, j = n - 1;
        while (i <= j) {
            if (l_max <= r_max) {
                res += max(0, min(l_max, r_max) - height[i]);
                l_max = max(l_max, height[i]);
                ++i;
            } else {
                res += max(0, min(l_max, r_max) - height[j]);
                r_max = max(r_max, height[j]);
                --j;
            }
        }
        return res;
    }
};
```
#### [632 minimum interval](https://leetcode-cn.com/problems/smallest-range/)

The easier way to think of is to start traversing from the first element of each array, use an array idx to store the subscript of the currently traversed element of each array, and update the maximum and minimum values of these elements each time. The time complexity of doing so is O(m * n), where m is the number of arrays and n is the number of all elements, but this will cause TLE. Instead of traversing the entire two-dimensional array every time, we can use a small root heap to save the minimum value of all currently traversed elements together with their array subscripts and subscripts. In this way, we can get the minimum value of the current elements in all arrays with O(1) time complexity each time, and then use a variable max_val to store the maximum value of the current elements in all arrays. Each time we pop out the top element from the small root heap, res is updated first. The resulting array, and then updates max_val with the value of the next subscript corresponding to this element, until the top element of the heap is the last element of the array. The time complexity is O(m * logn).
```c++
class Solution {
    struct element {
        int val;
        int vec_idx;
        int idx;

        element(int val, int vec_idx, int idx) : val(val), vec_idx(vec_idx), idx(idx) {}
    };

    struct Compare {
        bool operator()(const element &e1, const element &e2) {
            return e1.val > e2.val;
        }
    };

public:
    vector<int> smallestRange(vector<vector<int>> &nums) {
        int n = nums.size(), max_val = INT_MIN;
        vector<int> res(2, 0);
        res[1] = INT_MAX;
        priority_queue<element, vector<element>, Compare> heap;
        for (int i = 0; i < n; ++i) {
            heap.push(element(nums[i][0], i, 0));
            max_val = max(max_val, nums[i][0]);
        }
        while (true) {
            element e = heap.top();
            heap.pop();
            if (res[1] - res[0] > max_val - e.val)
                res[0] = e.val, res[1] = max_val;
            if (e.idx == nums[e.vec_idx].size() - 1)
                break;
            ++e.idx;
            e.val = nums[e.vec_idx][e.idx];
            heap.push(e);
            max_val = max(max_val, e.val);
        }
        return res;
    }
};
```
#### [76 minimum coverage substring](https://leetcode-cn.com/problems/minimum-window-substring/)

First find a string that meets the conditions from left to right, then use the sliding window method to remove one character at a time on the left, find an unused corresponding character to the right, and update the result if the length is smaller than the previously obtained string.
```c++
class Solution {
    struct Element {
        int pos;
        char c;

        Element(int pos, char c) : pos(pos), c(c) {}
    };

public:
    string minWindow(string s, string t) {
        string res;
        unordered_map<char, int> count;
        vector<Element> ele;
        unordered_set<int> used;
        for (auto c:t)
            ++count[c];
        for (int i = 0; i < s.size(); ++i)
            if (count.find(s[i]) != count.end())
                ele.push_back(Element(i, s[i]));
        int i = 0, j = 0, n = ele.size(), pos = 0, l = n, start = 0, end = 0;
        while (j < n && !count.empty()) {
            if (count.find(ele[j].c) != count.end()) {
                --count[ele[j].c];
                used.insert(j);
                end = max(end,j);
                if (count[ele[j].c] == 0)
                    count.erase(ele[j].c);
            }
            ++j;
        }
        if (!count.empty())
            return res;
        res = s.substr(ele[i].pos, ele[j - 1].pos - ele[i].pos + 1);
        while (start < n) {
            char target = ele[start].c;
            used.erase(start);
            ++start;
            int k = start;
            while (k < n) {
                if (ele[k].c == target && used.find(k) == used.end()) {
                    used.insert(k);
                    end = min(n - 1, max(end, k));
                    if (res.size() > ele[end].pos - ele[start].pos + 1)
                        res = s.substr(ele[start].pos, ele[end].pos - ele[start].pos + 1);
                    break;
                }
                ++k;
            }
            if (k == n)
                break;
        }
        return res;
    }
};
```
#### [Subarrays with 992 K different integers](https://leetcode-cn.com/problems/subarrays-with-k-different-integers/)

Use two pointers left and right to ensure that there are K different integers in the subarray in the sliding window. When the count of the leftmost number is greater than 1, it means that the array composed of [left, right] and the array composed of [left + 1, right] are subarrays containing K different integers that conform to the meaning of the question. And if [left + 1, right + 1] is also an array that conforms to the meaning of the question, then [left, right + 1] It is also an array that meets the meaning of the question, so ++acc and ++left are used. When the size of the hash table is equal to K, just add the current acc to the result. The time complexity is O(n) and the space complexity is O(n).
```c++
class Solution {
public:
    int subarraysWithKDistinct(vector<int> &A, int K) {
        unordered_map<int, int> count;
        int acc = 1, res = 0, left = 0;
        for (int right = 0; right < A.size(); ++right) {
            ++count[A[right]];
            while (count.size() > K) {
                --count[A[left]];
                if (count[A[left]] == 0)
                    count.erase(A[left]);
                ++left;
                acc = 1;
            }
            while (count[A[left]] > 1) {
                --count[A[left]];
                ++left;
                ++acc;
            }
            if (count.size() == K)
                res += acc;
        }
        return res;
    }
};
```
#### [239 sliding window maximum](https://leetcode-cn.com/problems/sliding-window-maximum/)

Use a double-ended queue similar to a monotonic stack to store the elements in the sliding window. When the number that needs to be pushed_back is greater than the previous number, it will continue to be smaller than its number pop_back. In this way, the front position of the double-ended queue must be the largest number in the current sliding window. When the sliding window moves, if the leftmost number is equal to the number of the front position in the double-ended queue, pop_front will be used. In this way, the number at the front position is still the largest number in the current sliding window. The time complexity of doing this is O(n) and the space complexity is O(n).
```c++
class Solution {
public:
    vector<int> maxSlidingWindow(vector<int> &nums, int k) {
        int n = nums.size();
        deque<int> d;
        vector<int> res;
        if (n == 0)
            return res;
        for (int i = 0; i < k; ++i) {
            while (!d.empty() && d.back() < nums[i])
                d.pop_back();
            d.push_back(nums[i]);
        }
        res.push_back(d.front());
        for (int i = k; i < n; ++i) {
            if (!d.empty() && nums[i - k] == d.front())
                d.pop_front();
            while (!d.empty() && d.back() < nums[i])
                d.pop_back();
            d.push_back(nums[i]);
            res.push_back(d.front());
        }
        return res;
    }
};
```

## Original references

- [Reference 1](https://leetcode-cn.com/tag/two-pointers/)
