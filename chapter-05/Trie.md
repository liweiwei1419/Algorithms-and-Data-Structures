# 高级数据结构：Trie

---

[TOC]

---


Trie 又称作字典树、前缀树。

## Trie 的结构

+ Trie 的设计很巧妙，它不像一般的字典那样，把一整个单词存在数据结构里。Trie 是利用单词的前缀去匹配一棵树，能匹配上，则说明这个字典里有这个单词；

+ **Trie 的结构有那么一点点像链表**，只不过链表的 `next` 指针指向的是一个 `Node` 节点，而 `Trie` 的 `next` 指针指向的是一个 `Map`；

+ Trie 本身携带的内容仅仅只是 `isWord` 这样一个布尔型变量，而沿着 `next` “指针”一路走下来的路径，就能表示一个单词。

下面是一个 Trie 树的基本结构：

```java
import java.util.TreeMap;

public class Trie {

    private class Node {
        // isWord 表示沿着根节点往下走，走到这个节点的时候是否是一个单词的结尾
        public boolean isWord;
        // 用哈希表也是可以的，因为这里不涉及排序操作
        public TreeMap<Character, Node> next;

        public Node(boolean isWord) {
            this.isWord = isWord;
            next = new TreeMap<>();
        }

        public Node() {
            this(false);
        }
    }

    private Node root;
    private int size;

    public Trie() {
        root = new Node();
        size = 0;
    }

    public int getSize() {
        return size;
    }
}
```

## Trie 的添加操作

```java
public void add(String word) {
    // root 是当前 Trie 对象的属性
    Node currNode = root;
    for (int i = 0; i < word.length(); i++) {
        Character c = word.charAt(i);
        if (currNode.next.get(c) == null) {
            currNode.next.put(c, new Node());
        }
        currNode = currNode.next.get(c);
    }
    // 如果已经添加过，才将 size++
    if (!currNode.isWord) {
        currNode.isWord = true;
        size++;
    }
}
```

## Trie 的查询操作

+ **理解 Trie 的查询只与待查询的字符串的长度有关**；
+ 下面这个方法查询整个单词在 Trie 中是否存在，所以在路径匹配完成以后，一定不要忘了判断匹配到的那个结点的 `isWord ` 属性，如果它是一个单词的结尾，才返回 `True`，

```java
// 基于 Trie 的查询
public boolean contains(String word){
    Node currNode = root;
    Character currC;
    for (int i = 0; i < word.length(); i++) {
        currC = word.charAt(i);
        if(currNode.next.get(currC)==null){
            return false;
        }
        currNode = currNode.next.get(currC);
    }
    return currNode.isWord;
}
```

## Trie 的前缀查询操作

前缀查询就更简单了，此时不需要判断 `isWord` 属性的值，只需要判断从树的根节点是不是很顺利地都能匹配单词的每一个字符。

```java
// 是否有某个前缀
public boolean isPrefix(String prefix) {
    Character c;
    Node currNode = root;
    for (int i = 0; i < prefix.length(); i++) {
        c = prefix.charAt(i);
        if (currNode.next.get(c) == null) {
            return false;
        }
        currNode = currNode.next.get(c);
    }
    // 只需要判断从树的根节点是不是很顺利地都能匹配单词的每一个字符
    // 所以，能走到这里来，就返回 True
    return true;
}
```

