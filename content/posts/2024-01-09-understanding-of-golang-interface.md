---
title: "2024 01 09 Understanding of Golang Interface"
date: 2025-01-09T15:08:13+08:00
draft: true
comment: true
description: "This post covers Golang Interface. Let's dive into it."
---

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-01/2024-01-09-understanding-of-golang-interface-01.jpeg)

This post covers Golang Interface. Let's dive into it. 

## Duck Typing

To understand Go's interfaces, it's crucial to grasp the Duck Typing concept.

So, what's Duck Typing?

The explanation usually revolves around a clever example: determining whether something is a duck based on its abilities. If it swims like a duck and quacks like a duck, then it can be considered a duck.

Dynamic languages like Python and JavaScript naturally support this feature, but compared to static languages, dynamic languages lack essential type checking.

Go's interface design is closely related to the Duck Typing concept but differs from dynamic languages. In Go, type checks occur at compile time.

## Understanding Go Interfaces

Go interfaces are like a group of methods that act as abstract types. They create a sort of invisible contract, allowing any type that follows the methods outlined in the interface to be recognized as part of that type.

For instance, suppose we define a Duck interface like this:

```go
type Duck interface {
	Quack()   // Duck quacks
	DuckGo()  // Duck moves
}
```

Now, let's imagine a Chicken type:

```go
type Chicken struct {
}

func (c Chicken) IsChicken() bool {
	fmt.Println("I am a chicken")
}
```

This chicken is special and can perform duck-like actions:

```go
func (c Chicken) Quack() {
	fmt.Println("Quack")
}

func (c Chicken) DuckGo() {
	fmt.Println("Strutting along")
}
```

Notice that we've only implemented the Duck interface methods without explicitly binding the chicken type to the Duck interface.

Let's create a function that performs actions a duck can do:

```go
func DoDuck(d Duck) {
	d.Quack()
	d.DuckGo()
}
```

Since the chicken implements all the methods of a duck, it's also considered a duck. Therefore, in the main function, we can do this:

```go
func main() {
	c := Chicken{}
	DoDuck(c)
}
```

It runs smoothly. Doesn't it resemble polymorphism in other languages? This is how Go implements polymorphism.

## Empty Interface

Now, let's talk about the empty interface.

An interface without any defined methods, represented as interface{}, means any type can meet it. That's why when a function parameter type is interface{}, it can accept parameters of any type.

For instance:

```go
package main

import "fmt"

func main() {
	var i interface{} = 2
	fmt.Println(i)
}
```

In more common scenarios, interface{} in Go is often used as a function parameter to mimic the generic effects found in other languages. Although for a long time, Go did not support generics, this approach serves as a workaround.

## Conclusion

To sum up, understanding Go interfaces revolves around the idea that interfaces are a collection of methods. This understanding is pivotal. Once grasped, comprehending other Go concepts like types, polymorphism, empty interfaces, reflection, type checking, and assertions becomes much easier.



