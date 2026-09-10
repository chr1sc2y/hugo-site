---
title: "LeetCode: Depth-First Search"
date: 2019-07-27T12:12:25+10:00
draft: false
categories: ["LeetCode"]
description: "A translated technical note on LeetCode: Depth-First Search, preserving the examples and context of the original article."
---
# LeetCode: Depth-First Search

> Originally published in Chinese on 2019-07-27; this English edition preserves the original scope and technical context.

## Title

#### [78 subsets](https://leetcode-cn.com/problems/subsets/)

Typical backtracking to find all possible situations.
```c++
class Solution {
    vector<vector<int>> res;
public:
    vector<vector<int>> subsets(vector<int> &nums) {
        res = vector<vector<int>>(1, vector<int>());
        vector<int> curr;
        DFS(nums, 0, curr);
        return res;
    }

    void DFS(vector<int> &nums, int idx, vector<int> &curr) {
        for (int i = idx; i < nums.size(); ++i) {
            curr.push_back(nums[i]);
            res.push_back(curr);
            DFS(nums, i + 1, curr);
            curr.pop_back();
        }
    }
};
```
#### [733 Image Rendering](https://leetcode-cn.com/problems/flood-fill/)

Start DFS or BFS from the given image[sr][sc], and modify the values of all adjacent points with the same value to newColor. Pay attention to determine whether the given image[sr][sc] is equal to newColor. Otherwise, if the visited array of extra space is not used to record the visited points, an infinite loop stack overflow will occur.
```c++
class Solution {
    int m, n;
    int dir[4][2] = {{0,  1}, {1,  0}, {0,  -1}, {-1, 0}};
    vector<vector<bool>> visited;
public:
    vector<vector<int>> floodFill(vector<vector<int>> &image, int sr, int sc, int newColor) {
        m = image.size(), n = m ? image[0].size() : 0;
        visited = vector<vector<bool>>(m, vector<bool>(n, false));
        if (image[sr][sc] != newColor)
            DFS(image, sr, sc, newColor);
        return image;
    }

    void DFS(vector<vector<int>> &image, int r, int c, int val) {
        int ori = image[r][c];
        image[r][c] = val;
        for (int d = 0; d < 4; ++d) {
            int i = r + dir[d][0];
            int j = c + dir[d][1];
            if (i >= 0 && i < m && j >= 0 && j < n && image[i][j] == ori)
                DFS(image, i, j, val);
        }
    }
};
```
#### [463 Island Perimeter](https://leetcode-cn.com/problems/island-perimeter/)

Perform DFS on the island and calculate the perimeter of the current point based on how many adjacent points there are around it.
```c++
class Solution {
    int x, y, res;
    vector<vector<bool>> visited;
    int dir[4][2] = {{0,  1},
                     {1,  0},
                     {0,  -1},
                     {-1, 0}};
public:
    int islandPerimeter(vector<vector<int>> &grid) {
        x = grid.size(), y = x ? grid[0].size() : 0, res = 0;
        visited = vector<vector<bool>>(x, vector<bool>(y, false));
        if (x == 0)
            return 0;
        for (int i = 0; i < x; ++i)
            for (int j = 0; j < y; ++j)
                if (grid[i][j] == 1) {
                    DFS(grid, i, j);
                    return res;
                }
        return res;
    }

    void DFS(vector<vector<int>> &grid, int i, int j) {
        visited[i][j] = true;
        int edge = 4;
        for (int l = 0; l < 4; ++l) {
            int a = i + dir[l][0];
            int b = j + dir[l][1];
            if (a >= 0 && a < x && b >= 0 && b < y && grid[a][b] == 1) {
                --edge;
                if (!visited[a][b])
                    DFS(grid, a, b);
            }
        }
        res += edge;
    }
};
```
#### [200 Number of islands](https://leetcode-cn.com/problems/number-of-islands/)

