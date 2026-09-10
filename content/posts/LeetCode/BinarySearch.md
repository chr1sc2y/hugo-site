---
title: "LeetCode: Binary Search"
date: 2019-06-23T19:26:39+10:00
draft: false
categories: ["LeetCode"]
description: "A translated technical note on LeetCode: Binary Search, preserving the examples and context of the original article."
---
# LeetCode: Binary Search

> Originally published in Chinese on 2019-06-23; this English edition preserves the original scope and technical context.

Binary search can find qualified values in an ordered array with high efficiency. The time complexity is O(logN) and the space complexity is O(1).

## Easy to make mistakes

1. How to calculate the intermediate value

    - k = i + (j - i) / 2
    - k = (i + j) / 2

    The second method will generally cause integer data to overflow, so only the first method is used.

2. Loop conditions

    - If you want to find a unique value, and the lower limit i and upper limit j will be +1 or -1 based on the intermediate value k when updating, then the loop condition is i <= j, j can be obtained, j = nums.size() - 1, and the intermediate value is calculated using k = i + (j - i + 1) / 2

    - If you want to find a value that is greater than or equal to or less than or equal to a certain condition, and one of the lower limit i and the upper limit j will not be +1 or -1 based on k, then the loop condition is i < j, j cannot be taken, j = nums.size(), and the intermediate value is calculated using k = i + (j - i) / 2

    The two methods have their own application scenarios. If they are not used correctly, boundary value problems will occur, or an infinite loop will cause TLE.

## Title

### 1. Find the number

#### [704 Binary Search](https://leetcode-cn.com/problems/binary-search/)

Searches the sorted array for the target value, returning the subscript if present, otherwise -1.

The most standard binary search. Because the lower limit i and the upper limit j will both be +1 and -1 when updated, let j = nums.size() - 1, and the loop condition is i <= j.
```c++
class Solution {
public:
    int search(vector<int> &nums, int target) {
        int i = 0, j = nums.size() - 1, k = 0;
        while (i <= j) {
            k = i + (j - i + 1) / 2;
            if (nums[k] > target)
                j = k - 1;
            else if (nums[k] < target)
                i = k + 1;
            else
                return k;
        }
        return -1;
    }
};
```
#### [374 Guess the number size](https://leetcode-cn.com/problems/guess-number-higher-or-lower/)

Select a number from 1 to n, and get -1: the number is hit through a predefined interface guess(int num); 1: the number is small; 0: the guess is correct.

It's almost the same as the previous question, except that the target is replaced by an interface.
```c++
int guess(int num);

class Solution {
public:
    int guessNumber(int n) {
        int i = 1, j = n, k = 0;
        while (i <= j) {
            k = i + (j - i + 1) / 2;
            auto res = guess(k);
            if (res == 1)
                i = k + 1;
            else if (res == -1)
                j = k - 1;
            else
                return k;
        }
        return 0;
    }
};
```
#### [367 valid perfect square numbers](https://leetcode-cn.com/problems/valid-perfect-square/)

Determine whether a positive integer is a perfect square.

Determine whether a number is a perfect square. Because the result obtained by squaring during the bisection process may exceed the upper limit of the 32-bit int type, long long is used.
```c++
class Solution {
public:
    bool isPerfectSquare(int num) {
        long long i = 0, j = num, k = 0, res = 0;
        while (i <= j) {
            k = i + (j - i + 1) / 2;
            res = k * k;
            if (res < num)
                i = k + 1;
            else if (res > num)
                j = k - 1;
            else
                return true;
        }
        return j * j == num;
    }
};
```
#### [The square root of 69 x](https://leetcode-cn.com/problems/sqrtx/)

Calculate the square root of a number, keeping only the integer part.

Implement the int sqrt(int x) function. It is more intuitive to judge by quotient than by product.
```c++
class Solution {
public:
    int mySqrt(int x) {
        if (x <= 1)
            return x;
        int i = 1, j = x, k = 0, sqrt = 0;
        while (i <= j) {
            k = i + (j - i + 1) / 2;
            sqrt = x / k;
            if (sqrt < k)
                j = k - 1;
            else if (sqrt > k)
                i = k + 1;
            else
                return k;
        }
        return j;
    }
};
```
### 2. Find upper/lower bounds

#### [35 Search insertion position](https://leetcode-cn.com/problems/search-insert-position/)

Find the target value in the sorted array, and if it does not exist, return the position where it will be inserted in order, with no duplicate elements in the array.

Find the first number greater than or equal to target. Because the upper limit j will be directly assigned to the value of k when updating, let j = nums.size(), and the loop condition is i < j.
```c++
class Solution {
public:
    int searchInsert(vector<int>& nums, int target) {
        int i = 0, j = nums.size(), k = 0;
        while (i < j) {
            k = i + (j - i) / 2;
            if (nums[k] < target)
                i = k + 1;
            else if (nums[k] > target)
                j = k;
            else
                return k;
        }
        return j;
    }
};
```
#### [744 Find the smallest letter greater than the target letter](https://leetcode-cn.com/problems/find-smallest-letter-greater-than-target/)

