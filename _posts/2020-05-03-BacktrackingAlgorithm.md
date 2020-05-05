---
layout:     post
title:      回溯算法的swift实现
subtitle:	  
date:       2020-04-28
author:     Neil
header-img: img/post-bg-coffee.jpeg
catalog: 	 true
tags:
    - 回溯算法的swift实现
---

## 有一个问题：字母大小写全排列  
给定一个字符串S，通过将字符串S中的每个字母转变大小写，我们可以获得一个新的字符串。返回所有可能得到的字符串集合。

示例:  
输入: S = "a1b2"  
输出: ["a1b2", "a1B2", "A1b2", "A1B2"]

输入: S = "3z4"  
输出: ["3z4", "3Z4"]

输入: S = "12345"  
输出: ["12345"]

## 递归

解题思路：新建一个数组，将遇到字母时转换大小写一并加入到数组中，最后返回便是要的结果。其中在加入数组的过程中，如遇到`a1b2`中的`a`，此时不管后面的如何的变化，结果应该是`a+后面的字符串`和`A+后面的字符串`，也即构成一个递归关系，将变换大小写后的字符串重新当作参数传入，并传入字符串的下标，当下标和字符串长度相等时候退出，因为是一个全排列，所以递归的时候每次循环都是从新传入的下标开始，比如把转换后`A1b2`新的字符串重新当作参数传入，这样后面`A1B2`和`A1b2`这样的组合才会出现,`swift`实现如下：


```
var letters:[String] = []

func letterCasePermutation(_ S: String) -> [String] {
    guard !S.isEmpty else {
        return []
    }
    letters.append(S)
    letter(S, 0)
    return letters
}

func letter(_ S: String, _ index:Int) {
    guard !S.isEmpty else {
        return
    }
    guard index < S.count else {
        return
    }
    for i in index..<S.count {
        let index = S.index(S.startIndex, offsetBy: i)
        if S[index] >= "a" ,S[index] <= "z" {
            var news = S
            news.replaceSubrange(index...index,with:S[index].uppercased())
            letters.append(news)
            letter(news,i+1)
        }
        if S[index] >= "A" , S[index] <= "Z" {
            var news = S
            news.replaceSubrange(index...index,with:S[index].lowercased())
            letters.append(news)
            letter(news,i+1)
        }
    }
}
```

## 回溯算法
>解决一个回溯问题，实际上就是一个决策树的遍历过程

回溯算法框架

```
result = []
def backtrack(路径, 选择列表):
    if 满足结束条件:
        result.add(路径)
        return

    for 选择 in 选择列表:
        做选择
        backtrack(路径, 选择列表)
        撤销选择
```

![](../img/local/backtrack4.png)

执行一次深度优先遍历，从树的根结点到叶子结点形成的路径就是一个全排列

这种在遍历的过程中，从深层结点回到浅层结点的过程中所做的操作就叫“回溯”。

```
List<List<Integer>> res = new LinkedList<>();

/* 主函数，输入一组不重复的数字，返回它们的全排列 */
List<List<Integer>> permute(int[] nums) {
    // 记录「路径」
    LinkedList<Integer> track = new LinkedList<>();
    backtrack(nums, track);
    return res;
}

// 路径：记录在 track 中
// 选择列表：nums 中不存在于 track 的那些元素
// 结束条件：nums 中的元素全都在 track 中出现
void backtrack(int[] nums, LinkedList<Integer> track) {
    // 触发结束条件
    if (track.size() == nums.length) {
        res.add(new LinkedList(track));
        return;
    }

    for (int i = 0; i < nums.length; i++) {
        // 排除不合法的选择
        if (track.contains(nums[i]))
            continue;
        // 做选择
        track.add(nums[i]);
        // 进入下一层决策树
        backtrack(nums, track);
        // 取消选择
        track.removeLast();
    }
}
```

从图和代码可以看出`track`中加入以`nums[i]`为起点，逐个加入`nums[i]`中的其他点,当`track`长度和`nums`的长度相等时，重新加入新的`track`得到全排列


参考  
[回溯算法解题套路框架](https://labuladong.gitbook.io/algo/di-ling-zhang-bi-du-xi-lie/hui-su-suan-fa-xiang-jie-xiu-ding-ban)  
[从全排列问题开始理解「回溯」算法（深度优先遍历 + 状态重置 + 剪枝）](https://leetcode-cn.com/problems/permutations/solution/hui-su-suan-fa-python-dai-ma-java-dai-ma-by-liweiw/)