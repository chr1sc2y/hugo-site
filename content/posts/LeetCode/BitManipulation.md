---
title: "LeetCode: Bit Manipulation"
date: 2019-06-19T19:26:39+10:00
draft: false
categories: ["LeetCode"]
description: "A translated technical note on LeetCode: Bit Manipulation, preserving the examples and context of the original article."
---
# LeetCode: Bit Manipulation

> Originally published in Chinese on 2019-06-19; this English edition preserves the original scope and technical context.

Bit operations include:

1. with &
2. or |
3. XOR ^
4. Negation ~
5. Move left <<
6. Move right >>

## Tips

1. Shift operation

    - x << 1: arithmetic left shift
        - All bits in the binary representation of a number are shifted one position to the left, which is equivalent to multiplying by 2
        - pad 0 on the right
    - x >> 1: arithmetic right shift
        - All bits in the binary representation of a number are shifted one position to the right, which is equivalent to dividing by 2
        - Complement the sign bit on the left, that is, complement 0 for positive numbers, and complement 1 for negative numbers (based on the two's complement code)
    - negative shift
        - Negative numbers are stored in the form of two's complement. When a negative number is shifted to the right, it needs to be inverted and converted into its complement, plus one to convert it into its complement, then moved to the right one bit to get a new complement, then subtracted by one to get a new one's complement, and then inverted and converted into the original code to get the result. For example, the binary representation of -7 is 10000111 (because 32 bits are too long, so an 8-bit int is used here), its complement is 11111000, and its complement is 11111001. Moving one position to the right is 11111100, and subtracting one gets the new complement 11111011. The original code is 10000100, that is -4; shifting the complement one bit to the left is 11110010, subtracting one to get the new complement 11110001, the original code is 10001110, which is -14
        - A simpler way to understand is to multiply left shift by 2 and right shift by 2. For example -7 >> 1 = -7 / 2 = -4, -7 << 1 = -14

## Title

### 1. Single number

#### [693 alternating bits binary number](https://leetcode-cn.com/problems/binary-number-with-alternating-bits/)

Checks whether two adjacent digits of a binary number are not equal.

Just judge by bit by bit & 1.
```c++
class Solution {
public:
    bool hasAlternatingBits(int n) {
        bool rel = n & 1;
        while (n > 0 && (n & 1) == rel) {
            rel = !rel;
            n >>= 1;
        }
        return n <= 0;
    }
};
```
#### [Complement of 476 numbers](https://leetcode-cn.com/problems/number-complement/)

Given a positive integer, find the binary representation of the negated result, excluding leading 0s.

Just XOR 1 for each bit. Subtract one from the first 2^n number greater than num to get a number whose binary representation is all ones and has a number of digits equal to the number of num digits.
```c++
class Solution {
public:
    int findComplement(int num) {
        long long util = 1;
        while (util <= num)
            util <<= 1;
        return num ^ (util - 1);
    }
};
```
#### [461 Hamming distance](https://leetcode-cn.com/problems/hamming-distance/)

Computes the number of bits by which the binary representations of two integers differ.

Just perform XOR judgment bit by bit.
```c++
class Solution {
public:
    int hammingDistance(int x, int y) {
        int res = 0;
        while (x > 0 || y > 0) {
            res += ((x & 1) ^ (y & 1));
            x >>= 1, y >>= 1;
        }
        return res;
    }
};
```
#### [762 prime number of set bits in binary representation](https://leetcode-cn.com/problems/prime-number-of-set-bits-in-binary-representation/)

Count the number of prime numbers in [L, R].

First, judge whether the number i is equal to 1 in the current position bit by bit, and then judge whether it is a prime number after getting the number of bits set. When judging prime numbers, you can first exclude the cases where modulo 6 is equal to 0, 2, 3, and 4, because these cases can be divisible by 6, 2, 3, 2 respectively, and then judge whether the modulo 6n - 1 and 6n + 1 are equal to 0. You can also store prime numbers less than or equal to 32 in a hash table or array first, so that the query time will be reduced. The time complexity is O(n), n is the number of [L, R].
```c++
class Solution {
public:
    int countPrimeSetBits(int L, int R) {
        int res = 0;
        for (int i = L; i <= R; ++i) {
            int m = i;
            int cnt = 0;
            while (m > 0) {
                if (m & 1)
                    ++cnt;
                m >>= 1;
            }
            if (IsPrime(cnt))
                ++res;
        }
        return res;
    }

    bool IsPrime(int num) {
        if (num < 4)
            return num > 1;
        else if (num % 6 != 1 && num % 6 != 5)
            return false;
        for (int i = 5; i <= sqrt(num); i += 6)
            if (num % i == 0 || num % (i + 2) == 0)
                return false;
        return true;
    }
};
```
#### [136 A number that appears only once](https://leetcode-cn.com/problems/single-number/)

Given an array, all other numbers appear twice, and only one number appears once. Find this number.

Because a ^ a = 0, you can XOR all the numbers in this array with 0, and the final number you get is the only number.
```c++
class Solution {
public:
    int singleNumber(vector<int>& nums) {
        int res = 0;
        for (auto &m:nums)
            res ^= m;
        return res;
    }
};
```

## Original references

- [Reference 1](https://leetcode-cn.com/tag/bit-manipulation/)
