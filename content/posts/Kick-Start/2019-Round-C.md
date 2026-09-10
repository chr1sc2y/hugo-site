---
title: "Kick Start 2019 Round C"
date: 2019-05-26T23:29:45+10:00
draft: false
categories: ["Kick Start"]
description: "A translated technical note on Kick Start 2019 Round C, preserving the examples and context of the original article."
---
# Kick Start 2019 Round C

> Originally published in Chinese on 2019-05-26; this English edition preserves the original scope and technical context.

## [Wiggle Walk (6pts, 12pts)](https://codingcompetitions.withgoogle.com/kickstart/round/0000000000050ff2/0000000000150aac)

When moving in an R * C matrix, you can directly skip the grid you have already walked through. The data is guaranteed not to move beyond the given matrix.

### Solution: Simulation

Use a visited array to record the grids that have been visited. If you encounter a grid that you have visited, you will directly skip it and traverse it later. Logically speaking, the time complexity of this method cannot pass the Hidden Test Set, but I don't know why it passed.

- Time complexity: O(n^2)
- Space complexity: O(n^2)
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

vector<vector<bool>> visited(50001, vector<bool>(50001, false));

void Forward(char &p, int &r, int &c) {
    if (p == 'E') {
        while (visited[r][c + 1])
            ++c;
        visited[r][++c] = true;
    } else if (p == 'W') {
        while (visited[r][c - 1])
            --c;
        visited[r][--c] = true;
    } else if (p == 'N') {
        while (visited[r - 1][c])
            --r;
        visited[--r][c] = true;
    } else if (p == 'S') {
        while (visited[r + 1][c])
            ++r;
        visited[++r][c] = true;
    }
}

void solve(const int &t) {
    int n, R, C, r, c;
    string str;
    scanf("%d %d %d %d %d", &n, &R, &C, &r, &c);
    cin >> str;
    visited = vector<vector<bool>>(50001, vector<bool>(50001, false));
    visited[r][c] = true;
    for (int i = 0; i < n; ++i)
        Forward(str[i], r, c);
    printf("Case #%d: %d %d\n", t, r, c);
}

int main() {
    int T;
    scanf("%d", &T);
    for (int t = 1; t <= T; ++t)
        solve(t);
    return 0;
}
```
## [Circuit Board (14pts, 20pts)](https://codingcompetitions.withgoogle.com/kickstart/round/0000000000050ff2/0000000000150aae)

Find the largest submatrix in the matrix whose maximum and minimum values in each row do not exceed K.

### Solution: Dynamic Programming

Define a matrix len[r][c] to store the longest array of grid r, c that meets the conditions on row r. Because the question requirement is to ensure that the maximum and minimum values ​​of each row do not exceed K, and there is no relationship between rows, so for each grid, traverse all the grids in the column, find the farthest position that these grids can reach, take the minimum value, and multiply the farthest position by the current height to get the answer.

- Time complexity: O(RRC)
- Space complexity: O(RC)
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

void solve(const int &t) {
    int R, C, K;
    scanf("%d %d %d", &R, &C, &K);
    vector<vector<int>> len(301, vector<int>(301, 1)), matrix(301, vector<int>(301));
    for (int r = 0; r < R; ++r)
        for (int c = 0; c < C; ++c)
            scanf("%d", &matrix[r][c]);
    for (int r = 0; r < R; ++r) {
        for (int c = 0; c < C; ++c) {
            int i = c - 1, current_max = matrix[r][c], current_min = matrix[r][c];
            while (i >= 0) {
                current_max = max(current_max, matrix[r][i]);
                current_min = min(current_min, matrix[r][i]);
                if (current_max - current_min > K)
                    break;
                --i;
            }
            len[r][c] = c - i;
        }
    }
    int res = 1;
    for (int r = 0; r < R; ++r) {
        for (int c = 0; c < C; ++c) {
            int min_c = len[r][c];
            for (int line = r; line >= 0; --line) {
                min_c = min(min_c, len[line][c]);
                res = max(res, min_c * (r - line + 1));
            }
        }
    }
    printf("Case #%d: %d\n", t, res);
}

int main() {
    int T;
    scanf("%d", &T);
    for (int t = 1; t <= T; ++t)
        solve(t);
    return 0;
}
```

## [Catch Some (18pts, 30pts)](https://codingcompetitions.withgoogle.com/kickstart/round/0000000000050ff2/0000000000150a0d)

// TODO

## Original references

- [Reference 1](https://codingcompetitions.withgoogle.com/kickstart/round/0000000000050ff2)
