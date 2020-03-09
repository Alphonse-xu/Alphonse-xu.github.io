---
title: Leetcode刷题记录：1.两数之和
date: 2020-02-21 21:55:26
tags:
    - Leetcode
    - 简单
categories:
    - Leetcode 
---

# 题目
给定一个整数数组 nums 和一个目标值 target，请你在该数组中找出和为目标值的那 两个 整数，并返回他们的数组下标。你可以假设每种输入只会对应一个答案。但是，你不能重复利用这个数组中同样的元素。

## 解法

1. 暴力搜索

遍历每个元素 xxx，并查找是否存在一个值与 target−xtarget - xtarget−x 相等的目标元素。

执行用时：136ms

内存消耗：9.4mb
~~~
class Solution {
public:
    vector<int> twoSum(vector<int>& nums, int target) {
        for(int i = 0;i < nums.size() - 1; i++)
        {
            int side = target - nums[i];
            for(int j = i + 1; j < nums.size(); j++)
            {
                if(nums[j] == side)
                {;
                    return {i,j};
                }
            }
        }
        return {};
    }
};
~~~

复杂度分析：
~~~
时间复杂度：O(n^2)
对于每个元素，我们试图通过遍历数组的其余部分来寻找它所对应的目标元素，这将耗费 O(n) 的时间。因此时间复杂度为 O(n^2)

空间复杂度：O(1)
~~~

2. 两遍哈希表查找

通过以空间换取速度的方式，我们可以将查找时间从 O(n)O(n)O(n) 降低到 O(1)O(1)O(1)。哈希表正是为此目的而构建的，它支持以 近似 恒定的时间进行快速查找。我用“近似”来描述，是因为一旦出现冲突，查找用时可能会退化到 O(n)O(n)O(n)。但只要你仔细地挑选哈希函数，在哈希表中进行查找的用时应当被摊销为 O(1)O(1)O(1)。

~~~
class Solution {
public:
    vector<int> twoSum(vector<int>& nums, int target) {
        unordered_map<int,int> numMap;
        for(int i = 0;i < nums.size();i++)
        {
            numMap[nums[i]] = i;
        }

        for(int i = 0;i < nums.size();i++)
        {
            if(numMap.find(target - nums[i]) != numMap.end() && numMap[target - nums[i]] != i)
            {
                return {i,numMap[target - nums[i]]};
            }
        }
        return {};
    }
};
~~~

执行用时：136ms

内存消耗：9.4mb

复杂度分析：
~~~
时间复杂度：O(n)，
我们把包含有 n 个元素的列表遍历两次。由于哈希表将查找时间缩短到 O(1)，所以时间复杂度为 O(n)

空间复杂度：O(n)
所需的额外空间取决于哈希表中存储的元素数量，该表中存储了 n 个元素。
~~~

方法三：一遍哈希表

事实证明，我们可以一次完成。在进行迭代并将元素插入到表中的同时，我们还会回过头来检查表中是否已经存在当前元素所对应的目标元素。如果它存在，那我们已经找到了对应解，并立即将其返回。

执行用时：136ms

内存消耗：9.4mb

~~~
class Solution {
public:
    vector<int> twoSum(vector<int>& nums, int target) {
        unordered_map<int,int> m;
        
        for(int i = 0;i < nums.size();i++)
        {
            int side = target - nums[i];
            if(m.find(side) != m.end())
            {
                return {m[side], i};
            }
            m[nums[i]] = i;
        }
        return {};
    }
};
~~~

还有一种貌似用时更快的写法

~~~
class Solution {
public:
    vector<int> twoSum(vector<int>& nums, int target) {
        unordered_map<int, int> m;
        vector<int> res;
        for(int i = 0; i< nums.size(); ++i){
            if(m.find(target-nums[i])==m.end()){
                m[nums[i]]=i;
            }
            else{
                res.push_back(m[target-nums[i]]);
                res.push_back(i);
                break;
            }
        }
        return res;
    }
};
~~~

复杂度分析：
~~~
时间复杂度：O(n)
我们只遍历了包含有 n 个元素的列表一次。在表中进行的每次查找只花费 O(1)O(1)O(1) 的时间

空间复杂度：O(n)
所需的额外空间取决于哈希表中存储的元素数量，该表最多需要存储 n 个元素
~~~

# 总结
哈希表在内存消耗有限增大的情况下大大提高了查找速度。大数据快查还是用哈希表较好。