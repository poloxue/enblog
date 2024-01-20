---
title: "Exploring High-Efficiency int to string Conversion Strategies and Source Code in Go"
date: 2024-01-20T17:47:23+08:00
draft: false
---

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-01/2024-01-20-int-to-string-in-golang-04.png)

In Go, converting an integer (`int`) to a string (`string`) is a common operation. 

This post will introduce several common methods to convert `int` to `string` in Go, focusing on the performance characteristics of these methods. It will also delve into the efficient algorithm implementation of `FormatInt`.


![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-01/2024-01-20-int-to-string-in-golang-01.png)

### Using `strconv.Itoa`

The most direct and commonly used method is the `Itoa` function from the `strconv` package. `Itoa` is an abbreviation for "Integer to ASCII". It provides a quick and concise way to convert integers to strings.

Example code:

```go
package main

import (
    "strconv"
    "fmt"
)

func main() {
    i := 123
    s := strconv.Itoa(i)
    fmt.Println(s)
}
```

`strconv.Itoa` converts the integer directly to its ASCII string representation, minimizing extra memory allocation and avoiding complex logic.

### Using `fmt.Sprintf`

Another method is to use the `Sprintf` function from the `fmt` package. This method is more powerful and flexible as it can handle various types and output them in a specified format.

Example code:

```go
package main

import (
    "fmt"
)

func main() {
    i := 123
    s := fmt.Sprintf("%d", i)
    fmt.Println(s)
}
```

Although `fmt.Sprintf` is very powerful, its performance is usually not as good as `strconv.Itoa`. This is because `fmt.Sprintf` internally uses reflection to determine the input value type, and involves more string concatenation and memory allocation.

### Using `strconv.FormatInt`

When you need more control or to deal with integer types other than `int` (such as `int64`), you can use the `FormatInt` function from the `strconv` package.

```go
package main

import (
    "strconv"
    "fmt"
)

func main() {
    var i int64 = 123
    s := strconv.FormatInt(i, 10)  // 10 for base 10
    fmt.Println(s)
}
```

`strconv.FormatInt` offers more fine-grained control over the conversion process, including the choice of base (e.g., decimal, hexadecimal). Like `strconv.Itoa`, `FormatInt` is also very performance efficient, providing a flexible and efficient solution.

If we look at the source code of `strconv.Itoa`, we find that `strconv.Itoa` is actually a special case of `strconv.FormatInt`.

```go
// Itoa is shorthand for FormatInt(int64(i), 10).
func Itoa(i int) string {
    return FormatInt(int64(i), 10)
}
```

Now the focus of analyzing high-performance code for int to string conversion is on dissecting `FormatInt`.

### Deep Dive into FormatInt

Based on the `itoa.go` source code from Go version 1.21, we can understand the efficient implementation of the integer-to-string conversion functions in the `strconv` package.


![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-01/2024-01-20-int-to-string-in-golang-02-en.png)

#### 1. Fast Path for Small Integers

For common small integers, the `strconv` package provides a fast path, directly returning precomputed strings to avoid runtime computation overhead.

```go
func FormatInt(i int64, base int) string {
	if fastSmalls && 0 <= i && i < nSmalls && base == 10 {
		return small(int(i))
	}
}

func small(i int) string {
	if i < 10 {
		return digits[i : i+1]
	}
	return smallsString[i*2 : i*2+2]
}
```

`fastSmalls` enables fast processing for small integers. `nSmalls` defines the upper limit of small integers (specifically 100), and `base == 10` ensures that only decimal integers use this fast path.

The `small` function retrieves the string representation of small integers by indexing into `smallsString`, which is very fast.

Values of `digits` and `smallsString` are as follows:

```go
const smallsString = "00010203040506070809" +
	"10111213141516171819" +
	"20212223242526272829" +
	"30313233343536373839" +
	"40414243444546474849" +
	"50515253545556575859" +
	"60616263646566676869" +
	"70717273747576777879" +
	"80818283848586878889" +
	"90919293949596979899"

const digits = "0123456789abcdefghijklmnopqrstuvwxyz"
```

These are mappings of decimal 0-99 to their corresponding strings.

#### 2. Efficient Implementation of the formatBits Function

The `formatBits` function is the core of the integer-to-string conversion. It is optimized for different bases.

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-01/2024-01-20-int-to-string-in-golang-03-en.png)

##### Decimal Conversion Optimization

For decimal conversion, `formatBits` uses a division and remainder-based algorithm, and accelerates the retrieval of two-digit strings with `smallsString`.

```go
if base == 10 {
	// ... (optimizations for 32-bit systems)
	us := uint(u)
	for us >= 100 {
		is := us % 100 * 2
		us /= 100
		i -= 2
		a[i+1] = smallsString[is+1]
		a[i+0] = smallsString[is+0]
	}
	// ... (handling the remaining digits)
}
```

- For 32-bit systems, it uses 32-bit operations to handle larger numbers, reducing the overhead of 64-bit division.
- It processes two digits at a time, directly obtaining corresponding characters from `smallsString`, avoiding the overhead of converting each digit individually.

##### Optimization for Bases that are Powers of Two

For bases that are powers of two, `formatBits` uses bitwise operations to optimize the conversion.

```go
} else if isPowerOfTwo(base) {
	shift := uint(bits.TrailingZeros(uint(base))) & 7
	b := uint64(base)
	m := uint(base) - 1 // == 1<<shift - 1
	for u >= b {
		i--
		a[i] = digits[uint(u)&m]
		u >>= shift
	}
	// u < base
	i--
	a[i] = digits[uint(u)]
}
```

- Bitwise operations are performed directly on the binary level, which is faster than division and remainder operations.
- Utilizing the characteristics of bases that are powers of two, it gets the digits of the number through shifting and masking operations.

#### Handling General Cases

For other bases, `formatBits` uses a general algorithm but still tries to minimize the use of division and remainder operations.

```go
} else {
	// general case
	b := uint64(base)
	for u >= b {
		i--
		// Avoid using r = a%b in addition to q = a/b
		// since 64bit division and modulo operations
		// are calculated by runtime functions on 32bit machines.
		q := u / b
		a[i] = digits[uint(u-q*b)]
		u = q
}
```

### Conclusion

Converting `int` to `string` is a very common requirement. The int to string conversion functions in Go's `strconv` package demonstrate a deep understanding and focus on performance by the Go

 standard library.

With fast processing of small integers, optimized decimal conversion algorithms, and special handling of bases that are powers of two, these functions offer efficient and stable performance. These optimizations ensure that even in scenarios with large amounts of data or where performance is critical, the functions in the `strconv` package deliver excellent performance.

Blog post: [Exploring High-Efficiency int to string Conversion Strategies and Source Code in Go](https://en.poloxue.com/posts/2024-01-20-int-to-string-in-golang/)

