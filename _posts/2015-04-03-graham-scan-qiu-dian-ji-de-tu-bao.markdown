---
layout: post
title:  "Graham Scan 求点集的凸包"
date: 2015-04-03 20:06:37 +0800
comments: true
categories: 算法相关
description: 本文总结如何求给定点集的凸包。
---

## 问题概述

凸包的定义很好理解，不过我们要理解的深刻一点，下面给出两种理解

### 直观的定义

{% img center /images/concrete_of_convex_hull.png %}

把给定的点集想象成钉在板子上的钉子，我们取来一根橡皮筋，然后将它撑开并围住所有的钉子，松手，啪的一声，橡皮筋将紧绷到钉子上，它的总长度也达到了最小，这个时候，如图所示，由橡皮筋所围住的区域就是给定点集的凸包。

### 抽象的定义

这里的例子来自刘汝佳和黄亮老师的《算法艺术与信息学竞赛》,现在假设你有一些金银合金，合金A，合金B，合金C，很多，并且每种合金的含金量和含银量也都各不相同，现在你可以通过按照某种比例来对这些合金进行混合，来制造出新的合金，那么制造出的这些合金的含金量和含银量在什么范围内？

好像和凸包没什么关系，请接着看下去。

我们先考虑上面问题的简化版

#### 单纯考虑含金量

显然，新制造出的合金的含金量只能在原来的合金的含金量范围之内，即在原先的最大含金量和最小含金量之间。

#### 加上含银量

加上含银量之后，我们先假设原先只有两种合金，合金A和合金B，那么，设原来合金A含金量为$x_A$,含银量为$y_A$,合金B的含金量为$x_B$,含银量为$y_B$，那么新制造的合金$C$我们表示为：$C = \lambda{A} + (1-\lambda)B$ , 那么

$$\begin{equation}
  \begin{cases}
    x_C = \lambda{x_A} + (1 - \lambda)x_B\\
y_C = \lambda{y_A} + (1 - \lambda)y_B\\
  \end{cases}
\end{equation}$$

其中，$\lambda \in \[ 0,1\]$ ，从几何上讲，$C$必然在线段内(是$A,B$的内分点，包括端点)，那么如果原来是三种合金，合金A，合金B，合金C，那么，新合金则必然在$\Delta{ABC}$内，推广到一般情况，如果原来有$N$种合金，则新合金则必然在这些点的凸包内。

上述把原先的两种合金推广到$N$种合金是在点的多少上推广，但始终是二维的平面，如果我们不限定只是两种金属的纯度，而是$M$种金属的纯度，即合金可以含有$M$种金属，那么原先有$N$种合金，每种合金$i$中金属$j(j = 1 ~ M)$的含量百分比为$x_{i,j}$，这个时候，便把二维上的问题推广到了$M$维。

这个时候，关于凸包的定义大概清楚了一些，显然对于凸包，凸包中的任意两个点的连线都在凸包内，并且凸包上的顶点比较特出，称它为极点，极点不能被凸包内部的点按照某种比例加权表示，通俗点，**原有合金不能被其他合金混合而成**，显然，上述列子中的**原有合金**便是凸包上的极点，即**凸包上的顶点是原点集中的点**。

关于凸包的应用也是很多的，例如我们求**点集的直径**，即点集中任意两点之间距离的最大值，有了点集的凸包，这个时候我们可以不考虑凸包内部的点，降低问题的时空复杂度。

## 求凸包

求凸包有很多方法，但是已得到证明，对于$N$个顶点，算法的时间下界是$O(N\log{N})$,本文只对**Graham Scan**算法做一总结，它的时间复杂度为$O(N\log{N})$。

### 算法描述

下面是算法的描述

 1. 在点集中找到最低的点$Points[0]$，如果两个点一样低（即它们的$Y$坐标相同而$X$坐标不同），我们取$X$值较小的那个。
 2. 对剩下的$N-1$个点，将这些点排好序，排序的依据是这些点各自按照逆时针与点$Points[0]$所成的极角的大小，如果两个点与$Points[0]$所成的极角大小相同，那么我们取靠近$Points[0]$的那个点，让它靠前。
 3. 初始化栈$S$，并且让$Points[0],Points[1],Points[2]$入栈。
 4. 对于剩下的$N-3$个点，对于每个点$Points[i]$，我们不断地做以下操作：考察下面的三个点，如果依次连起来不是**逆时针**的话(换句话说，他们没有造成**左转**)，那么就弹出栈顶元素。这三个点分别是，**靠近栈顶的那个点，栈顶的点**，$Points[i]$。然后，将点$Points[i]$压栈。
 5. 算法结束，此时将栈内的点依次连起来便可得到凸包。

### 具体过程

整个过程可以分为两步，第一步，对点按照指定的顺序排好。第二步，依次筛选点，决定点是入栈还是出栈。

#### 对点排序

如下图所示，我们首先确定一个起始的最低点（这个点一定是凸包的极点），然后我们将这些点按照顺序依次排列起来，排好之后我们可以看到这些点构成了一个闭合的回路。
在对这些点进行排序的时候，虽然我们是按照点$i$与起始点所成的极角大小做排序，但是我们没有必要算出来具体的极角大小，因为我们只是要一个相对的大小关系用来排序而已，具体过程在下文代码中会有体现。

