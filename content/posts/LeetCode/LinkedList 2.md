---
title: "LeetCode: Linked Lists (2)"
date: 2019-07-09T19:12:25+10:00
draft: false
categories: ["LeetCode"]
description: "A translated technical note on LeetCode: Linked Lists (2), preserving the examples and context of the original article."
---
# LeetCode: Linked Lists (2)

> Originally published in Chinese on 2019-07-09; this English edition preserves the original scope and technical context.

## Title

### 4. Double pointer

#### [19 Delete the penultimate N node of the linked list](https://leetcode-cn.com/problems/remove-nth-node-from-end-of-list/submissions/)

Delete the nth node from the last in the linked list.

It is not easy to directly access the nth position from the bottom in the linked list, so two pointers prev and tail are used. The tail goes forward n steps first, and then the two pointers go forward together until tail has no successor pointer. At this time, the successor pointer of prev is the nth position from the bottom, just delete it. Note that if the pointer to be deleted is the head pointer, it must be processed separately.
```c++
class Solution {
public:
    ListNode* removeNthFromEnd(ListNode* head, int n) {
        ListNode *prev = head, *tail = head;
        for (int i = 0; i < n; ++i)
            tail = tail->next;
        if (!tail) {
            head = head->next;
            delete prev;
            return head;
        }
        while (tail->next)
            tail = tail->next, prev = prev->next;
        ListNode *next = prev->next;
        prev->next = next->next;
        delete next;
        return head;
    }
};
```
#### [61 Rotate Linked List](https://leetcode-cn.com/problems/rotate-list/submissions/)

Given a linked list, move each node k positions to the right.

It is not easy to directly access the first k positions in the linked list, so two pointers are used. The first one goes forward k steps first, and then the two pointers go forward together until the first pointer has no successor pointer. Then the head node can be linked to the first pointer and the second pointer will be set to null. Note that k may be very large. You must first calculate the length of the linked list len ​​and then use k % len to calculate it.
```c++
class Solution {
public:
    ListNode *rotateRight(ListNode *head, int k) {
        if (!head)
            return nullptr;
        ListNode *first = head, *second = head, *traverse = head;
        int len = 0;
        while (traverse) {
            traverse = traverse->next;
            ++len;
        }
        k %= len;
        for (int i = 0; i < k; i++)
            first = first->next;
        while (first && first->next)
            first = first->next, second = second->next;
        first->next = head;
        auto ret = second->next;
        second->next = nullptr;
        return ret;
    }
};
```
#### [876 Middle node of the linked list](https://leetcode-cn.com/problems/middle-of-the-linked-list/solution/lian-biao-de-zhong-jian-jie-dian-by-leetcode/)

Find the middle node of the linked list.

Use two pointers, slow and fast. Fast moves two steps at a time, and slow moves one step at a time. When fast reaches the end, slow reaches the middle of the pointer.
```c++
class Solution {
public:
    ListNode* middleNode(ListNode* head) {
        auto slow = head, fast = head;
        while (fast && fast->next) {
            slow = slow->next;
            fast = fast->next->next;
        }
        return slow;
    }
};
```
#### [160 Intersection linked lists](https://leetcode-cn.com/problems/intersection-of-two-linked-lists/submissions/)

Find the starting node where two linked lists intersect.

Assume that the non-intersecting length of linked list A is l1, the unintersecting length of linked list B is l2, and the length of the intersecting part is lc. In order to make the two pointers travel the same length, you only need to let the two pointers return to the head of the other linked list when they reach the end and continue walking. Finally, when the lengths traveled by both pointers are both l1 + l2 + l3, they intersect.
```c++
class Solution {
public:
    ListNode *getIntersectionNode(ListNode *headA, ListNode *headB) {
        auto a = headA, b = headB;
        if (!a || !b)
            return nullptr;
        while (a != b) {
            a = a ? a->next : headB;
            b = b ? b->next : headA;
        }
        return a;
    }
};
```
#### [141 Linked List](https://leetcode-cn.com/problems/linked-list-cycle/)

Determine whether there is a cycle in a linked list.

Use two pointers, slow and fast, fast to move two steps at a time, and slow to move one step at a time. If there is a loop in the linked list, the two pointers will eventually meet, otherwise fast will reach the end first. When fast == nullptr or fast->next == nullptr, it means that the linked list has not changed and fast has reached the end.
```c++
class Solution {
public:
    bool hasCycle(ListNode *head) {
        auto slow = head, fast = head;
        while (fast && fast->next) {
            slow = slow->next;
            fast = fast->next->next;
            if (slow == fast)
                return true;
        }
        return false;
    }
};
```
#### [142 Linked List II](https://leetcode-cn.com/problems/linked-list-cycle-ii/)

Given a linked list, find the first node in the linked list.

