---
title: "Mastering the utilization of sets in Golang"
date: 2024-01-11T17:06:37+08:00
draft: true
comment: true
---

When coding in Go (or Golang), having a solid grasp of data structures is essential for problem-solving. This post will delve into the use of a data structure in Go: set.

## Commonly Data Structures in Golang.

Golang offers a relatively small set of builtin data structures. 

In our daily work, we frequently rely on slice and map. In fact, Go also has an array type, but due to the existence of slice, we rarely use the array type.

Besides the built-in data structures in Golang, the Go container package also provides additional data structures, such as heap, list, and ring. For experienced programmers, understanding how to use them is quite straightforward.

Today, I'd like to explain `set`. 

While some programming languages like Java have these two types of data structures, it's unfortunate that Golang doesn't provide support for them in any form.

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

Actually, there's a well-established package called golang-set that has implemented the set data structure in Golang using the same approach as mentioned earlier. You can access its repository at golang-set. According to its description, Docker also utilizes this package.

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



