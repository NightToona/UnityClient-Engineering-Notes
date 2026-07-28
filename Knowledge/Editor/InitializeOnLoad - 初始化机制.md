# `[InitializeOnLoad]`

#Editor 

## 是什么

在 Unity Editor 加载该程序集或重构代码时，自动执行该类的静态初始化。

---
## 为什么需要它

用于对 Unity Editor 进行工具拓展时添加的额外脚本代码，引入到 Unity 中统一控制运行。

触发时机：
- Unity 启动时
- Domain Reload
- 脚本重新编译后重新加载

除了`[InitializeOnLoad]`外，`[InitializeOnLoadMethod]`的作用也是相同的。

---
## 核心 API

```csharp
[InitializeOnLoad]
public class TcpServer()
{
	static TcpServer()
	{
		Start();
	}
	
	[InitializeOnLoadMethod]
	static void Init() 
	{
	}
}
```

`[InitializeOnLoad]`作用于类，`[InitializeOnLoadMethod]`作用于方法。

第一个特性，需要 静态构造函数 作为入口。适合：
- 管理器
- 单例系统
- Editor工具初始化

第二个特性，直接作用于调用该特性的方法，不需要类。


---
## 我在哪个项目里用过

#HierarchyTool 

---
## 容易踩坑

**注意：**
**InitializeOnLoad / InitializeOnLoadMethod** 不是 Update，它不会每帧执行。它是在特定时机执行一次初始化。（上方触发时机时会执行）


**用法对比表格**：

|      | InitializeOnLoad | InitializeOnLoadMethod |
| ---- | ---------------- | ---------------------- |
| 作用对象 | Class            | Method                 |
| 入口   | 静态构造函数           | 指定方法                   |
| 适合   | 系统初始化            | 单个初始化函数                |
| 复杂度  | 稍高               | 简单                     |


---
## 相关知识

