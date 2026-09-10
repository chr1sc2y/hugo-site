---
title: "LeetCode: Linked Lists (1)"
date: 2019-07-04T19:12:25+10:00
draft: false
categories: ["LeetCode"]
description: "A translated technical note on LeetCode: Linked Lists (1), preserving the examples and context of the original article."
---
# LeetCode: Linked Lists (1)

> Originally published in Chinese on 2019-07-04; this English edition preserves the original scope and technical context.

## Title

### 1. General questions

#### [2 Add two numbers](https://leetcode-cn.com/problems/add-two-numbers/)

Given two linked lists respectively representing the reverse order of two positive numbers, calculate the sum of the two linked lists.

Add them bit by bit.
```c++
class Solution {
public:
    ListNode *addTwoNumbers(ListNode *l1, ListNode *l2) {
        int acc = 0, val = 0;
        auto head = l1, tail = l1;
        while (l1 && l2) {
            val = l1->val + l2->val + acc;
            acc = val / 10;
            l1->val = val % 10;
            if (!l1->next)
                l1->next = l2->next, l2->next = nullptr;
            tail = l1;
            l1 = l1->next, l2 = l2->next;
        }
        while (l1) {
            val = l1->val + acc;
            acc = val / 10;
            l1->val = val % 10;
            tail = l1;
            l1 = l1->next;
        }
        if (acc)
            tail->next = new ListNode(1);
        return head;
    }
};
```
#### [21 Merge two ordered linked lists](https://leetcode-cn.com/problems/merge-two-sorted-lists/submissions/)

Merge two sorted linked lists.

Compare the size one by one and add it to the back of the current node, and move the corresponding linked list node.
```c++
class Solution {
public:
    ListNode* mergeTwoLists(ListNode* l1, ListNode* l2) {
        ListNode *head = new ListNode(0), *node = head;
        while (l1 && l2) {
            if (l1->val < l2->val)
                head->next = l1, l1 = l1->next;
            else
                head->next = l2, l2 = l2->next;
            head = head->next;
        }
        while (l1)
            head->next = l1, l1 = l1->next, head = head->next;
        while (l2)
            head->next = l2, l2 = l2->next, head = head->next;
        return node->next;
    }
};
```
#### [83 Remove duplicate elements from sorted linked list](https://leetcode-cn.com/problems/remove-duplicates-from-sorted-list/submissions/)

Remove all duplicate nodes from the linked list.

Compare the value of each node with the value of the following node, and delete the following node if they are the same.
```c++
class Solution {
public:
    ListNode* deleteDuplicates(ListNode* head) {
        auto ret = head;
        while (head) {
            while (head->next && head->next->val == head->val) {
                auto next = head->next;
                head->next = next->next;
                delete next;
            }
            head = head->next;
        }
        return ret;
    }
};
```
#### [82 Remove duplicate elements from sorted list II](https://leetcode-cn.com/problems/remove-duplicates-from-sorted-list-ii/submissions/)

Delete all duplicate nodes in the linked list and keep only the numbers that do not appear repeatedly in the original linked list.

In order to delete all duplicate nodes and retain only all numbers that have not appeared before, you need to check two nodes in advance to see if the values ​​of the next two nodes are the same. If they are the same, you need to delete these two nodes and all duplicate nodes after them.
```c++
class Solution {
public:
    ListNode* deleteDuplicates(ListNode* head) {
        ListNode *node = new ListNode(0), *ret = node, *next = nullptr, *temp = nullptr;
        node->next = head;
        while (node && node->next) {
            next = node->next->next;
            while (next && node->next->val == next->val) {
                temp = next;
                next = next->next;
                delete temp;
            }
            if (next != node->next->next) {
                temp = node->next;
                node->next = next;
                delete temp;
            } else
                node = node->next;
        }
        return ret->next;
    }
};
```
#### [203 Remove linked list elements](https://leetcode-cn.com/problems/remove-linked-list-elements/submissions/)

Delete all nodes in the linked list that are equal to the given value.

First determine whether the head node is equal to the given value, and then determine whether the following nodes are equal to the given value.
```c++
class Solution {
public:
    ListNode* removeElements(ListNode* head, int val) {
        while (head && head->val == val) {
            auto prev = head;
            head = head->next;
            delete prev;
        }
        auto ret = head;
        while (head) {
            while (head->next && head->next->val == val) {
                auto next = head->next;
                head->next = next->next;
                delete next;
            }
            head = head->next;
        }
        return ret;
    }
};
```
#### [817 Linked List Components](https://leetcode-cn.com/problems/linked-list-components/submissions/)

Given a linked list and an array, find the number of sub-lists in the linked list whose values are all in the array.

