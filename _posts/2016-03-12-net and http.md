---
layout: post
title: nodejs 中net 和 http模块
categories: [blog ]
tags: [nodeJS ]
description: net and http
---

- 首先： http 模块是构建在 net 模块之上的
- 实际上，http中收发的数据还是通过 net 模块中的 Socket 收发数据的
- http 会将收发的数据按照 HTTP 协议自动帮你解析和包装
  + 例如 http 模块自动将请求报文解析出来，然后挂载给了 req 请求对象
  + 你可以通过 req 请求对象去拿到你想要的信息
- 为什么既有 net 又有 http 呢？
  + http 只是一个基于 net 之上的一个模块，该模块遵循的 http 协议
  + 会对收发的数据进行 协议格式解析和包装
  + HTTP 协议只是适用于B/S模型
- 有的业务功能使用的是别的协议
    + 例如 一些智能终端，就用的是别的协议，而不是 HTTP
  + 但是他们都是基于最基本的 Socket 网络编程模型而构建的


```js
/**
 * 此案例只是为了解释浏览器和服务器的交互本质
 * 本质上就是浏览器通过 Socket 和 服务器 Socket 进行通信
 * 双方都通过 HTTP 协议进行交流
 * net 模块就是传输层的一个模块，只是为了纯粹的收发数据
 * http 模块构建与 net 模块之上，只不过对于收发的数据会进行解析和包装
 * 所有的BS模型都是使用的 HTTP 协议进行数据的解析和包装
 * 关于 HTTP 协议，有时间多参考：《图解 HTTP》，图文并茂，一本足以
 */

const net = require('net')

net
  .createServer()
  .on('connection', socket => {
    socket.on('data', data => {
      // console.log(data.toString())

      // 自己动手解析请求报文
      // 1. 按照空行将请求体和请求报文分割
      // 2. 按照换行将请求头分割
      //    数组中第 0 项就是请求首行
      //    其它所有项都是请求首部字段

      console.log(socket.remoteAddress, socket.remotePort)

      data = data.toString()
      const requestContext = data.split('\r\n\r\n')

      const requestHead = requestContext[0]
      const requestBody = requestContext[1]

      const req = {}

      // url method httpVersion
      // headers
      // { key: value, key:value } 给在给 req 对象的 headers

      const requestHeadFirst = requestHead.split('\r\n')[0].split(' ')

      req.method = requestHeadFirst[0]
      req.url = requestHeadFirst[1]
      req.httpVersion = requestHeadFirst[2]
      req.headers = {}

      const requestHeadFileds = requestHead.split('\r\n').slice(1)

      for (let item of requestHeadFileds) {
        const tmp = item.split(': ')
        req.headers[tmp[0]] = tmp[1]
      }

      // console.log(req)


      // net 模块中发送数据只是纯粹的发送你传入的字符串
      socket.write(`
HTTP/1.1 200 OK
Server: Itcast
Connection: keep-alive
foo: bar
Content-Type: text/html; charset=utf-8

<h1>hello world</h1>`)
      socket.end()
    })
  })
  .listen(3000, () => {
    console.log('server is running at port 3000.')
  })

```
