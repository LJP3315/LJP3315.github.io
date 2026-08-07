---
title: "Unity入门-PlayerPrefs"
date: 2026-08-06
categories: tech
tags: Unity入门
---

1. 什么是 PlayerPrefs ？

   Unity 提供的用于存储玩家数据的公共类

2. PlayerPrefs 存储

   通过键值对来存储，只能存储 `int`，`float`，`string`三种类型的数据

3. PlayerPrefs 可以作为本地数据存储工具，保存游戏音量、是否开启音乐等简单的设置选项

---

举例：

```csharp
PlayerPrefs.SetInt("Score", 100);
// 表示保存一个叫 Score 的整数，值为 100

// 数据的读取：
int score = PlayerPrefs.GetInt("Score");
```

---

PlayerPrefs 的使用流程

```bash
保存数据 -> 退出游戏 -> 再次打开游戏 -> 读取数据
```

---

