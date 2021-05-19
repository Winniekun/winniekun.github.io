---
title: Cookie和Session
date: 2020-07-06 09:36:34
tags: http
categories: 网络
thumbnail:
---

## 前言

<!--more-->

Session和Cookie的作用是为了保持访问用户与后端服务器的交互状态

## Cookie

Cookie的作用通俗来讲就是当一个用户通过HTTP请求访问服务器的时候, 服务器将**一些Key/Value的键值对**返回给客户端, 并且还可以给这些数据添加一些限制条件: 存活时间, 生效范围等, 在符合限制条件的情况下, 该用户再次发起请求, 数据将会被完整的返回给服务器.

```java
@RequestMapping(value = "/cookie/set", method = RequestMethod.GET)
@ResponseBody
public String setCookie(HttpServletResponse response){
    // 生成cookie
    Cookie cookie = new Cookie("wkk", CommunityUtil.generateUUID());
    // 设置生效范围
    cookie.setPath("/community/alpha");
    // 设置cookie的生效时间（默认存在内存中， 设置时间之后会存在硬盘中）
    cookie.setMaxAge(10);
    // 发送给客户端
    response.addCookie(cookie);
    return "set cookie";
}
```

![Cookie](https://i.loli.net/2020/07/06/wKDn9qhSYXgevE2.png)





W3C在设计Cookie的时候, 实际上考虑的是为了**记录用户在一段时间的web应用的行为路径.** 因为HTTP是无状态协议, 当用户访问完一次请求结束之后, 后端服务器无法知道下一次来访的是否还是上次访问的用户.   

使用Cookies带来的优势:

1. 短时间内, 如果与用户相关的数据频繁被访问, 可以为该数据添加缓存, 提高数据的访问性能

若是同一客户端发送的请求, 每次发出的请求都会包含第一次访问服务器之后, 服务器设置的信息, 这样服务器就可以根据Cookie来划分用户.

## Session

虽然Cookie的出现可以让服务器识别每个客户的请求, 但是每次客户的访问都必须返回这些Cookie, 如果Cookie很多, 无疑是增加了客户端和服务端之间数据的传输量, 而Session的存在就是为了解决这个问题.

![Session](https://i.loli.net/2020/07/06/WLICDZ4cBMKu6k7.png)

同一个客户端每次和服务端交互的时候, 不需要每次都传回所有的Cookie值, 而只要传回一个ID, 这个ID是客户端第一次访问服务器时生成的, 而且每个客户端是唯一的, 这样每个客户端就有了唯一的ID, 客户端只要返回ID即可, 这个ID就是名为**JSSIONID**的一个Cookie



## 安全问题

虽然Cookie和Session都可以跟踪客户端的访问记录,  但是工作方式不同, Cookie是通过把所有要保存的数据通过HTTP的头部从客户端传递到服务度端, 然后又从服务端传到客户端, 所有的数据都存储到客户端的浏览器中, 这些Cookie数据可以直接访问到, 设置可以修改, 所以安全性受到很大的挑战

相比而言, Session的安全性高很多, 因为Session的数据存储在服务器里, 只是通过Cookie传递一个SessionID而已, 所以Session更适合存储用户隐私和重要的数据



