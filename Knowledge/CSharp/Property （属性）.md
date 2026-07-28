# Property （属性）

#CSharp

## 是什么

类似于Java中用于保护类中的变量不被直接访问修改，起到安全防护作用。
（对外暴露一个“像变量一样访问”的接口，但实际执行的是方法）

他是 C# 提供的一种访问成员变量的方式，一个变量包含 `get / set`访问器 方法，
未声明属性会由编译器自动生成隐藏字段。

---
## 为什么需要它

如果直接通过 public 字段访问一个变量容易造成：
- 错误修改
- 值访问不安全
- 错误设置值与类型 等情况。

所以 Property 可以在 读取和写入 时加入定义逻辑：
- 数据检查
- 修改限制
- 触发事件
- 输出格式
- 日志 等。

以此来保护对象内部状态。

---
## 核心 API

```csharp
// 自动属性
public int HP { get; set; }

// 只读取
public int HP { get; }

// 类内部私有 Set
public int HP { get; private set; }

// 完整属性
public int HP
{
	get => hp;
	
	set
	{
		hp = Mathf.Max(0, value);
	}
}
```

完整属性中的 `hp` 是指其内部真正存储值的变量，但外部无法访问它。

除此之外访问修饰符也可以修饰 `get / set` ，但其修饰范围决定了外部修饰范围（其他类对该类的变量操作）。

---
### 拓展：表达式主体属性

`=>` 是 C# 提供的简化语法，用于替代只有单条语句的成员。

例如： 
```cs
public int HP => hp; 
```

等价于： 
```cs
public int HP 
{
	get { return hp; }
} 
```

注意： 
- `=>` 可以用于简化 get 
- set 没有独立符号 
- 需要复杂逻辑时仍使用完整属性

---
## 我在哪个项目里用过

在Unity中的很多变量和API中都会见到，例如：
- transform.position
- gameObject.activeSelf
- Time.deltaTime
这些很多都是 Property 。

---
## 容易踩坑

不要误认为访问器没有开销，实际上只要执行都会出现开销。

⭕**不要 默认认为读取没有成本！**

---
## 相关知识

