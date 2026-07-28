# EditorWindow

#CSharp 

## 是什么

`using` 是 C# 中用于简化代码书写和管理资源的关键字。

根据使用位置不同，主要有三种用途：

1. 引入`namespace`(命名空间)
2. 创建类型别名
3. 自动释放实现 `IDisposable` 的资源

---
## 用途一：引入命名空间（Namespace Import）

**作用**：引入命名空间，使代码可以直接访问其中的类型，避免重复书写完整名称。

**例如**：`using UnityEditor; using System;`等

**本质**：它只是告诉编译器，当前文件允许直接使用该命名空间中的类型。

---
## 用途二：类型别名

**作用**：为命名空间或类型创建别名，解决名称冲突或提高可读性。

**例如**：
`UnityEngine.Color` 和 `System.Drawing.Color` 两个命名空间的类型相同，会产生歧义。
**解决方法**：
`using UnityColor = UnityEngine.Color; using SysColor = System.Drawing.Color`

**本质**：通过重新给命名空间类型重命名，避免不同空间中同名类型冲突。

---
## 🌟用途三：using Statement（资源管理）

**作用**：自动释放实现 `IDisposable` 接口的对象资源。常见资源有：文件流FileStream、数据库连接、网络连接、内存流Stream。

HierarchyTool中，在XML序列化时就有使用到。

#### 为什么需要

某些对象会占用系统资源，如果不释放，可能会导致：
- 文件被占用
- 系统资源无法回收

例如：`FileStream stream = File.Create("data.txt");`

打开文件后，程序通过FileStream获得文件句柄。不使用`using`会导致无法释放资源。

#### using 等价于 try-finally

```cs
①
using(FileStream stream = File.Create("data.txt"))
{
    WriteData(stream);
}

②
FileStream stream = File.Create("data.txt");
try
{
    WriteData(stream);
}
finally
{
    stream.Dispose();
}

```

此处的`Dispose()`专用于显式释放**非管理和管理资源**。而`using`则是当离开代码块时自动隐式执行。

---
## 我在哪个项目里用过

#HierarchyTool 

---
## 三种 using 对比

|类型|作用|示例|
|---|---|---|
|命名空间导入|简化类型引用|`using System.IO;`|
|类型别名|解决冲突|`using UColor = UnityEngine.Color;`|
|资源管理|自动释放资源|`using(Stream s)`|

---
## 相关知识

