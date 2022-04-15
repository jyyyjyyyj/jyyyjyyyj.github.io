---
layout: post
title: 单调栈
subtitle:  Monotonic stack
tags: [code]
---

本篇博客的内容是单调栈。顾名思义，单调栈就是元素单调排序的栈，分为单调递增栈和单调递减栈。

当往栈中存入新元素时，为了维护栈的单调性，对于单调递增栈，如果栈顶元素大于新元素，那么要不断地弹出栈顶元素直到栈空了或者栈顶元素小于新元素。对于单调递减栈则是弹出小于新元素的栈顶元素。


在之前的[这篇博客](https://jyyyjyyyj.github.io/2022-04-08-advanced_algorithm2/)中曾经提到了两道用单调栈解决的题目：一个是[接雨水](https://leetcode-cn.com/problems/trapping-rain-water/solution/)，另一个是[柱状图中最大的矩形](https://leetcode-cn.com/problems/trapping-rain-water/solution/)。做完这两道题之后我感觉还是不太会用单调栈（主要是不知道到底什么情况下可以用），所以这篇博客里记录了一些有关单调栈的问题的题解，通过找一找这些问题的共性来熟悉单调栈的使用方法。


## NO. 316 去除重复字母

**题目：**

给你一个字符串 s ，请你去除字符串中重复的字母，使得每个字母只出现一次。需保证 返回结果的字典序最小（要求不能打乱其他字符的相对位置）。

例如：

输入： “bcabc”
输出：“abc”

**题解：**

题目的需求是删除重复的字母，且保证删除之后的字符串是所有可能结果中字典序最小的。这道题可以用贪心算法+单调栈来解（虽然不看题解完全想不到就是了）。

首先可以用一个长度为26的数组记录下每个字母出现的次数，这样就能知道需要删除哪些字母。其次，为了保证结果是字典序最小，那么肯定需要尽量删除排序靠前的，且值较大的元素。我们在遍历字符串的时候，用一个单调递增栈存储遍历过的字符。当栈顶元素a大于当前遍历到的元素b，且元素a在未被遍历到的部分还会出现（说明a可以被删掉），那么就将a删掉，并且在最初建立的数组中将a对应的值减一。

于此同时，我们需要注意的是栈中不可以出现重复的元素，因此还需要另一个长度为26的数组来记录栈中元素的出现情况。

代码如下：

```c++
class Solution {
public:
    string removeDuplicateLetters(string s) {
        vector<int> vis(26), num(26);

        //记录每个单词出现的次数
        for (char ch : s) {
            num[ch - 'a']++;
        }

        //单调递增栈
        string stk;
        for (char ch : s) {
            //栈中是否有这个字符
            if (!vis[ch - 'a']) {
                while (!stk.empty() && stk.back() > ch) {
                    //栈顶元素比当前元素大
                    //且该元素后面还会出现
                    //就把他删了
                    if (num[stk.back() - 'a'] > 0) {
                        vis[stk.back() - 'a'] = 0;
                        stk.pop_back();
                    } else {
                        break;
                    }
                }
                //将当前字符push进去
                vis[ch - 'a'] = 1;
                stk.push_back(ch);
            }
            //出现次数-1
            num[ch - 'a'] --;
        }
        return stk;
    }
};
```



## NO.321 拼接最大数

**题目：**

给定长度分别为 m 和 n 的两个数组，其元素由 0-9 构成，表示两个自然数各位上的数字。现在从这两个数组中选出 k (k <= m + n) 个数字拼接成一个新的数，要求从同一个数组中取出的数字保持其在原数组中的相对顺序。

求满足该条件的最大数。结果返回一个表示该最大数的长度为 k 的数组。

一个例子：

输入：
nums1 = [3, 4, 6, 5]， nums2 = [9, 1, 2, 5, 8, 3]， k = 5

输出: [9, 8, 6, 5, 3]


**题解：**

题目需要从两个数组里选出共k个数字，并将他们按照原顺序拼接，得到最大值。

假设我们从nums1中选取i个数字，那么从nums2中要选取(k-i)个数字。从单个数组中选数字的过程可以采用单调递减栈来解决，当当前元素大于栈顶元素时，就弹出栈顶元素。在维护单调递减栈的过程中，需要保证栈中的元素个数大于等于我们从nums1（或nums2）中选取的元素的个数，这个可以通过统计栈中已经被删除的元素数量来实现，比如对于nums1来说，我们共需要删掉nums1.size() - i 个数字。如果在遍历结束之前就删够了元素，那么就接受所有未被遍历到的元素，并退出循环，如果遍历结束之后还没有删够，那么只返回从栈底往上数的前i个元素。

这一步结束之后，我们已经获取了k个数字，接下来需要将他们合并，在保持原顺序的同时获得最大值。这个合并类似于归并排序中的合并过程，但是如果双方数组里有一模一样的元素，那么需要通过判断后续元素的大小来决定优先选择哪一个。

代码有些长，需要写多个函数，包括：单调栈选取元素，合并元素，以及当合并时遇到相同的元素需要优选选择哪一个的判断函数。

```c++
class Solution {
public:
    vector<int> maxNumber(vector<int>& nums1, vector<int>& nums2, int k) {
        vector<int> rtn(k,0);
        int m = nums1.size();
        int n = nums2.size();
        
        //需要从nums1中选择的最少的数字
        int minnum = max(0,k-n);
        //最多
        int maxnum = min(k,m);
        //从nums1中选取i个数字，从nums2中选取k-i个
        
        for(int i = minnum; i <= maxnum; i++)
        {
            vector<int> sub1 = select_num(nums1,i);
            vector<int> sub2 = select_num(nums2,k-i);
            vector<int> cur = merge(sub1,sub2);
            //比较二者的值
            if(comp(cur,rtn))
                rtn = cur;
        }
        return rtn;
    }

    vector<int> select_num(vector<int> nums, int k)
    {
        if(k == 0)
            return {};
        //单调递减栈
        vector<int> stk;

        //需要删除的数字
        int cnt = nums.size() - k; 

        for (int it : nums) 
        {
            //栈非空，数字还没删够，且栈顶元素小于当前元素
            while (cnt != 0 && !stk.empty() && it > *stk.rbegin()) 
            {
                stk.pop_back();
                cnt--;
            }
            stk.push_back(it);
        }

        //如果还没删完
        while (cnt != 0) {
            stk.pop_back();
            cnt--;
        }
        return stk;
    }
    
    //合并数组
    vector<int> merge(vector<int>& sub1, vector<int>& sub2) {
        if(sub1.size() == 0)
            return sub2;
        if(sub2.size() == 0)
            return sub1;
        vector<int> rtn;
        int st1 = 0,st2 = 0;
        while(st1 < sub1.size() && st2 < sub2.size())
        {
            if(sub1[st1] > sub2[st2])
            {
                rtn.push_back(sub1[st1]);
                st1++;
            }
            else if(sub1[st1] < sub2[st2])
            {
                rtn.push_back(sub2[st2]);
                st2++;
            }
            else //注意，如果此时两个数组中的元素相等，需要进行额外的判断
            {
                if(comp2(sub1,sub2,st1,st2))
                {
                    rtn.push_back(sub1[st1]);
                    st1++;
                }
                else
                {
                    rtn.push_back(sub2[st2]);
                    st2++;
                }
            }
        }
        while(st1 < sub1.size())
        {
            rtn.push_back(sub1[st1]);
            st1++;
        }
        while(st2 < sub2.size())
        {
            rtn.push_back(sub2[st2]);
            st2++;
        }
        return rtn;
    }

    bool comp(vector<int> cur, vector<int> rtn) {
        int k = cur.size();
        for(int i = 0; i < k; i++)
        {
            if(cur[i] > rtn[i])
                return true;
            else if(cur[i] < rtn[i])
                return false;
        }
        return false;
    }

    //当合并数组时有两个相等的元素，判断应该选取哪一个
    //返回true则选择sub1中的元素，反之选择sub2中的元素
    //需要比较后面的元素的大小才能决定
    bool comp2(vector<int> sub1, vector<int> sub2, int idx1, int idx2)
    {
        while(idx1 < sub1.size() && idx2 < sub2.size())
        {
            if(sub1[idx1] == sub2[idx2])
            {
                idx1++;
                idx2++;
            }
            else if(sub1[idx1] > sub2[idx2])
                return true;
            else
                return false;
        }
        if(idx1<sub1.size())
            return true;
        return false;
    }
};
```

## NO. 739 最大温度

**题目：**

给定一个整数数组 temperatures ，表示每天的温度，返回一个数组 answer ，其中 answer[i] 是指在第 i 天之后，才会有更高的温度。如果气温在这之后都不会升高，请在该位置用 0 来代替。

一个例子：

输入: temperatures = [73,74,75,71,69,72,76,73]

输出: [1,1,4,2,1,1,0,0]

**题解：**

这道题可以用单调递减栈来解决（比前两道要简单很多）。在遍历temperatures的时候维护一个单调递减栈，栈内存储的是日期（就是temperatures元素对应的下标）。如果当前气温大于栈顶值对应的气温，说明这一天的气温比前面的某几天要高，那么就将小于当前元素的栈顶都pop出来，并算出相应的日期差。

代码如下：

```c++
class Solution {
public:
    vector<int> dailyTemperatures(vector<int>& temperatures) {
        int n = temperatures.size();
        stack<int> stk; //单调递减
        vector<int> rtn(n,0);
        for(int i = 0; i < n; i++)
        {
            while(!stk.empty() && temperatures[stk.top()] < temperatures[i])
            {
                //int cur = stk.top();
                rtn[stk.top()] = i-stk.top();
                stk.pop();
            }
            stk.push(i);
        }
        return rtn;
    }
};
```

----

对比这几道题的题目可以发现，题目要求里都出现了类似于“求最大最小值”、“保留数组原顺序”、“不能打乱相对顺序”的要求。所以如果遇到需要求最大/最小值，且无法进行排序操作的场合，可以考虑采用单调栈来处理。而且如果需要处理的是“最大值”或“较大值”，就用单调递减栈（后两题），反之就用单调递增栈（第一题）。

-----

## Reference

- [去除重复字母](https://leetcode-cn.com/problems/remove-duplicate-letters/)

- [拼接最大数](https://leetcode-cn.com/problems/create-maximum-number/)

- [每日温度](https://leetcode-cn.com/problems/daily-temperatures/)
