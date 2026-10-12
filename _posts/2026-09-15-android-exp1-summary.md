---
title: "Android实验一总结"
date: 2026-09-15
categories: school
tags: Android
---

实验一在 HelloAndroid 模板的基础上，把原本静态的问候语改成了「点击按钮显示当前时间」，主要修改的代码如下所示：

```kotlin
@Composable
fun Greeting(name: String, modifier: Modifier = Modifier) {
    var displayText by remember { mutableStateOf("Hello $name!") }
    Column(modifier = modifier) {
        Text(
            text = displayText
        )
        Button(onClick = {
            val currentTime = getCurrentTime().format(Date())
            displayText = "当前时间：$currentTime"
        }) {
            Text("显示时间")
        }
    }
}

fun getCurrentTime() : SimpleDateFormat {
    return SimpleDateFormat("HH:mm:ss", Locale.getDefault())
}
```

---

### 逐行解释代码

**1. 状态的定义**

```kotlin
var displayText by remember { mutableStateOf("Hello $name!") }
```

这行代码定义了一个可变的状态变量，并让界面能够自动响应它的变化。

* `mutableStateOf()`：这个方法将 displayText 包装成 Compose 的状态，当值改变时，UI 界面会自动重新绘制
* `by remember`：组件重新绘制时记住变量的值，防止重置回初始值。`by` 为 Kotlin 的委托特性

**2. 布局与交互组件**

* `Column`：垂直布局容器，传入参数为 modifier 样式
* `Button`：按钮组件，当用户点击按钮，触发 onClick 方法

**3. 时间的格式化**

```kotlin
SimpleDateFormat("HH:mm:ss", Locale.getDefault())
```

* 定义时间的输出格式：`HH` 表示 24 小时制的小时，`mm` 表示分钟，`ss` 表示秒
* 使用的是 Java 标准库：`java.text.SimpleDateFormat` 和 `java.util.Locale`

**4. 获取当前时间**

* `Date()`：构造函数，获取系统当前的绝对时间
* `SimpleDateFormat().format()`：将传入的参数进行格式化
