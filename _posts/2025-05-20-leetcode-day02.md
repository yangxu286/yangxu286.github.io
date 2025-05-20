---
title: Leetcode Day02：二分初探
date: 2025-05-19 12:05:00 +0800
tags: [算法, Leetcode, 二分]
categories: [算法]
---
注：为随手写的刷题笔记，请谨慎参考，欢迎指出错漏。

### 二分

二分这么好背板还是背板舒服，背板又属左开右开区间最好背：

```c++
    int lowerBound(vector<int>& nums, int target){
        int len = nums.size();
        int left = -1;
        int right = len;
        while(left + 1 < right){
            int mid = left + (right - left) / 2;
            if(nums[mid] < target){
                left = mid;
            }
            else if(nums[mid] >= target){
                right = mid;
            }
        }
        return right;
    }
```

重点要理解的就在于left及其左（闭区间则left-1及其左）全部满足<target，right及其右（闭区间则right+1及其右）全部满足>=target。

---

不过背板也没那么可靠，尤其是在脱离常规的一个二分查找的时候，就要认真考虑下标的含义。

#### 搜索旋转排序数组

奇葩的我，一开始忘记了O(logn)，所以就先暴力找到旋转点，然后进行一个下标换算过后的普通二分。后来想起这一点，想的就是先二分找到旋转点，再二分进行下标换算。但想着想着感觉二分找到旋转点如果真的可行也有些难写（应该是我太菜），所以就看了题解。

实际上做法：每次二分必然是一边顺序一边乱序，判断依据就是区间的左端点和右端点比大小。顺序的话，就可以很方便地判断target是否在内。如果在内，则在里面搜索，如不在内，则进入乱序区间，再重复上述操作。

```c++
int search(vector<int>& nums, int target) {
        int len = nums.size();
        int left = 0;
        int right = len - 1;
        while(left <= right){
            int mid = left + (right - left) / 2;
            if(nums[mid] == target) return mid;
            if(nums[left] <= nums[mid]){
                if(target >= nums[left] && target < nums[mid]){
                    right = mid - 1;
                }
                else{
                    left = mid + 1;
                }
            }
            else{
                if(target > nums[mid] && target <= nums[right]){
                    left = mid + 1;
                }
                else{
                    right = mid - 1;
                }
            }
        }
        return -1;
    }
```

但是，仍然是因为我太菜，不理解为什么要判断left~mid-1区间的有序，却用`nums[left] <= nums[mid]`才可以，`nums[left] < nums[mid - 1]`不可以。

后来又在题解区翻翻，找到了正确的找旋转点的写法：

```c++
class Solution {
    public int search(int[] nums, int target) {
        int min = 0, n = nums.length;
        for (int l = 1, r = n - 1; l <= r;) {
            int m = (l + r) / 2;
            if (nums[0] < nums[m]) l = m + 1;
            else { r = m - 1; min = m; }
        }
        for (int l = min, r = l + n - 1; l <= r;) {
            int m = (l + r) / 2, i = m % n;
            if (target < nums[i]) r = m - 1;
            else if (target > nums[i]) l = m + 1;
            else return i;
        }
        return -1;
    }
}
```

这个思路和我相比的优越之处在于，我试图找的旋转点是“左比右大”的特异点，而这个思路其实和官方题解也有一定的相似之处，一个是左右区间必有一个有序，一个是通过比较左右端点大小判断有序。第一个二分的目的是找到**从下标0开始的有序部分的末尾**，根据题目的性质可以简化成**比下标0大的最大位置+1**，这样就很好理解了。

况且，这个思路还可以顺带解决了hot100的下一道题，即[寻找旋转排序数组中的最小值](https://leetcode.cn/problems/find-minimum-in-rotated-sorted-array/)。然而，就会浮现出个问题：为什么left要从1开始，否则会报错？

我的想法是，假如m到了0上，而m不是旋转点，那么`nums[0] < nums[m]`就一定是false，这就会导致错误地收敛了右端点，把m认为是旋转点。这样的情况很罕见但有，比如[2,1]。不过转念一想，假如m到了0上且0正好就是旋转点（相当于没有旋转），则只会执行第一个分支`left = mid + 1`，min根本没有被赋值的机会，所以也可以返回默认的min值，即0。我进行了两个小小的修改也能达到同样的效果，且更好理解。这种做法有一种不大规范但巧妙狡猾的感觉，仁者见仁智者见智，我感觉不是很好。

```c++
class Solution {
public:
    int findMin(vector<int>& nums) {
        int len = nums.size();
        int left = 0;//从0开始
        int right = len - 1;
        int min = 0;
        while(left <= right){
            int mid = left + (right - left) / 2;
            if(nums[0] <= nums[mid]){//和自身比较时不会收敛右端点
                left = mid + 1;
            }
            else{
                right = mid - 1;
                min = mid;
            }
        }
        return nums[min];
    }
};
```

搞了一个下午，还是感觉二分细节很不清楚，循环不变量、mid是取上还是取下等等都不太明白，明天再来梳理。