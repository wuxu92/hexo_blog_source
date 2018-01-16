---
date: 2015/10/11 12:34:50
title: 使用golang实现简单排序算法及其性能比较
categories:
- programming
tags:
- lang
- golang
- go
---

这几天重新看了下三个简答的排序算法：选择排序，插入排序和希尔排序，并使用golang做了一个简单的实现与测试。本科学习算法的时候基础太差，又不认真，现在简单的算法理解起来都很吃力，希尔排序算法看了好久勉强看懂，竟也不是完全理解，用go语言实现以下其实都还是比较简单的,但是由于golang暂时还没有泛型支持，所以只能对int类型切片做一个简单实现了。

## 选择排序 ##
选择排序是最简单的排序方法了，实现很简单：

```golang
func SelectSort(arr []int) []int {
	length := len(arr)
	for i:=0; i<length; i++ {
		for j := i+1; j<length; j++ {
			if arr[j] < arr[i] {
				arr[i], arr[j] = arr[j], arr[i]
			}
		}
	}
	return arr
}
```
冒泡排序的特点是每次只在i之后的元素做比较和排序
<!-- more -->
## 插入排序 ##
插入排序是每次只和已经有序的i之前的元素比较和交换

```
func InsertSort(arr []int) []int{
	length := len(arr)
	for i:=0; i<length; i++ {
		for j:=i; j>0 && arr[j-1] > arr[j]; j-- {
			arr[j-1], arr[j] = arr[j], arr[j-1]  
		}
	}
	return arr
}
```
插入排序存在的问题是每次交换只是相邻元素的交换，如果一个很小的元素的初始位置很靠后，那么需要比较和交换很多次才能被交换到最终的位置。这样使用一次交换很长一段距离的改进“希尔排序”能极大提升速度。

## 希尔排序 ##
希尔排序是先使得数组中任意间隔为h的元素都是有序，希尔排序使用一个步长序列(h)来确定每一次排序的h的值。h序列的选择对性能有影响，但是并有没有对所有输入模型都很好的最佳h序列。

> 已知的最好步长序列是由Sedgewick提出的(1, 5, 19, 41, 109,...)，该序列的项来自9 * 4^i - 9 * 2^i + 1和2^{i+2} * (2^{i+2} - 3) + 1这两个算式。这项研究也表明“比较在希尔排序中是最主要的操作，而不是交换。”用这样步长序列的希尔排序比插入排序和堆排序都要快，甚至在小数组中比快速排序还快，但是在涉及大量数据时希尔排序还是比快速排序慢。
> 另一个在大数组中表现优异的步长序列是（斐波那契数列除去0和1将剩余的数以黄金分区比的两倍的幂进行运算得到的数列）：(1, 9, 34, 182, 836, 4025, 19001, 90358, 428481, 2034035, 9651787, 45806244, 217378076, 1031612713,…)[2]

一个简单的实现（使用序列 ：1，4，13，40）：

```golang
func ShellSort(arr []int) []int {
	length := len(arr)
	// 确定h
	h := 1
	for ;h<length/3; {
		h = h*3+1
	}
	for h>=1 {
		for i:=h; i<length; i++ {
			for j:=i; j>=h && arr[j]<arr[j-h]; j -= h {
				arr[j], arr[j-h] = arr[j-h], arr[j]
			}
		}
		h /= 3
	}
	return arr
}
```
其实可以发现，最后h=1时，里面两层for循环就是一个完整的插入排序，之前的h做的h数组有序操作是一些准备工作，使得各个元素的位置都比较接近他们最终的位置而减少最后交换的次数。

> 大量实验证明，平均每个增幅所带来的比较次数约为N^(1/5)。只有在N很大是这个增长幅度才会变得明显。

## 归并排序 ##
归并排序使用了将两个小的有序数组合并成一个大有序数组的思想。它的性质是保证对于任意长度的N的数组排序所花费的时间和NlogN成正比，缺点是所需要的额外空间与N成正比（需要与原数组同样大小的额外空间）。
归并排序有自顶向下和自底向上两种方式，下面是自定向下的排序实现：

