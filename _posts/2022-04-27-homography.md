---
layout: post
title: 单应变换
subtitle: Homographic transformation
tags: [math]
---


<head>
    <script src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML" type="text/javascript"></script>
    <script type="text/x-mathjax-config">
        MathJax.Hub.Config({
            tex2jax: {
            skipTags: ['script', 'noscript', 'style', 'textarea', 'pre'],
            inlineMath: [['$','$']]
            }
        });
    </script>
</head>


<style> 
  img{ 
     width: 60%; 
     padding-left: 20%; 
  } 
</style>


假设我们从两个不同的视角拍摄同一个物体的照片，那么这个物体在两张照片中所显示的形状是不一样的。举一个很简陋，啊不，简单的例子：假设桌子上放了一本书，那么从书的正上方俯视这本书，书的形状应该是一个矩形（下图左侧）；但是如果我们蹲在桌子前看向这本书（类似于平视），那么此时我们看到的书的形状应该类似于一个梯形，离得远的那条边看上去会小一些（下图右侧）。


![enter description here](../assets/2022-04-27/exp1.png)

虽然我们看向的是同一本书，但是由于视角的变化，我们看见的书的形状不同。单应变换就是将俯视图里的点映射到平视图里的对应位置（反过来也行）的过程，也就是从一个视角映射到另一个视角的过程。所以单应变换也可以称为视角变换（perspective transformation）。

单应变换可以用矩阵乘法实现，可以把它理解为采用其次坐标系的仿射变换。仿射变换就是线性变换+平移。这里提一下两种变换的重要的性质：

- 两条平行的线在经过仿射变换之后依旧是平行的。
  
- 在同一条线上的点经过单应变换后依旧在同一条线上。
 
 举个例子，二维的仿射变换是这样的：

$$\left[ \begin{matrix}
a_{11} & a_{12} & a_{13} \\
a_{21} & a_{22} & a_{23} \\
0         & 0          & 1
\end{matrix} \right]
*
\left[ \begin{aligned}
x_1 \\
y_1 \\
1\\
\end{aligned} \right] = 
\left[ \begin{aligned}
x_2 \\
y_2 \\
1\\
\end{aligned} \right]
$$

而单应变换是这样的：

$$\left[ \begin{matrix}
a_{11} & a_{12} & a_{13} \\
a_{21} & a_{22} & a_{23} \\
a_{31} & a_{32} & 1
\end{matrix} \right]
*
\left[ \begin{aligned}
x_1 \\
y_1 \\
1\\
\end{aligned} \right] = 
\left[ \begin{aligned}
u \\
v \\
w\\
\end{aligned} \right] = \frac{1}{w}
\left[ \begin{aligned}
x_2 \\
y_2 \\1\\
\end{aligned} \right]
$$

关于单应变换矩阵为什么长这样，可以参考[这篇博客](https://towardsdatascience.com/understanding-homography-a-k-a-perspective-transformation-cacaed5ca17)，里面很详细地介绍了单应变换矩阵是如何产生的，这里就不做赘述了。


## 求解单应变换矩阵

我们可以发现，3×3的单应变换矩阵中有8个未知数，那么解得他们至少需要8个等式，因此，我们需要至少4个点在变换前后的$(x,y)$坐标。如果我们知道一张图片的四个顶点的坐标的话，那么正好能够进行求解。

将上面提到的等式的左侧展开，可以得到如下的结果：

$$\begin{align}
x_1*a_{11}+y_1*a_{12}+a_{13} = u \\
x_1*a_{21}+y_1*a_{22}+a_{23} = v\\
x_1*a_{31}+y_1*a_{32}+ 1 = w
\end{align}
$$

由于$x_1 = \frac{u}{w}$，$x_2 = \frac{v}{w}$，我们可以将前2个等式的左右两侧同时除以w，得到：

$$
\frac{x_1*a_{11}+y_1*a_{12}+a_{13}}{x_1*a_{31}+y_1*a_{32}+ 1} = x_2 \\
\frac{x_1*a_{21}+y_1*a_{22}+a_{23} }{x_1*a_{31}+y_1*a_{32}+ 1}= y_2
$$

同时乘以分母，再将一些元素移到左边，就可以得到：

$$
x_1*a_{11}+y_1*a_{12}+a_{13} - x_1x_2*a_{31} - x_2y_1*a_{32} = x_2 \\
x_1*a_{21}+y_1*a_{22}+a_{23} - x_1x_2*a_{31} - x_2y_1*a_{32} = y_2
$$

这样整理一下，就可以写出$Ax=b$的矩阵乘法形式了：

$$
\left[ \begin{matrix}
x_{11} &y_{11}&1&0&0&0&-x_{11}x_{12}&-x_{12}y_{11}\\
0&0&0&x_{11} &y_{11}&1&-x_{11}x_{12}&-x_{12}y_{11}\\
x_{21} &y_{21}&1&0&0&0&-x_{21}x_{22}&-x_{22}y_{21}\\
0&0&0&x_{21} &y_{21}&1&-x_{21}x_{22}&-x_{22}y_{21}\\
x_{31} &y_{31}&1&0&0&0&-x_{31}x_{32}&-x_{32}y_{31}\\
0&0&0&x_{31} &y_{11}&1&-x_{31}x_{32}&-x_{32}y_{31}\\
x_{41} &y_{41}&1&0&0&0&-x_{41}x_{42}&-x_{42}y_{41}\\
0&0&0&x_{41} &y_{41}&1&-x_{41}x_{42}&-x_{42}y_{41}
\end{matrix}\right]*
\left[\begin{matrix}
a_{11}\\a_{12}\\a_{13}\\a_{21}\\a_{22}\\a_{23}\\a_{31}\\a_{32}
\end{matrix}\right] = \left[\begin{matrix}
x_{12}\\y_{12}\\x_{22}\\y_{22}\\x_{32}\\y_{32}\\x_{42}\\y_{42}
\end{matrix}\right]
$$

对上述的方程组进行求解，就可以得出单应变换矩阵$H$。

需要注意的是，单应变换是针对于二维坐标的，但有的时候我们处理的坐标系是三维的（将$z$值固定），这个时候对应的4×4的变换矩阵应该长这样：

$$
H=\left[\begin{matrix}
a_{11} & a_{12}&0&a_{13}\\
a_{21} & a_{22}&0&a_{23}\\
0 & 0&1&0\\
a_{31} & a_{32}&0&1
\end{matrix}\right]
$$

要将$z$轴对应的行和列给空出来。

## Reference

- [Understanding Homography](https://towardsdatascience.com/understanding-homography-a-k-a-perspective-transformation-cacaed5ca17)
