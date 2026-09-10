---
title: "Kick Start 2019 Round B"
date: 2019-04-21T14:41:22+10:00
draft: false
categories: ["Kick Start"]
description: "A translated technical note on Kick Start 2019 Round B, preserving the examples and context of the original article."
---
# Kick Start 2019 Round B

> Originally published in Chinese on 2019-04-21; this English edition preserves the original scope and technical context.

## [Building Palindromes (5pts, 12pts)](https://codingcompetitions.withgoogle.com/kickstart/round/0000000000050eda/0000000000119866)

Determine whether the substring in the given interval is a palindrome string.

### Solution: Prefix Sum

To determine whether a string is a palindrome, you only need to determine whether the number of odd-numbered characters in the string is less than or equal to 1. However, if you traverse the given interval every time, it will definitely time out, so we need to preprocess the given original string and calculate the prefix sum of each position (the total number of characters from the subscript 0 to the subscript i - 1 position). In this way, the time complexity of querying is only O(1).

- Time complexity: O(N)
- Space complexity: O(N)
```C++
// C++
#include <iostream>
#include <cmath>
#include <cstdio>
#include <math.h>
#include <limits>
#include <algorithm>
#include <vector>
#include <stack>
#include <queue>
#include <string>
#include <map>
#include <set>
#include <unordered_map>
#include <unordered_set>

using namespace std;


int main() {
    int total_test_case_number;
    cin >> total_test_case_number;
    for (int case_number = 1; case_number <= total_test_case_number; ++case_number) {
        int N, Q, m, n, total = 0;
        cin >> N >> Q;
        string words;
        cin >> words;
        vector<vector<int>> odds(N + 1, vector<int>(26));
        for (int i = 1; i <= N; ++i) {
            for (int j = 0; j < 26; ++j)
                odds[i][j] = odds[i - 1][j];
            ++odds[i][words[i - 1] - 'A'];
        }
        for (int q = 0; q < Q; ++q) {
            cin >> m >> n;
            int odd = 0;
            for (int j = 0; j < 26; ++j)
                odd += ((odds[n][j] - odds[m - 1][j]) % 2 != 0);
            total += odd <= 1;
        }
        printf("Case #%d: %d\n", case_number, total);
    }

    return 0;
}
```
## [Energy Stones (17pts, 24pts)](https://codingcompetitions.withgoogle.com/kickstart/round/0000000000050eda/00000000001198c3)

