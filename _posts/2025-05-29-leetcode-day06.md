---
title: Leetcode Day06：动态规划+哈希
date: 2025-05-29 17:12:00 +0800
tags: [算法, Leetcode, 动态规划, 哈希]
categories: [算法]
---
不知不觉就偷懒了好几天，偷懒的效果非常显著，不想做题，脑子变钝，题解看不进。

#### 416.分割等和子集

这题算是和之前做的dp不一样的思路了（非多项式复杂度），想不出也很正常。不过通过观察也可以发现数组长度和数的大小都很小，也是一个线索。

不过如何判断一个问题是否属于NP完全问题？

很像01背包，但是下面这个做法像01背包吗？回去看一下。

以下全部为抄题解。思路是dp\[i]\[j]为遍历到数组的第i个数时是否有一种方案使被选取的正整数的和等于j，那么对于每个i都可以从i-1获得状态。也由此可以把dp数组压缩到一维。

此外还可以进行一些事前的排除，详见代码（注意这里很方便的`accumulate`和`max_element`）：

```c++
class Solution {
public:
    bool canPartition(vector<int>& nums) {
        int len = nums.size();
        if(len < 2) return false;
        int sum = accumulate(nums.begin(), nums.end(), 0);
        int maxNum = *max_element(nums.begin(), nums.end());
        if(sum & 1) return false;
        int target = sum / 2;
        if(maxNum > target) return false;
        vector<bool> dp(target + 1, 0);
        dp[0] = true;
        for(int i = 0; i < len; i++){
            int num = nums[i];
            for(int j = target; j >= num; j--){
                dp[j] = (dp[j] || dp[j - num]);
            }
        }
        return dp[target];
    }
};
```

摸了70.爬楼梯和118.杨辉三角这两道简单题，然后熟悉了一下哈希相关。之前打oi一直是手搓的，不过leetcode应该不怎么会卡，都用的unordered_map。

#### 1.两数之和

熟悉unordered_map相关操作：find()如果找不到，返回hashtable.end()。

```c++
class Solution {
public:
    vector<int> twoSum(vector<int>& nums, int target) {
        int len = nums.size();
        unordered_map<int, int> hashtable;
        for(int i = 0; i < len; i++){
            auto it = hashtable.find(target - nums[i]);
            if(it != hashtable.end()){
                return {it->second, i};
            }
            hashtable[nums[i]] = i;
        }
        return {};
    }
};
```

另外，使用{}会自动返回一个初始化的对应的数据类型的对象，比如这里就是一个空的vector<int>。

#### 49.字母异位词分组

代码（抄官方题解）:

```c++
class Solution {
public:
    vector<vector<string>> groupAnagrams(vector<string>& strs) {
        unordered_map<string, vector<string>> hashTab;
        for(string str : strs){
            string key = str;
            sort(key.begin(), key.end());
            hashTab[key].emplace_back(str);
        }
        vector<vector<string>> ans;
        for(auto it = hashTab.begin(); it != hashTab.end(); it++){
            ans.emplace_back(it->second);
        }
        return ans;
    }
};
```

熟悉unordered_map相关操作：使用`for(auto it = hashTab.begin(); it != hashTab.end(); it++)`来遍历。

`emplace_back`对应于`push_back`，区别是

1. **push_back**：
   - 先创建临时对象，然后将其复制/移动到容器中
   - 需要一个已构造好的对象
   - 会调用拷贝构造函数或移动构造函数
2. **emplace_back**：
   - 直接在容器内部构造元素
   - 将参数完美转发给构造函数
   - 避免了创建临时对象和额外的拷贝/移动操作

对于构造成本高的对象，推荐使用emplace_back。但一般可以通用（至少这题我亲测可以）。

使用下面的方式来遍历（c++11特性）：

```c++
for (声明变量 : 容器) {
    // 循环体
}
```

#### sort用法

太久了记忆不清了……。其实只要记得第一种和第二种就够用？

```c++
// 排序范围 [first, last)
template< class RandomIt >
void sort(RandomIt first, RandomIt last);

// 使用自定义比较函数 comp
template< class RandomIt, class Compare >
void sort(RandomIt first, RandomIt last, Compare comp);

//lambda函数
std::sort(vec.begin(), vec.end(), [](int a, int b) {
    return a > b;  // 降序
});

//稳定排序（不改变相对元素的相对顺序）
std::stable_sort(vec.begin(), vec.end());

//部分排序（排序前N个元素）
std::partial_sort(vec.begin(), vec.begin() + N, vec.end());
```



