---
layout: post
title: 把代码块的外观给改了
subtitle: Changing the syntax highlighting
tags: [others]
---

我终于把代码块的外观给改了！

这个博客的框架是从一位网友那里fork过来的，我很喜欢这种简洁的风格，然而最开始的代码块外观不是特别好看（并没有批评这位老哥的意思）。之前我看不懂这些配置页面布局的文件，所以一直没有改。

但是……这些都是过去式了！今晚我突然心血来潮又想起来这件事情，在google和stack overflow上遨游了半个多小时之后，终于找到了一个很简单的修改办法。我参考的是[这篇博客](https://mycyberuniverse.com/syntax-highlighting-jekyll.html)写的教程。

首先在`_config.yml`里加上以下两行代码：

```
markdown: kramdown
highlighter: rouge
```


然后就是在`css\main.scss`里添加设置代码块风格的代码了。我从[Pygments](https://github.com/richleland/pygments-css)下载了现成的.css文件（怎么可能自己写呢:laughing:），然后把他们放到了css文件夹下。有很多种主题可以选，在[这里](https://richleland.github.io/pygments-css/)可以预览。

在`main.scss`里添加这样一行代码，选了什么主题文件名就写什么，我选的是perldoc：

```
@import url(perldoc.css);
```

这样就大功告成了，终于有赏心悦目的代码块了！不过代码块的背景宽度是固定的，如果某一行代码写得太长的话，可能会超出背景格子，这个需要多注意。

写点代码来看看效果，先写个Cpp：

```c++
void test()
{
	int x = 1, y = 2;
	cout << "Hello World!" << endl;
	return;
}
```

最近在学python，再写个python：

```python
def test(): 
	x, y = 1, 2
	print("Hello World!")  
```


除此之外，我还学会了在markdown里加emoji：:laughing: :smile: :+1:
