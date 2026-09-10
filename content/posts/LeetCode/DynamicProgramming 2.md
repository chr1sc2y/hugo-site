---
title: "LeetCode: Dynamic Programming (2)"
date: 2019-06-28T10:09:13+10:00
draft: false
categories: ["LeetCode"]
description: "A translated technical note on LeetCode: Dynamic Programming (2), preserving the examples and context of the original article."
---
# LeetCode: Dynamic Programming (2)

> Originally published in Chinese on 2019-06-28; this English edition preserves the original scope and technical context.

## Title

### 3. Array related

#### [300 Longest Increasing Subsequence](https://leetcode-cn.com/problems/longest-increasing-subsequence/submissions/)

Find the length of the longest ascending subsequence in an unordered array.

Use an array dp[i] to represent the longest rising subsequence up to the i-th number, and traverse each number j before i each time. If nums[i] > nums[j], then j and i can form a rising subsequence, and let dp[i] = max(dp[i], dp[j] + 1) to get the longest rising subsequence.
```c++
class Solution {
public:
    int lengthOfLIS(vector<int>& nums) {
        int n = nums.size(), res = 2;
        if (n <= 1)
            return n;
        vector<int> dp(n, 1);
        for (int i = 1; i < n; ++i)
            for (int j = 0; j < i; ++j)
                if (nums[i] > nums[j]) {
                    dp[i] = max(dp[i], dp[j] + 1);
                    res = max(res, dp[i]);
                }
        return res;
    }
};
```
#### [53 maximum subsequence sum](https://leetcode-cn.com/problems/maximum-subarray/)

Find the sum of consecutive subarrays in an array that has the largest sum.

Use a variable val from the beginning to save the sum of consecutive subarrays up to the current number. Each number has only two choices of adding or not adding to the sum of previous consecutive subarrays. When the sum of previous consecutive subarrays is greater than 0, add the sum of previous consecutive subarrays, otherwise do not add.
```c++
class Solution {
public:
    int maxSubArray(vector<int> &nums) {
        int res = INT_MIN, val = 0;
        for (auto &n:nums) {
            val = max(val, 0) + n;
            res = max(res, val);
        }
        return res;
    }
};
```
#### [718 longest repeated subarray](https://leetcode-cn.com/problems/maximum-length-of-repeated-subarray/)

Given two arrays, find the length of the longest common subarray in the two arrays.

For two characters A[i] and B[j], if A[i] == B[j], it means A[i], B[j] and the subarray before them may be a common subarray, and their maximum length is dp[i][j] = dp[i - 1][j - 1] + 1. Otherwise, they cannot form a common subarray, dp[i][j] = 0. Just use two levels of loops to traverse two arrays. The time complexity is O(m * n) and the space complexity is O(m * n).
```c++
class Solution {
public:
    int findLength(vector<int> &A, vector<int> &B) {
        int m = A.size(), n = B.size(), res = 0;
        if (m == 0 || n == 0)
            return 0;
        vector<vector<int>> dp(m + 1, vector<int>(n + 1, 0));
        for (int i = 1; i <= m; ++i)
            for (int j = 1; j <= n; ++j) {
                dp[i][j] = A[i - 1] == B[j - 1] ? dp[i - 1][j - 1] + 1 : 0;
                res = max(res, dp[i][j]);
            }
        return res;
    }
};
```
#### [983 lowest ticket price](https://leetcode-cn.com/problems/minimum-cost-for-tickets/)

Given all the dates to be traveled, there are three types of passes: one-day pass, seven-day pass, and thirty-day pass. Seek minimum consumption.

For the minimum consumption on day i, you only need to select the lowest consumption of the three types of consumption: the minimum consumption one day ago plus the consumption of one-day tickets, the minimum consumption seven days ago plus seven-day tickets, and the minimum consumption thirty days ago plus thirty-day tickets. Therefore, there is a state transfer equation dp[i] = min({dp[i - 1] + costs[0], dp[i - 7] + costs[1], dp[i - 30] + costs[2]}).
```c++
class Solution {
public:
    int mincostTickets(vector<int> &days, vector<int> &costs) {
        vector<int> dp(366, INT_MAX);
        dp[0] = 0;
        int j = 0, n = days.size();
        for (int i = 1; i <= 365 && j < n; ++i) {
            if (i == days[j]) {
                dp[i] = min({dp[i - 1] + costs[0], dp[max(0, i - 7)] + costs[1], dp[max(0, i - 30)] + costs[2]});
                ++j;
            } else
                dp[i] = dp[i - 1];
        }
        return dp[days[n - 1]];
    }
};
```
#### [813 Grouping of the largest sum of averages](https://leetcode-cn.com/problems/largest-sum-of-averages/)

