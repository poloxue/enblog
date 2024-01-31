---
title: "From Fatal Errors to sync.Map: Concurrency Strategies for Maps in Go."
date: 2025-01-21T17:15:14+08:00
draft: false
comment: true
---

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-01/2024-01-21-fatal-error-in-concurrent-accessing-map-02.png)

Why does Go throw a fatal error, instead of a panic, when multiple goroutines concurrently access and modify the same map? This article will guide you through the underlying principles and introduce solutions to handle map concurrency issues in Go.

### Map Data Race

Firstly, let's define a Map data race. It occurs when two or more goroutines access the same piece of data simultaneously without proper synchronization, and at least one of the goroutines modifies the data. This situation can lead to unpredictable behavior or even crashes.

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-01/2024-01-21-fatal-error-in-concurrent-accessing-map-01.png)

Although maps are a common data structure in Go, providing fast Key/Value storage, Go's default map is not safe for concurrent use. Unsynchronized concurrent access to a map can lead to data races.

### Fatal Error from Map Data Race

In Go, errors are typically handled by returning an Error or a panic. However, the detection of a data race in a map triggers a fatal error, leading to an immediate crash. Go opts for a stricter approach here.

A simple example to demonstrate how a fatal error can be triggered:

```go
package main

import (
    "sync"
)

func main() {
    m := make(map[int]int)
    wg := sync.WaitGroup{}
    for i := 0; i < 10; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            for j := 0; j < 1000; j++ {
                m[j] = j
            }
        }()
    }
    wg.Wait()
}
```

This code snippet may lead to a fatal error like this:

```bash
fatal error: concurrent map writes

goroutine 6 [running]:

... omitted

exit status 2
```

### Why Fatal Error Instead of Panic?

Fatal errors do not allow for runtime recovery. The rationale behind this might be:

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-01/2024-01-21-fatal-error-in-concurrent-accessing-map-03-en.png)

**Immediate Issue Exposure:**

This approach ensures that as soon as a data race occurs, the program stops immediately, forcing an immediate confrontation with the issue. It helps in quickly identifying and solving concurrency bugs and encourages writing concurrency-safe code.

**Data Integrity Protection:**

Data races can have severe consequences, especially in complex concurrent systems. Uncoordinated access and modification by multiple goroutines can lead to inconsistent or corrupted internal states of a map. This uncertainty can result in erratic program behavior and hard-to-track bugs.

### Diving into the Source Code: Map Concurrency Detection

When Go detects concurrent writes to a map, it throws a fatal error through the `throw` function. This happens in the `mapassign` function. Here's a simplified version of `mapassign`:

```go
func mapassign(t *maptype, h *hmap, key unsafe.Pointer) unsafe.Pointer {
    // Check for other goroutines writing to the map
    if h.flags&hashWriting != 0 {
        throw("concurrent map writes")
    }
    
    // ...other map assignment logic...
    
    // Set flag indicating a goroutine is writing to the map
    h.flags |= hashWriting
    
    // ...execute map assignment logic...
    
    // Clear the writing flag after the write is done
    h.flags &^= hashWriting
    
    return val
}
```

The critical part is the condition `h.flags&hashWriting`, which triggers the fatal error.

### How to Avoid Data Races

In Go, the most common concurrency control mechanisms are channels or tools from the sync package. Additionally, Go provides a concurrent-safe map type - sync.Map.

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-01/2024-01-21-fatal-error-in-concurrent-accessing-map-04.png)

#### sync.Mutex

Here's how to use sync.Mutex to prevent data races:

```go
package main

import (
    "sync"
)

func main() {
    m := make(map[int]int)
    var mu sync.Mutex
    wg := sync.WaitGroup{}
    for i := 0; i < 10; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            for j := 0; j < 1000; j++ {
                mu.Lock()
                m[j] = j
                mu.Unlock()
            }
        }()
    }
    wg.Wait()
}
```

In this example, `sync.Mutex` ensures that only one goroutine can write to the map at a time, thus preventing data races.

#### sync.Map

Introduced in Go 1.9, sync.Map is designed for concurrent scenarios. It has special features like lock-free reads and fine-grained locking mechanisms, simplifying concurrent programming.

Here's an example using sync.Map:

```go
package main

import (
    "sync"
    "fmt"
)

func main() {
    var sm sync.Map
    wg := sync.WaitGroup{}

    // Write data
    for i := 0; i < 10; i++ {
        wg.Add(1)
        go func(n int) {
            defer wg.Done()
            sm.Store(n, n*n)
        }(i)
    }

    // Read data
    for i := 0; i < 10; i++ {
        wg.Add(1)
        go func(n int) {
            defer wg.Done()
            if value, ok := sm.Load(n); ok {
                fmt.Printf("Key: %v, Value: %v\n", n, value)
            }
        }(i)
    }

    wg.Wait()
}
```

sync.Map's `Store` method is used for storing Key/Value pairs, and `Load` method for retrieving data. This way, we can safely use a map across multiple goroutines without worrying about data races.

### Conclusion

This article explored how Go handles data races in concurrent operations on maps, showcasing Go's emphasis on concurrency safety. By utilizing tools like sync.Mutex and sync.Map, we can effectively avoid data races, ensuring the stability and efficiency of our concurrent applications.

Understanding these mechanisms is crucial for writing robust Go programs.

Blog post: [From Fatal Error to sync.Map: Concurrency Strategies for Maps in Go](https://en.poloxue.com/posts/2024-01-21-fatal-error-in-concurrent-accessing-map/)
