---
title: "About"
date: 2018-03-22T21:41:23+08:00
draft: false
---

一个不爱公开记录和交流的程序员。喜欢Vim，喜欢命令行，喜欢Linux。

主要使用C、PHP、Java等语言，很喜欢ANSI C，曾经喜欢Borland Delphi。

长期开发Linxu环境下公司内部应用，个人开发使用一台老MacBookPro。

最近喜欢上了Go语言，年纪大了，好记性不如烂笔头，在此做些学习记录吧。

------


{{< highlight go "linenos=inline" >}}

// Copyright © 2018 tzx All rights reserved.

package unsafe

import (
	"reflect"
	"unsafe"
)

// String converts byte slice to string.
func String(b []byte) string {
	return *(*string)(unsafe.Pointer(&b))
}

{{< / highlight >}}