## 例题：LeetCode 第 208 题 Implement Trie (Prefix Tree)
传送门：[LeetCode 第 208 题 Implement Trie (Prefix Tree) 英文地址](https://leetcode.com/problems/implement-trie-prefix-tree/description/)、[LeetCode 第 208 题 实现 Trie (前缀树) 中文地址](https://leetcode-cn.com/problems/implement-trie-prefix-tree/description/)。

说明：这道问题要求我们实现一个 Trie (前缀树)，包含 insert, search, 和 startsWith 这三个操作。其实就是我们上面列出的“添加”、“查询”、“前缀查询”操作。

我的 Python 解答：

```python
class Trie(object):
    class Node:
        def __init__(self):
            self.is_word = False
            self.dict = dict()

    def __init__(self):
        """
        Initialize your data structure here.
        """
        self.root = Trie.Node()

    def insert(self, word):
        """
        Inserts a word into the trie.
        :type word: str
        :rtype: void
        """
        cur_node = self.root
        for alpha in word:
            if alpha not in cur_node.dict:
                cur_node.dict[alpha] = Trie.Node()
            # 这里不要写成 else ，那就大错特错了
            cur_node = cur_node.dict[alpha]
        if not cur_node.is_word:
            cur_node.is_word = True

    def search(self, word):
        """
        Returns if the word is in the trie.
        :type word: str
        :rtype: bool
        """
        cur_node = self.root
        for alpha in word:
            if alpha not in cur_node.dict:
                return False
            else:
                cur_node = cur_node.dict[alpha]
        return cur_node.is_word

    def startsWith(self, prefix):
        """
        Returns if there is any word in the trie that starts with the given prefix.
        :type prefix: str
        :rtype: bool
        """
        cur_node = self.root
        for alpha in prefix:
            if alpha not in cur_node.dict:
                return False
            else:
                cur_node = cur_node.dict[alpha]
        return True
```

说明：其实很简单，多写几遍也就熟悉了。

## Trie 字典树和简单的模式匹配

### 例1：LeetCode 第 211 题 Add and Search Word - Data structure design

传送门：[LeetCode 第 211 题 Add and Search Word - Data structure design 英文地址](https://leetcode.com/problems/add-and-search-word-data-structure-design/description/
)、[LeetCode 第 211 题 添加与搜索单词 - 数据结构设计 中文地址](https://leetcode-cn.com/problems/add-and-search-word-data-structure-design/description/)。

我的 Python 解答：

```python
class WordDictionary(object):
    class Node:
        def __init__(self):
            self.is_word = False
            self.dict = dict()

    def __init__(self):
        """
        Initialize your data structure here.
        """
        self.root = WordDictionary.Node()

    def addWord(self, word):
        """
        Adds a word into the data structure.
        :type word: str
        :rtype: void
        """

        cur_node = self.root
        for alpha in word:
            if alpha not in cur_node.dict:
                cur_node.dict[alpha] = WordDictionary.Node()
            cur_node = cur_node.dict[alpha]
        if not cur_node.is_word:
            cur_node.is_word = True

    def search(self, word):
        """
        Returns if the word is in the data structure. A word could contain the dot character '.' to represent any one letter.
        :type word: str
        :rtype: bool
        """
        # 注意：这里要设置辅助函数
        return self.match(self.root, word, 0)

    def match(self, node, word, index):
        if index == len(word):
            return node.is_word
        alpha = word[index]
        if alpha == '.':
            for next in node.dict:
                if self.match(node.dict[next], word, index + 1):
                    return True
            # 注意：这里要返回
            return False
        else:
            # 注意：这里要使用 else 
            if alpha not in node.dict:
                return False
            # 注意：这里要使用 return 返回
            return self.match(node.dict[alpha], word, index + 1)
```

## Trie 字典树和字符串映射

### 例2：LeetCode 第 677 题 Map Sum Pairs

传送门：[LeetCode 第 677 题 Map Sum Pairs 英文地址](https://leetcode.com/problems/map-sum-pairs/)、[LeetCode 第 677 题 键值映射 中文地址](https://leetcode-cn.com/problems/map-sum-pairs/description/)。

我的 Python 解答：

```python
class MapSum(object):
    class Node:
        def __init__(self):
            self.val = 0
            self.dict = dict()

    def __init__(self):
        """
        Initialize your data structure here.
        """
        self.root = MapSum.Node()

    def insert(self, key, val):
        """
        :type key: str
        :type val: int
        :rtype: void
        """
        cur_node = self.root
        for aplha in key:
            if aplha not in cur_node.dict:
                cur_node.dict[aplha] = MapSum.Node()
            cur_node = cur_node.dict[aplha]
        cur_node.val = val

    def sum(self, prefix):
        """
        :type prefix: str
        :rtype: int
        """
        cur_node = self.root
        for alpha in prefix:
            if alpha not in cur_node.dict:
                return 0
            cur_node = cur_node.dict[alpha]
        return self.presum(cur_node)

    def presum(self, node):
        """
        计算以 node 为根结点的所有字符串的和
        :param node:
        :return:
        """
        s = node.val
        for next in node.dict:
            s += self.presum(node.dict[next])
        return s
```

## 参考资料
1、subetter 的文章《字典树》

https://subetter.com/articles/trie-tree.html

（本节完）