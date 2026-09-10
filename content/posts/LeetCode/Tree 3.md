---
title: "LeetCode: Trees (3)"
date: 2019-08-24T19:12:25+10:00
draft: false
categories: ["LeetCode"]
description: "A translated technical note on LeetCode: Trees (3), preserving the examples and context of the original article."
---
# LeetCode: Trees (3)

> Originally published in Chinese on 2019-08-24; this English edition preserves the original scope and technical context.

## Title

### 4. Recursive solution

#### [617 Merge Binary Trees](https://leetcode-cn.com/problems/merge-two-binary-trees/)

Merge two binary trees.

Determine whether each node exists and merge them all into one tree.
```c++
class Solution {
public:
    TreeNode *mergeTrees(TreeNode *t1, TreeNode *t2) {
        if (!t1 && !t2)
            return nullptr;
        else if (!t1)
            return t2;
        else if (!t2)
            return t1;
        t1->val += t2->val;
        t1->left = mergeTrees(t1->left, t2->left);
        t1->right = mergeTrees(t1->right, t2->right);
        return t1;
    }
};
```
#### [226 Invert Binary Tree](https://leetcode-cn.com/problems/invert-binary-tree/)

Flip a binary tree.

First flip the left and right subtrees respectively, and then exchange the positions of the two.
```c++
class Solution {
public:
    TreeNode *invertTree(TreeNode *root) {
        if (!root)
            return nullptr;
        TreeNode *left = invertTree(root->left), *right = invertTree(root->right);
        root->right = left;
        root->left = right;
        return root;
    }
};
```
#### [104 Maximum depth of binary tree](https://leetcode-cn.com/problems/maximum-depth-of-binary-tree/)

Find the maximum depth of a binary tree.

The depth of each layer is 1, plus the greater depth in the left and right subtrees, which is the maximum depth.
```c++
class Solution {
public:
    int maxDepth(TreeNode *root) {
        if (!root)
            return 0;
        return 1 + max(maxDepth(root->left), maxDepth(root->right));
    }
};
```
#### [965 univalued binary tree](https://leetcode-cn.com/problems/univalued-binary-tree/)

Determine whether a binary tree is a single-valued binary tree.

Just determine whether the value of each node and its left and right nodes are the same.
```c++
class Solution {
public:
    bool isUnivalTree(TreeNode *root) {
        if (!root)
            return true;
        return (root->left ? root->val == root->left->val : true) && (root->right ? root->val == root->right->val : true) && isUnivalTree(root->left) && isUnivalTree(root->right);
    }
};
```
#### [559 Maximum depth of N-ary tree](https://leetcode-cn.com/problems/maximum-depth-of-n-ary-tree/)

Find the maximum depth of an N-ary tree.

The depth of each layer is 1, plus the maximum depth among all its subtrees is the maximum depth.
```c++
class Solution {
public:
    int maxDepth(Node* root) {
        if (!root)
            return 0;
        int depth = 0;
        for (auto &c:root->children)
            depth = max(depth, maxDepth(c));
        return 1 + depth;
    }
};
```
#### [563 Slope of binary tree](https://leetcode-cn.com/problems/binary-tree-tilt/)

Compute the slope of a binary tree.

For each node, calculate the sum of its left subtree and right subtree, add the absolute value of the difference to the total slope, and then return the sum of the left subtree, right subtree, and its own value, and call it recursively.
```c++
class Solution {
    int res;
public:
    int findTilt(TreeNode *root) {
        res = 0;
        CalcTilt(root);
        return res;
    }

    int CalcTilt(TreeNode *node) {
        if (!node)
            return 0;
        int left = CalcTilt(node->left);
        int right = CalcTilt(node->right);
        res += abs(left - right);
        return node->val + left + right;
    }
};
```
#### [508 The most frequent subtree element sum](https://leetcode-cn.com/problems/most-frequent-subtree-sum/submissions/)

Find the sum of subtree elements that appear most frequently in a binary tree.