Every time DFS is performed, all the nodes are an island, and DFS can complete the entire array.
```c++
class Solution {
    vector<vector<bool>> visited;
    int x, y, res;
    int dir[4][2] = {{0,  1}, {1,  0}, {0,  -1}, {-1, 0}};
public:
    int numIslands(vector<vector<char>> &grid) {
        x = grid.size(), y = x ? grid[0].size() : 0, res = 0;
        visited = vector<vector<bool>>(x, vector<bool>(y, 0));
        if (x == 0)
            return 0;
        for (int i = 0; i < x; ++i)
            for (int j = 0; j < y; ++j)
                if (grid[i][j] == '1' && !visited[i][j]) {
                    ++res;
                    DFS(grid, i, j);
                }
        return res;
    }

    void DFS(vector<vector<char>> &grid, int i, int j) {
        visited[i][j] = true;
        for (int d = 0; d < 4; ++d) {
            int a = i + dir[d][0];
            int b = j + dir[d][1];
            if (a >= 0 && a < x && b >= 0 && b < y && grid[a][b] == '1' && !visited[a][b])
                DFS(grid, a, b);
        }
    }
};
```
#### [Maximum area of 695 islands](https://leetcode-cn.com/problems/max-area-of-island/)

Perform DFS on each island and update the maximum area each time.
```c++
class Solution {
    vector<vector<bool>> visited;
    int x, y, res;
    int dir[4][2] = {{0,  1}, {1,  0}, {0,  -1}, {-1, 0}};
public:
    int maxAreaOfIsland(vector<vector<int>> &grid) {
        x = grid.size(), y = x ? grid[0].size() : 0, res = 0;
        visited = vector<vector<bool>>(x, vector<bool>(y, 0));
        if (x == 0)
            return 0;
        for (int i = 0; i < x; ++i)
            for (int j = 0; j < y; ++j)
                if (grid[i][j] == 1 && !visited[i][j]) {
                    int area = 1;
                    DFS(grid, i, j, area);
                }
        return res;
    }

    void DFS(vector<vector<int>> &grid, int i, int j, int &area) {
        visited[i][j] = true;
        res = max(res, area);
        for (int d = 0; d < 4; ++d) {
            int a = i + dir[d][0];
            int b = j + dir[d][1];
            if (a >= 0 && a < x && b >= 0 && b < y && grid[a][b] == 1 && !visited[a][b]) {
                ++area;
                DFS(grid, a, b, area);
            }
        }
    }
};
```
#### [841 Keys and Rooms](https://leetcode-cn.com/problems/keys-and-rooms/)

DFS each room.
```c++
class Solution {
    vector<bool> visited;
    int m, n;
public:
    bool canVisitAllRooms(vector<vector<int>> &rooms) {
        m = n = rooms.size();
        visited = vector<bool>(n, false);
        return DFS(rooms, 0);
    }

    bool DFS(vector<vector<int>> &rooms, int room_num) {
        --m;
        visited[room_num] = true;
        if (m == 0)
            return true;
        for (auto &r:rooms[room_num])
            if (!visited[r] && DFS(rooms, r))
                return true;
        return false;
    }
};
```
#### [113 Path Sum II](https://leetcode-cn.com/problems/path-sum-ii/)

Perform DFS on the entire tree and make judgments on leaf nodes.
```c++
class Solution {
    vector<vector<int>> res;
public:
    vector<vector<int>> pathSum(TreeNode *root, int sum) {
        res = vector<vector<int>>();
        vector<int> path;
        DFS(root, sum, path);
        return res;
    }

    void DFS(TreeNode *root, int sum, vector<int> &path) {
        if (!root)
            return;
        path.push_back(root->val);
        if (!root->left && !root->right) {
            if (sum - root->val == 0)
                res.push_back(path);
        } else {
            DFS(root->left, sum - root->val, path);
            DFS(root->right, sum - root->val, path);
        }
        path.pop_back();
    }
};
```
#### [130 Surrounded regions](https://leetcode-cn.com/problems/surrounded-regions/)

