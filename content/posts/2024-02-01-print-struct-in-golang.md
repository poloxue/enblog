---
title: "2024 02 01 Print Struct in Golang"
date: 2025-01-31T22:48:02+08:00
draft: false
---

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-02/2024-02-01-print-struct-in-golang-01.png)

Have you ever needed to print structures?

Structures are somewhat like boxes. Unlike slices and maps, they can hold various types such as numbers, strings, slices, maps, or even other structures.

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-02/2024-02-01-print-struct-in-golang-02.png)

But how do we view what's inside these boxes? That's where the ability to print structures becomes essential.

Printing structures is crucial as it helps us inspect and understand code, boosts debugging efficiency, and ensures correct code execution.

In this article, we'll explore how to print structures in Go, including deeper structures like pointers, slices, and maps.

Let's dive in!

## Defining Structures

First, let's define a structure, which will be used in all the methods discussed.

The code is as follows:

```go
type Author struct {
	Name string
	Age  int8
	Sex  string
}

type Article struct {
	ID      int64
	Title   string
	Author  *Author
	Content string
}
```

We've created an `Article` structure to represent an article. It contains basic properties like `ID`, `Title`, and `Content`, and a `Author` structure pointer to store author information.


I'll introduce four basic ways to print structures.

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-02/2024-02-01-print-struct-in-golang-03.png)

## Using `fmt.Printf`

The simplest method is `fmt.Printf`. For detailed display, use it with the `%+v` format specifier. This prints the structure's field names and values.

For example:

```go
func main() {
	article := Article{
		ID:      1,
		Title:   "How to Print a Structure in Golang",
		Author:  &Author{"poloxue", 18, "male"},
		Content: "This is a blog post",
	}
	fmt.Printf("%+v\n", article)
}
```

Output:

```bash
{ID:1 Title:How to Print a Structure in Golang Author:0xc0000900c0 Content:This is a blog post}
```

This code prints all field values of the `article` structure. However, the `Author` field shows only the pointer address - `Author:0xc0000900c0`, not its content.

This is as expected since `*Author` is a pointer type, and its value is the address.

To print the content of the `Author` field, use `fmt.Printf` to print what the pointer refers to.

```go
fmt.Print("%+v\n", article.Author)
```

Output:

```bash
&{Name:poloxue Age:18 Sex:male}
```

In my testing, I found an interesting behavior:

If you print a structure pointer, it automatically dereferences, thus printing its contents. Both

```go
Printf("%+v\n", article.Author)
```

and

```go
Printf("%\n", *article.Author)
```

will print the structure's contents.

However, if you print a structure containing a structure pointer field, it prints the address, not the content.

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-02/2024-02-01-print-struct-in-golang-04.png)

I suspect this design prevents deep recursion or circular references.

## Implementing the `String` Method

Besides printing the `Author` field separately, another method is implementing the `String` method.

This mechanism in Go allows a type to satisfy the `Stringer` interface by implementing the `String` method, altering the print output.

`Stringer` is defined as:

```go
type Stringer interface {
	String() string
}
```

When using `fmt.Printf` to print a structure, it calls the defined `String` method, allowing you to control the structure's output format. For instance:

```go
func (a Article) String() string {
	return fmt.Sprintf("Title: %s, Author: %s, Content: %s", a.Title, a.Author.Name, a.Content)
}

func main() {
	article := Article{
		ID:      1,
		Title:   "How to Print a Structure in Golang",
		Author:  &Author{"poloxue", 18, "male"},
		Content: "This is a blog post",
	}
	fmt.Println(article)
}
```

Output:

```bash
Title: How to Print a Structure in Golang, Author: poloxue, Content: This is a blog post
```

The output is formatted as defined in the `String` method. This method is also performance-efficient, directly delivering results.

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-02/2024-02-01-print-struct-in-golang-05.png)

At this point, we can print structures effectively. However, there are downsides.

First, it's not very visually appealing, and readability is poor. This hinders quick issue identification.

Secondly, output customization is required each time. If it's just for debugging, a more convenient direct print method is preferable.

## `json.MarshalIndent`

For a more visually appealing output, `json.MarshalIndent` is useful. This function converts structures to JSON format with controlled indentation, enhancing readability.

Here's an example:

```go
import (
    "encoding/json"
    "fmt"
)

func main() {
	article := Article{
		ID:      1,
		Title:   "How to Print a Structure in Golang",
		Author:  &Author{"poloxue", 18, "male"},
		Content: "This is a blog post",
	}
	articleJSON, _ := json.MarshalIndent(article, "", "    ")
	fmt.Println(string(articleJSON))
}
```

The third parameter in `json.MarshalIndent` sets the indent size. Let's check the output.

Output:

```json
{
    "ID": 1,
    "Title": "How to Print a Structure in Golang",
    "Author": {
        "Name": "poloxue",
        "Age": 18,
        "Sex": "male"
    },
    "Content": "This is a blog post"
}
```

The `Article` structure printed in this attractive JSON format is much more readable.

## Using the `reflect` Package for Complex Structures

For complete control over structure printing, the `reflect` package is handy. It provides detailed information about Go variables, including structure field names and types.

Here's an example:

