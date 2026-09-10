---
title: "LeetCode: Trees (1)"
date: 2019-07-13T19:12:25+10:00
draft: false
categories: ["LeetCode"]
description: "A translated technical note on LeetCode: Trees (1), preserving the examples and context of the original article."
---
# LeetCode: Trees (1)

> Originally published in Chinese on 2019-07-13; this English edition preserves the original scope and technical context.

## Title

### 1. Tree traversal

#### [144 Preorder traversal of binary trees](https://leetcode-cn.com/problems/binary-tree-preorder-traversal/)

Preorder traversal of a binary tree.

Preorder traversal traverses a binary tree in the order of the root node, left child node, and right child node. There are two methods: recursive and iterative. For the iterative method, first add the node to the result array, then use a stack to save the right and left child nodes, access them in sequence, and repeat the process.
```c++
class Solution {
        vector<int> res;
public:
    vector<int> preorderTraversal(TreeNode *root) {
        res = vector<int>();
        Preorder(root);
        return res;
    }

    void Preorder(TreeNode *root) {
        if (!root)
            return;
        res.push_back(root->val);
        Preorder(root->left);
        Preorder(root->right);
    }
};
```

```c++
class Solution {
public:
    vector<int> preorderTraversal(TreeNode *root) {
        vector<int> res;
        if (!root)
            return res;
        stack<TreeNode *> pre;
        pre.push(root);
        while (!pre.empty()) {
            TreeNode *node = pre.top();
            pre.pop();
            res.push_back(node->val);
            if (node->right)
                pre.push(node->right);
            if (node->left)
                pre.push(node->left);
        }
        return res;
    }
};
```
#### [589 Preorder traversal of N-ary tree](https://leetcode-cn.com/problems/n-ary-tree-preorder-traversal/)

Preorder traversal of an N-ary tree.

Similar to preorder traversal of a binary tree, there are two methods: recursive and iterative.
```c++
class Solution {
    vector<int> res;
public:
    vector<int> preorder(Node *root) {
        res = vector<int>();
        Preorder(root);
        return res;
    }

    void Preorder(Node *root) {
        if (!root)
            return;
        res.push_back(root->val);
        for (auto &c:root->children)
            Preorder(c);
    }
};
```

```c++
class Solution {
public:
    vector<int> preorder(Node *root) {
        vector<int> res;
        if (!root)
            return res;
        stack<Node *> pre;
        pre.push(root);
        while (!pre.empty()) {
            Node *node = pre.top();
            pre.pop();
            res.push_back(node->val);
            for (auto iter = node->children.rbegin(); iter < node->children.rend(); ++iter)
                pre.push(*iter);
        }
        return res;
    }
};
```
#### [94 In-order traversal of binary trees](https://leetcode-cn.com/problems/binary-tree-inorder-traversal/)

Inorder traversal of a binary tree.

In-order traversal traverses a binary tree in the order of left child node, root node, and right child node. There are two methods: recursion and iteration. For the iterative method, use a stack to save the parent node for last access. Find the leftmost node first, add it to the result array, then access its right child node, and repeat the process.
```c++
class Solution {
    vector<int> res;
public:
    vector<int> inorderTraversal(TreeNode* root) {
        res = vector<int>();
        Inorder(root);
        return res;
    }

    void Inorder(TreeNode *root) {
        if (!root)
            return;
        Inorder(root->left);
        res.push_back(root->val);
        Inorder(root->right);
    }
};
```

```c++
class Solution {
public:
    vector<int> inorderTraversal(TreeNode *root) {
        vector<int> res;
        if (!root)
            return res;
        stack<TreeNode *> in;
        TreeNode *node = root;
        while (node || !in.empty()) {
            while (node) {
                in.push(node);
                node = node->left;
            }
            if (!in.empty()) {
                node = in.top();
                in.pop();
                res.push_back(node->val);
                node = node->right;
            }
        }
        return res;
    }
};
```
#### [145 Postorder traversal of binary trees](https://leetcode-cn.com/problems/binary-tree-postorder-traversal/)

Postorder traversal of a binary tree.

