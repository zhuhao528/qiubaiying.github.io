---
layout:     post
title:      分治算法多数问题的swift实现
subtitle:	  
date:       2020-04-28
author:     Neil
header-img: img/post-bg-coffee.jpeg
catalog: 	 true
tags:
    - 分治算法swift实现
---

## 分治的思想

   在计算机科学中，分治法是一种很重要的算法。字面上的解释是“分而治之”，就是把一个复杂的问题分成两个或更多的相同或相似的子问题，再把子问题分成更小的子问题……直到最后子问题可以简单的直接求解，原问题的解即子问题的解的合并。这个技巧是很多高效算法的基础，如排序算法(快速排序，归并排序)，傅立叶变换(快速傅立叶变换)……

## 分治算法的步骤

分治法在每一层递归上都有三个步骤：

1. 将原问题分解为若干个规模较小，相互独立，与原问题形式相同的子问题；
2. 若子问题规模较小而容易被解决则直接解，否则递归地解各个子问题
3. 将各个子问题的解合并为原问题的解。

## 多数元素问题

给定一个大小为 n 的数组，找到其中的多数元素。多数元素是指在数组中出现次数大于 ⌊ n/2 ⌋ 的元素。
你可以假设数组是非空的，并且给定的数组总是存在多数元素。

示例 1:

输入: [3,2,3]
输出: 3

示例 2:

输入: [2,2,1,1,1,2,2]
输出: 2

```
class Solution {
    func majorityElement(_ nums: [Int]) -> Int {
        var maxValue = nums[0]
        var maxCount = 0
        for i in 0 ..< nums.count-1 {
            var count = 0
            for j in i+1 ..< nums.count{
                if nums[i] == nums[j] {
                    count += 1
                }
            }
            if count > maxCount {
                maxCount = count
                maxValue = nums[i]
            }
        }
        return maxValue
    }
}
```

上面的代码虽然能够解决问题，但是时间复杂度O(N*N)，时间复杂度较高

### 采用分治算法

如果数 a 是数组 nums 的众数，如果我们将 nums 分成两部分，那么 a 必定是至少一部分的众数。

我们可以使用反证法来证明这个结论。假设 a 既不是左半部分的众数，也不是右半部分的众数，那么 a 出现的次数少于 l / 2 + r / 2 次，其中 l 和 r 分别是左半部分和右半部分的长度。由于 l / 2 + r / 2 <= (l + r) / 2，说明 a 也不是数组 nums 的众数，因此出现了矛盾。所以这个结论是正确的。

这样以来，我们就可以使用分治法解决这个问题：将数组分成左右两部分，分别求出左半部分的众数 a1 以及右半部分的众数 a2，随后在 a1 和 a2 中选出正确的众数。

算法

我们使用经典的分治算法递归求解，直到所有的子问题都是长度为 1 的数组。长度为 1 的子数组中唯一的数显然是众数，直接返回即可。如果回溯后某区间的长度大于 1，我们必须将左右子区间的值合并。如果它们的众数相同，那么显然这一段区间的众数是它们相同的值。否则，我们需要比较两个众数在整个区间内出现的次数来决定该区间的众数。

```
func numCount(_ nums:[Int], _ low :Int , _ high : Int, _ num : Int) -> Int {
    var count = 0
    for i in low ... high {
        if num == nums[i] {
            count += 1
        }
    }
    return count
}

func divideAndConquerAlgorithm(_ nums: [Int] , _ low : Int ,_ high : Int) -> Int {
    
    guard low != high else {
        return nums[low]
    }
                    
    let mid = Int((low + high)/2)
    
    let left =  divideAndConquerAlgorithm(nums, low, mid)
    let right = divideAndConquerAlgorithm(nums, mid+1, high)
    
    if left == right {
        return left
    }
                    
    return numCount(nums, low, mid, left) > numCount(nums, mid+1, high, right) ? left : right
}

func majorityElement(_ nums: [Int]) -> Int {
    
    guard nums.count > 0 else {
        return 0
    }
    
    return divideAndConquerAlgorithm(nums, 0, nums.count-1)
}
```

>执行用时 :176 ms, 在所有 Swift 提交中击败了71.09% 的  
>用户内存消耗 :21.3 MB, 在所有 Swift 提交中击败了100.00%的用户

复杂度分析：

时间复杂度：$O\left( n\log n \right)$。函数 `majorityElement()` 会求解 2 个长度为 n/2​	的子问题，并做两遍长度为的线性扫描。因此，分治算法的时间复杂度可以表示为：

$$
T\left( n \right)=2T\left( \; \frac{
n}{2} \right)+2n
$$

根据 主定理，本题满足第二种情况，所以时间复杂度可以表示为：

$$
\; 
T\left( n \right)
=\Theta \left( n^{\log _{b^{a}}}\log n \right)=\Theta \left(n^{\log _{2^{2}}}\log n \right)=\Theta \left( n\log n \right)
$$

空间复杂度：$ O\left( \log n \right) $。尽管分治算法没有直接分配额外的数组空间，但在递归的过程中使用了额外的栈空间。算法每次将数组从中间分成两部分，所以数组长度变为 1 之前需要进行 $ O\left( \log n \right) $ 次递归，即空间复杂度为 $ O\left( \log n \right) $

比如：[5,6,5,6] 第一次递归是[5,6]和[5,6] 第二次递归是 [5]、[6]、[5]、[6] 所以一共递归两次满足上述公式的解

### 思考

在`let right = divideAndConquerAlgorithm(nums, mid+1, high)`后面添加打印语句最后

```
print(left)
print(right)
```

当遇到数组`[5,6,5]`的时候，根据楼上代码如果打印`left`和`right`打印的结果是

得到的打印结果是

```
--------
5
6
--------
--------
6
5
--------
```
打印结果便是
也即将数组分成了[5,6]和[5] 前者最后算出大数是6 和后者的大数是5 在进行数量的比较，最后得出6 的数量 并不大于 5 最后返回了 5 得到的结果也满足题目的要要求

参考  
[五大常用算法之一：分治算法](https://www.cnblogs.com/steven_oyj/archive/2010/05/22/1741370.html)  
[多数元素题解](https://leetcode-cn.com/problems/majority-element/solution/duo-shu-yuan-su-by-leetcode-solution/)


