---
title: "Diving Deep into Go's HTTP Mechanics: More Than Just Requests"
date: 2024-01-24T15:07:32+08:00
draft: true
---

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-01/2024-01-24-http-client-in-golang-01.png)

This post will implement all examples from the quick start documentation of `requests` using Go, systematically learning the use of the http client. Despite being titled a quick start, the content is quite extensive.

# Quick Start

First, let's make a GET request. The code is very simple, as shown below:

```go
func get() {
	r, err := http.Get("https://api.github.com/events")
	if err != nil {
		panic(err)
	}
	defer func() { _ = r.Body.Close() }()

	body, _ := ioutil.ReadAll(r.Body)
	fmt.Printf("%s", body)
}
```

The `http.Get` method retrieves a Response and an error, `r` and `err` respectively. `r` allows access to response information, and `err` enables error checking.

`r.Body` should be closed after being read, which can be done using `defer`. Reading the content can be achieved with `ioutil.ReadAll`.

# Request Methods

Apart from GET, HTTP has a series of other methods including POST, PUT, DELETE, HEAD, and OPTIONS. The GET method in the quick experience is implemented in a convenient way, hiding many details. Let's not use it for now.

We will introduce a general method to help us implement requests for all HTTP methods, mainly involving two important types, Client and Request.

A Client is the client that sends HTTP requests. All requests are initiated by the Client. It provides convenient request methods. For instance, a Get request can be implemented through `client.Get(url)`. A more general way is through `client.Do(req)`, where `req` is of the Request type.

A Request is a structure that describes request information, such as the request method, address, header, etc., all of which can be set through it. Request can be created with `http.NewRequest`.

Below are the implementation codes for all HTTP methods.

GET
```go
r, err := http.DefaultClient.Do(
	http.NewRequest(http.MethodGet, "https://api.github.com/events", nil))
```

POST
```go
r, err := http.DefaultClient.Do(
	http.NewRequest(http.MethodPost, "http://httpbin.org/post", nil))
```

PUT
```go
r, err := http.DefaultClient.Do(
	http.NewRequest(http.MethodPut, "http://httpbin.org/put", nil))
```

DELETE
```go
r, err := http.DefaultClient.Do(
	http.NewRequest(http.MethodDelete, "http://httpbin.org/delete", nil))
```

HEAD
```go
r, err := http.DefaultClient.Do(
	http.NewRequest(http.MethodHead, "http://httpbin.org/get", nil))
```
OPTIONS
```go
r, err := http.DefaultClient.Do(
	http.NewRequest(http.MethodOptions, "http://httpbin.org/get", nil))
```

These examples demonstrate the implementation of all HTTP methods. A few points need to be clarified.

DefaultClient, provided by the net/http package, is the default client. For general requests, we don't need to create a new Client and can use the default one.

For GET, POST, and HEAD requests, Go provides more convenient implementation methods. Request doesn't need to be created manually.

Example code, each HTTP request method has two implementations.

GET
```go
r, err := http.DefaultClient.Get("http://httpbin.org/get")
r, err := http.Get("http://httpbin.org/get")
```

POST
```go
bodyJson, _ := json.Marshal(map[string]interface{}{
	"key": "value",
})
r, err := http.DefaultClient.Post(
	"http://httpbin.org/post",
	"application/json",
	strings.NewReader(string(bodyJson)),
)
r, err := http.Post(
	"http://httpbin.org/post",
	"application/json",
	strings.NewReader(string(bodyJson)),
)
```
This also demonstrates how to submit JSON data to a POST interface, mainly the setting of content-type, which is generally `application/json` for JSON interfaces.

HEAD
```go
r, err := http.DefaultClient.Head("http://httpbin.org/get")
r, err := http.Head("http://httpbin.org/get")
```

If you look at the source code, you'll find that `http.Get` calls `http.DefaultClient.Get`. It's the same, just provided as a convenient calling method. The same applies to Head and Post.

# URL Parameters

