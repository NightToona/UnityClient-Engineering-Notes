# Lambda表达式 与 闭包捕获变量

#CSharp

## 场景

之前有次写代码，
在使用 Lambda 批量为 Button 添加监听事件时，需要对每个 Button 事件对应到其相关的标签。

但标准情况下事件监听不允许带参方法传入，于是就用了 Lambda 来解决。
问题就出现了。

---
## 现象

循环绑定按钮事件：

```cs
for (int i = 0; i < buttons.Count; i++)
{
    buttons[i].onClick.AddListener( () => Click(i) );
}

void Click(int i)
{
	int num = i;
	// 执行有关第i的响应事件
}
```

点击任意按钮：

```Debug-Log
最后一个索引
最后一个索引
最后一个索引
```

而不是：

```Debug-Log
0
1
2
...
```

---
## 原因

Lambda 方法中所捕获的变量是 **引用变量（地址）**，

不是 **传递变量（复制）**，也不是 **快照（Snapshot）**。

每次传递进去的是这个 i 的地址，而不是传递 i 的值。

所以 `for()` 循环完成后，所有的事件中的 i 都是指向循环最后一次的 i 变量。

导致所有 Lambda 读取到的都是同一个变量。

---
## 怎么定位

用最简单的 C# 代码，在 Rider 中做个测试，
不一定需要是事件监听，
只要是符合 *变量捕获* 这个条件就可以。

保存、编译后，
使用 ==IL功能== 查看编译出的中间语言（Intermediate Language）

**使用 Rider 查看 IL 代码后发现：**
  
编译器会生成隐藏类（DisplayClass），  
用于保存被 Lambda 捕获的变量。  
因此多个 Lambda 实际访问的是同一个变量。

试试，看完IL你就懂了。

---
## 正确做法

创建局部副本temp：

```cs
for (int i = 0; i < buttons.Count; i++)
{
	int temp = i; // 局部副本
	buttons[i].onClick.AddListener(() => Click(temp));
}
```

此时每轮循环都会产生新的变量：

```
temp = 0
temp = 1
temp = 2
···
```

Lambda 捕获的是每次创建出来的不同地址的临时变量（`temp`）。

---
## 在哪跌过

**UI、事件监听、HierarchyTool**

当初是为了自动给按钮分配标签，自动按序号展开对应信息。

---
## 收获

**Lambda：**
捕获变量 ≠ 捕获变量当前值

**循环中绑定事件时：**
- 先创建局部副本
- 再进行捕获

---
## 原文指路

- [[Lambda专题 - 深入捕获变量问题]]