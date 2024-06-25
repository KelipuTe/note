---
draft: false
title: "WebSocket 协议"
summary: "WebSocket 协议；"
toc: true

categories:
  - 协议

tags:
  - 计算机
  - 协议

date: 2024-04-28 08:00:00 +0800
---

## 反向链接

[HTTP](/计算机/协议/HTTP)；

## 正文

### WebSocket 协议

在线文档：

- https://www.rfc-editor.org/rfc/rfc6455
- https://datatracker.ietf.org/doc/rfc6455

### 建立连接

```
1.3.  Opening Handshake

   The opening handshake is intended to be compatible with HTTP-based
   server-side software and intermediaries, so that a single port can be
   used by both HTTP clients talking to that server and WebSocket
   clients talking to that server.  To this end, the WebSocket client's
   handshake is an HTTP Upgrade request:

        GET /chat HTTP/1.1
        Upgrade: websocket
        Connection: Upgrade
        Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
        Sec-WebSocket-Version: 13

   To prove that the handshake was received, the server has to take two
   pieces of information and combine them to form a response.  The first
   piece of information comes from the |Sec-WebSocket-Key| header field
   in the client handshake:

        Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==

   For this header field, the server has to take the value (as present
   in the header field, e.g., the base64-encoded [RFC4648] version minus
   any leading and trailing whitespace) and concatenate this with the
   Globally Unique Identifier (GUID, [RFC4122]) "258EAFA5-E914-47DA-
   95CA-C5AB0DC85B11" in string form, which is unlikely to be used by
   network endpoints that do not understand the WebSocket Protocol.  A
   SHA-1 hash (160 bits) [FIPS.180-3], base64-encoded (see Section 4 of
   [RFC4648]), of this concatenation is then returned in the server's
   handshake.

   Concretely, if as in the example above, the |Sec-WebSocket-Key|
   header field had the value "dGhlIHNhbXBsZSBub25jZQ==", the server
   would concatenate the string "258EAFA5-E914-47DA-95CA-C5AB0DC85B11"
   to form the string "dGhlIHNhbXBsZSBub25jZQ==258EAFA5-E914-47DA-95CA-
   C5AB0DC85B11".  The server would then take the SHA-1 hash of this,
   giving the value 0xb3 0x7a 0x4f 0x2c 0xc0 0x62 0x4f 0x16 0x90 0xf6
   0x46 0x06 0xcf 0x38 0x59 0x45 0xb2 0xbe 0xc4 0xea.  This value is
   then base64-encoded (see Section 4 of [RFC4648]), to give the value
   "s3pPLMBiTxaQ9kYGzzhZRbK+xOo=".  This value would then be echoed in
   the |Sec-WebSocket-Accept| header field.

   The handshake from the server is much simpler than the client
   handshake.  The first line is an HTTP Status-Line, with the status
   code 101:

        HTTP/1.1 101 Switching Protocols

   Any status code other than 101 indicates that the WebSocket handshake
   has not completed and that the semantics of HTTP still apply.  The
   headers follow the status code.

   The |Connection| and |Upgrade| header fields complete the HTTP
   Upgrade.  The |Sec-WebSocket-Accept| header field indicates whether
   the server is willing to accept the connection.  If present, this
   header field must include a hash of the client's nonce sent in
   |Sec-WebSocket-Key| along with a predefined GUID.  Any other value
   must not be interpreted as an acceptance of the connection by the
   server.

        HTTP/1.1 101 Switching Protocols
        Upgrade: websocket
        Connection: Upgrade
        Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=

   These fields are checked by the WebSocket client for scripted pages.
   If the |Sec-WebSocket-Accept| value does not match the expected
   value, if the header field is missing, or if the HTTP status code is
   not 101, the connection will not be established, and WebSocket frames
   will not be sent.
```

WebSocket 连接的建立中有握手的过程，握手是基于 HTTP 进行的。

服务端和客户端启动的时候，都处于没有握手的状态。客户端发起握手请求。

握手请求的报文是固定的格式，这里只用到了必不可少的几个字段：

