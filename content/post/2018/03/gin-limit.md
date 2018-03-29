---
title: "Gin 源码学习（Gin Limit Middleware）"
subtitle: Gin Limit Middleware
tags: [ "Go", "Gin", "Gin Middleware", "Limit" ]
description: 记录一些值得学习的Go开源项目
categories: [ "Go", "Gin" ]
isCJKLanguage: true
date: 2018-03-23T22:40:52+08:00
draft: false
---

https://github.com/aviddiviner/gin-limit 请求限制

代码超级简单呐，利用空结构体和缓冲为n的chan做连接限制，请看：

```go
func MaxAllowed(n int) gin.HandlerFunc {
	sem := make(chan struct{}, n)
	acquire := func() { sem <- struct{}{} }
	release := func() { <-sem }
	return func(c *gin.Context) {
		acquire() // before request
		defer release() // after request
		c.Next()
	}
}
```

<!--more-->

但是不是有用，我本机测试了没看出来，当限制太小，连接请求太多时，套接口出错较多，这样也不太好不太稳定吧。以下是使用压测工具wrk -t12 -c400 -d20s http://localhost:8024/ （12线程400连接在20秒内连续请求），后台设置中间件函数MaxAllowed(20)时的测试数据（MacBookPro 2012 mid 8G）：

```c
Running 20s test @ http://localhost:8024/
  12 threads and 400 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency   195.50ms   28.03ms 325.07ms   80.47%
    Req/Sec   161.16     41.76   292.00     73.14%
  38585 requests in 20.10s, 23.36MB read
  Socket errors: connect 0, read 249, write 0, timeout 0
Requests/sec:   1919.73
Transfer/sec:      1.16MB
```

另外这个中间件的机制还没太弄懂，acquire和release两个函数不都是在一个连接到来的请求时做的吗，其他连接没有控制到啊。据自己理解，应该是限制一个TCP Listener最多Accept允许**n**个连接进入吧。