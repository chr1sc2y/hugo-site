---
title: "LeetCode: Trees (2)"
date: 2019-07-18T19:12:25+10:00
draft: false
categories: ["LeetCode"]
description: "A translated technical note on LeetCode: Trees (2), preserving the examples and context of the original article."
---
# LeetCode: Trees (2)

> Originally published in Chinese on 2019-07-18; this English edition preserves the original scope and technical context.

## Title

### 3. Binary search tree

#### [95 different binary search trees II](https://leetcode-cn.com/problems/unique-binary-search-trees-ii/)

Generate a binary search tree consisting of 1...n nodes.

In order to construct a binary search tree with i as the root node, we need to first construct all binary search trees with 1 ... i - 1 as the left subtree and all binary search trees with i + 1 ... n as the right subtree, and then arrange and combine these subtrees to obtain all binary search trees with i as the root node.
```c++
class Solution {
public:
    vector<TreeNode *> generateTrees(int n) {
        if (n == 0)
            return {};
        return Generate(1, n);
    }

    vector<TreeNode *> Generate(int m, int n) {
        vector<TreeNode *> nodes;
        if (m == n)
            nodes.push_back(new TreeNode(n));
        if (m > n)
            nodes.push_back(nullptr);
        if (m >= n)
            return nodes;
        for (int i = m; i <= n; ++i) {
            vector<TreeNode *> left = Generate(m, i - 1);
            vector<TreeNode *> right = Generate(i + 1, n);
            for (auto &l:left)
                for (auto &r:right) {
                    TreeNode *node = new TreeNode(i);
                    node->left = l, node->right = r;
                    nodes.push_back(node);
                }
        }
        return nodes;
    }
};
```
#### [98 Validate Binary Search Tree](https://leetcode-cn.com/problems/validate-binary-search-tree/)

Because the in-order traversal result of the binary search tree is an ordered array, one method is to save the in-order traversal result for judgment. You can also judge the size relationship between the child nodes and the root node according to the definition of the binary search tree.
```c++
class Solution {
public:
    bool isValidBST(TreeNode *root) {
        return IsValidSubtree(root, nullptr, nullptr);
    }

    bool IsValidSubtree(TreeNode *node, const int *min_val, const int *max_val) {
        if (!node)
            return true;
        if ((min_val && *min_val >= node->val) || (max_val && *max_val <= node->val))
            return false;
        return IsValidSubtree(node->left, min_val, &(node->val)) && IsValidSubtree(node->right, &(node->val), max_val);
    }
};
```
#### [108 Convert sorted array to binary search tree](https://leetcode-cn.com/problems/convert-sorted-array-to-binary-search-tree/)

Given an ordered array, convert it into a balanced binary search tree.

The result of the in-order traversal of the binary search tree is an ordered array, so you only need to find the middle element each time as the root node, the left subarray as the left subtree, and the right subarray as the right subtree, and construct it recursively.
```c++
class Solution {
public:
    TreeNode *sortedArrayToBST(vector<int> &nums) {
        return Build(nums, 0, nums.size());
    }

    TreeNode *Build(vector<int> &nums, int x, int y) {
        if (x >= y)
            return nullptr;
        int pos = x + (y - x) / 2;
        TreeNode *node = new TreeNode(nums[pos]);
        node->left = Build(nums, x, pos);
        node->right = Build(nums, pos + 1, y);
        return node;
    }
};
```
#### [235 nearest common ancestor of a binary search tree](https://leetcode-cn.com/problems/lowest-common-ancestor-of-a-binary-search-tree/)

Finds the nearest common ancestor of two specified nodes in a binary search tree.

It can be seen from the binary search tree that if the value of two nodes is greater than the root node, then they should both be on the right subtree of the root node; if the value of both nodes is less than the root node, then they should both be on the left subtree of the root node; otherwise they may be anywhere on the root node and its subtrees, then the root node is their nearest common ancestor.
```c++
class Solution {
public:
    TreeNode *lowestCommonAncestor(TreeNode *root, TreeNode *p, TreeNode *q) {
        if (!root)
            return nullptr;
        else if (root->val > p->val && root->val > q->val)
            return lowestCommonAncestor(root->left, p, q);
        else if (root->val < p->val && root->val < q->val)
            return lowestCommonAncestor(root->right, p, q);
        return root;
    }
};
```
#### [671 Second smallest node in a binary tree](https://leetcode-cn.com/problems/second-minimum-node-in-a-binary-tree/)

Given a binary tree whose number of child nodes is only 0 or 2, and the value of the root node must be less than or equal to the value of the child node, find the second smallest value among all nodes.

