---
layout: post
title: LeetBook《高级算法》题解
subtitle: Answers to LeeBook "Advanced Algorithm"
tags: [code]
---

<style> 
  img{ 
     width: 60%; 
     padding-left: 20%; 
  } 
</style>

快一个月没有更新博客了，最近身体状态一直不太好，研究课题和其他方面的学习都停滞了很久。不过随着气温的逐渐升高，精气神也应该提上来才对。

我最近正在做LeetCode上的LeetBook[《高级算法》](https://leetcode-cn.com/leetbook/detail/top-interview-questions-hard/)中的题目，里面题目根据涉及到的知识点分为：数组和字符串，链表，树和图，回溯算法，排序和搜索，动态规划，数学，设计问题以及其他。题目的整体难度比较高，我想如果全都做一遍对水平应该有比较大的提升。

这篇博客里记录了其中部分题目（都是第一时间没能做出来的）的题解，会随着做题的进度持续更新。

（说起来，我想做个目录方便以后查阅，不过目前还没搞懂该怎么做。）

## 数组和字符串


### NO. 289 生命游戏

**题目描述：**
给定一个包含 m × n 个格子的面板，每一个格子都可以看成是一个细胞。每个细胞都具有一个初始状态： 1 即为 活细胞 （live），或 0 即为 死细胞 （dead）。每个细胞与其八个相邻位置（水平，垂直，对角线）的细胞都遵循以下四条生存定律：
1. 如果活细胞周围八个位置的活细胞数少于两个，则该位置活细胞死亡;
   
2. 如果活细胞周围八个位置有两个或三个活细胞，则该位置活细胞仍然存活；

3. 如果活细胞周围八个位置有超过三个活细胞，则该位置活细胞死亡；

4. 如果死细胞周围正好有三个活细胞，则该位置死细胞复活；

下一个状态是通过将上述规则同时应用于当前状态下的每个细胞所形成的，其中细胞的出生和死亡是同时发生的。给你 m x n 网格面板 board 的当前状态，返回下一个状态。


**题解：**

生命游戏其实就是一个很经典的元胞自动机，每个细胞的下一个时间点的状态完全由其自身已经邻域的当前状态来决定。这道题的难点在于，如何采用原地算法来解决这道问题（也就是不能借助额外的空间）。

细胞的状态转换只有四种：0-0，0-1，1-0以及1-1。由于需要原地修改细胞的状态，并且不能让修改后的状态影响到其他细胞的状态判定，所以对于0-1和1-0这两种转换，可以先把细胞的值改为一个特定的临时值（比如0-1对应的临时值设为2，1-0对应3，这样对临时值做对2取余的操作后就可以得到他们原本的状态）。等遍历完所有细胞之后再把这个临时值改回去。

然后很容易就能写出代码啦：

```c++
class Solution {
public:
    int neighbors[3] = {0, 1, -1};
    int m, n;

    void gameOfLife(vector<vector<int>>& board) {
        m = board.size();
        n = board[0].size();

        // 遍历面板每一个格子里的细胞
        for (int row = 0; row < m; row++) 
        {
            for (int col = 0; col < n; col++) 
            {
                int livecount = 0;
                //计算活细胞数量
                for(int i = 0; i < 3; i++)
                {
                    for(int j = 0; j < 3; j++)
                    {
                        if(i == 0 && j == 0)
                            continue;
                        int r2 = row + neighbors[i];
                        int c2 = col + neighbors[j];
                        if(r2 < 0 || c2 < 0 || r2 >= m || c2 >= n)
                            continue;
                        if(board[r2][c2] %2 == 1)
                            livecount++;
                    }
                }

                //按照规则判定
                if ((board[row][col] == 1) && (livecount < 2 || livecount > 3)) 
                    // 活变死
                    board[row][col] = 3;
                

                if (board[row][col] == 0 && livecount == 3) 
                    // 死变活
                    board[row][col] = 2;
                
            }
        }
        for (int row = 0; row < m; row++) 
        {
            for (int col = 0; col < n; col++) 
            {
                if (board[row][col] == 3) 
                    board[row][col] = 0;
                else if(board[row][col] == 2)
                    board[row][col] = 1;
                
            }
        }
    }
};


```


### NO. 239 滑动窗口的最大值 
**题目描述：**

给你一个整数数组 nums，有一个大小为 k 的滑动窗口从数组的最左侧移动到数组的最右侧。你只可以看到在滑动窗口内的 k 个数字。滑动窗口每次只向右移动一位。

返回 滑动窗口中的最大值 。


**题解：**

我感觉很多涉及到“最大值”“最小值”字眼的题目都可以考虑用优先队列来求解，这道题也是如此。

我们首先将第一个滑动窗口中的值存入优先队列（大顶堆）。优先队列中存储的类型是`pair<int,int>`，其中pair.first是元素值，而pair.second是该元素在数组里的位置。随后，窗口每向右滑动一格，就将新加入的元素push进优先队列，同时检查优先队列的顶端元素，将坐标不在滑动窗口范围内的全pop出来。由于优先队列中的元素是按照值从大到小的顺序排列的，每次更新后的优先队列的顶端元素一定是滑动窗口中的最大值。

代码如下：

```c++
class Solution {
public:
    vector<int> maxSlidingWindow(vector<int>& nums, int k) {
	//建立优先队列
        priority_queue<pair<int,int>,vector<pair<int,int>>,func> q;
        vector<int> rtn;
        int maxval = -10001;
		
        if(nums.size() < k)
        {
            for(int it: nums)
                maxval = max(maxval,it);
            rtn.push_back(maxval);
            return rtn;
        }
		
		//首先得出第一个滑动窗口的最大值
        int i = 0;
        for(; i < k; i++)
        {
            q.push({nums[i],i});
        }
        auto tmp = q.top();
        maxval = tmp.first;
        rtn.push_back(maxval);
        
		//滑动窗口不断地向右移动，每移动一格，检查一下优先队列的顶端
		while(i < nums.size())
        {
            while(!q.empty() && q.top().second <= i-k)
                q.pop();
			//把新加入的元素push进优先队列
            q.push({nums[i],i});
			
			//获取优先队列的顶端值
            rtn.push_back(q.top().first);
            i++;
        }
        return rtn;
        
    }

    struct func
    {
        bool operator() (pair<int,int> a, pair<int,int> b) 
        {
            return a.first <= b.first;
        }
    };
};
```


## 树和图

### NO.124 二叉树中的最大路径和

**题目描述**

路径 被定义为一条从树中任意节点出发，沿父节点-子节点连接，达到任意节点的序列。同一个节点在一条路径序列中 至多出现一次 。该路径 至少包含一个 节点，且不一定经过根节点。

路径和 是路径中各节点值的总和。

给你一个二叉树的根节点 root ，返回其 最大路径和 。

以下是一个例子：

![enter description here](../assets/2022-03-30/binary_tree_sum.jpg)

最大路径用红色的线标识，最大路径和为15+20+7=42。


**题解：**

这道题可以用递归的方法来解。假设我们想要求解以某一个节点为根节点的最大路径和，我们可以分别求解 以该节点的左子节点为根节点的最大路径和 以及 以该节点的右子节点为根节点的最大路径和。将这二者与该节点的值相加就可以得出结果（需要注意的是，当某些节点对应的最大路径和小于0时，就舍弃这个节点）。

**题解：**

```c++
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode() : val(0), left(nullptr), right(nullptr) {}
 *     TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
 *     TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
 * };
 */
class Solution {
private:
    int maxsum = INT_MIN;

public:
    int getmaxsum(TreeNode* node) 
	{
        if (node == nullptr) {
            return 0;
        }
		//如果子树的最大路径和小于0，那么舍弃这部分的结果
        int leftval = max(getmaxsum(node->left), 0);
        int rightval = max(getmaxsum(node->right), 0);

        // 获得以节点node为根节点的最大路径和
        int curmaxsum = node->val + leftval + rightval;

        // 更新最大值
        maxsum = max(maxsum, curmaxsum);

        // 返回该节点对应的最大贡献，需要注意的是只能选取左右子树中的一支，因为如果有分叉的话，整体的路径就连不起来了
        return node->val + max(leftval, rightval);
    }

    int maxPathSum(TreeNode* root)
	{
        int tmp = getmaxsum(root);
        return maxsum;
    }
};

```


-----
待续