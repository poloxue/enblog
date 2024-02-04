---
title: "How to Check for File Existence in Go: Are There Potential Race Conditions?"
date: 2024-02-04T20:34:57+08:00
draft: true
---

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-02/2024-02-05-check-if-file-exists-in-golang-01.png)

In Go, checking if a file exists isn't as straightforward as it is in Python, where the `os.path.exists` function is readily available. However, Go offers an alternative approach through the `os.Stat` function in its standard library.

Let's dive in.

## Using `os.Stat` to Check File Status

Go doesn't provide a function like `os.Exist` directly. Instead, it offers `os.Stat` for checking file existence. This function retrieves file status information. By examining the error it returns, you can determine whether or not the file exists.

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-02/2024-02-05-check-if-file-exists-in-golang-02.png)

Here's an example:

```go
func main() {
  _, err := os.Stat("/path/to/file")
  if err != nil {
    if os.IsNotExist(err) {
      // The file does not exist
    } else {
      // Other errors
    }
  }
  // The file exists
}
```

In this code, the first return value, which is the file information, is not our main concern, so it's ignored. 

The critical part is the second return value, the `error`. If the file doesn't exist, `os.IsNotExist` checks if `error` is `os.ErrNotExist` to confirm the file's existence.

## Comparing With C

This approach in Go, using `os.Stat`, is similar to how one might check file existence in Unix C. 

Both methods are more low-level and standard compared to higher-level languages like Python, which offer convenient functions like `os.path.exist`.

In Unix C, you might do something like this:

```c
#include <errno.h>
#include <stdio.h>
#include <sys/stat.h>

int main() {
  struct stat buffer;
  int exist = stat("/path/to/file", &buffer);
  if (exist != 0) {
    if (errno == ENOENT) { /* File does not exist */ } 
    else { /* Other errors */ }
    return 0;
  }
  // File exists
  return 0;
}
```

You can see that the implementation in Go is very similar to that in C.

## Recommendations for Go 1.13 and Later

Starting with Go 1.13, it's recommended to use `os.Stat` in combination with `errors.Is`. This method offers more consistent and flexible error handling, even for wrapped errors.

Specifically, `errors.Is` is capable of recognizing errors, even those that have been encapsulated or wrapped.

At first, I was under the impression that os.IsNotExist could identify an error that had been wrapped. However, to confirm my assumption, I decided to conduct a simple test using a code snippet.

For instance:

```go
_, err := os.Stat("/path/to/file")  // A non-existent file path
werr := fmt.Errorf("Main: %w", err) // Wrapping the error
fmt.Println(os.IsNotExist(err))     // Returns true, file does not exist
fmt.Println(os.IsNotExist(werr))    // Returns false
fmt.Println(errors.Is(werr, os.ErrNotExist)) // Returns true, file does not exist
```

The test results are indicated in the comments.

As you can see, os.IsNotExist can only recognize the original error. If the error has been wrapped with fmt.Errorf, it is necessary to use errors.Is for identification.

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-02/2024-02-05-check-if-file-exists-in-golang-03.png)

In summary, while os.IsNotExist is usable, it has a specific scope of application, whereas errors.Is is more versatile.

## Directly Using `Open` to Avoid Race Conditions

In concurrent situations, file statuses can change between checks and operations. 

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-02/2024-02-05-check-if-file-exists-in-golang-04.png)

To avoid potential race conditions, it's better to attempt to open the file directly and then decide what to do based on the error returned.

Example:

```go
file, err := os.Open("/path/to/file")
if err != nil {
    if errors.Is(err, os.ErrNotExist) {
        // File does not exist
    } else {
        // Handle other types of errors
    }
}
```

Is `Open` an Atomic Operation?

Yes. System calls, including `open`, are atomic. 

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-02/2024-02-05-check-if-file-exists-in-golang-05-v1.png)

The operating system ensures the process is uninterrupted and will roll back if there's an issue. This ensures that operations like `open` are safe from disruptions, including checking file existence, creating file descriptors, allocating necessary resources, etc.

### Conclusion

This article discussed how to check if a file exists in Go, and how to avoid race conditions in file operations. It also delved into the atomic nature of Unix system calls. 

This knowledge is not just useful for Go programming but also provides a deeper understanding of system-level operations.

