---
title: "Kick Start 2019 Round A"
date: 2019-03-26T14:25:36+11:00
draft: false
categories: ["Kick Start"]
description: "A translated technical note on Kick Start 2019 Round A, preserving the examples and context of the original article."
---
# Kick Start 2019 Round A

> Originally published in Chinese on 2019-03-26; this English edition preserves the original scope and technical context.

## [Training (7pts, 13pts)](https://codingcompetitions.withgoogle.com/kickstart/round/0000000000050e01/00000000000698d6)

There are N people in total, select P people from them, and calculate the sum of the differences between the maximum skill rating of these P people and the skill ratings of other people.

$$ \sum_{i}^{j} max(rating) - rating[i] $$

### Solution: Sort + Prefix Sum

First sort the array, then traverse all consecutive subarrays of length P in the ordered array of length N, and calculate the sum of the differences between the maximum value in the subarray and other values.

$$ \sum_{i}^{j - 1} rating[j] - rating[i] $$

If directly traversing a subarray of length P will waste a lot of time, the above formula can be simplified as follows.

$$ \sum_{i}^{j - 1} rating[j] - rating[i] = rating[j] * (j - 1 - i) - \sum_{i}^{j - 1} rating[i] $$

In order to avoid repeated calculation of $$ \sum_{i}^{j - 1} rating[i] $$, you can use an array with a length of N+1 to save the prefix sum of the original array, so that each time you directly calculate prefix[j] - prefix[i], you can get $$ \sum_{i}^{j - 1} rating[i] $$, and the time complexity is O(N).

- Time complexity: O(NlogN)
- Space complexity: O(N)
```C++
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    int c = 1;
    int T;
    std::cin >> T;
    for (int t = 0; t < T; ++t) {
        int N, P;
        int min_hour = 1000000000;
        std::cin >> N >> P;
        std::vector<int> rating(N, 0);
        for (int n = 0; n < N; ++n)
            std::cin >> rating[n];
        std::sort(rating.begin(), rating.end());

        std::vector<int> prefix(N + 1, 0);
        for (int i = 0; i < N; ++i)
            prefix[i + 1] = prefix[i] + rating[i];

        for (int i = P - 1; i < N; ++i) {
            int sum = rating[i] * (P - 1) - (prefix[i] - prefix[i - P + 1]);
            min_hour = std::max(0, std::min(min_hour, sum));
        }

        std::cout << "Case #" + std::to_string(c) + ": " + std::to_string(min_hour) << std::endl;
        ++c;
    }
    return 0;
}
```
## [Parcels (15pts, 20pts)](https://codingcompetitions.withgoogle.com/kickstart/round/0000000000050e01/000000000006987d)

A medium difficulty BFS, DFS, binary search question. You need to first calculate the Manhattan distance in the graph, and then find an optimal position to make the distance between other points and these points the shortest.

### Solution #1: Manhattan Distance

The Manhattan distance can be calculated using the formula $$ |r1 - r2| + |c1 - c2| $$, so the Manhattan distance can be calculated directly using a two-layer loop.

The first step is to calculate the Manhattan distance from all points to each existing delivery office, and obtain the maximum transportation time max_time at this time. The time complexity is O(RC^2);

The second step is to traverse all possible points in sequence, calculate the maximum transportation time curr_max_time in the graph after adding the delivery office at that point, and compare it with max_time to obtain the minimum value. The time complexity is O(RC^2).

- Time complexity: O(RC^2)
- Space complexity: O(RC)
```C++
#include <iostream>
#include <cmath>
#include <queue>
#include <vector>
#include <string>
#include <limits.h>
#include <algorithm>

int main() {
    int num_case = 1;
    int T;
    std::cin >> T;
    for (int t = 0; t < T; ++t) {
        int R, C;
        std::cin >> R >> C;
        std::vector<std::vector<int>> grid(R, std::vector<int>(C, INT_MAX));
        for (int i = 0; i < R; ++i) {
            std::string temp_in;
            std::cin >> temp_in;
            for (int j = 0; j < C; ++j) {
                char &temp = temp_in[j];
                if (temp == '1') {
                    for (int x = 0; x < R; ++x) {
                        for (int y = 0; y < C; ++y) {
                            grid[x][y] = std::min(grid[x][y], abs(i - x) + abs(j - y));
                        }
                    }
                }
            }
        }

        int max_time = 0;
        for (int i = 0; i < R; ++i) {
            for (int j = 0; j < C; ++j) {
                max_time = std::max(max_time, grid[i][j]);
            }
        }

        if (max_time == 0) {
            std::cout << "Case #" + std::to_string(num_case) + ": 0" << std::endl;
            ++num_case;
            continue;
        }

        for (int i = 0; i < R; ++i) {
            for (int j = 0; j < C; ++j) {
                if (grid[i][j] != 0) {
                    int curr_max_time = 0;
                    for (int x = 0; x < R; ++x) {
                        for (int y = 0; y < C; ++y) {
                            curr_max_time = std::max(curr_max_time, std::min(grid[x][y], abs(i - x) + abs(j - y)));
                        }
                    }
                    max_time = std::min(max_time, curr_max_time);
                }
            }
        }

        std::cout << "Case #" + std::to_string(num_case) + ": " + std::to_string(max_time) << std::endl;
        ++num_case;
    }
    return 0;
}
```
### Solution #2: Breadth-First-Search

