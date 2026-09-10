---
title: "LeetCode: Dynamic Programming (1)"
date: 2019-06-26T18:08:10+10:00
draft: false
categories: ["LeetCode"]
description: "A translated technical note on LeetCode: Dynamic Programming (1), preserving the examples and context of the original article."
---
# LeetCode: Dynamic Programming (1)

> Originally published in Chinese on 2019-06-26; this English edition preserves the original scope and technical context.

## Title

### 1. Number related

#### [263 ugly number](https://leetcode-cn.com/problems/ugly-number/)

Determine whether a number num is an ugly number.

The general method is to find the first number greater than or equal to num from bottom to top to determine whether num is an ugly number. But this question has already given the number num, and the result can be obtained directly through modular operation.
```c++
class Solution {
public:
    bool isUgly(int num) {
        if (num < 1)
            return false;
        while (num % 2 == 0)
            num /= 2;
        while (num % 3 == 0)
            num /= 3;
        while (num % 5 == 0)
            num /= 5;
        return num == 1;
    }
};
```
#### [264 Ugly Number II](https://leetcode-cn.com/problems/ugly-number-ii/comments/)

Find the nth ugly number.

Use an array ugly to save the first m ugly numbers, multiply the ugly numbers corresponding to their current coefficients by three prime factors 2, 3, and 5 to get a new ugly number. The smallest one is the m + 1th ugly number. The time complexity is O(m * n), where m is the number of prime factors and n is the nth ugly number to be found.
```c++
class Solution {
public:
    int nthUglyNumber(int n) {
        vector<int> ugly(n, 1);
        int base_2 = 0, base_3 = 0, base_5 = 0;
        int m = INT_MAX;
        for (int i = 1; i < n; ++i) {
            ugly[i] = min({2 * ugly[base_2], 3 * ugly[base_3], 5 * ugly[base_5]});
            if (2 * ugly[base_2] == ugly[i])
                ++base_2;
            if (3 * ugly[base_3] == ugly[i])
                ++base_3;
            if (5 * ugly[base_5] == ugly[i])
                ++base_5;
            cout << ugly[i] << endl;
        }
        return ugly[n - 1];
    }
};
```
#### [313 Super Ugly Number](https://leetcode-cn.com/problems/super-ugly-number/)

Given an array of prime factors primes, find the nth ugly number.

It is exactly the same as the previous question, except that the original three prime factors 2, 3, and 5 are replaced by an array. The time complexity is O(n * m), where n is the nth ugly number and m is the length of the array.
```c++
class Solution {
public:
    int nthSuperUglyNumber(int N, vector<int>& primes) {
        int n = primes.size(), m = INT_MAX;;
        vector<int> ugly(N, 1), base(n, 0);
        for (int i = 1; i < N; ++i) {
            m = INT_MAX;
            for (int j = 0; j < n; ++j)
                m = min(m, primes[j] * ugly[base[j]]);
            ugly[i] = m;
            for (int j = 0; j < n; ++j)
                if (primes[j] * ugly[base[j]] == m)
                    ++base[j];
        }
        return ugly[N - 1];
    }
};
```
#### [279 perfect square numbers](https://leetcode-cn.com/problems/perfect-squares/)

Given a number n that can be expressed as m perfect squares, find the smallest m.

n can only be obtained by adding one to the optimal value of a number that is 1, 4, 9, etc. smaller than n, so use an array dp[n] to save the smallest number k of numbers less than or equal to n expressed as the sum of k perfect square numbers. According to the state transition equation dp[n] = min({dp[n], dp[n - 1], dp[n - 4], dp[n - 9], ...}) + 1 calculates the result. The time complexity is O(n * w), w is the number of perfect square numbers smaller than n, and the space complexity is O(n).
```c++
class Solution {
public:
    int numSquares(int n) {
        vector<int> dp(n + 1, INT_MAX);
        dp[0] = 0, dp[1] = 1;
        for (int i = 2; i <= n; ++i)
            for (int j = 1; i - j * j >= 0; ++j)
                dp[i] = min(dp[i], dp[i - j * j] + 1);
        return dp[n];
    }
};
```
#### [343 integer break](https://leetcode-cn.com/problems/integer-break/)

Given a number n, split it into the sum of at least two numbers and find the maximum product of these integers.