Find the smallest letter that is larger than the target letter in an ordered array. The letters in the array are cyclic.

Find the first number that is greater than the target. Compared with the previous question, there is only one less need to judge whether it is equal within the loop. From an int type array to a char type array, this has no impact.
```c++
class Solution {
public:
    char nextGreatestLetter(vector<char>& letters, char target) {
        int n = letters.size(), i = 0, j = n, k = 0;
        while (i < j) {
            k = i + (j - i) / 2;
            if (letters[k] <= target)
                i = k + 1;
            else if (letters[k] > target)
                j = k;
        }
        return j < n ? letters[j] : letters[0];
    }
};
```
#### [278 The first bad version](https://leetcode-cn.com/problems/first-bad-version/)

Products are developed based on previous versions. All versions after any wrong version are wrong. Find the first version with the error. Use an interface bool isBadVersion(version) to determine whether the version is wrong.

It's very standard to find the lower bound.
```c++
bool isBadVersion(int version);

class Solution {
public:
    int firstBadVersion(long long n) {
        long long i = 0, j = n + 1, k = 0;
        while (i < j) {
            k = i + (j - i) / 2;
            if (!isBadVersion(k))
                i = k + 1;
            else
                j = k;
        }
        return j;
    }
};
```
#### [875 Koko who loves eating bananas](https://leetcode-cn.com/problems/koko-eating-bananas/)

There are N piles of bananas. If you eat one pile per hour without eating the other pile, calculate the slowest speed that can be eaten within H hours.

Use speed as a binary search variable, and judge each time whether all the bananas can be eaten at the current speed. If so, j = k, and k may be the final result. Otherwise, i = k + 1, and k at this time must be smaller than the result.
```c++
class Solution {
public:
    int minEatingSpeed(vector<int>& piles, int H) {
        int i = 1, j = INT_MAX, k = 0;
        while (i < j) {
            k = i + (j - i) / 2;
            int res = CanEatAll(k, H, piles);
            if (res)
                j = k;
            else
                i = k + 1;
        }
        return j;
    }

    bool CanEatAll(const int &speed, int hour, vector<int>& piles) {
        for (auto &p:piles)
            hour -= p / speed + (p % speed > 0);
        return hour >= 0;
    }
};
```
### 3. Search based on location relationship

#### [378 The Kth smallest element in a sorted matrix](https://leetcode-cn.com/problems/kth-smallest-element-in-a-sorted-matrix/)

Sort each row and column in the n x n matrix in ascending order and find the kth smallest element in the matrix.

This question can be solved by combining the idea of ​​​​searching in the two-dimensional array in the sword offer with the binary search. When doing the binary search, you can get the result by calculating the number of numbers less than or equal to the middle value in the matrix each time. The time complexity is O(logm * n), m is matrix[n - 1][n - 1], and n is matrix.size().
```c++
class Solution {
public:
    int kthSmallest(vector<vector<int>>& matrix, int m) {
        int n = matrix.size(), i = matrix[0][0], j = matrix[n - 1][n - 1] + 1, k = 0;
        while (i < j) {
            k = i + (j - i) / 2;
            auto res = CountLess(matrix, k);
            if (res < m)
                i = k + 1;
            else
                j = k;
        }
        return i;
    }

    int CountLess(vector<vector<int>>& matrix, const int &target) {
        int n = matrix.size(), i = 0, j = n - 1, res = 0;
        while (i < n && j >= 0) {
            if (matrix[i][j] <= target)
                res += j + 1, ++i;
            else
                --j;
        }
        return res;
    }
};
```
#### [153 Find the minimum value in rotated sorted array](https://leetcode-cn.com/problems/find-minimum-in-rotated-sorted-array/)

A sorted array is rotated at a certain point to find the smallest element.

Judging based on the relationship between the middle value, the left and right side values ​​and the one digit on the left and right sides, if the middle value k is smaller than the left value, then k may be the result, let j = k; otherwise, compare the middle value with its right digit, if the middle value is greater than the value of the right digit, then the value of the right digit is the result, otherwise let i = k + 1.
```c++
class Solution {
public:
    int findMin(vector<int>& nums) {
        int n = nums.size(), i = 0, j = nums.size(), k = 0;
        if (n == 1 || nums[0] < nums[n - 1])
            return nums[0];
        else if (n == 2)
            return min(nums[0], nums[1]);
        while (i < j) {
            k = i + (j - i) / 2;
            if (nums[i] > nums[k])
                j = k;
            else if (k + 1 < j && nums[k + 1] > nums[i])
                i = k + 1;
            else
                return nums[k + 1];
        }
        return j >= n ? nums[i] : min(nums[i], nums[j]);
    }
};
```
#### [540 single elements in a sorted array](https://leetcode-cn.com/problems/single-element-in-a-sorted-array/)

In a sorted array, each element appears twice, and only one number appears once. Find this number.

