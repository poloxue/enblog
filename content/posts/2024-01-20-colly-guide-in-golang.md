---
title: "Colly: A Comprehensive Analysis and Practical Guide to High-Performance Web Crawling in Go"
date: 2020-01-20T18:43:59+08:00
draft: false
---

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-01/2024-01-20-colly-guide-in-golang-01.png)

Colly is a well-known web crawling framework implemented in Go, which is particularly suitable for high-concurrency and distributed scenarios - areas where web crawling technology thrives. Its primary attributes include being lightweight, fast, elegantly designed, and simple to distribute and extend.

This article, based on Colly's official documentation, provides a guide to learning Colly as well as my personal insights into the framework.

# How to Learn

When it comes to web crawling frameworks, Python's Scrapy is probably the most famous. It's often the first encounter many have with web crawling, and I'm no exception. Scrapy boasts comprehensive documentation and a rich set of extension components. When designing a web crawling framework, it's common to draw inspiration from Scrapy's design. I've read articles mentioning implementations in Go similar to Scrapy.

In comparison, learning resources for Colly are unfortunately scarce. Initially, I couldn't help but try to apply my experience with Scrapy to Colly, only to find that this approach doesn't work.

The natural next step was to seek out articles for guidance. However, articles on Colly are quite rare, and those available are mostly from official sources and seem somewhat incomplete. The solution? Bite the bullet and dive into the official learning materials, which generally consist of three parts: documentation, case studies, and source code.

Let's start with the official documentation today.

# Official Documentation