n can be split into the sum of 2 numbers, and these two numbers can be split into the sum of several numbers. Therefore, you only need to know the maximum product of the two numbers when it is split into two numbers, and calculate from bottom to top the maximum product when numbers less than or equal to n are split. The state transition equation is dp[i] = max(dp[i], dp[j] * dp[i - j]). The time complexity is O(n ^ 2) and the space complexity is O(n).
```c++
class Solution {
public:
    int integerBreak(int n) {
        vector<int> dp(n + 1, 0);
        if (n < 3)  return 1;
        if (n == 3) return 2;
        dp[2] = 2, dp[3] = 3;
        for (int i = 4; i <= n; ++i)
            for (int j = 1, k = i - j; j <= k; ++j, --k)
                dp[i] = max(dp[i], dp[j] * dp[k]);
        return dp[n];
    }
};
```
#### [1155 N ways to roll dice](https://leetcode-cn.com/problems/number-of-dice-rolls-with-target-sum/)

For each dice, it can have f ways to throw based on the previous ones. Its subsequent state is dp[i + 1][j + k], i is the number of dices that have been thrown, k is the f different ways it has been thrown, and j is the number of throws that sum to j so far. Bottom-up dynamic programming can be used.
```c++
class Solution {
public:
    int numRollsToTarget(int d, int f, int target) {
        int dp[d + 1][target + f + 1];
        memset(dp, 0, sizeof(dp));
        dp[0][0] = 1;
        for (int i = 0; i < d; ++i)
            for (int j = 0; j < target; ++j)
                if (dp[i][j])
                    for (int k = 1; k <= f; ++k)
                        if (j + k <= target)
                            dp[i + 1][j + k] = (dp[i + 1][j + k] + dp[i][j]) % 1000000007;
        return dp[d][target];
    }
};
```
### 2. Buy and sell stocks

#### [121 The best time to buy and sell stocks](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock/)

Given an array of stocks, only one transaction can be made to find the maximum profit.

Maximize the difference between the current value and the previous minimum value. The time complexity is O(n).
```c++
class Solution {
public:
    int maxProfit(vector<int>& prices) {
        int min_val = INT_MAX, res = 0;
        for (auto &p:prices) {
            min_val = min(min_val, p);
            res = max(res, p - min_val);
        }
        return res;
    }
};
```
#### [122 The best time to buy and sell stocks II](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-ii/)

Given an array of stocks, without limiting the number of transactions, find the maximum profit.

Each strictly increasing range is an opportunity for trading, so just add all the differences within the strictly increasing range. The time complexity is O(n).
```c++
class Solution {
public:
    int maxProfit(vector<int> &prices) {
        int res = 0;
        for (int i = 1; i < prices.size(); ++i)
            res += max(prices[i] - prices[i - 1], 0);
        return res;
    }
};
```
#### [123 The best time to buy and sell stocks III](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-iii/)

Given an array of stocks, only two transactions can be made to find the maximum profit.

Because there are two trades to be made, two values ​​need to be maximized: one is the maximum return up to day i, and one is the maximum return after day i. The first method is to traverse twice. The first time is to calculate the maximum profit up to the i-th day. The second reverse traversal is to calculate the maximum profit after the i-th day. The method is the same as the first question. Note that the second buying operation must be after the first selling operation and cannot occur on the same day. The time complexity is O(n).
```c++
class Solution {
public:
    int maxProfit(vector<int> &prices) {
        int n = prices.size(), pre_min = INT_MAX, post_max = 0, res = 0;
        vector<int> pre(n, 0), post(n, 0);
        for (int i = 0; i < n - 1; ++i) {
            pre_min = min(pre_min, prices[i]);
            pre[i] = max(pre[max(0, i - 1)], prices[i] - pre_min);
        }
        for (int i = n - 1; i >= 0; --i) {
            post_max = max(post_max, prices[i]);
            post[i] = max(post[min(n - 1, i + 1)], post_max - prices[i]);
        }
        for (int i = 0; i < n; ++i)
            res = max(res, pre[i] + post[i]);
        return res;
    }
};
```
The second method is based on the fact that there are only four possible operations per day: the first buy, the first sell res1, the second buy, and the second sell res2. The first purchase needs to maximize the minimum cost of buying the stock before, the first sale needs to maximize the difference between the stock price up to day i and the first purchase, the second purchase needs to maximize the purchase of the stock on day i minus the profit from the first sale, and finally the second sale needs to maximize the difference between the stock price up to day i and the second purchase. Finally, the optimal value of the second sale is obtained. The time complexity is O(n).
```c++
class Solution {
public:
    int maxProfit(vector<int> &prices) {
        int buy1 = INT_MIN, sell1 = 0, buy2 = INT_MIN, sell2 = 0;
        for (auto &p:prices) {
            buy1 = max(buy1, -p);
            sell1 = max(sell1, p + buy1);
            buy2 = max(buy2, sell1 - p);
            sell2 = max(sell2, p + buy2);
            cout << buy1 << ' ' << sell1 << ' ' << buy2 << ' ' << sell2 << ' ' << endl;
        }
        return sell2;
    }
};
```
#### [309 Best time to buy and sell stocks with cooldown period](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-with-cooldown/)