Post-order traversal traverses a binary tree in the order of left child node, right child node, and root node. There are two methods: recursive and iterative. For the iterative method, use a stack to save the parent node for last access. Find the leftmost node first. If it is already a leaf node, add it to the result array. Otherwise, access its right child node. Use last to save the last visited right child node to prevent access again. Repeat this process.
```c++
class Solution {
    vector<int> res;
public:
    vector<int> postorderTraversal(TreeNode* root) {
        res = vector<int>();
        Postorder(root);
        return res;
    }

    void Postorder(TreeNode *root) {
        if (!root)
            return;
        Postorder(root->left);
        Postorder(root->right);
        res.push_back(root->val);
    }
};
```

```c++
class Solution {
public:
    vector<int> postorderTraversal(TreeNode *root) {
        vector<int> res;
        if (!root)
            return res;
        stack<TreeNode *> post;
        TreeNode *node = root, *last = nullptr;
        while (node || !post.empty()) {
            while (node) {
                post.push(node);
                node = node->left;
            }
            if (!post.empty()) {
                node = post.top();
                if (node->right && node->right != last)
                    node = node->right;
                else {
                    res.push_back(node->val);
                    post.pop();
                    last = node;
                    node = nullptr;
                }
            }
        }
        return res;
    }
};
```
#### [590 Postorder traversal of N-ary tree](https://leetcode-cn.com/problems/n-ary-tree-postorder-traversal/)

Similar to post-order traversal of a binary tree, there are two methods: recursive and iterative.
```c++
class Solution {
    vector<int> res;
public:
    vector<int> postorder(Node *root) {
        res = vector<int>();
        Postorder(root);
        return res;
    }

    void Postorder(Node *root) {
        if (!root)
            return;
        for (auto &c:root->children)
            Postorder(c);
        res.push_back(root->val);
    }
};
```

```c++
class Solution {
public:
    vector<int> postorder(Node *root) {
        vector<int> res;
        if (!root)
            return res;
        stack<Node *> post;
        post.push(root);
        while (!post.empty()) {
            Node *node = post.top();
            post.pop();
            res.push_back(node->val);
            for (auto &c:node->children)
                post.push(c);
        }
        reverse(res.begin(), res.end());
        return res;
    }
};
```
#### [102 Binary tree level traversal](https://leetcode-cn.com/problems/binary-tree-level-order-traversal/)

Given a binary tree, return the node values traversed hierarchically.

Use a queue to save all the nodes of the current level of the binary tree. While popping and pushing these nodes into the result array, push the child nodes of these nodes into the queue.
```c++
class Solution {
public:
    vector<vector<int>> levelOrder(TreeNode *root) {
        vector<vector<int>> res;
        if (!root)
            return res;
        queue<TreeNode *> q;
        q.push(root);
        int n = 1;
        while (!q.empty()) {
            vector<int> lvl;
            for (int i = 0; i < n; ++i) {
                TreeNode *node = q.front();
                q.pop();
                lvl.push_back(node->val);
                if (node->left)
                    q.push(node->left);
                if (node->right)
                    q.push(node->right);
            }
            res.push_back(lvl);
            n = q.size();
        }
        return res;
    }
};
```
#### [107 Binary tree level order traversal II](https://leetcode-cn.com/problems/binary-tree-level-order-traversal-ii/)

Given a binary tree, return the bottom-up hierarchical traversal of its node values.

Same as the previous question, you just need to invert the result array, or use a stack to save the result, and then pop it into the result array.
```c++
class Solution {
public:
    vector<vector<int>> levelOrderBottom(TreeNode *root) {
        vector<vector<int>> res;
        if (!root)
            return res;
        queue<TreeNode *> q;
        int n = 1;
        q.push(root);
        while (!q.empty()) {
            vector<int> lvl;
            for (int i = 0; i < n; ++i) {
                TreeNode *node = q.front();
                q.pop();
                lvl.push_back(node->val);
                if (node->left)
                    q.push(node->left);
                if (node->right)
                    q.push(node->right);
            }
            res.push_back(lvl);
            n = q.size();
        }
        reverse(res.begin(), res.end());
        return res;
    }
};
```
#### [429 Level-order traversal of N-ary tree](https://leetcode-cn.com/problems/n-ary-tree-level-order-traversal/)

