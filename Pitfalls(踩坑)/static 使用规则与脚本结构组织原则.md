# static 使用规则与脚本结构组织原则

#Pitfalls  

## 场景

在 Unity 工具开发（Hierarchy Tool / Editor Tool / TCP结构设计）中，  
经常会出现以下结构混用：  
  
- static 数据类（全局 List / Node）  
- 实例逻辑类（MonoBehaviour / EditorWindow）  
- 工具方法类（Build / Parse / Traverse）

从而导致初始化混乱、数据易污染、逻辑混乱不易阅读维护。

---
## 本质

> `static` = **类级别共享状态**

不是用来“方便全局访问”，而是：

❗ 整个程序生命周期共享的一份数据

---
## 核心问题（踩坑点）

#### 1. static 初始化早于一切

static 字段在任何 MonoBehaviour / Editor 初始化前执行，导致：

`private static int port = Setting.port;` 
❌可能还是默认值

见该文：[[静态变量初始化生命周期问题]]

#### 2. static 数据 = 隐形全局状态

- Scene 切换不会回收；
- Editor Domain Reload 可能不一致；
- 多工具互相污染数据；

#### 3.数据类 static 化 = 结构崩坏起点

尤其是：
- Tree Node
- Hierarchy 结构
- Runtime 生成数据

👉 一旦 static：

> 所有实例变成“共享污染源”

---
## 📌如何使用

##### ✔合适情况
- 无状态工具方法 （Math / Helper）
- 全局唯一配置（且不会频繁变更）
- 缓存数据（有明确的生命周期，可控）
- 单例访问入口

##### ❌不合适情况
- 依赖外部初始化的数据
- 会变化的运行时数据（Hierarchy / Scene数据）
- 依赖 Unity 生命周期的对象
- 需要重建 / 重置的数据结构

---
## 正确结构组织原则

##### 1.数据类（Data）
- ❌ 不使用 `static`
- ✔ 每次重新生成，例如：`class Hierarchy`
##### 2.逻辑类（System）
- 可 `static` （但无状态），例如：`static class Builder`
##### 3.配置类（Config）
- 可 static，但必须显示初始化
```cs
static class Config
{
	public static int port;
}
```
##### 4.生命周期入口（Entry）
- 必须统一初始化：`Init()`

---
## 🚩顺序以及规则建议

> ❗ 数据不 static，逻辑可 static，配置必须显示初始化

一个脚本内（或模块内）：
1. Config / Field（配置 / 字段）
2. Data Definition（数据）
3. Public API
4. Core Logic（核心实现）
5. Private Helper（私有实现）

---
## 收获

- 不要指望 static 作为全局变量容器，static ≠ 全局变量容器
- 数据结构永远优先”实例化“
- 初始化顺序必须显示控制
- Unity 生命周期不可依赖隐式执行

> 不要用方便从而牺牲了安全性，所以不要滥用 `static`。

---
## 指路

- [[修饰符 - static、readonly、const]]