Divide the array into K adjacent non-empty sub-arrays and find the maximum sum of the averages of all sub-arrays.

Use a two-dimensional array dp[n][K] to represent the optimal value obtained by dividing the first i numbers into k groups. Each time, the j-th number to the n-th number are divided into one group, and the first j - 1 numbers are divided into k - 1 groups to obtain the maximum value of dp[i][k]. In order to quickly calculate the sum of the j-th number to the n-th number and the sum of the first j - 1 numbers, you can use a prefix sum array to save the sum of the first m numbers, and then use the state transition equation dp[i][k] = max(dp[i][k], dp[j][k - 1] + (pre[i] - pre[j]) / (i - j)) to find the optimal value.
```c++
class Solution {
public:
    double largestSumOfAverages(vector<int> &A, int K) {
        int n = A.size();
        if (n == 0)
            return 0;
        vector<double> pre(n + 1, 0);
        for (int i = 1; i <= n; ++i)
            pre[i] += pre[i - 1] + A[i - 1];
        vector<vector<double>> dp(n + 1, vector<double>(K + 1, 0));
        for (int i = 1; i <= n; ++i) {
            dp[i][1] = pre[i] / i;
            for (int k = 2; k <= K && k <= i; ++k)
                for (int j = 1; j < i; ++j)
                    dp[i][k] = max(dp[i][k], dp[j][k - 1] + (pre[i] - pre[j]) / (i - j));
        }
        return dp[n][K];
    }
};
```
#### [646 longest pair chain](https://leetcode-cn.com/problems/maximum-length-of-pair-chain/)

Sort the first element of each number pair from small to large, starting from the second number pair, and judge whether it and all the number pairs before it are consistent with the meaning of the question. If so, then dp[j] = max(dp[j], dp[i] + 1). The time complexity is O(n^2).
```c++
class Solution {
public:
    int findLongestChain(vector<vector<int>>& pairs) {
        sort(pairs.begin(), pairs.end());
        int n = pairs.size(), res = 1;
        vector<int> dp(n, 1);
        for (int j = 1; j < n; ++j)
            for (int i = 0; i < j; ++i)
                if (pairs[j][0] > pairs[i][1]) {
                    dp[j] = max(dp[j], dp[i] + 1);
                    res = max(res, dp[j]);
                }
        return res;
    }
};
```
### 4. Arithmetic sequence

#### [413 Arithmetic Sequence Division](https://leetcode-cn.com/problems/arithmetic-slices/)

Given an array, count the number of difference subarrays in the array.

The arithmetic sequence must be a subarray of length greater than 3 where the difference between two adjacent elements is equal, so you only need to know nums[i] - nums[i - 1] == nums[i - 1] - nums[i - 1]. Use a variable cul to represent the length of the arithmetic sequence so far, and diff to represent the previous tolerance. If the current difference is equal to diff, add cul. The time complexity is O(n) and the space complexity is O(1).
```c++
class Solution {
public:
    int numberOfArithmeticSlices(vector<int>& nums) {
        int res = 0, n = nums.size(), cul = 1, curr = 0;
        if (n < 3)
            return 0;
        int diff = nums[1] - nums[0];
        for (int i = 2; i < n; ++i) {
            if ((curr = nums[i] - nums[i - 1]) == diff) {
                res += cul;
                ++cul;
            }
            else {
                diff = curr;
                cul = 1;
            }
        }
        return res;
    }
};
```
#### [446 Arithmetic Slices II - Subsequence](https://leetcode-cn.com/problems/arithmetic-slices-ii-subsequence/)

Given an array, count the number of difference subsequences in the array.