Level-order traversal of an N-ary tree.

Use a queue to save all the nodes of the current level of the N-ary tree. While popping and pushing these nodes into the result array, push the child nodes of these nodes into the queue.
```c++
class Solution {
public:
    vector<vector<int>> levelOrder(Node *root) {
        vector<vector<int>> res;
        if (!root)
            return res;
        queue<Node *> q;
        q.push(root);
        int n = 1;
        while (!q.empty()) {
            vector<int> lvl;
            for (int i = 0; i < n; ++i) {
                Node *node = q.front();
                q.pop();
                lvl.push_back(node->val);
                for (auto &c:node->children)
                    q.push(c);
            }
            res.push_back(lvl);
            n = q.size();
        }
        return res;
    }
};
```
#### [987 Vertical order traversal of a binary tree](https://leetcode-cn.com/problems/vertical-order-traversal-of-a-binary-tree/)

Traverse a binary tree in vertical order.

Use a red-black tree to save the position of each node in the binary tree. If the position of the root node is (x, y), then the positions of its left child node and right child node are (x - 1, y + 1) and (x + 1, y + 1) respectively, and then save them to the result array in order.
```c++
class Solution {
    map<int, map<int, vector<int>>> matrix;
public:
    vector<vector<int>> verticalTraversal(TreeNode *root) {
        vector<vector<int>> res;
        matrix = map<int, map<int, vector<int>>>();
        Traverse(root, 0, 0);
        for (auto &m:matrix) {
            vector<int> temp;
            for (auto &n:m.second) {
                sort(n.second.begin(), n.second.end());
                temp.insert(temp.end(), n.second.begin(), n.second.end());
            }
            res.push_back(temp);
        }
        return res;
    }

    void Traverse(TreeNode *root, int x, int y) {
        if (!root)
            return;
        if (matrix.find(x) == matrix.end())
            matrix[x] = map<int, vector<int>>();
        if (matrix[x].find(y) == matrix[x].end())
            matrix[x][y] = vector<int>();
        matrix[x][y].push_back(root->val);
        Traverse(root->left, x - 1, y + 1);
        Traverse(root->right, x + 1, y + 1);
    }
};
```
#### [103 Zigzag level traversal of binary trees](https://leetcode-cn.com/problems/binary-tree-zigzag-level-order-traversal/)

Zigzag hierarchical traversal of a binary tree.

Use two stacks to save nodes from left to right and from right to left in turn, and then add them to the result array in turn.
```c++
class Solution {
public:
    vector<vector<int>> zigzagLevelOrder(TreeNode *root) {
        stack<TreeNode *> s1, s2;
        vector<vector<int>> res;
        if (!root)
            return res;
        s1.push(root);
        while (!s1.empty() || !s2.empty()) {
            vector<int> lvl;
            if (s2.empty()) {
                while (!s1.empty()) {
                    TreeNode *node = s1.top();
                    s1.pop();
                    lvl.push_back(node->val);
                    if (node->left)
                        s2.push(node->left);
                    if (node->right)
                        s2.push(node->right);
                }
            } else {
                while (!s2.empty()) {
                    TreeNode *node = s2.top();
                    s2.pop();
                    lvl.push_back(node->val);
                    if (node->right)
                        s1.push(node->right);
                    if (node->left)
                        s1.push(node->left);
                }
            }
            res.push_back(lvl);
        }
        return res;
    }
};
```
#### [124 Maximum path sum in binary tree](https://leetcode-cn.com/problems/binary-tree-maximum-path-sum/)

Use recursion to update the result by adding the maximum path of its left and right subtrees to each node value, and return the larger of its left and right subtrees plus its node value.
```c++
class Solution {
    int res;
public:
    int maxPathSum(TreeNode *root) {
        res = INT_MIN;
        Traverse(root);
        return res;
    }

    int Traverse(TreeNode *node) {
        if (!node)
            return 0;
        int left = max(0, Traverse(node->left));
        int right = max(0, Traverse(node->right));
        res = max(res, left + right + node->val);
        return max(left, right) + node->val;
    }
};
```
#### [968 Monitoring Binary Tree](https://leetcode-cn.com/problems/binary-tree-cameras/)