{% img center /images/ven_pic.bmp %}

#### 对点筛选

当我们有了这个闭合的回路之后，我们接下来要做的就是沿着这条闭合的回路进行遍历，在遍历的过程中我们会不断考察上述算法描述的那三个点，最终留在栈中的点就是凸包的极点，而其余的就是内点。

这个算法的实现其实是两层循环(见下面代码)，外层循环负责沿着回路遍历，遍历中的每次我们都会将$Poins[i]$入栈，内层循环则是不断对上述三个点进行考察并出栈的过程，两层循环结束之后，我们便把回路上属于凸包顶点的那些点保留在了栈中。

下面我们给出一个简单的图示来说明这种过程,定义$P$是离栈顶最近的点，$C$是当前栈顶的点，$N$是准备入栈的点。

{% img center /images/ven_1.bmp %}
这个时候，$(P,C,N)$构成了逆时针(左转)，于是接受$C$

{% img center /images/ven_2.bmp %}
$(P,C,N)$构成了顺时针(右转)，因此拒绝接受$C$。

{% img center /images/ven_3.bmp %}
这个时候，$(P,C,N)$构成了逆时针(左转)，于是接受$C$

{% img center /images/ven_4.bmp %}
这个时候，$(P,C,N)$构成了逆时针(左转)，于是接受$C$

{% img center /images/ven_5.bmp %}
$(P,C,N)$构成了顺时针(右转)，因此拒绝接受$C$。

{% img center /images/ven_6.bmp %}
这个时候，$(P,C,N)$构成了逆时针(左转)，于是接受$C$

### 实现

有了上述的算法流程之后，具体实现也变得很清楚，这里我们用 C# 实现整个过程，下面是主要代码，完整代码在[这里][888]下载

我们实现的过程中首先要解决的就是如何对点进行排序，排序的依据就是该点与初始点所成极角的大小，很自然，我们想到重写 Point 类的 **CompareTo** 方法，之后用**List**存储这些点便可实现排序。

下面是**Compare**的实现，接收两个点，并返回表示大小关系的 **1** 或 **-1**, 其中用到了一个方法 **Orientation**， 该方法接收三个参数（其中第一个参数是初始点），用来判断这三个点形成的是 **左拐**还是**右拐**，如果是左拐，很显然，**p2** 与初始点所成的极角要比 **p1** 的大，注意代码中对极角相同情况的处理。

```C#

        // 比较两点与初始点的极角,Dist计算两点之间距离
        public static int Compare(Point p1, Point p2)
        {
            int ori = Orientation(FirstPoint.Get(), p1, p2);
            if (ori == 0)
                return (Dist(FirstPoint.Get(), p2) >= Dist(FirstPoint.Get(), p1)) ? -1 : 1;
            return (ori == 2) ? -1 : 1;
        }

        // 返回（p,q,r）三点构成的拐向
        // 三点共线 --> 返回0
        // 右拐（顺时针） --> 返回1
        // 左拐 （逆时针）--> 返回2
        public static int Orientation(Point p, Point q, Point r)
        {
            float  val = (q.getY() - p.getY()) * (r.getX() - q.getX())
                - (q.getX() - p.getX()) * (r.getY() - q.getY());
            if (val == 0) return 0;
            return (val > 0) ? 1 : 2;
        }

```

好了，对点排序完成之后，剩下的就是如何遍历并做入栈出栈的操作了，遵循我们上文中的描述，下面是这个过程的具体实现

```C#

        public static List<Point> Convexhull(List<Point> pLists)
        {
            //首先确定初始点
            float yMin = pLists[0].getY();
            int minIndex = 0;
            foreach (Point p in pLists)
            {
                float y = p.getY();

                if ((y < yMin) || (yMin == y && p.getX() == pLists[minIndex].getX()))
                {
                    yMin = y;
                    minIndex = pLists.IndexOf(p);
                }
            }

            Point FP = FirstPoint.Create(pLists[minIndex].getX(), pLists[minIndex].getY());
            pLists.RemoveAt(minIndex);

            pLists.Sort();

            Stack<Point> S = new Stack<Point>();

            MyStack ST = new MyStack(S);
            ST.getStack().Push(FP);
            ST.getStack().Push(pLists[0]);
            ST.getStack().Push(pLists[1]);


            //处理剩下的 N-3 个点
            for (int i = 2; i < pLists.Count(); i++)
            {
                while ( PointUtil.Orientation( ST.nextToTop(), ST.getStack().Peek(), pLists[i]) != 2 )
                    ST.getStack().Pop();
                ST.getStack().Push(pLists[i]);
            }

            List<Point> retList = ST.getStack().ToList();
            retList.Sort();

            return retList;
        }

```

我们首先找到初始点，也就是最低（y坐标最小）的点，如果两点 y 坐标相同，则我们取 x 值较小的，之后我们将剩下的点排序之后便进行遍历的操作，每次遍历我们都会考察那三个点所成的拐向，最后返回点集，这些点是凸包上的极点，至此，算法结束。


 


  [888]: https://github.com/hengyicai/MyConvexHull.git
