---
title: "How to Use Enums in Go for Both Enhanced Safety and Efficiency?"
date: 2025-02-01T22:35:15+08:00
draft: true
---


![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-02/2024-02-02-how-to-use-enums-type-in-golang-01.png)

Enums, or enumerations, are used for representing a set range of values, making our code clearer and safer.

Take game programming as an example: game characters like warriors, mages, or archers can be represented using enums.

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-02/2024-02-02-how-to-use-enums-type-in-golang-08.png)

In Go, however, enums are not as straightforward as in some other programming languages. To effectively use enums in Go, it's crucial to understand the various ways they can be represented and their nuances.

This article will explore how to use enums in Go, from the simplest methods to more complex ones.

## Using `iota` and Constants

In Go, the most common method for representing enums is by using `iota` and constants.

What exactly is `iota`?

`iota` is a special keyword in Go. It helps in automatically creating a series of related constants, following certain rules, without the need for manually assigning values to each. This naturally fits with the purpose of enums.

Still unclear?

Let's see an example. We'll create constants with specific rules using `iota`:

Example code:

```go
type Weekday int

const (
    Sunday    Weekday = iota // 0
    Monday                   // 1
    Tuesday                  // 2
    Wednesday                // 3
    Thursday                 // 4
    Friday                   // 5
    Saturday                 // 6
)
```

In this example, the `Weekday` type has seven values, each for a day of the week. The internal values start at 0, with `iota` automatically incrementing each constant from Sunday to Saturday, assigning values from 0-6.

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-02/2024-02-02-how-to-use-enums-type-in-golang-02.png)

Now, manual assignment for each constant is unnecessary.

`iota` has other clever uses, but they're beyond the scope of this article.

This approach is simple and offers some level of type safety but also has its limitations.

I see two main shortcomings.

First, unlike enums in some languages, it can't directly convert from a string to an enum type. For instance, it can't convert the string "Sunday" directly into the `Sunday` enum value.

Second, its type safety isn't absolute.

With the `Weekday` type, we can't assign a variable of a clearly different type to a `Weekday` variable:

```go
day := 0 // int
// compiler: cannot use day (variable of type int) 
// as Weekday value in variable declaration [IncompatibleAssign]
var sunday Weekday = day 
```

However, we can assign a literal that is not of the `Weekday` class:

```go
// Assigning literal value 10 to variable day of type Weekday
var day Weekday = 10 
```

Clearly, the number 10 is not a valid `Weekday` value, but it can be assigned without causing an error.

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-02/2024-02-02-how-to-use-enums-type-in-golang-03.png)

In languages with robust enum mechanisms, such code would not compile.

Apart from this basic method, let's explore other forms of enum representation.

## Enums with String Conversion Support

In web development, especially when interfacing with front-end applications, there's often a need to represent enum values as strings. Let's look at how to enable this, allowing conversion between string variables and enum variables.

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-02/2024-02-02-how-to-use-enums-type-in-golang-04.png)

This is simple in Go; we can use string constants as enum values.

Example code:

```go
type HttpMethod string

const (
    Get    HttpMethod = "GET"
    Post   HttpMethod = "POST"
    Put    HttpMethod = "PUT"
    Delete HttpMethod = "DELETE"
)
```

This method is straightforward and easily integrates with data formats like JSON.

```go
type Request struct {
    Method HttpMethod
    URL    string
}

func main() {
    r := Request{Method: Get, URL: "https://zhihu.com"}
    result, _ := json.Marshal(r)
    fmt.Printf("%s\n", result)
}
```

Output:

```bash
{"Method":"GET","URL":"https://zhihu.com"}
```

But what if we want to keep the original `iota` integer enum values, which are more lightweight and use less memory? Can we do that? Let's see.

Define as follows:

```go
type HttpMethod int

const (
    Get    HttpMethod = iota
    Post
    Put
    Delete
)
```

We just need to add methods for converting between integers and strings to the enum type.

Code example:

```go
// Convert from string to HttpMethod
func NewFromString(method string) HttpMethod {
  switch method {
  case "Get":
    // ...omitted
  case "Delete":
  default:
    return Get // default is Get or error, panic
  }
}

// Convert from HttpMethod to string
func (h HttpMethod) String() string {
  switch h {
  case Get:
    return "Get"
    // ...omitted
  default:
    return "Unknown" // or error, panic
  }
}
```

Here, to support friendly JSON serialization and deserialization, where enum values are represented as strings, we need to add methods to `HttpMethod` to implement the json.Marshaler and json.Unmarshaler interfaces.

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-02/2024-02-02-how-to-use-enums-type-in-golang-05.png)

Code example:

```go
// MarshalJSON implements the json.Marshaler interface
func (h HttpMethod) MarshalJSON() ([]byte, error) {
    return json.Marshal(h.String())
}

// UnmarshalJSON implements the json.Unmarshaler interface
func (h *HttpMethod) UnmarshalJSON(data []byte) error {
    var method string
    if err := json.Unmarshal(data, &method); err != nil {
        return err
    }
    *h = NewFromString(method)
    return nil
}
```