The greedy method is discussed according to the situation. The three states are: 0 means that the node is not monitored, 1 means that the node has its own monitoring, and 2 means that the node is monitored by a child node; the combination of two child nodes of a node has 6 situations, namely: if both child nodes are 2 (22), then the current node is not monitored and needs to be monitored by the parent node, and 0 is returned; if at least one of the two child nodes is 0 (00, 01, 02, not monitored), then the current node needs to be equipped with monitoring to monitor the child nodes, and 1 is returned; in the remaining two cases, at least one child node has its own monitoring (11, 12), then the current node is monitored by the child node, and its child nodes have all been monitored or have their own monitoring, and 2 is returned; finally, it is necessary to separately determine whether the root node of the tree is in the 0 state, because there is no parent node that can be monitored.
```c++
class Solution {
    int res;
public:
    int minCameraCover(TreeNode *root) {
        res = 0;
        if (Traverse(root) == 0)
            ++res;
        return res;
    }

    int Traverse(TreeNode *node) {
        if (!node)
            return 2;
        int left = Traverse(node->left);
        int right = Traverse(node->right);
        if (left == 2 && right == 2)
            return 0;
        else if (left == 0 || right == 0) {
            ++res;
            return 1;
        }
        return 2;
    }
};
```
### 2. Construct a binary tree

#### [105 Constructing a binary tree from preorder and inorder traversal sequences](https://leetcode-cn.com/problems/construct-binary-tree-from-preorder-and-inorder-traversal/)

Construct a binary tree based on the results of preorder traversal and inorder traversal.

The first element in the pre-order traversal result must be the value of the root node of the binary tree. Therefore, if this value is found in the in-order traversal result, then all elements to the left of this value in the in-order traversal result must be on the left subtree of this root node, and all elements on the right must be on the right subtree of this root node. Assuming that the length of the left side is m, then m after the first element of the pre-order traversal result Each element also corresponds to these elements on the left subtree. Just call these two parts recursively to construct a new subtree.
```c++
class Solution {
public:
    TreeNode *buildTree(vector<int> &preorder, vector<int> &inorder) {
        return Build(preorder, 0, preorder.size(), inorder, 0, inorder.size());
    }

    TreeNode *Build(vector<int> &preorder, int x, int y, vector<int> &inorder, int m, int n) {
        if (x >= y || m >= n)
            return nullptr;
        TreeNode *node = new TreeNode(preorder[x]);
        int pos = m;
        while (pos < n && inorder[pos] != preorder[x])
            ++pos;
        node->left = Build(preorder, x + 1, x + 1 + pos - m, inorder, m, pos);
        node->right = Build(preorder, x + 1 + pos - m, y, inorder, pos + 1, n);
        return node;
    }
};
```
#### [106 Construct a binary tree from inorder and postorder traversal sequences](https://leetcode-cn.com/problems/construct-binary-tree-from-inorder-and-postorder-traversal/)

Construct a binary tree based on the results of post-order traversal and in-order traversal.

The last element in the post-order traversal result must be the value of the root node of the binary tree. Therefore, if this value is found in the in-order traversal result, then all elements to the left of this value in the in-order traversal result must be on the left subtree of this root node, and all elements on the right must be on the right subtree of this root node. Assuming that the length of the left side is m, then in the post-order traversal result, m will be m from the first element back Each element also corresponds to these elements on the left subtree. Just call these two parts recursively to construct a new subtree.
```c++
class Solution {
public:
    TreeNode *buildTree(vector<int> &inorder, vector<int> &postorder) {
        return Build(inorder, 0, inorder.size(), postorder, 0, postorder.size());
    }

    TreeNode *Build(vector<int> &inorder, int x, int y, vector<int> &postorder, int m, int n) {
        if (x >= y || m >= n)
            return nullptr;
        TreeNode *node = new TreeNode(postorder[n - 1]);
        int pos = x;
        while (pos < y && inorder[pos] != postorder[n - 1])
            ++pos;
        node->left = Build(inorder, x, pos, postorder, m, m + pos - x);
        node->right = Build(inorder, pos + 1, y, postorder, m + pos - x, n - 1);
        return node;
    }
};
```
#### [889 Construct a binary tree from preorder and postorder traversal](https://leetcode-cn.com/problems/construct-binary-tree-from-preorder-and-postorder-traversal/)