Perform DFS on all the outermost 'O's and mark them, and finally traverse the entire matrix and change all unmarked 'O's to 'X's.
```c++
class Solution {
    int dir[4][2] = {{0,  1}, {1,  0}, {0,  -1}, {-1, 0}};
    int m, n;
public:
    void solve(vector<vector<char>> &board) {
        m = board.size(), n = m ? board[0].size() : 0;
        for (int i = 0; i < m; ++i) {
            if (board[i][0] == 'O')
                DFS(board, i, 0);
            if (board[i][n - 1] == 'O')
                DFS(board, i, n - 1);
        }
        for (int j = 1; j < n - 1; ++j) {
            if (board[0][j] == 'O')
                DFS(board, 0, j);
            if (board[m - 1][j] == 'O')
                DFS(board, m - 1, j);
        }
        for (auto &bo:board)
            for (auto &b:bo)
                b = (b == 'O' ? 'X' : (b == 'M' ? 'O' : b));
    }

    void DFS(vector<vector<char>> &board, int x, int y) {
        board[x][y] = 'M';
        for (auto &d:dir) {
            int i = x + d[0], j = y + d[1];
            if (i >= 0 && i < m && j >= 0 && j < n && board[i][j] == 'O')
                DFS(board, i, j);
        }
    }
};
```
#### [529 Minesweeper Game](https://leetcode-cn.com/problems/minesweeper/submissions/)

First calculate the number of bombs in the 8 locations around each location. If the number is greater than or equal to 1, then mark it and end the search. If the number is 0, then continue to search the surrounding 8 locations.
```c++
class Solution {
    int m, n;
public:
    vector<vector<char>> updateBoard(vector<vector<char>> &board, vector<int> &click) {
        if (board[click[0]][click[1]] == 'M') {
            board[click[0]][click[1]] = 'X';
            return board;
        }
        m = board.size(), n = m ? board[0].size() : 0;
        DFS(board, click[0], click[1]);
        return board;
    }

    void DFS(vector<vector<char>> &board, int x, int y) {
        int b = 0;
        for (int i = x - 1; i <= x + 1; ++i)
            for (int j = y - 1; j <= y + 1; ++j)
                if (i >= 0 && i < m && j >= 0 && j < n && board[i][j] == 'M')
                    ++b;
        if (b != 0) {
            board[x][y] = static_cast<char>(b + '0');
            return;
        }
        board[x][y] = 'B';
        for (int i = x - 1; i <= x + 1; ++i)
            for (int j = y - 1; j <= y + 1; ++j)
                if (i >= 0 && i < m && j >= 0 && j < n && board[i][j] == 'E')
                    DFS(board, i, j);
    }
};
```
#### [473 Matchsticks to Square](https://leetcode-cn.com/problems/matchsticks-to-square/submissions/)

Because it is required to use all the matches to form a square, we first determine whether all the matches are a multiple of 4 and whether there is a number greater than sum / 4, and then sort the array from large to small. This can use a greedy strategy to reduce the number of searches. Otherwise, backtracking is required, and finally DFS is performed on the entire array.
```c++
class Solution {
    int match, n, sum;
    vector<bool> visited;
public:
    bool makesquare(vector<int> &nums) {
        sort(nums.begin(), nums.end(), [](int &a, int &b) { return a > b; });
        sum = accumulate(nums.begin(), nums.end(), 0), n = nums.size(), match = 4;
        visited = vector<bool>(n, false);
        if (n == 0 || sum % 4 != 0)
            return false;
        for (auto &m:nums)
            if (m > sum / 4)
                return false;
        for (int i = 0; i < n; ++i) {
            if (!visited[i] && DFS(nums, i, nums[i])) {
                visited[i] = true;
                --match;
            }
        }
        return match == 0;
    }

    bool DFS(vector<int> &nums, int m, int acc) {
        if (acc > sum / 4)
            return false;
        else if (acc == sum / 4)
            return true;
        for (int i = m + 1; i < n; ++i)
            if (!visited[i] && DFS(nums, i, acc + nums[i])) {
                visited[i] = true;
                return true;
            }
        return false;
    }
};
```
#### [980 different paths III](https://leetcode-cn.com/problems/unique-paths-iii/)