By placing key/value pairs in the URL, we can pass data to a specific address. These key/value pairs will follow a question mark, such as `http://httpbin.org/get?key=val`. Manually constructing a URL can be tedious, but we can use methods provided by net/http to achieve this.

For example, if you want to pass `key1=value1` and `key2=value2` to `http://httpbin.org/get`, the code is as follows:

```go
req, err := http.NewRequest(http.MethodGet, "http://httpbin.org/get", nil)
if err != nil {
	panic(err)
}

params := make(url.Values)
params.Add("key1", "value1")
params.Add("key2", "value2")

req.URL.RawQuery = params.Encode()

// Specific URL situation: http://httpbin.org/get?key1=value1&key2=value2
// fmt.Println(req.URL.String())

r, err := http.DefaultClient.Do(req)
```

url.Values helps organize QueryString. Checking the source code reveals that url.Values is actually `map[string][]string`. Calling the `Encode` method passes the organized string to the request `req`'s RawQuery. url.Values can also set an array parameter, similar to the following form:

`http://httpbin.org/get?key1=value1&key2=value2&key2=value3`

How to achieve this?

```go
params := make(url.Values)
params.Add("key1", "value1")
params.Add("key2", "value2")
params.Add("key2", "value3")
```

Looking at the last line of code, you just need to add another value to `key2`.

# Response Information

After successfully executing the request, how do we view the response information? To view the response information, it's good to know the common content of responses, such as the body content (Body), status information (Status), response headers (Header), content encoding (Encoding), etc.

## Body

The process of reading the Body content was demonstrated at the very beginning. Reading the response content can be done with `ioutil`.

```
body, err := ioutil.ReadAll(r.Body)
```
Response content varies. If it's JSON, you can directly use `json.Unmarshal` to decode it. JSON knowledge won't be introduced here.

`r.Body` implements the io.ReadeCloser interface. To reduce resource wastage, it should be released promptly, which can be done with `defer`.

```go
defer func() { _ = r.Body.Close() }()
```

## StatusCode

In the response information, besides the Body main content, there are other pieces of information, such as the status code and charset.

```
r.StatusCode
r.Status
```

`r.StatusCode` is the HTTP return code, and `Status` is the return status description.

## Header

Response header information can be obtained through `Response.Header`. It's worth mentioning that the key in the response header is case-insensitive.

```go
r.Header.Get("content-type

")
r.Header.Get("Content-Type")
```
You will find that `content-type` and `Content-Type` get the same content.

## Encoding

How to recognize the encoding of response content? We need the help of the `http://golang.org/x/net/html/charset` package. Let's define a function as follows:

```go
func determineEncoding(r *bufio.Reader) encoding.Encoding {
	bytes, err := r.Peek(1024)
	if err != nil {
		fmt.Printf("err %v", err)
		return unicode.UTF8
	}

	e, _, _ := charset.DetermineEncoding(bytes, "")

	return e
}
```

How to call it?

```go
bodyReader := bufio.NewReader(r.Body)
e := determineEncoding(bodyReader)
fmt.Printf("Encoding %v\n", e)

decodeReader := transform.NewReader(bodyReader, e.NewDecoder())
```

We generate a new reader using `bufio`, then use `determineEncoding` to detect content encoding, and perform encoding conversion through `transform`.

# Image Download

If the accessed content is an image, how do we download it? For example, the image at the following address.

https://pic2.zhimg.com/v2-5e8b41cae579722bd6b8a612bf1660e6.jpg

It's actually quite simple; you just need to create a new file and save the response content into it.

```go
f, err := os.Create("as.jpg")
if err != nil {
	panic(err)
}
defer func() { _ = f.Close() }()

_, err = io.Copy(f, r.Body)
if err != nil {
	panic(err)
}
```

`r` is the Response. We create a new file using `os`, and then save the response content into the file through `io.Copy`.

# Custom Request Headers

How to customize request headers? Request already provides the corresponding method, which can be completed through `req.Header.Add`.

For example, suppose we are going to access `http://httpbin.org/get`, but this address has set a crawling strategy for user-agent. We need to modify the default user-agent.