Because the root node on the binary tree must be less than or equal to the child node, the value of the root node of the entire tree must be the minimum value. You only need to traverse the entire tree and find the minimum value except the root node.
```c++
class Solution {
public:
    int findSecondMinimumValue(TreeNode *root) {
        if (!root)
            return -1;
        int first = root->val, *res = nullptr;
        queue<TreeNode *> q;
        q.push(root);
        while (!q.empty()) {
            TreeNode *node = q.front();
            q.pop();
            if (first < node->val)
                if (!res)
                    res = new int(node->val);
                else
                    *res = min(*res, node->val);
            if (node->left && node->right)
                q.push(node->left), q.push(node->right);
        }
        return res ? *res : -1;
    }
};
```
#### [230 Kth smallest element in binary search tree](https://leetcode-cn.com/problems/kth-smallest-element-in-a-bst/)

Find the kth smallest element in a binary search tree.

Because the results of in-order traversal of a binary search tree are in order, we can use in-order traversal and terminate early and return the result when the kth smallest element is found.
```c++
class Solution {
    int res;
public:
    int kthSmallest(TreeNode *root, int k) {
        res = 0;
        Inorder(root, k);
        return res;
    }

    void Inorder(TreeNode *root, int &k) {
        if (!root)
            return;
        Inorder(root->left, k);
        if (k <= 0)
            return;
        --k;
        if (k == 0) {
            res = root->val;
            return;
        }
        Inorder(root->right, k);
    }
};
```
#### [450 Delete nodes in binary search tree](https://leetcode-cn.com/problems/delete-node-in-a-bst/)

Given a binary search tree and a value, delete the corresponding node in the binary search tree.

According to the definition of a binary search tree, it is easy to find the corresponding node through the size relationship. After finding it, you only need to replace the original node with the largest node on the left subtree, which is the rightmost child node of the left child node. Pay attention to connecting the left subtree of the rightmost child node of the left child node to the right child node of its parent node.
```c++
class Solution {
public:
    TreeNode *deleteNode(TreeNode *root, int key) {
        if (!root)
            return nullptr;
        if (root->val == key) {
            if (!root->left)
                return root->right;
            auto node = root->left, head = node;
            if (!node->right) {
                node->right = root->right;
                return node;
            }
            while (node->right)
                head = node, node = node->right;
            head->right = node->left;
            node->left = root->left, node->right = root->right;
            return node;
        } else if (root->val < key)
            root->right = deleteNode(root->right, key);
        else if (root->val > key)
            root->left = deleteNode(root->left, key);
        return root;
    }
};
```
#### [669 Trim Binary Search Tree](https://leetcode-cn.com/problems/trim-a-binary-search-tree/)

Given a binary search tree, and a minimum bound L and a maximum bound R, prune the binary search tree so that all node values are in the range [L, R].

If the value of a node is outside the range, according to the definition of a binary search tree, just return the pruned result of the node in the corresponding direction; if the value of a node is within the range, just build its left and right subtrees respectively.
```c++
class Solution {
public:
    TreeNode *trimBST(TreeNode *root, const int &L, const int &R) {
        if (!root)
            return nullptr;
        if (root->val < L)
            return trimBST(root->right, L, R);
        if (root->val > R)
            return trimBST(root->left, L, R);
        root->left = trimBST(root->left, L, R);
        root->right = trimBST(root->right, L, R);
        return root;
    }
};
```
#### [530 Minimum absolute difference of binary search tree](https://leetcode-cn.com/problems/minimum-absolute-difference-in-bst/)

Find the minimum absolute value of the difference between any two nodes in a binary search tree.

Because the in-order traversal result of a binary search tree is ordered, the minimum absolute value of the difference between any two nodes must occur between two adjacent values. Therefore, an in-order traversal is performed and the minimum absolute value of the difference between the two nodes is updated at the same time.

class Solution {
    TreeNode *node;
    int res;
public:
    int getMinimumDifference(TreeNode *root) {
        node = nullptr;
        res = INT_MAX;
        Inorder(root);
        return res;
    }

    void Inorder(TreeNode *root) {
        if (!root)
            return;
        Inorder(root->left);
        if (!node)
            node = root;
        else
            res = min(res, abs(node->val - root->val));
        node = root;
        Inorder(root->right);
    }
};

#### [783 Minimum distance between binary search tree nodes](https://leetcode-cn.com/problems/minimum-distance-between-bst-nodes/)

Find the minimum absolute value of the difference between any two nodes in a binary search tree.

Same as above.

class Solution {
    TreeNode *node;
    int res;
public:
    int minDiffInBST(TreeNode *root) {
        node = nullptr;
        res = INT_MAX;
        Inorder(root);
        return res;
    }

