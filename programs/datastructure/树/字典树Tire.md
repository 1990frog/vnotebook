[TOC]

通常Trie只被用来处理字符串

# 字典
如果有n个条目，使用树结构，查询的时间复杂度是O(logn)，如果有100万个条目（2^20），logn大约是20
# Trie
![30214215-8a71ab1bca7c4920bd2143cef6596f08](_v_images/20200228001616879_1428495971.png)

查询每个条目的时间复杂度，和字典中一共有多少条目无关，时间复杂度为O(w)，w为查询单词的长度，大多数单词的长度小于10
每个节点有若干个指向下个节点的指针

```java
class Node{
    boolean isWord;
    Map<char,Node> next;
}
```

![400px-Trie_example](_v_images/20200228000711179_1735349854.png)



单词Trie
每个节点有26个指向下个节点的指针（不考虑大小写）
```java
class Node{
    char c;
    boolean isWord;
    Node next[26];
}

class Node{
    //char c;
    boolean isWord;
    Map<char,Node> next;
}
```

```java
import java.util.TreeMap;
Trie
getSize
add
contains
boolean isPrefix(String prefix)
```

# Trie和前缀搜索

# Trie字典树和字符串映射

# 更多的拓展

# Trie的删除操作

# Trie的局限性
最大的问题：空间
```java
class Node{
    //char c;
    boolean isWord;
    Tree<char,Node> next;
}
```

# 压缩字典树 Compressed Trie
Before compression
![trie07](_v_images/20200228001329705_1706578987.gif)
After compression
![trie08](_v_images/20200228001338775_1642973520.gif)
缺陷：
维护成本更高


# 三叉搜索树（Ternary Search Trie）


# 后缀树

# 更多字符串问题
子串查询
KMP
Boyer-Moore
Rabin-Karp

文件压缩

模式匹配

编译原理


DNA