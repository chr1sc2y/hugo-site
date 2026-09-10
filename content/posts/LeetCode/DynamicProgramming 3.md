---
title: "LeetCode: Dynamic Programming (3)"
date: 2019-07-01T18:22:45+10:00
draft: false
categories: ["LeetCode"]
description: "A translated technical note on LeetCode: Dynamic Programming (3), preserving the examples and context of the original article."
---
# LeetCode: Dynamic Programming (3)

> Originally published in Chinese on 2019-07-01; this English edition preserves the original scope and technical context.

## Title

### 6. String related

#### [712 Minimum ASCII delete sum for two strings](https://leetcode-cn.com/problems/minimum-ascii-delete-sum-for-two-strings/)

Given two strings, calculate the minimum sum of the ASCII values of the characters that need to be removed to make the two strings identical.

For the characters s1[i] and s2[j] in the two strings, if s1[i] == s2[j], then neither character needs to be deleted, so dp[i][j] = dp[i - 1][j - 1], otherwise at least one should be deleted, taking the minimum of the two, the state transition equation is dp[i][j] = min(dp[i - 1][j] + s1[i], dp[i][j - 1] + s2[j]). The time complexity is O(m * n) and the space complexity is O(m * n).
```c++
class Solution {
public:
    int minimumDeleteSum(string s1, string s2) {
        int n1 = s1.size(), n2 = s2.size();
        vector<vector<int>> dp(n1 + 1, vector<int>(n2 + 1, 0));
        for (int i = 1; i <= n1; ++i)
            dp[i][0] = dp[i - 1][0] + s1[i - 1];
        for (int j = 1; j <= n2; ++j)
            dp[0][j] = dp[0][j - 1] + s2[j - 1];
        for (int i = 0; i < n1; ++i)
            for (int j = 0; j < n2; ++j)
                if (s1[i] == s2[j])
                    dp[i + 1][j + 1] = dp[i][j];
                else
                    dp[i + 1][j + 1] = min(dp[i][j] + s1[i] + s2[j], min(dp[i][j + 1] + s1[i], dp[i + 1][j] + s2[j]));
        return dp[n1][n2];
    }
};
```
#### [5 longest palindromic substring](https://leetcode-cn.com/problems/longest-palindromic-substring/)

Find the longest palindrome substring in a string.

The simplest method is to traverse from one character to the previous/two characters on both sides. You can also follow the bottom-up dynamic programming idea and use a two-dimensional array dp[i][j] to determine whether the two characters s[i] and s[j] are equal, and whether the substring inside them is a palindrome string. The state transition equation is dp[i][j] = s[i] == s[j] && (dp[i + 1][j - 1] || j - i < 3).
```c++
class Solution {
public:
    string longestPalindrome(string s) {
        int n = s.size();
        string res;
        vector<vector<bool>> dp(n, vector<bool>(n, false));
        for (int i = 0; i < n; ++i)
            for (int j = i; j >= 0; --j)
                if (s[i] == s[j] && (i - j < 3 || dp[j + 1][i - 1])) {
                    dp[j][i] = true;
                    if (i - j + 1 > res.size())
                        res = s.substr(j, i - j + 1);
                }
        return res;
    }
};
```
#### [647 palindromic substrings](https://leetcode-cn.com/problems/palindromic-substrings/)

Find the number of palindrome substrings in a string.

Similar to the previous question, use a two-dimensional array dp[i][j] to determine whether the two characters s[i] and s[j] are equal, and whether the substring inside them is a palindrome string. If so, then s.substr(j, i - j + 1) will add 1 to the result. The time complexity is O(n ^ 2) and the space complexity is O(n ^ 2).
```c++
class Solution {
public:
    int countSubstrings(string s) {
        int res = 0, n = s.size();
        vector<vector<bool>> dp(n, vector<bool>(n, false));
        for (int i = 0; i < n; ++i)
            for (int j = i; j >= 0; --j)
                if (s[i] == s[j] && (i - j < 3 || dp[j + 1][i - 1]))
                    dp[j][i] = true, ++res;
        return res;
    }
};
```
#### [516 longest palindromic subsequence](https://leetcode-cn.com/problems/longest-palindromic-subsequence/)

Given a string, find the longest palindrome subsequence.

