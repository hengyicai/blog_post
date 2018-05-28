---
layout: post
title:  "二叉查找树"
date: 2015-02-22 16:02:15 +0800
comments: true
categories: 算法相关
tags: [二叉树, 二叉查找树, 数据结构]
keywords: 二叉树, 二叉查找树, 数据结构
description: 二叉查找树的C语言实现以及算法分析
---


本文是对二叉查找树的总结。

## 基本概念

首先二叉查找树是二叉树，回顾一下二叉树的概念，二叉树的每个节点都不能有多于两个的儿子，一般二叉树如下

{% img center /images/Binary_tree.png 450 250 %}

其中，左子树和右子树均可能为空。二叉树很重要的一个性质是平衡二叉树的深度要比**N**小得多，约为$O(\sqrt{N})$。其中**N**是二叉树的节点个数,对于本节的**二叉查找树**，其深度的平均值是$O(\log{N})$。

<!--more-->

**二叉查找树**(**Binary Search Tree**)，也叫做**二叉搜索树**，**有序二叉树**，它在二叉树的基础上多了下面三条性质

- 若某个节点的左子树不空，则左子树上所有节点的值均小于或等于它的根节点的值。
- 若某个节点的右子树不空，则右子树上所有节点的值均大于或等于它的根节点的值。
- 任意节点的左、右子树也分别为二叉查找树。
- 没有键值相等的节点，**no duplicate nodes**。

二叉查找树是一种很基础的数据结构，可以用来构建更为抽象的数据结构，后续博客会一一讲到。

如下所示的两棵树，只有左边的是二叉查找树。

{% img center /images/bi_search_tree.bmp %}


## 实现

现在给出在二叉查找树上的相关操作描述和 C 语言实现，实现过程中一般使用递归操作，因为二叉查找树的平均深度为$O(\log{N})$，所以就算是尾递归，也不必担心栈空间被用尽，方便起见，节点中数据类型默认为整型，并且假定，对于其他数据类型，**>** , **<** ,**=** 都可用于他们的比较，事实上，重写这些运算符也是很容易的。

下面给出操作的**头文件定义**

```C
#ifndef _Tree_H

struct TreeNode;
typedef struct TreeNode *Position;
typedef struct TreeNode *SearchTree;
typedef int ElementType;

SearchTree MakeEmpty( SearchTree T );
Position Find( ElementType X, SearchTree T );
Position FindMin( SearchTree T );
Position FindMax( SearchTree T );
SearchTree Insert( ElementType X, SearchTree T );
SearchTree Delete( ElementType X, SearchTree T );
ElementType Retrieve( Position P );

#endif
```

**节点定义**

```C
struct TreeNode
{
	ElementType Element;
	SearchTree Left;
	SearchTree Right;
};
```

### 初始化

初始化主要将树初始化为一棵空树，当然也可以初始化为一个单节点树，但是遵循树的递归定义，在这里，我们把它初始化为一棵空树

```C
/**初始化一棵空树**/
SearchTree MakeEmpty( SearchTree T )
{
	if( T != NULL )
	{
		MakeEmpty( T->Left );
		MakeEmpty( T->Right );
		free( T );
	}
	return NULL;
}
```

### 查找

这个操作一般需要返回指向树 **T** 中具有关键字 **X** 的节点的地址，如果这样的节点不存在，则返回 **NULL** ，在这里我们依然是递归查找，首先测试树 **T** 是否是 **NULL**，是 **NULL** 的话直接返回 **NULL** ,否则，若存储在 **T** 中的关键字是 **X** ，那么我们就可以返回 **T** 。否则，根据 **X** 和 **T** 中关键字的大小关系，对左子树或右子树进行递归查找。

```C
/**查找成功的话返回指向树T中具有关键字X的节点的指针，否则返回NULL**/
Position Find( ElementType X,SearchTree T )
{
	if( T == NULL )
		return NULL;
	if( X < T->Element )
		return Find( X, T->Left );
	else if( X > T->Element )
		return Find( X, T->Right );
	else
		return T;
}
```

