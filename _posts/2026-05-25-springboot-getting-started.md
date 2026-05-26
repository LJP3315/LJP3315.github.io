---
title: "SpringBoot入门"
date: 2026-05-25
categories: 技术
tags: JavaEE
---

PS: 实验八 SpringBoot 入门，做具体的事，关注具体的个体，经营好自己微观的生活和技能。



文件结构总括

```bash
  test/
  ├── pom.xml                          # Maven 项目配置
  └── src/
      ├── main/                        # 主代码
      │   ├── java/com/lyon/test/
      │   │   ├── TestApplication.java         # Spring Boot 启动类
      │   │   ├── controller/
      │   │   │   └── UserController.java      # 请求处理层（路由控制）
      │   │   ├── entity/
      │   │   │   └── User.java                # 实体类（数据模型）
      │   │   └── mapper/
      │   │       └── UserMapper.java          # 数据库操作层（ORM 映射）
      │   └── resources/
      │       ├── application.yaml             # Spring Boot 全局配置
      │       ├── schema.sql                   # 数据库建表脚本
      │       └── templates/                   # Thymeleaf 页面模板
      │           ├── index.html               # 首页
      │           ├── login.html               # 登录页
      │           ├── register.html            # 注册页
      │           └── result.html              # 操作结果页
      └── test/                        # 测试代码
          └── java/com/lyon/test/
              └── TestApplicationTests.java    # 启动类空测试
```

### 1. pom.xml 项目依赖

在新建项目时，添加以下依赖：

```bash
Spring Web --用于开发 Web 项目
MySQL Driver --JDBC驱动，连接 MySQL 数据库
MyBatis Framework --提供注解，用于操作数据库
Thymeleaf --模板引擎，渲染HTML页面
Lombok --自动生成实体的构造器
```

---

### 2. Application.java 程序的入口

作用：

* 初始化 Web 服务(默认端口为全局配置 application.yaml)
* 激活 MyBatis，Thymeleaf 框架组件
* 启动后，可通过浏览器进行访问 http://localhost:8080/

---

### 3. entity 实体类 - 数据模型

#### User.java 用户实体类

```java
package com.lyon.test.entity;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

// 通过 Lombok 依赖，直接为当前类自动生成各属性的getter,setter
// 生成全参构造与无参构造
@Data
@AllArgsConstructor
@NoArgsConstructor
public class User {
    private String username;
    private String password;
    
    // 生成 getUsername(), getPassword()
    // setUsername(), setPassword()
    // 重写 ToString() 相关逻辑
}
```

---

### 4.mapper 数据访问层 - 访问数据库

#### UserMapper.java 声明SQL语句

```java
package com.lyon.test.mapper;

import com.lyon.test.entity.User;
import org.apache.ibatis.annotations.Insert;
import org.apache.ibatis.annotations.Mapper;
import org.apache.ibatis.annotations.Select;

@Mapper  // 来自Spring自带的注解
public interface UserMapper {
    // 根据用户名查询用户
    @Select("select username, password from users where username = #{username}")
    User findByUsername(String username);

    // 插入用户
    @Insert("insert into users (username, password) values (#{username}, #{password})")
    void insert(User user);
}
```

---

### 5.controller 控制层 - 接收数据，返回页面

UserController.java 处理 4 个路由：首页 `/`、注册页 `/register`、登录页 `/login`，以及对应的 POST，表单提交逻辑

```java
package com.lyon.test.controller;

import com.lyon.test.entity.User;
import com.lyon.test.mapper.UserMapper;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestParam;

// @Controller, @Autowired, @GetMapping, PostMapping 源于Spring框架
@Controller
public class UserController {
    @Autowired
    private UserMapper userMapper;


    // 访问 localhost:8080/ 表示首页
    // 访问 localhost:8080/register 表示注册页面
    // 访问 localhost:8080/login 登录页面
    @GetMapping("/")
    public String showIndexPage() {
        return "index";
    }
    @GetMapping("/register")
    public String showRegisterPage() {
        return "register";
    }
    @GetMapping("/login")
    public String showLoginPage() {
        return "login";
    }

    // 表单提交逻辑：
    // 在对应网址获取数据表单，调用 doRegister
    // @RequestParam String name 通过 name 获取表单数据
    @PostMapping("/register")
    public String doRegister(@RequestParam String username,
                             @RequestParam String password,
                             Model model) {
        model.addAttribute("type", "注册");
        User existing = userMapper.findByUsername(username);
        if (existing != null) {
            model.addAttribute("error", "用户名已注册");
            return "result";
        }

        userMapper.insert(new User(username, password));
        model.addAttribute("username", username);
        return "result";
    }

    // 在对应网址获取数据表单，调用 doLogin
    @PostMapping("/login")
    public String doLogin(@RequestParam String username,
                          @RequestParam String password,
                          Model model) {
        model.addAttribute("type", "登录");

        User user = userMapper.findByUsername(username);
        if (user == null) {
            model.addAttribute("error", "用户不存在");
            return "result";
        }

        if (!user.getPassword().equals(password)) {
            model.addAttribute("error", "密码错误");
            return "result";
        }

        model.addAttribute("username", username);
        return "result";
    }
}
```

---

### 6. 数据库的创建

```mysql
# schema.sql

create database test;
use test;
create table users (
    username varchar(255) not null,
    password varchar(255) not null
);
```

---

### 7. application.yaml 配置文件

```yaml
spring:
  application:
    name: test

  datasource:
    url: jdbc:mysql://localhost:3306/test?useSSL=false&serverTimezone=Asia/Shanghai&characterEncoding=UTF-8
    username: root
    password: 123456
    driver-class-name: com.mysql.cj.jdbc.Driver

mybatis:
  type-aliases-package: com.lyon.test.entity
```

Spring Boot 的配置文件，用于设置应用程序的基本信息和数据库连接

* 程序命名为 test
* 配置数据库的地址，用户名，密码，MySQL驱动
  * localhost:3306：本地 MySQL 服务器
  * /test：使用名为 test 的数据库

---

### 8. result.html 结果显示界面 

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <title>操作结果</title>
</head>
<body>

<h2>操作结果</h2><br/>

<!--存在错误-->
<span th:if="${error}">
    <span th:text="${type}">操作类型</span>失败
    <br>
    原因:<span th:text="${error}">错误信息</span>
</span>

<!--不存在错误，error为空-->
<span th:unless="${error}">
    <span th:text="${type}">操作类型</span>成功
    <br>
    欢迎你，<span th:text="${username}">用户名</span>!
</span>

<br/><br/>

<a href="/">返回首页</a>
</body>
</html>
```

根据后端传过来的数据，动态地显示“登录/注册成功”或者“登录/注册失败”的页面

<span> 标签提供了一种将文本的一部分或者文档的一部分独立出来的方式

* th:if="\${error}"，th:unless="${error}" 类比 if-else
* th:text 用后端传过来的变量值，**替换**掉标签里面的原本的文字

----

### 总结

优先在 java 文件夹下创建 controller, mapper, entity 文件夹，在 resources/templates 文件夹下创建 html 文件作为前端

先在 entity 创建实体，并且进行数据库的建表

mapper 主要是 通过数据库的操作语句，看需求进行编写

controller 通过 @GetMapping(URL) 与 @PostMapping(URL) 注解在对应的网址执行前端的显示和表单的发送