```
GET /chat HTTP/1.1\r\n
Connection: Upgrade\r\n
Upgrade: websocket\r\n
Sec-WebSocket-Version: 13\r\n
Sec-WebSocket-Key: {客户端生成}\r\n
Content-Length: 0\r\n
\r\n
```

生成 Sec-WebSocket-Key 的步骤：
客户端生成一个随机字符串，然后 MD5 计算摘要，再 Base64 编码。

服务端在收到握手请求后，需要返回握手响应，这里只用到了必不可少的几个字段。

```
HTTP/1.1 101 Switching Protocols\r\n
Connection: Upgrade\r\n
Upgrade: websocket\r\n
Sec-WebSocket-Version: 13\r\n
Sec-WebSocket-Accept: {服务端生成}\r\n
Content-Length: 0\r\n
\r\n
```

生成 Sec-WebSocket-Accept 的步骤：
服务端拿到 Sec-WebSocket-Key 后和 "258EAFA5-E914-47DA-95CA-C5AB0DC85B11" 拼接，然后 SHA1 计算摘要，再 Base64 编码。

服务端返回握手响应后，就可以把自己的握手状态改成已握手了。

客户端在收到握手响应后，校验一下服务端生成的 Sec-WebSocket-Accept 对不对。
如果 Sec-WebSocket-Accept 没问题，就可以把自己的握手状态改成已握手了。

这样 WebSocket 连接就建立完成了。后续报文的解析方式从 HTTP 换到 WebSocket。

### 报文结构

```
5.2.  Base Framing Protocol

      0                   1                   2                   3
      0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
     +-+-+-+-+-------+-+-------------+-------------------------------+
     |F|R|R|R| opcode|M| Payload len |    Extended payload length    |
     |I|S|S|S|  (4)  |A|     (7)     |             (16/64)           |
     |N|V|V|V|       |S|             |   (if payload len==126/127)   |
     | |1|2|3|       |K|             |                               |
     +-+-+-+-+-------+-+-------------+ - - - - - - - - - - - - - - - +
     |     Extended payload length continued, if payload len == 127  |
     + - - - - - - - - - - - - - - - +-------------------------------+
     |                               |Masking-key, if MASK set to 1  |
     +-------------------------------+-------------------------------+
     | Masking-key (continued)       |          Payload Data         |
     +-------------------------------- - - - - - - - - - - - - - - - +
     :                     Payload Data continued ...                :
     + - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - +
     |                     Payload Data continued ...                |
     +---------------------------------------------------------------+

FIN:  1 bit

      Indicates that this is the final fragment in a message.  The first
      fragment MAY also be the final fragment.

Opcode:  4 bits

      Defines the interpretation of the "Payload data".  If an unknown
      opcode is received, the receiving endpoint MUST _Fail the
      WebSocket Connection_.  The following values are defined.

      *  %x0 denotes a continuation frame
      *  %x1 denotes a text frame
      *  %x2 denotes a binary frame
      *  %x3-7 are reserved for further non-control frames
      *  %x8 denotes a connection close
      *  %x9 denotes a ping
      *  %xA denotes a pong
      *  %xB-F are reserved for further control frames

Mask:  1 bit

      Defines whether the "Payload data" is masked.  If set to 1, a
      masking key is present in masking-key, and this is used to unmask
      the "Payload data" as per Section 5.3.  All frames sent from
      client to server have this bit set to 1.
      
Payload length:  7 bits, 7+16 bits, or 7+64 bits

      The length of the "Payload data", in bytes: if 0-125, that is the
      payload length.  If 126, the following 2 bytes interpreted as a
      16-bit unsigned integer are the payload length.  If 127, the
      following 8 bytes interpreted as a 64-bit unsigned integer (the
      most significant bit MUST be 0) are the payload length.  Multibyte
      length quantities are expressed in network byte order.  Note that
      in all cases, the minimal number of bytes MUST be used to encode
      the length, for example, the length of a 124-byte-long string
      can't be encoded as the sequence 126, 0, 124.  The payload length
      is the length of the "Extension data" + the length of the
      "Application data".  The length of the "Extension data" may be
      zero, in which case the payload length is the length of the
      "Application data".

Masking-key:  0 or 4 bytes

      All frames sent from the client to the server are masked by a
      32-bit value that is contained within the frame.  This field is
      present if the mask bit is set to 1 and is absent if the mask bit
      is set to 0.  See Section 5.3 for further information on client-
      to-server masking.
```

