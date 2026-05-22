---
title: "MySQL数据库连接"
date: 2026-05-21
categories: tech
tags: JavaEE
---

PS: Java实验7的主要内容，主要是使用数据库进行数据持久化

```xml
<!--        通过快捷键 Alt + Insert 快速添加依赖-->
<!--        添加mysql连接依赖-->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>8.0.33</version>
</dependency>
```

在使用数据库之前，在 pom.xml 文件添加连接依赖

---

具体为 LoginServlet 和 RegisterServlet 的验证逻辑部分的修改

```bash
# RegisterServlet 验证逻辑

设置编码
获取request输入
验证用户名或者密码是否为空
判断两次密码是否输入一致

使用sql查询，检查用户是否存在
"SELECT id FROM users WHERE username = ?"

将输入的数据插入数据库，在插入之前会验证是否插入成功
"INSERT INTO users (username, password) VALUES (?, ?)"
```

---

### DButil.java (数据库连接工具类)

```
public class DBUtil {
    private static final String URL      = "jdbc:mysql://localhost:3306/exp07?useSSL=false&serverTimezone=UTC&characterEncoding=utf8";
    private static final String USERNAME = "root";
    private static final String PASSWORD = "123456";

    static {
        try {
            // 通过反射方法，加载 MySQL 驱动
            // 导入了第三库，可以省略这段代码
            // 在 pom.xml 文件添加依赖，必须使用这段代码
            Class.forName("com.mysql.cj.jdbc.Driver");
        } catch (ClassNotFoundException e) {
            throw new RuntimeException("MySQL 驱动未找到，请检查 pom.xml 是否添加了 mysql-connector-java 依赖", e);
        }
    }
    
    public static Connection getConnection() throws SQLException {
        return DriverManager.getConnection(URL, USERNAME, PASSWORD);
    }
}
```

---

### RegisterServlet.java 代码逻辑

```java
@WebServlet(name = "RegisterServlet", value = "/register")
public class RegisterServlet extends HttpServlet {

    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {

        // 步骤1：设置编码
        // 步骤2：获取表单数据
        // 步骤3：服务端验证 — 检查输入是否为空
        // 步骤4：检查两次密码是否一致
        request.setCharacterEncoding("UTF-8");
        response.setContentType("text/html;charset=UTF-8");
        String username = request.getParameter("username");
        String password = request.getParameter("password");
        String confirmPassword = request.getParameter("confirmPassword");
        if (username == null || username.trim().isEmpty()) {
            request.setAttribute("registerSuccess", false);
            request.setAttribute("message", "用户名不能为空！");
            request.getRequestDispatcher("/registerResult.jsp").forward(request, response);
            return;
        }
        if (password == null || password.trim().isEmpty()) {
            request.setAttribute("registerSuccess", false);
            request.setAttribute("message", "密码不能为空！");
            request.getRequestDispatcher("/registerResult.jsp").forward(request, response);
            return;
        }
        if (!password.equals(confirmPassword)) {
            request.setAttribute("registerSuccess", false);
            request.setAttribute("message", "两次输入的密码不一致，请重新输入！");
            request.getRequestDispatcher("/registerResult.jsp").forward(request, response);
            return;
        }

        // 步骤5：先查询用户名是否已存在（防止重复注册）
        String checkSql = "SELECT id FROM users WHERE username = ?";
        try (Connection conn = DBUtil.getConnection();
             PreparedStatement pstmt = conn.prepareStatement(checkSql)) {
            pstmt.setString(1, username);
            try (ResultSet rs = pstmt.executeQuery()) {
                if (rs.next()) {
                    // 用户名已存在，注册失败
                    request.setAttribute("registerSuccess", false);
                    request.setAttribute("message", "用户名 \"" + username + "\" 已被占用，请换一个！");
                    request.getRequestDispatcher("/registerResult.jsp").forward(request, response);
                    return;
                }
            }
        } catch (SQLException e) {
            e.printStackTrace();
            request.setAttribute("registerSuccess", false);
            request.setAttribute("message", "数据库操作失败：" + e.getMessage());
            request.getRequestDispatcher("/registerResult.jsp").forward(request, response);
            return;
        }

        // 步骤6：用户名未被占用，执行插入操作
        String insertSql = "INSERT INTO users (username, password) VALUES (?, ?)";
        try (Connection conn = DBUtil.getConnection();
             PreparedStatement pstmt = conn.prepareStatement(insertSql)) {
            pstmt.setString(1, username);
            pstmt.setString(2, password);
            int rows = pstmt.executeUpdate(); // executeUpdate 返回受影响的行数
            if (rows > 0) {
                request.setAttribute("registerSuccess", true);
                request.setAttribute("message", "恭喜，" + username + "，注册成功！现在可以去登录了。");
            } else {
                request.setAttribute("registerSuccess", false);
                request.setAttribute("message", "注册失败，请稍后重试！");
            }
        } catch (SQLException e) {
            e.printStackTrace();
            request.setAttribute("registerSuccess", false);
            request.setAttribute("message", "数据库操作失败：" + e.getMessage());
        }
        // 步骤7：转发到结果页面
        request.getRequestDispatcher("/registerResult.jsp").forward(request, response);
    }
}
```

### LoginServlet.java 代码逻辑

```java
@WebServlet(name = "LoginServlet", value = "/login")
public class LoginServlet extends HttpServlet {

    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {

        // 步骤1：设置请求编码，防止中文乱码
        // 步骤2：从表单获取用户输入
        // 步骤3：服务端验证 —— 检查用户名和密码是否为空
        request.setCharacterEncoding("UTF-8");
        response.setContentType("text/html;charset=UTF-8");        
        String username = request.getParameter("username");
        String password = request.getParameter("password");
        if (username == null || username.trim().isEmpty()) {
            request.setAttribute("message", "用户名不能为空！");
            request.setAttribute("loginSuccess", false);
            request.getRequestDispatcher("/loginResult.jsp").forward(request, response);
            return;
        }
        if (password == null || password.trim().isEmpty()) {
            request.setAttribute("message", "密码不能为空！");
            request.setAttribute("loginSuccess", false);
            request.getRequestDispatcher("/loginResult.jsp").forward(request, response);
            return;
        }

        // 步骤4：编写 SQL 查询语句（? 是占位符，防止SQL注入）
        String sql = "SELECT * FROM users WHERE username = ? AND password = ?";
        try (Connection conn = DBUtil.getConnection();
             PreparedStatement pstmt = conn.prepareStatement(sql)) {
            // 将变量插入sql语句
            pstmt.setString(1, username);
            pstmt.setString(2, password);
            // 运行sql语句，获取结果集
            try (ResultSet rs = pstmt.executeQuery()) {
                if (rs.next()) {
                    request.setAttribute("loginSuccess", true);
                    request.setAttribute("message", "欢迎回来，" + username + "！");
                } else {
                    request.setAttribute("loginSuccess", false);
                    request.setAttribute("message", "用户名或密码错误，请重试！");
                }
            }
        } catch (SQLException e) {
            e.printStackTrace();
            request.setAttribute("loginSuccess", false);
            request.setAttribute("message", "数据库连接失败：" + e.getMessage());
        }
        request.getRequestDispatcher("/loginResult.jsp").forward(request, response);
    }
}
```

---

### 总结

相比 Servlet 那次实验

区别在于在 pom.xml 文件加入了 mysql 连接依赖

在Servlet代码文件，验证逻辑都替换为了数据库查询验证

新增的 DBUtil 数据库连接类(基于配置依赖的情况下)，为连接数据库提供方法接口

