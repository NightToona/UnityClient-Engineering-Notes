# Transform

#Editor 

## 是什么

Unity 中用于维护场景节点树的组件。
每个 GameObject 都必定有且仅有一个 Transform 组件。[^1]

负责控制：
- 位置（Position）
- 旋转（Rotation）
- 缩放（Scale）
- 父子关系（Parenting State）

---
## 为什么需要它

通过 Transform 组件，
可以完成对物体的 **运动、大小、父子关系与查找** 的控制。

---
## 核心 API

```csharp
//获取父对象
[Transform] <= ( transform.parent; )

//获取根对象
[Transform] <= ( transform.root; )

//获取子对象数量（不探查孙对象数量）
[int] <= ( transform.childCount; )

//获取指定子对象
[Transform] <= ( transform.GetChild(index); )

//遍历子对象（坑点3），也可使用for循环
foreach(var child in transform){ }
```

---
## 我在哪个项目里用过

#HierarchyTool  用于递归构建节点树时使用。

和其余需要控制物体运动的广泛场景。

---
## 容易踩坑

1.  `.parent` 返回的是 Transform，不是 GameObject；
2.  `GetChild()` 和 `childCount` 得到的参数都只是 **子对象**，即下一层级，不涉及到孙对象；
3.  Transform 组件中，迭代器模式通过 `IEnumerable` 和 `IEnumerator` 接口实现集合遍历。

---
## 相关知识

[^1]: [Unity - 脚本API：Transform](https://docs.unity.cn/ScriptReference/Transform.html)