```golang
var tmp []int
func MergeSort(arr []int) []int {
	tmp = make([]int, len(arr))
	mergeInnerSort(arr, 0, len(arr)-1)
	return arr
}

func mergeInnerSort(arr []int, lo, hi int) {
	if lo >= hi {
		return
	}
	// 省去最后一层递归，不知道实际效果怎么样
	if hi-lo == 1 {
		if arr[lo] > arr[hi] {
			arr[lo], arr[hi] = arr[hi], arr[lo]
		}
		return
	}
	mid := (lo + hi) / 2
	mergeInnerSort(arr, lo, mid)
	mergeInnerSort(arr, mid+1, hi)
	merge(arr, lo, mid, hi)
}

func merge(arr []int, lo, mid, hi int) {
	for i:=lo; i<=hi; i++ {
		tmp[i] = arr[i]
	}
	j := mid+1
	for k:=lo; k<=hi; k++ {
		if lo>mid {
			arr[k] = tmp[j]
			j++
		} else if j > hi {
			arr[k] = tmp[lo]
			lo++
		} else if tmp[lo] > tmp[j] {
			arr[k] = tmp[j]
			j++
		} else {
			arr[k] = tmp[lo]
			lo++
		}
	}
}
```
我们知道插入排序对于小数组的性能是非常好的，可以在归并排序的小数组使用插入排序代替递归。这样能带来一定的性能提升。
另外可以使用从下往上归并的思路，即先归并小数组，再不断长大。实现如下：

```golang
/**
 * merge sort from down to up
 */
func MergeDToU(arr []int) []int {
	n := len(arr)
	var sz, lo int
	tmp = make([]int, n)
	for sz=1; sz < n; sz += sz {
		for lo=0; lo<n-sz; lo += sz+sz {
			merge(arr, lo, lo+sz-1, min(lo+sz+sz-1, n-1))
		}
	}
	return arr
}

```
相比递归实现的算法更加简单，性能测试显示它们的速度并没有太大区别。自底向上的实现稍微慢一点点，但是差的不多。

## 快速排序 ##
快速排序是使用最为广泛的排序算法，它的主要缺点是非常脆弱，在实践时要很小心才能避免低劣的性能。很多时候会使得快速排序的性能退化到平方级别。

快速排序是一种分治的排序算法，使用切分和交换的思路实现。快排的要点在于切分以及切分点的选择。使用Golang实现的简单的快排如下：

```golang
func QuickSort(arr []int) []int {
	innerQuickSort(arr, 0, len(arr)-1)
	return arr
}

func innerQuickSort(arr []int, lo, hi int) {
	fmt.Println("lo, hi", lo, hi)
	if lo >= hi {
		return 
	}
	mid := innerParrtition(arr, lo, hi)
	innerQuickSort(arr, lo, mid)
	innerQuickSort(arr, mid+1, hi)
}

func innerParrtition(arr []int, lo, hi int) int {
	i, j := lo+1, hi
	k := arr[lo]
	for {
		for i<hi && k >= arr[i]{
			i++
		}
		for j>lo && k <= arr[j] {
			j--
		}
		if i>=j {
			break
		}
		arr[i], arr[j] = arr[j], arr[i]  // 交换
	}
	arr[lo], arr[j] = arr[j], arr[lo]
	return j
}
```
在快排的实现中要注意的是边界条件不要越界，这是很容易出现问题的地方。如上的实现方式中，左侧扫描最好在遇到大于等于切分元素的时候停下来，而右侧扫描则是遇到小于等于切分元素时停下来，尽管这样可能导致一些不必要的等值元素的交换，但是在某些典型应用中，能够避免算法的运行时间变为平方级别。

我们知道快排对于随机数组的性能是最好的，对于基本有序的数组的性能是比较差的。

快速排序有一些比较成熟的改进方法，当所示现的排序方法对性能有很高的要求，会被很多次执行或者作为库函数的话，这些优化是值得考虑的。

对小数组切换到插入排序，把 

```
if lo>=hi {
	return
}
```

改进为
 
```
if lo >= hi {
	InsertSort(arr, lo, hi)
	return
}
```

- 三取样切分：在选取切分点的时候使用一个长度为3的子数组的中位数来切分数组。
- 三向分切的快速排序，对于规模很大的数组，直接将数组切分为3段，这样可以取得更加的效率，但是实现更加复杂。

在快速排序之前，推荐随机地将数组打乱，这是值得的，这样能够防止出现最坏情况，并使执行时间可以预计。


## 性能测试 ##
在我的笔记本上面使用这三个排序算法对一个1000000规模的数组进行排序。

