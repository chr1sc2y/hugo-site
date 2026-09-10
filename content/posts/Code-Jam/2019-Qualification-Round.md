---
title: "Code Jam 2019 Qualification Round"
date: 2019-04-06T13:40:27+10:00
draft: false
categories: ["Code Jam"]
description: "A translated technical note on Code Jam 2019 Qualification Round, preserving the examples and context of the original article."
---
# Code Jam 2019 Qualification Round

> Originally published in Chinese on 2019-04-06; this English edition preserves the original scope and technical context.

## [Foregone Solution (6pts, 10pts, 1pts)](https://codingcompetitions.withgoogle.com/codejam/round/0000000000051705/0000000000088231)

Split a number with the digit 4 into two numbers without the digit 4.

### Solution: Construction

The entered number must contain the number 4. For the number 4 on each digit, we can split it into two numbers 2+2 (or 1+3). The maximum input data is 10 to the power of 100, so we can process it as a string.

- Time complexity: O(n)
- Space complexity: O(1)
```C++
// C++
#include <iostream>
#include <cmath>
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
    int T;
    cin >> T;
    for (int t = 1; t <= T; ++t) {
        string N;
        cin >> N;
        string a, b;
        for (auto c:N) {
            a += c == '4' ? '2' : c;
            b += c == '4' ? '2' : '0';
        }
        while (a[0] == '0')
            a.erase(a.begin());
        while (b[0] == '0')
            b.erase(b.begin());
        cout << "Case #" << t << ": " << a << " " << b << endl;
    }
}
```
## [You Can Go Your Own Way (5pts, 9pts, 10pts)](https://codingcompetitions.withgoogle.com/codejam/round/0000000000051705/00000000000881da)

When walking from (0,0) to (n-1,n-1) in an n*n matrix, you can only go right or downward. There is an existing path in the matrix, and it cannot overlap with this path. The most common approach is DFS/BFS, and the time complexity is O(n^2).

### Solution: Mirror

Because there is only one known path, we can mirror it diagonally, and the new path obtained must not coincide with the original path.

- Time complexity: O(2 * n - 2)
- Space complexity: O(1)
```C++
// C++
#include <iostream>
#include <cmath>
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
    int T;
    cin >> T;
    for (int t = 1; t <= T; ++t) {
        int N;
        string str;
        cin >> N >> str;
        for (int i = 0; i < N * 2 - 2; ++i)
            str[i] = str[i] == 'E' ? 'S' : 'E';
        cout << "Case #" << t << ": " << str << endl;
    }
}
```
## [Cryptopangrams (10pts, 15pts)](https://codingcompetitions.withgoogle.com/codejam/round/0000000000051705/000000000008830b)

Input the upper limit N and an array product of length L. Each number in the array is the product of adjacent prime numbers in a prime array res of length L+1. There are only 26 prime numbers in the prime number array. Return the result of sorting these 26 prime numbers and mapping them to A-Z. For a relatively small N, you can save all prime numbers less than or equal to N, calculate which two prime numbers product[0] is the product of, then calculate which two prime numbers product[1] is the product of, get res[1], and then calculate the other numbers in the prime number array res. When N is relatively large, the time and space usage will be very high.

### Solution: Greatest Common Denominator

Rather than finding all the prime numbers first and then finding which two prime numbers product[0] is the product of, we can use the euclidean division method we learned in elementary school to find this prime number, which can very effectively reduce time and space consumption.

The maximum value of N in the test case of this question is 10 to the power of 100. You need to handle large number multiplication by yourself using C++ (C++ only supports a maximum of 128-bit int numbers).

- Time complexity: O(L)
- Space complexity: O(L)
```Python
# Python 3
def GCD(a: int, b: int) -> int:
    if b == 0:
        return a
    return GCD(b, a % b)


T = int(input())
for t in range(1, T + 1):
    N, L = map(int, input().split())
    product = list(map(int, input().split()))
    res = [0 for _ in range(L + 1)]
    pos = -1
    for i in range(L - 1):
        if product[i] != product[i + 1]:
            res[i + 1] = GCD(product[i], product[i + 1])
            pos = i + 1
            break
    for i in range(pos + 1, L + 1):
        res[i] = product[i - 1] // res[i - 1]
    for i in range(pos - 1, -1, -1):
        res[i] = product[i] // res[i + 1]
    match = []
    for m in res:
        if m not in match:
            match.append(m)
    match.sort()
    ret = [chr(match.index(res[i]) + ord('A')) for i in range(len(res))]
    print("Case #{t}: {str}".format(t=t, str="".join(ret)))
```

## [Dat Bae (14pts, 20pts)](https://codingcompetitions.withgoogle.com/codejam/round/0000000000051705/00000000000881de)

// TODO

## Original references

- [Reference 1](https://codingcompetitions.withgoogle.com/codejam/round/0000000000051705)