The [official documentation](http://go-colly.org/docs/) focuses primarily on usage. If you have experience with web crawlers, a quick read through the documentation should suffice. I spent some time organizing the official documentation according to my understanding.

The main content isn't extensive and covers topics like [installation](http://go-colly.org/docs/introduction/install/), [getting started](http://go-colly.org/docs/introduction/start/), [configuration](http://go-colly.org/docs/introduction/configuration/), [debugging](http://go-colly.org/docs/introduction/configuration/), [distributed crawling](http://go-colly.org/docs/best_practices/distributed/), [storage](http://go-colly.org/docs/best_practices/storage/), [using multiple collectors](http://go-colly.org/docs/best_practices/multi_collector/), [configuration optimization](http://go-colly.org/docs/best_practices/crawling/), and [extensions](http://go-colly.org/docs/best_practices/extensions/).

Each of these documents is concise, to the point where scrolling is barely necessary.

# [How to Install](http://go-colly.org/docs/introduction/install/)

Installing Colly is as simple as installing any other Go library. Just run:

```bash
go get -u github.com/gocolly/colly
```

One command and you're done. So easy!

# [Getting Started](http://go-colly.org/docs/introduction/start/)

Let's quickly dive into using Colly with a simple hello world example. Here are the steps:

First, import Colly.

```go
import "github.com/gocolly/colly"
```

Second, create a collector.

```go
c := colly.NewCollector()
```

Third, set up event listeners and execute event handling through callbacks.

```go
// Find and visit all links
c.OnHTML("a[href]", func(e *colly.HTMLElement) {
	link := e.Attr("href")
	// Print link
	fmt.Printf("Link found: %q -> %s\n", e.Text, link)
	// Visit link found on page
	// Only those links are visited which are in AllowedDomains
	c.Visit(e.Request.AbsoluteURL(link))
})

c.OnRequest(func(r *colly.Request) {
	fmt.Println("Visiting", r.URL)
})
```

Let's also list the types of events Colly supports:

- OnRequest: Called before a request is executed
- OnResponse: Called after a response is received
- OnHTML: Listens for and executes a selector
- OnXML: Listens for and executes a selector
- OnHTMLDetach, stops listening, with the selector string as a parameter
- OnXMLDetach, stops listening, with the selector string as a parameter
- OnScraped, executed after scraping is complete, after all work is done
- OnError, callback for errors

Finally, `c.Visit()` starts the actual web page visit.

```go
c.Visit("http://go-colly.org/")
```

The complete code for this example can be found in the `basic` directory under the `_examples` folder in the Colly source code.

# [How to Configure](http://go-colly.org/docs/introduction/configuration/)

Colly is a flexible framework that provides a plethora of configuration options for developers. By default, each option is set to a reasonable default value.

Here's how you can create a collector with the default configuration:

```go
c := colly.NewCollector()
```

And here's how to configure a collector, for example by setting a user agent and allowing repeated visits. Code is as follows:

```go
c2 := colly.NewCollector(
	colly.UserAgent("xy"),
	colly.AllowURLRevisit(),
)
```

You can also change the configuration after creating a collector.

```go
c2 := colly.NewCollector()
c2.UserAgent = "xy"
c2.AllowURLRevisit = true
```

Collector configuration can be changed at any stage of the crawl. A classic example is randomly changing the user-agent, which can help us implement simple anti-crawling techniques.

```go
const letterBytes = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ"

func RandomString() string {
	b := make([]byte, rand.Intn(10)+10)
	for i := range b {
		b[i] = letterBytes[rand.Intn(len(letterBytes))]
	}
	return string(b)
}

c := colly.NewCollector()

c.OnRequest(func(r *colly.Request) {
	r.Headers.Set("User-Agent", RandomString())
})
```

As mentioned earlier, the collector's default configuration is already optimized, but it can also be changed through environment variables. This way, you don't have to recompile every time you want to change a configuration. The environment variable configuration takes effect when the collector is initialized, but it can be overridden after the collector is officially started.

The supported configuration options are as follows:

```go
ALLOWED_DOMAINS (string slice), allowed domains, e.g., []string{"segmentfault.com", "zhihu.com"}
CACHE_DIR (string), cache directory
DETECT_CHARSET (y/n), whether to detect response encoding
DISABLE_COOKIES (y/n), disable cookies
DISALLOWED_DOMAINS (string slice), prohibited domains, same type as ALLOWED_DOMAINS
IGNORE_ROBOTSTXT (y/n), whether to ignore ROBOTS protocol
MAX_BODY_SIZE (int), maximum response size
MAX_DEPTH (int - 0 means infinite), visit depth
PARSE_HTTP_ERROR_RESPONSE (y/n), parse HTTP response errors
USER_AGENT (string)
```

These are all very straightforward options.

Let's take a look at the HTTP configuration. These are all common configurations, such as proxies, various timeout times, etc.

```go
c := colly.NewCollector()
c.WithTransport(&http.Transport{
	Proxy: http.ProxyFromEnvironment,
	DialContext: (&net.Dialer{
		Timeout:   

 30 * time.Second,          // Timeout
		KeepAlive:  30 * time.Second,          // keepAlive timeout
		DualStack:  true,
	}).DialContext,
	MaxIdleConns:          100,               // Maximum number of idle connections
	IdleConnTimeout:       90 * time.Second,  // Idle connection timeout
	TLSHandshakeTimeout:   10 * time.Second,  // TLS handshake timeout
	ExpectContinueTimeout: 1 * time.Second,  
}
```

# [Debugging](http://go-colly.org/docs/best_practices/debugging/)

When using Scrapy, it provides a very convenient shell that helps us debug easily. Unfortunately, Colly doesn't have a similar feature. The debugger here mainly refers to the collection of runtime information.

Debugger is an interface. As long as we implement the two methods in it, we can complete the collection of runtime information.

```go
type Debugger interface {
    // Init initializes the backend
    Init() error
    // Event receives a new collector event.
    Event(e *Event)
}
```

There's a typical example in the source code, [LogDebugger](https://github.com/gocolly/colly/blob/master/debug/logdebugger.go). We only need to provide the corresponding io.Writer type variable. How do we use it?

Here's an example:

```go
package main

import (
	"log"
	"os"

	"github.com/gocolly/colly"
	"github.com/gocolly/colly/debug"
)

func main() {
	writer, err := os.OpenFile("collector.log", os.O_RDWR|os.O_CREATE, 0666)
	if err != nil {
		panic(err)
	}

	c := colly.NewCollector(colly.Debugger(&debug.LogDebugger{Output: writer}), colly.MaxDepth(2))
	c.OnHTML("a[href]", func(e *colly.HTMLElement) {
		if err := e.Request.Visit(e.Attr("href")); err != nil {
			log.Printf("visit err: %v", err)
		}
	})

	if err := c.Visit("http://go-colly.org/"); err != nil {
		panic(err)
	}
}
```

After running, open `collector.log` to see the output content.

# [Distributed](http://go-colly.org/docs/best_practices/distributed/)

Distributed web crawling can be considered at several levels: the proxy level, the execution level, and the storage level.

## Proxy Level

By setting up a proxy pool, we can assign download tasks to different nodes for execution, which helps improve the webpage download speed of the crawler. At the same time, this can effectively reduce the possibility of IP bans due to excessive crawl rates.

The code for implementing proxy IPs in Colly is as follows:

```go
package main

import (
	"github.com/gocolly/colly"
	"github.com/gocolly/colly/proxy"
)

func main() {
	c := colly.NewCollector()

	if p, err := proxy.RoundRobinProxySwitcher(
		"socks5://127.0.0.1:1337",
		"socks5://127.0.0.1:1338",
		"http://127.0.0.1:8080",
	); err == nil {
		c.SetProxyFunc(p)
	}
	// ...
}
```

proxy.RoundRobinProxySwitcher is a function built into Colly that implements proxy switching through polling. Of course, we can also fully customize it.

For example, here's a case of randomly switching proxies:

```go
var proxies []*url.URL = []*url.URL{
	&url.URL{Host: "127.0.0.1:8080"},
	&url.URL{Host: "127.0.0.1:8081"},
}

func randomProxySwitcher(_ *http.Request) (*url.URL, error) {
	return proxies[random.Intn(len(proxies))], nil
}

// ...
c.SetProxyFunc(randomProxySwitcher)
```

However, it's worth noting that the crawler is still centralized at this point, with tasks executed on a single node.

## Execution Level

This approach distributes tasks to different nodes for execution, achieving true distributed crawling.

To implement distributed execution, the first issue we face is how to distribute tasks to different nodes and ensure coordinated work among different task nodes.

First, we choose an appropriate communication scheme. Common communication protocols include HTTP, TCP (one is a stateless text protocol, the other is a connection-oriented protocol). In addition, there's a rich variety of RPC protocols to choose from, such as Jsonrpc, Facebook's Thrift, Google's gRPC, etc.

The document provides an HTTP service example code that is responsible for receiving requests and executing tasks. As follows:

```go
package main

import (
	"encoding/json"
	"log"
	"net/http"

	"github.com/gocolly/colly"
)

type pageInfo struct {
	StatusCode int
	Links      map[string]int
}

func handler(w http.ResponseWriter, r *http.Request) {
	URL := r.URL.Query().Get("url")
	if URL == "" {
		log.Println("missing URL argument")
		return
	}
	log.Println("visiting", URL)

	c := colly.NewCollector()

	p := &pageInfo{Links: make(map[string]int)}

	// count links
	c.OnHTML("a[href]", func(e *colly.HTMLElement) {
		link := e.Request.AbsoluteURL(e.Attr("href"))
		if link != "" {
			p.Links[link]++
		}
	})

	// extract status code
	c.OnResponse(func(r *colly.Response) {
		log.Println("response received", r.StatusCode)
		p.StatusCode = r.StatusCode
	})
	c.OnError(func(r *colly.Response, err error) {
		log.Println("error:", r.StatusCode, err)
		p.StatusCode = r.StatusCode
	})

	c.Visit(URL)

	// dump results
	b, err := json.Marshal(p)
	if err != nil {
		log.Println("failed to serialize response:", err)
		return
	}
	w.Header().Add("Content-Type", "application/json")
	w.Write(b)
}

func main() {
	// example usage: curl -s 'http://127.0.0.1:7171/?url=http://go-colly.org/'
	addr := ":7171"

	http.HandleFunc("/", handler)

	log.Println("listening on", addr)
	log.Fatal(http.ListenAndServe(addr, nil))
}
```

This example doesn't provide the code for the scheduler, but the implementation is not complicated. After the task is completed, the service returns the corresponding link to the scheduler, which is responsible for sending new tasks to the worker nodes for execution.

If it's necessary to decide the task execution node based on the node's load, additional service monitoring APIs are needed to obtain node performance data to assist the scheduler in decision-making.

## Storage Level

We have already achieved distributed crawling by distributing tasks to different nodes for execution. However, some data, such as cookies and visited URL records, need to be shared between nodes. By default, these data are saved in memory, meaning each collector has its own separate data.

We can achieve data sharing between nodes by saving data in storage systems like Redis or MongoDB. Colly supports switching between any storage as long as the corresponding storage implements the [colly/storage.Storage](https://godoc.org/github.com/gocolly/colly/storage#Storage) interface methods.

In fact, Colly has already built-in implementations for some storage types, which you can view in the [storage](http://go-colly.org/docs/best_practices/storage/) section. We'll also touch on this topic in the next section.

# [Storage](http://go-colly.org/docs/best_practices/storage/)

As we just mentioned, let's take a closer look at the storage options already supported by Colly.

[InMemoryStorage](https://github.com/gocolly/colly/blob/master/storage/storage.go), the default storage in Colly, stores data in memory. You can replace it by using `collector.SetStorage()`.

[RedisStorage](https://github.com/gocolly/redisstorage) is also available. Perhaps because Redis is more commonly used in distributed scenarios, the official website provides a [usage example](http://go-colly.org/docs/examples/redis_backend/).

Other options include [Sqlite3Storage](https://github.com/velebak/colly-sqlite3-storage) and [MongoStorage](https://github.com/zolamk/colly-mongo-storage).

# [Multiple Collectors](http://go-colly.org/docs/best_practices/multi_collector/)

The spiders we've demonstrated so far are relatively simple, with similar processing logic. For a complex spider, we can create different collectors responsible for handling different tasks.

How should we understand this? Let's take an example.

If you've been writing spiders for a while, you must have encountered the issue of fetching parent-child pages. Typically, the processing logic for parent pages is different from that of child pages, and there's usually a need for data sharing between parent and child pages. Those who have used Scrapy will know that Scrapy manages different page logic by binding callback functions to requests, and data sharing is achieved by binding data to requests to pass data from parent pages to child pages.

After research, we find that Colly does not support Scrapy's method. So, what should we do? This is the problem we need to solve.

For different page processing logic, we can define and create multiple collectors, i.e., different collectors responsible for handling different page logic.

```go
c := colly.NewCollector(
	colly.UserAgent("myUserAgent"),
	colly.AllowedDomains("foo.com", "bar.com"),
)
// Custom User-Agent and allowed domains are cloned to c2
c2 := c.Clone()
```

Typically, the collector for the child pages is the same as for the parent pages. In the example above, the collector `c2` for the child pages clones the configuration of the parent collector `c`.

For data passing between parent and child pages, we can use Context to transfer data between different collectors. Note that this Context is just a data-sharing structure implemented by Colly, not the Context from the Go standard library.

```go
c.OnResponse(func(r *colly.Response) {
	r.Ctx.Put("Custom-header", r.Headers.Get("Custom-Header"))
	c2.Request("GET", "https://foo.com/", nil, r.Ctx, nil)
})
```

This way, we can get the data passed from the parent in the child pages through `r.Ctx`. For this scenario, you can refer to the official example [coursera_courses](http://go-colly.org/docs/examples/coursera_courses/).

# [Configuration Optimization](http://go-colly.org/docs/best_practices/crawling/)

Colly's default configuration is optimized for a small number of sites. If you're crawling a large number of sites, some improvements are needed.

## Persistent Storage

By default, cookies and URLs in Colly are saved in memory. We need to switch to a persistent storage. As mentioned earlier, Colly has already implemented some common persistent storage components.

## Enable Asynchronous Execution to Speed Up Task Execution

By default, Colly will block and wait for a request to complete, leading to an increasing number of waiting tasks. We can set the `Async` option of the collector to `true` to handle requests asynchronously and avoid this problem. If you use this method, remember to add `c.Wait()`, otherwise, the program will exit immediately.

## Disable or Limit KeepAlive Connections

Colly enables KeepAlive by default to increase the crawling speed. However, this requires open file descriptors, and for long-running tasks, it's very easy to reach the maximum descriptor limit.

Here's an example code to disable KeepAlive in HTTP:

```go
c := colly.NewCollector()
c.WithTransport(&http.Transport{
    DisableKeepAlives: true,
})
```

# [Extensions](http://go-colly.org/docs/best_practices/extensions/)

Colly provides some extensions mainly related to common web crawling functionalities, such as referer, random_user_agent, url_length_filter, etc. The source code is located under [colly/extensions/](https://github.com/gocolly/colly/tree/master/extensions).

Let's understand how to use them through an example:

```go
import (
    "log"

    "github.com/gocolly/colly"
    "github.com/gocolly/colly/extensions"
)

func main() {
    c := colly.NewCollector()
    visited := false

    extensions.RandomUserAgent(c)
    extensions.Referrer(c)

    c.OnResponse(func(r *colly.Response) {
        log.Println(string(r.Body))
        if !visited {
            visited = true
            r.Request.Visit("/get?q=2")
        }
    })

    c.Visit("http://httpbin.org/get")
}
```

You just need to pass the collector into the extension function. It's that simple.

But can we implement an extension ourselves?

When using Scrapy, if you want to implement an extension, you need to understand quite a few concepts and read its documentation carefully. But Colly doesn't even mention this in the documentation. So what should we do? It seems we can only look at the source code.

Let's open the source code of the referer plugin:

```go
package extensions

import (
	"github.com/gocolly/colly"
)

// Referer sets valid Referer HTTP header to requests.
// Warning: this extension works only if you use Request.Visit
// from callbacks instead of Collector.Visit.
func Referer(c *colly.Collector) {
	c.OnResponse(func(r *colly.Response) {
		r.Ctx.Put("_referer", r.Request.URL.String())
	})
	c.OnRequest(func(r *colly.Request) {
		if ref := r.Ctx.Get("_referer"); ref != "" {
			r.Headers.Set("Referer", ref)
		}
	})
}
```

By adding some event callbacks to the collector, you can implement an extension. The source code is so simple that you don't need a documentation explanation to implement your own extension. Of course, if you look closely, you'll find that its approach is similar to that of Scrapy, both extending request and response callbacks, and Colly's simplicity is largely due to its elegant design and Go's simple syntax.

# Conclusion

After reading Colly's official documentation, you'll find that although the documentation is rudimentary, it basically covers everything that

 should be introduced. If there are parts that weren't covered, I've supplemented them in this article. Previously, when using Go's [elastic](https://github.com/olivere/elastic) package, I also found the documentation to be scarce, but a simple read of the source code made it clear how to use it.

Perhaps this is the simplicity of Go.

Finally, if you encounter any problems while using Colly, the official examples are definitely the best practice. I suggest taking the time to read them.

My blog post: [Colly: A Comprehensive Analysis and Practical Guide to High-Performance Web Crawling in Go](https://en.poloxue.com/posts/2024-01-20-colly-guide-in-golang/).
