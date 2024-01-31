---
title: "2024 01 25 Optional Parameters in Golang"
date: 2025-02-27T21:00:00+08:00
draft: false
---

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-01/2024-01-25-optional-parameters-in-golang-01.png)

In programming, a common scenario involves a function that generally requires only a few parameters, yet occasionally, it needs a set of variable optional parameters. In some languages, this challenge is addressed through function overloading or optional parameters. However, Go operates differently, as it neither supports function overloading nor has a built-in feature for optional parameters. So, how can one achieve this functionality in Go?

This article delves into this subject, progressively exploring various approaches to implement optional parameters in Go.

## Method 1: Variadic Args

While Go lacks support for optional parameters, it compensates by allowing variadic arguments, enabling functions to accept an indefinite number of parameters. This functionality is implemented by prefixing the parameter type with ....

Consider the following example:

```go
func printNumbers(numbers ...int) {
    for _, number := range numbers {
        fmt.Println(number)
    }
}

func main() {
    printNumbers(1, 2)
    printNumbers(1, 2, 3, 4)
}
```

In this example, we define a printNumbers function capable of accepting an unlimited number of integer arguments.

This approach is primarily suited to situations where all parameters are of the same type.


![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-01/2024-01-25-optional-parameters-in-golang-02.png)

But what about when parameters vary in type?

One solution involves using ...interface{} to extend the variadic arguments approach. However, this inevitably increases the overhead of reflection, type decision, or inference. Moreover, the parameters are determined by index, which elevates code complexity and significantly hampers readability.

Is there a better alternative?

## Method 2: Using Map


To handle an uncertain number of parameters of varying types, employing a map can be an effective strategy.


```go
func setConfig(configs map[string]interface{}) {
    if val, ok := configs["timeout"]; ok {
        fmt.Println("Timeout:", val)
    }
    if val, ok := configs["path"]; ok {
        fmt.Println("Path:", val)
    }
}

func main() {
    setConfig(map[string]interface{}{
        "timeout": 30,
        "path":    "/usr/bin",
    })
}
```

In this example, the setConfig function accepts a map as a parameter. The keys represent the names of configuration items, while the values are the respective configuration settings.

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-01/2024-01-25-optional-parameters-in-golang-03.png)



The primary drawback of this method is the loss of type safety. It also necessitates runtime type assertions for interface{} type parameters. Nevertheless, this approach offers clearer type distinction compared to variadic arguments.

Is there a way to preserve type safety?

## Method 3: Using Structs

If the goal is to maintain type safety while enjoying the flexibility of optional parameters, structs present a viable solution. But is it too cumbersome to create a new struct instance with each function call?


```go
type Config struct {
    Timeout int
    Path    string
}

func setConfig(config Config) {
    fmt.Println("Timeout:", config.Timeout)
    fmt.Println("Path:", config.Path)
}

func main() {
    setConfig(Config{
        Timeout: 30,
        Path:    "/usr/bin",
    })
}
```

This method offers the advantage of type safety and clearly indicates which parameters have been set.

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-01/2024-01-25-optional-parameters-in-golang-04.png)

However, it does require creating a new struct instance for each function call.

## Method 4: Functional Options Pattern

Is there an approach that maintains type safety and offers flexible optional parameters? The functional options pattern seems to provide a promising solution.

```go
type Config struct {
    Timeout int
    Path    string
}

type Option func(*Config)

func WithTimeout(timeout int) Option {
    return func(c *Config) {
        c.Timeout = timeout
    }
}

func WithPath(path string) Option {
    return func(c *Config) {
        c.Path = path
    }
}

func NewConfig(opts ...Option) *Config {
    config := &Config{}
    for _, opt := range opts {
        opt(config)
    }
    return config
}

func main() {
    config := NewConfig(
        WithTimeout(30),
        WithPath("/usr/bin"),
    )
    fmt.Println(config)
}
```

In this example, we introduce a Config struct and an Option type, where Option is a function that takes a *Config as a parameter.

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-01/2024-01-25-optional-parameters-in-golang-05-v2.png)

We also define WithTimeout and WithPath functions, which return an Option. This allows us to modify the Config structure's fields by passing various options when calling the NewConfig function, thus creating diverse configurations.

This method's strength lies in its flexibility and the ability to expand the API without disrupting existing code. However, its complexity might require some time to fully grasp.

## Conclusion

This blog post presented several strategies for implementing optional parameters in Go: variadic arguments, using maps, structs, and the functional options pattern. Each method has its specific application scenarios, pros, and cons. You can select the most suitable approach based on your requirements.

Blog address: [Implementing Optional Parameters in Go: Overloading? Variadic Args?](https://en.poloxue.com/posts/2024-01-25-optional-parameters-in-golang/)
