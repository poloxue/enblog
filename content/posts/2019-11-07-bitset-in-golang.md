---
title: "Beyond Basics: Do You Really Understand Bitsets in Go?"
date: 2019-11-07T15:40:46+08:00
draft: true
---

![](https://cdn.jsdelivr.net/gh/poloxue/images@2019-11/2019-11-07-bitset-in-golang-04.png)

I previously wrote a post titled [How to Use Set in Go](https://medium.com/me/stats/post/194e7b34c949), in which I introduced the simplest application scenario of bitset, the state flag, and also mentioned the implementation idea of bitset.

What's the difference between state flags and ordinary sets?

My conclusion is mainly one point: the number of elements in state flags is usually fixed. In general sets, however, the number of elements usually changes dynamically. What problems does this lead to?

Generally, an integer is sufficient to represent all the states in state flags. The largest int64 type has 64 binary bits, which can contain up to 64 elements, which is completely sufficient. But if it's a set, the number and value of elements are usually not fixed.

For instance, a bitset collection might initially only contain a few elements like 1, 2, and 4, which can be represented by a single int64 as follows:

![bitset representation](https://cdn.jsdelivr.net/gh/poloxue/images@main/2019-11-07-bitset-in-golang-01.png)

But what if another element, such as 64 (the range of an int64 is 0-63), is added? This exceeds the range an int64 can represent. What should be done?

If one int64 cannot represent it, then use multiple. The structure at this time is as follows:

![bitset with multiple int64](https://cdn.jsdelivr.net/gh/poloxue/images@2019-11/2019-11-07-bitset-in-golang-02.png)

A slice of int64 perfectly matches the structure above. So, we can define a new type `BitSet` as follows:

```go
type BitSet struct {
	data []int64
	size int
}
```

The `data` member is used to store the collection elements, and the advantage of the slice is that it can dynamically expand.

Also, because the number of elements in a bitset cannot be obtained through the `len` function, and the specific method is a bit more complex, a size field can be added to record the number of elements in the collection. Then you can add a `Size` method.

```go
func (set *BitSet) Size() int {
	return set.size
}
```

# Element Position

After defining the `BitSet` type, a new problem arises: how to locate the position to store the elements? In the context of flag bits, the value of the element is the position, so this problem does not need to be considered. But it's not the case for general collections.

First, let's look at the distribution of binary bits in `BitSet`.

![bitset binary distribution](https://cdn.jsdelivr.net/gh/poloxue/images@2019-11/2019-11-07-bitset-in-golang-03.png)

It looks like a grid with rows and columns. Let's assume `index` represents the row (index) and `pos` represents the column (position). The slice index goes from 0 to n, and n is related to the largest element in the collection.

Next, determine the values of `index` and `pos`. Actually, I've already introduced this in the previous article.

`index` can be obtained by dividing the element value by the word length, i.e., `value / 64`, which can be converted into an efficient bitwise operation, i.e., `value >> 6`.

`pos` can be obtained by taking the modulus of the element value by the word length, i.e., `value % 64`, which can be converted into an efficient bitwise operation, i.e., `value & 0x3f`. This gets the corresponding position, and then `1 << uint(value % 0xf)` can be used to convert the position into a value.

# Code Implementation

Theory aside, let's start coding!

First, define some constants.

```go
const (
	shift = 6    // 2^n = 64's n
	mask  = 0x3f // n=6, i.e., 2^n - 1 = 63, i.e., 0x3f
)
```

These are the two constants mentioned earlier for calculating `index` and `pos`.

Provide two functions for convenient calculation of corresponding values on `index` and `pos`. The code is as follows:

```go
func index(n int) int {
	return n >> shift
}

// Relative to the value of a specific flag in the flag bit scenario
func posVal(n int) uint64 {
	return 1 << uint(n&mask)
}
```

# Constructor Function

A function is provided to create the initial `BitSet`, and it supports setting the initial elements.

The function prototype is as follows:

```go
func NewBitSet(ns ...int) *BitSet {
	// ...
}
```

The output parameter `ns` is a variable-length parameter of type `int`, used to set the initial value in the collection.

If the input parameter `ns` is empty, `new(BitSet)` returns an empty collection.

```go
if len(ns) == 0 {
	return new(BitSet)
}
```

If the length is not empty, then you need to calculate the space to be allocated by calculating the `index` of the largest element.

```go
// Calculate how many spaces to allocate for the bitset
max := ns[0]
for _, n := range ns {
	if n > max {
		max = n
	}
}

// If max < 0, return an empty set directly.
if max < 0 {
	return new(BitSet)
}

// Calculate the index where the maximum value max is located through max >> shift+1
// and index + 1 is the space to be allocated
s := &BitSet{
	data: make([]int64, index(max)+1),
}
```

Now, elements can be added to the `BitSet`.

```go
for _, n := range ns {
	if n >= 0 {
		// n >> shift gets the index position, i.e., the row, generally called index
		// n&mask gets the column, generally called pos, like F1 0 F2 1
		s.data[n>>shift] |= posVal(n)
		// Increase the number of elements
		s.size++
	}
}

// Return the created BitSet
return s
```

All elements have been added!

# Methods of BitSet

Next, the focus is on adding some methods to `BitSet`. Mainly divided into two categories, one is common basic methods like add, delete, check, etc., and the other is specific operations of sets, such as intersection, union, difference.

# Basic Methods

Mainly a few methods, namely `Add` (add), `Clear` (clear), `Contains` (check) and return the number of elements. If you want better performance and space utilization, `Add` and `Clear` also have to consider flexibility.

## contains

First, talk about `Contains`, i.e., check if a certain element exists.

The function is defined as follows:

```go
func (set *BitSet) Contains(n int) bool {
	...
}
```

The input parameter is the element to be checked, and the output is the result of the

 check.

The implementation code is as follows:

```go
// Get the position of the corresponding int64 for the element, if it exceeds the range that data can represent, return directly.
i := index(n)
if i >= len(set.data) {
	return false
}

return set.data[i]&posVal(n) != 0
```

The core is the code `set.data[i]&posVal(n) != 0`, which determines whether a specified element exists.

## clear

Next, discuss `Clear`, to remove a certain element from the collection.

The function is defined as follows:

```go
func (set *BitSet) Clear(n int) *BitSet {
	// ...
}
```

The implementation code is as follows:

```go
// The element cannot be less than 0
if n < 0 {
	return set
}

// Calculate the slice index position, if it exceeds the range represented by the current index, return directly.
i := index(n)
if i >= len(set.data) {
	return set
}

// Check if the element exists
if d[i]&posVal(n) != 0 {
	set.data[i] &^= posVal(n)
	set.size--
}
```

Use `&^` to implement specific bit clearing. Also remember to update `set.size--`.

There's a flaw in the implementation above. If some bits are set to zero, there might be a situation where the high bits are all zero. In this case, you should shrink the data space through reslice.

How to operate specifically?

Check `set.data`, from high to low, find the first `uint64` that is not 0, and reslice based on this. Suppose this method is called `trim`.

The implementation code is as follows:

```go
func (set *Set) trim() {
	d := set.data
	n := len(d) - 1
	for n >= 0 && d[n] == 0 {
		n--
	}
	set.data = d[:n+1]
}
```

## add

Next, talk about the `Add` method, to add a certain element to the collection.

The function is defined as follows:

```go
func (set *BitSet) Add(n int) *BitSet {
	...
}
```

When adding an element, first check whether there is enough space to store the new element. If the index position of the new element is not within the range represented by the current `data`, you need to expand the capacity.

The implementation is as follows:
```go
// Check if there is enough space to store the new element
i := index(n)
if i >= len(set.data) {
	// Expand the capacity to i+1
	ndata := make([]uint64, i+1)
	copy(ndata, set.data)
	set.data = ndata
}
```

Once everything is ready, you can proceed to set the addition. Before adding, first check whether the collection already contains the element. After the addition is complete, remember to update `size`.

The implementation code is as follows:

```go
if set.data[i]&posVal(n) == 0 {
	// Set the element to the collection
	set.data[i] |= posVal(n)
	s.size++
}
```

That's all for the basic methods!

Of course, more methods can be added here, such as finding the next element of the current element, adding all values in a certain range to the collection, and so on.

# Set Methods

After introducing the basic methods, continue to introduce some specific methods for sets, such as intersection, union, difference.

## computeSize

Before officially introducing these methods, first introduce an auxiliary method used to calculate the number of elements in the collection. This method is introduced because there is no way to update `size` like before in add and delete operations for intersection, union, and difference, so it needs to be recalculated.

The implementation code is as follows:

```go
func (set *BitSet) computeSize() int {
	d := set.data
	n := 0
	for i, len := 0, len(d); i < len; i++ {
		if w := d[i]; w != 0 {
			n += bits.OnesCount64(w)
		}
	}

	return n
}
```

This is a non-exportable method, only for internal use. Traverse each `uint64` in `data`, if it's non-zero, count the number of elements in it. The counting of the number of elements uses the `bits.OnesCount64` method in the standard library.

## Method Definition

Continue to introduce several methods of the collection, their definitions are similar, all are operations of one `BitSet` with another `BitSet`, as follows:

```go
// Intersection
func (set *BitSet) Intersect(other *BitSet) *BitSet {
	// ...
}
// Union
func (set *BitSet) Union(other *BitSet) *BitSet {
	// ...
}
// Difference
func (set *BitSet) Difference(other *BitSet) *BitSet {
	// ...
}
```

## intersect

First, introduce `Intersect`, the method to calculate the intersection.

An important premise is that since the intersection is an `and operation`, the result must be in the smaller range set of the two participating operations. So, space allocation and traversal can be reduced to this range.

The implementation code is as follows:

```go
// First, get the length of this smaller range set
minLen := min(len(set.data), len(other.data))

// Allocate space with minLen
intersectSet := &BitSet{
	data: make([]uint64, minLen),
}

// Calculate the intersection by traversing with minLen
for i := minLen - 1; i >= 0; i-- {
	intersectSet.data[i] = set.data[i] & other.data[i]
}

intersectSet.size = set.computeSize()
```

Here, by traversing and performing `and operation` on each `uint64`, the intersection is implemented. After the operation is completed, remember to calculate the number of elements, i.e., the value of `size`, in `intersectSet`.

## union

Next, introduce the union method `Union`.

Its calculation logic is opposite to `Intersect`. The space occupied by the union result and the larger set of the two participating operations are used as the standard.

The implementation code is as follows:

```go
var maxSet, minSet *BitSet
if len(set.data) > len(other.data) {
	maxSet, minSet = set, other
} else {
	maxSet, minSet = other, set
}

unionSet := &BitSet{
	data: make([]uint64, len(maxSet.data)),
}
```
The `unionSet` created here, the `data` allocates space is `len(maxSet.data)`.

Because all elements in the two sets satisfy the final result, but the high part of `maxSet` cannot be calculated through traversal and operation with `minSet`, it can be directly copied into the result.

```go
minLen := len(minSet.data)
copy(unionSet.data[minLen:], maxSet.data[minLen:])
```

Finally, traverse the `data` of the two sets and calculate the remaining part through `or operation`.

```go
for i := 0; i < minLen; i++ ```go
{
	unionSet.data[i] = set.data[i] | other.data[i]
}

// Update and calculate size
unionSet.size = unionSet.computeSize()
```

## difference

Lastly, introduce the method for set difference, `Difference`, which is the operation for set difference.

The space allocated for the `differenceSet` result is determined by the subtracted set `set`. The other operations are similar to `Intersect` and `Union`. Bitwise operations are implemented through `&^`.

```go
setLen := len(set.data)

differenceSet := &BitSet{
	data: make([]uint64, setLen),
}
```

If the length of `set` is greater than `other`, then the part that cannot be operated on for set difference should be copied first.

```go
minLen := setLen
if setLen > otherLen {
	copy(differenceSet.data[otherLen:], set.data[otherLen:])
	minLen = otherLen
}
```

Record `minLen` for the following bitwise operations.

```go
// Traverse data to perform bitwise operations.
for i := 0; i < minLen; i++ {
	differenceSet.data[i] = set.data[i] &^ other.data[i]
}

differenceSet.size = differenceSet.computeSize()
```

# Traversing Elements of the Set

Let's talk separately about the traversal of elements in the set. Previously, the presence of set elements has always been checked through the `Contains` method. Is it possible to traverse all elements in the set?

Let's take another look at the structure of the bitset, as follows:

![bitset structure](https://cdn.jsdelivr.net/gh/poloxue/images@2019-11/2019-11-07-bitset-in-golang-02.png)

In the set above, the first element of the first row of `int64` is 1, with a bit set to zero at the end. By observation, it's noted that the value of the first element is whatever the number of preceding 0s.

The first element in the second row of `int64` doesn't have a 0 at the end, so is its value 0? Of course not, there's also the base of the 64 bits from the previous row, so its value is 64+0.

What rules can be summarized? Well, let's look at the code!

First, the function definition:

```go

func (set *BitSet) Visit(do func(int) (skip bool)) (aborted bool) {
	//...
}
```

The input parameter is a callback function. Through it, you get the value of the element, otherwise, every time you have to write a large loop of computational logic, which is hardly feasible. The return value of the callback function `bool` indicates whether to continue traversal. The return value of `Visit` indicates whether the function ended abnormally.

The implementation code is as follows:

```go
d := set.data
for i, len := 0, len(d); i < len; i++ {
	w := d[i]
	if w == 0 {
		continue
	}

	// Not good at theory, don't know how to describe this. Haha
	// This small piece of code can be understood as the reverse operation from element value to index,
	// but the obtained value is the first position value of the likes of 0, 64, 128.
	// 0 << 6 is still 0, 1 << 6 is 64, 2 << 6 is 128
	n := i << shift
	for w != 0 {
		// 000.....000100 For 64~128, it represents 66, which is 64 + 2, this 2 can be determined by the number of trailing zeros
		// So how to get the number of trailing 0s? The bits.TrailingZeros64 function can be used
		b := bits.TrailingZeros64(w)
		if do(n + b) {
			return true
		}
		// Clear the bit that has been checked
		// To ensure the number of trailing zeros can represent the value of the element
		w &^= 1 << uint64(b)
	}
}
```

It's also very convenient to use, as shown in the example code:

```go
set := NewBitSet(1, 2, 10, 99)
set.Visit(func(n int) bool {
	fmt.Println(n)
	return false
})
```

That's pretty much it!

# Summary

This blog post is mainly based on several open-source packages to introduce the implementation of bitset, such as [bit](https://github.com/yourbasic/bit) and [bitset](https://github.com/willf/bitset). Overall, bitwise operations are just not that intuitive, feels like running out of brain power.

