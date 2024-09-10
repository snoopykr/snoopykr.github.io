---
layout: single
title: "Golang Binary Search"
categories : Golang
tag: [Golang, Binary Search]
---

Golang Binary Search 구현 방법

### Recursive Function 활용

```golang
package main

import (
	"fmt"
)

// Recursive Function을 사용하는 경우 로우 인덱스, 하이 인덱스를 전달 받아야 한다.
func binarySearch(array []int, target int, lowIndex int, highIndex int) int {

	if highIndex < lowIndex {
		return -1
	}

	mid := int((lowIndex + highIndex) / 2)

	if array[mid] > target {

		return binarySearch(array, target, lowIndex, mid)
	} else if array[mid] < target {

		return binarySearch(array, target, mid+1, highIndex)
	} else {
		return mid
	}
}

func main() {
	s := []int{1, 2, 3, 4, 5, 6, 6, 10, 15, 15, 17, 18, 19, 30}

	find := binarySearch(s, 15, 0, len(s)-1)
	fmt.Printf("%d \n", find)
}
```

### Recursive Function 미 사용

```golang
package main

import (
	"fmt"
)

// Recursive Function과 다르게 로우 인덱스, 하이 인덱스를 받을 필요가 없다.
func iterBinarySearch(array []int, target int, lowIndex int, hightIndex int) int {

	startIndex := lowIndex
	endIndex := hightIndex

	var mid int

	for startIndex <= endIndex {

		mid = int((startIndex + endIndex) / 2)

		if array[mid] > target {

			endIndex = mid
		} else if array[mid] < target {

			startIndex = mid + 1
		} else {

			return mid
		}
	}

	return -1
}

func main() {
	s := []int{1, 2, 3, 4, 5, 6, 6, 10, 15, 15, 17, 18, 19, 30}

	find = iterBinarySearch(s, 10, 0, len(s)-1)
	fmt.Printf("%d\n", find)
}
```
