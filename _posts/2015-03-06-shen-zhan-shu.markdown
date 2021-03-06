---
layout: post
title:  "伸展树"
date: 2015-03-06 00:52:01 +0800
comments: true
categories: 算法相关
tags: [伸展树, 二叉平衡树, Splay Tree, 数据结构]
keywords: 伸展树, 二叉平衡树, Splay Tree, 数据结构
description: 伸展树的C语言实现以及分析
---


## 算法原理

伸展树是另一种二叉查找树，比**AVL树**简单一些，它支持在二叉查找树上的所有操作，虽然它不能保证最坏情况下的时间复杂度为$O(\log{N})$（AVL树可以保证），但是却可以保证在摊还情况下的运行时间为$O(\log{N})$。

注意以下几点

-  通常的平衡树（比如AVL树）需要存储额外的信息，比如节点高度，平衡因子，而伸展树不需要，有利于减少内存的使用。
-  其他的平衡树实现比较复杂。
-  如果第二次访问某个节点的花费比第一次访问它的花费要小，有利于提高性能。
-  90/10原则，90%的访问操作仅仅是访问10%的数据。
-  只要平均情况下的访问时间较优，这也是比较满意的。

<!--more-->

于是，很自然地我们想到，既然在二叉查找树中对节点访问的时间取决于这个节点的深度，那么我们只要**让那些经常被访问的节点靠近根节点**，这样一来，整体上对树中节点访问的时间就会减少，上文中提到，这个时间为$O(\log{N})$。

这里有一个很典型的例子：网络中的路由。路由器接收大量数据包，而又要以很快的速度根据数据包中的IP地址把它们转发到正确的线路，所以路由器需要一张很大的字典去查询，根据IP地址来找到正确的转发线路，一般来说如果一个数据包被转发了，那么下一个被转发的很可能是同一个IP地址的数据包，那么这个时候字典的数据结构用伸展树来实现就会有比较好的性能。

我们把被访问到的节点叫做**伸展节点**，那么每次对节点的访问，我们都要把**伸展节点**移到树根的位置，对节点的访问包括通常的**查找**，**删除**，**插入**操作。需要注意的是，如果某次查找，查找失败（要查找的节点不存在），那么我们会将**失败处的节点作为伸展节点**，比如说下面这棵树

{% img center /images/splay_tree_fail_find.png %}

我们查找**节点5**的时候在**节点6**处查找失败（**节点6**的左子树为空），这个时候我们便把**节点6**作为伸展节点，类似的情况有删除，如果在上图中我们需要删除**节点5**，而**节点5**不存在，那么我们便把**节点6**作为伸展节点，对于插入的情况，如果插入一个已经存在的节点，我们不会增加新的节点（另一种处理办法是在节点中保存它出现的频数），但是会将已经存在的那个节点作为伸展节点，移至根节点处，这样做的理由是

-  查找失败的时候，**失败处的节点**是**要查找节点的前驱或后继**，正如上面那棵树，要查找的节点是**5**，而**节点6**则是它的后继，那么下次查找的时候很可能会查找**节点5**的前驱和后继，因为一个节点的前驱或者后继在某种程度上是该节点的替代者，删除的时候道理一样。
-  对于插入操作，虽然本文的做法欠妥，因为插入一个已经存在的节点就好像是做查找一样，仅仅是把查找到的节点上移至了根节点，但是为了保证二叉查找树中节点不重复，我们还是决定这样做，当然也可以在节点的结构体中新增一个频数域保存节点的重复次数，但是对于通常的应用来讲，这样的处理是可以的，改进它也很简单。

将**伸展节点**移动到树根位置，这个过程我们是通过旋转来完成的，这里的旋转和**AVL树**中的旋转比较类似，我在[这篇博文][1]里说明了**单旋转**和**双旋转**的过程，但是它们的目的是不同的，这里的旋转是为了把节点调整到根节点的位置，而**AVL**树中的旋转是为了保证树的平衡，所以**AVL树**总是保证树是一棵平衡的二叉查找树。在伸展树中执行的旋转操作会带来以下两点影响