First convert the array into a hash table for easy query, and then traverse the entire linked list in order to determine whether the sub-linked list meets the conditions at the end of a sub-linked list or at the end of the traversal.
```c++
class Solution {
public:
    int numComponents(ListNode* head, vector<int>& G) {
        unordered_set<int> exist(G.begin(), G.end());
        bool cont = false, curr = false;
        int res = 0;
        while (head) {
            curr = exist.count(head->val);
            res += cont && !curr;
            cont = curr;
            head = head->next;
        }
        return res + cont;
    }
};
```
#### [24 Pairwise exchange of nodes in the linked list](https://leetcode-cn.com/problems/swap-nodes-in-pairs/)

Given a linked list, return the result of exchanging adjacent nodes in pairs.

Use three pointers to save the two nodes to be exchanged and their predecessor nodes. After the exchange, update the three pointers and exchange them in order.
```c++
class Solution {
public:
    ListNode *swapPairs(ListNode *head) {
        if (!head || !head->next)
            return head;
        ListNode *root = new ListNode(0), *prev = root;
        root->next = head;
        auto first = head, second = head->next;
        while (first && second) {
            auto temp = second->next;
            prev->next = second;
            second->next = first;
            first->next = temp;
            prev = first;
            first = temp;
            second = temp ? temp->next : nullptr;
        }
        return root->next;
    }
};
```
#### [430 Flattened multilevel doubly linked list](https://leetcode-cn.com/problems/flatten-a-multilevel-doubly-linked-list/submissions/)

Given a doubly linked list with child nodes, flatten it so that all nodes appear in a single-level doubly linked list.

For a certain node, if it has a child node, then it needs to use its child node as its new next node, use the last node on its child linked list as the prev node of its original next node, and call the entire process recursively.
```c++
class Solution {
public:
    Node *flatten(Node *head) {
        if (!head)
            return nullptr;
        Node *node = head, *next = nullptr;
        while (node) {
            if (node->child) {
                next = node->next;
                auto child = flatten(node->child);
                node->next = child;
                child->prev = node;
                node->child = nullptr;
                while (node->next)
                    node = node->next;
                node->next = next;
                if (next)
                    next->prev = node;
            }
            node = node->next;
        }
        return head;
    }
};
```
### 2. Linked list reversal