Use a variable zeros to record the number of 0s in the matrix. Each time it traverses to 0, that is, zeros - 1, until zeros == 0 and there are end points in the four directions of the current point, then the result is +1 and returned, and the next step of DFS continues.
```c++
class Solution {
    int m, n, zeros, res;
    vector<vector<bool>> visited;
    int dir[4][2] = {{0,  1},
                     {1,  0},
                     {0,  -1},
                     {-1, 0}};
public:
    int uniquePathsIII(vector<vector<int>> &grid) {
        m = grid.size(), n = m ? grid[0].size() : 0, zeros = m * n - 2, res = 0;
        visited = vector<vector<bool>>(m, vector<bool>(n, false));
        int sr, sc, er, ec;
        for (int i = 0; i < m; ++i)
            for (int j = 0; j < n; ++j)
                if (grid[i][j] == 1)
                    sr = i, sc = j;
                else if (grid[i][j] == -1)
                    --zeros;
        DFS(grid, sr, sc, 0);
        return res;
    }

    void DFS(vector<vector<int>> &grid, int r, int c, int count) {
        visited[r][c] = true;
        for (auto &d:dir) {
            int i = r + d[0], j = c + d[1];
            if (i >= 0 && i < m && j >= 0 && j < n) {
                if (grid[i][j] == 2 && count == zeros) {
                    ++res;
                    break;
                }
                if (grid[i][j] == 0 && !visited[i][j])
                    DFS(grid, i, j, count + 1);
            }
        }
        visited[r][c] = false;
    }
};
```
#### [37 Solving Sudoku](https://leetcode-cn.com/problems/sudoku-solver/)

Backtrack from '1' to '9' for each '.' grid, and determine whether there are the same values in the current row, column, and 3 * 3 grid until reaching the end of the matrix.
```c++
class Solution {
    int m, n;
public:
    void solveSudoku(vector<vector<char>> &board) {
        m = board.size(), n = board[0].size();
        DFS(board, 0, 0);
    }

    bool DFS(vector<vector<char>> &board, int i, int j) {
        if (j >= n)
            return DFS(board, i + 1, 0);
        else if (i >= m)
            return true;
        else if (board[i][j] != '.')
            return DFS(board, i, j + 1);
        for (char c = '1'; c <= '9'; ++c) {
            if (CheckNum(board, i, j, c)) {
                board[i][j] = c;
                if (DFS(board, i, j + 1))
                    return true;
                board[i][j] = '.';
            }
        }
        return false;
    }

    bool CheckNum(vector<vector<char>> &board, const int &i, const int &j, const char &c) {
        for (int k = 0; k < 9; ++k)
            if (board[k][j] == c || board[i][k] == c)
                return false;
        for (int a = 0; a < 3; ++a)
            for (int b = 0; b < 3; ++b)
                if (board[a + i / 3 * 3][b + j / 3 * 3] == c)
                    return false;
        return true;
    }
};
```
#### [79 word search](https://leetcode-cn.com/problems/word-search/)