In open-source projects, you might find packages that implement such enums, where code for conversion between strings and enums can be generated based on `iota` definitions.

Rob Pike developed a tool named [`stringer`](https://pkg.go.dev/golang.org/x/tools/cmd/stringer), which can generate the `String()` method for types like `HttpMethod`, although it doesn't offer full enum support.

```go
//go:generate stringer -type=HttpMethod
type HttpMethod int

const (
    Get    HttpMethod = iota
    Post
    Put
    Delete
)
```

By executing `go generate`, the `String` method will be generated for the `HttpMethod` type.

```
go generate
```

Remember, you need to install the `stringer` command separately.

However, even now, the issue of type safety persists. Codes like `var Hello HttpMethod = 10` are still valid.

Let's keep going.

## Struct-Based Enum Values

Enums can also be implemented using struct types in Go.

For example, let's create a color enum with not just the name but also its RGB value. For convenience, we can add an enum integer value.

```go
type Color struct {
    Index int
    Name  string
    RGB   string
}
```

Now, we have a color enum where each color has an index, name, and RGB value.

How to use it?

Just like before, we define them directly:

```go
var (
    Red   = Color{0, "Red", "#FF0000"}
    Green = Color{1, "Green", "#00FF00"}
    Blue  = Color{2, "Blue", "#0000FF"}
)
```

This method is more flexible but also more complex.

The advantage is that it can store more information. Con

verting between the integer value `Color.Index` and the string `Color.Name` becomes easy. However, to maximize integration with other libraries, like custom JSON conversion, you'll need to implement interfaces for serialization and deserialization, and for string formatting, the `Stringer` interface.

Also, since this structure is not of a constant type, there's the potential for data to be mutable. If safety is a concern, you could make the fields private and change them through methods.

## Struct-Like Namespaces

Here's an interesting design I found online.

If you have many enum types and are concerned about naming conflicts, you can use a struct as a "namespace" to organize related enum values together:

Example code:

```go
var Colors = struct {
    Red, Green, Blue Color
}{
    Red   = Color{0, "Red", "#FF0000"}
    Green = Color{1, "Green", "#00FF00"}
    Blue  = Color{2, "Blue", "#0000FF"}
}
```

In this example, `Colors` is an anonymous struct type. We can access colors through `Colors.xxx`.

At first, I thought this approach limited the range of enum values that could be defined. But I was wrong; you can still define new values using the `Color` type.

This approach is a bit clumsy and redundant. A new `package` could do the job more elegantly. But since it's a solution I came across, I'm including it here.

## Type Safety?

So far, none of the implementations have solved the issue of adding new enum values after an enum is defined.

What if we really want to prevent creating new enum values in a way like `HttpMethod(1)`?

That's simple.

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-02/2024-02-02-how-to-use-enums-type-in-golang-07.png)

We can encapsulate the enum implementation in a package, make the type lowercase, like `httpMethod`, and expose functions like `FromString` to enforce creation through these functions.

```go
package httpmethod

type httpmethod string

const (
  Get  = "Get"
  Post = "Post"
)

func FromString(method string) httpmethod {
  switch method {
  case "Get":
    return Get
  case "Post":
    return Post
  }
}
```

Now, enum creation must go through methods, where we can implement creation rules.

This method seems good, but it's not commonly used.

Why?

My guess is that developers usually don't create new enum values on the fly. Ensuring that boundary data is handled through conversion functions should be enough, right?

## Real-World Scenarios

In real-world applications, the usefulness of enums mainly lies in interfacing with other systems.

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-02/2024-02-02-how-to-use-enums-type-in-golang-06.png)

For instance, when receiving data from front-end APIs or databases, we might encounter outlier values. In such cases, we can provide conversion functions with checking rules. If we detect anomalies, we can choose to discard them, throw an error, or panic.

For internal systems using protocols like `protobuf`, we can define the enum range in the protocol and mark abnormal data.

Of course, mismatches in enum values between systems can occur due to release timing or communication oversights. These cases will also be handled as mentioned above, facilitating immediate detection.

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-02/2024-02-02-how-to-use-enums-type-in-golang-09.png)

In team collaboration scenarios, the best solution is to consider compatibility in system design, rather than dealing with chaos every time there's a change. This approach is most likely to prevent production incidents.

In any case, the focus should be on ensuring that data entering the system can pass through conversion checks, rather than unnecessarily restricting conversions like `HttpMethod("Get")`, because nobody codes that way.

## Conclusion

In Go, there are various ways to express enums, from simple `iota` to more complex struct-based methods. Each method has its appropriate use cases. As developers, it's best to choose the method that suits your specific needs.

Hopefully, this article helps you use enums more flexibly and effectively in Go.

