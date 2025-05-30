---
title: Leetcode Day07：哈希+stl迭代器
date: 2025-05-30 21:39:00 +0800
tags: [算法, Leetcode, 哈希]
categories: [算法]
---
接下来lc稳定在一题吧，期末时间真不多了。

#### 128.最长连续序列

放一下我自己写的clown代码，思路是一个数组存某段的长度，一个哈希映射到数组。但如果abc连续，遍历到b，发现a和c对应的数组下标不一样，就要全部更改所属的那一段，假了。

```c++
class Solution {
public:
    int longestConsecutive(vector<int>& nums) {
        vector<int> lengths;
        unordered_map<int, int> hashTable;
        int length = nums.size();
        int res = 1;
        for(int i = 0; i < length; i++){
            if(hashTable.find(nums[i]) != hashTable.end()){
                continue;
            }
            auto left = hashTable.find(nums[i] - 1);
            auto right = hashTable.find(nums[i] + 1);
            int tmp = 1;

            if(left == hashTable.end() && right == hashTable.end()){
                hashTable[nums[i]] = lengths.size();
                lengths.push_back(tmp);
                continue;
            }

            int dayitong = 0;
            if(left != hashTable.end()){
                tmp += lengths[left -> second];
                dayitong = left -> second;
            }
            if(right != hashTable.end()){
                tmp += lengths[right -> second];
                dayitong = right -> second;
            }
            hashTable[nums[i]] = dayitong;
            lengths[dayitong] = tmp;
            res = max(res, tmp);
            cout << nums[i] << ' ' << tmp << endl;
            if(left != hashTable.end()){
                left -> second = dayitong;
            }
            if(right != hashTable.end()){
                right -> second = dayitong;
            }
        }
        return res;
    }
};
```

不过我也看到有人又在我这种想法的基础上优化了一下，只维护两端端点，因为中间的数值也没用了，应该也是可行的，不过用这个方法的人说比较慢。（作业写不完了不然想继续改改的）

真正优雅的做法就不会有这么别扭的感觉，官方题解就是先哈希一遍所有数字（顺便用unordered_set去重），然后再遍历一遍set，判断每个数是不是开头，如果是开头就往下找看有多长。最后返回最大的长度即可。

```c++
class Solution {
public:
    int longestConsecutive(vector<int>& nums) {
        int len = nums.size();
        unordered_set<int> num_set;
        for(int i = 0; i < len; i++){
            num_set.insert(nums[i]);
        }
        int res = 0;
        for(int num : num_set){
            if(!num_set.count(num - 1)){
                int currentNum = num;
                int currentStreak = 1;

                while (num_set.count(currentNum + 1)) {
                    currentNum += 1;
                    currentStreak += 1;
                }

                res = max(res, currentStreak);
            }
        }
        return res;
    }
};
```

发现两个衍生的问题：

#### unordered_set的使用

insert()：插入元素，没啥好说的。

count()：检查指定键是否存在，1是存在，0是不存在。

#### stl的迭代器

在遍历集合的这一行`for(int num : num_set)`这边，int可以改成const int&，但不能改成int&，这是因为unordered_set的迭代器是常量迭代器，不允许我们随意修改，这样才能保证集合的性质。

那么哪些stl用的是常量迭代器呢？我问了claude。

STL中以下容器默认提供常量迭代器（`const_iterator`）或其遍历操作限制修改元素：

#### 关联容器

所有关联容器默认提供常量迭代器，因为修改元素可能破坏其内部有序性或哈希表结构：

1. **有序关联容器**
   - `std::set`
   - `std::multiset`
   - `std::map`（键部分是常量）
   - `std::multimap`（键部分是常量）
2. **无序关联容器**
   - `std::unordered_set`
   - `std::unordered_multiset`
   - `std::unordered_map`（键部分是常量）
   - `std::unordered_multimap`（键部分是常量）

#### 其他提供常量访问的容器

1. 只读视图容器
   - `std::string_view`
   - `std::span`（当用常量构造时）

#### 通用常量迭代器情况

1. **任何容器的`cbegin()`和`cend()`方法**
   - 这些方法显式返回常量迭代器，无论容器本身是否可修改
   - 例如：`vector<int>::const_iterator it = vec.cbegin();`
2. **对常量容器的引用产生的迭代器**
   - 例如：`const vector<int>& vec_ref;`
   - `vec_ref.begin()`将返回常量迭代器

#### 关于map和multimap的特别说明

对于`map`和`multimap`类型的容器，有一个重要细节：

- 它们的元素类型是`pair<const Key, T>`
- 键(`Key`)部分是常量，不能被修改
- 值(`T`)部分可以通过迭代器修改

例如：

```c++
std::map<int, string> m;
auto it = m.begin();
it->second = "新值"; // 可以修改值
// it->first = 10;  // 错误：不能修改键
```

使用常量迭代器是STL设计的重要安全机制，可以防止在遍历过程中破坏容器的内部结构。