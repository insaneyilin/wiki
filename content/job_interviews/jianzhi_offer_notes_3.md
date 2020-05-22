---
title: "剑指offer笔记(3)"
date: 2019-07-13 01:34
collection: algorithm interviews
tag: notes,algorithm
---

[TOC]

[《剑指Offer》](https://book.douban.com/subject/27008702/) 读书笔记。

## 第四章 解决面试题的思路

编码前讲清楚思路。

三种辅助思考方法：

- 画图
- 举例
- 分解

### 画图让抽象问题形象化

#### 面试题19：二叉树的镜像

> 操作给定的二叉树，将其变换为源二叉树的镜像。

> 二叉树的镜像定义：
> 
>         源二叉树 
>            8
>           /  \
>          6   10
>         / \  / \
>        5  7 9  11
> 
>        镜像二叉树
>            8
>           /  \
>          10   6
>         / \  / \
>        11 9 7   5

```cpp
TreeNode* mirror(TreeNode *&root) {
    if (!root || (!root->left && !root->right)) {
        return root;
    }
    auto tmp = root->left;
    root->left = mirror(root->right);
    root->right = mirror(tmp);
    return root;
}
```

#### 面试题20：顺时针打印矩阵

> 从二维矩阵左上角开始顺时针打印每一个元素。

这题完全考察思路是否清晰，如果没有把各个循环以及边界条件想清楚就开始写代码，很容易越写越乱。

画图，分析循环结束条件。

我的思路，直接用递归，注意找边界条件：

```cpp
class Solution {
public:
    vector<int> printMatrix(vector<vector<int> > matrix) {
        vector<int> res;
        if (matrix.empty()) {
            return res;
        }
        int row = matrix.size();
        int col = matrix[0].size();
        print_mat(matrix, 0, 0, row - 1, col - 1, &res);
        return res;
    }
    
    void print_mat(vector<vector<int> > &matrix, int r, int c, int row, int col,
                  vector<int> *res) {
        if (r > row || c > col) {
            return;
        }
        // 考虑边界，只有一行/一列的情况
        if (row - r + 1 == 1) {
            for (int j = c; j <= col; ++j) {
                res->push_back(matrix[r][j]);
            }
            return;
        }
        if (col - c + 1 == 1) {
            for (int i = r; i <= row; ++i) {
                res->push_back(matrix[i][c]);
            }
            return;
        }
        for (int j = c; j <= col - 1; ++j) {
            res->push_back(matrix[r][j]);
        }
        for (int i = r; i <= row - 1; ++i) {
            res->push_back(matrix[i][col]);
        }
        for (int j = col; j > c; --j) {
            res->push_back(matrix[row][j]);
        }
        for (int i = row; i > r; --i) {
            res->push_back(matrix[i][c]);
        }
        print_mat(matrix, r + 1, c + 1, row - 1, col - 1, res);
    }
};
```

本题测试用例编写：

- 矩阵有多行多列
- 矩阵只有一行/一列

### 举例让抽象问题具体化

通过例子进行模拟，找出规律。

#### 面试题21： Min Stack 的实现

> 定义一个栈，增加一个 min 函数，能够获取栈中最小的元素；该栈的 push, pop, min 操作时间复杂度要求都是 O(1)。

排序？时间复杂度不满足。

添加一个成员变量保存最小值，每次 push 一个新元素时更新。pop 操作时如何更新？

仅仅添加一个成员变量存放最小值不够，当最小元素被 pop 出栈时，我们希望能够得到次小元素。

如果每次 push 操作都把最小元素 push 进辅助栈，那么能够保证辅助栈的栈顶一直都是最小元素。当最小元素从数据栈中被 pop 出时，我们同时对辅助栈进行 pop，那么此时辅助栈的栈顶就是下一个最小元素。

总结下实现方法：添加一个辅助栈。每次push一个新元素的时候，同时将最小元素push到辅助栈中；每次pop一个元素出栈的时候，同时pop辅助栈。

#### 面试题22：栈的压入、弹出序列

> 输入两个整数序列，第一个序列表示栈的压入顺序，请判断第二个序列是否为该栈的合法弹出顺序。假设入栈的所有数字均不相同。例子，1 2 3 4 5 是压栈序列，4 5 3 2 1 是一个合法的弹出序列，但 4 3 5 1 2 不可能是一个弹出序列

直观的想法，建立一个辅助栈，把输入的第一个序列中的数字依次压入该栈，按照第二个序列的顺序依次从该栈中弹出数字。

举例：

入栈1,2,3,4,5

出栈4,5,3,2,1

遍历压栈顺序，先将第一个放入栈中，这里是1，然后判断栈顶元素是不是出栈顺序的第一个元素，这里是4，很显然1≠4，所以我们继续压栈，直到相等以后开始出栈，出栈一个元素，则将出栈顺序向后移动一位，直到不相等，这样循环等压栈顺序遍历完成，如果辅助栈还不为空，说明弹出序列不是该栈的弹出顺序。

```cpp
class Solution {
public:
    bool IsPopOrder(vector<int> pushV,vector<int> popV) {
        vector<int> s;
        int idx_pop = 0;
        for (int i = 0; i < pushV.size(); ++i) {
            s.push_back(pushV[i]);
            // 注意 idx_pop < popV.size()
            while (idx_pop < popV.size() && popV[idx_pop] == s.back()) {
                ++idx_pop;
                s.pop_back();
            }
        }
        
        return s.empty();
    }
};
```

#### 面试题23：层次遍历二叉树

> 要求遍历结果按层存储，给出非递归和递归实现。


非递归，利用队列，将 NULL 作为层间分隔符：

```cpp
class Solution {
    /**
    * @param root: The root of binary tree.
    * @return: Level order a list of lists of integer
    */
public:
    vector<vector<int>> levelOrder(TreeNode *root) {
        // write your code here
        vector<vector<int>> result;
        if (root == NULL)
            return result;

        queue<TreeNode*> q;
        TreeNode* const SEP = NULL;
        vector<int> cur_level;

        q.push(root);
        q.push(SEP);  // level separator
        while (!q.empty())
        {
            TreeNode *node = q.front();
            q.pop();
            if (node != SEP)
            {
                cur_level.push_back(node->val);
                if (node->left)
                    q.push(node->left);
                if (node->right)
                    q.push(node->right);
            }
            else  // node == NULL means one level is DONE
            {
                result.push_back(cur_level);

                cur_level.clear();

                if (q.size() > 0)  // not the last level, add a level separator
                    q.push(SEP);
            }
        }

        return result;
    }
};
```

递归，DFS 实现，对于层次遍历反而是其递归版本不太好想到：

```cpp
class Solution {
    /**
    * @param root: The root of binary tree.
    * @return: Level order a list of lists of integer
    */
public:
    vector<vector<int>> levelOrder(TreeNode *root) {
        // write your code here
        vector<vector<int>> result;
        if (root == NULL)
            return result;
        levelOrder(root, 1, result);
        return result;
    }

private:
    void levelOrder(TreeNode *root, int level, vector<vector<int>> &result)
    {
        if (root == NULL)
            return;
        
        if (result.size() < level)
            result.push_back(vector<int>());
        result[level - 1].push_back(root->val);
        levelOrder(root->left, level + 1, result);
        levelOrder(root->right, level + 1, result);
    }
};
```

#### 面试题24：BST 的后序遍历序列

> 输入一个整数数组，判断该数组是否为某二叉搜索树的后序遍历序列。假设数组中没有相同数字。

在后序遍历序列中，最后一个数字是树的根节点。数组中前面的数字可以分为两个部分：第一部分为左子树节点的值，它们都比根节点的值小；第二部分是右子树节点的值，它们都比根节点的值大。

然后可以用相同的方法确定每一部分对应的子树的结果，这样就形成了递归。

```cpp
class Solution {
public:
    bool VerifySquenceOfBST(vector<int> sequence) {
        return verify(sequence, 0, sequence.size() - 1);
    }

    bool verify(vector<int> &sequence, int left, int right) {
        if (sequence.empty())
            return false;

        if (right - left <= 2)
            return true;

        int i = 0;
        while (sequence[i] < sequence[right] && i <= right - 1)
            ++i;

        int j = i;
        while (j <= right - 1)
        {
            if (sequence[j++] < sequence[right])
            {
                return false;
            }
        }

        return verify(sequence, left, i - 1) && verify(sequence, i, right-1);
    }
};
```

#### 面试题25：二叉树中和为某一值的路径

> 给定一个二叉树，找出所有路径中各节点相加总和等于给定 目标值 的路径。一个有效的路径，指的是从根节点到叶节点的路径。

注意本题递归实现是 DFS + 回溯，下面代码中有一些细节要注意，比如 当前路径 path 的 pop_back：

```cpp
class Solution {
public:
    /**
     * @param root the root of binary tree
     * @param target an integer
     * @return all valid paths
     */
    vector<vector<int>> binaryTreePathSum(TreeNode *root, int target) {
        // Write your code here
        vector<vector<int>> result;
        if (root == NULL)
            return result;
        
        vector<int> path;
        path_sum(root, target, path, result);
        
        return result;
    }
    
private:
    void path_sum(TreeNode *root, int target, vector<int> &path, vector<vector<int>> &result) {
        if (root == NULL)
            return;
        
        path.push_back(root->val);
        
        if (root->left == NULL && root->right == NULL) {
            if (target == root->val) {
                result.push_back(path);
                //path.clear();  // 错误，不能清空
            }
            // return; // 错误，不能提前退出，后面的 pop_back 一定要被执行才可以
        }
        
        path_sum(root->left, target - root->val, path, result);
        path_sum(root->right, target - root->val, path, result);
        path.pop_back();
    }
};
```

### 分解让复杂问题简单化

“各个击破”；分治法；先完成子功能。

#### 面试题26：复杂链表的复制

> 给出一个链表，每个节点包含一个额外增加的随机指针可以指向链表中的任何节点或空的节点。返回一个深拷贝的链表。 

```cpp
struct RandomListNode {
    int label;
    RandomListNode *next, *random;
    RandomListNode(int x) : label(x), next(NULL), random(NULL) {}
};
```

直观的想法：把复制过程分为两步：第一步是复制原始链表上每一个节点，并用 next 链接起来；第二步是设置每个节点的 random 指针。由于 random 指针可能指向当前节点的前面也可能指向后面，每个节点的 random 指针设置都要耗费 O(n) 时间，所以这种方法的时间复杂度为 O(n^2)。

上面方法的时间主要花费在 random 指针的设置上，尝试进行优化。可以利用 hash 表，保存 `<old_node, new_node>`，这样设置新链表的 random 指针时可以根据 old_node 在 O(1) 时间内找到相应的 new_node。这种方法属于用空间换时间，额外 O(n) 空间复杂度，时间复杂度为 O(n)。

能否只用 O(1) 空间和 O(n) 时间？第三种方法，第一步仍然是复制每一个节点，但是这次我们把新链表的节点直接接到旧节点的后面。

这样第二步我们设置新节点的 random 指针时只要相应的也指向原链表中节点的后一个节点即可。

最后一步是把这个长链表拆分成两个链表：奇数位置的节点连接起来是原始链表，偶数位置的节点链接起来是新链表。

上面三步中每一步都是 O(n) 操作。

- 这一题考察了复杂问题的拆解能力
- 多思考空间、时间复杂度的优化

#### 面试题27：二叉搜索树转为双向链表

> 输入一棵 BST，将该 BST 转换成一个排序后的双向链表。要求不能创建任何新的节点，只能调整树节点的指针指向。

可以令树节点的 left 指向前驱，right 指向后继。注意 BST 的中序遍历序列是有序的。

递归思路，将 BST 看成三部分：根节点、左子树、右子树。把左、右子树都转换成排序的双向链表之后再和根节点链接起来。

注意，在有指针的题目中，如果我们要改变实参指针的指向，则一定要把函数形参声明为二重指针或引用类型：`Node *&p;`

#### 面试题28：字符串的排列（回溯法）

> 输入一个字符串，打印出该字符串的所有排列。如输入 abc，输出 abc, acb, bac, bca, cab, cba

思路：把一个字符串看成两部分组成：第一部分为第一个字符，第二部分是后面的所有字符。

```cpp
class Solution {
public:
    vector<string> Permutation(string str) {
        vector<string> res;
        if (str.empty())
            return res;

        perm(str, 0, res);

        std::sort(res.begin(), res.end());
        return res;
    }

    void perm(string str, int start, vector<string> &res) {
        if (start >= str.length())
        {
            res.push_back(str);
            return;
        }

        for (int i = start; i < str.length(); ++i)
        {
            swap(str[i], str[start]);
            perm(str, start + 1, res);
            swap(str[i], str[start]);  // 注意状态撤销
        }
    }
};
```

如果有重复字符：

```cpp
class Solution {
public:
    /**
     * @param nums: A list of integers.
     * @return: A list of unique permutations.
     */
    vector<vector<int> > permuteUnique(vector<int> &nums) {
        vector<vector<int> > res;
        if (nums.empty())
            return res;
        perm(res, nums, 0);
        
        std::sort(res.begin(), res.end());
        
        
        return res;
    }
    
        void perm(vector<vector<int> > &res, vector<int> &nums, int start) {
        if (start >= nums.size()) {
            res.push_back(nums);
            return;
        }
        
        for (int i = start; i < nums.size(); ++i) {
            int flag = 0;
            for (int j = start; j < i; ++j) {
                if (nums[j] == nums[i]) {
                    flag = 1;
                    break;
                }
            }
            if (flag)
                continue;
            
            swap(nums[start], nums[i]);
            perm(res, nums, start + 1);
            swap(nums[start], nums[i]);
        }
    }
};
```

扩展题目，字符串置换：给定两个字符串，请设计一个方法来判定其中一个字符串是否为另一个字符串的置换。
这就不需要用求排列的方法了，直接对字符排序，再进行比较即可：

```cpp
class Solution {
public:
    /**
    * @param A a string
    * @param B a string
    * @return a boolean
    */
    bool stringPermutation(string& A, string& B) {
        // Write your code here
        if (A.length() != B.length())
            return false;
        sort(A.begin(), A.end());
        sort(B.begin(), B.end());
        return A == B;
    }
};
```

