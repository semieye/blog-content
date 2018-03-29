---
title: "Go 源码学习（ go-cache ）"
subtitle: go-cache
tags: [ "Go", "go-cache", "Cache", "Go Libs" ]
description: 记录一些值得学习的Go开源项目
categories: [ "Go" ]
isCJKLanguage: true
date: 2018-03-25T20:36:52+08:00
draft: false
---

https://github.com/patrickmn/go-cache 缓存

上次学习 [<u>gin-contrib/cache</u> ](https://github.com/gin-contrib/cache) 的源码时，说到其支持的 [<u>go-cache</u>](https://github.com/robfig/go-cache) 内存缓存，但发现该库最后更新是5、6年前，并且fork自：[<u>patrickmn/go-cache</u>](https://github.com/patrickmn/go-cache) ，而原作者去年仍然在少量更新，并且有一些变化，今天对这个库阅读分析一下。

这是一个类似于`memcached`的`key-value`型内存缓存，但并不是独立服务程序，可以被当做一个库编译进程序，适合单机应用，线程安全。

<!--more-->

特点：支持缓存任意Object、拥有指定有效期或者永久有效，线程安全。跟最初版本相比，增加了一些变更，貌似是建议废弃原有代码中的`Save`、`Load`、`SaveFile`、`LoadFile`函数（但函数没有删除，仍然可以使用），看注释应该是由于用户`type`类型在序列化、反序列化时需要用`gob.Register()`进行注册，那不如交给用户自己去注册，而不是库里面不管三七二十一全部注册。同时新增了`c.Items()` and `NewFrom()`两个方法返回所有未失效的或载入一个`map[string]Item`，用户可以自行持久化用于停机后快速恢复。

缓存项目`Item`的结构：

```go
type Item struct {
	Object     interface{}
	Expiration int64
}
```

缓存`Cache`的结构：

```go
type Cache struct {
	*cache
	// If this is confusing, see the comment at the bottom of New()
}
type cache struct {
	defaultExpiration time.Duration
	items             map[string]Item
	mu                sync.RWMutex
	onEvicted         func(string, interface{})
	janitor           *janitor
}
```

库中导出方法和函数，具体使用可以参看项目`README`和`cache_test.go`文件：

```go
func (c *cache) Set(k string, x interface{}, d time.Duration)
func (c *cache) SetDefault(k string, x interface{})
func (c *cache) Add(k string, x interface{}, d time.Duration) error
func (c *cache) Replace(k string, x interface{}, d time.Duration) error
func (c *cache) Get(k string) (interface{}, bool)
func (c *cache) GetWithExpiration(k string) (interface{}, time.Time, bool)
func (c *cache) Increment(k string, n int64) error
...
func (c *cache) Decrement(k string, n int64) error
...
func (c *cache) Delete(k string)
func (c *cache) DeleteExpired()
func (c *cache) OnEvicted(f func(string, interface{}))
func (c *cache) Save(w io.Writer) (err error)
func (c *cache) SaveFile(fname string) error
func (c *cache) Load(r io.Reader) error
func (c *cache) LoadFile(fname string) error
func (c *cache) Items() map[string]Item
func (c *cache) ItemCount() int
func (c *cache) Flush()
func New(defaultExpiration, cleanupInterval time.Duration) *Cache
func NewFrom(defaultExpiration, cleanupInterval time.Duration, items map[string]Item) *Cache
```

代码中有许多可以学到技巧的地方。
如缓存的自动清理：

```go
func (j *janitor) Run(c *cache) {
	ticker := time.NewTicker(j.Interval)
	for {
		select {
		case <-ticker.C:
			c.DeleteExpired()
		case <-j.stop:
			ticker.Stop()
			return
		}
	}
}
```

对`GC`的考虑：

```go
func newCacheWithJanitor(de time.Duration, ci time.Duration, m map[string]Item) *Cache {
	c := newCache(de, m)
	// This trick ensures that the janitor goroutine (which--granted it
	// was enabled--is running DeleteExpired on c forever) does not keep
	// the returned C object from being garbage collected. When it is
	// garbage collected, the finalizer stops the janitor goroutine, after
	// which c can be collected.
	C := &Cache{c}
	if ci > 0 {
		runJanitor(c, ci)
		runtime.SetFinalizer(C, stopJanitor)
	}
	return C
}
```

另外，库中原先提供了一个实验性的共享缓存，现在被单独放在一个文件里面，但暂时没有提供导出函数。看了代码，应该是去除读写排他锁控制，而采用哈希散列来处理不同请求的缓存，结构是`[]map[string]Item`，大家读写不在一个`map`里面，自然就不需要加锁了，这样用在某些场景，可以大大提高读写速度。