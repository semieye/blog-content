---
title: "Gin 源码学习（Gin SSE Middleware）"
subtitle: Gin SSE Middleware
tags: [ "Go", "Gin", "Gin Middleware", "SSE" ]
description: 记录一些值得学习的Go开源项目
categories: [ "Go", "Gin" ]
isCJKLanguage: true
date: 2018-03-26T10:14:26+08:00
draft: false
---

https://github.com/gin-contrib/sse Server-Send Events

这个项目是`Server-Send Events`协议的Go语言实现，有关`Server-Send Events`的知识，可以参看这几个链接：[谷歌HTML5项目网站一篇精彩介绍](https://www.html5rocks.com/en/tutorials/eventsource/basics/) 和 [阮一峰的中文教程](http://www.ruanyifeng.com/blog/2017/05/server-sent_events.html) ，这是`HTML5`的一个技术规范，用于服务器向浏览器单向推送消息，相对于`Websocket`协议，更简单和轻量化。

这个项目被用在了Gin框架之中，用来实现Gin对SSE的支持（`c.SSEvent()`）。具体可以参看Gin框架源码中[实时聊天高级示例](https://github.com/gin-gonic/gin/tree/master/examples/realtime-advanced) 的代码。

下面简单分析和记录一下这个库的工作原理。

Gin中提供了一个方法，支持返回`SSEvent`数据，返回的是一个`sse.Event`：

```go
// SSEvent writes a Server-Sent Event into the body stream.
func (c *Context) SSEvent(name string, message interface{}) {
	c.Render(-1, sse.Event{
		Event: name,
		Data:  message,
	})
}
```

<!--more-->

这个返回结构就是`gin-contrib/sse`库中定义的，并且实现了Render接口的两个方法。

> Gin中Render接口定义：

```go
type Render interface {
	Render(http.ResponseWriter) error
	WriteContentType(w http.ResponseWriter)
}
```

> sse中Event结构定义：

```go
type Event struct {
	Event string
	Id    string
	Retry uint
	Data  interface{}
}
```

> sse中Event实现Render接口：

```go
func (r Event) Render(w http.ResponseWriter) error {
	r.WriteContentType(w)
	return Encode(w, r)
}

func (r Event) WriteContentType(w http.ResponseWriter) {
	header := w.Header()
	header["Content-Type"] = contentType

	if _, exist := header["Cache-Control"]; !exist {
		header["Cache-Control"] = noCache
	}
}
```

> sse中实现SSE协议约定（可参看SSE的协议说明）：

```go
const ContentType = "text/event-stream"

var contentType = []string{ContentType}
var noCache = []string{"no-cache"}

var fieldReplacer = strings.NewReplacer(
	"\n", "\\n",
	"\r", "\\r")

var dataReplacer = strings.NewReplacer(
	"\n", "\ndata:",
	"\r", "\\r")
```

> sse中实现SSE协议约定（这几个函数可以看源码，按照SSE协议写数据）：

```go
func Encode(writer io.Writer, event Event) error {
	w := checkWriter(writer)
	writeId(w, event.Id)
	writeEvent(w, event.Event)
	writeRetry(w, event.Retry)
	return writeData(w, event.Data)
}
```

sse库中还有一个`sse-decoder.go` ，貌似在Gin里面没用上，因为SSE协议是服务器向浏览器单向推送，单工通道，目前应该用不上解码数据吧。