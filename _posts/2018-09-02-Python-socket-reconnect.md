---

layout: post

title:  "Python socket断开重连总结"

categories: Python

tags:  socket.error exception reconnect

---
#### **socket异常**
可以用socket.error来捕获，通常我们捕获异常是利用send和recv函数来捕获，那么socket某一方的断开，也是通过send和recv来获知的，所以我们就可以从这里着手来处理。
## send函数异常
可以这么使用：
```python
try:
    sock.send(1024)
except socket.error:
    # reconnect
```
## recv函数异常
recv函数情况比较复杂，它处理异常分为两种情况：
1. 一种是对端关闭连接，此时对端socket对象销毁，会调用close函数，正常执行4次挥手关闭连接，此时，recv函数不会阻塞，它会返回空的字符串，可以这么处理：
```python
recv_str = sock.recv(1024)
if len(recv_str) == 0:
    # reconnect
```
2. 另一种情况是网络断开，此时的连接会保持，那么recv会一直阻塞，python是不能够捕获这样的异常的，所以只能对socket设置timeout来捕获。
如果你的程序是交互式的，你需要阻塞等待对端发来的指令进行操作，那么就不能设置timeout了，一个解决办法是双方建立心跳机制，你可以给对端发心跳，然后接收心跳，对这个心跳加timeout判断。

**Python官方文档对socket连接断开的描述**

> **When Sockets Die**

> Probably the worst thing about using blocking sockets is what happens when the other side comes down hard (without doing a close). **Your socket is likely to hang**. TCP is a reliable protocol, and it will wait a long, long time before giving up on a connection. If you’re using threads, the entire thread is essentially dead. There’s not much you can do about it.

**值得注意的是，当你写服务端程序时，在捕获异常进行reconnect时，千万记得把当前连接关闭**