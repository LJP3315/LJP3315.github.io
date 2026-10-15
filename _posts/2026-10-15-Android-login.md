---
title: "安卓 登录界面的两种写法"
date: 2026-10-15
categories: school
tags: Android
---

用于记录实验四中两种登录界面的写法，我愿称之为“茴香豆的N种不同写法”

## 写法一 (Column 垂直线性布局函数)

```kotlin
// 仅写Composable注解部分的方法，忽视主函数，减少代码量
// 展现核心的功能，修饰部分的代码会在之后补齐

@Composable
fun LoginScreenColumn() {
    // 保存用户输入
    // mutableStateOf()  创建初始值为空字符串的状态
    // remember  界面存在保存当前状态
    var username by remember { mutableStateOf("") }
    var password by remember { mutableStateOf("") }
    
    // 纵向排列组件
    Column {
        Text("登录")
        OutlinedTextField(
            // 关键参数：value(输入框当前的内容)
            // onValueChange(函数参数，输入发生变化时调用)
        	value = username,
            onValueChange = {username = it},
            // 输入框为空时，框中默认内容
            label = { Text("用户名") }
        )
        
        OutlinedTextField(
            value = password,
            onValueChange = {password = it},
            label = { Text("密码") },
            visualTransformation = PasswordVisualTransformation()
        )
        
        Button(onClick = {
            // 处理登录逻辑
        }) {
            Text("登录")
        }
    }
}
```

---

## 写法二  (ConstraintLayout 约束布局函数)

ConstraintLayout:约束布局，通过指定组件与其他组件、父容器之间的关系进行布局



```kotlin

@Composable
fun LoginScreenConstraint() {
    var username by remember { mutableStateOf("") }
    var password by remember { mutableStateOf("") }

    ConstraintLayout(
        modifier = Modifier
            .fillMaxSize()
            .padding(24.dp)
    ) {
        val (title, user, pass, login) = createRefs()

        Text(
            text = "登录",
            modifier = Modifier.constrainAs(title) {
                top.linkTo(parent.top, 8.dp)
                start.linkTo(parent.start)
                end.linkTo(parent.end)
            }
        )

        OutlinedTextField(
            value = username,
            onValueChange = { username = it },
            label = { Text("用户名") },
            modifier = Modifier.constrainAs(user) {
                // 用户名输入框顶部距离标题底部 24.dp
                top.linkTo(title.bottom, 24.dp)
                // 左侧连接父容器左侧
                // 右侧连接父容器右侧
                start.linkTo(parent.start)
                end.linkTo(parent.end)
                // 输入框填满左右约束间的空间
                width = Dimension.fillToConstraints
            }
        )

        OutlinedTextField(
            value = password,
            onValueChange = { password = it },
            label = { Text("密码") },
            visualTransformation = PasswordVisualTransformation(),
            modifier = Modifier.constrainAs(pass) {
                top.linkTo(user.bottom, 16.dp)
                start.linkTo(parent.start)
                end.linkTo(parent.end)
                width = Dimension.fillToConstraints
            }
        )

        Button(
            onClick = {},
            modifier = Modifier.constrainAs(login) {
                top.linkTo(pass.bottom, 24.dp)
                start.linkTo(parent.start)
                end.linkTo(parent.end)
                width = Dimension.fillToConstraints
            }
        ) {
            Text("登录")
        }
    }
}

```