Only when two characters are equal, it is possible for them to form a palindrome subsequence with the subsequence between them, so you only need to know the length of the longest palindrome subsequence between them. Otherwise, the longest palindrome subsequence between them can only be the length of the maximum palindrome subsequence from the left or right side of one character to another character. The state transition equation is dp[j][i] = s[i] == s[j] ? dp[j + 1][i - 1] + 2: max(dp[j + 1][i], dp[j][i - 1]). The time complexity is O(n ^ 2) and the space complexity is O(n ^ 2).
```c++
class Solution {
public:
    int longestPalindromeSubseq(string s) {
        int n = s.size();
        if (n <= 1)
            return n;
        vector<vector<int>> dp(n, vector<int>(n, 0));
        for (int i = 1; i < n; ++i) {
            dp[i][i] = 1;
            for (int j = i - 1; j >= 0; --j) {
                if (s[i] == s[j])
                    dp[j][i] = dp[j + 1][i - 1] + 2;
                else
                    dp[j][i] = max(dp[j + 1][i], dp[j][i - 1]);
            }
        }
        return dp[0][n - 1];
    }
};
```
### 7. Path and

#### [62 different paths](https://leetcode-cn.com/problems/unique-paths/)

Given a matrix, find the number of ways to go from the upper left corner to the lower right corner.

There is only one way to reach each grid in the first column and row. The rest of the grids can be reached by walking one grid above and to the left. Therefore, there is a state transition equation dp[i][j] = dp[i][j - 1] + dp[i - 1][j]. The time complexity is O(m * n) and the space complexity is O(m * n).
```c++
class Solution {
public:
    int uniquePaths(int m, int n) {
        if (m == 0 || n == 0)
            return 0;
        vector<vector<int>> dp(m, vector<int>(n, 1));
        for (int i = 1; i < m; ++i)
            for (int j = 1; j < n; ++j)
                    dp[i][j] = dp[i][j - 1] + dp[i - 1][j];
        return dp[m - 1][n - 1];
    }
};
```
For each grid, its value is equal to the sum of the number of methods to reach the upper and left grids, which is equivalent to assigning all the values ​​of the previous row to the next row after traversing one row, and adding the number of methods to the left grid when traversing the next row. This can simplify the assignment process to a one-dimensional array and reduce the space complexity to O(min(m, n)).
```c++
class Solution {
public:
    int uniquePaths(int m, int n) {
        if (m == 0 || n == 0)
            return 0;
        vector<int> dp(n, 1);
        for (int i = 1; i < m; ++i)
            for (int j = 1; j < n; ++j)
                dp[j] += dp[j - 1];
        return dp[n - 1];
    }
};
```
#### [63 different paths II](https://leetcode-cn.com/problems/unique-paths-ii/)

Given a matrix with obstacles in some locations, find out how many ways there are to go from the upper left corner to the lower right corner.

Compared with the previous question, obstacles have been added to some positions. First, the first column and the first row must be dealt with. If there is an obstacle in one position, the next position cannot be reached. Then for other grids, if it is an obstacle, it cannot be reached. Otherwise, it is still equal to the sum of its upper and left sides. The time complexity is O(m * n) and the space complexity is O(m * n).
```c++
class Solution {
public:
    int uniquePathsWithObstacles(vector<vector<int>>& grid) {
        int m = grid.size(), n = m != 0 ? grid[0].size() : 0;
        if (m == 0 || n == 0)
            return 0;
        vector<vector<long long>> dp(m, vector<long long>(n, 0));
        dp[0][0] = grid[0][0] ^ 1;
        for (int i = 1; i < m; ++i)
            dp[i][0] = (grid[i][0] ^ 1) & dp[i - 1][0];
        for (int j = 1; j < n; ++j)
            dp[0][j] = (grid[0][j] ^ 1) & dp[0][j - 1];
        for (int i = 1; i < m; ++i)
            for (int j = 1; j < n; ++j)
                dp[i][j] = grid[i][j] ? 0 : dp[i - 1][j] + dp[i][j - 1];
        return dp[m - 1][n - 1];
    }
};
```
#### [64 minimum path sum](https://leetcode-cn.com/problems/minimum-path-sum/)

Given a matrix with weights, find the minimum sum of weights from the upper left corner to the lower right corner.

