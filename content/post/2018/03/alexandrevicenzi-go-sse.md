---
title: "Go 源码学习（ go-sse ）"
subtitle: alexandrevicenzi-go-sse
tags: [ "Go", "SSE" ]
description: 记录一些值得学习的Go开源项目
categories: [ "Go", "SSE" ]
isCJKLanguage: true
date: 2018-03-26T16:29:26+08:00
draft: false
---

上次学习了`gossed` 项目 **https://github.com/benas/gossed** 

再来学习一下它用到的核心开源库：**https://github.com/alexandrevicenzi/go-sse** 

这个项目也是一个Go语言实现SSE的库，按照README里面的说明：支持多通道隔离，广播，自定义`headers`，支持`Last-Event-ID`，遵循[**SSE规范**](https://html.spec.whatwg.org/multipage/server-sent-events.html)。

这个库主要由三个核心源文件 `sse.go`，`client.go`，`channel.go` 组成，分别完成相应功能，具体看一下代码。

`client.go` 中Client结构，收到http请求后创建：	

```go
type Client struct {
    lastEventId,
    channel string
    send chan *Message
}
```

<!--more-->

`client.go` 中Channel结构，Channel由Client连接时创建：	

```go
type Channel struct {
    lastEventId,
    name string
    clients map[*Client]bool
}
```

`message.go` 中Message结构，这基本上就是SSE规范里面要传送的数据结构：	

```go
type Message struct {
    id,
    data,
    event string
    retry int
}
```

`sse.go`中server结构，实现`ServeHTTP`方法，这样就实现`net/http`包中的`Handler`接口：	

```go
type Server struct {
    options *Options
    channels map[string]*Channel
    addClient chan *Client
    removeClient chan *Client
    shutdown chan bool
    closeChannel chan string
}
```


实现`ServeHTTP`方法，注意，作者单建一个`goroutine`读取`closeNotify` chan阻塞住，收到http断开信号则写删除客户端的chan，并且如果有消息则发送消息：	

```go
func (s *Server) ServeHTTP(response http.ResponseWriter, request *http.Request) {
    ...
    if request.Method == "GET" {
        ...
        lastEventId := request.Header.Get("Last-Event-ID")
        c := NewClient(lastEventId, channelName)
        s.addClient <- c
        closeNotify := response.(http.CloseNotifier).CloseNotify()

        go func() {
            <-closeNotify
            s.removeClient <- c
        }()

        for msg := range c.send {
            ...
        }
    } else if request.Method != "OPTIONS" {
        response.WriteHeader(http.StatusMethodNotAllowed)
    }
}
```

server创建后，启动一个`goroutine`执行任务分发：	

```go
// NewServer creates a new SSE server.
func NewServer(options *Options) *Server {
    ...
    s := &Server{...}
    go s.dispatch()
    return s
}
```

`dispatch()`，利用读取三个chan来监控客户端连接、断开、服务停止：	

```go
func (s *Server) dispatch() {
    for {
        select {

        // New client connected.
        case c := <- s.addClient:
            ch, exists := s.channels[c.channel]

            if !exists {
                ch = NewChannel(c.channel)
                s.channels[ch.name] = ch
                log.Printf("go-sse: channel '%s' created.", ch.name)
            }

            ch.addClient(c)

        // Client disconnected.
        case c := <- s.removeClient:
            if ch, exists := s.channels[c.channel]; exists {
                ch.removeClient(c)

                log.Printf("go-sse: checking if channel '%s' has clients.", ch.name)

                if ch.ClientCount() == 0 {
                    delete(s.channels, ch.name)
                    ch.Close()
                    log.Printf("go-sse: channel '%s' has no clients.", ch.name)
                }
            }

        // Close channel and all clients in it.
        case channel := <- s.closeChannel:
            if ch, exists := s.channels[channel]; exists {
                delete(s.channels, channel)
                ch.Close()
            } else {
                log.Printf("go-sse: requested to close channel '%s', but it doesn't exists.", channel)
            }

        // Event Source shutdown.
        case <- s.shutdown:
            s.close()
            close(s.addClient)
            close(s.removeClient)
            close(s.closeChannel)
            close(s.shutdown)
            log.Printf("go-sse: server stoped.")
            return
        }
    }
}
```

最后，贴一下这个开源库的使用示例，后台Go程序：	

```go
func main() {
    s := sse.NewServer(nil)
    defer s.Shutdown()

    http.Handle("/", http.FileServer(http.Dir("./static")))
    http.Handle("/events/", s)

    go func () {
        for {
            s.SendMessage("/events/channel-1", sse.SimpleMessage(time.Now().String()))
            time.Sleep(5 * time.Second)
        }
    }()

    log.Println("Listening at :3000")
    http.ListenAndServe(":3000", nil)
}
```

前端HTML：	

```html
<!DOCTYPE html>
<html>
<head>
    <title>SSE Examples</title>
</head>
<body>
    <strong>Messages</strong>
    <br>
    <div id="messages"></div>

    <script type="text/javascript">
        e1 = new EventSource('/events/channel-1');
        e1.onmessage = function(event) {
            document.getElementById('messages').innerHTML += event.data + '<br>';
        };
    </script>
</body>
</html>

```

最后，有关`Last-Event-ID`的相关细节还没太明白，另外示例中有一些导出方法没有给出使用例子，需要使用者自行阅读代码，例如从Server实例获取Channel和Client，然后对其相应处理等等。列出一些重要的方法，等我用到的时候再仔细研究吧。

```go
func (s *Server) Channels() []string
func (s *Server) GetChannel(name string) (*Channel, bool)
func (s *Server) HasChannel(name string) bool
func (s *Server) CloseChannel(name string)
...
func (c *Channel) SendMessage(message *Message)
func (c *Channel) Close()
func (c *Channel) ClientCount() int
func (c *Channel) LastEventId() string
...
func (c *Client) SendMessage(message *Message)
func (c *Client) Channel() string
func (c *Client) LastEventId() string
```