#### [206 Reverse linked list](https://leetcode-cn.com/problems/reverse-linked-list/submissions/)
```c++
class Solution {
public:
    ListNode* reverseList(ListNode* head) {
        if (!head)
            return nullptr;
        ListNode *prev = nullptr, *curr = head, *next = head->next;
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
#### [92 Reverse linked list II](https://leetcode-cn.com/problems/reverse-linked-list-ii/submissions/)

Reverse the nodes at position m to n in the linked list.

First find the node at m - 1, truncate the nodes after it, then find the node at n, truncate the nodes after it, invert the middle section of the linked list and then link it to the wish list.
```c++
class Solution {
public:
    ListNode *reverseBetween(ListNode *head, int m, int n) {
        ListNode *root = new ListNode(0), *r = root, *prev, *last;
        root->next = head;
        int cnt = 1;
        while (cnt < m && r)
            r = r->next, ++cnt;
        prev = r;
        while (cnt <= n && r && r->next)
            r = r->next, ++cnt;
        last = r->next;
        r->next = nullptr;
        prev->next = Reverse(prev->next);
        while (prev && prev->next)
            prev = prev->next;
        prev->next = last;
        return root->next;
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
#### [369 Add one to singly linked list](https://leetcode-cn.com/problems/plus-one-linked-list/submissions/)

Use a singly linked list to represent an integer and calculate the result of adding one to it.

First reverse the linked list to facilitate carry, then perform the operations of adding one and carrying, and finally reverse the linked list again.
```c++
class Solution {
public:
    ListNode* plusOne(ListNode* head) {
        if (!head)
            return nullptr;
        head = Reverse(head);
        auto root = head;
        while (head) {
            if (head->val == 9) {
                head->val = 0;
                if (!head->next) {
                    head->next = new ListNode(1);
                    break;
                }
                head = head->next;
            }
            else {
                ++head->val;
                break;
            }
        }
        return Reverse(root);
    }

    ListNode *Reverse(ListNode* head) {
        if (!head)
            return nullptr;
        ListNode *prev = nullptr, *curr = head, *next = head->next;
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
#### [445 Add Two Numbers II](https://leetcode-cn.com/problems/add-two-numbers-ii/submissions/)

Given two linked lists representing two positive numbers respectively, calculate the sum of the two linked lists.

First reverse the two linked lists, then add them bitwise, and finally reverse the result and return it.
```c++
class Solution {
public:
    ListNode *addTwoNumbers(ListNode *l1, ListNode *l2) {
        if (!l1 || !l2)
            return !l1 ? l2 : l1;
        l1 = Reverse(l1);
        l2 = Reverse(l2);
        int carry = 0, sum = 0;
        ListNode *head = l1, *prev = l1;
        while (l1 && l2) {
            prev = l1;
            sum = l1->val + l2->val + carry;
            carry = sum / 10;
            l1->val = sum - carry * 10;
            if (l2->next && !l1->next)
                l1->next = l2->next, l2->next = nullptr;
            l1 = l1->next;
            l2 = l2->next;
        }
        while (l1) {
            prev = l1;
            sum = l1->val + carry;
            carry = sum / 10;
            l1->val = sum - carry * 10;
            l1 = l1->next;
        }
        if (carry && prev)
            prev->next = new ListNode(1);
        return Reverse(head);
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
#### [25 K reverse linked list in a group](https://leetcode-cn.com/problems/reverse-nodes-in-k-group/)

Given a linked list, flip each group of k nodes.

Use several pointers to record the starting and ending positions of the parts that need to be reversed, and then reverse them in sequence.
```c++
class Solution {
    ListNode *ReverseLinkedList(ListNode *head) {
        ListNode *prev = nullptr, *curr = head, *next = nullptr;
        while (curr) {
            next = curr->next;
            curr->next = prev;
            prev = curr;
            curr = next;
        }
        return prev;
    }

public:
    ListNode *reverseKGroup(ListNode *head, int k) {
        ListNode *root = new ListNode(0), *prev = root;
        prev->next = head;
        while (prev) {
            int i = 1;
            ListNode *start = prev->next, *end = prev->next, *curr = end, *next = nullptr;
            while (curr && i < k)
                curr = curr->next, ++i;
            if (i < k || !curr)
                break;
            next = curr->next;
            curr->next = nullptr;
            start = ReverseLinkedList(start);
            prev->next = start;
            end->next = next;
            prev = end;
        }
        return root->next;
    }
};
```
### 3. Double linked list

#### [328 odd-even linked list](https://leetcode-cn.com/problems/odd-even-linked-list/submissions/)

Arrange the odd-numbered nodes and even-numbered nodes in a linked list together.

Use two head nodes to represent the starting positions of the odd-numbered and even-numbered nodes respectively. Traverse the entire linked list, link the odd-numbered node after the odd-numbered starting node, link the even-numbered node after the even-numbered starting node, and finally link the even-numbered starting node after the last odd-numbered node.
```c++
class Solution {
public:
    ListNode* oddEvenList(ListNode* head) {
        if (!head || !head->next)
            return head;
        auto odd = head, even = head->next, node = even->next, even_head = even;
        bool flag = true;
        while (node) {
            if (flag) {
                odd->next = node;
                odd = odd->next;
                flag = false;
            } else {
                even->next = node;
                even = even->next;
                flag = true;
            }
            node = node->next;
        }
        odd->next = even_head;
        even->next = nullptr;
        return head;
    }
};
```
#### [86 separated linked list](https://leetcode-cn.com/problems/partition-list/)

Given a linked list and a value x, rearrange the linked list so that all nodes less than x come before nodes greater than or equal to x.

Use two head nodes sth and geq to represent the starting positions of nodes less than x and greater than or equal to x respectively. Traverse the entire linked list, link each node to the two head nodes, and finally link geq to sth, and then set the end of geq to a null pointer.
```c++
class Solution {
public:
    ListNode* partition(ListNode* head, int x) {
        auto sth = new ListNode(0), sth_head = sth, geq = new ListNode(0), geq_head = geq;
        while (head) {
            if (head->val < x) {
                sth->next = head;
                sth = sth->next;
            } else {
                geq->next = head;
                geq = geq->next;
            }
            head = head->next;
        }
        sth->next = geq_head->next;
        geq->next = nullptr;
        return sth_head->next;
    }
};
```
#### [725 split linked list](https://leetcode-cn.com/problems/split-linked-list-in-parts/submissions/)

Given a linked list, divide it into k consecutive parts.

First calculate the length of the linked list and the length of each of the k consecutive parts, and put the head node of each part into the array in turn.
```c++
class Solution {
public:
    vector<ListNode *> splitListToParts(ListNode *root, int k) {
        vector<ListNode *> res(k, nullptr);
        int len = 0;
        ListNode *head = root, *temp = nullptr;
        while (root)
            root = root->next, ++len;
        int n = len / k, m = len % k, i = 0;
        while (head) {
            res[i] = head;
            for (int j = 0; j < n + (m > 0) - 1; ++j)
                head = head->next;
            temp = head;
            head = head->next;
            temp->next = nullptr;
            ++i;
            --m;
        }
        return res;
    }
};
```

## Original references

- [Reference 1](https://leetcode-cn.com/tag/linked-list/)
