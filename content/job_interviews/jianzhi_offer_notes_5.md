---
title: "剑指offer笔记(5)"
date: 2019-07-13 01:34
collection: algorithm interviews
tag: notes,algorithm
---

[TOC]

[《剑指Offer》](https://book.douban.com/subject/27008702/) 读书笔记。

## 第 6 章 面试中的各种能力

- 沟通能力。题目的细节，主动提问。
- 表达能力。
- 学习能力。
- 应变能力。

面试是一个双向交流的过程。

例子，找出第 1500 个丑数。可以针对丑数定义提问。有些面试官故意不把信息给全。

在解题前要搞清楚面试官的意图。

知识迁移能力。”举一反三“，连续几个问题，由易到难。

#### 面试题38：数字在排序数组中出现的次数。

> 统计一个数字在排序数组中出现的次数。例如输入数组 {1, 2, 3, 3, 3, 3, 4, 5} 和数字 3，数字 3 在数组中出现了 4 次，输出 4。

输入的数组是有序的，自然想到能够利用二分查找。

偷懒的做法，直接用 C++ 的 lower_bound 和 upper_bound 函数。

- lower_bound 返回非递减序列中第一个大于等于 val 的元素位置
- upper_bound 返回非递减序列中第一个大于 val 的元素位置

自己实现 upper_bound 和 lower_bound。首先分析 lower_bound，即在数组中找到第一个 k 出现的位置。二分查找总是先拿数组中间的数字和 k 作比较。如果中间的数字比 k 大，那么 k 只有可能出现在数组的前半段，下一轮只在前半段查找。如果中间的数字比 k 小，那么 k 只可能出现在数组的后半段。如果中间的数字等于 k ？此时要看中间数字的前一个数字，如果前一个不是 k ，则中间数字就是第一个 k，否则继续在前半段中查找。

```cpp
class Solution {
public:
    int GetNumberOfK(vector<int> data, int k) {
        return get_end_k(data, k) - get_start_k(data, k);
    }

private:
    // 返回 [left, right] 中第一个 k 出现的位置
    int get_start_k(const vector<int> &data, int k) {
        int left = 0, right = data.size() - 1;
        while (left <= right)
        {
            int mid = (left + right) / 2;
            if (k > data[mid])
            {
                left = mid + 1;
            }
            else
            {
                right = mid - 1;
            }
        }

        return left;
    }

    // 返回 [left, right] 中最后一个 k 后一个位置
    int get_end_k(const vector<int> &data, int k) {
        int left = 0, right = data.size() - 1;
        while (left <= right)
        {
            int mid = (left + right) / 2;
            if (k < data[mid])
            {
                right = mid - 1;
            }
            else
            {
                left = mid + 1;
            }
        }

        return left;
    }
};
```

上面的实现很巧妙，合并了 `k == data[mid]` 的情况。不放心的话每次判断的时候可以单独判断 `k == data[mid]` 。

二分查找非常容易写错， while 里的条件不同，left、right 定义不同，则 if 中的 left、right 更新方式都不同。最好好好整理一下二分查找的各种实现。

#### 面试题39：二叉树的深度

> 给定一个二叉树，找出其最大深度。二叉树的深度为根节点到最远叶子节点的距离。

直接根据二叉树的递归定义来做：

```cpp
class Solution {
public:
    /**
    * @param root: The root of binary tree.
    * @return: An integer
    */
    int maxDepth(TreeNode *root) {
        // write your code here
        if (root == NULL)
            return 0;

        return max(maxDepth(root->left), maxDepth(root->right)) + 1;
    }
};
```

#### 面试题39-2：判断平衡二叉树

> 平衡二叉树任意节点的左右子树深度相差不超过 1 。

利用上面求二叉树深度的代码可以迅速写出如下的代码：

```cpp
class Solution {
public:
    bool IsBalanced_Solution(TreeNode* root) {
        if (root == NULL)
            return true;
        int left_depth = tree_depth(root->left);
        int right_depth = tree_depth(root->right);
        if (abs(left_depth - right_depth) > 1)
            return false;

        return IsBalanced_Solution(root->left) && IsBalanced_Solution(root->right);
    }

private:
    int tree_depth(TreeNode *root) {
        if (root == NULL)
            return 0;

        return max(tree_depth(root->left), tree_depth(root->right)) + 1;
    }
};
```

缺点是需要重复遍历同一节点多次（求每个节点对应子树的深度时会重复遍历一些节点），效率不高。

如何优化使得每个节点只被遍历一次？利用后序遍历。后序遍历二叉树，在遍历某个节点的左右子树之后，可以根据其左右子树的深度判断它是不是平衡的，同时得到当前节点的深度。

```cpp
class Solution {
public:
    bool IsBalanced_Solution(TreeNode* root) {
        int depth = 0;
        return is_balanced(root, depth);
    }

private:
    bool is_balanced(TreeNode *root, int &depth) {
        if (root == NULL)
        {
            depth = 0;
            return true;
        }

        int left_depth = 0, right_depth = 0;
        if (is_balanced(root->left, left_depth) && is_balanced(root->right, right_depth))
        {
            if (abs(left_depth - right_depth) <= 1) 
            {
                depth = max(left_depth, right_depth) + 1;
                return true;
            }
        }

        return false;
    }
};
```

求二叉树深度 -> 判断平衡二叉树。考察知识迁移能力。

#### 面试题40：数组中只出现一次的数字

> 一个整型数组里除了两个数字之外，其他的数字都出现了两次。请写程序找出这两个只出现一次的数字。

要求只遍历一次，使用常数空间。

异或运算：任何一个数异或它自己都等于 0；任何一个数异或 0 都等于它自己。

#### 面试题41：和为 S 的两个数字

> 输入一个递增排序的数组和一个数字S，在数组中查找两个数，使得他们的和正好是S，如果有多对数字的和等于S，输出两个数的乘积最小的。 

O(n^2) 的方法，先在数组中固定一个数字，再依次判断数组中其余的 n-1 个数字与它的和是不是等于 S。

两指针解法，头尾向中间求和。和比 S 小，头向中间移，和比 S 大，尾向中间移。

如果输入的数组不是有序的，先排序即可。

#### 面试题41-2：和为S的连续正数序列

> 小明很喜欢数学,有一天他在做数学作业时,要求计算出9~16的和,他马上就写出了正确答案是100。但是他并不满足于此,他在想究竟有多少种连续的正数序列的和为100(至少包括两个数)。没多久,他就得到另一组连续正数和为100的序列:18,19,20,21,22。现在把问题交给你,你能不能也很快的找出所有和为S的连续正数序列? Good Luck! 

> 输入一个正数 S，打印出所有和为 S 的连续正数序列（至少含两个数）。

沿着题目1的思路思考，可以使用两指针。用 small 和 big 分别表示连续序列的最小值和最大值。首先把 small 初始化为 1，big 初始化为 2。如果从 small 到 big 的序列和小于 S。我们可以增大 big。让这个序列包含更多的数字。因为这个序列至少要有两个数字，我们一直增加 small 到 (S + 1) / 2 为止。

```cpp
class Solution {
public:
    vector<vector<int> > FindContinuousSequence(int sum) {
        vector<vector<int> > res;
        int small = 1, big = 2;
        int half_sum = (sum + 1) / 2;
        while (small < half_sum)
        {
            int seq_sum = sequence_sum(small, big);
            if (seq_sum == sum)
            {
                res.emplace_back(gen_sequence(small, big));
                ++small;
            }
            else if (seq_sum < sum)
            {
                ++big;
            }
            else
            {
                ++small;
            }
        }

        return res;
    }

private:
    inline int sequence_sum(int small, int big) {
        return (small + big) * (big - small + 1) / 2;
    }

    vector<int> gen_sequence(int small, int big) {
        vector<int> res(big - small + 1);
        std::iota(res.begin(), res.end(), small);

        return res;
    }
};
```

这段代码可以进行时间效率上的优化，注意 sequence_sum 求和会导致很多重复的数字被遍历。可以进行增量求和。

```cpp
class Solution {
public:
    vector<vector<int> > FindContinuousSequence(int sum) {
        vector<vector<int> > res;
        int small = 1, big = 2;
        int half_sum = (sum + 1) / 2;
        int cur_sum = small + big;
        while (small < half_sum)
        {
            if (cur_sum < sum)
            {
                ++big;
                cur_sum += big;
            }
            else
            {
                if (cur_sum == sum)
                    res.emplace_back(gen_sequence(small, big));

                cur_sum -= small;
                ++small;
            }
        }

        return res;
    }

private:
    vector<int> gen_sequence(int small, int big) {
        vector<int> res(big - small + 1);
        std::iota(res.begin(), res.end(), small);

        return res;
    }
};
```

#### 面试题42：翻转单词顺序

> 输入一个英文句子，翻转句子中单词的顺序，但单词内字符的顺序不变。标点符号当成普通字符处理。如 "I am a student." 翻转后为 "student. a am I"。

思路是先将整个句子作为一个字符串翻转，再翻转每个单词。

```cpp
class Solution {
public:
    string ReverseSentence(string str) {
        std::reverse(str.begin(), str.end());
        int i = 0;
        int len = str.length();
        while (i < len) {
            int j;
            for (j = i + 1; j < len && str[j] != ' '; ++j);
            std::reverse(str.begin() + i, str.begin() + j);
            i = j + 1;
        }

        return str;
    }
};
```

加入以下条件：

- 单词的构成：无空格字母构成一个单词
- 输入字符串是否包括前导或者尾随空格？可以包括，但是反转后的字符不能包括
- 如何处理两个单词间的多个空格？在反转字符串中间空格减少到只含一个

则可以先以空格为 delimit 进行字符串分割，再反向连接（注意特殊情况的处理）：

```cpp
class Solution {
public:
    /**
    * @param s : A string
    * @return : A string
    */
    string reverseWords(string s) {
        // write your code here
        if (s.empty())
            return s;
        vector<string> words = split(s, ' ');
        if (words.size() == 0)
            return "";
        return std::accumulate(words.rbegin() + 1, words.rend(), words.back(), 
            [](const std::string &a, const std::string &b){
                return a + ' ' + b;
        });
    }

private:
    vector<string> split(string s, char delim = ' ') {
        vector<string> res;
        int len = s.length();
        for (int i = 0; i < len; ++i)
        {
            for (; i < len && s[i] == delim; ++i);
            int j;
            for (j = i + 1; j < len && s[j] != delim; ++j);
            if (i < len && i < j)
                res.push_back(s.substr(i, j - i));
            i = j;
        }

        return res;
    }
};
```

#### 面试题42-2：字符串的左旋转

即经典的数组左移位问题。

"abcdefg" 左旋转 2 位得到 "cdefgab"。

从翻转单词顺序得到启发。"hello world" 翻转单词后得到 "world hello" ，很像是左移了一个单词 hello（不考虑空格）。

先分别翻转两部分，再翻转整个字符串。"abcdefg" -> "bagfedc" -> "cdefgab"

```cpp
class Solution {
public:
    string LeftRotateString(string str, int n) {
        int len = str.length();
        if (len == 0)
            return str;
        n %= len;
        std::reverse(str.begin(), str.begin() + n);
        std::reverse(str.begin() + n, str.end());
        std::reverse(str.begin(), str.end());

        return str;
    }
};
```

### 抽象建模能力

- 选择合理的数据结构来描述问题
- 分析模型中的内在规律

#### 面试题43：n 个骰子的点数

> 把 n 个骰子扔在地上，所有骰子朝上一面的点数之和为 s ，输入 n 打印出 s 的所有可能值出现的概率。

骰子一共 6 个面，每个面对应 1 到 6 之间的一个数字。n 个骰子的点数和的最小值为 n，最大值为 6n。根据排列组合， n 个骰子的所有点数的排列数为 6^n。先统计出每一个点数和出现的次数，然后把每一个点数和出现的次数除以 6^n，就能求出每个点数出现的概率。

解法1：递归求点数和出现次数（时间效率低）

要想求出 n 个骰子的点数和，可以先把 n 个骰子分成两堆，第一堆只有一个，另一堆有 n - 1 个。单独的那一个有可能出现从 1 到 6 的点数。我们需要计算从 1 到 6 的每一个和剩下的 n-1 个骰子能够组成的点数和。对 n-1 个骰子继续递归。。。

点数和可能值为 [n, 6n]，定义一个长度为 6n - n + 1 的数组，和为 s 的点数出现的次数保存到数组的第 s - n 个元素中。

解法2：基于循环求点数和

考虑用两个数组来保存点数和的总数。在一次循环中，第一个数组中的第 n 个数字表示点数和为 n 出现的次数。在下一次循环中，我们加上一个新的骰子，此时和为 n 的骰子出现的次数应该等于上一次循环中点数和为 n-1, n-2, n-3, n-4, n-5, n-6 的次数的总和，所以我们把另一个数组的第 n 个数字设为前一个数组对应的第 n-1, n-2, n-3, n-4, n-5, n-6 之和。

```cpp
int g_max_value = 6;
void print_probility(int number)
{
    if (number < 1)
        return;

    vector<int> probabilities[2];
    probabilities[0].resize(g_max_value * number + 1, 0);
    probabilities[1].resize(g_max_value * number + 1, 0);

    int flag = 0;
    for (int i = 1; i <= g_max_value; ++i)
    {
        probabilities[flag][i] = 1;
    }

    for (int k = 2; k <= number; ++k)
    {
        for (int i = 0; i < k; ++i)
            probabilities[1 - flag][i] = 0;

        for (int i = k; i <= g_max_value * k; ++i)
        {
            probabilities[1 - flag][i] = 0;
            for (int j = 1; j <= i && j <= g_max_value; ++j)
            {
                probabilities[1 - flag][i] += probabilities[flag][i - j];
            }
        }

        flag = 1 - flag;
    }

    double total = pow((double)g_max_value, number);
    for (int i = number; i <= g_max_value * number; ++i)
    {
        double ratio = (double)probabilities[flag][i] / total;
        printf("%d: %e\n", i, ratio);
    }
}
```

#### 面试题44：扑克牌的顺子

> 从扑克牌中随机抽 5 张牌，判断是不是一个顺子，即这 5 张牌是不是连续的。大、小王可以看成任意数字。

不妨把大、小王都定义为 0 ，以与 1 至 13 区分开。

1. 将数组排序（如果出现重复的非 0 数，一定不是顺子）
2. 统计数组中 0 的个数
3. 统计排序后数组中相邻数字之间的空缺总数，如果空缺的总数小于等于 0 的个数，则是顺子

```cpp
class Solution {
public:
    bool IsContinuous(vector<int> numbers) {
        if (numbers.empty())
            return false;

        sort(numbers.begin(), numbers.end());

        int num_zeros = 0;
        int num_gaps = 0;

        for_each(numbers.begin(), numbers.end(), [&num_zeros](const int &x) {
            if (x == 0)
                ++num_zeros;
        });

        int len = numbers.size();
        int small = num_zeros;
        int big = small + 1;
        while (big < len)
        {
            if (numbers[small] == numbers[big])
                return false;

            num_gaps += (numbers[big] - numbers[small] - 1);
            ++small, ++big;
        }

        return num_zeros >= num_gaps;
    }
};
```

#### 面试题45：圆圈中最后剩下的数字（约瑟夫环）

> 0, 1, 2, ..., n-1 这 n 个数字排成一个圆圈，从数字 0 开始数，删除数到的第 m 个数字，然后从被删的数字的下一个数开始数，删除数到的第 m 个数字。。。求这个圆圈中剩下的最后一个数。

解法1：环形链表模拟

```cpp
class Solution {
public:
    int LastRemaining_Solution(unsigned int n, unsigned int m)
    {
        if (n < 1 || m < 1)
            return -1;

        list<int> numbers;
        for (int i = 0; i < n; ++i)
            numbers.push_back(i);

        auto cur = numbers.begin();
        while (numbers.size() > 1)
        {
            for (int i = 1; i < m; ++i)
            {
                ++cur;
                if (cur == numbers.end())
                    cur = numbers.begin();
            }

            auto next = ++cur;
            if (next == numbers.end())
                next = numbers.begin();

            --cur;
            numbers.erase(cur);
            cur = next;
        }

        return *cur;
    }
};
```

时间复杂度为 O(m * n)，空间复杂度 O(n)

解法2：数学性质

```
记 f(n, m) 为 [0, ..., n-1] 这 n 个数，每次删除第 m 个数（然后从被删的数的下一个位置开始计数），这样删到最后剩下的那一个数。

第一个被删除的数字为 (m - 1) % n。此时圆圈中的数从 n 个变成了 n-1 个。为了简单起见，令 k = (m-1) % n，则剩下的 n-1 个数为 [0, 1, 2, ..., k-1, k+1, ..., n-1]。下一次删除从数字 k+1 开始计数，相当于在剩下的序列中，k+1 排在最前面，形成 [k+1, ..., n-1, 0, 1, ..., k-1]。这个序列最后剩下的数字也是关于 n 和 m 函数，但由于此时该序列已经不符合 f(n, m) 的定义（从 0 开始的连续序列），因此该函数不同于前面定义的函数，记为 f'(n-1, m)。最初序列最后剩下的数字 f(n, m) 一定是删除一个数字之后的序列最后剩下的数字，即 f(n, m) == f'(n-1, m)

我们把剩下的 n-1 个数字 [k+1, ..., n-1, 0, 1, ..., k-1] 做一个映射：
k+1 -------------------------> 0
k+2 -------------------------> 1
...
n-1  -------------------------> n-k-2
0      -------------------------> n-k-1
1      -------------------------> n-k
...
k-1   -------------------------> n-2

我们把该映射定义为 p()，则 p(x) = (x-k-1) % n，其逆映射为 p^{-1}(x) = (x+k+1) % n。

映射后的序列仍然满足从 0 开始的连续序列，可以用 f 来表示，记为 f(n-1, m)。

f'(n-1, m) = p^{-1} [ f(n-1, m) ] = [f(n-1, m) + k + 1] % n，把 k = (m-1) % n 代入，得到：

f(n, m) = [f(n-1, m) + m] % n, n >= 1

当 n = 1 时，只有一个数 0，则 f(1, m) = 0
```


```cpp
class Solution {
public:
    int LastRemaining_Solution(unsigned int n, unsigned int m)
    {
        if (n < 1 || m < 1)
            return -1;

        int last = 0;
        for (int i = 2; i <= n; ++i)
            last = (last + m) % i;

        return last;
    }
};
```

### 发散思维能力

- 探索多种解法。
- 是否有探索新思路的激情。

#### 面试题46：求 1+2+...+n

> 要求不能使用乘除法、for、while、if、else、switch、case 等关键字以及三目条件运算语句。

解法1：利用构造函数求解

不能用循环，循环只是让相同的代码重复执行 n 遍而已，我们可以先定义一个类型，接着创建 n 个该类型的实例，那么这个类型的构造函数将被调用 n 次。

解法2：利用虚函数

不能使用 if ，所以不能直接用递归（递归终止条件）。不妨定义两个函数，一个函数充当递归函数的角色，另一个函数处理终止递归的情况。现在还需要的就是在两个函数里二选一，自然要用到 bool 变量，比如 true 时调用第一个函数，false 调用第二个函数。如何把数值 n 转换为 bool 变量。两次取反，非0 的 n 两次取反后为 true，0 两次取反后为 false。

```cpp
class A;
A* func[2];

class A
{
public:
    virtual unsigned int sum(unsigned int n)
    {
        return 0;
    }
};

class B : public A
{
public:
    unsigned int sum(unsigned int n)
    {
        return func[!!n]->sum(n - 1) + n;
    }
};

unsigned int sum_from_one_to_n(unsigned int n)
{
    A a;
    B b;
    func[0] = &a;
    func[1] = &b;

    return b.sum(n);
}
```

解法3：利用函数指针

本质上还是用两个函数，一个作为递归边界。

```cpp
typedef unsigned int(*fun)(unsigned int);

unsigned int terminator(unsigned int n)
{
    return 0;
}

unsigned int sum_from_one_to_n(unsigned int n)
{
    static fun f[2] = { terminator, sum_from_one_to_n };
    return n + f[!!n](n - 1);
}
```

解法4：利用模板

```cpp
template <unsigned int n>
struct sum_solution_t
{
    enum
    {
        N = sum_solution_t<n - 1>::N + n
    };
};

template <>
struct sum_solution_t<1>
{
    enum
    {
        N = 1
    };
};
```

sum_solution_t<100>::N 就是 1 到 100 的累加和。

这里利用了模板类型特化作为递归边界。

#### 面试题47：不用加减乘除做加法

> 写一个函数，求两个整数之和，要求在函数体内不得使用 + - * / 运算符。

如何分析这个问题？不能用四则运算，剩下的只有位运算了。怎么找规律？

```
先考虑我们是如何进行 10 进制加法的。
以 5 + 17 = 22 为例子。可以分三步进行：
1. 只做各个位上的数字相加，不考虑进位 -> 5+7 取个位为 2，得 12
2. 做进位，5+7的进位为 10
3. 将前面的两个结果加起来，得 5 + 17 = 22

现在从二进制来考虑。
5 的二进制为 00101
17 的二进制为 10001

1. 不考虑进位，10100 
2. 只看进位，00010
3. 将两个结果相加，10110

注意第一步不考虑进位做各个二进制位的相加，相当于异或运算。
第二步只看进位，可以用“与操作+左移一位”实现。
第三步再将两个结果相加，注意不能直接相加，于是重复前面两步，这就形成了递归，直到不产生进位为止（此时异或结果就是最终结果）。

例子：

====================================
7 + 2 = 9

7 : 0111
2 : 0010
9 : 1001

不考虑进位的二进制“加”：

0111
0010
------
0101

相当于“异或”。

怎么处理进位？

当两个数对应位上都为 1 时会发生进位。

使用“与”操作：

0111
0010
------
0010

0010 左移一位 0100，和 0101 异或得到 0001（不考虑进位的“加法”），和0101与得到 0100；

0100 左移一位 1000，和 0001 异或得到 1001，和 0001 与得到 0000（终止）。
====================================
```

```cpp
class Solution {
public:
    int Add(int num1, int num2)
    {
        int sum = 0, carry = 0;

        do 
        {
            sum = num1 ^ num2;
            carry = (num1 & num2) << 1;

            num1 = sum;
            num2 = carry;
        } while (carry != 0);

        return sum;
    }
};
```

#### 面试题48：不能被继承的类

> 设计一个不能继承的 C++ 类

标准解法：将构造函数设为私有函数

C++ 中子类的构造函数会自动调用父类的构造函数，子类的析构函数也会自动调用父类的析构函数。将一个类的构造函数和析构函数都定义为私有，当一个类试图从它那继承（并有 public 的构造函数）的时候，会导致编译错误。

问题是这个类的构造函数是私有的，如何创建实例？可以利用公有的静态类方法。

```cpp
class A
{
public:
    static A* create_instance()
    {
        return new A();
    }

    static void delete_instance(A* a)
    {
        delete a;
        a = NULL;
    }

private:
    A() {}
    ~A() {}
};

class B : public A
{
public:
    B() {}   // 编译错误
    ~B() {}  // 编译错误
};
```

更优雅的解法：利用虚拟继承

```cpp
template <typename T> 
class MakeSealed
{
    friend T;

private:
    MakeSealed() {}
    ~MakeSealed() {}
};

class A : virtual public MakeSealed<A>
{
public:
    A() {}
    ~A() {}
};

// 由于 A 是由 MakeSealed<A> 虚拟继承来的，在调用 B 的构造函数
// B() 时会跳过 A 的构造函数 A()，直接调用 MakeSealed<A> 的构造函数
// 但是 B 不是 MakeSealed<A> 的友元，所以编译不通过。
class B : public A
{
public:
    B() {}   // B 不是 MakeSealed<A> 的友元类型
    ~B() {}
};
```

注意如果 A 不用虚拟继承，那么 B 还是可以通过编译的。（关键是虚拟继承情况下构造函数的调用顺序发生了变化）