There is only one way to reach each cell in the first column and row, so initialize it first. Because each grid can only be reached from above and to the left, there is a state transition equation grid[i][j] += min(grid[i - 1][j], grid[i][j - 1]). The time complexity is O(m * n). It can be operated directly on the given matrix, so the space complexity is O(1).
```c++
class Solution {
public:
    int minPathSum(vector<vector<int>>& grid) {
        int m = grid.size(), n = m != 0 ? grid[0].size() : 0;
        if (m == 0)
            return 0;
        for (int i = 1; i < m; ++i)
            grid[i][0] += grid[i - 1][0];
        for (int j = 1; j < n; ++j)
            grid[0][j] += grid[0][j - 1];
        for (int i = 1; i < m; ++i)
            for (int j = 1; j < n; ++j)
                grid[i][j] += min(grid[i - 1][j], grid[i][j - 1]);
        return grid[m - 1][n - 1];
    }
};
```
#### [120 triangle minimum path sum](https://leetcode-cn.com/problems/triangle/)

Given a weighted triangle, find the minimum path sum from top to bottom. Each step can be moved to the lower left or lower right.

Because each cell can only be reached from its upper left and upper right, there is a state transition equation tri[i][j] += min(tri[i - 1][j], tri[i - 1][j - 1]). The time complexity is O(m * n). It can be operated directly on the given matrix, so the space complexity is O(1).
```c++
class Solution {
public:
    int minimumTotal(vector<vector<int>> &tri) {
        int n = tri.size(), res = INT_MAX;
        if (n <= 1)
            return n == 0 ? 0 : tri[0][0];
        for (int i = 1; i < n; ++i) {
            for (int j = 0; j < tri[i].size(); ++j) {
                if (j == 0)
                    tri[i][j] += tri[i - 1][j];
                else if (j >= tri[i - 1].size())
                    tri[i][j] += tri[i - 1][j - 1];
                else
                    tri[i][j] += min(tri[i - 1][j], tri[i - 1][j - 1]);
                if (i == n - 1)
                    res = min(res, tri[i][j]);
            }
        }
        return res;
    }
};
```
#### [931 Minimum falling path sum](https://leetcode-cn.com/problems/minimum-falling-path-sum/)

Given a weighted square, find the minimum path sum from top to bottom. Each step can be moved to the lower left, lower or lower right.

Each cell can be reached from its upper left, upper and upper right, so there is a state transition equation A[i][j] += min({A[i - 1][j - 1], A[i - 1][j], A[i - 1][j + 1]}).
```c++
class Solution {
public:
    int minFallingPathSum(vector<vector<int>>& A) {
        int m = A.size(), n = m != 0 ? A[0].size() : 0, val = INT_MAX, res = INT_MAX;
        if (m == 0)
            return 0;
        for (int i = 1; i < m; ++i) {
            for (int j = 0; j < n; ++j) {
                val = A[i - 1][j];
                if (j < n - 1)
                    val = min(val, A[i - 1][j + 1]);
                if (j > 0)
                    val = min(val, A[i - 1][j - 1]);
                A[i][j] += val;
            }
        }
        return *min_element(A[m - 1].begin(), A[m - 1].end());
    }
};
```
### 8. Others

#### [650 keyboard with only two keys](https://leetcode-cn.com/problems/2-keys-keyboard/)

There is a character 'A', which can only be copied and pasted. Find the minimum number of operations to obtain n 'A's.

m 'A's can only be obtained through the paste operation. Find the minimum number of times that m can be obtained through the copy-paste operation among all the numbers that can be divided into m.
```c++
class Solution {
public:
    int minSteps(int n) {
        vector<int> dp(n + 1, n);
        dp[1] = 0;
        for (int i = 1; i <= n; ++i) {
            int res = dp[i] + 1;
            for (int j = i; j <= n; j += i) {
                dp[j] = min(dp[j], res);
                ++res;
            }
        }
        return dp[n];
    }
};
```
#### [651 4-key keyboard](https://leetcode-cn.com/problems/4-keys-keyboard/submissions/)

There are four keys on a keyboard: enter 'A', select all, copy, and paste. You can press the keyboard N times to find the maximum number of 'A's that can be displayed.

Because N is the last operation, there are only two operations: input 'A' and paste. You only need to find the best solution that can be obtained by inputting 'A' at each step based on the previous step, and selecting, copying, and pasting based on each of the previous three steps.
```c++
class Solution {
public:
    int maxA(int N) {
        vector<int> dp(N + 1, 0);
        for (int i = 1; i <= N; ++i) {
            dp[i] = dp[i - 1] + 1;
            for (int j = i - 1; j >= 2; --j)
                dp[i] = max(dp[i], dp[j - 2] * (i - j + 1));
        }
        return dp[N];
    }
};
```

## Original references

- [Reference 1](https://leetcode-cn.com/problemset/all/?search=%E4%B8%91%E6%95%B0)