- FIN，1 bit，是不是消息的最后一个分片。0=不是；1=是；
- OPCODE，4 bit，操作码（报文类型）。
    - 0x1=文本帧；0x2=二进制帧；0x8=连接关闭；0x9=ping；0xA=pong；
- MASK，1 bit，Payload Data 有没有用掩码。0=没有；1=有；
- Payload len，7 bit，Payload Data 的长度。取值范围是 0~2^7-1，也就是 0~127。
- Extended payload length，0 bit、16 bit、64 bit。扩展的 Payload len。
    - 如果 Payload len<126，Extended payload length 是 0 bit。
    - 如果 Payload len=126，Extended payload length 是 16 bit。
    - 如果 Payload len=127，Extended payload length 是 64 bit。
- Masking-key，4 byte，4 个掩码。
    - 如果 MASK=0，则没有 Masking-key，占 0 byte。
    - 如果 MASK=1，则有 Masking-key，占 4 byte。
- Payload Data，有效载荷。

### 掩码

```
5.3.  Client-to-Server Masking

   The masking key is contained completely within the frame, as defined
   in Section 5.2 as frame-masking-key.  It is used to mask the "Payload
   data" defined in the same section as frame-payload-data, which
   includes "Extension data" and "Application data".

   The masking key is a 32-bit value chosen at random by the client.
   When preparing a masked frame, the client MUST pick a fresh masking
   key from the set of allowed 32-bit values.  The masking key needs to
   be unpredictable; thus, the masking key MUST be derived from a strong
   source of entropy, and the masking key for a given frame MUST NOT
   make it simple for a server/proxy to predict the masking key for a
   subsequent frame.  The unpredictability of the masking key is
   essential to prevent authors of malicious applications from selecting
   the bytes that appear on the wire.  RFC 4086 [RFC4086] discusses what
   entails a suitable source of entropy for security-sensitive
   applications.

   The masking does not affect the length of the "Payload data".  To
   convert masked data into unmasked data, or vice versa, the following
   algorithm is applied.  The same algorithm applies regardless of the
   direction of the translation, e.g., the same steps are applied to
   mask the data as to unmask the data.

   Octet i of the transformed data ("transformed-octet-i") is the XOR of
   octet i of the original data ("original-octet-i") with octet at index
   i modulo 4 of the masking key ("masking-key-octet-j"):

     j                   = i MOD 4
     transformed-octet-i = original-octet-i XOR masking-key-octet-j

   The payload length, indicated in the framing as frame-payload-length,
   does NOT include the length of the masking key.  It is the length of
   the "Payload data", e.g., the number of bytes following the masking
   key.
```

客户端发消息给服务端的时候需要掩码，服务端发消息给客户端的时候不需要掩码。

有掩码的时候，Masking-key 占 4 个字节，Payload len 后面是 Masking-key，然后才是 Payload Data。
没有掩码的时候，Masking-key 也没有，占 0 个字节. Payload len 后面直接是 Payload Data。

解析的时候，Payload Data 的第 i 个字节和 Masking-key 的第 i MOD 4 个字节异或。
比如，Payload Data 的第 3、4、5、6 个字节分别和 Masking-key 的第 3、4、1、2 个字节异或。

### 心跳机制

心跳机制的代码可以放在很多地方，没有定死说一定要写在哪里。
常见的有两种：一种是协议内部处理；一种是交给上层应用处理；

协议内部处理。这种方式，将心跳机制抽象出来，可以简化上层应用的代码。
协议内部统一管理心跳包的发送逻辑，避免上层应用各自重复实现心跳机制。
上层应用无需关注心跳包的发送逻辑，这样可以提高代码的可读性和可维护性。

交给上层应用处理。这种方式，灵活性更高，可控性更强。

上层应用可以根据具体业务需求自行定义心跳包的格式和发送频率。
上层应用可以更精细地控制心跳包的发送逻辑。比如，网络繁忙时降低发送频率。
