# 为什么不用 Resources.FindObjectsOfTypeAll 获取Hierarchy

#Pitfalls #HierarchyTool 

## 场景

开发 Hierarchy Tool 时，
尝试使用：
`Resources.FindObjectsOfTypeAll(typeof(对象))`

获取所有节点。

---
## 现象

Hierarchy中只有5个对象，
打印结果却出现13个对象。

并出现：
- InternalIdentityTransform
- 编辑器隐藏对象
- 其他内部对象

---
## 原因

**Resources.FindObjectsOfTypeAll 获取的是**：
"所有已加载到内存中的对象"  
  
**而不是**：  
"专门针对Hierarchy中的对象"

当然，内存中的对象也包含层级中的对象。

---
## 正确做法

改用：  
`SceneManager.GetActiveScene().GetRootGameObjects()  `
获取根对象。

最后对每个根对象，
递归遍历Transform树并储存。

---
## 收获

**获取Hierarchy结构时**：    
从 Scene 出发，不要从 Resources 出发。

**但只是要Hierarchy中所有对象**：
就可以使用 Resource 通过过滤得到。

---
## 指路

- [[FindObjectsOfTypeAll]]
- [[GetRootGameObjects]]