```go
import (
    "fmt"
    "reflect"
)

func main() {
	article := Article{
		ID:      1,
		Title:   "How to Print a Structure in Golang",
		Author:  &Author{"poloxue", 18, "male"},
		Content: "This is a blog post",
	}

	val := reflect.ValueOf(article)
	for i := 0; i < val.NumField(); i++ {
		field := val.Type().Field(i)
		fmt.Printf(
			"Type: %v, Field: %s, Value: %v\n",
			field.Type,
			field.Name,
			val.Field(i),
		)
	}
}
```

Output:

```bash
Type: int64, Field: ID, Value: 1
Type: string, Field: Title, Value: How to Print a Structure in Golang
Type: *main.Author, Field: Author, Value: &{poloxue 18 male}
Type: string, Field: Content, Value: This is a blog post
```

We've outputted each field's type, name, and value.

`reflect` provides great flexibility, but you must define the specific print format. `Printf` internally also uses `reflect`.

For in-depth information printing, even pointer type fields, `reflect` allows further detailed printing.

The core idea is using `(*Article)(nil)` to get a `nil` of type `*Article`, like a placeholder in the type's memory space.

## Performance Testing

After trying various printing methods, I conducted a simple performance test.

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-02/2024-02-01-print-struct-in-golang-06.png)

Test results are as follows:

```bash
BenchmarkFmtPrintf-16              2631248	    447.3 ns/op
BenchmarkJSONMarshalIndent-16       997448	     1016 ns/op
BenchmarkCustomStringMethod-16     5135541	    225.5 ns/op
BenchmarkReflection-16             2030233	    594.9 ns/op
```

The results show that the custom `String` method is fastest, while `json.MarshalIndent` is slowest.

If performance matters, it's best to use a custom `String` method for printing structures.

A note: to fully test the custom `String` method's effect, avoid using `fmt.Sprintf` for internal formatting. Instead, use functions from `strconv`.

Example code:

```go
func (a Article) String() string {
	return "{ID:" + strconv.Itoa(int(a.ID)) + ", Title:" + a.Title + ",AuthorName:" + a.Author.Name + "}"
}
```

This truly tests the custom `String` method's advantage. Avoid nesting to prevent ineffective results.

For why `strconv` is faster, see my previous article: [Efficient int to string conversion in Go](https://www.poloxue.com/posts/2024-01-20-int-to-string-in-golang/)

## Third-Party Libraries

Besides the four methods, third-party libraries provide convenient solutions. They turn common patterns into easy-to-use libraries.

### `go-spew`

First, let's look at `go-spew`.

It deeply prints Go data structures, useful for debugging. It recursively prints structure fields, including nested ones.

For example:

```go
import "github.com/davecgh/go-spew/spew"

func main() {
	article := Article{
		ID:      1,
		Title:   "How to Print a Structure in Golang",
		Author:  &Author{"poloxue", 18, "male"},
		Content: "This is a blog post",
	}
	spew.Dump(article)
}
```

It prints all the contents of the `article` structure.

Output:

```bash
(main.Article) {
 ID: (int64) 1,
 Title: (string) (len=34) "How to Print a Structure in Golang",
 Author: (*main.Author)(0xc000100330)({
  Name: (string) (len=7) "poloxue",
  Age: (int8) 18,
  Sex: (string) (len=4) "male"
 }),
 Content: (string) (len=19) "This is a blog post"
```

The output is detailed.

For custom formats, configure `spew` using `ConfigState`, like indentation and depth.

Example code:

```go
// Spew configuration
spewConfig := spew.ConfigState{
	Indent:                  "\t", // Indent with Tab
	DisableMethods:          true,
	DisablePointerMethods:   true,
	DisablePointerAddresses: true,
	MaxDepth:                1, // Print depth 1
}

spewConfig.Dump(article)
```

Output:

```bash
(main.Article) {
	ID: (int64) 1,
	Title: (string) (len=34) "How to Print a Structure in Golang",
	Author: (*main.Author)({
		<max depth reached>
	}),
	Content: (string) (len=19) "This is a blog post"
}
```

The `Author` field's contents aren't printed due to the depth setting.

### pretty

Another library, `pretty`, is less powerful but useful for debugging complex structures.

```go

import (
    "fmt"
    "github.com/kr/pretty"
)

func main() {
  // Omitted...
  fmt.Printf("%# v\n", pretty.Formatter(article))
}
```

Output:

```bash
main.Article{
    ID:      1,
    Title:   "How to Print a Structure in Golang",
    Author:  &main.Author{Name:"poloxue", Age:18, Sex:"male"},
    Content: "This is a blog post",
}
```

The output is a formatted structure.

## Conclusion

This article covers various methods to print structures in Go. From simple `fmt.Printf` to reflection and third-party libraries, there's a rich choice.

Even simple topics, when explored deeply, offer abundant content.

One aspect not covered is logging complex structures in logs, crucial for troubleshooting. We might discuss this separately later.

I hope this article helps you in printing and debugging Go structures and other complex structures, making the process clearer.

Thank you for reading.

Blog post: [Printing Structures in Go? Improve Your Code Debugging Efficiency](https://end.poloxue.com/2024-02-01-print-struct-in-golang)