Construct a binary tree based on the results of pre-order traversal and post-order traversal.

The first element in the pre-order traversal result corresponds to the last element in the post-order traversal result, which is the value of the root node of the binary tree, and the root node is constructed from this; the second element in the pre-order traversal result must be the left child node of the root node, and the value in the post-order traversal result must be the last value on the left subtree. Then you only need to find the position of this value in the post-order traversal result, you can determine the length of the left subtree and the right subtree, and call recursively to construct a new subtree.
```c++
class Solution {
public:
    TreeNode *constructFromPrePost(vector<int> &pre, vector<int> &post) {
        return Build(pre, 0, pre.size(), post, 0, post.size());
    }

    TreeNode *Build(vector<int> &pre, int x, int y, vector<int> &post, int m, int n) {
        if (x >= y || m >= n)
            return nullptr;
        TreeNode *node = new TreeNode(pre[x]);
        if (x == y - 1)
            return node;
        int pos = m;
        while (pos < n && post[pos] != pre[x + 1])
            ++pos;
        node->left = Build(pre, x + 1, x + pos - m + 2, post, m, pos + 1);
        node->right = Build(pre, x + pos - m + 2, y, post, pos + 1, n - 1);
        return node;
    }
};
```
#### [1008 Preorder traversal to construct a binary tree](https://leetcode-cn.com/problems/construct-binary-search-tree-from-preorder-traversal/)

Given the result of a preorder traversal, construct its corresponding binary search tree.

According to the definition of preorder traversal and binary search tree, the first element of the array is the root node. All subsequent elements smaller than it are on its left subtree, and all elements larger than it are on its right subtree. Just solve it recursively.
```c++
class Solution {
public:
    TreeNode *bstFromPreorder(vector<int> &preorder) {
        return Build(preorder, 0, preorder.size());
    }

    TreeNode *Build(vector<int> &preorder, int x, int y) {
        if (x >= y)
            return nullptr;
        TreeNode *root = new TreeNode(preorder[x]);
        int i = x + 1;
        while (i < y && preorder[i] < preorder[x])
            ++i;
        root->left = Build(preorder, x + 1, i);
        root->right = Build(preorder, i, y);
        return root;
    }
};
```
#### [1028 Restore a binary tree from preorder traversal](https://leetcode-cn.com/problems/recover-a-tree-from-preorder-traversal/)

Given the result of a preorder traversal and strings connected by '-' characters of different lengths, construct the corresponding binary tree.

The length of '-' represents the current level. If the length of '-' after the current node is equal to the current level plus one, then the following number constitutes the left node of the current node; if the length of '-' after the left node is equal to the current level plus one, then the following number constitutes the right node of the current node; otherwise, if the length of '-' after the current node or after the left node is less than or equal to the same level, it means that the current node is already a leaf node, and it has no left and right child nodes, so return directly. Use a variable pos to save the currently traversed string position, and just call the process recursively.
```c++
class Solution {
public:
    TreeNode *recoverFromPreorder(string S) {
        int pos = 0;
        return Build(S, 0, pos, 0);
    }

    TreeNode *Build(const string &S, int x, int &pos, int lvl) {
        int num = 0, i = x;
        while (S[i] >= '0' && S[i] <= '9')
            ++i;
        TreeNode *node = new TreeNode(stoi(S.substr(x, i - x + 1)));
        pos = i;
        while (S[i] == '-')
            ++i;
        if (i - pos == lvl + 1)
            node->left = Build(S, i, pos, lvl + 1);
        else
            return node;
        i = pos;
        while (S[i] == '-')
            ++i;
        if (i - pos == lvl + 1)
            node->right = Build(S, i, y, pos, lvl + 1);
        return node;
    }
};
```

## Original references

- [Reference 1](https://leetcode-cn.com/tag/tree/)
