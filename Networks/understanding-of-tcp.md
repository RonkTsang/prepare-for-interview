# Understanding of TCP

TCP，传输控制协议(Transmission Control Protocol)，是 TCP/IP 模型运输层中的其中一种协议

## TCP 和 UDP 的区别

- TCP 面向连接而 UDP 是无连接的
- TCP 提供可靠交付(无差错、不丢失、不重复、并且按序到达)，UDP 使用尽最大努力交付
- TCP 连接只能是一对一而 UDP 支持一对一、一对多、多对一和多对多的交互通信
- TCP 面向字节流而 UDP 是面向报文的
- TCP 有拥塞控制，而 UDP 没有拥塞控制
- TCP 因为拥塞控制，速率降低，但 UDP 不会，没有太大时延

## TCP 三次握手

握手前，客户端和服务端都处于连接 CLOSE 状态，**客户端会主动打开连接，服务端被动打开连接**

**第一次握手**，客户端执行 CONNECT 原语，发送一个 SYN 为 1，seq 为 x 的初始序号。TCP 规定，SYN 报文段不能携带数据，但是消耗掉一个序号，这时客户端进入 SYN-SENT 状态

如果服务端要接收客户端的连接请求(第一次握手)，必须先创建一个 TCB，使服务端进程处于 LISTEN 状态，如果没有，则它发送一个设置了 RST 的应答报文，拒绝客户的连接请求

**第二次握手**，如果服务端处于 LISTEN 状态，并同意建立连接，就向客户端发送一个 SYN 和 ACK 都为 1，确认号 ack 为 x+1，seq 为 y，这时报文也不能携带数据，但同样要消耗一个序号，这时服务端进入 SYN-RCVD 状态

**第三次握手**，客户端确认 ACK = 1，seq = x + 1，ack = y + 1。ACK 报文可以携带数据，但如果不携带数据，那么则不消耗序号，那么下次再建立请求时，seq 还是 x + 1。这时，客户端进入 ESTABLISHED 状态

在服务端收到客户端的确认后，也进入 ESTABLISHED 状态

这时就可以进行数据的传送了

> 为什么是三次握手，而不是两次？

因为有可能存在 **已失效的连接请求报文段**。简单的说，就是之前有一个没有丢失，但是却长时间滞留了的报文段，在连接释放之后到达了服务端，那么服务端就会误以为客户端发出了一个全新的连接请求(第一次握手)，于是就向客户端发送确认报文段(第二次握手)，同意建立连接。如果不采用三次握手，那么连接就已经建立了，服务端就会等待客户端发送数据，可是客户端完全不知情，这样就浪费了服务端的很大资源

## TCP 四次挥手

断开前，客户端和服务端都处于 ESTABLISHED 状态，客户端 **主动关闭连接**

**第一次挥手**，客户端发送 FIN = 1，seq = u，u 等于前面已传送过的数据的最后一个字节的序号加 1。这时客户端进行 FIN-WAIT-1 状态，TCP 规定，即使不携带数据，也消耗一个序号

**第二次挥手**，服务端对客户端的连接释放报文响应，ACK = 1，ack = u + 1，还有只有的序号 seq = v，v 等于服务端前端已传送数据的最后一个字节的序号加 1。然后服务端就进入 CLOSE-WAIT 状态。此时客户端到服务端的连接已经释放，但服务端还可以给客户端发数据

客户端收到服务端的确认后，进入 FIN-WAIT-2 状态。

**第三次挥手**，当服务端没有东西给客户端发之后，服务端发送 FIN = 1，seq = w，w 为服务端向客户端已传送数据的最后一个字节的序号加 1。除此之外，还要发送上次确认的 ACK = 1 和 ack = u + 1。这时服务端进入 LAST-ACK 状态

**第四次挥手**，这次到客户端对服务端进行确认，发送 ACK = 1，ack = w + 1 和 seq = u + 1。然后进入 TIME-WAIT 状态。收到客户端确认信息后，服务端进入 CLOSE 状态。但这时候，**TCP 还没有完全释放连接**。必须经过 **时间等待计时器(TIME WAIT timer)** 设置的时间 2MSL后，客户端才进入 CLOSE 状态

MSL 建议为 2 分钟，也就是说，当客户端进行 TIME-WAIT 状态后，还要等待 4 分钟才进入 CLOSE 状态。TCP 连接才真正释放

> 为什么要等待这 2MSL 时间？

1. 为了确保客户端和服务端都正常进入 CLOSE 状态。因为有可能客户端的最后一个 ACK 报文丢失了，那么服务端就可以在这 2MSL 里重发 FIN 和 ACK。客户端收到后再重新传一次 ACK，再次启动 TIME-WAIT timer。直到双方都进入 CLOSE 状态
2. 为了确保不会有时延的请求报文去到下一个新的连接中

> 为什么第二次挥手 ACK 和 FIN 不一起发送？(为什么需要四次挥手)

因为 TCP 是全双工通信的，接收到 FIN 代表没有数据再发来，但是我却还可以继续发送数据过去