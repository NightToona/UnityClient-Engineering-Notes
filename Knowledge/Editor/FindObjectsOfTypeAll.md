# Resources.FindObjectsOfTypeAll

#Editor

## 是什么

获取当前已==加载到内存中==的所有<*指定类型对象*>[^1]。

与其相类似的API为 `Resources.FindObjectsOfType`[^2]，
不同在于即使==对象未激活==（ `Active == false`）也能获取。

---
## 它的用途

用于获取：
- 隐藏的指定类型对象；
- 获取内存中的内部对象；
- 获取 Prefer、材质、网格、纹理 等；

通过直接查找内存中的资源，进行 Editor工具开发 或是 Editor编辑拓展。

---
## 核心 API

```csharp
GameObject[] obj = Resources.FindObjectsOfTypeAll(typeof(对象));
```

**特点**：
- 可获取未激活对象
- 可获取资源
- 可获取内部对象

---
## 我在哪个项目里用过

#HierarchyTool 

---
## 容易踩坑

返回时需要对数据进行手动过滤信息流。

因为该函数可以返回任何类型的 unity 对象，包括 ==内部对象、Internal对象、Preview对象== 等等无用信息。

过滤时一般配合：

- `EditorUtility.IsPersistent(Object) => bool`
	确认对象是否储存在磁盘，当 `Object` 位于场景中返回`false`。[^3]

- `Object.hideFlags == HideFlags.(Properties API)`
	控制物体销毁、保存和检查器可见性。用于过滤当前物体是否为目标标签物体。[^4]

等手段。

---
## 相关知识

[^1]: [Unity - 脚本API：Resources.FindObjectsOfTypeAll](https://docs.unity.cn/ScriptReference/Resources.FindObjectsOfTypeAll.html)

[^2]: [Unity - 脚本API：Object.FindObjectOfType](https://docs.unity.cn/ScriptReference/Object.FindObjectOfType.html)

[^3]: [Unity - 脚本API：EditorUtility.IsPersistent](https://docs.unity.cn/ScriptReference/EditorUtility.IsPersistent.html)

[^4]: [Unity - 脚本API：HideFlags](https://docs.unity.cn/ScriptReference/HideFlags.html)
