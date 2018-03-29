---
title: "Gin 源码学习（Gin JWT Middleware）"
subtitle: Gin JWT Middleware
tags: [ "Go", "Gin", "Gin Middleware", "JWT" ]
description: 记录一些值得学习的Go开源项目
categories: [ "Go", "Gin", "JWT" ]
isCJKLanguage: true
date: 2018-03-29T08:40:52+08:00
draft: false
---

记录一个Gin框架的JWT中间件，项目地址：**https://github.com/appleboy/gin-jwt** 

这个项目是利用 [**dgrijalva/jwt-go**](https://github.com/dgrijalva/jwt-go)  项目来实现`JSON Web Tokens` ，简称`JWT`。什么是`JWT`，可以参看这个文档 [JSON Web Token (JWT)](http://self-issued.info/docs/draft-ietf-oauth-json-web-token.html) ，也可以百度，这有个中文介绍：[什么是 JWT -- JSON WEB TOKEN](https://www.jianshu.com/p/576dbf44b2ae) 。

<!--more-->

JWT的原理和实现就不多少了，看看在Gin中怎么用这个Middleware吧。这个中间件就一个源文件：`auth_jwt.go` ，核心是这个结构，我保留几个重要的成员，节省阅读篇幅：

```go
type GinJWTMiddleware struct {
	...
	// 算法.
	SigningAlgorithm string
	// 密钥.
	Key []byte
	// 有效期.
	Timeout time.Duration
	// 刷新Token有效期.
	MaxRefresh time.Duration
	// 认证回调函数.
	Authenticator func(userID string, password string, c *gin.Context) (string, bool)
	// 认证后处理.
	Authorizator func(userID string, c *gin.Context) bool
	...
	// 认证失败函数.
	Unauthorized func(*gin.Context, int, string)
	...
}
```

库中已经写好了`LoginHandler`和`RefreshHandler`函数，可以直接拿来使用，`LoginHandler`在用户请求时会调用自定义的认证回调函数，成功后返回一个 `token` ，`RefreshHandler`会验证请求头部的`token`并返回一个新`token`。

我写了一个测试程序演示如何使用，首先初始化`jwt.GinJWTMiddleware`结构，写好你自己的认证处理函数，如下所示：

```go
func init() {
	// the jwt middleware
	authMiddleware = &jwt.GinJWTMiddleware{
		Realm:         "Realmname",
		Key:           []byte("Secretkey"),
		Timeout:       time.Hour * 12,
		MaxRefresh:    time.Hour * 24,
		Authenticator: jwtAuthFunc,
		Unauthorized:  jwtUnAuthFunc,
        // 其他默认
	}
}

func jwtAuthFunc(userID string, password string, c *gin.Context) (string, bool) {
	// TODO register then login by username/password
	if userID == "admin" && password == "xxxxx" {
		return userID, true
	}
	return userID, false
}

func jwtUnAuthFunc(c *gin.Context, code int, message string) {
	c.JSON(code, gin.H{
		"code":    code,
		"message": message,
	})
}
```

然后，在建立Gin的路由时使用这个中间件，可以指定某些入口使用`JWT`认证，例如：

```go
func routerEngine() *gin.Engine {
	r := gin.New()
	...
	r.POST("/login", authMiddleware.LoginHandler) //login不验JWT
	r.POST("/other_uri_donot_need_authz", otherHandler1) //不验JWT的
	...
	auth := r.Group("/", authMiddleware.MiddlewareFunc()) //这个组全认证
	auth.POST("/refresh_token", authMiddleware.RefreshHandler) //刷新token
	auth.POST("/other_uri_need_authz", otherHandler2) //验JWT的
	...
	return r
}
```

需要验证 `JWT` 的请求，需要在请求头部 `HTTP Header` 中携带`Authorization:Bearer token_string` 。

这样，一个实现JWT认证的服务就基本实现了，库中还提供了例如生成token的方法，当然如果还不够用，可以去 [**dgrijalva/jwt-go**](https://github.com/dgrijalva/jwt-go)  项目中获得更多的函数方法。