Example code:

```go
req, err := http.NewRequest(http.MethodGet, "http://httpbin.org/get", nil)
if err != nil {
	panic(err)
}

req.Header.Add("user-agent", "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_0)")
```

This can complete the task as described above.

# Complex POST Requests

We've already demonstrated how to submit JSON data to a POST interface. Next, we will introduce other ways to submit data to a POST interface, namely form submission and file submission.

## Form Submission

Form submission is a very common function, so in net/http, in addition to providing standard usage, it also offers a simplified method.

Let's first introduce the standard implementation method.

For example, suppose we are going to submit a form with name `poloxue` and password `123456` to `http://httpbin.org/post`.

```go
payload := make(url.Values)
payload.Add("name", "poloxue")
payload.Add("password", "123456")
req, err := http.NewRequest(
	http.MethodPost,
	"http://httpbin.org/post",
	strings.NewReader(payload.Encode()),
)
if err != nil {
	panic(err)
}
req.Header.Add("Content-Type", "application/x-www-form-urlencoded")

r, err := http.DefaultClient.Do(req)
```

The POST payload is a string like `name=poloxue&password=123456`, so we can organize it with `url.Values`.

The content submitted to `NewRequest` must be of a type that implements the Reader interface, so it needs to be converted with `strings.NewReader`.

The content-type of form submission should be `application/x-www-form-urlencoded`, which also needs to be set.

The complex method has been introduced. Now, let's introduce the simplified method. In fact, form submission can be completed by simply calling `http.PostForm`. Example code is as follows:

```go
payload := make(url.Values)
payload.Add("name", "poloxue")
payload.Add("password", "123456")
r, err := http.PostForm("http://httpbin.org/post", form)
```

It's that simple.

## File Submission

File submission should be one of the more complex contents in HTTP requests. Actually, it's not too difficult. Different from other requests, we need to spend some effort reading the file and organizing the data for submitting the POST.

For example, suppose I have an image file named `as.jpg`, located in the `/Users/polo` directory. Now I want to submit this picture to `http://httpbin.org/post`.

We need to first organize the content for POST submission, as follows:

```go
filename := "/Users/polo/as.jpg"

f, err := os.Open(filename)
if err != nil {
	panic(err)
}
defer func() { _ = f.Close() }()

uploadBody := &bytes.Buffer{}
writer := multipart.NewWriter(uploadBody)

fWriter, err := writer.CreateFormFile("uploadFile", filename)
if err != nil {
	fmt.Printf("copy file writer %v", err)
}

_, err = io.Copy(fWriter, f)
if err != nil {
	panic(err)
}

fieldMap := map[string]string{
	"filename": filename,
}
for k, v := range fieldMap {
	_ = writer.WriteField(k, v)
}

err = writer.Close()
if err != nil {
	panic(err)
}
```

I believe the data organization can be completed in several steps:

- First, open the file to be uploaded and use `defer f.Close()` to prepare for resource release.
- Second, create a `bytes.Buffer`, named `uploadBody`, for storing the upload content.
- Third, create `writer` through `multipart.NewWriter` to write the content provided by the file to the buffer.
- Fourth, create the upload file through `writer.CreateFormFile` and write the content into it through `io.Copy`.
- Finally, add other additional information through `writer.WriteField`. Remember to close the writer at the end.

At this point, the data for file upload is organized. Next, you only need to call the `http.Post` method to complete the file upload.

```go
r, err := http.Post("http://httpbin.org/post", writer.FormDataContentType(), uploadBody)
```

One thing to note is that the request's content-type needs to be set, which can be obtained through `writer.FormDataContentType()`.

With this, the file submission is also completed. I wonder if it feels very simple.

# Cookie

Mainly involves two parts: reading the response cookie and setting the request cookie. The method to get the response cookie is very simple, just call `r.Cookies`.

Let's focus on how to set request cookies. There are two ways to set cookies: one is set on the Client, and the other is set on the Request.