Just perform DFS once in the matrix.
```c++
class Solution {
    int m, n;
    int dir[4][2] = {{0,  1}, {1,  0}, {0,  -1}, {-1, 0}};
    vector<vector<bool>> visited;
public:
    bool exist(vector<vector<char>> &board, string word) {
        m = board.size(), n = m ? board[0].size() : 0;
        visited = vector<vector<bool>>(m, vector<bool>(n, false));
        for (int i = 0; i < m; ++i)
            for (int j = 0; j < n; ++j)
                if (board[i][j] == word[0] && DFS(board, i, j, word.substr(1)))
                    return true;
        return false;
    }

    bool DFS(vector<vector<char>> &board, int i, int j, string word) {
        if (word == "")
            return true;
        visited[i][j] = true;
        for (auto &d:dir) {
            int a = i + d[0], b = j + d[1];
            if (a >= 0 && a < m && b >= 0 && b < n && board[a][b] == word[0] && !visited[a][b] &&
                DFS(board, a, b, word.substr(1)))
                return true;
        }
        visited[i][j] = false;
        return false;
    }
};
```
#### [212 Word Search II](https://leetcode-cn.com/problems/word-search-ii/)

The simplest method is to perform DFS in the matrix once for each word. In this case, the time complexity is O(m * n * k * l), where m is the length of the matrix, n is the width of the matrix, l is the number of words, and k is the longest length of all words. We can build a dictionary tree for all words, and then perform a DFS in the matrix. At each point in the matrix, we determine whether the current letter is in the next array of the root node of the dictionary tree. If so, search for the letters around it and continue traversing the dictionary tree. The time complexity of doing so is O(m * n * k).
```c++
class Solution {
    struct TrieNode {
        vector<TrieNode *> next;
        bool end;

        TrieNode() {
            next = vector<TrieNode *>(26, nullptr);
            end = false;
        }
    };

    TrieNode *root;
    int m, n;
    unordered_set<string> res;
    vector<string> ret;
    vector<vector<bool>> visited;
    int dir[4][2] = {{0,  1}, {1,  0}, {0,  -1}, {-1, 0}};
public:
    vector<string> findWords(vector<vector<char>> &board, vector<string> &words) {
        res = unordered_set<string>();
        m = board.size(), n = board[0].size();
        visited = vector<vector<bool>>(m, vector<bool>(n, false));
        BuildTrie(words);
        for (int i = 0; i < m; ++i)
            for (int j = 0; j < n; ++j)
                if (root->next[board[i][j] - 'a'])
                    DFS(board, i, j, root->next[board[i][j] - 'a'], string(1, board[i][j]));
        ret = vector<string>(res.begin(), res.end());
        return ret;
    }

    void BuildTrie(vector<string> &words) {
        root = new TrieNode();
        TrieNode *node;
        for (auto &s:words) {
            node = root;
            for (auto c:s) {
                if (!node->next[c - 'a'])
                    node->next[c - 'a'] = new TrieNode();
                node = node->next[c - 'a'];
            }
            node->end = true;
        }
    }

    void DFS(vector<vector<char>> &board, int i, int j, TrieNode *node, string word) {
        if (!node)
            return;
        if (node->end)
            res.insert(word);
        visited[i][j] = true;
        for (auto &d:dir) {
            int a = i + d[0], b = j + d[1];
            if (a >= 0 && a < m && b >= 0 && b < n && node->next[board[a][b] - 'a'] && !visited[a][b])
                DFS(board, a, b, node->next[board[a][b] - 'a'], word + board[a][b]);
        }
        visited[i][j] = false;
    }
};
```
#### [749 quarantine viruses](https://leetcode-cn.com/problems/contain-virus/)