For a certain number nums[i], it is known that the difference between it and the previous number is diff = nums[i] - nums[j]. You need to know the maximum number of arithmetic subsequences with a tolerance of diff before nums[j]. You can use a hash table array to represent the maximum number of arithmetic subsequences with a tolerance of diff before the number nums[j]. If nums[i] - nums[j] == diff, then go to the position The number of arithmetic subsequences whose tolerance is diff up to i dp[i][diff] += dp[j][diff]. Note that += should be used instead of =, because if there are multiple identical numbers before nums[i], each one needs to be counted as an independent arithmetic subsequence. The time complexity is O(n ^ 2) and the space complexity is O(n ^ 2). What’s very funny is that when the dynamic programming solution to this question was submitted on LeetCode, the runtime was 1000ms +-, but when submitted on LeetCode-CN, the execution time was 1400ms ~ 1800ms, and it occasionally times out. The tolerance of the timeout [case](https://leetcode-cn.com/submissions/detail/21590899/testcase/) exceeds The representation range of int32, if you add the judgment diff < INT_MIN || diff > INT_MAX and continue directly, it will pass normally.
```c++
class Solution {
public:
    int numberOfArithmeticSlices(vector<int>& A) {
        int n = A.size(), res = 0;
        long long diff = 0;
        vector<unordered_map<long long, int>> dp(n);
        for (int i = 1; i < n; ++i) {
            for (int j = 0; j < i; ++j) {
                diff = static_cast<long long>(A[i]) - A[j];
                if (diff < INT_MIN || diff > INT_MAX)
                    continue;
                dp[i][diff] += dp[j][diff] + 1;
                res += dp[j][diff];
            }
        }
        return res;
    }
};
```
#### [1027 Longest Arithmetic Sequence](https://leetcode-cn.com/problems/longest-arithmetic-sequence/submissions/)

Given an array, calculate the length of the longest arithmetic subsequence in the array.

For a certain number nums[i], it is known that the difference between it and the previous number is diff = nums[i] - nums[j]. You need to know how long the arithmetic subsequence with a tolerance of diff before nums[j] is. You can use a hash table array to represent the longest length of the arithmetic subsequence with a tolerance of diff before each number. On this basis, +1 can be used to get the length of the longest arithmetic subsequence. The time complexity is O(n ^ 2) and the space complexity is O(n ^ 2).
```c++
class Solution {
public:
    int longestArithSeqLength(vector<int> &A) {
        int n = A.size(), res = 2;
        vector<unordered_map<int, int>> dp(n);
        for (int i = 1; i < n; ++i) {
            for (int j = 0; j < i; ++j) {
                int diff = A[i] - A[j];
                if (dp[j].find(diff) == dp[j].end())
                    dp[i][diff] = 2;
                else {
                    dp[i][diff] = dp[j][diff] + 1;
                    res = max(res, dp[i][diff]);
                }
            }
        }
        return res;
    }
};
```
### 5. Fibonacci Sequence

#### [70 Climbing Stairs](https://leetcode-cn.com/problems/climbing-stairs/)

If you can climb 1 or 2 stairs each time, how many ways can you climb n stairs?

The method of climbing to the current staircase is equal to the sum of the methods of climbing the previous two stairs, so there is a state transition equation dp[i] = dp[i - 1] + dp[i - 2], and because the current state only depends on the first two states, only two variables s1 and s2 can be used to save the results of the first two states. The time complexity is O(n) and the space complexity is O(1).
```c++
class Solution {
public:
    int climbStairs(int n) {
        if (n <= 1)
            return 1;
        int s1 = 1, s2 = 1, temp;
        for (int i = 1; i < n; ++i) {
            temp = s2;
            s2 += s1;
            s1 = temp;
        }
        return s2;
    }
};
```
#### [746 Use minimum cost to climb stairs](https://leetcode-cn.com/problems/min-cost-climbing-stairs/)

You can climb 1 or 2 stairs each time. Each staircase has a weight. Find the minimum cost of climbing n stairs.

Similar to the previous question, but each time you only need to choose the smaller weight among the first two stairs.
```c++
class Solution {
public:
    int minCostClimbingStairs(vector<int>& cost) {
        int n = cost.size();
        if (n == 0)
            return 0;
        else if (n == 1)
            return cost[0];
        else if (n == 2)
            return cost[0] + cost[1];
        int s1 = cost[0], s2 = cost[1];
        for (int i = 2; i < n; ++i) {
            int temp = s2;
            s2 = min(s1, s2) + cost[i];
            s1 = temp;
        }
        return min(s1, s2);
    }
};
```
#### [740 Delete and Earn Points](https://leetcode-cn.com/problems/delete-and-earn/)

Given an array, select one nums[i] at a time, obtain the number of nums[i] multiplied by the number of points in nums[i], and calculate the maximum number of points that can be obtained.

For nums[i], the maximum number of points that can be obtained can only be the maximum number of points of nums[i - 1] or the maximum number of points of nums[i - 2] plus the number of points that can be obtained by nums[i]. Therefore, there is a state transition equation dp[i] = max(dp[i - 1], dp[i - 2] + val[i]), and the time complexity is O(n). Each point is only related to its two previous points, so only two variables p1 and p2 can be used to save the results of the first two points, and the space complexity is O(1).
```c++
class Solution {
public:
    int deleteAndEarn(vector<int>& nums) {
        vector<int> val(10001, 0);
        for (auto &m:nums)
            val[m] += m;
        int res = 0, prev = 0, curr = 0;
        for (int i = 1; i <= 10000; ++i) {
            res = max(curr, prev + val[i]);
            prev = curr;
            curr = res;
        }
        return res;
    }
};
```
#### [198 House Robber](https://leetcode-cn.com/problems/house-robber/)

Given an array with weights, take non-adjacent numbers and find the maximum value that can be obtained.

For a certain point, if the maximum value that can be obtained until the previous point is greater than the sum of the current point and the maximum value of the previous two points, then it is not taken. Otherwise, the current point is taken. The sum of the value of the current point and the maximum value of the two previous points is the optimal value of the current point. Therefore, there is a state transition equation dp[i] = max(dp[i - 1], dp[i - 2] + nums[i]), and the time complexity is O(n). The current point is only related to the previous two points, so only two variables p1 and p2 can be used to save the results of the previous two points, and the space complexity is O(1).
```c++
class Solution {
public:
    int rob(vector<int>& nums) {
        int n = nums.size();
        if (n <= 2)
            return n == 0 ? 0 : n == 1 ? nums[0] : max(nums[0], nums[1]);
        int p1 = nums[0], p2 = max(nums[0], nums[1]), temp;
        for (int i = 2; i < n; ++i) {
            temp = p2;
            p2 = max(p1 + nums[i], p2);
            p1 = temp;
        }
        return p2;
    }
};
```
#### [213 House Robber II](https://leetcode-cn.com/problems/house-robber-ii/)

Given an array with weights, the arrays are adjacent from beginning to end, take non-adjacent numbers, and find the maximum value that can be obtained.

The adjacent ends of the array means that the first point and the last point cannot be taken at the same time. Therefore, the dynamic programming from the first point to the penultimate point will get the maximum value that can be obtained by including the first point but not including the last point. The dynamic programming from the second point to the last point will get the maximum value that can be obtained by including the last point but not including the first point. The same method used in the previous question can be done twice to get the result. The time complexity is O(n) and the space complexity is O(1).
```
class Solution {
public:
    int rob(vector<int>& nums) {
        int res = 0, n = nums.size();
        if (n <= 2)
            return n == 0 ? 0 : n == 1 ? nums[0] : max(nums[0], nums[1]);
        int p1 = nums[0], p2 = max(nums[0], nums[1]);
        for (int i = 2; i < n - 1; ++i) {
            int temp = p2;
            p2 = max(p2, p1 + nums[i]);
            p1 = temp;
        }
        res = p2;
        p1 = nums[1], p2 = max(nums[1], nums[2]);
        for (int i = 3; i < n; ++i) {
            int temp = p2;
            p2 = max(p2, p1 + nums[i]);
            p1 = temp;
        }
        return max(res, p2);
    }
};
```
#### [337 House Robber III](https://leetcode-cn.com/problems/house-robber-iii/)

Given a weighted binary tree, take non-adjacent numbers and find the maximum value that can be obtained.

For a node, if you take the node itself, you cannot take two child nodes. If you take two child nodes, you cannot take its own and four child nodes. Therefore, compare the sum of its two child nodes with the sum of itself and the four grandchild nodes, and return the maximum value, that is, max(dp[node->left->left] + dp [node->left->right] + dp[node->right->left] + dp [node->right->right] + dp[node], dp[node->left] + dp[node->right]). Because recursion will cause a lot of repeated calculations, a hash table is used to save the optimal value of the node that has been calculated. When recursing to the node, the value is directly obtained to prevent TLE. The time complexity is O(n) and the space complexity is O(n).
```c++
class Solution {
    unordered_map<TreeNode *, int> val;
public:
    int rob(TreeNode *node) {
        if (!node)
            return 0;
        if (val.find(node) != val.end())
            return val[node];
        int p1 = 0, p2 = 0;
        if (node->left)
            p1 += rob(node->left->left) + rob(node->left->right);
        if (node->right)
            p1 += rob(node->right->left) + rob(node->right->right);
        p1 += node->val;
        p2 += rob(node->left) + rob(node->right);
        val[node] = max(p1, p2);
        return val[node];
    }
}
```
#### [96 different binary search trees](https://leetcode-cn.com/problems/unique-binary-search-trees/)

Find how many binary search trees there are with n nodes.

For a certain number i as the root node, no matter what i is, its left subtree is always composed of i - 1 nodes, and its right node is always composed of n - i nodes. For example, when n = 3, if 3 is used as the root node, then its left subtree must be composed of two nodes, 1 and 2. Then we only need to know how many types of binary search trees are composed of two nodes, and then use the number of types of this left subtree dp[2] Multiply the number of types of the right subtree dp[0] to know the number of types composed of 3 nodes, with 3 as the root node. Secondly, we need to let 1 and 2 be the root nodes in turn. Then their left subtrees have dp[0] and dp[1] respectively. Therefore, the state transition equation dp[i] += dp[j - 1] * dp[i - j] is obtained, and the time complexity is O(n ^ 2), the space complexity is O(n).
```c++
class Solution {
public:
    int numTrees(int n) {
        vector<int> dp(n + 1, 0);
        dp[0] = dp[1] = 1;
        for (int i = 2; i <= n; ++i)
            for (int j = 1; j <= i; ++j)
                dp[i] += dp[j - 1] * dp[i - j];
        return dp[n];
    }
};
```
#### [873 The length of the longest Fibonacci subsequence](https://leetcode-cn.com/problems/length-of-longest-fibonacci-subsequence/)

Given a strictly increasing array, find the length of the longest Fibonacci subsequence in it.

According to the definition of the Fibonacci sequence, to determine whether A[i] and A[j] can form a Fibonacci sequence in the original array, you only need to know whether A[i - j] is in the original array, and whether A[i] - A[j] < A[j] < A[i] is true, so we can use a two-dimensional array dp[n][n] to represent A[i] and A[j] and A[i - j] The maximum length of the Fibonacci subsequence formed. In order to find whether A[i - j] is in the original array, we can use a hash table pos to save the subscript of A[i - j] in the original array. After obtaining the subscript k = pos[A[i] - A[j]], we use the state transition equation dp[i][j] = dp[j][k] + 1 to update the longest length.

Note that when judging whether the subscript exists in the hash table, pos.find(A[i] - A[j]) == pos.end() must be used instead of pos[A[i] - A[j]] to obtain it directly. In this way, although the result 0 can still be obtained if A[i] - A[j] does not exist in the hash table, the efficiency is very low and will cause TLE.
```c++
class Solution {
public:
    int lenLongestFibSubseq(vector<int> &A) {
        int n = A.size(), res = 0;
        vector<vector<int>> dp(n, vector<int>(n, 0));
        unordered_map<int, int> pos;
        for (int i = 0; i < n; ++i) {
            pos[A[i]] = i;
            for (int j = 0; j < i; ++j) {
                auto k = pos.find(A[i] - A[j]) == pos.end() ? -1 : pos[A[i] - A[j]];
                dp[i][j] = A[i] - A[j] < A[j] && k != -1 ? dp[j][k] + 1 : 2;
                res = max(res, dp[i][j]);
            }
        }
        return res < 3 ? 0 : res;
    }
};
```

## Original references

- [Reference 1](https://leetcode-cn.com/problemset/all/?search=%E4%B8%91%E6%95%B0)
