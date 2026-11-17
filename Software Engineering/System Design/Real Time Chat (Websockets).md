
## **WebSocket — Why, How and How It Works Internally**

## **1. Why WebSockets?**

TCP provides a reliable, ordered byte stream.

HTTP uses TCP to implement request-response communication.

In traditional HTTP communication:

- Client sends a request.
- Server processes the request and sends a response.
- The HTTP exchange finishes.
- The underlying TCP connection may be closed or reused, depending on the HTTP version and connection management.

For a real-time chat application, repeatedly establishing connections, polling for messages, and transmitting HTTP headers for frequent small messages can be inefficient.

Therefore, we use WebSockets.

WebSocket uses an existing TCP connection to enable **full-duplex communication**.

This means:

- Client can send messages independently.
- Server can send messages independently.
- Both can communicate asynchronously.
- No new HTTP request is required for every message.

`NOTE: Important thing to notice is that eventhough HTTP/1.1 or HTTP/2 supports long polling or Keep-Alive schemes that still doesnt override the underlying rule of HTTP - that is u send response for a request so every response needs a request , SSE are long list of responses for the same request, HTTP/2 is complicated but still in Websockets it enables "FULL-DUPLEX" Communication that means that the server can also act as an independent entity no request is needed to invoke the server`
## **2. How does WebSocket achieve this?**

Through the HTTP Upgrade handshake.

**Step 1 — TCP connection**

The client establishes a TCP connection with the server through the TCP three-way handshake.

**Step 2 — HTTP Upgrade request**

The client sends an HTTP request asking to upgrade the connection to WebSocket.
```HTTP
GET /chat HTTP/1.1 
Host: example.com 
Upgrade: websocket 
Connection: Upgrade 
Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ== 
Sec-WebSocket-Version: 13 
Sec-WebSocket-Protocol: chat
```

**Step 3 — Server accepts the upgrade**

The server validates the request and responds:
```HTTP
HTTP/1.1 101 Switching Protocols 
Upgrade: websocket 
Connection: Upgrade 
Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo= 
Sec-WebSocket-Protocol: chat
```

**Step 4 — WebSocket connection established**

After the successful handshake:

- The existing TCP connection remains open.
- No new TCP connection is created.
- Both sides switch to WebSocket protocol handling.
- Both sides exchange WebSocket frames over the same TCP connection.


