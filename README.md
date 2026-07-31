# 个人学习路线 | GameClient Engineering Notes

![Status|136](https://img.shields.io/badge/status-Keeping%20Update-orange)

这是我在游戏客户端方向学习与工程实践过程中的记录。

仓库包含：
- 项目开发记录
- 技术学习笔记
- 问题排查与踩坑记录
- 阶段性总结

内容主要来源于实际项目开发过程，而非单纯整理资料。

---

# Projects

## Unity Hierarchy Tool

一个用于在 Rider 中实时查看 Unity Hierarchy 的辅助工具。

目标：

> 减少 Unity 与 IDE 之间频繁切换，提高开发过程中查看场景信息的效率。

技术：
- Unity Editor Extension
- C#
- TCP通信
- XML序列化
- Rider Plugin
- Kotlin
- IntelliJ Platform

架构：

Unity Editor
↓
Hierarchy Data
↓
TCP
↓
Rider Plugin
↓
Tree UI


详细记录：
- 项目设计
- 通信协议
- 数据结构
- 开发过程
- 问题复盘


---

# Knowledge

技术学习记录：

## C# 

包含：
- 面向对象
- 异步编程
- TCP通信
- XML序列化
- Editor扩展


## Unity Editor

包含：
- Editor生命周期
- EditorWindow
- Hierarchy监听
- 编辑器工具开发


## Programming Pattern

包含：
- 状态机
- 状态控制
- 架构设计


## 快速学习

记录进入陌生技术领域的过程：
- Kotlin
- Rider Plugin开发


---
# Pitfalls

开发过程中遇到的问题与解决方案。

例如：
- TCP Buffer复用导致数据残留
- TCP连接状态异常
- XML编码差异
- 生命周期相关问题


---
# Daily Log

记录每日开发过程：

包括：
- 今日完成内容
- 遇到的问题
- 学习总结
- 下一步计划


---
# About

该仓库记录个人从游戏客户端方向学习到工程实践的过程。

重点关注：
- Unity工具开发
- 客户端架构
- 网络通信
- 编辑器扩展
- 工程化实践


---
## License

This repository is for personal learning and portfolio demonstration.

All rights reserved.

The content may be viewed for learning purposes, but redistribution, reproduction, or claiming as original work is not permitted without permission.