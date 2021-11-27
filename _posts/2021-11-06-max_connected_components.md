---
title: 查找二值图像的最大连通量
subtitle: Max connected components in a bitmap
tags: [code]

---

半个月没写博客了，最近一直在忙着改研究课题的代码（不过进度很慢），并没有看什么新的paper，游戏的制作也搁置了一段时间。虽然的确没什么好写的，但是我觉得还是保持博客的更新比较好，所以简单地写一个最近代码里用到的算法吧。

最近在代码里写了一个检测二值图像的最大连通量的功能，最大连通量的计算可以用深度优先搜索（DFS）来实现，简单地来说就是先选择图像里的一个未被访问过的值为1的像素，然后依次访问它的邻域，如果邻域里有未被访问过的，且值为1的像素，那就接着访问这个像素的邻域，直到找不到符合要求的像素为止。这个时候记录一下这个连通域里像素的数量，然后开始新一轮的搜索，直到所有像素都被访问过。最后返回最大的连通量。


代码是用C++写的，待搜索的二值图像存放在`Eigen::MatrixXi bitmap`里，用`Eigen::MatrixXi label`来记录访问情况。如果用`int bitmap[][]`矩阵可能会更快一点。

```C++
#include<Eigen/dense>
#include<vector>
using namespace std;

//邻域的范围
const vector<vector<int>> directions = { {0, 1},{0, -1},{1, 0},{-1, 0} };


//label记录访问情况，bitmap为二值图像，i和j是坐标值
int dfs(Eigen::MatrixXi& label, Eigen::MatrixXi bitmap,int i, int j) {
    //超出图像范围了
	if (i < 0 || j < 0 || i >= label.rows() || j >= label.cols())
	{
		return 0;
	}
	//访问过了
	if (label(i, j) == 1)
	{
		return 0;
	}
	
	//找到一个没有访问过的点，把它标记为已访问
	label(i,j) = 1; // mark visited.
	
	//如果像素值为0
	if (bitmap(i, j) == 0)
	{
		return 0;
	}
	
	//值为1，就把这个像素的值加到连通数量上
	int val= 1;
	for (auto & d : directions) {
	//对这个点的邻域继续进行搜索
		val += dfs(label, bitmap, i + d[0], j + d[1]);
	}
	return val;
}

//return the max connected points number
int max_connented_components(Eigen::MatrixXi bitmap)
{
	int maxval = 0;
	int row = bitmap.rows();
	int col = bitmap.cols();
	Eigen::MatrixXi label(row,col);
	label.setZero();
	//遍历每一个像素
	for (int i = 0; i < row; i++)
	{
		for (int j = 0; j < col; j++)
		{
		    //像素值不为0，且没有访问过
			if (bitmap(i, j) == 1 && label(i, j) == 0)
			{
				int val= dfs(label, bitmap, i, j);
				//记录当前的最大连通量
				maxval = val > maxval ? val : maxval; 
			}
		}
	}
	return maxval;
}

```
原理并不难，leetcode上面也有一些关于DFS的题目和它高度相似，比如“水域大小” “统计封闭岛屿的数目”等等。






<br>
<br>

###2021-11-27更新：
重新看了遍代码，觉得写得太啰嗦了，因为原来的代码里不能修改bitmap，所以新建了一个同样大小的矩阵label来存储访问情况，这样的话耗费的内存比较多。
我重新写了个用vector存储bitmap的版本，bitmap里被访问过的元素直接设为0，这样的话会简洁很多：

```C++
int max_connected_components(vector<vector<int>>& bitmap) 
{
	w=bitmap.size();
	if(!w)
    		return 0;
	h = bitmap[0].size();
	//把访问过的标记为-1
	int maxsz = 0;
	for(int i = 0; i <w;i++)
	{
    		for(int j = 0; j < h; j++)
    		{
			if(bitmap[i][j])
			{
	   			int sz = dfs(i,j,bitmap);
	    			maxsz = max(maxsz,sz);
			}
    		}
	}

	return maxsz;
}

int dfs(int i, int j, vector<vector<int>>& bitmap)
{
	if(i >=w || i<0 || j >=h || j<0)
    		return 0;
	else if(bitmap[i][j] == 1)
	{
    		bitmap[i][j] = 0; //visited
    		return (1+dfs(i-1,j,bitmap)+
    		dfs(i+1,j,bitmap)+
    		dfs(i,j-1,bitmap)+
    		dfs(i,j+1,bitmap));
	}
	return 0;
}
```

## Reference
- [LeetCode 水域大小](https://leetcode-cn.com/problems/pond-sizes-lcci/)