Same as the previous question, use two pointers slow and fast, fast moves two steps at a time, and slow moves one step at a time. If there is a loop in the linked list, the two pointers will eventually meet. At this time, the distance moved by the fast pointer is l1 + l2 + c, where l1 is the length of the outer part of the loop in the linked list, l2 is the length of the common part walked by the two pointers in the loop, c is the length of the loop, and the distance moved by the slow pointer is l1 + l2, because the moving speed of the fast pointer is Twice of slow, so l1 + l2 + c = 2 * (l1 + l2), so c = l1 + l2, and because the slow pointer has already walked through the part of length l2 in the ring, leaving only the part of length l1, you only need to use a new pointer to move one step at a time from the head of the linked list, and let the slow pointer move one step at a time. Eventually, they will move a distance of l1 and meet at the entrance of the ring.
```c++
class Solution {
public:
    ListNode *detectCycle(ListNode *head) {
        if (!head || !head->next)
            return nullptr;
        auto slow = head->next, fast = head->next->next;
        while (fast && fast->next && fast != slow) {
            fast = fast->next->next;
            slow = slow->next;
        }
        if (!fast || !fast->next)
            return nullptr;
        auto lin = head;
        while (lin != slow)
            lin = lin->next, slow = slow->next;
        return lin;
    }
};
```
### 5. Comprehensive

#### [234 Palindrome Linked List](https://leetcode-cn.com/problems/palindrome-linked-list/submissions/)

Determine whether a linked list is a palindrome linked list.

First find the middle node of the linked list, then flip the right linked list, truncate the left linked list from the middle, and compare again whether each node on both sides is equal.
```c++
class Solution {
public:
    bool isPalindrome(ListNode *head) {
        if (!head || !head->next)
            return true;
        auto prev = head, slow = head, fast = head;
        while (fast && fast->next) {
            prev = slow;
            slow = slow->next;
            fast = fast->next->next;
        }
        auto node = Reverse(fast ? slow->next : slow);
        prev->next = nullptr;
        while (head && node && head->val == node->val)
            head = head->next, node = node->next;
        return !head && !node;
    }

    ListNode *Reverse(ListNode *head) {
        if (!head)
            return head;
        ListNode *prev = nullptr, *curr = head, *next = head->next;
        while(curr) {
            next = curr->next;
            curr->next = prev;
            prev = curr;
            curr = next;
        }
        return prev;
    }
};
```
#### [109 Convert Sorted List to Binary Search Tree](https://leetcode-cn.com/problems/convert-sorted-list-to-binary-search-tree/)

Given an ordered linked list, convert it into a balanced binary search tree.

First find the middle node of the linked list, use it as the root node of the tree, use the left part of the linked list of the middle node as its left subtree, and the right part of the linked list as its right subtree.
```c++
class Solution {
public:
    TreeNode *sortedListToBST(ListNode *head) {
        if (!head)
            return nullptr;
        ListNode *prev = nullptr, *slow = head, *fast = head;
        while (fast && fast->next)
            prev = slow, slow = slow->next, fast = fast->next->next;
        if (prev)
            prev->next = nullptr;
        auto root = new TreeNode(slow->val);
        root->left = sortedListToBST(prev ? head : nullptr);
        root->right = sortedListToBST(slow->next);
        return root;
    }
};
```
#### [426 Convert binary search tree to sorted doubly linked list](https://leetcode-cn.com/problems/convert-binary-search-tree-to-sorted-doubly-linked-list/submissions/)

Convert a binary search tree into a doubly circular linked list.

For a root node, the predecessor node after it is converted into a doubly linked list should be the node with the largest value in its left subtree, that is, the rightmost child node of its left child node. Therefore, you only need to find this node first, and then connect the root node to it end to end. Before that, you should first perform operations on the left subtree of the root node, and the same goes for the right subtree. In addition, the smallest and largest nodes must be saved and connected end to end to form a circular linked list.
```c++
class Solution {
    Node *first, *last;
public:
    Node *treeToDoublyList(Node *root) {
        first = last = nullptr;
        TreeToList(root);
        if (first)
            first->left = last;
        if (last)
            last->right = first;
        return first;
    }

    void TreeToList(Node *root) {
        if (!root)
            return;
        if (!first || first->val > root->val)
            first = root;
        if (!last || last->val < root->val)
            last = root;
        auto left = root->left, right = root->right;
        TreeToList(left);
        TreeToList(right);
        while (left && left->right)
            left = left->right;
        if (left)
            left->right = root;
        root->left = left;
        while (right && right->left)
            right = right->left;
        if (right)
            right->left = root;
        root->right = right;
    }
};
```
#### [143 Reorder linked list](https://leetcode-cn.com/problems/reorder-list/submissions/)

Rearrange a linked list from L0→L1→…→Ln-1→Ln to L0→Ln→L1→Ln-1→L2→Ln-2→….

