---
title: "Code Jam 2019 Round 1A"
date: 2019-04-13T15:28:11+10:00
draft: false
categories: ["Code Jam"]
description: "A translated technical note on Code Jam 2019 Round 1A, preserving the examples and context of the original article."
---
# Code Jam 2019 Round 1A

> Originally published in Chinese on 2019-04-13; this English edition preserves the original scope and technical context.

## [Pylons (8pts, 23pts)](https://codingcompetitions.withgoogle.com/codejam/round/0000000000051635/0000000000104e03)

When moving in an m*n grid, the position after each move cannot be on the same row/column/diagonal as the previous position.

### Solution: BackTracking

It is similar to the Eight Queens problem, but each restriction is only related to the previous position and can be solved by backtracking.

- Time complexity: O(m^2 * n^2)
- Space complexity: O(m * n)
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

int m, n;

bool BackTracking(int t, int i, int j, vector<vector<bool>> &visited, vector<vector<int>> &res) {
    visited[i][j] = true;
    res[t] = {i, j};
    if (t + 1 == m * n)
        return true;
    for (int x = 0; x < m; ++x) {
        for (int y = 0; y < n; ++y) {
            int r = (x + i) % m, c = (y + j) % n;
            if (!visited[r][c] && r != i && c != j && r + c != i + j && r - c != i - j &&
                BackTracking(t + 1, r, c, visited, res))
                return true;
        }
    }
    visited[i][j] = false;
    return false;
}

int main() {
    int total_test_case_number;
    cin >> total_test_case_number;
    for (int case_number = 1; case_number <= total_test_case_number; ++case_number) {
        bool rev = false;
        cin >> m >> n;
        if (m > n) {
            rev = true;
            swap(n, m);
        }
        vector<vector<bool>> visited(m, vector<bool>(n, false));
        vector<vector<int>> res(m * n, vector<int>());
        printf("Case #%d: ", case_number);
        if (BackTracking(0, 0, 0, visited, res)) {
            printf("POSSIBLE\n");
            for (int i = 0; i < m * n; ++i)
                if (!rev)
                    printf("%d %d\n", res[i][0] + 1, res[i][1] + 1);
                else
                    printf("%d %d\n", res[i][1] + 1, res[i][0] + 1);
        } else
            printf("IMPOSSIBLE\n");
    }

    return 0;
}
```
## [Alien Rhyme (10pts, 27pts)](https://codingcompetitions.withgoogle.com/codejam/round/0000000000051635/0000000000104e05)

Find a pair of words with the same suffix. The length of the suffix can be defined by yourself. The suffixes of other words cannot be the same as this pair, so that there are the most pairs of words.

### Solution: Suffix

Flip each word first (if you don't flip it, take the substr from the middle to the end. If you flip it, you only need to take the first m letters). Starting from the length of the longest word in descending order, take the suffix of each word. If two words have the same suffix, remove the two words, and the result is +2. Because the suffixes are taken sequentially starting from the longest word length, it can ensure that words with the same suffix are not missed.

- Time complexity: O(N * m)
    - m: longest word length
    - N: number of words
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

#include "Print.h"

using namespace std;


int main() {
    int total_test_case_number;
    cin >> total_test_case_number;
    for (int case_number = 1; case_number <= total_test_case_number; ++case_number) {
        int N, sum = 0, max_len = 0;
        cin >> N;
        vector<string> words(N);
        for (int m = 0; m < N; ++m) {
            cin >> words[m];
            max_len = max(max_len, static_cast<int>(words[m].size()));
            reverse(words[m].begin(), words[m].end());
        }
        unordered_map<string, int> status;
        vector<bool> visited(N, false);
        int total = 0;
        for (int len = max_len; len > 0; --len) {
            for (int m = 0; m < N; ++m) {
                if (visited[m] || words[m].size() < len)
                    continue;
                string str = words[m].substr(0, len);
                if (status.find(str) != status.end()) {
                    if (status[str] != -1) {
                        sum += 2;
                        visited[status[str]] = true, visited[m] = true;
                        status[str] = -1;
                    }
                } else
                    status[str] = m;
            }
        }
        printf("Case #%d: %d\n", case_number, sum);
    }

    return 0;
}
```

## [Golf Gophers (11pts, 21pts)](https://codingcompetitions.withgoogle.com/codejam/round/0000000000051635/0000000000104f1a)

// TODO

## Original references

- [Reference 1](https://codingcompetitions.withgoogle.com/codejam/round/0000000000051635)