Every other element in the sorted array appears twice, find the only number that appears only once. The equality relationship between odd and even bits will change after the unique number appears. Use this to make judgments. A more intuitive way of writing is to judge the situations of k % 2 == 0 and k % 2 == 1 respectively. This is more complicated to write. You can directly use --k when k % 2 == 1, and then directly judge the relationship between nums[k] and nums[k + 1].
```c++
class Solution {
public:
    int singleNonDuplicate(vector<int> &nums) {
        int i = 0, j = nums.size() - 1, k = 0, n = nums.size();
        while (i <= j) {
            k = i + (j - i + 1) / 2;
            cout << k << endl;
            if (k % 2 == 0) {
                if (k + 1 < n && nums[k] == nums[k + 1])
                    i = k + 1;
                else if (k - 1 >= 0 && nums[k] == nums[k - 1])
                    j = k - 1;
                else
                    return nums[k];
            } else {
                if (k + 1 < n && nums[k] == nums[k + 1])
                    j = k - 1;
                else if (k - 1 >= 0 && nums[k] == nums[k - 1])
                    i = k + 1;
                else
                    return nums[k];
            }
        }
        return 0;
    }
};
```

```c++
class Solution {
public:
    int singleNonDuplicate(vector<int> &nums) {
        int i = 0, j = nums.size(), k = 0, n = nums.size();
        while (i < j) {
            k = i + (j - i) / 2;
            if (k % 2 == 1)
                --k;
            if (k + 1 < n && nums[k] == nums[k + 1])
                i = k + 2;
            else
                j = k;
        }
        return nums[j];
    }
};
```
### 4. Comprehensive

#### [Find target value in 1095 mountains array](https://leetcode-cn.com/problems/find-in-mountain-array/)

Finds the smallest index in an array of mountains that is equal to the target value.

Because it is known that the array must be a mountain array, there must be a unique top of the mountain. Use binary search to find the subscript of the top of the mountain. If the top of the mountain is less than the target value, then there must be no target value in the array, and -1 is returned. Otherwise, use binary search on the left side of the top of the mountain to find the target value. If not, use binary search on the right side of the top of the mountain to find the target value.
```c++
class Solution {
public:
    int findInMountainArray(int target, MountainArray &mountainArr) {
        int n = mountainArr.length(), i = 0, j = n - 1, k = 0, peak = 0;
        while (i <= j) {
            k = i + (j - i + 1) / 2;
            if (mountainArr.get(k) > mountainArr.get(k - 1) && mountainArr.get(k) > mountainArr.get(k + 1))
                break;
            else if (mountainArr.get(k) < mountainArr.get(k - 1))
                j = k - 1;
            else
                i = k + 1;
        }
        if (mountainArr.get(k) < target)
            return -1;
        else if (mountainArr.get(k) == target)
            return k;

        peak = k;
        i = 0, j = peak;
        while (i < j) {
            k = i + (j - i) / 2;
            if (mountainArr.get(k) < target)
                i = k + 1;
            else
                j = k;
        }
        if (mountainArr.get(j) == target)
            return j;

        i = peak + 1, j = n;
        while (i < j) {
            k = i + (j - i) / 2;
            if (mountainArr.get(k) < target)
                j = k - 1;
            else if (mountainArr.get(k) > target)
                i = k + 1;
            else
                j = k;
        }
        if (i < n && mountainArr.get(i) == target)
            return i;
        return -1;
    }
};
```
### 5. Guess the number

#### [719 Find the kth smallest distance pair](https://leetcode-cn.com/problems/find-k-th-smallest-pair-distance/)

The simplest way is to traverse twice to calculate the difference of all number pairs and save them in a small root heap, and then find the minimum distance less than or equal to k in sequence, but the time complexity of doing so is O(n ^ 2) and will time out. We can sort the array first, and then use low = 0, high = nums[n - 1] - nums[0] to represent the minimum and maximum values of the possible results, and use the dichotomy method to determine whether the intermediate value mid satisfies the difference between the number pairs less than or equal to mid and whether the number of differences is less than or equal to k. When judging, because the array is ordered, we can use double pointers to keep the difference in the middle less than or equal to mid, thereby calculating the number of differences between pairs less than or equal to mid. The time complexity is O(nlogn + nlogm), where n is the length of the array, m is the difference between the maximum value and the minimum value of the array, nlogn is the average time complexity of sorting, and in nlogm n is the time complexity of using double pointers, and logm is the time complexity of the dichotomy method.
```c++
class Solution {
public:
    int smallestDistancePair(vector<int> &nums, int k) {
        sort(nums.begin(), nums.end());
        int n = nums.size(), low = 0, high = nums[n - 1] - nums[0], mid = 0;
        while (low < high) {
            mid = low + (high - low) / 2;
            if (IsDistMoreThanK(nums, mid, k))
                high = mid;
            else
                low = mid + 1;
        }
        return high;
    }

    bool IsDistMoreThanK(const vector<int> &nums, int m, const int &k) {
        int left = 0, right = 0, count = 0;
        while (right < nums.size()) {
            while (nums[right] - nums[left] > m)
                ++left;
            count += right - left;
            ++right;
        }
        return count >= k;
    }
};
```

## Original references

- [Reference 1](https://leetcode-cn.com/tag/binary-search/)
