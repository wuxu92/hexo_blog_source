---
title: Golang 实现 word-ladder
date: 2016-03-23 16:22:18
categories:
- leetcode
tags:
- leetcode
- golang 
---

前面一周都在憋论文，终于到昨天出了一版，虽然还有很多工作要做，也准备要休息两天再改了。今天试着写了几道面试题，有段时间没有写代码了，生疏很多了，这里记录一下用golang解决[wordladder这道题](https://leetcode.com/problems/word-ladder/)吧。

这道题理解很简单，

> Given two words (beginWord and endWord), and a dictionary's word list, find the length of shortest transformation sequence from beginWord to endWord, such that:
> Only one letter can be changed at a time
> Each intermediate word must exist in the word list
> For example,
> 
Given:
beginWord = "hit"
endWord = "cog"
wordList = ["hot","dot","dog","lot","log"]
As one shortest transformation is "hit" -> "hot" -> "dot" -> "dog" -> "cog",
return its length 5.
> 
Note:
Return 0 if there is no such transformation sequence.
All words have the same length.
All words contain only lowercase alphabetic characters.
<!-- more -->
解法也比较多，我用的一个生成树查找的方法， 树用队列保存，队列使用golang的切片实现。golang定义结构体很方便，对切片的操作也很方便，思路还是比较简单，只是很久没有使用go了，很多地方有坑，具体在代码中说明，命名比较乱，candi表示candidate的...：

```golang
// WordLadder Q4/Q127
// @see https://leetcode.com/problems/word-ladder/

type candi struct {
  item string	// 当前处理的元素
  deep int		// 当前节点的深度
  dict []string	// 保存当前节点可能的下一个转换的列表，这个列表是不断缩小的
}

func WordLadder(start, end string, dict []string) int {
  if start == end {
    return 0
  }
  var candis []candi
  // 将start作为第一个元素
  candis = append(candis, candi{item: start, deep: 0, dict: dict})

  var item candi
  for len(candis) != 0 {
    // 取第一个元素，并将其从队列中删除
    // 这里要注意不能写成： item, candis := candis[0], candis[1:]，这样candis会成为for里面的局部变量
    item, candis = candis[0], candis[1:]
    // 如果当前item距离end只有1，则直接返回
    if wordDist(end, item.item) == 1 {
      return item.deep
    }

    dict = item.dict
    // 遍历当前点的dict列表，找出下一步
    for i:=0; i < len(dict); i++{
      if wordDist(item.item, dict[i]) == 1 {
        item_dict := append(dict[:i], dict[i+1:]...)
        newItem := candi{item: dict[i], deep: item.deep+1, dict: item_dict}
        candis = append(candis, newItem)
      }
    }
  }
  // 不存在路径则返回-1，题目要求返回0
  return -1
}

func wordDist(a, b string) (dist int) {
  // 虽然保证了长度相同，但是还是检查一下长度
  lena, lenb := len(a), len(b)
  if lena != lenb {
    return -1
  }

  for i:=0; i< lena; i++ {
    if a[i] != b[i] {
      dist++
    }
  }
  return dist
}

```

这个函数的返回和题目要求的不完全一样，比如题目用例返回的是5，这里会返回3是因为没有加上start和end，这其实影响不大了。这种方法的缺点是dict很大的时候，内存开销可能会比较大，因为每一个节点都会保存自己的dict切片。