```golang
package sort

func TestPressure(t *testing.T) {
	var input [10000000]int
	randSource := rand.NewSource(time.Now().Unix())
	r := rand.New(randSource)
	for i:=0; i<10000000; i++ {
		input[i] = r.Intn(500000)
	}
	_ = ShellSort(input[:])
	endTime3 := time.Now().UnixNano()
	fmt.Println("shell", len(input),"sort cost: ", endTime3-endTime2, "nano second")
	
	var input2 [10000000]int
	for i:=0; i<10000000; i++ {
		input2[i] = r.Intn(500000)
	}
	endTime3_ := time.Now().UnixNano()
	MergeSort(append(input2[:],12))
	endTime4 := time.Now().UnixNano()
	fmt.Println("merge", len(input2),"sort cost: ", endTime4-endTime3_, "nano second")

	endTime5_ := time.Now().UnixNano()
	QuickSort(append(input2[:], 12))
	endTime6 := time.Now().UnixNano()
	fmt.Println("quick", len(input2),"sort cost: ", endTime6-endTime5_, "nano second")

}
```
插入排序和选择排序的性能都比较差，就删掉了。这里对10,000,000大小的切片进行排序，希尔排序和归并排序的性能结果如下:

```
shell 10000000 sort cost:  4285846600 nano second
merge 10000000 sort cost:  2223476600 nano second
quick 10000000 sort cost:  1442955700 nano second
```
希尔排序与归并排序性能相差一倍的样子，快速排序则是最快的。

对排序方法写一些测试用例

**这里有一些问题，数组作为引用传递在后面的测试会被前面的覆盖，先使用append方法返回一个新的切片，在切掉最后一个添加的元素来实现新切片功能。也可以make一个新的使用copy方法**

```
package sort

import (
	"testing"
	"reflect"
	"fmt"
	"time"
	"math/rand"
	)

type OrderTestCase struct {
	input, expect []int
}

var testCases = []OrderTestCase {
	{ []int{2,1,9,11,5,6}, []int{1,2,5,6,9,11} },
	{ []int{100,9,8,6,5,4}, []int{4,5,6,8,9,100} },
	{ []int{100,9,8,8,8,4}, []int{4,8,8,8,9,100} },
	{ []int{100,90,118,6000,5,4}, []int{4,5,90,100,118,6000} },
}

func (st *OrderTestCase) RunSelectTest(t *testing.T) {
	var in = append(st.input, 0)[:len(st.input)]
	fmt.Println("input s", in)
	var result = SelectSort(in)
	if !reflect.DeepEqual(result, st.expect) {
		t.Error("Unexpected result, expected: ", st.expect, " result:", result)
	}
}

func TestSelect(t *testing.T) {
	for _, testcase := range testCases {
		testcase.RunSelectTest(t)
	}
}

func (ot *OrderTestCase) RunInsertTest(t *testing.T) {
	var in = append(ot.input, 0)[:len(ot.input)]
	fmt.Println("input i", ot.input)
	res := InsertSort(in)
	if !reflect.DeepEqual(res, ot.expect) {
		t.Error("Unexpected result, expect: ", ot.expect, " result: ", res)
	}
}

func TestInsert(t *testing.T) {
	for _, testcase := range testCases {
		testcase.RunInsertTest(t)
	}
}

func (ot *OrderTestCase) RunShellTest(t *testing.T) {
	in := append(ot.input, 0)[:len(ot.input)]
	fmt.Println("input shell: ", in)
	res := ShellSort(in)
	if !reflect.DeepEqual(res, ot.expect) {
		t.Error("Unexpected result, expect: ", ot.expect, " result: ", res)
	}
}
func TestShell(t *testing.T) {
	for _, testcase := range testCases {
		testcase.RunShellTest(t);
	}
}

func (ot *OrderTestCase) RunMergeTest(t *testing.T) {
	in := append(ot.input, 0)[:len(ot.input)]
	fmt.Println("input m", in)
	res := MergeSort(in)
	if !reflect.DeepEqual(res, ot.expect) {
		t.Error("Unexpected result, expect: ", ot.expect, " result: ", res)
	}
}

func TestMerge(t *testing.T) {
	for _, testcase := range testCases {
		testcase.RunMergeTest(t)
	}
}
```

使用reflect.DeepEqual()方法比较两个切片是否完全相等。
