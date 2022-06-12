---
title: 快排之golang实现
tags: [go 数据结构与算法]
copyright: true
date: 2018-11-18 14:36:38
permalink:
categories: 数据结构与算法
description: 快速排序的go实现
image: https://static001.geekbang.org/resource/image/f0/bb/f00e5b5659834d2bb64a923da90910bb.jpg
---
<p class="description"></p>

<img src="https://" alt="" style="width:100%" />

<!-- more -->

- 快速排序由C. A. R. Hoare在1962年提出。它的基本思想是：通过一趟排序将要排序的数据分割成独立的两部分，其中一部分的所有数据都比另外一部分的所有数据都要小，然后再按此方法对这两部分数据分别进行快速排序，整个排序过程可以递归进行，以此达到整个数据变成有序序列。

利用分治法可将快速排序的分为三步：

- 在数据集之中，选择一个元素作为”基准”（pivot）。
- 所有小于”基准”的元素，都移到”基准”的左边；所有大于”基准”的元素，都移到”基准”的右边。这个操作称为分区 (partition) 操作，分区操作结束后，基准元素所处的位置就是最终排序后它的位置。
- 对”基准”左边和右边的两个子集，不断重复第一步和第二步，直到所有子集只剩下一个元素为止。

快速排序平均时间复杂度为`O(n log n)`,最坏情况为`O(n2)`，不稳定排序。

这里实现了两种方式的快排，第一种是单路的，实现代码如下：

```go
func partition(sortArray []int, left, right int) int {
	key := sortArray[right]
	i := left - 1

	for j := left; j < right; j++ {
		if sortArray[j] <= key {
			i++
			swap(i, j)
		}
	}

	swap(i+1, right)

	return i + 1
}
```



第二种是双路的：

```go
unc partition2(arr []int,left,right int)(p int)  {

	if left > right {
		return
	}

	i,j,pivot := left,right ,arr[left]

	for i<j  {

		for i < j && arr[j] >pivot  {
			j--
		}

		for i < j && arr[i] <= pivot  {
			i++
		}

		if i < j {
			arr[i] ,arr[j] = arr[j],arr[i]
		}

	}

	arr[i],arr[left] = arr[left],arr[i]

	return i
```

测试代码如下：

```go
import ("fmt")


const MAX = 10

var sortArray = []int{41, 24, 76, 11, 45, 64, 21, 69, 19, 36}
func main() {
    fmt.Println("before sort：")

	quickSort(sortArray, 0, MAX-1)

	fmt.Println("after sort:")

}

func quickSort(sortArray []int, left, right int) {
	if left < right {
		pos := partition2(sortArray, left, right)//修改此处测试不同的实现方式
		quickSort(sortArray, left, pos-1)
		quickSort(sortArray, pos+1, right)
	}
}

```

BUT!

要表达快排的思想，还是使用Python比较透彻：

```python
def quickSort(array):

    if len(array) < 2:
        return array
    else:
        pivot = array[0]
        less = [i for i in array[1:] if i <= pivot]
        greater = [i for i in array[1:] if i > pivot]
    return quickSort(less) + [pivot] + quickSort(greater)
```

是不是将快排的分治思想表达地淋漓尽致，简洁美观。

​										

​									End！

<hr />
