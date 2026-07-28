# EditorPrefs

#Editor 

## 是什么

> Unity Editor 提供的轻量级的值储存系统，用于保存编辑器环境中的用户配置数据。[^1]

类似于：
- PlayerPrefs：游戏运行时数据
- EditorPrefs：编辑器工具配置数据

用于保存 Editor 工具中的持久化配置，例如：
- TCP 端口号
- 窗口布局
- 用户工具偏好
- 插件初始化参数等

---
## 为什么需要它

Editor 扩展通常需要保存用户设置。

如果直接使用普通变量：
```cs
int port = 44571;
```

那么 Unity 关闭或文件重新编译后数据会丢失。

虽然可以通过文件 I/O 保存配置，但 EditorPrefs 提供了 Unity 内置的简单持久化方案，可以快速保存编辑器工具所需的小型配置数据。


---
## 核心 API

**数据写入**
```cs
EditorPrefs.SetInt();
EditorPrefs.SetFloat();
EditorPrefs.SetBool();
EditorPrefs.SetString();

// 传入参数为<String,Type>
```

**数据读取**
```cs
EditorPrefs.GetInt();
EditorPrefs.GetFloat();
EditorPrefs.GetBool();
EditorPrefs.GetString();

// 传入参数为<String>或<String, 默认值>
```

**数据管理**
```cs
EditorPrefs.Haskey(); // 获取键哈希值
EditorPrefs.DeleteKey(); // 删除指定键
EditorPrefs.DeleteAll(); // 删除全部键
```

---
## 与 PlayerPrefs 区别

|      | EditorPrefs  | PlayerPrefs |
| ---- | ------------ | ----------- |
| 使用环境 | Unity Editor | 游戏运行时       |
| 目标用户 | 开发者          | 玩家          |
| 用途   | 工具配置         | 游戏设置        |
| 命名空间 | UnityEditor  | UnityEngine |



---
## 我在哪个项目里用过

#HierarchyTool 

用于保存：
- TCP端口号
- 工具配置参数

---
## 容易踩坑

#### 1. 不适合保存大量数据

EditorPrefs **适合**：
- int
- float
- bool
- string

**不适合**：
- 大量结构化数据
- Hierarchy 树数据
- 配置文件等

复杂数据应当考虑使用 json、XML、文件存储等方式。

#### 2. 只限用于 Editor 环境中

因为 EditorPrefs 使用的是 UnityEditor API，Unity 打包文件后并不会打包相应的DLL文件。

---
## 相关知识

[^1]: [Unity - 脚本API：EditorPrefs](https://docs.unity.cn/ScriptReference/EditorPrefs.html)
