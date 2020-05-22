---
title: "剑指offer笔记(2)"
date: 2019-07-13 01:34
collection: algorithm interviews
tag: notes,algorithm
---

[TOC]

[《剑指Offer》](https://book.douban.com/subject/27008702/) 读书笔记。

## 第三章 高质量的代码

规范的代码：

- 清晰的书写
- 清晰的布局
- 合理的命名

代码的完整性：

- 功能测试，注意一些功能要求中没有明说但可能隐含的情况（如大数运算，负数等）
- 边界测试，递归、循环的终止条件，输入极值等
- 负面测试，错误的输入

错误处理方法：

- 函数返回值
- 设置全局变量
- 异常

#### 面试题11：数值的整数次方

> 实现 pow 函数，不需要考虑大数问题。

Naive 的解法：

```cpp
double Power(double base, int exponent) {
  double result = 1.0;
  for (int i = 1; i <= exponent; ++i) {
    result *= base;
  }
  return result;
}
```

上面代码存在的问题：

- exponent 小于 1 时怎么办？

当指数为负数时，可以先对指数求绝对值，然后算出次方的结果，再取倒数。既然有倒数，就要考虑有没有可能对 0 取倒数，如果出现对 0 取倒数（base 为 0 而 exponent 为 负）的情况，抛出异常。此外，还有一个边界条件是 base 和 exponent 都为 0，此时数学上无意义，向面试官说明即可。

上面的代码已经考虑的比较全面了，但还有一个问题：效率。

\begin{equation}
a^n =
\begin{cases}
a^{n/2} * a^{n/2}, \quad \ & n 为偶数, \\\
a^{(n-1)/2} * a^{(n-1)/2} * a, \quad \ \ & n 为奇数,
\end{cases}
\end{equation}

```cpp
double power_with_unsigned_exponent(double base, unsigned int exponent)
{
    if (exponent == 0)
        return 1.0;
    if (exponent == 1)
        return base;

    double result = power_with_unsigned_exponent(base, exponent >> 1);
    result *= result;
    if (exponent % 2 == 1)
        result *= base;

    return result;
}
```

#### 面试题12：打印 1 到最大的 n 位数

> 输入数字 n，按顺序打印从 1 到最大的 n 位整数。
> 例子，输入 3，打印从 1 到 999。

陷阱：n 有可能非常大，需要考虑大数。

解法1：模拟大数自增运算。

解法2：把问题转换成数字排列，使用递归。

#### 面试题13：在 O(1) 时间删除链表节点

> 给定一个单链表的头指针和一个节点指针，定义一个函数在 O(1) 时间删除该节点。

要删除节点 i，需要从链表头节点开始遍历，找到节点 i 的前一个节点。这样的方法时间复杂度为 O(n)。

另一种方法：既然给出了要删除节点的指针，可以直接得到该节点的下一个节点，把下一个节点的内容复制到要删除的节点里覆盖掉原来的内容，删除下一个节点即可，这样时间复杂度为 O(1)。一个特殊情况，如果要删除的节点是尾节点，仍然需要从链表头开始遍历。因此该方法平均时间复杂度为 O(1)，最坏情况时间复杂度为 O(n)。

```cpp
class Solution {
public:
    /**
     * @param node: a node in the list should be deleted
     * @return: 假设 node 非表头或表尾
     */
    void deleteNode(ListNode *node) {
        // write your code here
        ListNode *t = node->next;
        node->next = t->next;
        node->val = t->val;
        delete t;
    }
};
```

#### 面试题14：奇偶分割数组

> 分割一个整数数组，使得所有奇数在所有偶数前面。

类似 std::partition()，注意是否要求 stable。

非 stable 版本（类似 quick sort）:

```cpp
class Solution {
 public:
  void ReorderArray(vector<int> &array) {
    int l = 0;
    int r = array.size() - 1;
    while (l < r) {
      while (l < r && array[l] % 2 == 1) {
        ++l;
      }
      while (l < r && array[r] % 2 == 0) {
        --r;
      }
      if (l >= r) {
        break;
      }
      swap(array[l], array[r]);
    }
  }
};
```

stable 版本：

```cpp
class Solution {
public:
    void ReorderArray(vector<int> &nums) {
        for (int i = 0; i < nums.size(); ++i)
        {
            for (int j = nums.size() - 1; j > i; --j)
            {
                if (nums[j] % 2 == 1 && nums[j - 1] % 2 == 0)
                {
                    swap(nums[j], nums[j - 1]);
                }
            }
        }
    }
};
```

stable 版本复杂度 O(n^2)，如果允许使用额外空间，可以做到 O(n)。

#### 面试题15：链表中倒数第 k 个节点

为了得到第 k 个节点，最自然的想法是先走到链表的尾部，再从尾部回溯 k 步，然而单链表不允许回溯。

设置两个指针，第一个指针比第二个指针先走 k 步。当第一个指针走到尾部时，第二个指针指向的就是倒数第 k 个节点。

很容易写出以下代码：

```cpp
ListNode* find_kth_node(ListNode *head, int k) {
    ListNode *p1 = head;
    ListNode *p2 = head;

    for (int i = 0; i < k; ++i) {
        p1 = p1->next;
    }

    while (p1 != NULL) {
        p1 = p1->next;
        p2 = p2->next;
    }

    return p2;
}
```

上面的代码不鲁棒：

- 输入的为空指针怎么办？
- 输入的 k 不大于 0 怎么办？
- k 大于链表中节点总个数怎么办？

考虑特殊情况后的代码：

```cpp
ListNode* find_kth_node(ListNode *head, int k) {
    if (head == NULL || k <= 0)
        return NULL;

    ListNode *p1 = head;
    ListNode *p2 = head;

    for (int i = 0; i < k; ++i) {
        p1 = p1->next;
        if (p1 == NULL && i < k - 1) {
            return NULL;
        }
    }
    while (p1 != NULL) {
        p1 = p1->next;
        p2 = p2->next;
    }
    return p2;
}
```

#### 面试题16：翻转链表

非常经典。先画一个只有 4 个节点的 list 模拟下，找出规律。

```
class Solution {
public:
    /**
     * @param head: The first node of linked list.
     * @return: The new head of reversed linked list.
     */
    ListNode *reverse(ListNode *head) {
        // write your code here
        if (head == NULL || head->next == NULL)
            return head;

        ListNode *new_head = NULL;
        ListNode *p = head;
        while (p != NULL) {
            ListNode *t = p->next;
            p->next = new_head;
            new_head = p;
            p = t;
        }
        return new_head;
    }
};
```

可能出错的情况：

- 输入的链表头为 NULL 或者只有一个节点时
- 反转后的链表出现断裂
- 返回的翻转后的链表的头节点不是原链表的尾节点

测试用例设计：

- 输入的链表头为 NULL
- 输入的链表只有一个节点
- 输入的链表有多个节点

#### 面试题17：合并两个有序链表

考察基本功，可以用 dummy head 技巧简化选取头结点：

```cpp
class Solution {
public:
    /**
     * @param ListNode l1 is the head of the linked list
     * @param ListNode l2 is the head of the linked list
     * @return: ListNode head of linked list
     */
    ListNode *mergeTwoLists(ListNode *l1, ListNode *l2) {
        // write your code here
        if (l1 == NULL)
            return l2;
        if (l2 == NULL)
            return l1;
            
        ListNode dummy(-1);
        ListNode *p = &dummy;
        while (l1 != NULL && l2 != NULL) {
            if (l1->val < l2->val) {
                p->next = l1;
                l1 = l1->next;
            } else {
                p->next = l2;
                l2 = l2->next;
            }
            p = p->next;
        }
        
        if (l1 != NULL)
            p->next = l1;
        else if (l2 != NULL)
            p->next = l2;
        
        return dummy.next;
    }
};
```

#### 面试题18：树的子结构

> 输入两棵二叉树 A 和 B，判断 B 是否为 A 的子结构

第一步在树 A 中找到和 B 的根节点的值一样的节点 R，第二步再判断树 A 中以 R 为根节点的子树是否包含和树 B 一样的结构。

下面是判断“子树”的解法：

```cpp
class Solution {
public:
    /**
     * @param T1, T2: The roots of binary tree.
     * @return: True if T2 is a subtree of T1, or false.
     */
    bool isSubtree(TreeNode *T1, TreeNode *T2) {
        // write your code here
        if (T2 == NULL)
            return true;
        if (T1 == NULL)
            return false;
        
        bool result = false;
        
        if (T1->val == T2->val) {
            result = is_same_tree(T1, T2);
            // result = tree1_contains_tree2(T1, T2);
        }
            
        if (!result)
            result = isSubtree(T1->left, T2);
        if (!result)
            result = isSubtree(T1->right, T2);

        return result;
    }
    
private:
    // 相同子结构
    bool tree1_contains_tree2(TreeNode *T1, TreeNode *T2) {
        if (T2 == NULL)
            return true;
        if (T1 == NULL)
            return false;
        if (T1->val != T2->val)
            return false;
        
        return tree1_contains_tree2(T1->left, T2->left) && 
            tree1_contains_tree2(T1->right, T2->right);
    }
    
    // 相同子树
    bool is_same_tree(TreeNode *T1, TreeNode *T2) {
        if (T1 == NULL && T2 == NULL)
            return true;
        
        if (T1 == NULL && T2 != NULL ||
            T1 != NULL && T2 == NULL)
            return false;
        
        if (T1->val != T2->val)
            return false;
        
        return is_same_tree(T1->left, T2->left) && 
            is_same_tree(T1->right, T2->right);
    }
};
```
