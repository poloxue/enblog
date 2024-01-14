---
title: "Mastering Set Implementations in Go: Exploring Map-Based and BitSet Approaches"
date: 2024-01-11T17:06:37+08:00
draft: true
comment: true
---

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-01/2024-01-11-set-in-golang-01.png)

When coding in Go (or Golang), having a solid grasp of data structures is essential for problem-solving. This post will delve into the implementation of a data structure in Go: Set.

## Commonly Data Structures in Golang.

Golang offers a relatively small set of builtin data structures. 

In our daily work, we frequently rely on slice and map. In fact, Go also has an array type, but due to the existence of slice, we rarely use the array type.

Besides the built-in data structures in Golang, the Go container package also provides additional data structures, such as heap, list, and ring. For experienced programmers, understanding how to use them is quite straightforward.

Today, I'd like to explain `set`. 

While some programming languages like Java have these this type of data structure, it's unfortunate that Golang doesn't provide support for them in any form.

## Approach to Implementation

Irrespective of the programming language, we have two ways to implement a set type: `map` and `bitset`.

## Creating Set using Map

As we all know, the keys in a map must be unique, which aligns with the requirements of a set that guarantees the uniqueness of keys in a map type. Additionally, we can check if a key exists using the code _, ok := m[key], which is a highly efficient method.

### Writing the Code

A simple implmentation is as follows:

```go
set := make(map[string]bool) // New empty set
set["Foo"] = true            // Add
for k := range set {         // Loop
    fmt.Println(k)
}
delete(set, "Foo")    // Delete
size := len(set)      // Size
exists := set["Foo"]  // Membership
```

We've created a variable `set` of type map[string]bool to represent a set for storing strings. This is easy to understand. However, there's an issue: the value associated with this map type variable is bool, which means it consumes additional memory in addition to the set itself. This should not be the case when using a dedicated set type.

We can set the value to an empty struct. Why? Empty structs in Golang consume zero memory. To confirm its correctness, let's verify it.

```go
unsafe.Sizeof(struct{}{}) // The result is 0
```

The optimized code is as follows:

```go
type void struct{}
var member void

set := make(map[string]void) // New empty set
set["Foo"] = member          // Add
for k := range set {         // Loop
    fmt.Println(k)
}
delete(set, "Foo")      // Delete
size := len(set)        // Size
_, exists := set["Foo"] // Membership
```

Now, we've got the hang of creating a set type in Golang. Congratulations!

### Third Party Package - `golang-set`

Actually, there's a well-established package called golang-set that has implemented the Set data structure in Golang using the same approach as mentioned earlier. You can access its repository at [golang-set](https://github.com/deckarep/golang-set). According to its description, Docker also utilizes this package.

This package offers two ways to use it: a thread-safe set and a non-thread-safe set.

A simple example:

```go
package main

import (
	"fmt"

	mapset "github.com/deckarep/golang-set"
)

func main() {
	// By default, a thread-safe set is created. If thread safety is not needed,
	// you can use NewThreadUnsafeSet, and the usage is the same.
	s1 := mapset.NewSet(1, 2, 3, 4)
	fmt.Println("s1 contains 3: ", s1.Contains(3))
	fmt.Println("s1 contains 5: ", s1.Contains(5))

	// Accepts interface parameters, allowing you to pass values of any type
	s1.Add("poloxue")
	fmt.Println("s1 contains poloxue: ", s1.Contains("poloxue"))
	s1.Remove(3)
	fmt.Println("s1 contains 3: ", s1.Contains(3))

	s2 := mapset.NewSet(1, 3, 4, 5)

	// Union
	fmt.Println(s1.Union(s2))
}
```

Output:

```bash
s1 contains 3:  true
s1 contains 5:  false
s1 contains poloxue:  true
s1 contains 3:  false
Set{4, polxue, 1, 2, 3, 5}
```

Here, I present an easy way to employ this package. If you find yourself confused, reading the source code of this package might help. You'll find familiar methods for working sets, like `Intersect` and `Difference`.

## Creating Set using BitSet

Moving on, we'll explore the implementation of Set using Bitset, with each element occupying only one bit. Consequently, an `int8` can represent 8 elements, significantly saveing memory.

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-01/2024-01-11-set-in-golang-02.png)

Bitset is frequently used for BitMap and Flag applications. 

To illustrate, consider implementing a Flag type with Bitset. Suppose we require three separate flags for Permission A, Permission B, and Permission C, all of which may be concurrently active. These are defined as F1, F2, and F3.

The code is as follows:

```go
``// Define Bits as an alias for uint8, a compact integer type.
type Bits uint8

// Define constants using iota for bit shifting, creating bit flags.
const (
    F0 Bits = 1 << iota  // F0 is 1 (00000001 in binary).
    F1                   // F1 is 2 (00000010 in binary), automatically shifted.
    F2                   // F2 is 4 (00000100 in binary), automatically shifted.
)

// Set function sets (turns on) the specified flag in the bitset.
func Set(b, flag Bits) Bits { return b | flag }

// Clear function clears (turns off) the specified flag in the bitset.
func Clear(b, flag Bits) Bits { return b &^ flag }

// Toggle function toggles the specified flag in the bitset.
func Toggle(b, flag Bits) Bits { return b ^ flag }

// Has function checks if the specified flag is set in the bitset.
func Has(b, flag Bits) bool { return b&flag != 0 }

func main() {
    var b Bits  // Declare a variable b of type Bits.

    // Set and toggle some flags in b.
    b = Set(b, F0)      // Set the F0 flag.
    b = Toggle(b, F2)   // Toggle the F2 flag.

    // Iterate over the flags and check if they are set in b.
    for i, flag := range []Bits{F0, F1, F2} {
        fmt.Println(i, Has(b, flag))  // Print index and flag status (true/false).
    }
}
```

