---
layout: page
title:  "WebSocket Protocol - Opening Handshake, HTTP Upgrade, Negotiations"
date:   2020-02-02 16:07:19 +0200
categories: networking
---

I've recently became interested in WebScoket protocol, so after reading a bit, I've decided to sumamrize my learnings in an article or two. I'll start by writing about the initial handshake needed for establishing a websocket connection. For more information, refer to the initial [RFC 6455](https://tools.ietf.org/html/rfc6455). I won't mention many details about WebSocket extensions or websocket frames, that'd be a topic for another article.

## Background

First of all, WebSocket was designed with a certain purpose. There are about 3 main reasons why this protocol makes sense in the RFC. To sumamrize that, the biggest advantage is that WebSocket really needs only one TCP connection and that's how two sides (peers) can exchange messages at any time after the connection becomes open. Therefore, it's reasonable to say that WebScoket is a lightweight protocol, much leaner than HTTP in terms of TCP connections as well as overhead and that it can be used for message exchange in both directions (from client to server as well as from server to client) in (near) real time.


## HTTP request

It's important to realise that WebSocket protocol doesn't stand on its own, nor it needs only lower level protocols such as TCP, IP or similar. It also needs HTTP (at least version 1.1) without which there's no websocket connection. This is not a concidence, since the whole WebSocket protocol was designed in a way it can easily fit into an existing infrastructure.

So the first request coming from a client is a standard HTTP request with just some specific headers that I'll mention and explain in a bit:

```
GET / HTTP/1.1
Host: 192.168.43.135:12345
Connection: Upgrade
Pragma: no-cache
Cache-Control: no-cache
Upgrade: websocket
Origin: file://
Sec-WebSocket-Version: 13
User-Agent: Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/52.0.2743.116 Safari/537.36
Accept-Encoding: gzip, deflate, sdch
Accept-Language: zh-CN,zh;q=0.8,en-US;q=0.6,en;q=0.4
Sec-WebSocket-Key: bKdPyn3u98cTfZJSh4TNeQ==
Sec-WebSocket-Extensions: permessage-deflate; client_max_window_bits
```

`GET` is the only HTTP method allowed when a client wants to use WebSocket protocol, HTTP version must be at least 1.1. In terms of the headers, not all of these headers are needed, but the following ones are mandatory:

```
Host
Connection
Upgrade
Sec-WebSocket-Version
Sec-WebSocket-Key
```

`Connection` has to contain value `upgrade` and `Upgrade` header has to contain value `websocket`. `Web-Socket-Version` has to be sent from a client. It can later be negotiated if a server e.g. supports only higher versions. `Sec-WebSocket-Key` is a unique random base64 encoded string. You can also see header `Sec-WebSocket-Extensions` and `Sec-WebSocket-Protocol`which are used for negotiating extensions and subprotocols over websocket protocol. I'll mention these extensions in a bit (perhaps in a different article), but it's good to know from the off that extensions and subprotocols are possible, that there might be more of them used and that they are negotiated during the opening handshake between a client and server. Also notice header `Origin`, this one is not mandatory every time, but it is mandatory if a request is sent from web browsers.

Other HTTP headers could be used (just like they are in the example above), e.g. headers related to cache, authentication etc, or even other "websocket" headers like `Sec-WebSocket-Protocol`.

## Server side

When a client's request arrives at the server, the server will check for a few things. First, it will check all those mandatory headers and correct values. Based on that, there're actually a couple of scenarios that might happen:

- server might reply with 4xx status code - this will typically happen when e.g. some of the mandatory headers are missing (then it will likely return 400 Bad Request), further authentication is needed (401 Unauthorized might be returned), or a wrong protocol version is used (426 Upgrade Required would be an appropriate status code in this example), which might be followed by a version negotiation (one or more `Sec-WebScoket-Version` headers is sent back to the client)
- server might reply with 3xx in an attempt to redirect a client somewhere else; it all depends on the client then if it decides to follow the suggested redirection
- server replies with 101 Switching Protocols - this is a successful response

So when 101 Switching Protocols happens, the response typically looks as follows:

```
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: 4EaeSCkuOGBy+rjOSJSMV+VMoC0=
WebSocket-Origin: file://
WebSocket-Location: ws://192.168.43.135:12345/
```

Important headers to notice are:

```
Upgrade
Connection
Sec-WebSocket-Accept
```

These are mandatory and the client will check for them. Let's focus a bit more on `Sec-WebSocket-Accept`. Again, it's a base64 encoded string, but it's computed a bit differently. Server takes the value of `Sec-WebSocket-Key` and concatenates it with a string literal (mentioned somewhere in the RFC), then it will use SHA1 on that and then finally use base64 on the result. A client will compute the same value and later compare it with what the server sent, obviously they have to match in order to establish a successful websocket connection.

If some optional headers are used (`Sec-WebSocket-Extensions` or `Sec-WebSocket-Protocol`), a server should choose zero or some of the values initially sent by a client. If the server sends a different value(s) (not initially mentioned by the client), such a response is considered invalid and the connection won't be established (it must fail).

Only after the client checks these headers and their values and if everything is as expected, the websocket connection is established.

### Version negotiation

During the mentioned opening handshake, a version negotiation might happen. Client always sends only one `Sec-WebSocket-Version` header, but a server might actually reply with more of them, one or more values in one such header. The following examples are therefore identical:

```
Sec-WebSocket-Version: 13, 14, 15
```

and 

```
Sec-WebSocket-Version: 13
Sec-WebSocket-Version: 14, 15
```

The client then has to decide whether it can use a different version of the protocol. If the answer is yes, a new opening handshake can be initialized by the client.

### Extension negotiation

Client might send one or more `Sec-WebSocket-Extensions` headers in an initial opening handshake. Again, the values might be split into multiple headers, therfore the following two examples are identical:

```
Sec-WebSocket-Extensions: foo, bar
```

and

```
Sec-WebSocket-Extensions: foo
Sec-WebSocket-Extensions: bar
```

If no `Sec-WebSocket-Extensions` is sent from a client, no extensions will be used.

Server then replies with none or some extensions. If the server replies with some extensions, all of them have to be in the initial list provided by the client in the request. If the server wishes to use an extension that was not mentioned by the client, the connection must fail. The header sent from the server has to be mentined only ones (so no splitting).

It's also important to mention how the extensions will be used. If a server sends back that it agrees to use e.g. two extensions: `Sec-WebSocket-Extensions: foo, bar`, then operations on the data will be bar(foo(data)).

### (Sub)Protocol negotiation

Subprotocols might be used, and header `Sec-WebSocket-Protocol` is here to allow that. Very much like with the `Sec-WebSocket-Extensions`, the header might initially be sent by a client and there might be header splitting. A server might or might not reply with such a header, if the server agrees to use some of the subprotocols mentioned by the client, it sends the header back with correct values. If a value that was not in the initial client's list of subprotocols is registered by the client, the conenction must fail.

Values in this header are ordered by preference.

### Conclusion

This is about everything I can think of off the top of my head right now. It's better to read the RFC, this is just a quick overview of the most important bits. Things to remember are the following ones: it's a protocol designed to be lighweight, used for a bidirectional message exchange between two peers, one TCP connection is opened provided an opening handshake is successful, a websocket connection is opened by upgrading a standard HTTP request-response exchange. The websocket protocol has a room for extensions and subprotocols (e.g. many opcodes are left for future use), so we might see much more in this area in the future.