Eat all items before they are consumed so that the most energy can be obtained, similar to the [knapsack problem](https://zh.wikipedia.org/wiki/%E8%83%8C%E5%8C%85%E9%97%AE%E9%A2%98).

### Solution 1: Dynamic Programming (Visible Test Set)

For the Visible Test Set, the seconds consumed to eat each item are the same, so it can be simplified to a simple knapsack problem. In order to maximize the energy gained (minimize the total loss of all items), items with the highest loss should be eaten first, so they are sorted by loss.

The initial state is dp[index][time] = 0, index = 0, time = 0. The state transition equation is

1. If the current item still has energy remaining, eat the current item.
```
    if (stones[index].energy > stones[index].lost * time)
        res = max(res, stones[index].energy - stones[index].lost * time + DP(index + 1, time + stones[index].seconds));
```
2. Don’t eat the current item
```
    res = max(res, DP(index + 1, time));
```
- Time complexity: O(N * (S * N))
- Space complexity: O(N * (S * N))
```C++
//C++
#include <iostream>
#include <cmath>
#include <cstdio>
#include <math.h>
#include <limits>
#include <algorithm>
#include <vector>
#include <stack>
#include <queue>
#include <string>
#include <map>
#include <set>
#include <unordered_map>
#include <unordered_set>

using namespace std;

struct Stone {
    int seconds;
    int energy;
    int lost;
};
vector<Stone> stones;
vector<vector<int>> dp;
int N;

int DP(int index, int time) {
    if (index >= N)
        return 0;
    int &res = dp[index][time];
    if (res != -1)
        return res;
    if (stones[index].energy > stones[index].lost * time)
        res = max(res, stones[index].energy - stones[index].lost * time + DP(index + 1, time + stones[index].seconds));
    res = max(res, DP(index + 1, time));
    return res;
}

int solve(const int &T) {
    cin >> N;
    stones = vector<Stone>(N);
    for (int i = 0; i < N; ++i)
        cin >> stones[i].seconds >> stones[i].energy >> stones[i].lost;
    dp = vector<vector<int>>(N, vector<int>(stones[0].seconds * N, -1));
    sort(stones.begin(), stones.end(), [](const Stone &s1, const Stone &s2) {
        return s1.lost > s2.lost;
    });
    return DP(0, 0);
}

int main() {
    int T;
    cin >> T;
    for (int t = 1; t <= T; ++t)
        printf("Case #%d: %d\n", t, solve(t));
    return 0;
}
```
### Solution 2: Dynamic Programming (Hidden Test Set)

For Hidden Test Set, each item consumes different time and cannot be sorted directly using lost. For every two items s1 and s2, as long as s1.lost * s2.seconds > s2.lost * s1.seconds is satisfied, a smaller total loss can be guaranteed. At the same time, the size of the dp array needs to be calculated according to the time consumption of each item.

- Time complexity: O(N * (S * N))
- Space complexity: O(N * (S * N))
```C++
// C++
#include <iostream>
#include <cmath>
#include <cstdio>
#include <math.h>
#include <limits>
#include <algorithm>
#include <vector>
#include <stack>
#include <queue>
#include <string>
#include <map>
#include <set>
#include <unordered_map>
#include <unordered_set>

using namespace std;

struct Stone {
    int seconds;
    int energy;
    int lost;
};
vector<Stone> stones;
vector<vector<int>> dp;
int N;

void solve(const int &T) {
    scanf("%d", &N);
    stones = vector<Stone>(N);
    int total_time = 0;
    for (int i = 0; i < N; ++i) {
        scanf("%d %d %d", &stones[i].seconds, &stones[i].energy,&stones[i].lost);
        total_time += stones[i].seconds;
    }
    sort(stones.begin(), stones.end(), [](const Stone &s1, const Stone &s2) {
        return s1.lost * s2.seconds > s2.lost * s1.seconds;
    });
    int res = 0;
    dp = vector<vector<int>>(N + 1, vector<int>(total_time + 1, -1));
    for (int i = 1; i <= N; ++i) {
        dp[i - 1][0] = 0;
        for (int j = 0; j < stones[i - 1].seconds; ++j)
            dp[i][j] = dp[i - 1][j];
        for (int j = stones[i - 1].seconds; j <  dp[i - 1].size(); ++j) {
            dp[i][j] = max(dp[i - 1][j], dp[i - 1][j - stones[i - 1].seconds] + stones[i - 1].energy - (j - stones[i - 1].seconds) * stones[i - 1].lost);
            res = max(res, dp[i][j]);
        }
    }
    printf("Case #%d: %d\n", T, res);
}

int main() {
    int T;
    scanf("%d", &T);
    for (int t = 1; t <= T; ++t)
        solve(t);
    return 0;
}
```
## [Diverse Subarray (14pts, 28pts)](https://codingcompetitions.withgoogle.com/kickstart/round/0000000000050eda/00000000001198c1)

Given a maximum value S, select a continuous subarray such that the number of identical values in the subarray does not exceed the total number of S.

### Solution 1: Brute Force (Visible Test Set)

The array length of Visible Test Set is N <= 1000. It only requires two levels of loops to do the exhaustive calculation and calculate the total number that meets the requirements each time.

- Time complexity: O(N^2)
- Space complexity: O(N)
```C++
// C++
#include <iostream>
#include <cmath>
#include <cstdio>
#include <math.h>
#include <limits>
#include <algorithm>
#include <vector>
#include <stack>
#include <queue>
#include <string>
#include <map>
#include <set>
#include <unordered_map>
#include <unordered_set>

using namespace std;


int main() {
    int total_test_case_number;
    cin >> total_test_case_number;
    for (int case_number = 1; case_number <= total_test_case_number; ++case_number) {
        int N, S, res = 0;
        cin >> N >> S;
        vector<int> trinkets(N);
        for (int i = 0; i < N; ++i)
            cin >> trinkets[i];
        for (int j = 0; j < N; ++j) {
            unordered_map<int, int> types;
            for (int i = j; i >= 0; --i) {
                ++types[trinkets[i]];
                int total = 0;
                for (auto t:types) {
                    if (t.second <= S)
                        total += t.second;
                }
                res = max(res, total);
            }
        }

        printf("Case #%d: %d\n", case_number, res);
    }

    return 0;
}
```

### Solution 2: Segment Tree (Hidden Test Set)

// TODO

## Original references

- [Reference 1](https://codingcompetitions.withgoogle.com/kickstart/round/0000000000050eda)
