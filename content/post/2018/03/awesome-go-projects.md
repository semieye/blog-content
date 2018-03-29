---
title: Go开源项目（自己整理记录的）
subtitle: Awesome Go Projects on GitHub
tags: [ "Go", "Github" ]
description: 记录一些值得学习的Go开源项目
categories: [ "Go" ]
date: 2018-03-23T16:29:06+08:00
lastmod: 2018-03-25T17:06:26+08:00
isCJKLanguage: true
weight: 100
---

记录一些值得学习的Go开源项目，以下是部分链接，留待以后学习。`<置顶>`
<!--more-->

主要是学习其中的一些中小型项目，学习和借鉴其中蕴含的思想和技术，有些或者直接拿来使用 :-) 。中大型项目例如`Docker`、`Kubernetes`、`etcd`、`Flynn`、`nsq` 等等…...，太多太大了，有精力再说，估计没有那么多的时间和精力。

#### HTTP WEB：
- [ ] https://github.com/gin-gonic/gin
- [ ] https://github.com/labstack/echo
- [ ] https://github.com/revel/revel
- [ ] https://github.com/kataras/iris
- [ ] https://github.com/astaxie/beego
- [ ] https://github.com/mholt/caddy
- [ ] https://github.com/vaniila/hyper `这可能是个大神`

#### Gin Middlewares

- [x] https://github.com/gin-contrib
- [x] https://github.com/gin-gonic/contrib
- [x] https://github.com/aviddiviner/gin-limit
- [x] https://github.com/gin-contrib/cache
- [x] https://github.com/appleboy/gin-jwt
- [x] https://github.com/gin-contrib/cors
- [x] https://github.com/gin-contrib/sse

#### spf13 大神的：

- [ ] `安全类型转换` https://github.com/spf13/cast
- [ ] `性能统计分析` https://github.com/spf13/nitro
- [ ] `静态网站生成` https://github.com/gohugoio/hugo
- [ ] `抽象文件系统` https://github.com/spf13/afero
- [ ] `通用配置文件` https://github.com/spf13/viper
- [ ] `强大命令行库` https://github.com/spf13/cobra

#### Gorilla Web Toolkits

- [ ] https://github.com/gorilla  [官网资料](http://www.gorillatoolkit.org)
- [ ] https://github.com/gorilla/pad
- [ ] https://github.com/gorilla/mux
- [ ] https://github.com/gorilla/context
- [ ] https://github.com/gorilla/websocket
- [ ] https://github.com/gorilla/sessions
- [ ] https://github.com/gorilla/schema
- [ ] https://github.com/gorilla/handlers
- [ ] https://github.com/gorilla/csrf
- [ ] https://github.com/gorilla/reverse
- [ ] https://github.com/gorilla/rpc

#### RDBMS / ORM

- [ ] https://github.com/jinzhu/gorm [官网资料](http://gorm.io/zh_CN/docs/index.html)
- [ ] https://github.com/jinzhu/gorm/dialects/mysql
- [ ] https://github.com/go-sql-driver/mysql

#### BoltDB /  MongoDB

- [ ] https://github.com/boltdb/bolt `不再维护`
- [ ] https://github.com/coreos/bbolt `最新`
- [ ] https://github.com/asdine/storm
- [ ] https://github.com/evnix/boltdbweb
- [ ] https://github.com/tidwall/buntdb
- [ ] https://github.com/go-mgo/mgo `不再维护`
- [ ] https://github.com/globalsign/mgo `最新`

#### Web Git

- [ ] https://github.com/gogits/gogs
- [ ] https://github.com/go-gitea/gitea

#### Logger

- [ ] https://github.com/sirupsen/logrus
- [ ] https://github.com/rifflock/lfshook
- [ ] https://github.com/x-cray/logrus-prefixed-formatter
- [ ] https://github.com/ngmoco/timber
- [ ] https://github.com/alecthomas/log4go

#### UUID / GUID

- [x] https://github.com/nats-io/nuid
- [x] https://github.com/satori/go.uuid
- [x] https://github.com/nu7hatch/gouuid
- [x] https://github.com/zheng-ji/goSnowFlake

#### Cache

- [x] https://github.com/patrickmn/go-cache
- [ ] https://github.com/muesli/cache2go
- [ ] https://github.com/golang/groupcache

#### Daemon And Tools

- [ ] https://github.com/fvbock/endless 后台
- [ ] https://github.com/mattn/go-isatty 终端
- [ ] https://github.com/json-iterator/go JSON
- [ ] https://github.com/stretchr/testify/assert 测试
- [ ] https://github.com/smartystreets/goconvey 测试
- [ ] https://github.com/dustin/go-broadcast chan广播
- [x] https://github.com/benas/gossed SSE
- [x] https://github.com/alexandrevicenzi/go-sse SSE
- [ ] https://github.com/urfave/cli CLI
- [ ] https://github.com/konojunya/go-frame PRINT
- [ ] https://github.com/konojunya/gost GoGist
- [ ] https://github.com/dustin/go-rs232 RS232

#### JSON WEB Token

- [x] https://github.com/appleboy/gin-jwt
- [x] https://github.com/appleboy/go-jwt-server
- [x] https://github.com/dgrijalva/jwt-go

#### PUSH
- [ ] https://github.com/appleboy/gorush
- [ ] https://github.com/Terry-Mao/gopush-cluster

#### NSQ

- [ ] https://github.com/nsqio/nsq [协议文档](http://nsq.io/clients/tcp_protocol_spec.html)
- [ ] https://github.com/nsqio/go-nsq

#### NATs

- [ ] https://github.com/nats-io/gnatsd [协议文档](https://nats.io/documentation/internals/nats-protocol)
- [ ] https://github.com/nats-io/go-nats

#### NATs Streaming
- [ ] https://github.com/nats-io/nats-streaming-server
- [ ] https://github.com/nats-io/go-nats-streaming
