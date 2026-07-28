# TCPServer 单次连接导致无法重新建立通信

#Pitfalls  

## 场景

在实现 HierarchyTool 的 TCP Server 时，
第一次尝试实现客户端连接测试。

最开始想的逻辑：

```cs
Client = await Listener.AcceptTcpClientAsync();

Stream = Client.GetStream();

await ReceiveLoop();
````

**目标**：
> 等待客户端连接后进行通信，客户端关闭后释放资源。

---
## 现象

第一次启动客户端，正常连接。

```
Server:
Client connected

Client:
通信正常
```

但是客户端关闭后，再次启动客户端
出现：
- 可以连接；
- 但是无法正常继续通信。
- Unity 控制台报错提示此电脑已终止连接

同时对于服务端循环的位置产生疑惑：
```
while(true)
{
}
```

应该放在哪里？怎么用？

---
## 原因

TCP Server并不是一次连接任务。服务端需要持续提供：

> 等待连接 → 通信 → 释放 → 等待下一次连接

的循环服务。

第一次代码：
```
Client = await Listener.AcceptTcpClientAsync();
```

只会等待一次客户端连接。

当Client连接、ReceiveLoop结束、资源释放之后：
程序没有重新回到：

```
AcceptTcpClientAsync()
```

因此无法继续等待新的客户端。

---
## 怎么定位

检查 TCP Server 生命周期：

```
Server启动

↓

监听端口

↓

等待客户端连接

↓

建立通信

↓

ReceiveLoop

↓

客户端断开

↓

释放资源

↓

重新等待连接
```

如果断开后没有重新回到：

```
等待客户端连接
```

说明服务循环缺失。

---
## 正确做法

通过外层循环维护服务端生命周期：
```
while(true)
{
    Client = await Listener.AcceptTcpClientAsync();
    Stream = Client.GetStream();
    await ReceiveLoop();
    Stream.Close();
    Client.Close();
}
```

整体结构：
```
Accept
↓
Receive
↓
Close
↓
Accept
```

其中：
- Accept负责等待新的客户端；
- Receive负责当前连接通信；
- Close负责释放当前资源。
---
## 关于退出检测

一开始误认为：
> 需要不断循环检测客户端是否还存在。实际上异步IO不需要主动轮询。

例如：
```
await Stream.ReadAsync(buffer);
```

本身就是等待事件。

当：
- 收到数据；
- 客户端正常关闭；
- 连接异常；
都会恢复执行。

其中：
```
length == 0
```

表示：
> 客户端正常关闭连接。

---
## 收获

TCP Server中的 `while(true)` 并不是：
> 不断检测有没有客户端。

而是：
> 保持服务持续运行，让服务在一次事件结束后继续等待下一次事件。

异步等待替代了传统轮询。

实际结构：
```
监听循环
↓

连接循环
↓

通信循环
```

不同循环负责不同生命周期。

---
## 指路

- [[TCP - 简述]]
- [[TCP - 深入1 生命周期]]