Here, we use a uint8 type for defining a Flag type. Operations on the BitSet, including Set, Clear, Toggle, and Has, can be effectively carried out through bit operations. Given its natural suitability, the bitset is particularly adept at processing Set data structures, allowing all operations to be implemented via bit operations. Some examples follow:

- Intersect：a & b
- Union：a | b
- Difference：a & (~b)

In the earlier example, we saw how to manage a dataset limited to what a single int8 can represent, which is no more than 8 elements. For larger datasets, the solution involves using a combination of BitSet and Slice. The structure of this data type would be:

```go
type Bitset struct {
  data []int64
}
```

How to figure out which bit corresponds to the desired element in this data structure?

That involves using two indices: the slice index identifies the segment, while the bit index reveals which bit in an int64 is associated with the element.

You can obtain the slice index using division. For example, dividing 65 by 64 gives the slice index of 65. For operations requiring higher performance, bit shifting, such as 65 >> 6, can be employed, where 6 is the number of shifts, aligning with n in 2^n.

To determine the bit index, we use the modulus of the division. For example, 65 % 64 yields 1. In high-performance scenarios, this can be efficiently done through bit manipulation, as shown by 65 & 0b00111111 or 65 & 63.

A simple example would be:

```go
package main

import (
	"fmt"
)

// Define constants for shifting and masking.
const (
	shift = 6            // Number of bits to shift.
	mask  = 0x3f         // 0b00111111 in binary, used as a mask.
)

// Bitset is a structure for managing a set of bits.
type Bitset struct {
	data []int64  // An array of int64 to hold the bits.
}

// NewBitSet creates a new Bitset with a given bit set.
func NewBitSet(n int) *Bitset {
	// Calculate the index in the array where the bit should be set.
	index := n >> shift  // Right shift n by 'shift' bits.

	// Create a new Bitset with a slice large enough to include the index.
	set := &Bitset{
		data: make([]int64, index+1),
	}

	// Set the specific bit in the data slice.
	set.data[index] |= 1 << uint(n&mask)  // Use bitwise OR to set the bit.

	return set
}

// Contains checks if a specific bit is set in the Bitset.
func (set *Bitset) Contains(n int) bool {
	// Calculate the index and check if the bit is set.
	index := n >> shift  // Determine the array index.
	return set.data[index]&(1<<uint(n&mask)) != 0  // Use bitwise AND to check the bit.
}

func main() {
	// Create a new BitSet and check if specific bits are set.
	set := NewBitSet(65)  // Initialize Bitset with bit 65 set.
	fmt.Println("set contains 65", set.Contains(65))  // Expect true.
	fmt.Println("set contains 64", set.Contains(64))  // Expect false.
}
```

Output:

```bash
set contains 65 true
set contains 64 false
```

This example is solely for demonstration purposes. It features a BitSet with the Contains method, while other functionalities like Add, Intersect, Union, and Difference are omitted. If these interest you, feel free to experiment with them. 

As with Set implemented using Map, the BitSet package is also available on GitHub at [yourbasic/bit](https://github.com/yourbasic/bit). 

Let’s proceed to write an example demonstrating its use:

```go
``package main

import (
	"fmt"

	"github.com/yourbasic/bit"  // Import the yourbasic/bit package.
)

func main() {
	// Create a new set with specified elements.
	s := bit.New(2, 3, 4, 65, 128)
	fmt.Println("s contains 65", s.Contains(65))  // Check if 65 is in the set.
	fmt.Println("s contains 15", s.Contains(15))  // Check if 15 is in the set.

	s.Add(15)  // Add 15 to the set.
	fmt.Println("s contains 15", s.Contains(15))  // Now 15 should be in the set.

	// Find the smallest element in the set greater than 20.
	fmt.Println("next 20 is ", s.Next(20))
	// Find the largest element in the set less than 20.
	fmt.Println("prev 20 is ", s.Prev(20))

	// Create another set.
	s2 := bit.New(10, 22, 30)

	// Create a new set that is the union of s and s2.
	s3 := s.Or(s2)
	fmt.Println("next 20 is ", s3.Next(20))  // Find the smallest element greater than 20 in the union set.

	// Visit each element in the set and print it. 
	// The Visit function stops if the callback returns true.
	s3.Visit(func(n int) bool {
		fmt.Println(n)
		return false  // Return false to continue visiting each element.
	})
}
```

Output:

```bash
s contains 65 true
s contains 15 false
s contains 15 true
next 20 is 65
prev 20 is 15
next 20 is 22
2
3
4
10
15
22
30
65
128
```

This example is easy to comprehend, featuring basic operations like Add, Contains, and Visit. However, one limitation of BitSet is that it only supports integer elements. If performance is less of a priority, using a map set might be a better choice.

## Summary

This post has explored the principles behind two types of set implementations in Go, along with a basic guide on how to use the corresponding packages. I believe that, with this post, the essentials of using Set in Go are largely covered.
