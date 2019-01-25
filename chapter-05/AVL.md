# 高级数据结构：平衡二叉树

---

[TOC]

---


+ 理解树的高度

AVL 树保持平衡的思想，在快速排序的 partition 的优化中，也有类似的体现，即：为了避免递归深度加深，partition 选取标定点的时候，使用随机选取标定点的方式。

所以 AVL 树本质上是一颗二分搜索树，但这棵二分搜索树会“自动调整树的高度”，所以又叫可以自平衡的二叉树。

+ 理解为什么在统计意义上，红黑树比 AVL 树的性质要好。


## AVL 树本质上是一棵平衡二叉树


+ AVL 树是发明它的 3 位作者的名字的首字母。AVL 树最早的可以自平衡的二叉树。为了防止二分搜索树退化成链表，提出了平衡二叉树的概念；
+ AVL 树要保持的非常重要的性质：任意一个结点，左子树和右子树的高度**差**不能超过 $1$ ；
+ 平衡因子：左右子树的高度的差；
+ “AVL 树是一棵平衡二叉树” ，这就是 AVL 树的定义，即除了保证是一棵二分搜索树以外，“每个结点的左右子树的高度差不超过 $1$”；
+ 既然平衡因子是由子树的高度差来定义的，那么实现 AVL 树的时候，我们要先计算**以某个结点的为根结点的树的高度**，然后才能得到平衡因子。

> 某个结点为根结点的树的高度的计算公式：
>
> ```java
> node.height = 1 + Math.max(getHeight(node.left), getHeight(node.right));
> ```
>
>

