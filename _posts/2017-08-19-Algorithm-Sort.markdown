---
layout: post
title:  "Sorting Algorithm"
date:   2017-08-19 17:46:29 +0100
tags: [JAVA,Algorithm]
author: Temi Lee
modifyauthor: Temi Lee
lastmodify: 2017-08-21 03:46:29
---


**First Of All:**
讲下有关算法复杂度的三个符号:
- O (big o) eg:O(f(n)) 给出了算法运行时间的上界,也就是最坏情况下的时间复杂度
- Θ (Theta) Θ(f(n)) 表示算法运行时间的上界和下界,表示渐进的确界，并非所有的算法都有 Θ(f(n))
- Ω (Omega) Ω(f(n)),给出了算法运行时间的下界,也就是最好情况下的时间复杂度

一些排序算法:
- **冒泡排序:** 两两比较相邻记录的关键字,如果反序则交换 时间复杂度: Ω(n) , O(n^2)

*冒泡排序图示:*
![冒泡排序][1]

*冒泡排序 java 代码:*

{% highlight java %}

/*
 * 冒泡排序
 * 使用双重 for 循环比较相邻的元素。如果第一个比第二个大，就交换他们两个。
 * 对每一对相邻元素作同样的工作，从开始第一对到结尾的最后一对。在这一点，最后的元素应该会是最大的数。
 * 针对所有的元素重复以上的步骤，除了最后一个。
 * 持续每次对越来越少的元素重复上面的步骤，直到没有任何一对数字需要比较。
 * @param numbers 需要排序的整型数组
 */
public static void bubbleSort(int[] numbers)
{
    int temp = 0;
    int size = numbers.length;
    for(int i = 0 ; i < size-1; i ++)
    {
    for(int j = 0 ;j < size-1-i ; j++)
    {
        if(numbers[j] > numbers[j+1])  //交换两数位置
        {

        // 使用中间变量交换两个变量值
        temp = numbers[j];
        numbers[j] = numbers[j+1];
        numbers[j+1] = temp;

        //不使用中间变量的处理方法:(a = 3;b = 5; a = a+b ; b = a-b; a = a-b)
        //nubers[j] = nubers[j] + nubers[j+1]
        //nubers[j+1] = nubers[j] - nubers[j+1]
        //nubers[j] = nubers[j] - nubers[j+1]
        }
    }
    }
}

{% endhighlight %}

- **快速排序:** 利用 `分治` 的思想通过一趟排序将要排序的数据分割成独立的两部分，其中一部分的所有数据都比另外一部分的所有数据都要小，然后再按此方法对这两部分数据分别进行快速排序，整个排序过程可以递归进行，以此达到整个数据变成有序序列。
    时间复杂度:Ω(f(n)) O(nlogn)

*快速排序图示:*

{% highlight java %}



{% endhighlight %}

图中黄色为基准位，排序过程中需递归将基准位左边的数值小于基准位右边的数值

![快速排序][2]

[1]: /img/blog/algorithm/maopaosort.gif
[2]: /img/blog/algorithm/quicksort.gif