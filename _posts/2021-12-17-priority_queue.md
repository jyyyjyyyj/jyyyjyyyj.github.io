---
layout: post
title: C++ 优先队列
subtitle: C++ priority queue
tags: [code]
---

距离有了leetcode排名已经过去了一个多月，这一个多月里我又做了一百道题，排名变成了一个月前的1/2，每天吃完早饭（没错，终于不再是半夜睡觉中午起床了）之后的第一件事就是上号做几道题。然而依旧觉得自己的水平没怎么提升……

![enter description here](./images/img2.png)

于是想多做一些困难题，赫赫，困难题果真是很困难呢！其中很多题都涉及到了优先队列(priority_queue)，所以本篇博客的内容就是优先队列。

优先队列是一种特殊的队列。队列是遵循先进先出原则的，而优先队列中元素的出队顺序是按照优先级排列（比如升序或者降序）。优先队列是由堆来实现的，本质上是一棵完全二叉树。在C++里使用优先队列需要`#include<queue>`。

### 优先队列的定义
大顶堆：`priority_queue<int> queue;`
这是最简单的一种队列的定义，表示值越大的元素优先级越高。如果想要实现小顶堆（值越小优先级越大），那么可以这样定义：
`priority_queue<int,vector<int>,greater<int>> queue;`

其中第一个参数为元素的类型，第二个参数为存放元素的容器类型（必须是数组类型的容器，比如vector，不可以用list），第三个参数为比较函数。元素类型和比较函数可以自己定义。需要注意的是，如果元素类型是自定义的，那么后面的两个参数也必须自定义好，不然C++不知道以什么方式来排序。以下是我从某个网站上找来的一个自定义比较函数示例：
```c++
class mycomparison
{
	bool reverse;
public:
	mycomparison(const bool& revparam=false)
    	reverse=revparam;
	bool operator() (const int& lhs, const int&rhs)
	const{
    	if (reverse) 
			return (lhs>rhs);  //小顶堆
    	else 
			return (lhs<rhs);  //大顶堆
  	}
};

//定义优先队列
priority_queue<int,vector<int>,mycomparison> q1;    //大顶堆
priority_queue<int,vector<int>,mycomparison> q2(mycomparison(true));  //小顶堆
```
上面这段函数可以通过改变`reverse`的值来改变元素的排列方式。需要注意的是，优先队列中用到的进行比较的操作符是"<"，因此在自定义比较函数的时候，可以写成`bool operator()`或`bool operator<`，但是不可以写成`bool operator>`。


### 优先队列的操作
队列中可以使用的函数，在优先队列里都可以使用。优先队列的常用函数包括：
```c++
queue.size();     //返回元素的数量
queue.push(x);    //存入一个元素
queue.pop();      //删除顶端元素
queue.top();      //返回顶端元素
queue.empty();    //返回一个布尔量，表示队列是否为空
```

### 一个例子
以LeetCode第23题，合并k个升序列表为例。该题的输入是k个列表，每个列表内的节点已经按照升序排序，需要将这k个列表合并为一个列表，按照升序排列并输出。这是一道困难题。

解题思路是，首先将每个列表的头结点存入优先队列（按照升序排列），每次取出值最小的那个结点，并到新的列表里，然后将该节点的下一个节点（如果有的话）存入优先队列，一直持续到优先队列中所有的元素都被取完。以下是我写的代码：

```c++
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode() : val(0), next(nullptr) {}
 *     ListNode(int x) : val(x), next(nullptr) {}
 *     ListNode(int x, ListNode *next) : val(x), next(next) {}
 * };
 */
class Solution {
public:
    ListNode* mergeKLists(vector<ListNode*>& lists) {
        priority_queue<ListNode*, vector<ListNode*>, func> queue;
        for(auto &it:lists)
        {
            //获取头节点
            if(it)
                queue.push(it);
        }
        ListNode* head = new ListNode(0);
        ListNode* rtn = head;
        while(!queue.empty())
        {
            ListNode* tmp = queue.top();
            head->next = tmp;
            head = head->next;
            queue.pop();
            if(tmp->next) //这个列表还没有遍历完，将其下一个元素存进来
            {
                queue.push(tmp->next);
            }
        }
        return rtn->next;
    }

    struct func
    {
        bool operator() (ListNode* a, ListNode* b) 
        {
            return a->val > b->val; //小顶堆
        }
    };
};
```


### Reference
- [参考1](http://www.cplusplus.com/reference/queue/priority_queue/priority_queue/)
- [参考2](https://blog.csdn.net/weixin_36888577/article/details/79937886)
- [leetcode23](https://leetcode-cn.com/problems/merge-k-sorted-lists/)