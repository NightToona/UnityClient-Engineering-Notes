# XML 序列化特性（Attribute）

#CSharp 

## 是什么

**特性（Attribute）👇**

> 给类型、字段、属性附加额外元数据，让框架通过反射读取并执行特殊操作。

**XML 特性👇**

> XML 特性是用于描述 XML 序列化规则的 Attribute，通过**反射**读取这些元数据，控制对象与 XML 之间的映射关系。

---
## 为什么需要它

在 C# 中，对象默认序列化时，序列化器只能按照类本身的结构生成数据。

```cs
public class Hierarchy
{
	public string sceneName;
}

默认生成：
<Hierarcgy>
	<sceneName>SampleScene</sceneName>
</Hierarcgy>
```

但实际项目中，通常需要：控制各节点名称、控制数据层级、区分属性和子节点等。

因此需要通过 Attribute（特性） 给序列化器提供额外信息。

---
## 常用特性

##### 1.`[XmlRoot("")]`

> 作用：控制 XML 根节点名称。

```cs
示例：

[XmlRoot("Hierarchy")] 
public class HierarchyData
{
}

输出：

<Hierarchy>
</Hierarchy>
```

##### 2.`[XmlAttribute("")]`

> 作用：将成员序列化为 XML 属性。

```cs
示例：

[XmlAttribute("SceneName")] public string sceneName { get; set; }

输出：

<Hierarchy SceneName="SampleScene">
</Hierarchy>
```

==适合简单信息、元数据、标识信息。==

##### 3.`[XmlElement("")]`

> 作用：将成员序列化为 XML 子节点。

```cs
示例：

[XmlElement("GameObject")] public List<HierarchyNode> roots;

输出：


<GameObject></GameObject>
<GameObject></GameObject>

```

==用于**将成员序列化**为 XML 子元素，常用于复杂对象和集合展开。==

##### 4.`[XmlArray("")] / [XmlArrayItem("")]`

> 作用：用于控制集合结构

```cs
示例：

[XmlArray("Children")]
[XmlArrayItem("GameObject")]
public List<HierarchyNode> children;

输出：

<Children>
    <GameObject></GameObject>
    <GameObject></GameObject>
</Children>
```

❗ **和`[XmlElement("")]`的区别：** XmlArray 会增加一层包装节点，而 XmlElement 不会。

---
## 我在哪个项目里用过

#HierarchyTool 

其中的数据结构：
```cs
[XmlRoot("Hierarchy")]
public class HierarchyData
{
    [XmlAttribute("SceneName")]
    public string sceneName;

    [XmlAttribute("ExportTime")]
    public DateTime exportTime;

    [XmlElement("GameObject")]
    public List<HierarchyNode> roots;
}

HierarchyNode
{
    Name
    ID
    Active
    Script
    Children
}
```

生成：
```xml
<Hierarchy SceneName="SampleScene" ExportTime="2026-07-07">
    <GameObject Name="Player">
        <GameObject Name="Camera"/>
    </GameObject>
</Hierarchy>
```

---
## 总结

**Attribute 本质：**

> 给代码添加额外元数据，让框架通过反射读取并执行特殊行为。


XML 序列化中：
```test
Class
 |
 | Attribute描述
 ↓
XmlSerializer
 |
 | Reflection读取
 ↓
XML
```

在工具开发中，Attribute 可以让数据模型和输出格式解耦，不需要为了 XML 格式修改原始数据结构。

---
## 相关知识

