---
title: "The Go Developer's Guide to HTTP File Uploads using net/http Package"
date: 2024-11-25T15:05:54+08:00
draft: false
comment: true
description: "The previous post presented a comprehensive introduction to sending HTTP requests using Go's net/http, covering all methods commonly used in daily scenarios. However, as the focus was primarily on usage, some topics, such as the implementation principles of HTTP file uploads, might still be challenging to understand without a deeper understanding of the underlying mechanics."
---

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-01/2024-01-25-golang-http-upload-file-01.webp)

The previous post presented a comprehensive introduction to sending HTTP requests using Go's net/http, covering all methods commonly used in daily scenarios. However, as the focus was primarily on usage, some topics, such as the implementation principles of HTTP file uploads, might still be challenging to understand without a deeper understanding of the underlying mechanics.

Today, based on this topic, we will delve into how Go implements file uploads.

## Introduction

In simple terms, HTTP file upload can be divided into three steps: organizing the request body, setting the Content-Type, and sending the POST request. We won't delve into POST requests here but will instead focus on the request body and the content type of the request body.

The request body is commonly used for POST requests. Although GET also supports a request body, it is conventionally ignored by the server due to established norms.

What is Content-Type?

The format of the request body is not fixed and can be quite varied. To clarify the content type of the request body, HTTP defines a request header called Content-Type.

Common Content-Type options include `application/x-www-form-urlencoded` (default form submission), `application/json` (json), `text/xml` (xml format), `text/plain` (plain text), and `application/octet-stream` (binary stream), among others.

## Submitting Forms

File upload can be understood as a special case of form submission. Let's introduce the whole process with a simple example of form submission.

Below is the HTTP request text for form submission.

```http
POST http://httpbin.org/post HTTP/1.1
Content-Type: application/x-www-form-urlencoded

username=poloxue&password=123456
```

The Content-Type is `application/x-www-form-urlencoded`, and the data is organized in a urlencoded manner.

Let's first implement it with an HTML form, as shown below:

```html
<form method="post">
    <input type="text" name="username">
    <input type="password" name="password">
    <input type="submit">
</form>
```

Submitting the form via POST, the default Content-Type is `application/x-www-form-urlencoded`.

Go's implementation code:

```go
data := make(url.Values)
data.Set("username", "poloxue")
data.Set("password", "123456")

// Organize data in urlencoded format
body, _ := data.Encode()

// Create a request and set the content type
request, _ := http.NewRequest(
    http.MethodPost,
    "http://httpbin.org/post",
    bytes.NewReader(body),
)

request.Header.Set(
    "content-type",
    "application/x-www-form-urlencoded",
)

http.DefaultClient.Do(request)
```

Reflect on the three steps mentioned earlier: organizing request body data, setting Content-Type, and sending the request.

Go's net/http package also offers a more concise method, http.Post.

```go
http.Post(
    "http://httpbin.org/post",
    "application/x-www-form-urlencoded",
    bytes.NewReader(body),
)
```

## File Upload RFC 1867

The need for file uploads is common, but the default form submission method does not support it.

If it's a single file upload, it can be implemented through a binary stream in the body. However, for more complex scenarios, like uploading multiple files, a custom upload protocol is required, and both the client and server need to provide corresponding support.

Wouldn't it be better if there was a standard for such a common need? To address this issue, [RFC 1867](https://tools.ietf.org/html/rfc1867 "RFC 1867") was created. Its main content includes:

- Adding a file option for the input tag's type.
- Adding the `multipart/form-data` option for the form's enctype.

Below is a form that supports file submission.

```html
<form
    action="http://httpbin.org/post"
    method="post"
    enctype="multipart/form-data"
>
  <input type="text" name="words"/>
  <input type="file" name="uploadfile1">
  <input type="file" name="uploadfile2">
  <input type="submit">
</form>
```

After submitting the form, you will see the content of the request roughly like this:

```HTTP
POST http://httpbin.org/post HTTP/1.1
Content-Type: multipart/form-data; boundary=285fa365bd76e6378f91f09f4eae20877246bbba4d31370d3c87b752d350

multipart/form-data; boundary=285fa365bd76e6378f91f09f4eae20877246bbba4d31370d3c87b752d350
--285fa365bd76e6378f91f09f4eae20877246bbba4d31370d3c87b752d350
Content-Disposition: form-data; name="uploadFile1"; filename="uploadfile1.txt"
Content-Type: application/octet-stream

upload file1
--285fa365bd76e6378f91f09f4eae20877246bbba4d31370d3c87b752d350
Content-Disposition: form-data; name="uploadFile1"; filename="uploadfile2.txt"
Content-Type: application/octet-stream

upload file2
--285fa365bd76e6378f91f09f4eae20877246bbba4d31370d3c87b752d350
Content-Disposition: form-data; name="words"

123
--285fa365bd76e6378f91f09f4eae20877246bbba4d31370d3c87b752d350--
```

Note: If you use Chrome's developer tools, for performance reasons, you won't be able to see this part of the content. Moreover, if the submission is a binary stream, it's just a mess, and there's nothing to see.

Apart from `multipart/form-data`, Content-Type also includes additional content such as `boundary=xxx`. `Boundary` means boundary and is similar to `&` in the `application/x-www-form-urlencoded` method, used to separate different input fields. The reason why `boundary` is so complicated is that general text content can be separated using `&`, but if it's a file stream, `&` might conflict with the content, hence the higher requirement for the uniqueness of the boundary.

The detailed format of `multipart/form-data` content will not be introduced here. Let's continue to discuss how to implement this functionality in Go.

## Implementation

How to implement file uploads in Go?

The main logic is still the three steps: organizing data, setting Content-Type, and sending the request. However, the organization of this part of the data is much more complicated than the urlencoded method of the form submission.

Go's simplicity shines through at this moment because the standard library `mime/multipart` has already provided very convenient methods, eliminating the need for manual organization.

Suppose we now want to implement the functionality of the previous form submission, i.e., submit two files, uploadfile1 and uploadfile2, and a field words.

First, create a `byte.Buffer` type variable, `body`, to save the data. On top of it, create a `multipart.Writer` to organize the data to be submitted. The code is as

 follows:

```go
bodyBuf := &bytes.Buffer{}
writer := multipart.NewWriter(payloadBuf)
```

First, organize the file content. The organization logic for the two files is the same, so let's introduce it using uploadfile1 as an example. On top of `writer`, create a `fileWriter` to write the content of the file uploadFile1,

```go
fileWriter, err := writer.CreateFormFile("uploadFile1", filename)
```

Open the file to be uploaded, uploadfile1, and copy the file content to `fileWriter`, as follows:

```go
f, err := os.Open("uploadfile1")
    ...
io.Copy(fileWriter, f)
```

Adding fields is very simple. Suppose we set `words` to 123. The code is as follows:

```go
writer.WriteField("words", "123")
```

After completing all content settings, be sure to close the `Writer`. Otherwise, the request body will lack the end boundary.

```go
writer.Close()
```

The data organization is completed.

Next, just set the data to `http.Post`.

```go
r, err := http.Post(
    "http://httpbin.org/post",
    writer.FormDataContentType(),
    body,
)
```

The form submission supporting file upload is completed.

## Conclusion

This post mainly introduces how to implement file uploads using Go, essentially organizing the request body for submitting files. To clearly understand the process of organizing the request body, one must be familiar with the related HTTP protocol, [rfc 1867](https://tools.ietf.org/html/rfc1867 "rfc 1867").

Blog post: [The Go Developer's Guide to HTTP File Uploads](http://en.poloxue.com/posts/2019-12-10-golang-http-upload-file/)