The matrix will continue to change. After each round of DFS, two operations are required. One is to mark the isolated viruses, and the other is to infect (extend) the unisolated viruses. You can save all the unisolated viruses first and then extend them in sequence. It is more complicated to write.
```c++
class Solution {
    int m, n;
    int dir[4][2] = {{0,  1},
                     {1,  0},
                     {0,  -1},
                     {-1, 0}};
    vector<vector<bool>> visited;
public:
    int containVirus(vector<vector<int>> &grid) {
        m = grid.size(), n = m ? grid[0].size() : 0;
        int res = 0;
        bool exist = true;
        while (exist) {
            exist = false;
            int perimeter = 0, co_x = 0, co_y = 0;
            visited = vector<vector<bool>>(m, vector<bool>(n, false));
            for (int i = 0; i < m; ++i) {
                for (int j = 0; j < n; ++j) {
                    if (grid[i][j] == 1 && !visited[i][j]) {
                        exist = true;
                        int peri = CalcPeri(grid, i, j);
                        if (peri > perimeter) {
                            perimeter = peri;
                            co_x = i, co_y = j;
                        }
                    }
                }
            }
            res += perimeter;
            if (exist) {
                Contain(grid, co_x, co_y);
                Infect(grid);
            }
        }
        return res;
    }

    int CalcPeri(vector<vector<int>> &grid, int i, int j) {
        int peri = 4, res = 0;
        visited[i][j] = true;
        for (auto &d:dir) {
            int a = i + d[0], b = j + d[1];
            if (a >= 0 && a < m && b >= 0 && b < n) {
                if (grid[a][b] != 0)
                    --peri;
                if (grid[a][b] == 1 && !visited[a][b])
                    res += CalcPeri(grid, a, b);
            } else
                --peri;
        }
        return res + peri;
    }

    void Contain(vector<vector<int>> &grid, int i, int j) {
        grid[i][j] = 2;
        for (auto &d:dir) {
            int a = i + d[0], b = j + d[1];
            if (a >= 0 && a < m && b >= 0 && b < n && grid[a][b] == 1)
                Contain(grid, a, b);
        }
    }

    void Infect(vector<vector<int>> &grid) {
        vector<pair<int, int>> infect;
        for (int i = 0; i < m; ++i)
            for (int j = 0; j < n; ++j)
                if (grid[i][j] == 1)
                    infect.push_back(pair<int, int>(i, j));
        for (auto &f:infect) {
            for (auto &d:dir) {
                int a = f.first + d[0], b = f.second + d[1];
                if (a >= 0 && a < m && b >= 0 && b < n && grid[a][b] == 0)
                    grid[a][b] = 1;
            }
        }
    }
};
```
#### [51 N Queens](https://leetcode-cn.com/problems/n-queens/)

A very classic backtracking problem, use DFS to search every possibility until the last row is searched, and use the sum and difference of the horizontal and vertical coordinates of the current position to determine whether there is a queen on the two diagonals.
```c++
class Solution {
public:
    vector<vector<string>> solveNQueens(int n) {
        vector<vector<string>> res;
        string temp = "";
        for (int i = 0; i < n; ++i)
            temp += ".";
        vector<string> board(n, temp);
        unordered_map<int, bool> left_diagonal, right_diagonal;
        vector<bool> row(n, false), col(n, false);
        for (int i = 0; i < n; ++i) {
            for (int j = 0; j < n; ++j) {
                left_diagonal[i + j] = false;
                right_diagonal[i - j] = false;
            }
        }
        Backtrack(0, n, board, res, col, left_diagonal, right_diagonal);
        return res;

    }

    void Backtrack(int i, int &n, vector<string> &board, vector<vector<string>> &res,
                   vector<bool> &col, unordered_map<int, bool> &left_diagonal,
                   unordered_map<int, bool> &right_diagonal) {
        if (i == n) {
            res.push_back(board);
            return;
        }
        for (int j = 0; j < n; ++j) {
            if (!col[j] && !left_diagonal[i + j] && !right_diagonal[i - j]) {
                col[j] = true;
                left_diagonal[i + j] = true;
                right_diagonal[i - j] = true;
                board[i][j] = 'Q';
                Backtrack(i + 1, n, board, res, col, left_diagonal, right_diagonal);
                board[i][j] = '.';
                col[j] = false;
                left_diagonal[i + j] = false;
                right_diagonal[i - j] = false;
            }
        }
    }
};
```

## Original references

- [Reference 1](https://leetcode-cn.com/tag/depth-first-search/)
