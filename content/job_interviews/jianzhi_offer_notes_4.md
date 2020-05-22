---
title: "剑指offer笔记(4)"
date: 2019-07-13 01:34
collection: algorithm interviews
tag: notes,algorithm
---

[TOC]

[《剑指Offer》](https://book.douban.com/subject/27008702/) 读书笔记。

## 第五章 优化空间和时间效率

高级开发人员必须考虑的问题。

往往时间复杂度更重要一些。

### 时间效率

形参尽量使用指针或引用类型，避免值传递开销。

递归的时间效率 < 循环的时间效率。(递归能够省去一些中间变量的定义，参考 [递归与循环相比时间优势的真正来源](https://blog.csdn.net/ccutyear/article/details/52819123))

合理的算法与数据结构。查找，顺序查找 O(n)，有序用二分查找 O(log(n))

#### 面试题29：数组中出现次数超过一半的数字

> 给定一个整型数组，找出主元素，它在数组中的出现次数严格大于数组元素个数的二分之一。

可以用 Hash 表记录每个元素出现次数，但这样空间复杂度为 O(n)。

尝试给出时间复杂度 O(n)，空间复杂度 O(1) 的算法。

解法1：基于 partition

partiton 可以得到数组的第 k 大数字。本题令 k = n/2 即可。注意这种方法时间复杂度为 O(n) 而不是 O(n * log(n))，分析的时候利用递归式。

解法2：设置一个计数器来保存主元素

```cpp
int major_num(std::vector<int> nums) {
  int res = nums[0];
  int cnt = 1;
  for (int i = 0; i < nums.size(); ++i) {
    if (res == nums[i]) {
      ++cnt;
    } else {
      --cnt;
      if (cnt == 0) {
        res = nums[i];
        cnt = 1;
      }
    }
  }
  return res;
}
```

#### 面试题30：最小的 k 个数

> 输入 n 个数，找出其中最小的 k 个数。

解法1：利用 partition，时间复杂度O(n)

如果允许改变原数组，空间复杂度 O(1)，否则需要先复制一份原数组，空间复杂度 O(n)。

基于数组的第 k 个元素进行调整，使得比第 k 个数字小的所有数字都位于数组的左边，比第 k 个数字大的所有数字都位于数组的右边。这样调整之后，左边的 k 个数字就是最小的 k 个数字（这 k 个数字不一定有序）。

```cpp
// 按照“左小右大”划分数组，每次取 left 位置元素作为 pivot
int partition(vector<int> &nums, int left, int right)
{
    int pivot = nums[left];
    while (left < right)
    {
        while (left < right && nums[right] > pivot)
            --right;
        nums[left] = nums[right];

        while (left < right && nums[left] < pivot)
            ++left;
        nums[right] = nums[left];
    }
    nums[left] = pivot;

    return left;
}

vector<int> get_k_least_numbers(vector<int> nums, int k) {
    vector<int> result;
    if (nums.empty())
        return result;

    int left = 0;
    int right = nums.size() - 1;
    int index = partition(nums, left, right);
    while (index != k - 1)
    {
        if (index > k - 1)
        {
            right = index - 1;
            index = partition(nums, left, right);
        }
        else
        {
            left = index + 1;
            index = partition(nums, left, right);
        }
    }

    for (int i = 0; i < k; ++i)
    {
        result.push_back(nums[i]);
    }

    return result;
}
```

解法2：利用优先队列，维护包含 k 个最小数字的优先队列，时间复杂度 O(n * log(k))，空间复杂度 O(k)。适合处理海量数据。

STL 中的 priority_queue 或者 set/multiset 都可以用。

priority_queue 是容器适配器，其实现容器默认为 vector。

set 和 multiset 一般都是基于红黑树（一种平衡 BST）实现的。

创建一个大小为 k 的容器存储最小的 k 个数字。每次从 n 个数中读入一个数。如果容器中的已有数字少于 k 个，直接把新读入的数放到容器中。如果容器中已经有 k 个数，找出已有 k 个数中的最大值，和这次新读入的数比较，保留较小的数在容器中。

最小的 k 个数应该用大顶堆；最大的 k 个数用小顶堆。

```cpp
vector<int> get_k_least_numbers(vector<int> nums, int k) {
    vector<int> result;
    if (nums.empty())
        return result;

    // 最小的 k 个数用大顶堆
    multiset<int, std::greater<int>> heap;

    for (int i = 0; i < nums.size(); ++i)
    {
        if (heap.size() < k)
        {
            heap.insert(nums[i]);
        }
        else
        {
            auto iter_greatest = heap.begin();
            if (nums[i] < *iter_greatest)
            {
                heap.erase(iter_greatest);
                heap.insert(nums[i]);
            }
        }
    }

    result.assign(heap.begin(), heap.end());

    return result;
}
```

#### 面试题31：连续子数组的最大和

> 输入一个整数数组，数组里既有正数也有负数，数组中一个或连续的多个整数组成一个子数组，求所有子数组的和的最大值，要求时间复杂度 O(n)。

经典题之一。

本质是动态规划思想。

```
设数组为 a[0], ..., a[n]。

记 D[i] 为以 a[i] 为结尾的连续子数组的最大和。

则 D[0] = a[0]

D[i] = max(D[i - 1] + a[i], a[i]), i >= 1
```

```cpp
class Solution {
public:
    /**
    * @param nums: A list of integers
    * @return: A integer indicate the sum of max subarray
    */
    int maxSubArray(vector<int> nums) {
        // write your code here
        if (nums.empty())
        {
            return -1;
        }
        vector<int> D(nums.size());
        D[0] = nums[0];
        int max_sum = D[0];
        for (int i = 1; i < nums.size(); ++i)
        {
            if (D[i - 1] <= 0)
            {
                D[i] = nums[i];
            }
            else
            {
                D[i] = D[i - 1] + nums[i];
            }
            if (D[i] > max_sum)
            {
                max_sum = D[i];
            }
        }

        return max_sum;
    }
};
```

分析一下上面的状态转移方程，当 `D[i - 1] <= 0` 时，`D[i] = a[i]`，实际上可以不用数组 `D[]` 。

```cpp
class Solution {
public:
    /**
    * @param nums: A list of integers
    * @return: A integer indicate the sum of max subarray
    */
    int maxSubArray(vector<int> nums) {
        // write your code here
        if (nums.empty())
        {
            return 0;
        }
        int cur_sum = nums[0];
        int max_sum = cur_sum;
        for (int i = 1; i < nums.size(); ++i)
        {
            if (cur_sum <= 0)
            {
                cur_sum = nums[i];
            }
            else
            {
                cur_sum += nums[i];
            }
            if (cur_sum > max_sum)
            {
                max_sum = cur_sum;
            }
        }

        return max_sum;
    }
};
```

#### 面试题32：统计从 1 到 n 的整数中 1 出现的次数

> 例子，1 到 12 这 12 个整数中包含 1 的数有 1，10，11，12。1 一共出现了 5 次。

解法1：暴力求解

对每个数字都判断其中出现了多少个 1， 最后累加起来。

可以每次通过对 10 求余数判断整数的个位数字是不是 1。如果这个数字大于10，除以 10 后再判断个位数字是不是 1。

输入数字 n，n 有 log(n) 位。该方法的时间复杂度为 O(n * log(n))

解法2：找数字规律

以一个稍微大一点的数字为例子，如 21345。把 1 到 21345 分成两段，一段是从 1 到 1345，另一段是 1346 到 21345。

先分析从 1346 到 21345 的数，1 的出现分为两种情况：(1) 1 出现在最高位（本例中是万位），从 1346 到 21345 里，1 出现在 10000 至 19999 这 10000 个数字的万位，出现了 10000 次。注意并不是对所有 5 位数万位出现的次数都是 10000 次。例如 12345，2346 到 12345 中出现 1 的数为 10000 到 12345，1 出现了 2346 次。 (2) 1 出现在除最高位之外的其他四位中的情况。把 1346 到 21345 再分成两段，1346 至 11345 和 11346 至 21345，在每一段除万位之外的 4 位中，选择其中一位是 1，其余 3 位可以在 0 至 9 这 10 个数字中任意选择，根据排列组合规则， 1 总共出现的次数为 2 * 10^3 = 2000 次。

至于从 1 到 1345 中 1 出现的次数，可以利用递归求得，将其分为 1 至 345 和 346 至 1345 两段。

这种思路是每次去掉最高位做递归，递归的次数和位数相同。一个数 n 有 O(log(n)) 位，因此这种思路时间复杂度为 O(log(n))

```cpp
class Solution {
public:
    /*
     * param k : As description.
     * param n : As description.
     * return: How many k's between 0 and n.
     */
    int digitCounts(int k, int n) {
        int cnt = 0, multiplier = 1, left_part = n;

        while (left_part > 0) {
            // split number into: left_part, curr, right_part
            int curr = left_part % 10;
            int right_part = n % multiplier;

            // count of (c000 ~ oooc000) = (ooo + (k < curr)? 1 : 0) * 1000
            cnt += (left_part / 10 + (k < curr)) * multiplier;

            // if k == 0, oooc000 = (ooo - 1) * 1000
            if (k == 0 && multiplier > 1) {
                cnt -= multiplier;
            }

            // count of (oook000 ~ oookxxx): count += xxx + 1
            if (curr == k) {
                cnt += right_part + 1;
            }

            left_part /= 10;
            multiplier *= 10;
        }

        return cnt;
    }
};
```

#### 面试题33：把数组排成最小的数

> 输入一个正整数数组，把数组中的数字拼接起来排成一个数，输出能够排成的最小的数字。

例子，数组 {3, 32, 321}，能够排成的最小的数是 321323。

这题的本质是排序，根据一个排序规则将数组排列。

```cpp
auto cmp = [](const string &a, const string &b) {
    return a + b < b + a;
};
```

这里用字符串表示整数，便于进行拼接操作。

这是不是一个有效的排序规则？从数学上进行证明：

- 自反性
- 对称性
- 传递性

### 时间效率与空间效率的平衡

时间换空间，空间换时间。具体情况具体分析。

#### 面试题34：丑数

> 素数因子只包含 2、3、5 的数为丑数。求按从小到大顺序的第 1500 个丑数。例子，6、8 都是丑数，但 14 不是，因为它包含素因子 7。规定 1 为最小的丑数。

解法1：逐个判断每个整数是不是丑数

时间效率低。

解法2：空间换时间

一个丑数（除了1）是另一个丑数乘以 2 或 3 或 5 的结果。可以创建一个数组，里面的数字是排好序的丑数，每一个丑数都是前面的丑数乘以 2 或 3 或 5 得到的。

现在的问题是如何确保数组里面的丑数都是排好序的。直接利用 C++ 中的 heap（priority_queue）或者 bst（set ）来解决。

```cpp
class Solution {
public:
    /*
    * @param k: The number k.
    * @return: The kth prime number as description.
    */
    long long kthPrimeNumber(int k) {
        // write your code here
        long long ugly_number = 0;
        priority_queue<long long, vector<long long>, greater<long long>> heap;

        heap.emplace(1);
        for (int i = 0; i <= k; ++i)
        {
            ugly_number = heap.top();
            heap.pop();
            if (ugly_number % 3 == 0)
            {
                heap.emplace(ugly_number * 3);
            }
            else if (ugly_number % 5 == 0)
            {
                heap.emplace(ugly_number * 3);
                heap.emplace(ugly_number * 5);
            }
            else
            {
                heap.emplace(ugly_number * 3);
                heap.emplace(ugly_number * 5);
                heap.emplace(ugly_number * 7);
            }
        }
        return ugly_number;
    }
};
```

另一种解释方法：

> 要得到第N个丑陋数，最直接的想法就是从1开始递增循环，直到找到第N个丑陋数，但是这样明显开销太大，而且我们没有用到已经生成的丑陋数的信息。如果有一个方法能够顺序只生成丑陋数就好了。仔细观察可以发现，丑陋数的因子也必定是丑陋数，它一定是某个丑陋数乘2、3、5得到的。但问题在于，小的丑陋数乘5不一定比大的丑陋数乘2要小，我们没法直接使用目前最大的丑陋数来乘2、3、5顺序得到更大的丑陋数。不过，我们可以确定的是，小的丑陋数乘2，肯定小于大的丑陋数乘2。所以我们使用三个指针，分别记录乘2、3、5得出的目前最大丑陋数，这样我们通过比较这三种最大丑陋数（这里最大是相对于只乘2、只乘3、只乘5三种不同情况下最大的丑陋数），就得到了所有数里最大的丑陋数。
> 
> https://segmentfault.com/a/1190000003480992

#### 面试题35：第一个只出现一次的字符

> 例子，对于"abaccdeff"，输出 'b'。

直观想法：从头开始扫描这个字符串中的每个字符，当访问到某字符时，将该字符与其他字符比较，如果没有重复，输出。时间复杂度 O(n^2)。

空间换时间，利用 Hash 表，记录每个字符出现的次数。两次扫描字符串，第一次生成 Hash 表，第二次判断次数并输出。

```cpp
char first_not_repeated_char(const string &str)
{
    if (str.empty())
        return '\0';
    unordered_map<char, int> char_cnts;
    for (int i = 0; i < str.length(); ++i)
    {
        ++(char_cnts[str[i]]);
    }
    for (int i = 0; i < str.length(); ++i)
    {
        if (char_cnts[str[i]] == 1)
        {
            return str[i];
        }
    }
    return '\0';
}
```

#### 面试题36：数组中的逆序对

> 在数组中的两个数字如果前面一个大于后面的数字，则这两个数字构成一个逆序对，求一个数组中的逆序对总数。
> 
> 例子，数组 {7, 5, 6, 4} 中，一共存在 5 个逆序对，(7, 5), (7, 6), (7, 4), (5, 4), (6, 4)

<div class="mermaid">
graph TB
  id1[7 5 6 4]-->id2[7 5]
  id1-->id3[6 4]
  id2-->id4[7]
  id2-->id6[5]
  id3-->id5[6]
  id3-->id7[4]
  id4-->id8[5 7]
  id6-->id8
  id5-->id9[4 6]
  id7-->id9
  id8-->id10[4 5 6 7]
  id9-->id10
</div>

先把数组分割成子数组，先统计出子数组内部的逆序对的数目，然后再统计出两个相邻子数组之间的逆序对的数目。在统计逆序对的过程中，还需要对数组进行排序。

利用归并排序的思想完成求逆序对数目。

```
归并排序是将数列 a[l, h] 分成两半 a[l, mid] 和 a[mid+1, h] 分别进行归并排序，然后再将这两半合并起来。在合并的过程中（设 l<=i<=mid，mid+1<=j<=h），当 a[i] <= a[j] 时，并不产生逆序数；当 a[i] > a[j] 时，在前半部分中比 a[i] 大的数都比 a[j] 大（即 a[i] 到 a[mid] 都大于 a[j]），逆序数要加上 mid - i + 1。因此，可以在归并排序中的合并过程中计算逆序数.
```

```cpp
class Solution {
public:
    int InversePairs(vector<int> data) {
        vector<int> tmp(data.size());
        int res = 0;
        res = merge_sort(data, tmp, 0, data.size() - 1);
        return res;
    }

private:
    // return the number of inverse pairs
    int merge_sort(vector<int> &data, vector<int> &tmp, int left, int right)
    {
        int inv_count = 0;
        if (left >= right)
            return inv_count;
        int mid = (left + right) / 2;
        inv_count += merge_sort(data, tmp, left, mid);
        inv_count += merge_sort(data, tmp, mid + 1, right);
        inv_count += merge(data, tmp, left, mid, right);

        return inv_count;
    }

    int merge(vector<int> &data, vector<int> &tmp, int left, int mid, int right)
    {
        int i, j, k;
        int inv_count = 0;

        i = left;  // index for left subarray
        j = mid + 1;  // index for right subarray
        k = left;  // index for resultant merged subarray
        while (i <= mid && j <= right)
        {
            if (data[i] > data[j])
            {
                tmp[k++] = data[j++];

                // add number of inversions
                inv_count += (mid - i + 1);
            }
            else
            {
                tmp[k++] = data[i++];
            }
        }

        while (i <= mid)
        {
            tmp[k++] = data[i++];
        }
        while (j <= right)
        {
            tmp[k++] = data[j++];
        }
        for (int _i = left; _i <= right; ++_i)
        {
            data[_i] = tmp[_i];
        }

        return inv_count;
    }
};
```

#### 面试题37：两个链表的第一个公共节点

> 输入两个链表，找出它们的第一个公共节点。

解法1：暴力法

在第一个链表上顺序遍历每个节点，对每个节点，在第二个链表上顺序遍历，判断。假设两个链表的长度分别为 m 和 n，则时间复杂度为 O(m * n)。

解法2：利用两个辅助栈

如果两个链表有公共节点，那么公共节点出现在两个链表的尾部。如果从尾部开始往前比较，最后一个相同的节点就是我们要找的节点。但是链表不支持从后向前遍历，因此可以利用两个辅助栈。分别把两个链表的节点放入两个栈里，这样两个链表的尾节点就位于两个栈的栈顶，接下来比较两个栈顶的节点是否相同，直到找到最后一个相同的节点。

该方法时间复杂度 O(m + n)，空间复杂度 O(m + n)。

解法3：不用辅助空间

仍然顺着解法 2 的思路，之所以从尾部出发，是因为两个链表从尾节点出发到达公共节点的长度相同。如果我们知道两个链表的长度，那么让长的链表从头部出发先走若干步，接着再同时在两个链表上遍历，这样找到的第一个相同的节点就是它们的第一个公共节点。

时间复杂度 O(m + n)，空间复杂度 O(1)。

```cpp
 /**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode(int x) : val(x), next(NULL) {}
 * };
 */
class Solution {
public:
    /**
    * @param headA: the first list
    * @param headB: the second list
    * @return: a ListNode
    */
    ListNode *getIntersectionNode(ListNode *headA, ListNode *headB) {
        // write your code here
        int m = get_length_of_list(headA);
        int n = get_length_of_list(headB);
        int diff_len = (m > n) ? m - n : n - m;
        
        ListNode *p_long = headA;
        ListNode *p_short = headB;
        if (m < n)
        {
            p_short = headA;
            p_long = headB;
        }

        for (int i = 0; i < diff_len; ++i)
        {
            p_long = p_long->next;
        }

        while (p_short != NULL && p_long != NULL && 
            p_short != p_long)
        {
            p_short = p_short->next;
            p_long = p_long->next;
        }

        return p_short;
    }

private:
    int get_length_of_list(ListNode *head) {
        int len = 0;
        while (head != NULL)
        {
            ++len;
            head = head->next;
        }
        return len;
    }
};
```

本题的启发是，有时候空间换时间并不是必要的。