-  使得**伸展节点**到达树根位置
-  使得**伸展节点**到**根节点**的路径上的其他节点离**伸展节点**的距离变短

由于第二点影响，伸展树中的旋转操作显然会**使得树越来越平衡**。

一共有三种旋转，熟悉**AVL树**的话你会对这三种旋转感到很亲切。

下面是三种旋转

### 最简单的旋转

这种情况下，我们要把**节点X**（左图中）或**节点Y**（右图中）移到根节点，做一次单旋转即可

```C
/******************************
*     y             x
*    / \           / \   
*   x   C   <->   A   y
*  / \               / \
* A   B             B   C
*
******************************/
```

### 左左情况或右右情况

这里的旋转和**AVL树**中的**左左情况**以及**对称的右右情况**类似，也是一次单旋转即可将**节点X**（左图中）或**节点Z**（右图中）移至根节点

```C
/*************************************************
*        z             x
*       / \           / \
*      y   D         A   y
*     / \      <->      / \              (A < x < B < y < C < z < D)
*    x   C             B   z
*   / \                   / \
*  A   B                 C   D
*
**************************************************/
```

### 左右情况或右左情况

这里的也和**AVL树**中旋转类似，我们通过一次双旋转，把**节点X**移到根节点处

```C
/************************************************
*       z                x                y
*      / \              / \              / \
*     y   D            /   \            A   z       (A < y < B < x < z < D)
*    / \         ->   y     z    <-        / \
*   A   x            / \   / \            x   D
*      / \          A   B C   D          / \
*     B   C                             B   C
*
************************************************/
```

将节点移动至根节点处，我们从伸展节点开始，依次向上执行这些旋转就行了，这三种旋转都比较简单，具体可以参考我的[另一篇博文][1]。

下面我们就上面的旋转来看一个比较完整的例子，尽管这个例子还是很小。

### 查找示例

假设我们有下面这棵二叉查找树，显然，按照从小到大的顺序依次插入**1,2,3,4,5,6,7**便得到了这棵比较差的二叉查找树(因为它的高度很高)，现在假设对最深的**节点1**进行一次查找，如图所示

{% img center /images/splay_tree_1.png %}

这个时候**节点1**便是伸展节点，我们对它进行一次旋转，显然这个旋转是第二种情况的左左旋转，旋转之后，如下图

{% img center /images/splay_tree_2.png %}

这个时候我们接着旋转，如下图

{% img center /images/splay_tree_3.png %}

最后，执行一次旋转，得到下面最终的结果

{% img center /images/splay_tree_4.png %}

这个时候已经将**节点1**移动到了树根的位置，而整棵树也变得比之前要平衡一些，我们现在访问**节点2**已经不用像刚才那样需要花费$N-2$个时间单元，试想一下，如果我们访问之前访问过的节点，这个时候我们已经把它移到了树根的位置，可以立即访问到，而我们访问比较深的节点的时候，随着将它移动到树根的这个过程，整棵树又变得更加平衡，尽管这种讲法不太严谨，但是在实际的情况下，伸展树的性能确实要好一些。

另外，我们也可以发现，**如果要访问的那个节点很深，虽然这次要花费多一点的时间单元，但是将它移动到树根之后，这种移动对未来的操作是有好处的**，就像上面那样。而我们**访问一个离根节点很近的节点的时候，尽管这次访问耗时很少，但是移动它所对树造成的影响却不一定有好处，有时甚至会使树变得不平衡一些。**这两种情况的综合结果就是我们最终会得到一个比较平衡的树，也就是对树各种操作的摊还耗时较少，从这个角度讲，这两种情况本身也符合着某种平衡。

### 删除示例

下面我们看一下在伸展树中删除的情况，删除的操作也是比较简单的，我们先找到那个要删除的节点，这个时候它便是**伸展节点**，我们把它移动到根节点的位置，执行删除之后，剩下两棵没有爸爸的**子树L**和**子树R**，我们可以找到**子树L**中最大的**节点LMAX**，**节点LMAX**必然没有**右孩子**，于是我们把这个**节点LMAX**旋转到**子树L**的**根节点位置**，而它本身没有**右孩子**，于是我们让先前的**子树R**成为**节点LMAX**的**右孩子**，至此，删除操作完成，下面是简单的示例