注意首先是对树 **T** 做是否为 **NULL** 的判断，而这句必须放在第一句，不然的话就会产生**死递归**。另外，虽然两个递归查找是**尾递归**，但是由于树的平均深度为$O(\log{N})$，所以不会产生栈空间被用尽的情况(关于平均深度在后面会有讨论)。

接下来是 **FindMin** 和 **FindMax**，这两个操作分别返回树中最小元素的位置和最大元素的位置，字面上好像是返回最小元素的值和最大元素的值，但是和 **Find** 操作保持一致，我们在这里返回地址，取得其值也是很容易的。以查找最小元素为例，从根节点一直向左走就行，终点就是最小元素，查找最大元素也类似，这里分别以**递归**和**迭代**来实现 **FindMin** 和 **FindMax** 。

**FindMin**

```C
/**找到最小元素的位置**/
Position FindMin( SearchTree T )
{
	if( T == NULL )
		return NULL;
	else if( T->Left == NULL )
		return T;
	else
		return FindMin( T->Left );
}
```

**FindMax**

```C
/**找到最大元素的位置**/
Position FindMax( SearchTree T )
{
	if( T != NULL )
		while( T->Right != NULL )
			T = T->Right;
	return T;
}
```

### 插入

插入的操作在这里是比较简单的，因为插入的节点在第一次插入后都是叶子节点，而我们要做的只是为它找到插入的位置，在这里我们忽略了一点，如果插入一个已经存在于树 **T** 中的值我们应该怎么做，最简单的处理办法就是什么都不做(本操作中采用)，或者我们可以在树的节点结构中再维护一个**频数域**，对于相同的值，我们增加它的频数，这种处理办法另一个有好处的地方就是在树的**延迟删除**操作中，后面到删除的时候我们再讲。

上面讲到插入操作其实做的工作是寻找插入的位置，下面的图示说明了在插入 **5** 前后二叉树的状态。

{% img center /images/insert_bi_search_tree.bmp %}

为了插入 **5** ,我们遍历该树就好像在运行 **Find** ，当遍历到节点 **4** 的时候，需要向右走，但是右边不存在子树，因此，我们找到了插入节点 **5** 的位置。

```C
/**向树中插入元素**/
SearchTree Insert( ElementType X, SearchTree T )
{
	if( T == NULL )
	{
		//此时 T 是一棵空树，分配空间并创建一个单节点
		T = malloc( sizeof( struct TreeNode ) );
		if( T == NULL )
			printf("Out of space!!!\n");
		else
		{
			T->Element = X;
			T->Left = T->Right = NULL;
		}
	}
	else
	{
		//递归插入 X 到合适的子树中
		if( X < T->Element )
			T->Left = Insert( X, T->Left );
		else if( X > T->Element )
			T->Right = Insert( X, T->Right );
		else
			//X已经在树中，什么也不做
			;
	}
	return T;
}
```

### 删除

删除操作相对于前面的几种操作稍微复杂了一些，因为可能有多种情况，如果**要删除的节点是叶子节点**，这是最简单的情况，我们可以立即删除；如果**要删除的节点有一个儿子**，那么我们可以调整之后再进行删除，调整过程就是在这个节点没有之后局部调整一下指针指向(具体来说是让该节点的父亲指向它的孩子)，使得整棵树依旧是一棵二叉查找树，这种情况下的调整比较简单，如下图所示

{% img center /images/delete_bi_search_tree.bmp %}

假设我们要删除节点 **4** ，而节点 **4** 有一个左孩子，那么我们要做的调整就是使节点 **4** 的左孩子取代节点 **4** 的位置成为节点 **2** 的右孩子，这种独生子的情况下孩子总是可以替代父亲，而调整的范围也仅仅限于三代：当前要删除的节点，它的父节点，它的子节点，调整的过程也比较简单。

比较复杂的一种是**要删除的节点有两个儿子**，这个时候依旧要调整指针指向使得整棵树维持二叉查找树，一般的删除策略是用其右子树中的最小的节点来代替该节点并递归地删除那个最小的节点( 用左子树中最大的节点来代替也行，这两种策略都会使树越来越不平衡 )，因为右子树中的最小节点不可能有左儿子(因为它是最小的，而左儿子始终要大于它),所以第二次删除就退化为了只有一个孩子的删除情况，这种情况我们刚刚讨论过，下面是一个简单的删除示意图

