# TCP - 简述

#CSharp

## 是什么

TCP（Transmission Control Protocol）是一种面向连接、可靠传输的网络协议。

分为 C/S 端，客户端（Client）需要连接到服务端（Server）才能进行收发数据。

TCP 本身只负责传输 `Byte[]`，字符串通常会转换成 UTF-8 字节数组后发送。

---
## 核心底层流程

在底层代码实现中，Tcp 需要以下流程才能成功使用：

```txt
Server:
Socket() -> Bind() -> Listen() -> Accept() -> Receive() -> Send() -> Close()
|--— TcpListener() -> .Start() -> .AcceptTcpClient() ---—|

Client:
Socket() -> Connect() -> Send() -> Receive() -> Close()

```

---
## 核心实现代码与API

```csharp
// 服务端实现
TcpListener listener = new TcpListener(IPAddress.Any, port);
listener.Start(); //API：开始监听C端连接
TcpClient client = listener.AcceptTcpClient(); //API：等待C端连接
NetworkStream stream = client.GetStream();
while(true)
{
	stream.Read();  //API：接收数据
	stream.Write(); //API：写入发送缓冲区
	stream.Flush(); //API：刷新缓冲区（一般Write会自动发送，无需手动）
}


// 客户端实现
TcpClient client = new TcpClient();
client.Connect(ip, port); //API：连接S端
NetworkStream stream = client.GetStream(); //API：获取网络流
while(true)
{
	// Send
	// Receive
}
```

各类API见上方注释。

---
## 我在哪个项目里用过

#HierarchyTool 

- Hierarchy Tool 作品

---
## 理解&容易踩坑

**①**
TcpListener 用于S端创建连接，TcpClient 用于C端负责连接，
C/S端均通过 NetworkStream 来读写数据、手法数据。

**②**⭐⭐  [[静态变量初始化生命周期问题]]
TcpListener 在创建时，`TcpListener(IPAddress.Any, port)`会立即绑定对应端口。  
后续修改变量 `port`，不会影响已经创建完成的 Listener。
需要切换端口只有 `Stop() -> new port -> Start()` 这个方法。

---
## 相关知识

- [[TCP - 深入1 生命周期]]
- [[TCP - 深入2 状态控制机制]]
- [[TCP - 深入3 包体结构]]