还是接着上面查找的例子来，现在假设我们要删除**节点4**，如图所示

{% img center /images/splay_tree_del_1.png %}

这个时候我们在**节点1**，**节点6**，**节点4**之间做一次双旋转，将**节点4**旋转到根节点的位置，如下图

{% img center /images/splay_tree_del_2.png %}

紧接着，删除**节点4**，下图是删除之后剩下的两棵子树

{% img center /images/splay_tree_del_3.png %}

我们找到左子树中最大的**节点3**，然后将它旋转到根节点的位置，如下

{% img center /images/splay_tree_del_4.png %}

最后我们让原先的右子树成为**节点3**的右孩子，完成删除操作。

### 插入示例

插入的操作也是简单的，我们先执行普通的插入，将节点插入到合适的位置，然后再将该节点通过旋转移动到根节点的位置，接着上面的例子，我们现在重新插入**节点4**,。

插入之前的树如下

{% img center /images/splay_tree_ins_1.png %}

我们在**节点5**处找到了插入的位置，于是插入**节点4**，如下

{% img center /images/splay_tree_ins_2.png %}

之后在**节点4**，**节点5**，**节点6**之间进行一次旋转，得到下图

{% img center /images/splay_tree_ins_3.png %}

最后进行一次旋转，得到最终的结果如下

{% img center /images/splay_tree_ins_4.png %}

至此，完成插入操作。

## 实现

上述讨论的查找，删除，插入的过程是从伸展节点处开始，一直向根节点处进行调整，使得伸展节点移动到根节点的位置，也就是**自底向上**的伸展过程，考虑到自底向上实现方式的空间成本和效率，这里我们采用迭代的方式实现**自顶向下**的伸展过程(自底向上一般采用递归的方式),但是具体的旋转过程还是上述的三种旋转情况。

在**自顶向下**的伸展过程中，我们利用两棵额外的树来存放**伸展过程中的节点**，最终将这两棵树和包含伸展节点的树(此时伸展节点已经到达了树根的位置)进行合并，完成伸展过程，注意，这里的这两棵额外的树不是原来树的子树。

两棵额外的树分别是**树L**和**树R**，在伸展过程中任何一个时刻，我们都有一个当前节点X，它是其子树的根(这个节点X是伸展节点到原树根节点的路径上的某一个节点)，在下面的图示中，也就是那棵**中间树**，这个节点初始的时候是根节点，然后一路向下走，这里的路是**原树的根节点到伸展节点的路径**，在节点X向下走的过程中，我们把比它大的那些节点放在树R中，把比它小的那些节点放在树L中，那么，当这个节点X走到伸展节点的时候(注意，伸展节点不一定是叶子节点)，我们便有了三棵树，树L，树R，和最终包含伸展节点的那棵树，原来树L和树R是空树，现在可能已经各自挂满了节点，最后，我们把这三棵树进行合并，合并之后的那棵新树就是**经过伸展之后的树**，那么我们便在各种操作之后完成了对树的伸展操作。

伸展树中最主要的是**伸展过程**，也就是将伸展节点一直移动到树根的这个过程，而这个伸展的过程则是查找，删除，插入等操作的基础，我们以三种旋转情况为主线，说明自顶向下的伸展过程。

### 简单情况下的单旋转

{% img center /images/SplayTree_1.bmp %}

如图所示，树L和树R初始的时候分别是一棵空树，现在我们要把节点Y移动到根节点的位置，那么我们将节点Y移动到根节点的位置之后，将节点X以及节点X的右子树接到树R中，变成图中右半部分的样子，这种是最简单的情况，因为我们要移动的节点Y就是节点X的左孩子(对称的情况是Y位于节点X的右孩子处，这里不再赘述)

### 左左情况或右右情况

