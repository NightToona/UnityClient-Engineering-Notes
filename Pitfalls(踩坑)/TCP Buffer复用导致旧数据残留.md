# TCP Buffer复用导致旧数据残留

#Pitfalls  

## 场景

在实现 HierarchyTool 的 TCP 通信测试是否正常时，
通过 `NetworkStream.ReadAsync()` 接收客户端发送的数据。

接收逻辑：
```cs
byte[] buffer = new byte[1024];

int length = await Stream.ReadAsync(buffer);

string message = Encoding.UTF8.GetString(buffer);
````

测试客户端使用 Sokit 进行消息发送。

---
## 现象

第一次发送：222
正常收到：222

第二次发送：1
但是服务端收到：122

出现了上一次发送数据残留的问题。

---
## 原因

`ReadAsync()` 返回的数据长度是正确的。

问题在于：

TCP读取写入使用的 `byte[]` 是重复利用的缓存空间。

**当新的数据长度小于上一次数据时，旧数据仍然保留在数组中。**

如果后续转换字符串时没有使用本次读取长度：

```
Encoding.UTF8.GetString(buffer)
```

就会把无效区域一起转换。

---
## 怎么定位

检查：

1. `ReadAsync()` 返回的 length：

```
Debug.Log(length);
```

确认读取长度是否正确。

2. 检查字符串转换：

```
Encoding.UTF8.GetString(buffer)
```

是否忽略了 length。

---
## 正确做法

##### 1. 每次使用前清理`byte[]`

使用`Array.Clear()`对`buffer`进行清空。
防止前后数据交叉污染。

##### 2. 使用读取长度限制有效区域

使用读取范围限制，防止转换读取到无效区部分：
`string message = System.Text.Encoding.UTF8.GetString( buffer, 0, length );`

其中：
- buffer：缓存数组
- 0：读取开始位置
- length：本次有效数据长度

---
## 收获

TCP中的 `byte[]` 只是临时存储空间。

真正有效的数据并不是整个数组，而是：

> ReadAsync返回的实际读取长度。

以后处理流数据时，需要始终关注：
- 数据来源；
- 数据长度；
- 数据边界。

以及养成良好的局部变量重复使用前要还原状态。

---
## 指路

- [[TCP - 深入1 生命周期]]