![](https://liweiwei1419.github.io/images/algorithms/avl/avl-1.png)

![](https://liweiwei1419.github.io/images/algorithms/avl/avl-2.png)


## 计算结点的高度和平衡因子 

### 计算结点的高度

+ 计算每一个结点的高度值；
+ 添加新的结点的高度的起始值为 $1$；
+ 辅助函数，返回高度值。很简单，用 $3$ 行代码就能搞定；
+ 在添加结点的时候，维护高度值；
+ 设置辅助函数，返回平衡因子（左子树的高度 - 右子树的高度）。

### 编写内部结点类

```java
private class Node {
    private K key;
    private V value;
    private Node left;
    private Node right;
    /**
     * 添加了属性：高度
     */
    private int height;

    public Node(K key, V value) {
        this.key = key;
        this.value = value;
        this.left = null;
        this.right = null;
        // 默认起始的高度值为 1 ，很容易理解，单结点的高度就是 1
        this.height = 1;
    }
}
```

获得结点的高度，只是封装了对于空结点的判断：

```java
/**
 * 获得结点 node 的高度
 *
 * @param node
 * @return
 */
private int getHeight(Node node) {
    if (node == null) {
        return 0;
    }
    return node.height;
}
```

向树中添加结点，更新结点高度，计算平衡因子：

```java
public void add(K key, V value) {
    root = add(root, key, value);
}

private Node add(Node node, K key, V value) {
    if (node == null) {
        size++;
        return new Node(key, value);
    }
    if (key.compareTo(node.key) == 0) {
        node.value = value;
    } else if (key.compareTo(node.key) < 0) {
        node.left = add(node.left, key, value);
    } else {
        assert key.compareTo(node.key) > 0;
        node.right = add(node.right, key, value);
    }

    // 体会这一行代码：在递归函数中，从下至上（自底向上）地更新了影响到的结点的高度
    node.height = 1 + Math.max(getHeight(node.left), getHeight(node.right));

    // 计算一下平衡因子
    int balanceFactor = getBalanceFactor(node);
    if (Math.abs(balanceFactor) > 1) {
        System.out.println("该结点不平衡，平衡因子为：" + balanceFactor);
    }
    return node;
}
```


### 计算平衡因子

注意：平衡因子是根据高度即时计算出来的，不作为结点的属性。

```java
/**
 * 平衡因子，是根据高度即时计算出来的，不作为结点的属性
 *
 * @param node
 * @return
 */
private int getBalanceFactor(Node node) {
    if (root == null) {
        return 0;
    }
    return getHeight(node.left) - getHeight(node.right);
}
```


## 检查是否是二分搜索树性和平衡性


### 检查是否满足 BST 的性质：中序遍历

```java
/**
 * 检查是否满足 BST 的性质：利用 BST 中序遍历的有序性
 *
 * @return
 */
public boolean isBST() {
    List<K> keys = new ArrayList<>();
    inOrderCheckBST(root, keys);
    for (int i = 1; i < keys.size(); i++) {
        if (keys.get(i - 1).compareTo(keys.get(i)) > 0) {
            return false;
        }
    }
    return true;
}

private void inOrderCheckBST(Node root, List<K> keys) {
    if (root == null) {
        return;
    }
    inOrderCheckBST(root.left, keys);
    keys.add(root.key);
    inOrderCheckBST(root.right, keys);
}
```

### 检查是否满足 AVL 树的性质：前序遍历

```java
/**
 * 判断该二叉树是否是一棵平衡二叉树
 *
 * @return
 */
public boolean isBalanced() {
    return isBalanced(root);
}

/**
 * 检查是否满足 AVL 的性质：利用前序遍历，逐个检查以当前 node 为根的子树是否满足 AVL 的性质
 *
 * @param node
 * @return
 */
public boolean isBalanced(Node node) {
    if (node == null) {
        return true;
    }
    if (Math.abs(getBalanceFactor(node)) > 1) {
        return false;
    }
    // 注意：还要继续判断左右子树的平衡因子
    return isBalanced(node.left) && isBalanced(node.right);
}
```


## 在失去平衡的时候，为了保持平衡，执行的一些操作（这部分内容非常重要，是 AVL 树的核心）

这部分一定要画图才好理解。



![](https://liweiwei1419.github.io/images/algorithms/avl/avl-3.png)

![](https://liweiwei1419.github.io/images/algorithms/avl/avl-4.png)

![](https://liweiwei1419.github.io/images/algorithms/avl/avl-5.png)

![](https://liweiwei1419.github.io/images/algorithms/avl/avl-6.png)

![](https://liweiwei1419.github.io/images/algorithms/avl/avl-7.png)


我理解这部分内容的时候画的草稿。


![](https://liweiwei1419.github.io/images/algorithms/avl/avl-8.png)

![](https://liweiwei1419.github.io/images/algorithms/avl/avl-9.png)




## 左旋转和右旋转的实现

左旋转和右旋转是针对 LL、RR、LR、RL 这 4 种情况的基本操作。

### 右旋转

```java
/**
 * 辅助函数：右旋转，针对 LL 失衡（作为全过程）和 RL 失衡（作为子过程）
 * 右旋转示意图：
 *
 * @param node
 * @return
 */
private Node rightRotate(Node node) {
    // 这一句完全可以不要，只是针对画出来的图，方便比对
    Node x = node;
    Node y = node.left;

    Node w = y.right;

    y.right = x;
    x.left = w;

    // 不要忘记重新计算高度差
    x.height = Math.max(getHeight(x.left), getHeight(x.right)) + 1;
    y.height = Math.max(getHeight(y.left), getHeight(y.right)) + 1;

    return y;
}
```

### 左旋转
```java
/**
 * 辅助函数：左旋转，针对 RR 失衡（作为全过程）和 LR 失衡（作为子过程）
 * 右旋转示意图：
 *
 * @param node
 * @return
 */
private Node leftRotate(Node node) {
    // 这一句完全可以不要，只是针对画出来的图，方便比对
    Node x = node;

    Node y = x.right;
    Node w = y.left;

    y.left = x;
    x.right = w;

    x.height = Math.max(getHeight(x.left), getHeight(x.right)) + 1;
    y.height = Math.max(getHeight(y.left), getHeight(y.right)) + 1;

    return y;
}
```

## 在“添加”操作中，增加维持平衡的操作

### 基本步骤

1. 重新计算平衡因子（左子树高度 - 右子树高度）；
2. 如果失衡，继续判断是 LL、RR、LR、RL 当中的哪一种情况；
3. 针对不同的情况进行相应的“左旋转”、“右旋转”操作。

```java
private Node add(Node node, K key, V value) {
    if (node == null) {
        size++;
        return new Node(key, value);
    }
    if (key.compareTo(node.key) == 0) {
        node.value = value;
    } else if (key.compareTo(node.key) < 0) {
        node.left = add(node.left, key, value);
    } else {
        assert key.compareTo(node.key) > 0;
        node.right = add(node.right, key, value);
    }

    // 体会这一行代码：在递归函数中，从下至上（自底向上）地更新了影响到的结点的高度
    node.height = 1 + Math.max(getHeight(node.left), getHeight(node.right));

    // 计算一下平衡因子
    int balanceFactor = getBalanceFactor(node);
//        if (Math.abs(balanceFactor) > 1) {
//            System.out.println("该结点不平衡，平衡因子为：" + balanceFactor);
//        }

    // 在失衡的时候，做平衡维护

    // LL
    if (balanceFactor > 1 && getBalanceFactor(node.left) >= 0) {
        return rightRotate(node);
    }

    // RR
    if (balanceFactor < -1 && getBalanceFactor(node.right) <= 0) {
        return leftRotate(node);
    }

    // LR
    if (balanceFactor > 1 && getBalanceFactor(node.left) < 0) {
        node.left = leftRotate(node.left);
        return rightRotate(node);
    }

    // RL
    if (balanceFactor < -1 && getBalanceFactor(node.right) > 0) {
        node.right = rightRotate(node.right);
        return leftRotate(node);
    }
    return node;
}
```

## 在“删除”操作中，增加维持平衡的操作

### 基本步骤

在原来的 BST 删除的代码中，一共要修改 6 处，稍显麻烦，但一点都不难，有一点耐心，多一些细心，就不难完成代码的修改。具体修改之处，我已经作为注释添加在下面的代码中。


```java
private Node remove(Node node, K key) {
    if (node == null) {
        return null;
    }

    // 代码修改之处1：声明一个 retNode 结点，后序统一进行结点的平衡因子判定和维护
    Node retNode;
    if (key.compareTo(node.key) < 0) {
        node.left = remove(node.left, key);
        // 代码修改之处2：不马上返回，使用 retNode
        // 否则要在这里做结点的平衡因子判定和维护，后序一样，代码会非常冗长
        // return node
        retNode = node;
    } else if (key.compareTo(node.key) > 0) {
        node.right = remove(node.right, key);
        retNode = node;
    } else {
        assert key.compareTo(node.key) == 0;
        if (node.left == null) {
            Node rightNode = node.right;
            node.right = null;
            size--;
            retNode = rightNode;
            // 代码修改之处3：这几个分支之间是互斥的关系，所以要写成 if else 的形式
        } else if (node.right == null) {
            Node leftNode = node.left;
            node.left = null;
            size--;
            retNode = leftNode;
        } else {
            // 当前 node 的后继（也可以使用后继来代替它）
            Node successor = minimum(node.right);
            size++;
            // 代码修改之处4：removeMin 方法中没有结点的平衡因子判定和维护的逻辑，为此有两种方案，我们采用方案2
            // 方案1：在 removeMin 方法中添加结点的平衡因子判定和维护的逻辑
            // 方法2：递归调用自己，因为我们就是要在"自己"这个函数中，编写结点的平衡因子判定和维护的逻辑
            // successor.right = removeMin(node.right);
            successor.right = remove(node.right, successor.key);
            successor.left = node.left;
            node.left = null;
            node.right = null;
            size--;
            retNode = successor;
        }
    }
    
    // 代码修改之处5：我们代码的语义是删除结点，当删除的结点是叶子结点的时候，我们返回的是空结点
    // 那么在当前递归中，就不会有结点的平衡因子判定和维护的逻辑
    // 从后序的结点的平衡因子判定和维护的逻辑中，我们也可以看出，它会使用 retNode.left 和 retNode.right 进行左旋和右旋
    // 而对空节点进行左旋和右旋是没有意义的
    if (retNode == null) {
        return null;
    }
    // 代码修改之处6：添加结点的平衡因子判定和维护的逻辑，
    // 这里与 add 方法中的代码是一模一样的，只是针对的是结点 retNode（返回的子树的新的根结点）
    
    retNode.height = 1 + Math.max(getHeight(retNode.left), getHeight(retNode.right));

    // 计算一下平衡因子
    int balanceFactor = getBalanceFactor(retNode);
// 在失衡的时候，做平衡维护

    // LL
    if (balanceFactor > 1 && getBalanceFactor(retNode.left) >= 0) {
        return rightRotate(retNode);
    }

    // RR
    if (balanceFactor < -1 && getBalanceFactor(retNode.right) <= 0) {
        return leftRotate(retNode);
    }

    // LR
    if (balanceFactor > 1 && getBalanceFactor(retNode.left) < 0) {
        retNode.left = leftRotate(retNode.left);
        return rightRotate(retNode);
    }

    // RL
    if (balanceFactor < -1 && getBalanceFactor(retNode.right) > 0) {
        retNode.right = rightRotate(retNode.right);
        return leftRotate(retNode);
    }
    return retNode;
}
```

## 使用 LeetCode 上的问题的测试用例测试我们编写 AVL 数据结构

## 一些关于 BST 的知识复习


+ 从根结点开始，一直向左边走，会来到 BST 的最小值；
+ 从根结点开始，一直向右边走，会来到 BST 的最大值；
+ 在 insert 的时候，做了比较的工作，remove 最小和最大的时候，就只须要按照位置走下去就可以了；
+ BST 结点的删除操作，有些麻烦，但是还是要掌握的；

+ 添加泛型。

```java
public class BST<K extends Comparable<K>, V>{
    
}
```


### 非递归的前序遍历实现

```java
// 非递归的前序遍历实现
public void preOrder1() {
    Stack<Node> stack = new Stack<>();
    Node currentNode = root;

    Node tempNode;
    while (currentNode != null || !stack.isEmpty()) {
        while (currentNode != null) {
            System.out.printf("%s ", currentNode.key);
            stack.push(currentNode);
            currentNode = currentNode.left;
            // 等到左边全部遍历完成以后，stack 里面就存好了刚刚访问的顺序
            // 这些 Node 的左边节点全部访问过了
        }
        if (!stack.isEmpty()) {
            tempNode = stack.pop();
            currentNode = tempNode.right;
        }
    }
}

// 非递归的前序遍历实现2
public void preOrder2() {
    Stack<Node> stack = new Stack<>();
    stack.add(root);
    Node currentNode = null;
    while (!stack.isEmpty()) {
        currentNode = stack.pop();
        System.out.printf("%s ", currentNode.key);
        // 特别注意：应该先加入右边节点到栈中
        if (currentNode.right != null) {
            stack.add(currentNode.right);
        }
        if (currentNode.left != null) {
            stack.add(currentNode.left);
        }
    }
}
```

### 二分搜索树的中序遍历的非递归实现

```java
// 二分搜索树的中序遍历的非递归实现
public void inOrder1() {
    Stack<Node> stack = new Stack<>();
    Node currentNode = root;
    Node tempNode = null;

    while (currentNode != null || !stack.isEmpty()) {
        while (currentNode != null) {
            stack.add(currentNode);
            currentNode = currentNode.left;
        }
        if (!stack.isEmpty()) {
            tempNode = stack.pop();
            System.out.printf("%s ", tempNode.key);
            currentNode = tempNode.right;
        }
    }
}
```

### 返回前驱和后继，是不是很眼熟啊，为什么要这么做呢？

```java
// 返回以 node 为根的二分搜索树中，小于等于 key 的最大值
private Integer floor(Node node, int key) {
    if (node == null) {
        return null;
    }
    if (node.key == key) {
        return node.value;
    }
    if (key < node.key) {
        return floor(node.left, key);
    }
    Integer tempValue = floor(node.right, key);
    if (tempValue != null) {
        return tempValue;
    }
    return node.value;
}

// 返回以 node 为根的二分搜索树中，小于等于 key 的最大值
private Integer ceiling(Node node, int key) {
    if (node == null) {
        return null;
    }
    if (key == node.key) {
        return node.value;
    }
    if (key > node.key) {
        return ceiling(node.right, key);
    }
    Integer tempValue = ceiling(node.left, key);
    if (tempValue != null) {
        return tempValue;
    }
    return node.value;
}
```

（本节完）

