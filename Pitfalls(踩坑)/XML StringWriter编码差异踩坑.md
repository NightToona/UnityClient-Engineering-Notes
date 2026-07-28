# 编码踩坑：UTF-8 / UTF-16

#Pitfalls  

## 场景


在 HierarchyTool 中，通过 `XmlSerializer` 将 Hierarchy 数据序列化为 XML 字符串，并通过 TCP 发送给客户端查看。

发送时使用UTF-8进行编码转换：
```csharp
Encoding.UTF8.GetBytes(message);
```

但是发现生成的 XML 声明为UFT-16：
```xml
<?xml version="1.0" encoding="utf-16"?>
```

导致 XML 声明编码与实际传输编码不一致。

---
## 现象

Sokit 客户端接收到的数据中，编码被声明为UTF-16：
```xml
<?xml version="1.0" encoding="utf-16"?>
```

但实际 TCP 发送使用：
```cs
Encoding.UTF8
```

即：
```
声明：UTF-16

实际：UTF-8
```

当前数据量较小时不会明显报错，但存在编码解析异常风险，或是接收后编码转换出现异常。

---
## 原因

`XmlSerializer` 本身不会固定编码，而是根据输出目标决定。

当前使用：
```cs
StringWriter writer = new StringWriter();
```

而 `StringWriter` 默认设置编码为：
```text
UTF-16
```

因此生成 XML 时自动写入：
```xml
encoding="utf-16"
```

但是后续 TCP 发送时又转换为 UTF-8：
```cs
Encoding.UTF8.GetBytes(xmlData);
```

导致两者不一致。

---
## 怎么定位

1. 查看生成的 XML 声明：
```xml
<?xml version="1.0" encoding="utf-16"?>
```

2. 检查实际发送编码：
```cs
Encoding.UTF8.GetBytes()
```

3. 对比 XML 声明编码和实际 byte 编码方式，若不相同则需要修改。

---
## 正确做法

**保持 ==XML声明编码 = 实际传输编码==。**

例如统一使用 UTF-8：

覆写`StringWriter`属性：
```cs
public class Utf8StringWriter : StringWriter
{
    public override Encoding Encoding => Encoding.UTF8; // 含义请查看“属性”知识卡
}
```

替代默认：
```cs
StringWriter
```

使 XML 输出：
```xml
<?xml version="1.0" encoding="utf-8"?>
```

并继续使用进行编码转换：
```cs
Encoding.UTF8.GetBytes()
```
---
## 知识点

- XML 声明中的 `encoding` 表示解析数据时使用的编码方式。
- `StringWriter` 默认使用 UTF-16。
- `XmlSerializer` 会根据 Writer 的编码信息生成 XML 声明。
- 实际网络传输编码由 `Encoding.GetBytes()` 决定。
- 数据声明编码与实际编码必须保持一致，否则容易导致编码转换出错。

---
## 指路

- [[Property （属性）]]
- [[XML 序列化特性]]