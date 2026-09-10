---
title: "Code Jam 2019 Round 1C"
date: 2019-05-07T22:02:15+10:00
draft: false
categories: ["Code Jam"]
description: "A translated technical note on Code Jam 2019 Round 1C, preserving the examples and context of the original article."
---
# Code Jam 2019 Round 1C

> Originally published in Chinese on 2019-05-07; this English edition preserves the original scope and technical context.

## [Robot Programming Strategy (10pts, 18pts)](https://codingcompetitions.withgoogle.com/codejam/round/00000000000516b9/0000000000134c90)

Knowing the order of everyone's Rock, Paper, Scissors moves, you can compete with everyone at the same time in each round to find a winning strategy.

### Solution: Eliminiating

Each round traverses the moves of everyone in the current round. If there are three situations (R, P, S) at the same time, there is no winning strategy, and IMPOSSIBLE is directly output; otherwise, the winning or tied strategy is returned.

There is no need to consider the subsequent moves of the defeated opponents, so use a defeated array to save the defeated opponents so that they can be skipped directly. Because the current round may exceed the length of the opponent's move sequence, i % size must be used to obtain the opponent's current move.

- Time complexity: O(A^2)
- Space complexity: O(A)
```C++
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

char Decide(const char &R, const char &P, const char &S) {
    if (R && P && S)
        return 'X';
    if (R && P)
        return 'P';
    if (R && S)
        return 'R';
    if (P && S)
        return 'S';
    if (R)
        return 'P';
    if (P)
        return 'S';
    return 'R';
}

bool Defeate(const char &current, const char &opponent) {
    return (current == 'R' && opponent == 'S') || (current == 'S' && opponent == 'P') ||
           (current == 'P' && opponent == 'R');
}

void solve(const int &t) {
    int A;
    scanf("%d", &A);
    int i = 0;
    vector<string> opponent(A);
    vector<bool> defeated(A, false);
    bool R, P, S;
    string res;
    for (int a = 0; a < A; ++a)
        cin >> opponent[a];
    while (true) {
        int current_opponent = 0;
        R = false, P = false, S = false;
        for (int a = 0; a < A; ++a) {
            if (!defeated[a]) {
                ++current_opponent;
                if (opponent[a][i % opponent[a].size()] == 'R')
                    R = true;
                else if (opponent[a][i % opponent[a].size()] == 'P')
                    P = true;
                else
                    S = true;
            }
        }
        if (current_opponent == 0)
            break;
        char result = Decide(R, P, S);
        if (result == 'X') {
            res = "IMPOSSIBLE";
            break;
        }
        res += result;
        for (int a = 0; a < A; ++a) {
            if (!defeated[a] && Defeate(result, opponent[a][i % opponent[a].size()]))
                defeated[a] = true;
        }
        ++i;
    }
    printf("Case #%d: %s\n", t, res.c_str());
}

int main() {
    int T;
    scanf("%d", &T);
    for (int t = 1; t <= T; ++t)
        solve(t);
    return 0;
}
```

## [Power Arrangers (11pts, 21pts)](https://codingcompetitions.withgoogle.com/codejam/round/00000000000516b9/0000000000134e91)

// TODO

## [Bacterial Tactics (15pts, 25pts)](https://codingcompetitions.withgoogle.com/codejam/round/00000000000516b9/0000000000134cdf)

// TODO

## Original references

- [Reference 1](https://codingcompetitions.withgoogle.com/codejam/round/00000000000516b9)
