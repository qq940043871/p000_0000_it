# Java基础 - 架构师学习笔记

## Java基础语法

Java基础语法是学习Java编程的第一步，掌握好基础知识对于后续深入学习至关重要。本章将介绍Java的基本语法元素，包括数据类型、变量、运算符、控制流程等。

### 数据类型

Java是一种强类型语言，每个变量都必须声明其数据类型。Java的数据类型分为两大类：

#### 基本数据类型

| 数据类型 | 大小 | 默认值 | 取值范围 |
|---------|------|--------|----------|
| byte    | 8位  | 0      | -128 ~ 127 |
| short   | 16位 | 0      | -32,768 ~ 32,767 |
| int     | 32位 | 0      | -2,147,483,648 ~ 2,147,483,647 |
| long    | 64位 | 0L     | -9,223,372,036,854,775,808 ~ 9,223,372,036,854,775,807 |
| float   | 32位 | 0.0f   | ±3.40282347E+38F |
| double  | 64位 | 0.0d   | ±1.79769313486231570E+308 |
| char    | 16位 | '\u0000' | 0 ~ 65,535 |
| boolean | 未明确规定 | false | true 或 false |

#### 引用数据类型

引用数据类型包括类、接口、数组等。它们的默认值为null。

### 变量

变量是存储数据的容器。在Java中，变量必须先声明后使用。

```java
// 变量声明和初始化
int age;           // 声明变量
age = 25;          // 赋值

// 声明并初始化
int score = 95;

// 常量声明
final double PI = 3.14159;
```

### 运算符

Java提供了丰富的运算符，用于执行各种数学和逻辑操作。

#### 算术运算符

| 运算符 | 描述 | 示例 |
|-------|------|------|
| +     | 加法 | a + b |
| -     | 减法 | a - b |
| *     | 乘法 | a * b |
| /     | 除法 | a / b |
| %     | 取模 | a % b |
| ++    | 自增 | ++a 或 a++ |
| --    | 自减 | --a 或 a-- |

### 控制流程

控制流程语句用于控制程序的执行顺序。

#### 条件语句

```java
// if-else语句
if (age >= 18) {
    System.out.println("成年人");
} else {
    System.out.println("未成年人");
}

// switch语句
switch (grade) {
    case 'A':
        System.out.println("优秀");
        break;
    case 'B':
        System.out.println("良好");
        break;
    case 'C':
        System.out.println("及格");
        break;
    default:
        System.out.println("不及格");
}
```

#### 循环语句

```java
// for循环
for (int i = 0; i < 10; i++) {
    System.out.println(i);
}

// while循环
int i = 0;
while (i < 10) {
    System.out.println(i);
    i++;
}

// do-while循环
int j = 0;
do {
    System.out.println(j);
    j++;
} while (j < 10);
```

## 实践练习

尝试编写一个简单的Java程序，实现以下功能：

- 声明几个不同类型的变量并赋值
- 使用各种运算符进行计算
- 使用条件语句判断数值范围
- 使用循环语句输出数列
