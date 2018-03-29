---
title: "Go 源码学习（ gossed ）"
subtitle: benas-gossed
tags: [ "Go", "SSE" ]
description: 记录一些值得学习的Go开源项目
categories: [ "Go", "SSE" ]
isCJKLanguage: true
date: 2018-03-26T15:09:26+08:00
draft: false
---

上次说到Gin框架中有`Server-Send Events`的实现，这次针对SSE又进一步做了些研究，涉及到两个项目，分两篇学习。项目地址：**https://github.com/benas/gossed** 

这个项目是利用[**alexandrevicenzi/go-sse**](https://github.com/alexandrevicenzi/go-sse) 项目来实现将标准输入数据转发给浏览器，这是个有趣的项目，可以用浏览器来实时监控某些你想要监控的数据。:)

![](https://raw.githubusercontent.com/benas/ssed/master/examples/system-stats/screenshot.png)

<!--more-->

`gossed`用来做系统监控还真是不错的，看看上面项目自带的`system-stats` 的示例截图，仅仅是几句脚本就可以将系统运行数据动态展示，简单好用。

脚本复制一下，以后说不定用上。system-stats.sh 的内容：

```shell
#!/usr/bin/env bash

CPU=$(top -l 1 | grep "CPU usage" | awk '{print $3}' | tr '%' ' ')
MEMORY=$(top -l 1 | grep "PhysMem" | awk '{print $2}' | tr 'M' ' ')
PROCESSES=$(top -l 1 | grep "Processes" | awk '{print $2}')
THREADS=$(top -l 1 | grep "Processes" | awk '{print $10}')

echo "{\"cpu\": \"$CPU\", \"memory\": \"$MEMORY\", \"processes\": \"$PROCESSES\", \"threads\": \"$THREADS\"}";
```
循环执行并用管道传递给`gossed` 程序：	
```shell
$> while sleep 1; do ./system-stats.sh; done | gossed
```

这样你就可以在浏览器里面监控系统资源情况啦（使用示例中的html文件，如非本机则需要改下IP）。

> 这个小工具的实现很简单，看下源码：（就是这么短，我甚至可以全部复制过来，不占什么篇幅，浏览一下也花不了多少时间。）

```go
package main

import (
	"bufio"
	"flag"
	"log"
	"net/http"
	"os"
	"strconv"
	"io/ioutil"
	"fmt"

	"github.com/alexandrevicenzi/go-sse"
)

func main() {
	log.SetOutput(ioutil.Discard)

	port := flag.Int("port", 3000, "port on which events will be sent")
	flag.Parse()

	s := sse.NewServer(&sse.Options{
		Headers: map[string]string{
			"Access-Control-Allow-Origin":  "*",
			"Access-Control-Allow-Methods": "GET, OPTIONS",
			"Access-Control-Allow-Headers": "Keep-Alive,X-Requested-With,Cache-Control,Content-Type,Last-Event-ID",
		},
	})

	defer s.Shutdown()

	http.Handle("/", s)

	go func() {
		scanner := bufio.NewScanner(os.Stdin)
		for scanner.Scan() {
			s.SendMessage("/", sse.SimpleMessage(scanner.Text()))
		}
	}()

	fmt.Println("Listening on port: " + strconv.Itoa(*port))
	http.ListenAndServe(":"+strconv.Itoa(*port), nil)
}
```

核心就是`import`了`"github.com/alexandrevicenzi/go-sse"` 这个库。利用这个库，创建了一个`sse`的服务，监听在`3000`端口，访问`http` 的`/`目录就可以获取其转发的`os.Stdin` 标准输入数据。

`HTML` 文件中建立`EventSource` 事件监听示例：

```javascript
    <script type="text/javascript">
        var source = new EventSource("http://localhost:3000/");
        source.onmessage = function(event) {
            var content = document.getElementById('content');
            content.innerHTML = content.innerHTML + event.data + '<br/>';
        };
    </script>
```

