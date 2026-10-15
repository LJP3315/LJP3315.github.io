---
title: "安卓 Kotlin语法"
date: 2026-10-15
categories: school
tags: Android
---

主要记录实验三题目出现的语法内容，并进行简单的拓展

## 实验1

### 内容：

学生成绩等级判断。使用when表达式实现成绩等级映射，并在main() 函数中计算多名学生的统计数据（平均分、最高分、最低分）；

```kotlin
package exp3.实验1

// 声明学生类：
// 数据类，自动生成常用方法：toString(), equals()......
data class Student(val name: String, val score: Int)

// 通过分数获得学生的成绩等级
fun getGrade(score: Int) : String {
    // 使用 when 表达式判断等级
    // 注意 when 中一定需要 else 保证覆盖所有分支
    return when (score) {
        in 90..100 -> "A"
        in 80..89 -> "B"
        in 70..79 -> "C"
        in 60..69 -> "D"
        in 0..59 -> "E"
        else -> "Error"
    }
}

fun main() {
    // 创建学生列表
    val students = listOf(
        Student("张三", 99),
        Student("李四", 80),
        Student("王五", 70),
        Student("赵六", 60),
        Student("钱七", 59),
    )

    // 输出学生的基本信息：姓名+分数+等级
    for (student in students) {
        val grade = getGrade(student.score)
        println("学生姓名：${student.name}，学生分数：${student.score}，学生分数等级：$grade")
    }

    // 获得学生的统计数据，平均分，最高分，最低分
    // 通过 map 获取学生类分数属性的映射
    // 相当于将学生列表转换为了分数列表
    val scores = students.map {it.score}
    val avgScore = scores.average()
    val maxScore = scores.max()
    val minScore = scores.min()

    // 输出学生统计数据：全体学生的平均分，最高分，最低分
    println("全体学生的平均分为${avgScore}，最高分为$maxScore，最低分为$minScore")
}
```

### 结果：

![](/assets/images/School/26-10-15-1.jpg){: loading="lazy"}

---

## 实验2

### 内容：

学生类的设计与使用。设计Student类，学习构造函数、属性声明及成员函数的使用，实现学生信息的打印与及格判断

```kotlin
package exp3.实验2

// Kotlin类设计：
// 主构造函数直接写在类名后面
// 放在构造函数的参数会自动被声明为该类的成员属性
// init 初始化块
// 校验逻辑书写
// 成员函数，写在函数内部，可直接访问类的属性，实现对象的行为

// 创建学生类，成员属性为：id,name,score
class Student(val id : String, val name : String, var score : Double) {
    // 初始化块
    // 判断传入的分数是否正常
    // 创建对象时调用
    init {
        if (score < 0 || score > 100) {
            println("ERROR!! 学生{$name}的分数存在问题！")
        }
    }

    // 内部成员方法，判断是否及格
    fun isPass() : Boolean {
        return score >= 60.0
    }

    // 打印当前学生的详细信息
    fun printInfo() {
        // 类比 C++ 中的 {(bool) ? 字符串1 : 字符串2}
        val passStatus = if (isPass()) "及格" else "不及格"
        println("学号：$id，姓名：$name，成绩：$score，状态：$passStatus")
    }
}

fun main() {
    // 创建学生列表
    val students = listOf(
        Student("1", "张三", 100.0),
        Student("2", "李四", 80.0),
        Student("3", "王五", 40.0),
        Student("4", "赵六", -60.0),
        Student("5", "钱七", 1000.0),
    )

    // 打印学生的详细信息
    for (student in students) {
        student.printInfo()
    }
}
```

### 结果：

![](/assets/images/School/26-10-15-2.jpg){: loading="lazy"}

---

## 实验3

### 内容：

集合与Lambda。结合data class 与Kotlin集合操作，练习使用filter、map及sortedByDescending等常用Lambda高阶函数进行数据处理与排序；

```kotlin
package exp3.实验3

// filter(过滤，根据你给出的条件，保留符合的元素)
// map(映射，类比之前讲学生列表映射为分数列表)
// sortedBy / sortedByDescending(排序)
// 前者为升序，后者为降序

// lambda表达式示例：
// 通过lambda表达式，我可以在主函数中实现函数的声明
//fun main() {
//    val sum: (Int, Int) -> Int = { x, y ->
//        println("sum of $x, $y")
//        x + y
//    }
//    val plus: (Int, Int, Int) -> Int = { x, y, z ->
//        println("三个数的和为：")
//        x + y + z
//    }
//    println(plus(1,2, 1000))
//}

data class Student(val name: String, val score: Int, val gender: String)

fun main() {
    val students = listOf(
        Student("张三", 85, "男"),
        Student("李四", 59, "男"),
        Student("王五", 92, "女"),
        Student("赵六", 76, "女"),
        Student("孙七", 45, "男"),
        Student("周八", 88, "男")
    )

    println("全体学生名单:")
    students.forEach {
        println("姓名：${it.name}, 分数：${it.score}")
    }

    println("及格学生名单:")
    val passStudents = students.filter { it.score >= 60 }
    passStudents.forEach {
        println("姓名：${it.name}, 分数：${it.score}")
    }

    val passNames = passStudents.map { it.name }
    println("及格学生姓名列表：")
    println(passNames)

    val sortedStudents = students.sortedByDescending { it.score }

    sortedStudents.forEachIndexed { i, student ->
        println("第${i+1}名: ${student.name}")
    }

    println("优秀学生姓名（链式调用）")
    val topStudentNames = students
        .filter { it.score >= 80}
        .sortedByDescending { it.score }
        .map { it.name }

    println(topStudentNames)
}
```

### 结果：

![](/assets/images/School/26-10-15-3.jpg){: loading="lazy"}