{% img center /images/delete2_bi_search_tree.bmp %}

在上图中，我们要删除的节点是根节点的左孩子 **2** ,于是我们先用它的右子树中最小的节点 **3** 来代替它，接着我们再删除右子树中的节点 **3** ,此时删除节点 **3** 就比较容易了，因为它只有一个右孩子 **4**。

下面是具体的实现

```C
/**删除树中节点值为 X 的节点**/
SearchTree Delete( ElementType X, SearchTree T )
{
	Position TmpCell;

	if( T == NULL )
		printf("Element not found!!!\n");
	else if( X < T->Element )
		T->Left = Delete( X, T->Left );
	else if( X > T->Element )
		T->Right = Delete( X, T->Right );
	else /**要删除的节点找到了**/
	{
		//要删除的这个节点有两个孩子
		if( T->Left && T->Right )
		{
			//用该节点右子树中最小的节点代替该节点
			TmpCell = FindMin( T->Right );
			T->Element = TmpCell->Element;
			T->Right = Delete( T->Element, T->Right );
		}
		else /**要删除的这个节点有一个或0个孩子**/
		{
			TmpCell = T;
			if( T->Left == NULL )
				T = T->Right;
			else if( T->Right == NULL )
				T = T->Left;
			free( TmpCell );
		}
	}
	return T;
}
```

如果删除的次数不多，且节点值经常重复的时候，通常使用的策略是**延迟删除**，当一个元素要被删除的时候，我们并不立即执行删除，而只是做一个被删除的记号，如果树的节点中有一个专门记录频数的域，那么我们可以使这个频数减一，如果节点值在删除之后又重新插入，那么我们只要使频数增一就可以了，省去删除和插入节点时的开销，试想一下树的深度在**千万级别**的时候，节省不必要的开销是非常必要的。

### 测试

主函数中我们初始化一棵树，然后插入一个值并输出，接着随机插入一些值，中序打印该树可以发现，输出的顺序正好是按照大小排好序的值。

```C
void main()
{
	//初始化一棵树
	SearchTree my_tree;
	my_tree = MakeEmpty( my_tree );
	
	//插入一个值并输出
	my_tree = Insert( 15, my_tree );
	printf("%d\n",Retrieve( Find( 15, my_tree ) ) );
	
	//随机顺序插入一些值
	my_tree = Insert( 18, my_tree );
	my_tree = Insert( 0, my_tree );
	my_tree = Insert( 7, my_tree );
	my_tree = Insert( 888, my_tree );
	my_tree = Insert( 14, my_tree );
	my_tree = Insert( 8, my_tree );
	my_tree = Insert( 4, my_tree );
	my_tree = Delete( 4, my_tree );
	
	//中序打印该树
	PrintTree( my_tree );
}
```

结果
{% img center /images/Test.png %}

## 分析

下面对算法的复杂度做简要的分析，我们从上述的几种操作来看，除了 **MakeEmpty** 之外，其他的操作都是$O(d)$，其中$d$是要访问的节点的深度，为了证明上面的那些操作复杂度为$O(\log{N})$，我们现在证明，假定所有树(节点)出现的机会均等，则树的所有节点的平均深度为$O(\log{N})$。

引入**内部路径长的概念**，一棵树所有节点的深度的和成为内部路径长，我们用$D(N)$表示，计算出$D(N)$之后，那么每个节点的平均深度也就是$\frac{D(N)}{N}$,我们现在证明内部路径长$D(N)$为$O(N\log{N})$。

首先，$D(1)$ = 0 ,一棵 $N$ 节点树是由一棵 $i$ 节点树和一棵 $(N-i-1)$ 节点的右子树以及深度为 $0$ 的一个根节点组成，其中 $0\leq{i}<N$ ,$D(i)$ 为根的左子树的内部路径长，但是在原树中，所有节点的深度都要加一(因为还有根节点)，同样的结论对于右子树也是成立的，于是有下式

$$D(N) = D(i) + D(N - i - 1) + (N - 1)$$


未完...











