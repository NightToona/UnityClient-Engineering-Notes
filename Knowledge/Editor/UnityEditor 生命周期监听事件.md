# Unity Editor 生命周期监听事件

#Editor 

## 是什么

Unity Editor 提供了一刺裂生命周期事件API，用于监听 Editor 状态变化。

就像 Unity运行时 提供的API一样，只不过 Editor 考虑的内容会更多：
- 编辑器关闭
- 脚本重新编译
- 数据变化
- Editor刷新等等。

因此通常通过事件回调，在特定时机执行逻辑。

---
## 核心 API（使用+=绑定）

#### 1. `EditorApplication.quitting`

**作用**：监听 UnityEditor 即将关闭事件。

**常用于**：
- 关闭 Socket
- 保存数据
- 清理临时文件
- 停止后台任务/服务

#### 2. `AssemblyReloadEvents.beforeAssemblyReload`

**作用**：监听程序集重新加载之前

**常见触发**：
- 修改 C# 脚本
- Unity 重新编译
- Domain Reload（播放时 域重新加载）

#### 3. `EditorApplication.update`

**作用**：类似于游戏中的 Update，每次 Editor 刷新时调用。
**==注意==**：这不是游戏运行时的Updat）。

**常用于**：
- Editor 工具轮询
- 检测状态变化
- 更新窗口内容

#### 4. `EditorApplication.hierarchyChanged`

**作用**：监听Hierarchy窗口内容结构变化（包括但不限于：创建删除GameObject、修改层级与物体基础属性）。

**常用于**：
- Hierarchy监控工具
- 自动刷新数据
- Editor扩展工具

---
## 我在哪个项目里用过

#HierarchyTool 

**用于**：
- `quitting`
    1. 关闭TCP连接
    2. 清理监听服务
- `beforeAssemblyReload`
    1. 防止脚本重新编译后残留Socket
- `hierarchyChanged`
    1. 监听Unity Hierarchy变化

---
## 容易踩坑

#### 1. 静态事件需要主动解绑

例如：
```cs
EditorApplication.update += Update;
```

如果生命周期管理不当，可能导致事件重复注册，所以必须要“挂载+卸载”同时存在的意识。

#### 2. Editor事件不是游戏生命周期

例如：
```cs
MonoBehaviour.Update()
```
和
```cs
EditorApplication.update
```

运行环境不同。
前者属于游戏运行时，后者属于编辑器工具环境。

---
## 相关知识

