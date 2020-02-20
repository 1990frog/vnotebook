[TOC]

# 连接问题
网络中节点间的连接状态
网络是个抽象的概念：用户之间形成的网络
数学中的集合类实现

# 并查集UnionFind
对于一组数据，主要支持两个动作：
union(p,q)
isConnected(p,q)




```java
getSize()
find(int p)
isConnected(int p,int q)
unionElements(int p,int q)
```

![](_v_images/20200216213214697_1602896862.png)

---

![](_v_images/20200216213540910_281543615.png)

![](_v_images/20200216214110473_415823091.png)

![](_v_images/20200216214359814_1399146217.png)

![](_v_images/20200216220415850_676648888.png)

![](_v_images/20200216220433575_1727700355.png)


![](_v_images/20200216220516150_1170461681.png)

![](_v_images/20200216220849258_114356544.png)
![](_v_images/20200216220859825_1636095636.png)

![](_v_images/20200216221109163_16831771.png)

rank pk size

![](_v_images/20200217115330401_1595817679.png)

# 路径压缩 Path Compression

![](_v_images/20200217115804922_917301207.png)

![](_v_images/20200217120050449_2060826853.png)

![](_v_images/20200217120259119_56979783.png)

rank不是深度也不是个数，仅是个排名

并查集的时间复杂度分析
O(h)
严格意义上：O(log*n) iterated logarithm

![](_v_images/20200217163854057_1427639654.png)

log*n比logn还快，近乎是O(1)级别的