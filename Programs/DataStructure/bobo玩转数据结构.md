[TOC]

# 种类
线性结构：
数组、栈、队列、链表、哈希表......
树结构：
二叉树、二分搜索树、AVL、红黑树、Treap、Splay、堆、Trie、线段树、K-D树、并查集、哈弗曼树...
图结构：
邻接矩阵、邻接表

# 数组
+ 数组最大的优点：快速查询。scores[2]
+ 数组最好应用于“索引有语意”的情况
+ 但并非所有语意的索引都适用于数组：例如身份证，数字太长，索引所占空间太多，不划算

java数组为静态数组

# 时间复杂度分析
O(1)
O(n)
O(lgn)
O(nlogn)
O(n^2)

大O描述的是算法的运行时间和输入数据之间的关系

O(n)
n是nums中的元素个数
算法和n呈线性关系
为什么要用大O，叫做O(n)？
忽略常数。实际时间T=c1*n+c2
```java
public static int sum(int[] nums){
    int sum=0;
    for(int num:nums)
        sum+=sum;
    return sum;
}
```

O渐进时间复杂度，描述n趋近于无穷的情况

T=2*n+2 O(n)
T=2000*n+10000 O(n)
T=1*n*n+0 O(n^2)
T=2*n*n+300n+10 O(n^2)

分析动态数组的时间复杂度：
增：O(n)，如果只对最后一个元素操作依然是O(n)？因为resize？
删：O(n)，如果只对最后一个元素操作依然是O(n)？因为resize？
改：已只索引O(1)；未知索引O(n)
查：已只索引O(1)；未知索引O(n)

resize的复杂度分析
添加操作：O(n)最坏情况
addLast(e)O(1)
addFirst(e)O(n)
add(index,e)O(n/2)=O(n)



resize O(n)
9次addLast操作，触发resize，总共进行了17次基本操作，平均，每次addLast操作，进行2次基本操作。
假设capacity=n，n+1次addLast，触发resize，总共进行了2n+1次基本操作，平均，每次addLast操作，进行2次基本操作。


resize O(n)
平均，每次addLast操作，进行2次基本操作，这样均摊计算，时间复杂度是O(1)的！在这个例子里，这样均摊计算，比计算最坏情况有意义。


均摊复杂度 amortized time complexity
resize O(n)
addLast 的均摊复杂度为O(1)

复杂度震荡
但是，当我们同时看addLast和removeLast操作：
addLast O(n)
removeLast O(n)
addLast O(n)
removeLast O(n)
capacity = n


复杂度震荡
出现问题的原因：removeLast时resize过于着急（Eager）
解决方案：Lazy
当size==capacity/4时，才将capacity减半


“索引有语意”的情况就可以直接使用索引来获取数据O(1)


# 均摊复杂度和防止复杂度的震荡

# 栈 Stack
+ 栈也是一种线性结构
+ 相比数组，栈对应的操作是数组的子集
+ 只能从一端添加元素，也只能从一端取出元素
+ 这一端称为栈顶
+ 栈是一种后进先出的数据结构
+ Last In First Out（LIFO）
+ 在计算机的世界里，栈拥有着不可思议的作用

# 栈的应用
无处不在的Undo操作（撤销）
程序调用的系统栈

![](_v_images/20191119224719039_791756788.png)
A2：A的第二行
编译器内部系统栈记录每次调用记录的点

# 栈的实现
Stack<E>

+ void push(E)
+ E pop()
+ E peek()
+ int getSize()
+ boolean isEmpty()