{% img center /images/SplayTree_2.bmp %}

如图所示，现在我们将节点X的左孩子Y的左孩子Z移动至根节点，这种情况属于左左情况，右右的情况和这个相似，那么这个时候我们将节点Z移动至根节点之后，便把节点Y和节点X连同他们的对应子树接到树R中，这里要提一下的是，这种旋转对于节点Z是否是叶子节点是没有要求的，它可以有一个孩子或者两个孩子，第一种简单情况下的节点Y也是这样。

### 左右情况或右左情况

{% img center /images/SplayTree_3.bmp %}
这种情况下的处理也很好理解，我们要把节点X的左孩子的右孩子移动至根节点，于是在将节点Z移动至根节点之后，我们把对应的节点Y和节点X放到合适的位置。

### 合并

随着这种自顶下下的伸展过程，很容易想象得到，树L和树R会不断地增长，而原树中的节点因为逐渐移到了那两棵树中所以会变得越来越少，自顶向下的结束标志就是我们走到了伸展节点，这个时候我们要进行合并，如下图

{% img center /images/Splay_tree_combin.bmp %}

这个合并的过程很好理解，因为树L中的节点都是小于节点X(及其子树)的，所以很自然将树L接到节点X的左边并且将原先节点X的左孩子接到树L的右方(因为树L中的节点小于节点A)，右边的情况也是一样，所以完成了树的合并。

### 示例

下面我们用一个例子来说明这个伸展过程以及最后的合并操作。

{% img center /images/splay_tree_demo_1.bmp %}

如图所示，现在我们要伸展**节点19**，第一步的时候我们自顶向下到了节点25，这是第一种情况，于是我们将25移到根节点的位置，并且将节点12和节点5接到树L上，如下图

{% img center /images/splay_tree_demo_1.5.bmp %}

然后我们再接着往下，到了节点15，这个时候是左左的情况，我们将节点15移到根节点的位置，并作如下图所示的调整

{% img center /images/splay_tree_demo_2.bmp %}

因为要伸展节点19，所以继续向下，到了节点18，再做如下调整

{% img center /images/splay_tree_demo_3.bmp %}

这个时候再往下走的时候发现节点18的右孩子是空，所以这个时候节点18作为伸展节点(回忆刚开始的时候说的那几种例外情况)，这个时候我们停止往下，进行合并的操作，将树L和树R和现在的树进行合并，合并之后的树如下图

{% img center /images/splay_tree_demo_4.bmp %}

在访问节点19失败之后我们便按照这样的方式将节点18移动到了根节点的位置，本次伸展过程结束。

有了以上的示例，我们开始写伸展过程的代码，这个时候代码应该是相当易懂的。

我们先有以下的定义和声明

```C
	
        #define Infinity 30000
        #define NegInfinity (-30000)
        struct SplayNode;
        typedef struct SplayNode *SplayTree;
        typedef int ElementType;
        typedef struct SplayNode *Position;

        struct SplayNode
        {
            ElementType Element;
            SplayTree      Left;
            SplayTree      Right;
        };

        static Position NullNode = NULL;
```

这些声明在之前的博客中也已经有过说明，下面是伸展的过程

### 伸展过程

```C
	/* 自顶向下从节点 X 开始伸展节点值为 Item 的节点 */
        SplayTree
        Splay( ElementType Item, Position X )
        {
            static struct SplayNode Header;
            Position LeftTreeMax, RightTreeMin;

            Header.Left = Header.Right = NullNode;

            LeftTreeMax = RightTreeMin = &Header;
            NullNode->Element = Item;

            while( Item != X->Element )
            {
                if( Item < X->Element )
                {
                    if( Item < X->Left->Element )			
                        X = SingleRotateWithLeft( X );
                    if( X->Left == NullNode )			
                        break;

                    /* 连接到右边 */
                    RightTreeMin->Left = X;
                    RightTreeMin = X;
                    X = X->Left;
                }
                else
                {
                    if( Item > X->Right->Element )			
                        X = SingleRotateWithRight( X );
                    if( X->Right == NullNode )			
                        break;

                    /* 连接到左边 */
                    LeftTreeMax->Right = X;
                    LeftTreeMax = X;
                    X = X->Right;
                }
            }

            /* 对树进行合并 */
            LeftTreeMax->Right = X->Left;
            RightTreeMin->Left = X->Right;
            X->Left = Header.Right;
            X->Right = Header.Left;

            return X;
        }

```

