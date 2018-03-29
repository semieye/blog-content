---
title: "Gin 源码学习（Gin Cache Middleware）"
subtitle: Gin Cache Middleware
tags: [ "Go", "Gin", "Gin Middleware", "Cache" ]
description: 记录一些值得学习的Go开源项目
categories: [ "Go", "Gin" ]
isCJKLanguage: true
date: 2018-03-24T18:55:26+08:00
draft: false
---

https://github.com/gin-contrib/cache 缓存

<!--网易云音乐 {{% music "28196554" %}}-->

这个项目是Gin的一个Middleware，用于缓存请求结果，目前支持 [<u>go-cache</u>](https://github.com/robfig/go-cache) 内存缓存（ps：这个库最新的应该是这个地址：[<u>patrickmn/go-cache</u>](https://github.com/patrickmn/go-cache)）、[<u>memcached</u>](https://github.com/bradfitz/gomemcache) 缓存、[<u>redis</u>](https://github.com/garyburd/redigo/redis) 缓存三种类型。这三个库以后再阅读学习，今天先完成这个项目的学习。缓存用的key采用`urlEscape`函数生成，规则为：前缀+冒号+URL(或其MD5)。

示例中仅仅给了一个`CachePage`的用法：

```go
	r.GET("/cache_ping", cache.CachePage(store, time.Minute, func(c *gin.Context) {
		c.String(200, "pong "+fmt.Sprint(time.Now().Unix()))
	}))
```

<!--more-->

但是`cache.go`中还有两个没想明白的函数，不知道该怎么用：

```go
func Cache(store *persistence.CacheStore) gin.HandlerFunc {
	return func(c *gin.Context) {
		c.Set(CACHE_MIDDLEWARE_KEY, store)
		c.Next()
	}
}

func SiteCache(store persistence.CacheStore, expire time.Duration) gin.HandlerFunc {
	return func(c *gin.Context) {
		var cache responseCache
		url := c.Request.URL
		key := urlEscape(PageCachePrefix, url.RequestURI())
		if err := store.Get(key, &cache); err != nil {
			c.Next()
		} else {
			c.Writer.WriteHeader(cache.Status)
			for k, vals := range cache.Header {
				for _, v := range vals {
					c.Writer.Header().Add(k, v)
				}
			}
			c.Writer.Write(cache.Data)
		}
	}
}
```

`Cache`函数使用了一个接口指针作为参数，不知道该怎么传入参数，所以我提了一个[**issue**](https://github.com/gin-contrib/cache/issues/25)希望能够得到作者的解答，我觉得也许不需要指针也是可以的。代码只有一行，使用`c.Set`将缓存`store`的实例保存，应该是为了后续处理使用Set或Get等方法存取自己的缓存数据。

`SiteCache`函数定义了一个`expire` 参数，但实际并没有用到，看代码的意思是每个请求先取缓存，取到就返回缓存，没取到就继续下一步处理。

还有一个`CachePageAtomic`函数，与`CachePage`函数类似，因为`CachePage`函数中在缓存不存在时可能会有替换Writer的情况，使用了sync.Mutex进行加互斥锁保证操作的原子性。

在`cache_test.go`中，有一些有用的地方，可以学习，例如测试缓存HTML模板文件：

```go
func TestCacheHtmlFile(t *testing.T) {
	store := persistence.NewInMemoryStore(60 * time.Second)

	router := gin.New()
	router.LoadHTMLFiles("example/template.html")
	router.GET("/cache_html", CachePage(store, time.Second*3, func(c *gin.Context) {
		c.HTML(http.StatusOK, "template.html", gin.H{"values": fmt.Sprint(time.Now().UnixNano())})
	}))

	w1 := performRequest("GET", "/cache_html", router)
	w2 := performRequest("GET", "/cache_html", router)

	assert.Equal(t, 200, w1.Code)
	assert.Equal(t, 200, w2.Code)
	assert.Equal(t, w1.Body.String(), w2.Body.String())
}
```

其中`performRequest`函数使用httptest包发测试请求、截获响应报文：

```go
func performRequest(method, target string, router *gin.Engine) *httptest.ResponseRecorder {
	r := httptest.NewRequest(method, target, nil)
	w := httptest.NewRecorder()
	router.ServeHTTP(w, r)
	return w
}
```

总而言之，通过这个库，可以了解到如何缓存WEB请求，以后可以在项目中用到部分功能和思想，例如缓存静态页面、数据较为固定的展示页面等等。