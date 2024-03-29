---
title: "My Notes on Grooking Algorithms"
date: 2022-03-16T15:14:08+01:00
draft: false
tags: ["algorithms"]
categories: ["professional", "learning"]
author: "Victor Avelar"
# post features
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "Grooking algorithms is an illustrated guide on algorithms and data structures."
canonicalURL: ""
disableHLJS: true # to disable highlightjs
disableShare: false
disableHLJS: false
hideSummary: false
searchHidden: false
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
cover:
    image: "https://images.manning.com/book/3/0b325da-eb26-4e50-8a2a-46042c647083/Bhargava-Algorithms_hires.png" # image path/url
    alt: "Bhargava-Algorithms" # alt text
    caption: "Grookign algorithms by Aditya Bhargava" # display caption under cover
    relative: false # when using page bundles set this to true
    hidden: false # only hide on current single page
editPost:
    URL: "https://github.com/VictorAvelar/victoravelar.dev/content"
    Text: "Suggest Changes" # edit text
    appendFilePath: true # to append file path to Edit link
---

> The book comes with code samples using python, I did my best to rewrite most of them using golang.
>
> Some of the most complex exercises deserve its own article and I will try to do so.
>
> If you find the notes interesting, you should consider [getting a copy of the book](https://www.manning.com/books/grokking-algorithms).

> ### :warning: This is a work in progress, you can expect the article to grow.

# Chapter 1 - Introduction

An algorith is a set of instructions for accomplishing a task. Every piece of code could be called an algorithm.

## Binary Search

Binary search is an algorithm that receives a sorted list as input (:warning: binary search won't work if the list is not sorted.) and a search subject. It returns the position in the list where the search subject is located.

```go
// This is go(ish) pseudo code.
list := []int{5,10,15,20, 25}

fmt.Println(binarySearch(list, 10))
// Output: 1
```

In general, for any list of size `n`, binary search will take `log n` steps to run in the worst case. In comparison, linear search will take `n` steps.

> :bulb: When speaking about time/space complexity the logarithm is always base 2

### Using it in go

There are the three custom binary search functions: [sort.SearchInts](https://golang.org/pkg/sort/#SearchInts), [sort.SearchStrings](https://golang.org/pkg/sort/#SearchStrings) or [sort.SearchFloat64s](https://golang.org/pkg/sort/#SearchFloat64s).

```go
package main

import (
    "fmt"
    "sort"
)

func main() {
   list := []int{5,10,15,20, 25}

   fmt.Println(sort.SearchInts(list, 10))
}
```

Go playground: https://go.dev/play/p/22tGiSz-z6D

### :bulb: Go 1.18 and generics

Starting from v1.18 generics support is available in Go, with this release also an experimental package arrived that offers a _"general"_ binary search implementation for slices.

In the long run this implementation should replace the 3 functions available in v1.17 and before.

- Slice pkg: https://pkg.go.dev/golang.org/x/exp/slices#BinarySearch

Binary search is an extract from the original implementation :top:

```go
// BinarySearch searches for target in a sorted slice and returns the smallest
// index at which target is found. If the target is not found, the index at
// which it could be inserted into the slice is returned; therefore, if the
// intention is to find target itself a separate check for equality with the
// element at the returned index is required.
func BinarySearch[Elem constraints.Ordered](x []Elem, target Elem) int {
	return search(len(x), func(i int) bool { return x[i] >= target })
}

// where search is defined as
func search(n int, f func(int) bool) int {
	// Define f(-1) == false and f(n) == true.
	// Invariant: f(i-1) == false, f(j) == true.
	i, j := 0, n
	for i < j {
		h := int(uint(i+j) >> 1) // avoid overflow when computing h
		// i ≤ h < j
		if !f(h) {
			i = h + 1 // preserves f(i-1) == false
		} else {
			j = h // preserves f(j) == true
		}
	}
	// i == j, f(i-1) == false, and f(j) (= f(i)) == true  =>  answer is i.
	return i
}

---
// and then it can be used as follows
list := []int{5,10,15,20,25}
want := 10

if idx := BinarySearch(list, want); list[idx] == want {
    return idx
} else {
    return nil
}
```

## Running time

Generally you want to choose the most efficient algorithm whether you're optimizing for time or space.

## Big O

It's a special notation that tells you the rate in which the execution time or the necessary space grows.

> :bulb: Big O notation is about the worst case-scenario.

Common notations are:

- `O(1)` or constant
- `O(n)` or linear
- `O(n^2)` or quadratic
- `O(log n)` or logarithmic
- `O(2^n)` or exponential
- `O(n!)` or factorial
- `O(n log n)` or linearithmic

:pushpin: `n` == `len(list)`

Example: Operations performed for lists of size 1, 8, 64, and 1024.

| Big O      | len(1) | len(8) | len(64)           | len(1024)                                 |
| ---------- | ------ | ------ | ----------------- | ----------------------------------------- |
| O(1)       | 1      | 1      | 1                 | 1                                         |
| O(n)       | 1      | 8      | 64                | 1024                                      |
| O(n^2)     | 1      | 64     | 4096              | 1048576                                   |
| O(log n)   | 1      | 3      | 6                 | 10                                        |
| O(2^n)     | 1      | 256    | 1.8446744 x 10^19 | 1.797693134862315907729305190789 x 10^308 |
| O(n!)      | 1      | 40320  | 1.2688693 X 10^89 | infinity (too big to compute)             |
| O(n log n) | 1      | 24     | 384               | 10240                                     |

# Chapter 2 - Selection sort

## How memory works

Each time you want to store something in memory, you need to ask the computer for some space, then you will get the address of the next available memory slot and you can store your information in there.

```go
// We are telling go to store 100 as a variable which will use memory space
variable := 100

// If we want to get the address in memory for our variable we need to do this
address := &variable

println(address, variable) // outputs something like 0xc00003c768 100
```

## Arrays and linked lists

### Linked lists

A linked list contains a reference to the location of the next item in the list, this way a bunch of memory addresses can be linked together.

Example:

```
a->b->c->d->null
```

This feature allow items to be stored anywhere in memory, you can always reach them using the address stored in the list.

### Arrays

An array is a continous list stored in memory where you know the location and value of all items.

Example:

```
[]int{0,1,2,3,4}
```

This forces an array to allocate continous space in memory leading to data relocation when more space is needed and no contigous space in memory is available.

### Arrays vs Linked lists

- Arrays are great if you want to read random or non-contigous elements.
- Linked lists are great if you are going to read all the items one at a time.

### Terminology

The elements in an array are numbered, most programming languages start with 0 and not 1. When you see a reference to an array element located at index 1 it is refering to the second element of the array and not the first one.

> :bulb: The position of an element in an array is called _index_.

These are the run times for common operations on arrays and linked lists:

|           | Arrays | Lists |
| --------- | ------ | ----- |
| Reading   | O(1)   | O(n)  |
| Insertion | O(n)   | O(1)  |

### Insertions

Whit lists it is easy to insert, you need to change the pointers to the next and current elements.

```go
type node struct {
	Value int
	Next *node
}

root := node{10, nil}

// Imagine we need to insert another node into our list
root.Next = &node{5, nil}

// Imagine we need to insert another node into our list
root.Next = &node{5, nil}

fmt.Printf("root is %v and next is %v\n", root, root.Next)
// Output: root is {10 0xc000108050} and next is &{5 <nil>}

// Lets imagine now they asks us to insert a new node with value 8 after the first node.

n := &node{8, nil}
n.Next = root.Next
root.Next = n
fmt.Printf("root is %v and next is %v, and next.Next is %v", root, root.Next, root.Next.Next)
// Output: root is {10 0xc000096260} and next is &{8 0xc000096230}, and next.Next is &{5 <nil>}
```

Go playground: https://go.dev/play/p/V_zOwyM936O

### Deletions

When it comes to deletions the situation doesn't change much, lists are better, you just need to move the pointer and :bomb: an item is off the list.

```go
// main()
type node struct {
	Val  int
	Next *node
}

root := &node{10, nil}

root.Next = &node{9, nil}

root.Next.Next = &node{8, nil}

n := root

for n != nil {
	fmt.Println(n.Val)
	n = n.Next
}

// Now let's suppose we only want even numbers in our list,
// so we need to remove the 9

root.Next = root.Next.Next

n = root

for n != nil {
	fmt.Println(n.Val)
	n = n.Next
}

```

Go playground: https://go.dev/play/p/HWwpT8tteu1

With arrays everything needs to be moved but unlike insertions deletions will always work.

> :bulb: array insertions can fail when there is no more space in memory and the array cannot be dynamically extended.

### :bookmark:

It is a common practice to keep track of the first and last element of a linked list, that way removing or inserting at the start / end of the list is always O(1).

There are two different types of access, _random access_ and _sequential access_.

**:pushpin: Linked lists can only do sequential access**

**:pushpin: Arrays are better if you need random access**

> :bulb: Random access means that you can inmmediatly jump to any point of the array.

```go
arr := []int{1,2,3,4,5}

// Random access to 4
arr[3] // prints 4

// With a list you will get sequential access:
// Image the list above is represented using linked nodes: 1->2->3->4->5->nil
root.Next.Next.Next // prints 4
```

Common run time for operations on arrays and linked lists.

|          | Arrays | Lists |
| -------- | ------ | ----- |
| Reading  | O(1)   | O(n)  |
| Writing  | O(n)   | 0(1)  |
| Deleting | O(n)   | O(1)  |

## Selection sort

Given a comparable value sort by picking the highest (desc) / lowest (asc) value and moving it to the righ position, this is know as in place sorting, do this until your list is sorted.

```go
// Sort the given array
arr := []int{20,234,2,23,90,4}

func selectionSort(list []int) []int {
	var n = len(list)
	for i := 0; i < n; i++ {
		var minIdx = i
		for j := i; j < n; j++ {
			if list[j] < list[minIdx] {
				minIdx = j
			}
		}
		list[i], list[minIdx] = list[minIdx], list[i]

		// See how the array changes on every pass.
		fmt.Println(list)
	}

	return list
}

// main()
fmt.Println(selectionSort(arr))
```

Go playground: https://go.dev/play/p/FHhposwF6Nk

## Chapter 3 - Recursion

In computer science, recursion is a method of solving a computational problem where the solution depends on solutions to smaller instances of the same problem.

To be able to write a recursive function, you need one or several base cases and one recursive case, in plain words this means that your function needs rules to know when to stop and what to do when those rules are not met.

The most famous example to practice recursion is creating a function that returns the nth number in Fibonnaci's sequence:

```go

func nthFib(n int) int {
	// Initial base case, when the requested number is 0 we know
	// we don't need to do any work.
	if n == 0 {
		return 0
	}

	// Another base case: The first 2 fib numbers after 0 are 1
	// 0 1 1
	if n <=2 {
		return 1
	}

	// Now we need to write our recursive case, we know the fib sequence is created
	// by adding the 2 previous numbers before the one you need.
	//
	// To illustrate, let's calculate the first 5 fib numbers:
	// 0, 1, 1, 2, 3, 5.
	// From that example we visualize that the fifth fib number is creating by
	// adding the numbers at position 4 and 3, or 3 + 2.
	// Therefore we can create our recursive case by saying:
	return nthFib(n - 1) + nthFib(n - 2)
	// This code will always work because we now that in order to hit our recursive
	// case our n number needs to be > 0.
}

```

Go playground: https://go.dev/play/p/Y5Vdg6VcMUW

### The stack

A stack is a data structure where new information is pushed to the front of the stack and when you need to retrieve a workload, you will always take whatever is at the very top of the stack.

Example:

```go
// considering we have push and read as functions and:
// push adds information to the top of our stack while read
// retrieves the record at the top, then.

stack.push(1)
stack.push(10)
stack.push(7)

println(stack.read()) // prints 7
```

Your computer performs processes using an internal stack knwon as the `call stack`, it will allocate a stack frame for every action it needs to perform.

### Recursion and the call stack

Recursive functions leverage the internal call stack to avoid using additional data structures.

Imagine a function called `factorial(int num) int` that calculates the factorial value of the given number, to write it in a recursive way we do something like:

```go
func factorial(int num) int {
	// declare your base case
	if num == 1 {
		return 1
	}

	// declare your recursive case
	return num * factorial(num - 1)
}
```

Let's check how the call stack will be used to calculate the result of `factorial(3)`

_on the stack I will use f to refer to factorial_

| Code                   | Stack                |
| ---------------------- | -------------------- |
| factorial(3)           | f(3)                 |
| factorial(2)           | f(3) -> f(2)         |
| factorial(1)           | f(3) -> f(2) -> f(1) |
| (base case) = return 1 | f(3) -> f(2)         |
| (2 \* 1) = return 2    | f(3)                 |
| (3 \* 2) = return 6    | (empty)              |

See how first the program performs three calls to factorial and then starts using the result to compute the subsequent still pending calls in the stack.

### Alternatives to stacks

- Write your code using loops
- You can use `tail recursion` (advanced) (not supported by all programming languages)
