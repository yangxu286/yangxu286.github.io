---
title: Leetcode Day04：贪心+动态规划
date: 2025-05-22 18:45:00 +0800
tags: [算法, Leetcode, 贪心, 动态规划]
categories: [算法]
---
注：为随手写的刷题笔记，请谨慎参考，欢迎指出错漏。
### 贪心（续）

#### 763.划分字母区间

和跳跃游戏有谜之相似之处，只要求出每个字母的最右下标，那么新遇到这个字母时，就知道这一段的右端起码必须延伸到这个最右下标那边。那么，持续维护当前的最右下标即可。

```c++
class Solution {
public:
    vector<int> partitionLabels(string s) {
        int rightest[26] = {0};
        int len = s.length();
        for(int i = len - 1; i >= 0; i--){
            if(rightest[s[i] - 'a'] == 0){
                rightest[s[i] - 'a'] = i;
            }
        }
        bool visited[26] = {0};
        vector<int> res;
        int shortestDividePoint = 0;
        int curFragLength = 0;
        for(int i = 0; i < len; i++){
            curFragLength++;
            if(!visited[s[i] - 'a']){
                shortestDividePoint = max(shortestDividePoint, rightest[s[i] - 'a']);
                visited[s[i] - 'a'] = true;
            }
            if(shortestDividePoint == i){
                res.push_back(curFragLength);
                curFragLength = 0;
            }
        }
        return res;
    }
};
```

### 动态规划

#### 279.完全平方数&322.零钱兑换

思路很相似且较为容易，主要思路就是f[i] = min(f[i - s[1]] + 1, f[i - s[2]] + 1, ...)。

#### 139.单词拆分

比较普通的思路是通过dp[i]存能否拼成字符串的前i位，每遍历一位，若dp[i - wordDict[j].length()]为true，则匹配后面的一段。官方题解是遍历分割点（长度比字典集的长度短），分别检查前一半（dp[j]）和后一段（使用unordered_set存字典集然后比较）。

虽然能过，但这么做有个隐藏的可优化的空间，那就是对于每一个i都要进行检查。如果从前往后拼，只更新拼成的长度位置的dp为true，对于无法拼成的i就根本不需要进行拼的尝试。

```c++
class Solution {
public:
    bool wordBreak(string s, vector<string>& wordDict) {
        int sLen = s.length();
        int dictLen = wordDict.size();
        vector<bool> dp (sLen + 1, false);
        for(int i = -1; i < sLen - 1; i++){
            if(i != -1 && !dp[i]) continue;
            for(int j = 0; j < dictLen; j++){
                string frag = s.substr(i + 1, wordDict[j].length());
                if(frag == wordDict[j]){
                    dp[i + wordDict[j].length()] = true;
                }
            }
        }
        return dp[sLen - 1];
    }
};
```