在迭代的伸展过程中，不同于递归，我们使用**树LeftTreeMax**和**树RightTreeMin**来存放伸展过程中的节点(你可以联想到阶乘的迭代求法和递归求法，迭代需要保存过程量，而递归不需要在程序中保存过程量)，这段程序中使节点 X 沿着通往伸展节点的方向逐级往下走，并且不断地把一些节点放到上述的两棵树中，其中 **SingleRotateWithRight()** 和 **SingleRotateWithLeft()** 分别是右右情况下的单旋转和左左情况下的单旋转。

### 查找

查找的代码应该是很容易的，我们直接对要查找的节点进行伸展，如果查找的节点不存在，那么我们返回NULL，否则返回伸展后树的根节点

```C
	/* 查找节点值为X的节点，若节点存在，返回此节点，否则返回NULL */
        SplayTree Find( ElementType X, SplayTree T )
        {
            Position P = Splay( X, T );
	    if( P->Element == X )
		return P;
	    else
		return NULL;
        }
```

### 插入

插入的过程也是比较容易理解的，代码如下

```C
	/* 向树 T 中插入一个节点值为 Item 的节点，若节点已经存在，则什么都不做 */
	SplayTree Insert( ElementType Item, SplayTree T )
        {
            static Position NewNode = NULL;

            if( NewNode == NULL )
            {
                NewNode = malloc( sizeof( struct SplayNode ) );
                if( NewNode == NULL )
                    FatalError( "Out of space!!!" );
            }
            NewNode->Element = Item;

            if( T == NullNode )
            {
                NewNode->Left = NewNode->Right = NullNode;
                T = NewNode;
            }
            else
            {
                T = Splay( Item, T );
                if( Item < T->Element )
                {
                    NewNode->Left = T->Left;
                    NewNode->Right = T;
                    T->Left = NullNode;
                    T = NewNode;
                }
                else
                if( T->Element < Item )
                {
                    NewNode->Right = T->Right;
                    NewNode->Left = T;
                    T->Right = NullNode;
                    T = NewNode;
                }
                else
                    return T; 
            }

            NewNode = NULL;  
            return T;
        }
```

插入的时候，我们先对插入的节点进行伸展，如果它不在树中(这也是一般的情况)，那么我们向伸展之后的树的根节点处插入即可，如果节点已经在树中，那么我们什么都不做。

### 删除

删除的操作

```C
        SplayTree
        Remove( ElementType Item, SplayTree T )
        {
            Position NewTree;

            if( T != NullNode )
            {
                T = Splay( Item, T );
                if( Item == T->Element )
                {
                    /* 找到了该节点 */
                    if( T->Left == NullNode )
                        NewTree = T->Right;
                    else
                    {
                        NewTree = T->Left;
                        NewTree = Splay( Item, NewTree );
                        NewTree->Right = T->Right;
                    }
                    free( T );
                    T = NewTree;
                }
            }

            return T;
        }
```

删除的时候，我们先对要删除的节点进行伸展，这个时候它已经移到了根节点的位置，如果这个时候它的左孩子是空节点，那么我们直接删除，将它的右子树作为新的树即可，如果它的两棵左右子树都不为空，那么我们先将它删除，然后剩下两棵左右子树，然后我们把左子树中最大的节点移动至根节点，然后再把原先的右子树连接到此根节点上成为它的右子树即可，说起来有点拗口，前文中说明了这点，不明白的话可以再回过头看一下。

至此，我们完成了对伸展树的讨论，欢迎对错误和不当之处进行批评指正。


  [1]: http://hengyicai.gitcafe.io/blog/2015/02/23/avlshu/