## Setting Cookie on Client

Let's look at the example code:

```go
cookies := make([]*http.Cookie, 0)

cookies = append(cookies, &http.Cookie{
	Name:   "name",
	Value:  "poloxue",
	Domain: "httpbin.org",
	Path:   "/cookies",
})
cookies = append(cookies, &http.Cookie{
	Name:   "id",
	Value:  "10000",
Domain: "httpbin.org",
	Path:   "/elsewhere",
})

url, err := url.Parse("http://httpbin.org/cookies")
if err != nil {
	panic(err)
}

jar, err := cookiejar.New(nil)
if err != nil {
	panic(err)
}
jar.SetCookies(url, cookies)

client := http.Client{Jar: jar}

r, err := client.Get("http://httpbin.org/cookies")
```

In the code, we first created an `http.Cookie` slice and then added 2 Cookie data to it. Through cookiejar, 2 newly created cookies were saved.

This time we can't use the default `DefaultClient` anymore. Instead, we need to create a new Client and bind the cookiejar with the client that holds cookie information. After that, just use the newly created Client to initiate the request.

## Setting Cookie on Request

Setting cookies on the request can be achieved through `req.AddCookie`. Example code:

```go
req, err := http.NewRequest(http.MethodGet, "http://httpbin.org/cookies", nil)
if err != nil {
	panic(err)
}

req.AddCookie(&http.Cookie{
	Name:   "name",
	Value:  "poloxue",
	Domain: "httpbin.org",
	Path:   "/cookies",
})

r, err := http.DefaultClient.Do(req)
```

It's quite simple and doesn't need much explanation.

What's the difference between setting a cookie on the Client and on the Request? One obvious difference is that cookies set on the Request are only effective for that request, while cookies on the Client are always effective as long as you use this newly created Client.

# Redirects and Request History

By default, all types of requests will automatically handle redirects.

Python's requests package does not redirect HEAD requests, but the test results show that net/http's HEAD automatically redirects.

Redirect control in net/http can be handled through a member named `CheckRedirect` in the Client, which is a function type. Defined as follows:

```go
type Client struct {
	...
	CheckRedirect func(req *Request, via []*Request) error
	...
}
```

Next, let's see how to use it.

Suppose we want to implement a function: to prevent circular redirects, the number of redirects should not exceed 10 times, and we also want to record the history of Responses.

Example code:

```go
var r *http.Response
history := make([]*http.Response, 0)

client := http.Client{
	CheckRedirect: func(req *http.Request, hrs []*http.Request) error {
		if len(hrs) >= 10 {
			return errors.New("redirect to many times")
		}

		history = append(history, req.Response)
		return nil
	},
}

r, err := client.Get("http://github.com")
```

First, a variable of `http.Response` slice named `history` was created. Then in `http.Client`, `CheckRedirect` was assigned an anonymous function to control the behavior of redirects. The first parameter of the `CheckRedirect` function represents the Request that will be requested next, and the second parameter represents the Request that has already been requested.

When a redirect occurs, the current Request will save the Response from the previous request, so here we can append `req.Response` to the `history` variable.

# Timeout Settings

If there's no response from the server after the Request is sent, it can be quite awkward. So, we might wonder if we can set timeout rules for requests? Undoubtedly, we can.

Timeout can be divided into connection timeout and response read timeout, both of which can be set. But in normal circumstances, without such a clear distinction, you can also set a total timeout.

## Total Timeout

The setting of the total timeout time is bound to a member named `Timeout` in the Client, `Timeout` is `time.Duration`.

Suppose the timeout is 10 seconds, example code:

```go
client := http.Client{
	Timeout:   time.Duration(10 * time.Second),
}
```

## Connection Timeout

Connection timeout can be achieved through the `Transport` in the Client. `Transport` has a member function named `Dial`, which can be used to set the connection timeout. `Transport` is the data transporter at the bottom layer of HTTP.

Suppose the connection timeout is set to 2 seconds, example code:

