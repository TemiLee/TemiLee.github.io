---
layout: post
title:  "Sorting Algorithm"
date:   2017-08-19 17:46:29 +0100
tags: LINUX
author: Temi Lee
modifyauthor: Temi Lee
lastmodify: 2017-08-19 17:46:29
---


**First Of All:**
讲下有关算法复杂度的三个符号:
- O (big o) eg:O(f(n)) 给出了算法运行时间的上界,也就是最坏情况下的时间复杂度
- Θ (Theta) Θ(f(n)) 表示算法运行时间的上界和下界,表示渐进的确界，并非所有的算法都有 Θ(f(n))
- Ω (Omega) Ω(f(n)),给出了算法运行时间的下界,也就是最好情况下的时间复杂度

一些排序算法:
- 冒泡排序:两两比较相邻记录的关键字,如果反序则交换 时间复杂度: Ω(n) , O(n^2)

![冒泡排序][1]

- 通过一趟排序将要排序的数据分割成独立的两部分，其中一部分的所有数据都比另外一部分的所有数据都要小，然后再按此方法对这两部分数据分别进行快速排序，整个排序过程可以递归进行，以此达到整个数据变成有序序列。
    时间复杂度:O(nlogn)




[1]: /img/blog/algorithm/maopaosort.gif