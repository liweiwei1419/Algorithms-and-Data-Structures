---
title: 【算法日积月累】10-堆排序、heapify、原地堆排序
date: 2019-01-11 08:00:00
author: liwei
top: false
mathjax: true
summary: 本文介绍了 heapify 与原地堆排序。
categories: 算法与数据结构
tags:
  - 优先队列
permalink: algorithms-and-data-structures/heapify-and-heap-sort
---

# 【算法日积月累】10-堆排序、heapify、原地堆排序

---

[TOC]

---

我们在上一小节，介绍了将一个元素 insert 到最大堆中，和从最大堆中取出一个元素，仅仅通过这两个操作，就可以完成排序任务。方法很简单，把待排序数组中的元素全部 insert 到最大堆里，然后再一个一个取出来，因为我们要按照升序排序，因此从后向前放置从最大堆中拿出的元素。

下面的代码是我以前用 Java 写的。

![image-20190125203100954](https://ws2.sinaimg.cn/large/006tNc79ly1fzj3ufp9k7j30qm0i8te4.jpg)

这个方法有一个缺点，那就是要使用和数组元素个数相等的最大堆空间，即空间复杂度是 $O(n)​$。

而这一节我们要介绍的是一种直接使用堆排序的 `sink` 方法完成排序任务的方法，这种方法仅使用 $O(1)$ 的空间复杂度，用于一个临时表变量的存储。

首先，我们介绍一个操作，名为 heapify 。

### 什么是 heapify

使得一个数组是堆有序的操作就叫做“heapify”。具体的做法是：从最后一个非叶子结点开始到索引为 $0$ 的位置，逐个 `sink`。

在上一步“堆排序”中，我们注意到，有这样一个操作“把待排序数组**按顺序**放入一个堆（入队）”，这一步得一个接着一个按照顺序放入堆中，实际上，可以通过一个称之为 heapify 的操作，让这个数组自行调整成一个最大堆，即使之“堆有序”，而此时无需借助 $O(n)$ 空间就完成了最大堆的构建。事实上，只需对数组中一半的元素执行 shift down 就可以了。

以下代码还是使用了  $O(n)$ 空间，主要是为了说明 heapify。

heapify 如下所示：从索引的 `self.count // 2` 位置开始，直到索引为 $0$ 的元素结束，逐个下沉，就可以让一个数组堆有序。

说明：索引的 `self.count // 2` 位置是从下到上第 1 个非叶子结点的索引

Python 代码：

```python
class MaxHeap:
    def __init__(self, nums):
        self.capacity = len(nums)
        self.data = [None] * (self.capacity + 1)
        self.count = len(nums)
        self.__heapify(nums)

    def __heapify(self, nums):
        # 挨个赋值
        for i in range(self.capacity):
            self.data[i + 1] = nums[i]
        for i in range(self.count // 2, 0, -1):
            self.__sink(i)
```

heapify 以后挨个取出来，倒着放回去，也可以完成排序，就不用一个一个放进去，做上浮的操作了。整体上排序会比一个一个放进去快一些。

###原地堆排序 

我们把“原地堆排序”拆解为以下 3 个部分：

1、首先，转换思维，堆从索引 $0$ 开始编号；

代码很多地方都要改，好在并不复杂，正好可以帮助我们复习堆的 `sink` 操作，如果只是用于排序任务，不需要 `swim` 操作；

2、 `sink` 操作要设计成如下的样子，设计一个表示 end 的变量，表示待排序数组的 `[0,end]`（注意是闭区间）范围是堆有序的。

上一节我们将一个数组通过 heapify 的方式逐渐地整理成一个最大堆。而原地堆排序的思想是非常直观的，从 shift down 的操作我们就可以得到启发，堆中最大的那个元素在数组的 0 号索引位置，我们把它与此时数组中的最后一个元素交换，那么数组中最大的元素就放在了数组的末尾，此时再对数组的第一个元素执行 shift down，那么 shift down 操作都执行完以后，数组的第 1 个元素就存放了当前数组中的第 2 大的元素。依次这样做下去，就可以将一个数组进行排序。

我们整理一下，其实这个思想跟“选择排序”是一样的，只不过我们每一轮选出一个当前未排定的数中最大的那个，即“选择排序”+“堆”就是“堆排序”。

![](https://liweiwei1419.github.io/images/algorithms/原地堆排序示意图.jpg)

“堆排序”代码实现的注意事项：

1、此时最大堆中数组的索引从 $0$ 开始计算。与之前索引从 $1$ 开始的最大堆实现比较，性质就发生了变化，但并不会不好找，我们可以自己在纸上画一个完全二叉树就可以很清晰地发现规律：${\rm parent}(i)=\cfrac{i-1}{2}$，${\rm left \quad child}(i) = 2 \times i +1$，${\rm right \quad child}(i) = 2 \times i +2$，最后一个非叶子结点的索引是：$\cfrac{count-1}{2}$；

2、原地堆排序，因为索引从 $0$ 号开始，相应的一些性质在索引上都发生变化了；

3、注意到我们只有 shift down 的操作，对于 shift down 的实现，一些细节就要很小心，shift down 是在一个区间内进行的，我们在设计新的 shift down 方法的实现的时候，应该设计待排序数组区间的右端点。

Python 代码：

```python
def __sink(nums, end, k):
    # end ：数组 nums 的尾索引，
    # __sink 方法维持 nums[0:end]，包括 nums[end] 在内堆有序
    assert k <= end
    temp = nums[k]
    while 2 * k + 1 <= end:
        # 只要有孩子结点：有左孩子，就要孩子结点
        t = 2 * k + 1
        if t + 1 <= end and nums[t] < nums[t + 1]:
            # 如果有右边的结点，并且右结点还比左结点大
            t += 1
        if nums[t] <= temp:
            break
        nums[k] = nums[t]
        k = t
    nums[k] = temp


def __heapy(nums):
    l = len(nums)
    for i in range((l - 1) // 2, -1, -1):
        __sink(nums, l - 1, i)


def heap_sort(nums):
    l = len(nums)
    __heapy(nums)

    for i in range(l - 1, 0, -1):
        nums[0], nums[i] = nums[i], nums[0]
        __sink(nums, i - 1, 0)
```

（本节完）