```go
t := &http.Transport{
	Dial: func(network, addr string) (net.Conn, error) {
		timeout := time.Duration(2 * time.Second)
		return net.DialTimeout(network, addr, timeout)
	},
}
```
In the `Dial` function, we connect to the network using `net.DialTimeout`, achieving the connection timeout functionality.

## Read Timeout

Read timeout is also set through the `Transport` of the Client, for example, setting the read timeout of the response to 8 seconds.

Example code:
```go
t := &http.Transport{
	ResponseHeaderTimeout: time.Second * 8,
}
Combined with all, the Client creation code is as follows:

t := &http.Transport{
	Dial: func(network, addr string) (net.Conn, error) {
		timeout := time.Duration(2 * time.Second)
		return net.DialTimeout(network, addr, timeout)
	},
	ResponseHeaderTimeout: time.Second * 8,
}
client := http.Client{
	Transport: t,
	Timeout:   time.Duration(10 * time.Second),
}
```

Apart from the above timeout settings, `Transport` also has some other settings about timeout. You can check the definition of `Transport`. There are three other timeout-related definitions:

```go
// IdleConnTimeout is the maximum amount of time an idle
// (keep-alive) connection will remain idle before closing
// itself.
// Zero means no limit.
IdleConnTimeout time.Duration

// ResponseHeaderTimeout, if non-zero, specifies the amount of
// time to wait for a server's response headers after fully
// writing the request (including its body, if any). This
// time does not include the time to read the response body.
ResponseHeaderTimeout time.Duration

// ExpectContinueTimeout, if non-zero, specifies the amount of
// time to wait for a server's first response headers after fully
// writing the request headers if the request has an
// "Expect: 100-continue" header. Zero means no timeout and
// causes the body to be sent immediately, without
// waiting for the server to approve.
// This time does not include the time to send the request header.
ExpectContinueTimeout time.Duration
```

These are `IdleConnTimeout` (connection idle timeout, `keep-live` activated), `TLSHandshakeTimeout` (TLS handshake time), and `ExpectContinueTimeout` (seems to be already included in `ResponseHeaderTimeout`, see comments).

With this, the timeout setting is complete. Compared to Python's requests, it is indeed a lot more complex.

# Request Proxy

Proxies are quite important, especially for developers working on crawlers. So how to set up a proxy in net/http? This task still relies on the `Transport` member of the Client, which is quite important.

`Transport` has a member named `Proxy`. Let's see how to use it. Suppose we want to request Google's homepage through a proxy, the proxy address is `http://127.0.0.1:8087`.

Example code:

```go
proxyUrl, err := url.Parse("http://127.0.0.1:8087")
if err != nil {
	panic(err)
}
t := &http.Transport{
	Proxy:           http.ProxyURL(proxyUrl),
	TLSClientConfig: &tls.Config{InsecureSkipVerify: true},
}
client := http.Client{
	Transport: t,
	Timeout:   time.Duration(10 * time.Second),
}

r, err := client.Get("https://google.com")
```

The main focus should be on the code creating `http.Transport`. There are two parameters, namely `Proxy` and `TLSClientConfig`, used to set the proxy and disable HTTPS verification, respectively. I found that the request can be successful even without setting `TLSClientConfig`, but I haven't studied the specific reasons carefully.

# Error Handling

Error handling doesn't need much introduction. In Go, general errors mainly involve checking the returned error, and HTTP requests do the same. Depending on the situation, they return corresponding error information, such as timeouts, network connection failures, etc.

Errors in the example code are thrown out through panic, but real projects definitely don't handle it this way. We need to record relevant logs and always be prepared for error recovery.

# Summary

This post, guided by Python's requests documentation, has organized how the examples in the requests quick start documentation are implemented in Go. It's worth mentioning that Go also provides a clone version corresponding to requests, [GitHub address](https://github.com/levigross/grequests). I haven't looked at it yet, but interested friends can study it.

Blog post: [A Guide to Go's HTTP Request System](https://en.poloxue.com/posts/2024-01-24-http-client-in-golang/)