Given an array of stocks, with no limit on the number of transactions, and one day between selling and buying, find the maximum profit.

Similar to the second method of the previous question, we can use two arrays buy and sell to represent buying and selling operations respectively. For buying operations, we need to maximize the difference between the optimal value sold two days ago and buying today. For selling operations, we need to maximize the difference between the current price and the optimal value bought the day before. The time complexity is O(n).
```c++
class Solution {
public:
    int maxProfit(vector<int>& prices) {
        int n = prices.size();
        if (n < 2)
            return 0;
        vector<int> buy(n, 0), sell(n, 0);
        buy[0] = -prices[0], sell[0] = 0;
        for (int i = 1; i < n; ++i) {
            buy[i] = max(buy[i - 1], sell[max(0, i - 2)] - prices[i]);
            sell[i] = max(sell[i - 1], buy[i - 1] + prices[i]);
        }
        return sell[n - 1];
    }
};
```
#### [714 The best time to buy and sell stocks with transaction fees](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-with-transaction-fee/)

Given an array of stocks, there is no limit to the number of transactions, and there is a certain handling fee for each sale. Find the maximum profit.

Similar to the previous question, the difference is that there is no transaction interval, and the handling fee needs to be subtracted every time a sell operation is performed. The time complexity is O(n).
```c++
class Solution {
public:
    int maxProfit(vector<int>& prices, int fee) {
        int n = prices.size();
        if (n < 2)
            return 0;
        vector<int> buy(n, 0), sell(n, 0);
        buy[0] = -prices[0];
        for (int i = 1; i < n; ++i) {
            buy[i] = max(buy[i - 1], sell[i - 1] - prices[i]);
            sell[i] = max(sell[i - 1], buy[i - 1] + prices[i] - fee);
        }
        return sell[n - 1];
    }
};
```
#### [188 The best time to buy and sell stocks IV](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-iv/)

Given an array of stocks, up to k transactions can be performed to find the maximum profit.

After completing all the above five questions, this question will be very simple. Compared with the second question, because k in this question is unknown, a loop is used to calculate the optimal values ​​of all possible k transactions. Therefore, a three-dimensional array dp[n][k][2] is used, or divided into two two-dimensional arrays buy[n][k] and sell[n][k] to represent the optimal buy and sell values ​​of k transactions in the first n days. Similarly, buying and selling operations are performed on the basis of the previous selling and buying. Use buy[i][j] = max({buy[i][j - 1], buy[i - 1][j], sell[i - 1][j - 1] - prices[i]}) to represent the optimal buying value of j transactions in the previous i days. The first item will be filled in as buy[i][j - when j <= i / 2 1] to prevent errors caused by null values or default values in subsequent operations. The second item is the result that the current buy operation cannot obtain the optimal value, and the third item is the result that the current buy operation can obtain the optimal value; the corresponding sell operation is sell[i][j] = max({sell[i][j - 1], sell[i - 1][j], buy[i - 1][j] + prices[i]}).

It is worth noting that when k is much greater than twice the length of the array, or k is very large, constructing a two-dimensional array will cause MLE. In this case, you can directly use the idea of ​​​​the second question to solve it. The time complexity is O(n * k) and the space complexity is O(n * k).
```c++
class Solution {
public:
    int maxProfit(int k, vector<int> &prices) {
        int n = prices.size(), res = 0;
        if (n < 2 || k == 0)
            return 0;
        if (k >= n * 2) {
            for (int i = 1; i < n; ++i)
                res += max(0, prices[i] - prices[i - 1]);
            return res;
        }
        vector<vector<int>> buy(n, vector<int>(k, INT_MIN)), sell(n, vector<int>(k, 0));
        buy[0][0] = -prices[0], sell[0][0] = 0;
        for (int i = 1; i < n; ++i) {
            buy[i][0] = max(buy[i - 1][0], -prices[i]);
            sell[i][0] = max(sell[i - 1][0], buy[i - 1][0] + prices[i]);
            for (int j = 1; j < k; ++j) {
                buy[i][j] = max({buy[i][j - 1], buy[i - 1][j], sell[i - 1][j - 1] - prices[i]});
                sell[i][j] = max({sell[i][j - 1], sell[i - 1][j], buy[i - 1][j] + prices[i]});
            }
        }
        return sell[n - 1][k - 1];
    }
};
```

## Original references

- [Reference 1](https://leetcode-cn.com/problemset/all/?search=%E4%B8%91%E6%95%B0)
