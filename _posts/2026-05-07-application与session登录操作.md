---
title: "application与session登录操作"
date: 2026-05-07
categories: 学习记录
tags: JavaEE
---

PS: 实验四中将application与session用于存储登录/注册信息

#### application（应用上下文）:

* 代表整个 Web 应用程序

  * 服务器启动时创建，用于所有用户之间的数据共享
  * 生命周期，当 Web 应用关闭时销毁

* ```java
  // 实例代码:
  // 存入数据（所有用户都能看到）
  application.setAttribute("totalVisits", 100);
  
  // 获取数据
  int visits = (Integer) application.getAttribute("totalVisits");
  ```

---

#### session（会话）：

* 用于跟踪单个用户与服务器之间的交互状态

  * 每个用户都有自己独立的 session 对话，互不干扰
  * 生命周期：调用 getSession() 是创建，关闭浏览器或者长时间未操作时销毁对象

* ```java
  // 实例代码
  // 存入数据
  session.setAttribute("username", "张三");
  
  // 获取数据
  String user = (String) session.getAttribute("username");
  ```

---

#### 修改的代码：

```java
// treatRegister.jsp

// 获取当前的 user 字典
// map的形式为 <username, password>
// 如果 user 为空，说明没有进行过注册操作，新建字典
// 判断是否包含当前注册的 username
// 没有注册，则进行注册操作
Map<String, String> users = (Map<String, String>) application.getAttribute("users");

if (users == null) {
  users = new HashMap<String, String>();
}

if (users.containsKey(username)) {
  message = "用户名已存在";
} else {
  users.put(username, password);
  application.setAttribute("users", users);
  message = "注册成功";
  flag = true;
}
```

---

```java
// treatLogin.jsp

// 在登录之前，获取网站的 user 对象
// 如果当前用户名存储在网站中，且密码相同
// 登录操作成功
Map<String, String> users = (Map<String, String>) application.getAttribute("users");
// 检查用户是否存在且密码正确
if (users != null && users.containsKey(username) && users.get(username).equals(password)) {
  message = "登录成功";
  flag = true;

  // 将用户名存储到session中，表示用户已登录
  session.setAttribute("currentUser", username);
} else {
  message = "用户名或密码错误";
}
```

---

#### 实验3与实验4的变化：

实验3：

用户的信息是在代码写好的，无法进行添加对象

实验4：

通过application记录登录操作所得到的用户信息

通过session获取用户信息，进行登录操作

---

#### 总结：

使用application用于存储用户数据

使用session用于创建临时的会话