First divide the linked list into two parts from the middle, reverse the second half, and then link the nodes at the corresponding positions in sequence.
```c++
class Solution {
public:
    void reorderList(ListNode* head) {
        if (!head)
            return;
        auto prev = head, slow = head, fast = head, first = head, second = head;
        while (fast && fast->next)
            prev = slow, slow = slow->next, fast = fast->next->next;
        if (fast) {
            second = slow->next;
            slow->next = nullptr;
        } else {
            second = slow;
            prev->next = nullptr;
        }
        second = Reverse(second);
        while (second) {
            auto n1 = first->next, n2 = second->next;
            first->next = second;
            second->next = n1;
            first = n1;
            second = n2;
        }
    }

    ListNode *Reverse(ListNode *head) {
        ListNode *prev = nullptr, *curr = head, *next = nullptr;
        while (curr) {
            next = curr->next;
            curr->next = prev;
            prev = curr;
            curr = next;
        }
        return prev;
    }
};
```
#### [138 Copy linked list with random pointer](https://leetcode-cn.com/problems/copy-list-with-random-pointer/submissions/)

Given a linked list, each node contains a random pointer pointing to a node in the linked list, and returns a deep copy of the linked list.

Because the node pointed by the random pointer may not have been copied during the first traversal, you can copy all the nodes first, use a hash table to save the corresponding relationships of the nodes, and then traverse again to link the nodes pointed by the random pointer in sequence. The space complexity of this is O(n). You can also copy each node once, link the copied node to the node, and then traverse twice to modify the node pointed by the random pointer and separate the original linked list and its copy. The space complexity of doing so is O(1).
```c++
class Solution {
public:
    Node *copyRandomList(Node *head) {
        if (!head)
            return head;
        Node *node = head;
        while (node) {
            Node *cop = new Node(node->val, node->next, node->random);
            node->next = cop;
            node = node->next->next;
        }
        node = head;
        while (node && node->next) {
            if (node->random)
                node->next->random = node->random->next;
            node = node->next->next;
        }
        node = head;
        Node *res = head->next;
        while (node) {
            Node *next = node->next;
            node->next = next->next;
            next->next = next->next ? next->next->next : nullptr;
            node = node->next;
        }
        return res;
    }
};
```
#### [23 Merge K sorted linked lists](https://leetcode-cn.com/problems/merge-k-sorted-lists/submissions/)

Merge k ordered linked lists.

The simplest way is to traverse all the head nodes each time, take out the one with the smallest value, and add it to the linked list to be returned. The time complexity is O(m * n), where m is the number of linked lists, n is the total number of nodes, and the space complexity is O(1).
```c++
class Solution {
public:
    ListNode *mergeKLists(vector<ListNode *> &lists) {
        ListNode *root = new ListNode(0), *node = root;
        while (node) {
            ListNode *min_ptr = nullptr;
            for (auto &l :lists)
                if (l && (!min_ptr || l->val < min_ptr->val))
                    min_ptr = l;
            node->next = min_ptr;
            for (auto &l :lists)
                if (l && l == min_ptr)
                    l = l->next;
            node = node->next;
        }
        return root->next;
    }
};
```
On this basis, a small root heap can be used to save all head nodes. Each time the node at the top of the heap is taken out, the node is added to the linked list to be returned, and it is judged whether the node has a successor node. If so, it is added to the heap and maintained. Note that the comparison function of the priority queue needs to be overloaded. The time complexity is O(n * logm) and the space complexity is O(m).
```c++
class Solution {
    struct compare {
        bool operator()(ListNode *l1, ListNode *l2) {
            return l1->val > l2->val;
        }
    };

public:
    ListNode *mergeKLists(vector<ListNode *> &lists) {
        ListNode *root = new ListNode(0), *node = root;
        priority_queue<ListNode *, vector<ListNode *>, compare> heap;
        for (auto &l:lists)
            if (l)
                heap.push(l);
        while (!heap.empty()) {
            ListNode *curr = heap.top();
            node->next = curr;
            node = node->next;
            heap.pop();
            if (curr->next)
                heap.push(curr->next);
        }
        return root->next;
    }
};
```
You can also use the divide-and-conquer method to merge linked lists in pairs, saving the space and time spent on heap maintenance. Because the number of linked lists is m, it takes logm time to merge all linked lists using the divide-and-conquer method. The time complexity is O(n * logm) and the space complexity is O(1).
```c++
class Solution {
    ListNode *MergeLists(ListNode *l1, ListNode *l2) {
        ListNode *root = new ListNode(0), *node = root;
        while (l1 && l2) {
            if (l1->val < l2->val) {
                node->next = l1;
                l1 = l1->next;
            } else {
                node->next = l2;
                l2 = l2->next;
            }
            node = node->next;
        }
        node->next = l1 ? l1 : l2;
        return root->next;
    }

public:
    ListNode *mergeKLists(vector<ListNode *> &lists) {
        int n = lists.size();
        for (int interval = 1; interval < n; interval *= 2) {
            for (int i = 0; i < n; i += interval * 2)
                if (i + interval < n)
                    lists[i] = MergeLists(lists[i], lists[i + interval]);
        }
        return n == 0 ? nullptr : lists[0];
    }
};
```

## Original references

- [Reference 1](https://leetcode-cn.com/tag/linked-list/)
