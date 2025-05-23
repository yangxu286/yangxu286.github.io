---
title: Leetcode Day05：动态规划
date: 2025-05-23 23:20:00 +0800
tags: [算法, Leetcode, 动态规划]
categories: [算法]
---
注：为随手写的刷题笔记，请谨慎参考，欢迎指出错漏。
今天脖子很酸很酸，所以只能水一水了。
#### 152.乘积最大子数组

想了二十分钟没想出来，看了官方题解，思路是除了维护以i结尾的最大乘积，还维护以i结尾的最小乘积，这样正数和负数绝对值最大的结果都可以被分别保留下来。我隐约有想到需要采取存绝对值之类的方法，但想不到这个。

```c++
class Solution {
public:
    int maxProduct(vector<int>& nums) {
        int len = nums.size();
        long fmax = nums[0];
        long fmin = nums[0];
        long ans = nums[0];
        for(int i = 1; i < len; i++){
            long fmax_last = fmax;
            long fmin_last = fmin;
            fmax = max(fmax_last * nums[i], max(fmin_last * nums[i], (long)nums[i]));
            fmin = min(fmax_last * nums[i], min(fmin_last * nums[i], (long)nums[i]));
            ans = max(fmax, ans);
        }
        return ans;
    }
};
```

但我还是觉得官方题解下面的一个思路比较类人：对于最后的解，不会任何一边有正数，有的话肯定能乘正数变大或不变；不会同时两边有负数，有的话肯定能乘俩负数变大或不变。所以答案一定有一端是0或整个数组的端。因此从左右两端分别开始算乘积，遇到0就重新当起点，ans取这个过程中遇到的最大值即可。

不过这个题解ans还特别考虑了只有当前nums[i]为最大的情况。

```c++
class Solution {
public:
    int maxProduct(vector<int>& nums) {
        int ans = INT_MIN, n = nums.size(), pre = 1, bck = 1;
        for (int i = 0, j = n - 1; i < n; ++i, --j) {
            pre *= nums[i], bck *= nums[j];
            ans = max(max(ans, nums[i]), max(pre, bck));
            if (nums[i] == 0) pre = 1;
            if (nums[j] == 0) bck = 1;
        }

        return ans;
    }
};
```