For the first step, you can save all existing delivery offices in a queue, and then use BFS to directly calculate the shortest Manhattan distance to obtain the maximum transportation time max_time at this time. The time complexity is O(RC);

The second step is similar to the first step. BFS can also be used to calculate the maximum transportation time after adding delivery offices at all possible points. The time complexity is O(RC^2).

- Time complexity: O(RC^2)
- Space complexity: O(RC)
```C++
#include <iostream>
#include <cmath>
#include <queue>
#include <vector>
#include <string>
#include <limits.h>
#include <algorithm>

int dir_x[4] = {-1, 0, 1, 0};
int dir_y[4] = {0, -1, 0, 1};

int BFS0(const int &R, const int &C, int x, int y,
         std::vector<std::vector<int>> &grid, std::queue<std::pair<int, int>> &que) {
    int max_time = 0;
    int size = que.size(), degree = 1;
    while (!que.empty()) {
        x = que.front().first, y = que.front().second;
        que.pop();
        max_time = std::max(max_time, grid[x][y]);
        for (int i = 0; i < 4; ++i) {
            int new_x = x + dir_x[i], new_y = y + dir_y[i];
            if (new_x >= 0 && new_x < R && new_y >= 0 && new_y < C && grid[new_x][new_y] == -1) {
                que.push(std::pair<int, int>(new_x, new_y));
                grid[new_x][new_y] = degree;
            }
        }
        --size;
        if (size == 0) {
            size = que.size();
            ++degree;
        }
    }
    return max_time;
}

int BFS1(const int &R, const int &C, int x, int y,
         std::vector<std::vector<int>> &grid, std::queue<std::pair<int, int>> &que) {
    std::vector<std::vector<bool>> visited(R, std::vector<bool>(C, false));
    visited[x][y] = true;
    int max_time = INT_MIN;
    int size = 1, degree = 0;
    while (!que.empty()) {
        x = que.front().first, y = que.front().second;
        que.pop();
        max_time = std::max(max_time, std::min(grid[x][y], degree));
        for (int i = 0; i < 4; ++i) {
            int new_x = x + dir_x[i], new_y = y + dir_y[i];
            if (new_x >= 0 && new_x < R && new_y >= 0 && new_y < C && !visited[new_x][new_y]) {
                que.push(std::pair<int, int>(new_x, new_y));
                visited[new_x][new_y] = true;
            }
        }
        --size;
        if (size == 0) {
            size = que.size();
            ++degree;
        }
    }
    return max_time;
}

int main() {
    int num_case = 1;
    int T;
    std::cin >> T;
    for (int t = 0; t < T; ++t) {
        int R, C;
        std::cin >> R >> C;
        std::vector<std::vector<int>> grid(R, std::vector<int>(C, -1));
        std::queue<std::pair<int, int>> que;
        for (int i = 0; i < R; ++i) {
            std::string temp_in;
            std::cin >> temp_in;
            for (int j = 0; j < C; ++j) {
                char &temp = temp_in[j];
                if (temp == '1') {
                    que.push(std::pair<int, int>(i, j));
                    grid[i][j] = 0;
                }
            }
        }
        int size = que.size();
        if (size == R * C) {
            std::cout << "Case #" + std::to_string(num_case) + ": 0" << std::endl;
            ++num_case;
            continue;
        }

        // BFS for existing delivery offices
        int max_time = BFS0(R, C, 0, 0, grid, que);

        // BFS for all possible positions
        for (int i = 0; i < R; ++i) {
            for (int j = 0; j < C; ++j) {
                if (grid[i][j] != 0) {
                    que = std::queue<std::pair<int, int>>();
                    que.push(std::pair<int, int>(i, j));
                    int curr_max_time = BFS1(R, C, i, j, grid, que);
                    max_time = std::min(max_time, curr_max_time);
                }
            }
        }

        std::cout << "Case #" + std::to_string(num_case) + ": " + std::to_string(max_time) << std::endl;
        ++num_case;
    }
    return 0;
}
```
### Solution #3: Binary Search

