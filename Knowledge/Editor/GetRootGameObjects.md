# Scene.GetRootGameObjects

#Editor 

## 是什么

用于按序返回场景中的所有 **根游戏对象**。[^1]
包括 *激活对象* 与 *未激活对象*。

**例如：**
Canvas  
├─Panel
Player  
├─Weapon
Environment

**返回：**
Canvas  
Player  
Environment

---
## 为什么需要它

用于查找当前层级中所有物体对象，用途广泛：
- 遍历 Hierarchy；
- 导出场景结构；
- 场景分析工具；
- 统计工具 等等。

---
## 核心 API

```csharp
// 获取最根物体，即父物体为null的对象
GameObject[] roots = SceneManager.GetActiveScene().GetRootGameObjects()
```


---
## 我在哪个项目里用过

#HierarchyTool 

---
## 容易踩坑

需注意该方法只返回 根部对象，对于获取其 **子对象** 需要通过**迭代**实现。
使用 `Transform` 组件进行迭代，获取 `.gameObject / .name` 等。[^2]

---
## 相关知识

[^1]: [Unity - 脚本API：SceneManagement.Scene.GetRootGameObjects](https://docs.unity.cn/ScriptReference/SceneManagement.Scene.GetRootGameObjects.html)

[^2]: [Unity - Scripting API: Transform](https://docs.unity.cn/ScriptReference/Transform.html)