Calculate the sum of subtree elements of the left subtree and right subtree of a node, plus its own value to get a complete sum of subtree elements. Just recursively call to calculate all nodes and count.
```c++
class Solution {
    unordered_map<int, int> count;
public:
    vector<int> findFrequentTreeSum(TreeNode *root) {
        vector<int> res;
        count = unordered_map<int, int>();
        Traverse(root);
        int n = 0;
        for (auto &c:count) {
            if (c.second > n) {
                n = c.second;
                res.clear();
                res.push_back(c.first);
            } else if (c.second == n)
                res.push_back(c.first);
        }
        return res;
    }

    int Traverse(TreeNode *root) {
        if (!root)
            return 0;
        int val = Traverse(root->left) + Traverse(root->right) + root->val;
        ++count[val];
        return val;
    }
};
```
### 5. Stack solution

#### [623 Add one row to a binary tree](https://leetcode-cn.com/problems/add-one-row-to-tree/)

Given a binary tree, append a row of nodes with value v at level d.

Use a stack to save all the nodes of a layer and traverse them layer by layer. Note that d = 1 needs to be handled separately.
```c++
class Solution {
public:
    TreeNode *addOneRow(TreeNode *root, int v, int d) {
        if (!root)
            return nullptr;
        if (d == 1) {
            TreeNode *new_root = new TreeNode(v);
            new_root->left = root;
            return new_root;
        }
        queue<TreeNode *> q;
        q.push(root);
        int depth = 1, n = 1;
        while (!q.empty() && depth < d) {
            for (int i = 0; i < n; ++i) {
                TreeNode *node = q.front();
                q.pop();
                if (depth == d - 1) {
                    TreeNode *left = node->left, *right = node->right;
                    node->left = new TreeNode(v);
                    node->right = new TreeNode(v);
                    node->left->left = left;
                    node->right->right = right;
                }
                if (node->left)
                    q.push(node->left);
                if (node->right)
                    q.push(node->right);
            }
            ++depth;
            n = q.size();
        }
        return root;
    }
};
```
### 6. Find nodes

#### [1123. The nearest common ancestor of the deepest leaf node](https://leetcode-cn.com/problems/lowest-common-ancestor-of-deepest-leaves/)

Find the nearest common ancestor of the deepest leaf node of a binary tree.

You can first use level-order traversal to find the depth of the binary tree, and then find the common ancestor of all leaf nodes through one recursion.
```c++
class Solution {
    TreeNode *res;
    int lvl;
public:
    TreeNode *lcaDeepestLeaves(TreeNode *root) {
        if (!root)
            return nullptr;
        res = nullptr;
        lvl = 0;
        queue<TreeNode *> nodes;
        nodes.push(root);
        int n = 1;
        while (!nodes.empty()) {
            for (int i = 0; i < n; ++i) {
                TreeNode *node = nodes.front();
                nodes.pop();
                if (node->left) nodes.push(node->left);
                if (node->right) nodes.push(node->right);
            }
            ++lvl;
            n = nodes.size();
        }
        FindLCA(root, 1);
        return res;
    }

    bool FindLCA(TreeNode *root, int l) {
        if (root && l == lvl) {
            res = root;
            return true;
        } else if (!root)
            return false;
        bool left = FindLCA(root->left, l + 1), right = FindLCA(root->right, l + 1);
        if (left && right)
            res = root;
        return left || right;
    }
};
```
But in fact we do not need to know the depth of the tree, we only need to know that the deepest node is the leaf node, and if the depth of the deepest node of the left subtree and right subtree of a node is the same, then this node is their most recent common ancestor, and this node can be returned.
```c++
class Solution {
public:
    TreeNode *lcaDeepestLeaves(TreeNode *root) {
        return FindLCA(root).first;
    }

    pair<TreeNode *, int> FindLCA(TreeNode *root) {
        if (!root)
            return pair<TreeNode *, int>(nullptr, 0);
        auto left = FindLCA(root->left), right = FindLCA(root->right);
        if (left.second > right.second)
            return pair<TreeNode *, int>(left.first, left.second + 1);
        if (left.second < right.second)
            return pair<TreeNode *, int>(right.first, right.second + 1);
        return pair<TreeNode *, int>(root, left.second + 1);
    }
}
```

## Original references

- [Reference 1](https://leetcode-cn.com/tag/tree/)