    void Inorder(TreeNode *root) {
        if (!root)
            return;
        Inorder(root->left);
        if (!node)
            node = root;
        else
            res = min(res, abs(node->val - root->val));
        node = root;
        Inorder(root->right);
    }
};

#### [501 Mode in binary search tree](https://leetcode-cn.com/problems/find-mode-in-binary-search-tree/)

Find all modes in a binary search tree.

Because the results of the in-order traversal of the binary search tree are ordered, you can directly perform an in-order traversal and update the result array at the same time.
```c++
class Solution {
    vector<int> res;
    TreeNode *node;
    int n, m;
public:
    vector<int> findMode(TreeNode *root) {
        res = vector<int>();
        node = nullptr;
        n = m = 0;
        Inorder(root);
        return res;
    }

    void Inorder(TreeNode *root) {
        if (!root)
            return;
        Inorder(root->left);
        if (!node || node->val != root->val)
            m = 1;
        else
            ++m;
        node = root;
        if (m > n) {
            n = m;
            res.clear();
        }
        if (m == n)
            res.push_back(node->val);
        Inorder(root->right);
    }
};
```
#### [538 Convert binary search tree to cumulative tree](https://leetcode-cn.com/problems/convert-bst-to-greater-tree/)

Convert a binary search tree to a cumulative tree.

According to the definition of a binary search tree, the value of each node must be smaller than the node value on the right subtree, so the entire tree is traversed in right, middle, and left order, and the cumulative value on the right is added to the root node.
```c++
class Solution {
    int val;
public:
    TreeNode *convertBST(TreeNode *root) {
        val = 0;
        Accumulate(root);
        return root;
    }

    void Accumulate(TreeNode *root) {
        if (!root)
            return;
        Accumulate(root->right);
        root->val += val;
        val = root->val;
        Accumulate(root->left);
    }
};
```
#### [700 Search in a binary search tree](https://leetcode-cn.com/problems/search-in-a-binary-search-tree/)

Search for a specific value in a binary search tree.

Just search according to the characteristics of the binary search tree.
```c++
class Solution {
public:
    TreeNode *searchBST(TreeNode *root, const int &val) {
        if (!root)
            return nullptr;
        if (root->val < val)
            return searchBST(root->right, val);
        else if (root->val > val)
            return searchBST(root->left, val);
        else
            return root;
    }
};
```
#### [701 Insertion operation in a binary search tree](https://leetcode-cn.com/problems/insert-into-a-binary-search-tree/)

Insert a value into a binary search tree.

Search downward according to the relationship between the given value and the value of the node until an empty node is found, create a new node and return.
```c++
class Solution {
public:
    TreeNode *insertIntoBST(TreeNode *root, const int &val) {
        if (!root)
            return new TreeNode(val);
        else if (val < root->val)
            root->left = insertIntoBST(root->left, val);
        else if (val > root->val)
            root->right = insertIntoBST(root->right, val);
        return root;
    }
};
```
#### [938 Range sum of binary search tree](https://leetcode-cn.com/problems/range-sum-of-bst/)

Given a binary search tree, calculate the sum of the values of all nodes between L and R.

Just judge the value of the root node L <= val <= R.
```c++
class Solution {
public:
    int rangeSumBST(TreeNode* root, const int &L, const int &R) {
        if (!root)
            return 0;
        int left = 0, right = 0;
        if (root->val >= L)
            left = rangeSumBST(root->left, L, R);
        if (root->val <= R)
            right = rangeSumBST(root->right, L, R);
        return left + right + (root->val >= L && root->val <= R ? root->val : 0);
    }
};
```
#### [99 Recover binary search tree](https://leetcode-cn.com/problems/recover-binary-search-tree/)

Recover a binary search tree in which two nodes were mistakenly swapped.

Because only two nodes are mistakenly exchanged, you only need to do an in-order traversal to find these two nodes from the size relationship of adjacent nodes, and exchange the previous node with the wrong position relationship for the first time with the node after the second time with the wrong position relationship.
```c++
class Solution {
    TreeNode *first, *second, *prev;
public:
    void recoverTree(TreeNode *root) {
        first = second = prev = nullptr;
        Inorder(root);
        swap(first->val, second->val);
    }

    void Inorder(TreeNode *root) {
        if (!root)
            return;
        Inorder(root->left);
        if (prev && prev->val >= root->val && !first)
            first = prev;
        if (prev && prev->val >= root->val && first)
            second = root;
        prev = root;
        Inorder(root->right);
    }
};
```

## Original references

- [Reference 1](https://leetcode-cn.com/tag/tree/)