The previous method searches every possible point, and the time complexity reaches the square level, so it cannot pass the Hidden Test Set. Because the question requires the minimum value that meets the requirements, it is natural to think of using the dichotomy method to find the lower bound.

For a given mid value, if a new delivery office can be added so that the maximum transportation time is less than or equal to mid, then there may be values ​​less than mid (1 ... k-1) that make the conclusion true; if the given mid value does not make the conclusion true, then all values ​​greater than or equal to mid (k ... INT_MAX) cannot make the conclusion true.

In order to confirm that the shortest distance from the point in the figure to the delivery office is less than mid, you can rotate the figure 45 degrees and use i + j and i - j as the values ​​of the lower left and upper right diagonals respectively to calculate whether the transportation can be completed when mid is used as the maximum transportation time.

$$ distance((x1, y1), (x2, y2))= \max (abs(x1 + y1 - (x2 + y2)), abs(x1 - y1 - (x2 - y2))) $$

- Time complexity: O(RClog(R+C))
- Space complexity: O(RC)
```C++
#include <iostream>
#include <cmath>
#include <queue>
#include <vector>
#include <string>
#include <limits.h>
#include <algorithm>

int dir_x[4] = {-1, 0, 1, 0};
int dir_y[4] = {0, -1, 0, 1};
int R, C;
std::vector<std::vector<int>> grid;

int BFS(int x, int y, std::queue<std::pair<int, int>> &que) {
    int max_time = 0;
    int size = que.size(), time = 1;
    while (!que.empty()) {
        x = que.front().first, y = que.front().second;
        que.pop();
        max_time = std::max(max_time, grid[x][y]);
        for (int i = 0; i < 4; ++i) {
            int new_x = x + dir_x[i], new_y = y + dir_y[i];
            if (new_x >= 0 && new_x < R && new_y >= 0 && new_y < C && grid[new_x][new_y] == -1) {
                que.push(std::pair<int, int>(new_x, new_y));
                grid[new_x][new_y] = time;
            }
        }
        --size;
        if (size == 0) {
            size = que.size();
            ++time;
        }
    }
    return max_time;
}

bool CanDeliver(int &max_time) {
    bool deliver = true;
    int left_low = INT_MAX, left_high = INT_MIN, right_low = INT_MAX, right_high = INT_MIN;
    for (int i = 0; i < R; ++i) {
        for (int j = 0; j < C; ++j) {
            if (grid[i][j] > max_time) {
                deliver = false;
                left_low = std::min(left_low, i + j + max_time);
                left_high = std::max(left_high, i + j - max_time);
                right_low = std::min(right_low, i - j + max_time);
                right_high = std::max(right_high, i - j - max_time);
            }
        }
    }
    if (deliver)
        return true;
    for (int i = 0; i < R; ++i) {
        for (int j = 0; j < C; ++j) {
            int left = i + j, right = i - j;
            if (left_high <= left && left <= left_low && right_high <= right && right <= right_low)
                return true;
        }
    }
    return false;
}

int main() {
    int num_case = 1;
    int T;
    std::cin >> T;
    for (int t = 0; t < T; ++t) {
        std::cin >> R >> C;
        grid = std::vector<std::vector<int>>(R, std::vector<int>(C, -1));
        std::queue<std::pair<int, int>> que;
        for (int i = 0; i < R; ++i) {
            std::string temp_in;
            std::cin >> temp_in;
            for (int j = 0; j < C; ++j) {
                char &temp = temp_in[j];
                if (temp == '1') {
                    que.push(std::pair<int, int>(i, j));
                    grid[i][j] = 0;
                }
            }
        }
        int size = que.size();
        if (size == R * C) {
            std::cout << "Case #" + std::to_string(num_case) + ": 0" << std::endl;
            ++num_case;
            continue;
        }

        // BFS
        int max_time = BFS(0, 0, que);

        // Binary Search
        int lowest_time = 0, highest_time = INT_MAX;
        while (lowest_time < highest_time) {
            int mid_time = lowest_time + ((highest_time - lowest_time) >> 1);
            if (CanDeliver(mid_time))
                highest_time = mid_time;
            else
                lowest_time = mid_time + 1;
        }

        std::cout << "Case #" + std::to_string(num_case) + ": " + std::to_string(highest_time) << std::endl;
        ++num_case;
    }
    return 0;
}
```

## [Contention (18pts, 27pts)](https://codingcompetitions.withgoogle.com/kickstart/round/0000000000050e01/0000000000069881)

// TODO

## Original references

- [Reference 1](https://codingcompetitions.withgoogle.com/kickstart/round/0000000000050e01)
